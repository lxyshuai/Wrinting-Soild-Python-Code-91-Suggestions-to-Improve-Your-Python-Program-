## assert
1. 断言说明
    * ``__debug__``默认是True,只读不可修改
    * 断言有代价,影响性能.pyhon没有严格定义调试和发布模式.通过``-O``参数禁用断言,但这种方式并不优化字节码,而是忽略断言相关语句
2. 断言使用注意事项
    * 不要滥用,应该使用在正常逻辑不可达的地方或正常情况下总是为真的情况
    * 异常处理优于断言
    * 不要用断言判断用户输入,应该用条件判断+错误提示
    * 函数调用,需要确认返回值是否合理的时候可以使用断言
    * 但条件是业务逻辑继续下去的先决条件时可以使用断言

## 数值交换不使用中间变量
```python
x, y = y, x
```
* 不借助中间变量,代码更简洁,性能更好
* python表达式一般从左到右计算,但是表达式赋值先右后左
* 字节码分析参照书本(P22)

## Lazy evaluation(延迟计算)
1. 避免不必要计算
    * if x and y中,x为True不计算y.
    * if x or y中,x为False不计算y.
    * 根据x和y的值的可能性排序变量在if语句中位置
2. 节省空间,使无限循环变成可能
    * 生成式表达式``yield``

## 枚举
1. 使用类属性
    ```python
    class Seasons:
        Spring, Summer, Autumn, Winter = range(4)
     ```
2. 借助函数
    P25
3. 使用``collections.namedtuple``.

## 不使用type进行类型检查
* 动态型强类型脚本语言,运行时自动进行类型检查并根据需要隐式转换
* 通过异常处理类型
* 新式类和旧式类type返回结果又差异
* 用户自定义类型type不能准确返回结果
* 可以通过工厂函数或者isinstance()约束用户输入类型和我们期望一致

## 尽量转换为浮点类型再做除法
* 地板除和真实除法的区别和使用的地方
* 浮点数不用与判断,因为不精确

## eval()
* 防止eval()注入
* 如果使用对象不是信任的,避免使用eval(),在需要使用的地方用安全性更好的ast.literal_eval代替

## 使用enumerater()获取序列的索引和值
```python
li = [1, 2, 3, 4]
for i, e in enumerater(li):
    print i, e
```
* enumerater不适合字典,虽然不错误,但是结果与期望大相径庭

## == 和is 的使用场景
* is 是对象标识符,用来检查对象的标示符是否一致,也就是比较两个对象是否拥有同一块内存空间,不能被重载
* == 检验连个对象的值是否相等,实际调用``__eq()__``方法,可以重载.

## 使用Unicode保证兼容性
* str和Unicode(P38)
* 参考python问题中相关网页
* 因为print语句它的实现是将要输出的内容传送了操作系统，操作系统会根据系统的编码对输入的字节流进行编码
* 编码声明的作用:即每个文件在最上面的地方的# coding=gbk ，作用有三：
    1. 声明源文件中将出现非ascii编码，通常也就是中文；
    2. 在高级的IDE中，IDE会将你的文件格式保存成你指定编码格式。（如pycharm）
    3. 决定源码中类似于u’哈’这类声明的将’哈’解码成unicode所用的编码格式，也是一个比较容易让人迷惑的地方。

## 构建合理的包层次来管理module
* Package(包)包含模块,还包含一个``__init__.py``文件
```python
Package/__init__.py
    Module1.py
    Module2.py
    SubPackage/__init__.py
        Module1.py
        Module2.py
```
* 包导入
    1. 直接导入一个包
    ```python
    import Package
    ```
    2. 导入子模块或子包
    ```python
    from Package import Module1
    import Package.Module1
    from Package import Subpackage
    import Package.Subpackage
    from Package.Subpackage import Module1
    import Package.Subpackage.Module1
    ```
* ``__init__.py``最明显作用是使包和普通目录区分,其次可以在该文件声明模块级别的import语句从而使其包级别可见
```python
# 如果要importb包Package中Module1中的类Test,当__init__.py文件为空需要使用完整路径申明
from Package.Module1 import Test
# 如果在__init__.py文件中添加from Module1 import Test,可以直接使用如下导入
from Package import Test
# 当意图使用from Package import *将包Package中所有模块导入当前命名空间并不能使得导入的模块生效,Python不能正确判断使得导入的模块生效(因为不同平台间文件命名规则不同),
因此它仅仅执行__init__.py文件.如果要控制模块导,则需要修稿__init__文件``
# 通过定义__init__.py文件中的__all__变量,控制需要导入的子包或者模块
```






