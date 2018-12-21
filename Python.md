## Circular (or cyclic) imports in Python
>* What will happen if two modules import each other?

```
Imports are pretty straightforward really. Just remember the following:

'import' and 'from xxx import yyy' are executable statements. They execute when the running program reaches that line.

If a module is not in sys.modules, then an import creates the new module entry in sys.modules and then executes the code in the module. It does not return control to the calling module until the execution has completed.

If a module does exist in sys.modules then an import simply returns that module whether or not it has completed executing. That is the reason why cyclic imports may return modules which appear to be partly empty.

Finally, the executing script runs in a module named __main__, importing the script under its own name will create a new module unrelated to __main__.

Take that lot together and you shouldn't get any surprises when importing modules.
```

>* import 和 from ... import, 都是指令, 在程序跑到那一行的时候执行
>* 如果一个 module 不在 sys.modules 中(首次import), 那么会在 sys.modules 中创建一个新的模块入口, sys.modules 其实就是一个 Dict, 然后去执行 这个module 中的内容, 它会一直等待, 知道 这个module 中的内容执行完成。
>* 如果一个 module 在 sys.modules 中, 直接返回 sys.modules['module_name'] , 如果是循环引用的情况, 这个时候这里就是空的(因为没有执行完成).
>* 当前执行的脚本, 都在一个叫做 --main-- 的 module 中运行, 在这个下面 import, 和 --main-- 无关

## Marshal and Pickle

>* Marshal 和 Pickle 都是用来做数据序列化的
>* Marshal 处理不了 Python Obj, 它主要用来处理 .pyc 文件
>* Pickle 对共享对象(多个引用指向的同一对象), Pickle 会存储一次, 其他引用指向主副本, 这也是 Pickle 能处理循环引用的原因
>* 他们都处理不了 弱引用
>* Pickle 处理 类 必须是可导入, 且在存储对象时, 存在同一模块中(import 了也算)
>* Marshal 因为支持 .pyc 文件, 所以不保证全版本通用, 而 Pickle 向后兼容

## eval and exec

>* eval 函数是执行一段表达式
```
eval(expression, globals=None, locals=None)
```
>* expression：必选参数，可以是字符串，也可以是一个任意的code对象实例（可以通过compile函数创建）。如果它是一个字符串，它会被当作一个（使用globals和locals参数作为全局和本地命名空间的）Python表达式进行分析和解释。
>* globals：可选参数，表示全局命名空间（存放全局变量），如果被提供，则必须是一个字典对象。
>* locals: 可选参数，表示当前局部命名空间（存放局部变量），如果被提供，可以是任何映射对象。如果该参数被忽略，那么它将会取与globals相同的值。

>* exec 函数是执行一段代码, 不像 eval 只能执行一段表达式
```
exec(object[, globals[, locals]])
```
>* object：必选参数，表示需要被指定的Python代码。它必须是字符串或code对象。如果object是一个字符串，该字符串会先被解析为一组Python语句，然后在执行（除非发生语法错误）。如果object是一个code对象，那么它只是被简单的执行。

## Python MRO
>* MRO（Method Resolution Order）：方法解析顺序。
>* 只需要记住一点, 以前的 python 版本和 旧式类 有各种问题, python3 统一用 新式类 和 C3 算法解决多继承的问题
>* 其实问题就是, 父子节点中, 不管用广度还是深度优先, 在各种重写的情况下, 都会存在 找不到正确的 父类或者子类的方法 的问题发生
>* C3 的核心就是 merge 算法, 这个算法保证了 从子类 寻找 父类的关系
>* 遍历执行merge操作的序列，如果一个序列的第一个元素，在其他序列中也是第一个元素，或不在其他序列出现，则从所有执行merge操作序列中删除这个元素，合并到当前的mro中。

## cProfile and Profile
>* 一般来说用 cProfile, 官方文档上说, 这个基于C, 耗时更少, 也更精确
>* 伴生的库 pstat, 用来分析 profile 结果的库
>* 简单的 profile, 即
``` Python
import cProfile
import re
cProfile.run('re.compile("foo|bar")')
```
>* 相当于运行 exec(), 加上 Profile 分析
>* 方便的分析要像这样:
``` python
import cProfile, pstats, io
pr = cProfile.Profile()
pr.enable()

class A(object):
    """docstring for A"""
    def __init__(self):
        super(A, self).__init__()

    def Test(self):
        i = 0
        for x in range(10000000):
            i += x     
a = A()
a.Test()

pr.disable()
s = io.StringIO()
ps = pstats.Stats(pr, stream=s)
ps.print_stats()
print(s.getvalue())
```
>* 嗯, 它就会解析中间那一段代码, 奇怪的是, 这里只能执行函数调用的部分, 直接写个循环进去并不能检测到。
