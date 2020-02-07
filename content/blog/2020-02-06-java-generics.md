+++ 
date = "2020-02-06"
title = "Java Generics"
tags = ["java", "generics" ]
categories = ["blog"]
+++

I've been working for about a year now building applications in Java with Spring Boot.  Don't get me wrong, I think Java is capable, and Spring Boot is a great framework that has no comparison in .NET, but in terms of being able to get things done C# and .NET are by far my favorite language/environment. 

One thing I recently ran into is the use of generics in Java.  In particular I have a set of APIs and I'm conveying their requests and responses with DTOs.  I also have a database that's being developed via JPA entities.  Many of the APIs are just wrappers around being able to do something against the database.

Thus, I have a lot of boilerplate code to do this.

One of the key places this code can be reduced is via mapping libraries, and I've been using [Model Mapper](http://modelmapper.org/) as a way around this.  It worked great until still I started getting to collections.

My hope was to map a list of database entities to a list of DTOs.  The problem quickly became aparrent in Java's architecture.

Java is really two portions.  It's the language and the JRE (Java Runtime Environment).  The JRE is essentially an interpreter that interprets byte code produce by the compilation of Java language programs.  While generic are conveyed in the Java language, they are compiled out of the byte code that's interpreted by the JRE.  This means no reflection over generics at runtime over collections.  When I have a generic list, I just have a list of objects, and the language is doing all the casting to the appropriate types.

The rational was respectable.  Backwards compatibility.  Byte code with generics worked with byte code prior to them being implemented.

The key loss was reflection.  Reflection is generally the act of being able to iterate of type information at runtime.  Since it's not possible to reflect over generic information at runtime, in the case where I have a List of Entity and am trying to map to a List of DTO and have indicated that in my source code via generics, I can't find those types at runtime.  I just have two lists.

This means, libraries such as Model Mapper are stuck.  They simply can't find out the information about these properties at runtime, and cannot make simple inferences.

C# and .NET is much different in this regard.  When they released generics, they baked it into their CLR (common language runtime), the byte code interpreter for .NET.  This means, it is possible to find out the types during runtime while reflecting and thus, would be able to map in this case where Java cannot.

I miss .NET.