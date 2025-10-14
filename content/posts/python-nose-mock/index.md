---
title: "使用 nose 与 mock 对 Python 程序进行简单的单元测试"
slug: "python-nose-mock"
date: "2018-11-03T09:08:00+0000"
lastmod: "2025-01-17T01:50:51+0000"
draft: false
tags:
  - "Python"
  - "Test"
visibility: "public"
---
# 0X00 install

安装nose：`pip install nose`
安装mock：`pip install mock`

> Python3 中mock模块已成为标准库，无需单安装

在任意目录下执行`nosetests`看到有输出就是已经安装好了`nose`。进入到`Python shell`中执行`import mock`没有报错也就是`mock`安装好了。

# 0X01 用于测试的代码

这里先贴出这次被测的代码`simple_math.py`，是一个非常简单的数字计算类。

```python
    #!/usr/bin/env python
    # coding=utf-8

    class MyMath:
        def my_add(self, a, b):
            return a + b

        def my_subtraction(self, a, b):
            if a and b:
                return a - b
```

> 这里的代码是有问题的，毕竟是要拿来作为单元测试的样例的嘛。

# 0X02 编写单元测试

我们要针对上述文件创建一个新的`test.py`来测试其中的`MyMath`类。

```python
    #!/usr/bin/env python
    # coding=utf-8

    from simple_math import MyMath


    mt = MyMath()


    def test_add():
        res = mt.my_add(3, 5)
        assert res == 8


    def test_subtraction_001():
        res = mt.my_subtraction(233, 0)
        assert res == 233


    def test_subtraction_002():
        res = mt.my_subtraction(233, 11)
        assert res == 222
```

从代码中可以看到首先导入了需要测试的类`MyMath`，然后就写了几个`test_`开头的方法，方法内部是模拟调用`MyMath`中的方法，并将得到的结果与预期结果相互匹配，最终使用`assert`语法来判断是否返回了理想的值。测试代码写好之后在当前目录下执行`nosetest -v`来运行我们的单元测试吧，输出结果如下。

```
    test.test_add ... ok
    test.test_subtraction_001 ... FAIL
    test.test_subtraction_002 ... ok

    ======================================================================
    FAIL: test.test_subtraction_001
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "/usr/lib/python2.7/site-packages/nose/case.py", line 197, in runTest
        self.test(*self.arg)
      File "/home/shawn/Workstadion/utils_test/test.py", line 17, in test_subtraction_001
        assert res == 233
    AssertionError

    ----------------------------------------------------------------------
    Ran 3 tests in 0.002s

    FAILED (failures=1)
```

# 0X003 简单解释一下

上面的测试就执行完了，那我们来拆分一下这个单元测试的流程吧。

  1. 首先我们执行了`nosetest -v`命令，这个命令就是开始进行单元测试的，其中`-v`参数是用来展示更加丰富的输出的，如果不加这个参数的话运行结果会更清爽一些
  2. 执行命令之后nose会自己查找当前目录下名为`test.py`或其他以`test_`开头的python文件，并且执行这些文件中编写好的以`test_`开头的方法（就比如我们代码中的`test_add/test_subtraction`）
  3. 逐个执行上面找到的方法
  4. 输出最后结果

如果在执行单个`test_xxx`方法的时候没有抛出异常，那么就认为这个测试(test case)是通过了的，如果抛出了异常则认为此处的测试不能通过。但是在单元测试中有一点与普通Python程序不同，普通Python程序遇到抛出的异常时就会层层上抛，而这里会收集展示出来，然后继续运行下面的测试。

# 0X04 换一个姿势写测试

还是上面的被测代码，这次不是单纯使用多个方法来完成测试了。面向对象的思想也可以对应的放到单元测试中，比如我们针对`MyMath`类搞一个单元测试，这些测试内容也整理为一个类，不过值得注意的是**这里的类名一定要为`TestClass`**否则是运行不到的。

```python
    #!/usr/bin/env python
    # coding=utf-8

    from simple_math import MyMath


    class TestClass:
        def test_add(self):
            mt = MyMath()
            res = mt.my_add(3, 5)
            assert res == 8

        def test_subtraction_001(self):
            mt = MyMath()
            res = mt.my_subtraction(233, 0)
            assert res == 233

        def test_subtraction_002(self):
            mt = MyMath()
            res = mt.my_subtraction(233, 11)
            assert res == 222
```

# 0X05 setup与teardown方法

上面的测试中我们每一个test_case中都有一行`mt = MyMath()`是不是感觉有些蠢，其实是可以避免这个方法的。定义两个方法`setup/teardown`，这些方法在每执行一个testcase的时候都会执行，不同的是`setup`执行在testcase之前，而`teardown`执行在testcase之后。下面例子中就是这样的，每次使用`mt`对象时都实例化一个新的，用完再删掉。

```python
    #!/usr/bin/env python
    # coding=utf-8

    from simple_math import MyMath


    class TestClass:
        def setup(self):
            self.mt = MyMath()

        def teardown(self):
            self.mt = None

        def test_add(self):
            res = self.mt.my_add(3, 5)
            assert res == 8

        def test_subtraction_001(self):
            res = self.mt.my_subtraction(233, 0)
            assert res == 233

        def test_subtraction_002(self):
            res = self.mt.my_subtraction(233, 11)
            assert res == 222
```

虽然上面的`mt`对象没有必要每次生成新的，但是很多情况下其实是需要我们这么做的。考虑这么一种情况：有一个`class Student`需要测试，而且待测方法有非常非常多，不仅会计算Student实例的各个属性，还要对其进行更新、添加、删除等操作。那么这种情况下每次操作都生成一个新的Student实例并且在testcase结束之后删掉它就是非常有必要的了。

> 因为我们不能保证每个方法的幂等性，比如万一这个Student是男的，你在测试了`student.change_sex()`方法之后显然他就不再是男的了。那接下来再测试一些依赖与性别的地方时就会出问题，比如`get_gender()`方法你很有可能会写`assert student.get_gender() == "Male"`，那这个时候就出错了。

# 0X06 模拟一些对象

对，你没有对象的时候可以模拟对象出来(hhh。

还是，首先有一个场景：工作中编写单元测试，被测功能简单说是“从某一接口拿到数据，并对数据进行处理”。那么我们就可以写成这么一个操作

```python
    def test_operation_data_from_api(parms):
        response = get_data_from_api(parms)
        op_result = operation_data(
            id=response.id,
            order_type=response.order_type,
            administrator_name=response.administrator_name,
        )
        assert op_result.status == 'success'
```

这段代码看起来是没有问题的，但是如果`get_data_from_api`调用的API是收费API呢，每次跑单元测试都要去请求一次吗？如果API巨慢无比，每次都要几秒钟才回得来，所有测试有需要调用上百次这个接口呢，我们就干等着吗？这显然是不合理的。此时就可以使用最初提到的`mock`来模拟数据从而解决上述问题。

```python
    import mock

    def test_operation_data_from_api(parms):
        response = mock.Mock(   # Mock可以模拟几乎所有对象、方法等
            id=3,
            order_type='cpu',
            administrator_name='root'
        )
        op_result = operation_data(
            id=response.id,
            order_type=response.order_type,
            administrator_name=response.administrator_name,
        )
        assert op_result.status == 'success'
```

通过对response的模拟，我们可以实现不用真正去请求API就能继续测试的方法。因为我们这里编写测试的目的是正确处理response所以可以模拟response。如果我们的目的是测试`get_data_from_api()`这个方法的话那就不能想这样模拟了。

# 0X07 Mock模拟其他东西

mock可以模拟一个属性，多层属性还能模拟方法。模拟多个属性与模拟一个属性是一样的，只需要一路`.`下去就可以。中途遇到没有自己定义过的会自动生成一个`mock.Mock()`放进去。比较有意思的是模拟方法，使用`Mock.method_name.return_value`就可以模拟`method_name`方法，并且`return_value`就是这个方法的返回值。

```python
    #!/usr/bin/env python
    # coding=utf-8

    import mock


    if __name__ == '__main__':
        # 模拟一个属性
        mk = mock.Mock()
        mk.name = 233
        print mk.name

        # 模拟多层属性
        mk = mock.Mock()
        mk.base_attribute.name = 'Shawn'
        print mk.base_attribute.name

        # 模拟一个方法
        mk = mock.Mock()
        mk.get_data_from_api.return_value = {'data': []}
        print mk.get_data_from_api()
```