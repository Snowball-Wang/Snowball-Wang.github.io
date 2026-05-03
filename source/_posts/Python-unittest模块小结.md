---
title: Python unittest模块小结
date: 2019-07-16 19:18:00
tags:
- 技术日志
- Python
categories:
- 技术日志
---

**关键字：** 单元测试 Python unittest模块

## 0. 前言

最近在看关于[VPP](https://wiki.fd.io/view/VPP)项目中的CSIT的部分，VPP的Path test里主要就是应用了Python的unittest框架，正好在毕业论文项目中也使用到了unittest完成相应单元测试，[这篇博客](https://pymotw.com/3/unittest/index.html)对unittest框架的使用作了全面的介绍，故翻译之作为学习的存档，也算是一个小小的总结。

Python的`unittest`框架是基于Kent Beck（TDD测试驱动开发发明人）和Erich Gamma的XUnit框架开发完成的，这种模式在C、Perl、Java和Smalltalk等语言中也被重复利用。`unittest`框架中有三个基本概念，分别是**test fixtures**，**test suites**以及**test runner**，这三个概念的组合使用可以实现Python的自动化测试。

<!-- more -->

## 1. 基本测试结构

`unittest`单元测试主要包含两个部分：用来管理测试依赖环境的代码（这个就是test fixtures），以及测试代码本身。单独的测试可通过继承`TestCase`类来覆盖或者增添相应的测试函数。在下面的示例代码中，`SimplisticTest`类有仅有一个`test()`测试函数，如果变量`a`与变量`b`不相同，则会导致测试失败。

```python
# unittest_simple.py

import unittest

class SimplisticTest(unittest.TestCase):
    
    def test(self):
        a = 'a'
        b = 'a'
        self.assertEqual(a, b)
```

## 2. 运行测试

运行`unittest`单元测试最简单的方式可通过命令行来实现。

```
$ python3 -m unittest unittest_simple.py
.
----------------------------------------------------------------
Ran 1 test in 0.004s

OK
```

命令行输出对应测试用例所消耗的时间，以及每个测试用例的测试结果。输出第一行的`.`表明该测试用例已通过。我们可以使用`-v`指令来查看更加详细版本的测试结果。

```
$ python3 -m unittest -v unittest_simple.py
test (unittest_simple.SimplisticTest) ... ok

----------------------------------------------------------------
Ran 1 test in 0.001s

OK
```

## 3. 测试结果

`unittest`测试结果一共有三种情况，如下表所示。

| 测试结果 |                相关描述                |
| :------: | :------------------------------------: |
|    OK    |                测试通过                |
|   FAIL   | 测试没有通过，触发`AssertionError`异常 |
|  ERROR   |  测试触发了除`AssertionError`外的异常  |

`unittest`中并没有明确的方式来让测试通过，因此一个测试最终的状态取决于是否有异常被触发。

```python
# unittest_outcomes.py

import unittest


class OutcomesTest(unittest.TestCase):

    def testPass(self):
        return

    def testFail(self):
        self.assertFalse(True)

    def testError(self):
        raise RuntimeError('Test error!')
```

当测试失败或者产生一个错误时，测试的追踪结果包含在最终的命令行输出中。

```
$ python3 -m unittest unittest_outcomes.py
EF.
================================================================
ERROR: testError (unittest_outcomes.OutcomesTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_outcomes.py", line 18, in testError
    raise RuntimeError('Test error!')
RuntimeError: Test error!

================================================================
FAIL: testFail (unittest_outcomes.OutcomesTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_outcomes.py", line 15, in testFail
    self.assertFalse(True)
AssertionError: True is not false

----------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1, errors=1)
```

在上述的示例中，`testFail()`函数运行失败，命令行输出的追踪结果表明了错误代码的行数。

```python
# unittest_failwithmessage.py

import unittest

class FailureMessageTest(unittest.TestCase):
    
    def testFail(self):
        self.assertFalse(True, 'failure message goes here')
```

为了更容易地理解测试代码失败的原因，函数`fail*()`和`assert*()`都可接受`msg`参数，用来显示详细的错误信息。

```
$ python3 -m unittest -v unittest_failwithmessage.py
testFail (unittest_failwithmessage.FailureMessageTest) ... FAIL

================================================================
FAIL: testFail (unittest_failwithmessage.FailureMessageTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_failwithmessage.py", line 12, in testFail
    self.assertFalse(True, 'failure message goes here')
AssertionError: True is not false : failure message goes here

----------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
```

## 4. 测试是否为真

绝大多数的测试都是在判断一些条件是否为真。`unittest`提供了两种不同的编写真值检查测试代码的方法，这两种方法的选择取决于是从测试代码作者的视角还是从待测试代码预期结果的视角。

```python
# unittest_truth.py
import unittest


class TruthTest(unittest.TestCase):

    def testAssertTrue(self):
        self.assertTrue(True)

    def testAssertFalse(self):
        self.assertFalse(False)
```

如果代码得到的值为真（True），则使用`assertTrue()`方法来验证。如果代码得到的值为假（False），则使用`assertFalse()`方法来验证。

```
$ python3 -m unittest -v unittest_truth.py

testAssertFalse (unittest_truth.TruthTest) ... ok
testAssertTrue (unittest_truth.TruthTest) ... ok

----------------------------------------------------------------
Ran 2 tests in 0.002s

OK
```

## 5. 测试是否相等

作为一个特殊用例，`unittest`包含测试两值是否相等的方法。

```python
# unittest_equality.py

import unittest

class EqualityTest(unittest.TestCase):
    
    def testExpectEqual(self):
        self.assertEqual(1, 3 - 2)
    
    def testExpectEqualFails(self):
        self.assertEqual(2, 3 - 2)
        
    def testExpectNotEqual(self):
        self.assertNotEqual(2, 3 - 2)
        
    def testExpectNotEqualFails(self):
        self.assertNotEqual(1, 3 - 2)
```

当以上测试代码失败时，以上的测试代码将会打印包含被比较值的错误信息。

```
$ python3 -m unittest -v unittest_equality.py

testExpectEqual (unittest_equality.EqualityTest) ... ok
testExpectEqualFails (unittest_equality.EqualityTest) ... FAIL
testExpectNotEqual (unittest_equality.EqualityTest) ... ok
testExpectNotEqualFails (unittest_equality.EqualityTest) ...
FAIL

================================================================
FAIL: testExpectEqualFails (unittest_equality.EqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality.py", line 15, in
testExpectEqualFails
    self.assertEqual(2, 3 - 2)
AssertionError: 2 != 1

================================================================
FAIL: testExpectNotEqualFails (unittest_equality.EqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality.py", line 21, in
testExpectNotEqualFails
    self.assertNotEqual(1, 3 - 2)
AssertionError: 1 == 1

----------------------------------------------------------------
Ran 4 tests in 0.006s

FAILED (failures=2)
```

## 6. 测试是否几乎相等

除了严格的相等性，我们可通过`assertAlmostEqual()`函数和`assertNotAlmostEqual()`函数来测试两个浮点数是否大致相等。

```python
# unittest_almostequal.py

class AlmostEqualTest(unittest.TestCase):
    
    def testEqual(self):
        self.assertEqual(1.1, 3.3 - 2.2)
    
    def testAlmostEqual(self):
        self.assertAlmostEqual(1.1, 3.3 - 2.2, places=1)
    
    def testNotAlmostEqual(self):
        self.assertNotAlmostEqual(1.1, 3.3 - 2.0, places=1)
```

`assertAlmostEqual()`函数的运行流程是先将两个参数进行比较，比较后的绝对值利用`round()`函数保存`places`参数个小数点后的位置，再查看得到的值是否为0。我的理解`assertAlmostEqual(a, b, places=c)`函数可以等价于`assertEqual(round(abs(b-a), c), 0)`。

上述代码的测试结果为：

```
$ python3 -m unittest unittest_almostequal.py

.F.
================================================================
FAIL: testEqual (unittest_almostequal.AlmostEqualTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_almostequal.py", line 12, in testEqual
    self.assertEqual(1.1, 3.3 - 2.2)
AssertionError: 1.1 != 1.0999999999999996

----------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1)
```

## 7. 测试容器类型数据结构

除了常规通用的`assertEqual()`和`assertNotEqual()`方法，`unittest`也提供了针对诸如`list`、`dict`、`set`等容器数据类型的测试方法。

```python
# unittest_equality_container.py

import testwrap
import unittest

class ContainerEqualityTest(unittest.TestCase):
    
    def testCount(self):
        self.assertCountEqual(
            [1, 2, 3, 2],
            [1, 3, 2, 3],
        )
    
    def testDict(self):
        self.assertDictEqual(
            {'a':1, 'b':2},
            {'a':1, 'b':3},
        )
    
    def testList(self):
        self.assertListEqual(
            [1, 2, 3],
            [1, 3, 2],
        )
        
    def testMultiLineString(self):
        self.assertMultiLineEqual(
            textwrap.dedent("""
            This string
            has more than one
            line.
            """),
            textwrap.dedent("""
            This string has
            more than two
            lines.
            """),
        )
        
    def testSequence(self):
        self.assertSequenceEqual(
            [1, 2, 3],
            [1, 3, 2],
        )
        
    def testSet(self):
        self.assertSetEqual(
            set([1, 2, 3]),
            set([1, 2, 3, 4]),
        )
        
    def testTuple(self):
        self.assertTupleEqual(
            (1, 'a'),
            (1, 'b'),
        ) 
```

测试结果如下：

```
$ python3 -m unittest unittest_equality_container.py
FFFFFFF
================================================================
FAIL: testCount
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 15, in
testCount
    [1, 3, 2, 3],
AssertionError: Element counts were not equal:
First has 2, Second has 1:  2
First has 1, Second has 2:  3

================================================================
FAIL: testDict
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 21, in
testDict
    {'a': 1, 'b': 3},
AssertionError: {'a': 1, 'b': 2} != {'a': 1, 'b': 3}
- {'a': 1, 'b': 2}
?               ^

+ {'a': 1, 'b': 3}
?               ^


================================================================
FAIL: testList
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 27, in
testList
    [1, 3, 2],
AssertionError: Lists differ: [1, 2, 3] != [1, 3, 2]

First differing element 1:
2
3

- [1, 2, 3]
+ [1, 3, 2]

================================================================
FAIL: testMultiLineString
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 41, in
testMultiLineString
    """),
AssertionError: '\nThis string\nhas more than one\nline.\n' !=
'\nThis string has\nmore than two\nlines.\n'

- This string
+ This string has
?            ++++
- has more than one
? ----           --
+ more than two
?           ++
- line.
+ lines.
?     +


================================================================
FAIL: testSequence
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 47, in
testSequence
    [1, 3, 2],
AssertionError: Sequences differ: [1, 2, 3] != [1, 3, 2]

First differing element 1:
2
3

- [1, 2, 3]
+ [1, 3, 2]

================================================================
FAIL: testSet
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 53, in testSet
    set([1, 3, 2, 4]),
AssertionError: Items in the second set but not the first:
4

================================================================
FAIL: testTuple
(unittest_equality_container.ContainerEqualityTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_equality_container.py", line 59, in
testTuple
    (1, 'b'),
AssertionError: Tuples differ: (1, 'a') != (1, 'b')

First differing element 1:
'a'
'b'

- (1, 'a')
?      ^

+ (1, 'b')
?      ^


----------------------------------------------------------------
Ran 7 tests in 0.013s

FAILED (failures=7)
```

`assertIn()`函数可用来检测元素是否在容器内。

```python
# unittest_in.py

import unittest

class ContainerMembershipTest(unittest.TestCase):
    def testDict(self):
        self.assertIn(4, {1:'a', 2:'b', 3:'c'})
        
    def testList(self):
        self.assertIn(4, [1, 2, 3])
        
    def testSet(self):
        self.assertIn(4, set([1, 2, 3]))
```

任何支持`in`操作符或者容器API的对象都可以使用`assertIn()`方法。

```
$ python3 -m unittest unittest_in.py
FFF
================================================================
FAIL: testDict (unittest_in.ContainerMembershipTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_in.py", line 12, in testDict
    self.assertIn(4, {1: 'a', 2: 'b', 3: 'c'})
AssertionError: 4 not found in {1: 'a', 2: 'b', 3: 'c'}

================================================================
FAIL: testList (unittest_in.ContainerMembershipTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_in.py", line 15, in testList
    self.assertIn(4, [1, 2, 3])
AssertionError: 4 not found in [1, 2, 3]

================================================================
FAIL: testSet (unittest_in.ContainerMembershipTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_in.py", line 18, in testSet
    self.assertIn(4, set([1, 2, 3]))
AssertionError: 4 not found in {1, 2, 3}

----------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=3)
```

## 8. 测试异常

之前我们谈过，如果测试触发了不是`AssertionError`的异常，那么这个异常会被当做错误处理。当我们修改已经测试覆盖的代码出现错误时，上述的内容就能够很快帮助我们定位修改产生错误的代码位置。不过在有些场景下，我们需要测试去验证代码确实会产生指定的异常。举个例子，如果一个无效的值赋值给一个对象的属性，那么`assertRaises()`函数相比于将异常放在测试代码里处理，则更加清晰。以下面两个测试代码为例做进一步说明：

```python
# unittest_exception.py

import unittest

def raise_error(*args, **kwds):
    raise ValueError('Invalid value:' + str(args) + str(kwds))
    
class ExceptionTest(unittest.TestCase):
    
    def testTrapLocally(self):
        try:
            raise_error('a', b='c')
        except ValueError:
            pass
        else:
            self.fail('Did not see ValueError')
    
    def testAssertRaises(self):
        self.assertRaises(ValueError, raises_error, 'a', b='c')
```

两种测试方法的结果相同，但是使用`assertRaises()`方法显得更为简洁，测试结果如下所示。

```
$ python3 -m unittest -v unittest_exception.py
testAssertRaises (unittest_exception.ExceptionTest) ... ok
testTrapLocally (unittest_exception.ExceptionTest) ... ok

----------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

## 9. 测试夹具（Test Fixtures）

夹具是一个测试所需要的外部资源，其目的是搭建一个测试用例的环境以确保测试用例能够按照预期进行。举个例子，对一个类的测试可能需要另一个提供相应配置设置和共享资源的类的实例。还有些情况下的测试夹具则包括数据库连接和相应的临时文件。

`unittest`通过设置相应的钩子（hook）来实现测试所需的配置和清理测试夹具的功能。如果想要对每个单独的测试用例创建测试夹具，则需重载`TestCase`类中的`setUp()`函数，想要清理测试夹具，则重载`tearDown()`函数。如果想要管理一个测试类中所有用例的测试夹具，则需重载`TestCase`类中的`setUpClass()`和`setUpClass()`方法。对于模块来说，则可使用`setUpModule()`和`tearDownModule()`函数。

```python
# unittest_fixtures.py

import random
import unittest

def setUpModule():
    print('In setUpModule()')
    
def tearDownModule():
    print('In tearDownModule()')
    
class FixturesTest(unittest.TestCase):
    
    @classmethod
    def setUpClass(self):
        print('In setUpClass()')
        cls.good_range = range(1, 10)
        
    @classmethod
    def tearDownClass(self):
        print('In tearDownClass()')
        del cls.good_range
        
    def setUp(self):
        super().setUp()
        print('\nIn setUp()')
        # Pick a number sure to be in the range. The range is
        # defined as not including the "stop" value, so make
        # sure it is not included in the set of allowed values
        # for our choice.
        self.value = random.randint(
            self.good_range.start,
            self.good_range.stop - 1,
        )
        
    def tearDown(self):
        print('In tearDown()')
        del self.value
        super().tearDown()
        
    def test1(self):
        print('In test1()')
        self.assertIn(self.value, self.good_range)
        
    def test2(self):
        print('In test2()')
        self.assertIn(self.value, self.good_range)
```

测试运行的结果如下所示：

```
$ python3 -u -m unittest -v unittest_fixtures.py
In setUpModule()
In setUpClass()
test1 (unittest_fixtures.FixturesTest) ...
In setUp()
In test1()
In tearDown()
ok
test2 (unittest_fixtures.FixturesTest) ...
In setUp()
In test2()
In tearDown()
ok
In tearDownClass()
In tearDownModule()

----------------------------------------------------------------
Ran 2 tests in 0.002s

OK
```

可以看到不同的函数的执行位置与相关测试函数的测试流程。有时候在清理测试夹具时会出错，由此导致`teardown`方法不被触发。为了确保测试夹具能够被正确释放，，可使用`addCleanup()`方法。

```python
# unittest_addcleanup.py

import random
import shutil
import tempfile
import unittest


def remove_tmpdir(dirname):
    print('In remove_tmpdir()')
    shutil.rmtree(dirname)
    
class FixturesTest(unittest.TestCase):
    
    def setUp(self):
        super().setUp()
        self.tmpdir = tempfile.mkdtemp()
        self.addCleanup(remove_tmpdir, self.tmpdir)
    
    def test1(self):
        print('\nIn test1()')
        
    def test2(self):
        print('\nIn test2()')
```

上述的测试用例创建了一个临时目录，并当测试结束后使用`shutil`模块删除临时目录。

```
$ python3 -u -m unittest -v unittest_addcleanup.py

test1 (unittest_addcleanup.FixturesTest) ...
In test1()
In remove_tmpdir()
ok
test2 (unittest_addcleanup.FixturesTest) ...
In test2()
In remove_tmpdir()
ok

----------------------------------------------------------------
Ran 2 tests in 0.002s

OK
```

## 10. 针对不同输入重复测试

针对不同输入使用相同的测试逻辑的做法在测试流程中很常用。不同于针对每个小的测试用例都定义一个单独的测试方法，更常规的做法是使用一个包含多个相关断言的测试方法，但是这样会导致只要有一个断言失败，剩下的测试就是被跳过。更好的解决方案是在测试方法内部使用`subTest()`来创建测试函数内部上下文，这样如果一个测试失败，终端将会显示相应的失败内容，并且不影响剩余测试的执行。

```python
# unittest_subtest.py

import unittest

class SubTest(unittest.TestCase):
    
    def test_combined(self):
        self.assertRegex('abc', 'a')
        self.assertRegex('abc', 'B')
        # The next assertions are not verified!
        self.assertRegex('abc', 'c')
        self.assertRegex('abc', 'd')
        
    def test_with_subtest(self):
        for pat in ['a', 'B', 'c', 'd']:
            with self.subTest(pattern=pat):
                self.assertRegex('abc', pat)
```

在上述的代码中，在`test_combined()`函数中永远都执行不到`'c'`和`'d'`的断言，但是`test_with_subtest()`方法是可以在`'B'`测试失败后继续执行剩下的断言。需要注意的是，上述代码执行的结果虽然报了三处失败，但依旧只是两个测试用例。

```
$ python3 -m unittest -v unittest_subtest.py
test_combined (unittest_subtest.SubTest) ... FAIL
test_with_subtest (unittest_subtest.SubTest) ...
================================================================
FAIL: test_combined (unittest_subtest.SubTest)
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_subtest.py", line 13, in test_combined
    self.assertRegex('abc', 'B')
AssertionError: Regex didn't match: 'B' not found in 'abc'

================================================================
FAIL: test_with_subtest (unittest_subtest.SubTest) (pattern='B')
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_subtest.py", line 21, in test_with_subtest
    self.assertRegex('abc', pat)
AssertionError: Regex didn't match: 'B' not found in 'abc'

================================================================
FAIL: test_with_subtest (unittest_subtest.SubTest) (pattern='d')
----------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_subtest.py", line 21, in test_with_subtest
    self.assertRegex('abc', pat)
AssertionError: Regex didn't match: 'd' not found in 'abc'

----------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (failures=3)
```

## 11. 跳过相关测试

当测试所需的外界条件没有得到满足时，我们常常会跳过相关的测试。可通过`skip()`装饰器来跳过测试类和方法，`skipIf()`和`skipUnless()`装饰器用来在跳过测试前检测相应的条件。

```python
# unittest_skip.py

import sys
import unittest

class SkippingTest(unittest.TestCase):
    
    @unittest.skip('always skipped')
    def test(self):
        self.assertTrue(False)
        
    @unittest.skipIf(sys.version_info[0] > 2, 'only runs on python 2')
    def test_python2_only(self):
        self.assertTrue(False)
        
    @unittest.skipUnless(sys.platform == 'Darwin', 'only runs on macOS')
    def test_macos_only(self):
        self.assertTrue(True)
        
    def test_raise_skiptest(self):
        raise unittest.SkipTest('skipping via exception')
```

对于一些无法用单句表达式传递给`skipIf()`或者`skipUnless()`函数的复杂情况，测试用例可通过触发`SkipTest`异常来达到跳过测试的目的。

```
$ python3 -m unittest -v unittest_skip.py
test (unittest_skip.SkippingTest) ... skipped 'always skipped'
test_macos_only (unittest_skip.SkippingTest) ... skipped 'only
runs on macOS'
test_python2_only (unittest_skip.SkippingTest) ... skipped 'only
runs on python 2'
test_raise_skiptest (unittest_skip.SkippingTest) ... skipped
'skipping via exception'

----------------------------------------------------------------
Ran 4 tests in 0.001s

OK (skipped=4)
```

## 12. 忽略失败的测试用例

我们可通过`expectedFailure()`装饰器来忽略注定失败的测试用例。

```python
# unittest_expectedfailure.py

import unittest

class Test(unittest.TestCase):
    
    @unittest.expectedFailure
    def test_never_passes(self):
        self.assertTrue(False)
        
    @unittest.expectedFailure
    def test_always_passes(self):
        self.assertTrue(True)
```

如果一个测试用例一开始认为测试不通过却在实际测试中通过，这种情况也会被认为是一种特殊形式的测试失败，其终端显示的结果是**unexpected failure**。

```
$ python3 -m unittest -v unittest_expectedfailure.py
test_always_passes (unittest_expectedfailure.Test) ...
unexpected success
test_never_passes (unittest_expectedfailure.Test) ... expected
failure

----------------------------------------------------------------
Ran 2 tests in 0.001s

FAILED (expected failures=1, unexpected successes=1)
```

## 参考文献

1. https://pymotw.com/3/unittest/index.html
2. https://docs.python.org/3.5/library/unittest.html
3. https://stackoverflow.com/questions/12136762/assertalmostequal-in-python-unit-test-for-collections-of-floats