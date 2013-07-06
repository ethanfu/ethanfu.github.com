---
layout: post
title: "JUnit 4 与 TestNG 对比（翻译）"
description: "compare two java unit test framework"
category: Java
tags: [test,java]
---
这两天在看一本书[《Java测试新技术TestNG和高级概念》](http://book.douban.com/subject/3261693/)，作者是 TestNG 的创始人，了解了不少关于 TestNG 的知识，看了一篇文章基本把这本书的几个观点都体现了，那我就直接翻译原文就好，省得自己总结。这两天要不是等原作者的授权的话可能周末本文就发布了，没经过人家许可翻译人家文章应该的确也不是什么光彩的事情，等等无妨。原文链接[JUnit 4 Vs TestNG – Comparison](http://www.mkyong.com/unittest/junit-4-vs-testng-comparison/)，非常感谢作者写出好文，不过[mkyong](http://www.mkyong.com/author/mkyong/)先生写了的确好多文章，经常搜个文章 google 第一篇总是他的。如果有翻译问题，请拍砖。

————————————————————————————————————————————

Junit 4 和 TestNG 都是 Java 方面非常流行的单元测试框架。在功能上两个框架都非常类似。到底哪个比较好？在Java项目中我们应该选择哪个框架？

下图将会对Junit 4 和 TestNG 做个功能特征的对比。

![junit-vs-testngjpg](http://www.mkyong.com/wp-content/uploads/2009/05/junit-vs-testngjpg.jpg)

## 注解支持

Junit 4 和 TestNG 在注解方面的实现非常相似。

<table border="1">
	<tbody>
		<tr>
			<td>特性</td>
			<td>JUnit 4</td>
			<td>TestNG</td>
		</tr>
		<tr>
			<td>测试注解</td>
			<td>@Test</td>
			<td>@Test</td>
		</tr>
		<tr>
			<td>测试套件在执行之前需要执行的</td>
			<td>–</td>
			<td>@BeforeSuite</td>
		</tr>
		<tr>
			<td>测试套件在执行之后需要执行的</td>
			<td>–</td>
			<td>@AfterSuite</td>
		</tr>
		<tr>
			<td>在测试之前需要执行的</td>
			<td>–</td>
			<td>@BeforeTest</td>
		</tr>
		<tr>
			<td>在测试之后需要执行的</td>
			<td>–</td>
			<td>@AfterTest</td>
		</tr>
		<tr>
			<td>在一个测试方法所属于的任意一个组的第一个方法被调用之前执行</td>
			<td>–</td>
			<td>@BeforeGroups</td>
		</tr>
		<tr>
			<td>在一个测试方法所属于的任意一个组的最后一个方法被调用之后执行</td>
			<td>–</td>
			<td>@AfterGroups</td>
		</tr>
		<tr>
			<td>在当前类的第一个测试方法调用之前执行</td>
			<td>@BeforeClass</td>
			<td>@BeforeClass</td>
		</tr>
		<tr>
			<td>在当前类的最后一个测试方法调用之后执行</td>
			<td>@AfterClass</td>
			<td>@AfterClass</td>
		</tr>
		<tr>
			<td>每个测试方法之前需要执行</td>
			<td>@Before</td>
			<td>@BeforeMethod</td>
		</tr>
		<tr>
			<td>每个测试方法之后需要执行</td>
			<td>@After</td>
			<td>@AfterMethod </td>
		</tr>
		<tr>
			<td>忽略</td>
			<td>@ignore</td>
			<td>@Test(enbale=false)</td>
		</tr>
		<tr>
			<td>预期异常</td>
			<td>@Test(expected = ArithmeticException.class)</td>
			<td>@Test(expectedExceptions = ArithmeticException.class)</td>
		</tr>
		<tr>
			<td>超时</td>
			<td>@Test(timeout = 1000)</td>
			<td>@Test(timeout = 1000)</td>
		</tr>
	</tbody>
</table>
JUnit 4 和 TestNG 之间注解方面的区别主要有以下几点：

1. 在Junit 4 中，如果我们需要在方法前面使用`@BeforeClass`和`@AfterClass`，那么该测试方法则必须是静态方法。TestNG 在方法定义部分则更加的灵活，它不需要类似的约束。
2. 3个附加的setUp/tearDown级别：套件和分组（@Before/AfterSuite, @Before/AfterTest, @Before/AfterGroup）。想了解详细的请看[这里](http://testng.org/doc/documentation-main.html#annotations)

JUnit 4

	@BeforeClass
    public static void oneTimeSetUp() {
        // one-time initialization code   
    	System.out.println("@BeforeClass - oneTimeSetUp");
    }
TestNG

	@BeforeClass
    public void oneTimeSetUp() {
        // one-time initialization code   
    	System.out.println("@BeforeClass - oneTimeSetUp");
	}

在Junit 
4中，注解的命名是比较令人困惑的，例如 `Before`, `After` and `Expected`，我们不是很确切的能理解在方法前面有`Before`和`After`这样的注解是做什么的，同样`Expected`也如此。TestNG在这方面做的就好很多，注解使用了`BeforeMethod`，`AfterMethod`和`ExpectedException`，这样的名字就非常好理解了。

## 异常测试
异常测试的意思是在单元测试中应该抛出什么异常是合理的，这个特性在两个框架都已经实现。

JUnit 4

	@Test(expected = ArithmeticException.class)  
	public void divisionWithException() {  
		int i = 1/0;
	}
TestNG
	
	@Test(expectedExceptions = ArithmeticException.class)  
	public void divisionWithException() {  
		int i = 1/0;
	}
## 忽略测试	
忽略测试意思是在单元测试哪些是可以被忽略的，这个特性在两个框架都已经实现。

JUnit 4

	@Ignore("Not Ready to Run")  
	@Test
	public void divisionWithException() {  
		System.out.println("Method is not ready yet");
	}
TestNG

	@Test(enabled=false)
	public void divisionWithException() {  
		System.out.println("Method is not ready yet");
	}
## 时间测试
时间测试意思是如果一个单元测试运行的时间超过了一个指定的毫秒数，那么测试将终止并且标记为失败的测试，这个特性在两个框架都已经实现。

JUnit 4

    @Test(timeout = 1000)  
	public void infinity() {  
		while (true);  
	}
TestNG

	@Test(timeOut = 1000)  
	public void infinity() {  
		while (true);  
	}
## 套件测试
套件测试就是把几个单元测试组合成一个模块，然后运行，这个特性两个框架均已实现。然而却是用了两个不同的方式来实现的。

JUnit 4

 `@RunWith` 和 `@Suite`注解被用于执行套件测试。下面的代码是所展示的是在`JunitTest5`被执行之后需要`JunitTest1` 和 `JunitTest2`也一起执行。所有的声明需要在类内部完成。

	@RunWith(Suite.class)
	@Suite.SuiteClasses({
	    JunitTest1.class,
	    JunitTest2.class
	})
	public class JunitTest5 {
	}
	
TestNG

执行套件测试是使用XML文件配置的方式来做。下面的 XML 的文件可以使得`TestNGTest1`和`TestNGTest2`一起执行。

	<!DOCTYPE suite SYSTEM "http://beust.com/testng/testng-1.0.dtd" >
	<suite name="My test suite">
	  <test name="testing">
	    <classes>
	       <class name="com.fsecure.demo.testng.TestNGTest1" />
	       <class name="com.fsecure.demo.testng.TestNGTest2" />
	    </classes>
	  </test>
	</suite>

TestNG可以在这块做的更好，使用了`组`的概念，每个方法都可以被分配到一个组里面，可以根据功能特性来分组。例如：

这是一个有4个方法，3个组(method1, method2 和 method4)的类


    @Test(groups="method1")
	public void testingMethod1() {  
	  System.out.println("Method - testingMethod1()");
	}  
 
	@Test(groups="method2")
	public void testingMethod2() {  
		System.out.println("Method - testingMethod2()");
	}  
 
	@Test(groups="method1")
	public void testingMethod1_1() {  
		System.out.println("Method - testingMethod1_1()");
	}  
 
	@Test(groups="method4")
	public void testingMethod4() {  
		System.out.println("Method - testingMethod4()");
	}
下面XML文件定义了一个只是执行`methed1`的组的单元测试

	<!DOCTYPE suite SYSTEM "http://beust.com/testng/testng-1.0.dtd" >
	<suite name="My test suite">
	  <test name="testing">
	  	<groups>
	      <run>
	        <include name="method1"/>
	      </run>
	    </groups>
	    <classes>
	       <class name="com.fsecure.demo.testng.TestNGTest5_2_0" />
	    </classes>
	  </test>
	</suite>
使用分组的概念，集成测试就会更加强大。例如，我们可以只是执行所有测试中的组名为`DatabaseFuntion`的测试。

## 参数化测试
参数化测试意思是给单元测试传多个参数值。这个特性在JUnit 4 和TestNG。然后两个框架实现的方式却完全不同。

JUnit 4

`@RunWith` 和 `@Parameter` 注解用于为单元测试提供参数值，`@Parameters`必须返回 List[]，参数将会被作为参数传给类的构造函数。

	@RunWith(value = Parameterized.class)
	public class JunitTest6 {
	 
		 private int number;
	 
		 public JunitTest6(int number) {
		    this.number = number;
		 }
	 
		 @Parameters
		 public static Collection<Object[]> data() {
		   Object[][] data = new Object[][] { { 1 }, { 2 }, { 3 }, { 4 } };
		   return Arrays.asList(data);
		 }
	 
		 @Test
		 public void pushTest() {
		   System.out.println("Parameterized Number is : " + number);
		 }
	}
它在使用上有许多的限制；我们必须遵循 JUnit 的方式去声明参数，参数必须通过构造函数的参数去初始化类的成员来用于测试。返回的参数类型必须是`List []`，数据已经被限定为String或者是一个原始值。

TestNG

使用XML文件或者`@DataProvider`注解来给测试提供参数。

XML文件配置参数化测试

只是在方法上声明`@Parameters`注解，参数的数据将由 TestNG 的 XML 配置文件提供。这样做之后，我们可以使用不同的数据集甚至是不同的结果集来重用一个测试用例。另外，甚至是最终用户，QA 或者 QE 可以提供使用 XML 文件来提供他们自己的数据来做测试。

Unit Test

      public class TestNGTest6_1_0 {
 
	   @Test
	   @Parameters(value="number")
	   public void parameterIntTest(int number) {
	      System.out.println("Parameterized Number is : " + number);
	   }
 
      }
XML 文件

	<!DOCTYPE suite SYSTEM "http://beust.com/testng/testng-1.0.dtd" >
	<suite name="My test suite">
	  <test name="testing">
	 
	    <parameter name="number" value="2"/> 	
	 
	    <classes>
	       <class name="com.fsecure.demo.testng.TestNGTest6_0" />
	    </classes>
	  </test>
	</suite>
	
`@DataProvider` 注解做参数化测试

使用XML文件初始化数据可以很方便，但是测试偶尔需要复杂的类型，一个String或原始值并不能完全满足。 TestNG 的@ DataProvider的注解，可以更好的把复杂的参数类型映射到一个测试方法来处理这种情况。


`@DataProvider` 可以使用 Vector, String 或者 Integer 类型的值作为参数

	@Test(dataProvider = "Data-Provider-Function")
	public void parameterIntTest(Class clzz, String[] number) {
	   System.out.println("Parameterized Number is : " + number[0]);
	   System.out.println("Parameterized Number is : " + number[1]);
	}
 
	//This function will provide the patameter data
	@DataProvider(name = "Data-Provider-Function")
	public Object[][] parameterIntTestProvider() {
		return new Object[][]{
		   {Vector.class, new String[] {"java.util.AbstractList", "java.util.AbstractCollection"}},
		   {String.class, new String[] {"1", "2"}},
		   {Integer.class, new String[] {"1", "2"}}
		};
	}
	
`@DataProvider` 作为对象的参数

P.S “TestNGTest6_3_0” 是一个简单的对象，使用了get和set方法。

	@Test(dataProvider = "Data-Provider-Function")
	public void parameterIntTest(TestNGTest6_3_0 clzz) {
	   System.out.println("Parameterized Number is : " + clzz.getMsg());
	   System.out.println("Parameterized Number is : " + clzz.getNumber());
	}
 
	//This function will provide the patameter data
	@DataProvider(name = "Data-Provider-Function")
	public Object[][] parameterIntTestProvider() {
 
		TestNGTest6_3_0 obj = new TestNGTest6_3_0();
		obj.setMsg("Hello");
		obj.setNumber(123);
 
		return new Object[][]{
	    	{obj}
		};
	}
TestNG的参数化测试使用起来非常的友好和灵活 (不管是XML配置还是在类里面注解的方式). 它可以使用许多复杂的数据类型作为参数的值，并且没有什么限制。如上面的例子所示， we even can pass in our own object (TestNGTest6_3_0) for parameterized test

## 依赖测试

参数化测试意味着测试的方法是有依赖的，也就是要执行的的方法在执行之前需要执行的部分。如果依赖的方法出现错误，所有的子测试都会被忽略，不会被标记为失败。

JUnit 4

JUnit 框架主要聚焦于测试的隔离，暂时还不支持这个特性。

TestNG

它使用`dependOnMethods`来实现了依赖测试的功能，如下：

    @Test
	public void method1() {
	   System.out.println("This is method 1");
	}
 
	@Test(dependsOnMethods={"method1"})
	public void method2() {
		System.out.println("This is method 2");
	}
	
如果`method1()`成功执行，那么`method2()`也将被执行，否则`method2()`将会被忽略。


## 讨论总结

当我们做完所有特性的对比以后，我建议使用 TestNG 作为 Java 项目的主要单元测试框架，因为 TestNG 在参数化测试、依赖测试以及套件测试（组）方面功能更加强大。TestNG 意味着高级的测试和复杂的集成测试。它更加的灵活，特别是对大的套件测试。另外，TestNG 也涵盖了 JUnit4 的全部功能。那就没有任何理由使用 Junit了。

## 参考资料

TestNG

————

[http://en.wikipedia.org/wiki/TestNG](http://en.wikipedia.org/wiki/TestNG)

[http://www.ibm.com/developerworks/java/library/j-testng/](http://www.ibm.com/developerworks/java/library/j-testng/)

[http://testng.org/doc/index.html](http://testng.org/doc/index.html)

[http://beust.com/weblog/](http://beust.com/weblog/)

JUnit

———–

[http://en.wikipedia.org/wiki/JUnit](http://en.wikipedia.org/wiki/JUnit)

[http://www.ibm.com/developerworks/java/library/j-junit4.html](http://www.ibm.com/developerworks/java/library/j-junit4.html)

[http://junit.sourceforge.net/doc/faq/faq.htm](http://junit.sourceforge.net/doc/faq/faq.htm)

[http://www.devx.com/Java/Article/31983/0/page/3](http://www.devx.com/Java/Article/31983/0/page/3)

[http://ourcraft.wordpress.com/2008/08/27/writing-a-parameterized-junit-test/](http://ourcraft.wordpress.com/2008/08/27/writing-a-parameterized-junit-test/)


TestNG VS JUnit

——————

[http://docs.codehaus.org/display/XPR/Migration+to+JUnit4+or+TestNG](http://docs.codehaus.org/display/XPR/Migration+to+JUnit4+or+TestNG)

[http://www.ibm.com/developerworks/java/library/j-cq08296/index.html](http://www.ibm.com/developerworks/java/library/j-cq08296/index.html)

[http://www.cavdar.net/2008/07/21/junit-4-in-60-seconds/](http://www.cavdar.net/2008/07/21/junit-4-in-60-seconds/)


{% include JB/setup %}
