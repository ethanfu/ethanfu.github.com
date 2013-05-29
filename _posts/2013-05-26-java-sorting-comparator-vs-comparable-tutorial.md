---
layout: post
title: "Java排序: Comparator vs Comparable 入门"
description: ""
category: tech 
tags: [java]
---
{% include JB/setup %}

今天有朋友问了下关于Comparator和Comparable的问题，很多人都曾经有过对这个问题的疑惑，今天把这个说明下。本篇文章译自[Java Sorting: Comparator vs Comparable Tutorial](http://www.digizol.com/2008/07/java-sorting-comparator-vs-comparable.html)。接下来是正文：

Java Comparators and Comparables?他们是什么？我们为什么要用他们？有一些读者也反应过这个问题。那么我们这篇文章将会详细讨论java.util.Comparator和java.lang.Comparable，并且会附上部分代码做更详细的阐述。

## 前提

* Java基础知识

## 系统要求

* 已经安装JDK

## What are Java Comparators and Comparables?

基本可以说是见名知义吧，它们在Java中用于做对象的比较。Java objects使用它们可以根据一个预先定义好的顺序去完成排序。

接下来我们会解释其中的两个概念。

## Comparable

一个comparable对象用于自己和另外的对象做对比。它本身必须实现java.lang.Comparable的接口，为了能够对比其他的示例。

## Comparator

一个comparator对象能够对比不同的对象。它不能用于同一个类的不同示例的对比，但是可以用于其他的类的示例做对比。它必须实现java.util.Comparator的接口。

## 我们需要对比对象么？

最简单的答案就是需要对比。当有一个对象的列表的时候，在有些情况下你必须把这些对象按照不同的排序规则来排序。举例来说：考虑一个web页面需要显示职工的列表。通常情况下职工列表是按照职工的ID来排序。同样也可以根据姓名或者年龄来排序。在这些情况下这两个概念就非常便于使用了。

## 如何使用

在Java中有两个接口来实现Comparable和Comparator，每一个都有一个用户必须实现的接口。分别是：

**java.lang.Comparable: int compareTo(Object o1)**

这个方法用于当前对象与o1对象做对比，返回int值，分别的意思是：

* positive – 当前对象大于o1
* zero – 当前对象等于o1
* negative – 当前对象小于o1

**java.util.Comparator: int compare(Object o1, Objecto2)**

这个方法用于o1与o2对象做对比，返回int值，分别的意思是：

* positive – o1大于o2
* zero – o1等于o2
* negative – o1小于o2

`java.util.Collections.sort(List)` 和 `java.util.Arrays.sort(Object[])` 方法被用来排列使用内在排序（natural ordering）方式的对象。(译者注：可参见[java.util.List#sort()](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#sort(java.util.List\)) )

`java.util.Collections.sort(List, Comparator)` 和 `java.util.Arrays.sort(Object[], Comparator)`方法在Comparator如果可供比较的时候会被用到。 

Employee的例子可以非常好的来解释这两个概念。首先我们写一个简单的Java bean来代表Employee。

	public class Employee {
		private int empId;
		private String name;
		private int age;
		
		public Employee(int empId, String name, int age) {
			// set values on attributes
		}
		// getters & setters
	}
	
下面我们将创建一个Employees列表，用来处理不同的排序需求。在下面的代码中Employees比较随意的被add到一个List中。

	import java.util.*;

	public class Util {
	
		public static List<Employee> getEmployees() {
		
			List<Employee> col = new ArrayList<Employee>();
			
			col.add(new Employee(5, "Frank", 28));
			col.add(new Employee(1, "Jorge", 19));
			col.add(new Employee(6, "Bill", 34));
			col.add(new Employee(3, "Michel", 10));
			col.add(new Employee(7, "Simpson", 8));
			col.add(new Employee(4, "Clerk",16 ));
			col.add(new Employee(8, "Lee", 40));
			col.add(new Employee(2, "Mark", 30));
			
			return col;
		}
	}
## 内在顺序（natural ordering）排序
职员的内在排序将根据Employee的id来排序。所以上面的Employee的类必须追加可以对比的能力，如下面代码所示：

	public class Employee implements Comparable<Employee> {
		private int empId;
		private String name;
		private int age;
		
		/**
		* Compare a given Employee with this object.
		* If employee id of this object is 
		* greater than the received object,
		* then this object is greater than the other.
		*/
		public int compareTo(Employee o) {
			return this.empId - o.empId ;
		}
		….
	}	
添加的compareTo()的方法为这个类的实例替换了默认的内在排序。所以如果是一个Employee对象的集合使用Collections.sort(List)来排序；排序会根据compareTo()方法里面定义的规则来完成。

我们接下来再写一个类来测试这个内在排序的机制。这个类使用Collections.sort(List)方法来对给定的List按照内在顺序来完成排序。

	import java.util.*;

	public class TestEmployeeSort {
	
		public static void main(String[] args) {     
			List coll = Util.getEmployees();
			Collections.sort(coll); // sort method
			printList(coll);
		}
	
		private static void printList(List<Employee> list) {
			System.out.println("EmpId\tName\tAge");
			for (Employee e: list) {
				System.out.println(e.getEmpId() + "\t" + e.getName() + "\t" + e.getAge());
			}
		}
	}
执行下上面的代码，检查下输出结果。如下面所示：

	EmpId Name Age
	1 Jorge 19
	2 Mark 30
	3 Michel 10
	4 Clerk 16
	5 Frank 28
	6 Bill 34
	7 Simp 8
	8 Lee 40
我们通过输出可以看到的是，这个列表是根据employee的id来完成的排序。因为employee的id是个int值，所以这些employee的实例就按照1-8完成了排序。
## 根据其他字段排序

如果你需要根据employee的其他字段来进行排序，我们需要修改Employee的类的compareTo()方法。但是我们也会因此而失去基于Employee的ID的排序机制。如果我们在不同的场合需要有不同的字段来进行排序的话，这样使用可能不是一个好的选择。但是不要担心；Comparator可以帮助我们。

接下来我们写一个类实现`java.util.Comparator`的接口，你可以对Employees使用你希望使用的任何字段进行排序，而不需要对Employee的类本身进行任何改动。Employee的类不需要实现`java.lang.Comparable`或者`java.util.Comparator`接口。

### 根据name字段排序
下面的EmpSortByName是根据name的字段来对Employee的实例做排序的类。在这个类中，compare()方法实现了这个排序机制。在compare()方法中，我们可以得到两个Employee的实例，并通过返回值我们可以知道哪个对象比较大。

	public class EmpSortByName implements Comparator<Employee>{

		public int compare(Employee o1, Employee o2) {
			return o1.getName().compareTo(o2.getName());
		}
	}

**注意**这里的使用的是String类的compareTo()方法比较name字段（字段也是字符串）。

我们现在可以测试下这个排序的机制，你必须使用`Collections.sort(List, Comparator)`这个方法，而不是使用`Collections.sort(List)`方法。现在按照下面的代码来改造下TestEmployeeSort这个类，我们可以看下EmpSortByName comparator的方式的内部的排序方法是怎样的。

	import java.util.*;

	public class TestEmployeeSort {
	
		public static void main(String[] args) {
		
			List coll = Util.getEmployees();
				//Collections.sort(coll);
				//use Comparator implementation
				Collections.sort(coll, new EmpSortByName());
				printList(coll);
			}
			
			private static void printList(List<Employee> list) {
				System.out.println("EmpId\tName\tAge");
				for (Employee e: list) {
					System.out.println(e.getEmpId() + "\t" + e.getName() + "\t" + e.getAge());
				}
			}
		｝
	}
	
结果如下面所示：

	EmpId Name Age
	6 Bill 34
	4 Clerk 16
	5 Frank 28
	1 Jorge 19
	8 Lee 40
	2 Mark 30
	3 Michel 10
	7 Simp 8
	
检查Employees的排序是否已经按照name的字段进行了正确的排序。你看到输出的结果应该是按照字母排好序的。

### Sorting by empId field

甚至是先前我们使用Comparable实现的按照Employee ID实现的排序功能一样可以使用Comparator来实现，如下面代码所示：

	public class EmpSortByEmpId implements Comparator<Employee>{

		public int compare(Employee o1, Employee o2) {
			return o1.getEmpId() - o2.getEmpId();
		}
	}
	
## 最后

文章基本就写到这里了，但是想完全理解这些知识的话，仅仅写的这些还完全不够。接下来还有两个问题需要你自己来掌握：

* 职员同时使用name，age，id来进行排序（例如：当name相等的时候，接下来对比age，如果相等继续尝试id）
* 深入了解下`equals()`方法和`compare()/compareTo()`方法为什么必须保持一致？怎样保持一致？

如果你有对这些概念有什么问题，请在下面进行评论，我也会尽快回复您的。