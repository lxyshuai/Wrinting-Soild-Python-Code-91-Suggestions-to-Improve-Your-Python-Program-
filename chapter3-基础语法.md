## 有节制的使用fromm...import语句
* import机制:
    1. Python在初始化运行环境的石红会预先加载一批内建模块导内存中,这些模块相关信息被存放在sys.modules中
    2. 当加载一个新模块,解释器完成下面步骤:
        1. 在sys.modules中查看该模块是否存在,如果存在,将其导入到当前命名空间,加载结束.
        2. 如果在sys.modules中找不到对应的模块的名称，则为需要导入的模块创建一个字典对象，并将该对象信息插入sys.modules
        3. 加载前确认是否需要对模块对应的文件进行编译，如果需要则先进行编译
        4. 执行动态加载，在当前模块的命名空间执行编译后的字节码，并将其中所有的对象放入模块对应字典中
* 直接使用import和from a import B两者存在差异，后者直接将B暴露于当前命名空间，而将a加载到sys.modules
* from...import...无节制使用带来的问题:
    1. 命名空间冲突.一下几种情况考虑使用from...import...:
        1. 当只需要导入部分属性或方法
        2. 模块中高频的属性方法
        3. 导入包的子模块
    2. 循环嵌套导入问题

## 优先使用absolute import导入模块
* relative import:局部范围的模块将覆盖同名的全局范围的模块.相对导入格式为 from . import B 或 from ..A import B，.代表当前模块，..代表上层模块，...代表上上层模块，依次类推
* absoulte import:绝对导入的格式为import A.B 或 from A import B
* 相对导入可以避免硬编码带来的维护问题，例如我们改了某一顶层包的名，那么其子包所有的导入就都不能用了。但是 存在相对导入语句的模块，不能直接运行，否则会有异常
* 参考<<python核心编程>>

## ``i+=1`` != ``++i``
* ``++i``合法，解释为+(+i)
* ``--i``合法，解释为-(-i)

## 使用with自动关闭资源
```python
with context_expression [as target(s)]:
    with-body
```
```python
context_manager = context_expression
exit = type(context_manager).__exit__
value = type(context_manager).__enter__(context_manager)
exc = True   # True 表示正常执行，即便有异常也忽略；False 表示重新抛出异常，需要对异常进行处理
try:
    try:
        target = value  # 如果使用了 as 子句
        with-body     # 执行 with-body
    except:
        # 执行过程中有异常发生
        exc = False
        # 如果 __exit__ 返回 True，则异常被忽略；如果返回 False，则重新抛出异常
        # 由外层代码对异常进行处理
        if not exit(context_manager, *sys.exc_info()):
            raise
finally:
    # 正常退出，或者通过 statement-body 中的 break/continue/return 语句退出
    # 或者忽略异常退出
    if exc:
        exit(context_manager, None, None, None)
    # 缺省返回 None，None 在布尔上下文中看做是 False
```
* with语句代码块执行过程:
    1. 执行 context_expression，生成上下文管理器 context_manager
    2. 调用上下文管理器的 __enter__() 方法；如果使用了 as 子句，则将 __enter__() 方法的返回值赋值给 as 子句中的 target(s)
    3. 执行语句体 with-body
    4. 不管是否执行过程中是否发生了异常，执行上下文管理器的 __exit__() 方法，__exit__() 方法负责执行“清理”工作，如释放资源等。如果执行过程中没有出现异常，或者语句体中执行了语句 break/continue/return，则以 None 作为参数调用 __exit__(None, None, None) ；如果执行过程中出现异常，则使用 sys.exc_info 得到的异常信息为参数调用 __exit__(exc_type, exc_value, exc_traceback)
    5. 出现异常时，如果 __exit__(type, value, traceback) 返回 False，则会重新抛出异常，让with 之外的语句逻辑来处理异常，这也是通用做法；如果返回 True，则忽略异常，不再对异常进行处理
* 自定义上下文管理器
    * 开发人员可以自定义支持上下文管理协议的类。自定义的上下文管理器要实现上下文管理协议所需要的 __enter__() 和 __exit__() 两个方法：
        * context_manager.__enter__() ：进入上下文管理器的运行时上下文，在语句体执行前调用。with 语句将该方法的返回值赋值给 as 子句中的 target，如果指定了 as 子句的话
        * context_manager.__exit__(exc_type, exc_value, exc_traceback) ：退出与上下文管理器相关的运行时上下文，返回一个布尔值表示是否对发生的异常进行处理。参数表示引起退出操作的异常，如果退出时没有发生异常，则3个参数都为None。如果发生异常，返回
True 表示不处理异常，否则会在退出该方法后重新抛出异常以由 with 语句之外的代码逻辑进行处理。如果该方法内部产生异常，则会取代由 statement-body 中语句产生的异常。要处理异常时，不要显示重新抛出异常，即不能重新抛出通过参数传递进来的异常，只需要将返回值设置为 False 就可以了。之后，上下文管理代码会检测是否 __exit__() 失败来处理异常
