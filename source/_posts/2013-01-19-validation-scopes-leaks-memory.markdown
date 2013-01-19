---
layout: post
title: "Memory leaks with <i>validation_scopes</i>"
date: 2013-01-19 17:50
comments: true
categories: 
---

We had a requirement in our app to ignore few validations by just showing warnings to user and continuing the object to save.
The gem [_validation_scopes_](https://github.com/gtd/validation_scopes) seemed to be a right choice and we used it.

We faced memory issues and [figured out](/blog/2013/01/19/how-we-debugged-rails-memory-leak/)
<code>model.has_warnings?</code> of _validation_scopes_ was causing issue.

<!--more-->
Let's take a rails model class _Employee_
``` ruby
class Employee < ActiveRecord::Base
  attr_accessible :address, :age, :name
  validates_presence_of :name
  validation_scope :warnings do |s|
    s.validates_presence_of :address
  end
end
```

```
1.9.3p194 :001 > e = Employee.new(:name => 'Senthil V S')
 => #<Employee id: nil, name: "Senthil V S", age: nil, address: nil, created_at: nil, updated_at: nil>
1.9.3p194 :002 > e.valid?
 => true 
1.9.3p194 :003 > e.instance_variables
 => [:@attributes, :@association_cache, :@aggregation_cache, :@attributes_cache, 
 :@new_record, :@readonly, :@destroyed, :@marked_for_destruction, :@previously_changed,
 :@changed_attributes, :@mass_assignment_options, :@validation_context, :@errors] 
```

No problem till now. Now I call <code>has_warnings?</code> on this object.
```
1.9.3p194 :004 > e.has_warnings?
 => true 
1.9.3p194 :005 > e.instance_variables
 => [:@attributes, :@association_cache, :@aggregation_cache, :@attributes_cache,
 :@new_record, :@readonly, :@destroyed, :@marked_for_destruction, :@previously_changed,
 :@changed_attributes, :@mass_assignment_options, :@validation_context, :@errors, :@warnings] 
```

Notice a new instance variable(<code>@warnings</code>)gets set. Let's see what it has.

```
1.9.3p194 :034 > warning_object = e.instance_variable_get :@warnings
 => #<Employee id: nil, name: "Senthil V S", age: nil, address: nil, created_at: nil, updated_at: nil> 
1.9.3p194 :035 > warning_object.class
 => #<Class:0x007ffeddb0c1e0>
1.9.3p194 :036 > warning_object.class.ancestors
 => [#<Class:0x007ffeddb0c1e0>, ActiveModel::Validations::HelperMethods,
 ActiveModel::Validations, ActiveSupport::Callbacks, #<Class:0x007ffeddba5f70>,
 Delegator, #<Module:0x007ffedcb35eb0>, BasicObject]  
1.9.3p194 :037 > warning_object.__getobj__ == e
 => true 
```

What <code>@warnings</code>  has is an object of class defined during runtime by _DelegatorClass_ . 
The problem is that these runtime created classes don't get garbage collected.

```
1.9.3p194 :007 > warning_object_class_id = warning_object.class.object_id
 => 70366308884720 
1.9.3p194 :008 > e_object_id = e.object_id
 => 70366307844540 
```

We've taken the _object_id_ of the model object and runtime created delegator class(which is an instance of class _Class_ of course). Let's set the references to nil and run the GC.

```
1.9.3p194 :014 > local_variables
 => [:e_object_id, :warning_object_class_id, :warning_object, :e, :_]
 1.9.3p194 :009 > warning_object = e = nil
 => nil 
1.9.3p194 :010 > GC.start
 => nil 
```

Now let's use those object_ids to check whether they get GCed.

```
1.9.3p194 :011 > ObjectSpace._id2ref e_object_id
RangeError: 0x003fff6ec881bc is recycled object
	from (irb):11:in `_id2ref'
	from (irb):11
	from /Users/senthilvs/.rvm/gems/ruby-1.9.3-p194@global/gems/railties-3.2.10/lib/rails/commands/console.rb:47:in `start'
	from /Users/senthilvs/.rvm/gems/ruby-1.9.3-p194@global/gems/railties-3.2.10/lib/rails/commands/console.rb:8:in `start'
	from /Users/senthilvs/.rvm/gems/ruby-1.9.3-p194@global/gems/railties-3.2.10/lib/rails/commands.rb:41:in `<top (required)>'
	from script/rails:6:in `require'
	from script/rails:6:in `<main>'
1.9.3p194 :012 > ObjectSpace._id2ref warning_object_class_id
 => #<Class:0x007ffeddb0c1e0> 
```

So, the runtime created anonymous classes never get GCed and stay in memory. This the reason why
the memory was kept on increasing for every request.

We fixed it by replacing _validation_scopes_ with [_activemodel-warnings_](https://github.com/paneq/activemodel-warnings).