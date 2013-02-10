---
layout: post
title: "If else or Switch case to Polymorphism"
date: 2011-04-15 15:59
comments: true
categories: [design, oop, java]
---
If you try to express your logic in if-else way like this,
``` java
      private static String getSoundIfElseWay(String animal) {  
           if (animal.equalsIgnoreCase("Dog"))  
                return "Bark";  
           else if (animal.equalsIgnoreCase("Cat"))  
                return "Mew";  
           else if (animal.equalsIgnoreCase("Lion"))  
                return "Roar";  
           return "";  
      }  
```

then polymorphic way would be,
<!--more-->
``` java
 private static String getSoundPolymorphicWay(Animal animal){  
      return animal.say();  
 }
```
``` java
public abstract class Animal {  
      public abstract String say();  
 }
```
``` java
 public class Dog extends Animal{  
      @Override  
      public String say() {  
           return "Bark";  
      }  
 } 
```

``` java
 public class Cat extends Animal {  
      @Override  
      public String say() {  
           return "Mew";  
      }  
 }  
```
``` java
 public class Lion extends Animal {  
      @Override  
      public String say() {  
           return "Roar";  
      }  
 }

```

Look at the change in behavior based on the condition. If you could come up with a object than could take this behavior, push that to the corresponding object.

Same rule applies to switch-case also.