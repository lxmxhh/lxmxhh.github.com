---
layout: post
title: "轻松编写 C++ 单元测试"
date: 2013-08-28 22:35
comments: true
categories: C++ dev
---
作为一个TDD的脑残粉，凡事都想测试先行。现在回归C++开发，顿时觉得各种不方便。特别是手头没有好用的单元测试框架，开发效率简直无法忍受，写好的代码重构起来也畏手畏脚。
<!--more-->  
曾经尝试过CppUnit，作为JUnit的同胞，出自名门，但在C++开发中，尤其在VS环境下实在是太难用了（主要是由于 C++ 没有反射机制，而这是 JUnit 设计的基础）。偶然搜得googletest，一套由 google 发布的开源单元测试框架。

##应用 googletest 编写单元测试代码##
googletest 是由 Google 公司发布，且遵循 New BSD License （可用作商业用途）的开源项目，并且 googletest 可以支持绝大多数大家所熟知的平台。与 CppUnit 不同的是： googletest 可以自动记录下所有定义好的测试，不需要用户通过列举来指明哪些测试需要运行。  

###定义单元测试###
在应用 googletest 编写单元测试时，使用 `TEST()` 宏来声明测试函数。如：

清单 1. 用 `TEST()` 宏声明测试函数
<code>
	TEST(GlobalConfigurationTest, configurationDataTest) 
	TEST(GlobalConfigurationTest, noConfigureFileTest)
</code>
分别针对同一程序单元 `GlobalConfiguration` 声明了两个不同的测试函数，以分别对配置数据进行检查`configurationDataTest`，以及测试没有配置文件的特殊情况`noConfigureFileTest`。

###实现单元测试###
针对同一程序单元设计出不同的测试场景后（即划分出不同的 Test 后），开发者就可以编写单元测试分别实现这些测试场景了。

在 googletest 中实现单元测试，可通过 `ASSERT_*` 和 `EXPECT_*` 断言来对程序运行结果进行检查。 `ASSERT_*` 版本的断言失败时会产生致命失败，并结束当前函数； `EXPECT_*` 版本的断言失败时产生非致命失败，但不会中止当前函数。因此， `ASSERT_*` 常常被用于后续测试逻辑强制依赖的处理结果的断言，如创建对象后检查指针是否为空，若为空，则后续对象方法调用会失败；而 `EXPECT_*` 则用于即使失败也不会影响后续测试逻辑的处理结果的断言，如某个方法返回结果的多个属性的检查。

googletest 中定义了如下的断言：  
表 1： googletest 定义的断言（ Assert ）
<table class="table table-bordered table-striped table-condensed">
	<tr>
		<th>基本断言</th>
		<th>二进制比较</th>
		<th>字符串比较</th>
	</tr>
	<tr>
		<td>ASSERT_TRUE(condition); condition为真</td>
		<td>ASSERT_EQ(expected,actual); expected==actual</td>
		<td>ASSERT_STREQ(expected_str,actual_str); 两个 C 字符串有相同的内容</td>
	</tr>
	<tr>
		<td>EXPECT_TRUE(condition); condition为真</td>
		<td>EXPECT_EQ(expected,actual); expected==actual</td>
		<td>EXPECT_STREQ(expected_str,actual_str); 两个 C 字符串有相同的内容</td>
	</tr>
	<tr>
		<td>ASSERT_FALSE(condition); condition为假</td>
		<td>ASSERT_NE(val1,val2); val1!=val2</td>
		<td>ASSERT_STRNE(str1,str2); 两个 C 字符串有不同的内容</td>
	</tr>
	<tr>
		<td>EXPECT_FALSE(condition); condition为假</td>
		<td>EXPECT_NE(val1,val2); val1!=val2</td>
		<td>EXPECT_STRNE(str1,str2); 两个 C 字符串有不同的内容</td>
	</tr>
	<tr>
		<td></td>
		<td>ASSERT_LT(val1,val2); val1&ltval2</td>
		<td>ASSERT_STRCASEEQ(expected_str,actual_str); 两个 C 字符串有相同的内容，忽略大小写</td>
	</tr>
	<tr>
		<td></td>
		<td>EXPECT_LT(val1,val2); val1&ltval2</td>
		<td>EXPECT_STRCASEEQ(expected_str,actual_str); 两个 C 字符串有相同的内容，忽略大小写</td>
	</tr>
	<tr>
		<td></td>
		<td>ASSERT_LE(val1,val2); val1&lt=val2</td>
		<td>ASSERT_STRCASENE(expected_str,actual_str); 两个 C 字符串有不同的内容，忽略大小写</td>
	</tr>
	<tr>
		<td></td>
		<td>EXPECT_LE(val1,val2); val1&lt=val2</td>
		<td>EXPECT_STRCASENE(expected_str,actual_str); 两个 C 字符串有不同的内容，忽略大小写</td>
	</tr>
	<tr>
		<td></td>
		<td>ASSERT_GT(val1,val2); val1&gtval2</td>
		<td></td>
	</tr>
	<tr>
		<td></td>
		<td>EXPECT_GT(val1,val2); val1&gtval2</td>
		<td></td>
	</tr>
	<tr>
		<td></td>
		<td>ASSERT_GE(val1,val2); val1&gt=val2</td>
		<td></td>
	</tr>
	<tr>
		<td></td>
		<td>EXPECT_GE(val1,val2); val1&gt=val2</td>
		<td></td>
	</tr>
</table>
下面的实例演示了上面部分断言的使用：

清单 2. 一个较完整的 googletest 单元测试实例
<code>
	// Configure.h 
	 #pragma once 

	 #include <string> 
	 #include <vector> 

	 class Configure 
	 { 
	 private: 
	    std::vector<std::string> vItems; 

	 public: 
	    int addItem(std::string str); 

	    std::string getItem(int index); 

	    int getSize(); 
	 }; 

	 // Configure.cpp 
	 #include "Configure.h" 

	 #include <algorithm> 

	 /** 
	 * @brief Add an item to configuration store. Duplicate item will be ignored 
	 * @param str item to be stored 
	 * @return the index of added configuration item 
	 */ 
	 int Configure::addItem(std::string str) 
	 { 
		std::vector<std::string>::const_iterator vi=std::find(vItems.begin(), vItems.end(), str); 
	    if (vi != vItems.end()) 
	        return vi - vItems.begin(); 

	    vItems.push_back(str); 
	    return vItems.size() - 1; 
	 } 

	 /** 
	 * @brief Return the configure item at specified index. 
	 * If the index is out of range, "" will be returned 
	 * @param index the index of item 
	 * @return the item at specified index 
	 */ 
	 std::string Configure::getItem(int index) 
	 { 
	    if (index >= vItems.size()) 
	        return ""; 
	    else 
	        return vItems.at(index); 
	 } 

	 /// Retrieve the information about how many configuration items we have had 
	 int Configure::getSize() 
	 { 
	    return vItems.size(); 
	 } 

	 // ConfigureTest.cpp 
	 #include <gtest/gtest.h> 

	 #include "Configure.h" 

	 TEST(ConfigureTest, addItem) 
	 { 
	    // do some initialization 
	    Configure* pc = new Configure(); 
	    
	    // validate the pointer is not null 
	    ASSERT_TRUE(pc != NULL); 

	    // call the method we want to test 
	    pc->addItem("A"); 
	    pc->addItem("B"); 
	    pc->addItem("A"); 

	    // validate the result after operation 
	    EXPECT_EQ(pc->getSize(), 2); 
	    EXPECT_STREQ(pc->getItem(0).c_str(), "A"); 
	    EXPECT_STREQ(pc->getItem(1).c_str(), "B"); 
	    EXPECT_STREQ(pc->getItem(10).c_str(), ""); 

	    delete pc; 
	 }
</code>
###运行单元测试###
在实现完单元测试的测试逻辑后，可以通过 `RUN_ALL_TESTS()` 来运行它们，如果所有测试成功，该函数返回 0，否则会返回 1 。 `RUN_ALL_TESTS()` 会运行你链接到的所有测试――它们可以来自不同的测试案例，甚至是来自不同的文件。

因此，运行 googletest 编写的单元测试的一种比较简单可行的方法是：  
为每一个被测试的 class 分别创建一个测试文件，并在该文件中编写针对这一 class 的单元测试；  
编写一个 `Main.cpp` 文件，并在其中包含以下代码，以运行所有单元测试：

清单 3. 初始化 googletest 并运行所有测试
<code>
	#include <gtest/gtest.h> 

	 int main(int argc, char** argv) { 
	    testing::InitGoogleTest(&argc, argv); 

	    // Runs all tests using Google Test. 
	    return RUN_ALL_TESTS(); 
	 }
</code>
最后，将所有测试代码及 `Main.cpp` 编译并链接到目标程序中。  
此外，在运行可执行目标程序时，可以使用 `--gtest_filter` 来指定要执行的测试用例，如：  

*   `./foo_test` 没有指定 `filter` ，运行所有测试；
*   `./foo_test --gtest_filter=*` 指定 `filter` 为 `*` ，运行所有测试；
*   `./foo_test --gtest_filter=FooTest.*` 运行测试用例 `FooTest` 的所有测试；
*   `./foo_test --gtest_filter=*Null*:*Constructor*` 运行所有全名（即测试用例名 + “ . ” + 测试名，如 `GlobalConfigurationTest.noConfigureFileTest` ）含有"Null" 或"Constructor" 的测试；
*   `./foo_test --gtest_filter=FooTest.*-FooTest.Bar` 运行测试用例 `FooTest` 的所有测试，但不包括 `FooTest.Bar`。
这一特性在包含大量测试用例的项目中会十分有用。
***
关于 googletest 的更多信息，请访问其项目主页：[http://code.google.com/p/googletest/] [url1]

[url1]:http://code.google.com/p/googletest/ 

