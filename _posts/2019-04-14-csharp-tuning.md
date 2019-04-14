---
title: C# tuning
categories: [c#, optimizations, .net]
category: .net
last_modified_at: 2019-04-14 14:16:01 0100
---

# C# Tuning

It is believed that 97% of the time your code can be optimized, but instead you should focus on the 3% that is easy to optimize and can lead to maximum improvement in performance. Also, keep in mind you aren't doing premature optimization, i.e. thinking of the performance while coding for the first time. It is always better to write the code first and then optimize it later.

Here are some of the tips that can improve C# code performance radically.

1. Use For loop instead of Foreach loop

   Foreach uses more local variable space and hence is slower than For loop. I ran an experiment in which values were inserted  into a list from a list using For and Foreach loop. I found that For loop was considerably faster than Foreach loop.

   |Type                |For/Foreach      |
   |:-------------------|----------------:|
   |Typed array         |Good for both    |
   |Array list          |Better with for  |
   |Generic collections |Better with for  |

2. Use StringBuilder instead of String for concatenation operations

   For heavy string concatenation operations, String adversely affects performance because String object creates a new object while concatenating two String values, whereas StringBuilder performs operations on the same object. Strings are faster for comparison operation, but a poor choice for concatenation.

3. Use direct assign method instead of getter-setter properties.

   The get and set blocks are like functions, so there is function call overhead for them every time we access our private variables.

4. Use the keyword "Using".

   If database connections are left open by mistake, the number of open connections keep rising and eventually database stops accepting further connections. "Using" keyword will ensure that the DataReader and Connection objects are closed before exiting the loop and deallocate the memory.

5. Use Array instead of ArrayList to store values of the same type.

   List stores the data in form of objects. So when we store the value types, it first converts it to a reference type. Thus, boxing-unboxing creates a performance bottleneck.

6. Substitute classes with structures, wherever possible.

   If you want to, say, define a group of constants or declare the value of fields at a global level, structures would serve your purpose. Structures are value types and value is stored in one location, whereas classes are reference types and reference is created to a memory location where the value is stored. It takes more time to access a value stored on heap and so classes offer poor performance.

7. Use TryGetValue instead ContainsKey

   TryGetValue. This method optimizes Dictionary usage. It gets a value (at a key) from a Dictionary. And it eliminates unneeded lookups, making programs better.
   Some notes. With TryGetValue, we can combine the "try" part of seeing if a key exists, and the "get" part of getting the existing value. This saves 1 lookup.

8. Use Return instead a ref or out parameter

   When possible, use return statements, t's two much faster. If a method returns just one value, return it—do not place it in a ref or out parameter. This will improve program performance.

9. Exceptions

   Exception handling is relatively slow even if no exceptions are thrown. Code motion, which is a subset of optimization that includes hoisting expressions outside of loops, yields a performance gain.

   (try=>loop=> throw Exception =>catch) 5x much faster than (loop=>try=>throw Expcetion=>catch)

10. Avoid Boxing Unboxing method

   For mission-critical applications, it’s always better to avoid boxing and unboxing. However, in .NET Core, we have many other types that internally use objects and perform boxing and unboxing. Most of the types under System.Collections and System.Collections.Specialized use objects and object arrays for internal storage, and when we store primitive types in these collections, they perform boxing and convert each primitive value to an object type, adding extra overhead and negatively impacting the performance of the application. Other types of System.Data, namely DateSet, DataTable, and DataRow, also use object arrays under the hood.

   Types under the System.Collections.Generic namespace or typed arrays are the best approaches to use when performance is the primary concern. For example, HashSet, LinkedList, and List are all types of generic collections.

## NB

> DateTime is a struct.
> String is a Class
> boxing => ValueType to object type, inboxing object type => valueType

![c# 7.2](https://i.imgur.com/yXN3PE1.png "c#7.2 optimizations")

### Tools

* [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet)

## References

* [Référence 1](https://stackify.com/net-application-optimization/)
