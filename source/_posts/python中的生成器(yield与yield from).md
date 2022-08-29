---
title: python中的生成器(yield与yield from)
date: 2021-12-31 
comment: true
tags: 
- Asyncio专题
- 生成器
- 源码分析
categories:
- python

valine:
  appId: TsHNizG5g9yTU8EUAa1EIkH5-gzGzoHsz #Your_appId
  appKey: KSmBGGzCyE4Pnop6qF4aD39B #Your_appkey
  placeholder: ヽ(○´∀`)ﾉ♪ # Comment box placeholder
  pageSize: 10 # Pagination size
  lang: zh-CN
---

# 导读

本文作为**asyncio专题**系列的引导文章，主要定位是引导读者熟悉`asyncio` 相关的前置知识**生成器**。文章字数较多（不算代码字数 1w+，算上代码 1.8w+），全部阅读完大概需要43分钟。

笔者深知字数较多的文章会让人丧失继续阅读的兴趣，并且读者水平不同，可能有人早已掌握文章内容，因此我在下面简单描述了本文四个小节的重点内容，**您可以根据自身情况选择阅读**。

## 初始阶段，迭代器

1. 来自 PEP234。本节重点描述了迭代器的功能与优势。
2. 迭代器的出现增强了 python 中的迭代能力，并且给 dict 和 file 类型提供更方便、快速的迭代方式。
3. 其主要依赖的魔术方法有两个**iter**和**next**。
4. 存在的问题是**当调用者想要维护迭代的中间值的时候，会比较困难，通常需要用回调函数等不直观的方式**。

## 第二阶段，生成器

1. 来自 PEP255。生成器完全继承迭代器协议。生成器是非常重要的一个特性，后续的协程也是基于它，因此本节用了大部分的篇幅**分析生成器源码**，力求从**根本上理解生成器的运行原理**。
2. 生成器中，解决了**迭代器**中存在的问题，它允许生成器函数从停止的地方继续执行，并且提供了新函数让调用者与生成器函数交互。
3. 新增了一个表达式`yield`，以及新增了`send`函数。
4. 该阶段仍然存在一些问题，比如**生成器之间的调用（代码解耦）**、**获取最后 return 的返回值**、**生成器与调用者之间的异常处理**等。

## 第三阶段，委托生成器

1. 来自 PEP380。委托生成器的作用是增强生成器，它**解决了上面生成器有关的问题**。坏处是**委托生成器异常的难理解**(PEP380 的作者也同意这个观点)。但是官方给了一段伪代码，这段伪代码直观的阐明了委托生成器的工作原理，因此本节的重点在于**结合实际例子代入到伪代码中，详细描述了委托生成器的执行逻辑**。
2. 委托生成器会作为类似**中间人**的角色，将左面的表达式变成迭代器，不断的从调用者获取值传给迭代器处理，又把迭代器的处理结果传给调用者，在这中间过程中又做了完整的异常处理。
3. 新增一个表达式`yield from`。
4. 该阶段在 PEP342 中又增加了一些方法 close，throw 等，为用生成器实现协程铺平道路，但是依旧不可否认的是**生成器虽然可以实现协程，但它并不是真正的协程，使用的关键字与其他语言也不同，因此并不方便掌握**。

## 现阶段，协程 async/await

1. 在 PEP492 中，终于提出了真正的协程。本节的重点是用一个简单的小例子，表明了生成器和`await`在使用上的区别。
2. 协程和生成器，最明显的不同就是`await`必须配合可调用对象使用，表现出来的作用与`yield from`的均相同。
3. 在该阶段引入了很多新特性。包括一个新的概念可调用对象`awaitable`，一个重要的魔术方法`__await__`实现自定义协程，两个重要的魔术方法`__aenter__`和`__aexit__`用来实现自定义异步上下文管理器，两个表达协程的关键字`async/await`以及其他众多的新增概念（本系列文章后面均会提及）。
4. 本文定位在于熟练掌握协程的前置知识，因此对于协程本身相关并没有太多的内容。

# 幼年期的迭代器 iterator

关键字`yield`首次被引入到 python 中是在 PEP255，但是它依赖的另外一个特性尤在它之前出现，那就是迭代器。

PEP234 中首次引入了迭代器这一概念。并且引入了一个内置函数`iter`用来创建迭代器。`iter`函数有两种用法：

```python
# 1、iter(iterable) 以一个可迭代对象作为参数
iter_obj = [1, 2, 3]  # list可能是我们最熟悉的可迭代对象
list_iter = iter(iter_obj)
print(list_iter)  # <list_iterator object ...>

# 2、iter(callable, sentinel) 以一个可调用对象做参数 并设置哨兵
def call_obj():  # 定义一个函数作为可调用对象
    return [1, 2, 3]
call_iter = iter(call_obj, 2)
print(call_iter)  # <callable_iterator object ...>
```

而所有的迭代器都有一个**魔术方法next**，以控制当我们使用内置函数`next`来调用它们进行迭代时候的行为。

```python
print(hasattr(list_iter , '__next__'))  # True
print(hasattr(call_iter , '__next__'))  # True
```

对于任意迭代器，我们都可以使用两种方式对他们进行迭代，一种是`for`循环的方式，另一种就是上面所说的用内置函数`next`。

并且这两种方式的迭代都做同一件事：从迭代器中取出元素。在下面的例子中我们先用`next`取出一个元素，之后再用`for`循环取出剩下的两个元素。

```python
print(f'next函数迭代：{next(list_iter)}')  # next函数迭代：1
for item in list_iter:
    print(f'for循环迭代：{item}')  # for循环迭代：2  for循环迭代：3
```

它们的区别是：对于`for`循环来说，迭代器中如果没有元素会直接终止循环；而`next`函数如果没有取到元素，则会报指定异常`StopIteration`。这样做的一个好处是迭代器的迭代过程不会被任何其他异常所阻断。

```python
# 在上面的例子中我们已经取出了list_iter的所有元素
next(list_iter)  # StopIteration
```

除了这些默认行为之外。在迭代器特性中，python 还为我们增强了`dict`和`file`两种基本类型的迭代。我们以字典为例。

**判断某个 key 是否在字典中存在**。如果没有迭代器，我们应写为：

```python
if dict.has_key(k): ...
```

但是有了迭代器，我们可以用更直观的方式表达这一语意：

```python
# if k in dict: ...
data = {
    'Jinx': '爆爆',
    'Vi': '蔚',
    'Caitlyn': '小蛋糕',
    'Jayce': '议员',
    'Viktor': '光荣进化'
}
if 'Jinx' in data:
    print(data['Jinx'])  # 爆爆
```

**对一个 dict 进行迭代访问**。该场景下，我们想要像访问其他可迭代序列一样，访问到字典的所有元素，在这样的迭代过程中，不应该修改`dict`，并且应该允许为现有的 key 设置 value，这意味着我们可以直接把字典的遍历设置成对字典 key 的遍历。

```python
for k in data:
    print(k)  # Jinx Vi Caitlyn Jayce Viktor
```

当然，还有另外一种方式也可以做到：

```python
for k in data.keys():
    print(k)  # Jinx Vi Caitlyn Jayce Viktor
```

比较两种方法，显然是第一种更快，因为它少了一步访问`dict.keys`函数的过程，而是直接遍历了`dict`的 key。

除了对`dict`和`file`的增强以外，迭代器更加重要的一个作用是控制自定义类的迭代行为。也就是说迭代器的强大之处在于它向后兼容所有模仿序列和映射的自定义类和扩展对象。

下面的例子，我们自定义一个类，来更改基本类型`dict`的默认迭代行为：

```python
class MyDict(dict):
    def __init__(self, data):
        super().__init__(data)
        self.idx = 0  # 维护当前迭代元素

    def __iter__(self):
        return self

    def __next__(self):
        try:
            item = list(self.values())[self.idx]
        except IndexError:
            raise StopIteration  # 必须使用该异常
        self.idx += 1
        return item

iter2 = iter(MyDict(data))
print(next(iter2))  # 爆爆
print(next(iter2))  # 蔚
for item in iter2:
    print(item)  # 小蛋糕 议员 光荣进化
```

在这个例子的结果中，我们把`dict`默认迭代 key 的行为变成了默认迭代 value，并且顺利的使用内置函数`next`和`for` 循环遍历了`dict`中的所有 value。

可以明显的看到除了`init`外，这个自定义类 MyDict 还**同时**实现了两个**魔术方法`iter`和`next`**。这两个魔术方法在 PEP234 中均有提到，其中**魔术方法 iter**是迭代器的核心函数，它的返回值必须是一个可迭代对象。它的含义是告诉解释器该自定义类是可迭代的，以及要迭代的对象是谁。而**魔术方法 next**的功能比较明显，即控制`MyDict`的默认迭代行为，并且把自定义类变成迭代器类型。

迭代器的作用：

1、使数据的迭代变得可以扩展；增强迭代性能；
2、提供更强性能的 dict 迭代；
3、为迭代提供一个接口，而不是让这一过程看起来像是随机访问；
4、兼容性极强，可迭代序列、可调用对象、自定义类、扩展对象等都可以定义迭代的行为；
5、使迭代的代码可以更加简洁；

# 迭代器进化-->生成器

在 PEP255 中引入了新的概念，也就是本文的主角：生成器以及它的衍生品 yield。

## 问题

**当生产者函数想要在生成的值之间维护状态时**。通常要用麻烦的方式，比如用回调函数等，但遗憾的是，即便可以实现，代码也不好理解。

幸好，我们有了迭代器，无论何时，只要生产者函数需要下一个值就可以简单的通过内置函数 next 来获取，这样使代码看起来简洁、优雅、易读，并且只需要把当前正在使用的值放到内存中，也可以简化内存的占用，还有一个好处是随时可以结束对生产者函数产出值的处理，即不在调用 next 函数生产新的值。

但是使用迭代器，会引发另一个问题：**必须要手动维护当前函数的执行状态**。然而这是一笔非常大的开销。

```python
class MyDict(dict):
    def __init__(self, data):
        ...
        self.idx = 0  # 维护当前迭代元素
    def __next__(self):
        ...
        self.idx += 1
```

以上节迭代器中的 MyDict 为例，我想要让 MyDict 每次都向迭代字典的下一个值，就要用一个额外的下标参数 idx 去控制，这样才能在第二次调用 next 的时候生产出第二个 value 值。显然如果在更复杂的场景中，**“维护当前状态”这件事会花费更大的开销**。

## yield

为了解决这件事，生成器应运而生。生成器可以保存函数运行的局部状态，以便函数在停止的地方再次恢复。

### 有两种方式定义生成器

1、类似列表推导式，但是生成器使用小括号。创建一个过滤列表数字，只保留偶数的生成器。

```python
data = [1, 2, 3, 4]
l = [item for item in data if item % 2 == 0]
print(l1)  # [2, 4]
gen1 = (item for item in data if item % 2 == 0)
print(g1)  # <generator object gen...>
```

2、另外一种方式就是在 PEP255 中新引入的一个关键字`yield`。**即该 yield 所在的函数，整体就是一个生成器**。

```python
def gen(data):
    for item in data:
        if item % 2 == 0:
            yield item
gen2 = gen(data)
print(gen2)  # <generator object gen...>
```

### 生成器完全支持迭代器协议

为了保持迭代器相同的特性，生成器完全支持了迭代器的协议（即实现了`iter`和`next`两个魔术方法），因此生成器可以使用迭代器的方式调用。

```python
print(next(gen2))  # 2
for item in gen2:
    print(item)  # 4
print(next(gen2))  # StopIteration
```

#### 生成器会记录函数当前执行位置，下次执行从该位置开始

用一个简单例子来说明该特性。（该例子没有任何的实际作用，仅仅是为了阐明生成器在代码中的执行流程。）

```python
def simple_coro2(a):  # 代码1
    print(f'start a = {a}')  # 代码2
    b = yield a  # 代码3
    print(f'receive b = {b}')  # 代码4
    c = yield a + b  # 代码5
    print(f'receive c = {c}')  # 代码6
    
coro2 = simple_coro2(66)  # 执行该函数
print(coro2)  # <generator object simple_coro2...>
```

现在得到了一个生成器，可以看到如果是生成器函数，当我们直接执行它的时候，仅仅是得到一个生成器对象。如果想操作该生成器函数，必须使用内置函数`next()`和`generator.send()`。

生成器函数有两个功能：**产出**和**让步**。简单讲，就是将`yield`右边的值传递给调用者，并且将执行权交给调用者。

```python
val = next(coro2)  # 打印start a = 66
print(val) # 66 将yield右面的{a}传给调用者
```

此时为了使生成器函数向下执行，可以继续使用`next()`。

```python
next(coro2)
# TypeError: 'int' and 'NoneType'
```

继续`next()`发现，报错`TypeErroe`。这是因为在代码 3 处参数 b 在`yield`的左面，表示接收了一个`yield`(调用者)提供的值，是需要调用方使用`send()`函数提供的，而不是 `next()`。如果没有使用`send()`函数提供数据，强行使用`next()`让生成器继续向下，则会产生一个默认值`None`做为 b 的值。所以再执行到代码 5 的时候 a 是`int`类型、b 是`None`，因此报错。正确的做法是使用`send()`方法为生成器函数提供值。

```python
val2 = coro2.send(15)  # receive b = 15
print(val2)  # 81

val3 = coro2.send(10)  # receive c = 10
```

可以看到，在调用者调用`send()`之后，不仅将`send()`提供的值传递给生成器函数，还能让生成器函数继续向下执行，而且最重要的一点**生成器函数记住了它自己执行到了`yield`处**，下次执行就从`yield`（代码 3）的左面继续执行。

为了更加熟悉生成器的运行，引用《流畅的 python》中的例子。该例子中我们要完成**定义一个生成器，求平均值并且返回结果。**

```python
# count是参与计算的值的个数，average是平均值
Result = namedtuple('Result', 'count average')
def averager():
    total = 0.0
    cnt = 0
    average = None
    while True:  # 死循环 会一直接值 并计算
        term = yield average  # 代码1
        if not term:  # 代码2
            break
        total += term
        cnt += 1
        average = total / cnt  # 代码3
    return Result(cnt, average)

coro_avg = averager()  # 实例化生成器
next(coro_avg)  # 预先激活生成器 执行到代码1
# 调用者发送一个值10 执行到代码3计算出平均值
# 再执行到带代码1 通过yield将average给调用者
print(coro_avg.send(10))  # 10
print(coro_avg.send(20))  # 15
try:
    # 当调用者传给生成器一个None的时候，代码2中的break会被执行，推出死循环
    print(coro_avg.send(None))
except StopIteration as e:
    # 从异常中获取return的返回值。
    print(e.value)  # Result(count=2, average=15.0)
```

1. 代码 1：产出`yield`值给调用者，并且让出执行权给调用者，并且接受调用者回传的值。
2. 代码 2：调用者传入`None`的时候作为循环结束信号，生成器抛出异常`StopIteration`，退出循环，走到 return 语句。
3. 代码 3：主要逻辑，计算平均值。

## 源码分析

### 字节码
**想要知道生成器是如何工作的**，我们可以去字节码里寻找答案。

```python
from dis import dis
print(dis(simple_coro2))

......
             10 CALL_FUNCTION            1
             12 POP_TOP
  6          14 LOAD_FAST                0 (a)
             16 YIELD_VALUE
             18 STORE_FAST               1 (b)
......
```
我们把这个字节码连起来读一下就是**调用一个函数，从栈顶弹出执行元素，加载yield后面的常量**，`yield`这个关键字对应的字节码**YIELD_VALUE**。

在执行`CALL_FUNCTION`字节码的时候，就会判断出该函数是生成器函数，从而最后返回一个生成器，而不是直接执行。

### 虚拟机栈帧
在[以前的文章](https://mp.weixin.qq.com/s/p9Nhb4ddJi5clJkcBDLfRQ)中，已经提到了python的字节码是由虚拟机去执行的，并且python代码被编译后会产生`PyCodeObject`对象。但是想要明白生成器的工作过程，还要简单了解一下虚拟机到底是如何执行这些字节码的。

首先虚拟机会为要执行的代码准备好**执行环境**，也就是保存了执行上下文的栈帧对象。为了与后文的内部栈帧区分，我们叫它**外部栈帧**，它的头文件在`Include/cpython/frameobject.h`。
```cpp
struct _frame {
    PyObject_HEAD
    struct _frame *f_back;      /* 前一个_frame，也就是该栈帧的调用者 */
    struct _interpreter_frame *f_frame; /* 指向实际栈帧数据的指针*/
    PyObject *f_trace;          /* Trace function */
    ......
};
```
在头文件中，我们找到了一个重要的结构体，**内部栈帧`_interpreter_frame`**，该结构体中承载着虚拟机执行环境的真正数据，它的相关头文件在`Include/internal/pycore_frame.h`。
```cpp
typedef struct _interpreter_frame {
    PyFunctionObject *f_func; /* 函数对象引用 */
    PyObject *f_globals; /* 全局名字空间 */
    PyObject *f_builtins; /* 内建名字空间 */
    PyObject *f_locals; /* 局部名字空间 *、
    PyCodeObject *f_code; /* 代码对象引用 */
    PyFrameObject *frame_obj; /* 栈帧对象 */
    PyObject *generator; /* 生成器引用 */
    struct _interpreter_frame *previous;  /* 前一个内部栈帧 */
    int f_lasti;       /* 最后执行的字节码 */
    int stacktop;     /* Offset of TOS from localsplus  */
    PyFrameState f_state;  /* 栈帧状态 */
    ......
} InterpreterFrame;
```
该结构体里面有三个最关键的属性：
1. f_code：该属性是最核心的当前正在执行的代码对象，字节码就在其中。
2. f_lasti：该属性也是核心属性，它记录了上一条执行的字节码序号，默认是-1，表示没开始执行。这样就能知道下一条将要执行的字节码是什么。
3. previous：指向上一个栈帧对象，这样每个栈帧就能串联起来。

函数调用的时候，准备执行环境，先创建外层栈帧对象，这样当**调用者调用函数时候**创建会内部栈帧，随着调用函数调用的增长，外部栈帧的`back`和内部栈帧的`previous`会各自保存上一个栈帧，这样就行成了一条调用链，函数执行完毕后，根据这两个属性找到它们的调用者，将结果返回给调用者，这样就行成了代码的执行过程。
```python
def add(a, b):
    return a + b
print(add(1, 1))  
```
当前栈帧是函数`add`，它的`back`属性指向了`print`，所以当它执行出结果后，会寻着`back`返回给`print`。

### 生成器源码
#### nclude/cpython/genobject.h

```cpp
#define _PyGenObject_HEAD(prefix)
    PyObject_HEAD
    PyCodeObject *prefix##_code;  // 代码对象
    PyObject *prefix##_qualname;  // 生成器名称
    _PyErr_StackItem prefix##_exc_state;  // 执行状态
    ......
```

#### bjects/genobject.c

```cpp
// 创建生成器
static PyObject *
gen_new_with_qualname(PyTypeObject *type, PyFrameObject *f,
                      PyObject *name, PyObject *qualname)
{
    PyCodeObject *code = f->f_frame->f_code;  // 从栈帧获取代码段
    PyGenObject *gen = PyObject_GC_NewVar(PyGenObject, type, size);  // 创建生成器对象
    if (gen == NULL) {
        Py_DECREF(f);
        return NULL;
    }
    gen->gi_frame_valid = 1;
    f->f_owns_frame = 0;
    f->f_frame = frame;
    frame->generator = (PyObject *) gen;  // 记录当前生成器
    gen->gi_code = PyFrame_GetCode(f);  // 获取代码块
    _PyObject_GC_TRACK(gen);  // 跟踪GC
    return (PyObject *)gen;
}
```

调用`next`函数时候，向下执行生成器函数。

```cpp
static PyObject *
gen_iternext(PyGenObject *gen)
{
    PyObject *result;
    if (gen_send_ex2(gen, NULL, &result, 0, 0) == PYGEN_RETURN) {
    ......}
    return result;
}
```

调用`send`方法给生成器函数传递数据，同时向下执行生成器函数。

```cpp
static PyObject *
gen_send(PyGenObject *gen, PyObject *arg)
{
    return gen_send_ex(gen, arg, 0, 0);
}

static PyObject *
gen_send_ex(PyGenObject *gen, PyObject *arg, int exc, int closing)
{
    PyObject *result;
    if (gen_send_ex2(gen, arg, &result, exc, closing) == PYGEN_RETURN) {
    ......
    }
    return result;
}
```

在生成器函数向下执行的时候，无论是`next`还是`send`都调用了`gen_send_ex2()`这个函数，只不过在`send`的时候传递了参数（`gen_send_ex2`的第二个参数）。

```cpp
static PySendResult
gen_send_ex2(PyGenObject *gen, PyObject *arg, PyObject **presult,
             int exc, int closing)
{
    PyThreadState *tstate = _PyThreadState_GET();
    InterpreterFrame *frame = (InterpreterFrame *)gen->gi_iframe;  // 内部栈帧，它一直被复用。
    PyObject *result;

    *presult = NULL;

    // f_lasti < 0代表第一次调用生成器。此时，如果传递了参数，并且参数不是None将引发异常。
    也就是说第一次调用生成器如果用的是send(arg)，并且arg不是None，将引发该异常。
    因此实际使用中，我们会让所有生成器函数生成后自动调用一次next()，一般称为预激生成器。
    if (frame->f_lasti < 0 && arg && arg != Py_None) {
        const char *msg = "can't send non-None value to a "
                            "just-started generator";
                            .......
        return PYGEN_ERROR;
    }
    // 判断生成器是否已经执行。
    ......
    // 生成器执行完成的异常判断
    ......

    // 推送arg的值到执行帧栈上，这个值由send函数发送。
    result = arg ? arg : Py_None;
    Py_INCREF(result);
    _PyFrame_StackPush(frame, result);

    // 从此处开始往后的是核心代码
    // 把当前的的执行帧栈变成前一个（可以类比链表指针）
    // （还记得内部栈帧中previous属性的作用吗？）
    frame->previous = tstate->cframe->current_frame;
    // 记录当前执行状态 也就是保存当前执行到了哪里  previous_item保存了当前执行信息
    gen->gi_exc_state.previous_item = tstate->exc_info;
    // exc_info记录当前执行状态
    tstate->exc_info = &gen->gi_exc_state;
    ......
    // 执行字节码
    result = _PyEval_EvalFrame(tstate, frame, exc);
    // 从previous_item中恢复出当前的执行信息，继续从上一个位置执行。
    tstate->exc_info = gen->gi_exc_state.previous_item;
    gen->gi_exc_state.previous_item = NULL;
    // 流程结束返回执行结果
    *presult = result;
    return result ? PYGEN_RETURN : PYGEN_ERROR;
}
```
生成器为什么能在从上次停止的地方继续执行呢？是因为在停止的时候，执行权切换给调用者，此时生成器中**把当前栈帧变成了前一个栈帧**，也就是当前栈帧赋值给了内部栈帧的previous。而在本节**虚拟机栈帧**部分，我们已经知道了调用者进行值传递的时候会把值给到上一个栈帧。而上一个栈帧其实就保存了当前栈帧的所有信息，代码对象执行进度等。所以它生成器又可以从停止的地方继续执行。

#### 生成器的暂停
在上面的字节码中，我们可以看到一个`yield`相关的字节码**YIELD_VALUE**，该字节码负责了暂停的工作。所有字节码的执行都在`Python/ceval.c`中，在该文件的`_PyEval_EvalFrameDefault`函数中可以找到所有的字节码。
```cpp
// _PyEval_EvalFrameDefault匹配字节码的函数

// 在这里就能找到我们在上面看到的YIELD_VALUE
TARGET(YIELD_VALUE) {
    assert(frame->depth == 0);
    PyObject *retval = POP();  // 代码1
    ......
    frame->f_state = FRAME_SUSPENDED;
    _PyFrame_SetStackPointer(frame, stack_pointer);
    _Py_LeaveRecursiveCall(tstate);  // 代码2
    /* 代码3 */
    tstate->cframe = cframe.previous;
    tstate->cframe->use_tracing = cframe.use_tracing;
    return retval;
}
```
1. 代码1：在执行`YIELD_VALUE`之前已经执行了一条`LOAD_FAST`，而这个字节码把`yield <value>`中的常量`value`加载到了栈顶。所以第一行代码`PyObject *retval = POP();`就是加载出这个`value`作为返回值。
2. 代码2：当前栈帧离开调用链。
3. 代码3：切换执行权，执行权恢复上一个栈帧（也就是它的调用者）。

生成器恢复的时候会继续执行下一条字节码`STORE_FAST`，该字节码就从栈顶获取到了`send`发来的数据。

### 生成器源码总结

在刚才的源码中可以分析出，生成器的核心执行流程，**它复用生成器函数的帧栈，也就是源码中的 frame，当需要执行字节码的时候，它会把当前的执行信息保存在 previous_item 中，当字节码执行完之后，继续回到生成器函数的时候，会从 previous_item 中把执行信息拿出来，这样就可以保证继续从 yield 处继续执行。**

# 生成器超进化-->委托生成器

## 生成器的局限性

在上一小节我们了解到了`yield`可以让生成器与调用者交互，但是它的局限性在于**如果是多个生成器之间互动，会变得非常麻烦**。你可能需要处理很多的异常，此外`yield`虽然可以用`next`将值给到调用者，但是**调用者想拿到最后的 return 中的值，必须要捕获`StopIteration`异常，然后从异常中取得 return 的值**。

```python
def chain(*iterables):
    for it in iterables:
        for i in it:
            yield i
    return 'ok'

s = 'abc'
b = tuple(range(3))
instance = chain(s, b)
for _ in range(6):
    next(instance)
try:
    print(next(instance))
except StopIteration as e:
    print(e)  # ok
```

可以看到，我们为了获取返回值“ok”，就要处理一个`StopIteration`异常。并且为了处理两层嵌套的结构，我们依旧使用了两层 for 循环，看起来 yield 并没有起到什么特殊的效果（它与 return 作用差不多）。

## 委托生成器

为了解决`yield`的这些问题，在 PEP380 中提出的委托生成器`yield from <表达式>`这个语法，也是作为增强版的`yield` 出现，先来简单的看一下它的作用。在上面的`chain`函数中，使用两层循环才完成，现在使用`yield from`仅需要一层循环。

```python
def chain(*iterables):
    for it in iterables:
        yield from it
    return 'ok'
instance = chain(s, b)
print(list(instance))  # ['a', 'b', 'c', 0, 1, 2]
```

### 关键的伪代码

当然它的作用肯定不仅仅只是增强循环功能，否则也不会被引入为新的特性。PEP380 中用一段**伪代码**解释了**委托生成器的执行流程**，我们可以从执行流程中看到，其实`yield from`的作用体现在两方面：一是调用者不需要考虑异常的处理，而是可以直接传递给生成器即可，二是调用者不用再从`StopIteration`中获取到 return 的返回值，而是可以直接从委托生成器的返回值接受。

```python
_i = iter(EXPR)  # 伪代码1 将传进来的表达式变成迭代器 _i
try:
    _y = next(_i)  # 伪代码2 next()调用迭代器 预激
except StopIteration as _e:
    _r = _e.value  # 到StopIteration时候把return的值给_r。
else:  # 当获取到值 _y 或者 _r 之后执行else里面的逻辑
    while 1:  # 注意这里是个死循环
        try:
            # 循环第一步就是用yield产出这个值
            #  并且又接收了调用者传过来的值 _s
            _s = yield _y  # 伪代码3
        except GeneratorExit as _e:
        except BaseException as _e:
            # ...... 处理了两个异常。
        else:  # 在异常之后又走到了这个else
            try:
            # 注意这个_s是我们接到的调用者传过来的值，用来继续迭代
            # 如果它是None，则调用next(),否则调用send()将它发送给_i
                if _s is None:
                    _y = next(_i) # 伪代码4
                else:
                    _y = _i.send(_s) # 伪代码5
            except StopIteration as _e:
                _r = _e.value
                break
RESULT = _r  # _r 就是最后return中的值 也是yield form的返回值
```

总结一下这段委托生成器的作用：

> \_i: 生成器函数;  
> \_y: 生成器产出的值;  
> \_r: 最终结果，生成器 return 的值。  
> \_s: 调用者发送的值；  
> \_e: 产生的异常;

1. 对于委托生成器左面的表达式，先变成迭代器，然后预激迭代器，产出第一个值\_y，如果没有可产出值，则直接把 return 的返回值给\_r；
2. 接下来是一个死循环。循环第一步，用 yield 把\_y 返回给调用者，并接收调用者传过来的值\_s；
3. 接到\_s 之后，根据\_s 是否为 None，调用 next()或者 send(\_s)让迭代器\_i 继续产出值\_y，直到\_i 迭代完成(遇到 StopIteration)退出死循环，返回 return 的值。

分析完这段代码之后，可以发现委托生成器的作用远远不止增强嵌套迭代的处理，而是在生成器和调用者之间打开了一条通道，把最外层的调用者和内层生成器联系起来。

### 实际的例子

引用《流畅的python》中的一个实际例子，来说明委托生成器的工作流程。

该例子**计算一些学生的身高和体重的平均值，生成一个数据报告**。

```python
# 测试数据集
data = {
    'girls;kg': [40.9, 38.5, 44.3, 42.2, 45.2, 41.7, 44.5, 38.0, 40.6, 44.5],
    'girls;m': [1.6, 1.51, 1.4, 1.3, 1.41, 1.39, 1.33, 1.46, 1.45, 1.43],
    'boy;kg': [39.0, 40.8, 43.2, 40.8, 43.1, 38.6, 41.4, 40.6, 36.3],
    'boy;m': [1.38, 1.5, 1.32, 1.25, 1.37, 1.48, 1.25, 1.49, 1.46]
}

Result = namedtuple('Result', 'count average')
def averager():
    """计算平均值函数  生成器"""
    total = 0.0
    cnt = 0
    average = None
    while True:
        term = yield average  # 代码g1
        if not term:
            break
        total += term
        cnt += 1
        average = total / cnt  # 代码g2
    return Result(cnt, average)

def grouper(results, key):
    """委托生成器"""
    while True:
        results[key] = yield from averager()

def report(results):
    """报告产出函数"""
    for key, result in sorted(results.items()):
        group, unit = key.split(';')
        print(f'{result.count} {group} averaging {result.average} {unit}')

def main(data):
    """主要调用者函数"""
    results = {}
    for key, vals in data.items():  # 代码0 外循环
        group = grouper(results, key)  # 代码1
        next(group)  # 代码2
        for val in vals:  # 内循环
            group.send(val)  # 代码3
        group.send(None)  # 代码4
    print(results)
    report(results)

# 执行结果
{'girls;kg': Result(count=10, average=42.040000000000006), 'girls;m': Result(count=10, average=1.4279999999999997), 'boy;kg': Result(count=9, average=40.422222222222224), 'boy;m': Result(count=9, average=1.3888888888888888)}
9 boy averaging 40.422222222222224 kg
9 boy averaging 1.3888888888888888 m
10 girls averaging 42.040000000000006 kg
10 girls averaging 1.4279999999999997 m
```

为了弄懂委托生成器到底是怎么执行的，我们把这个例子代入到委托生成器的执行流程的伪代码中(后面用**伪代码**代指)。

无论是生成器还是委托生成器，都是从调用者调用之后才开始执行的，所以先看**调用者函数**。

1. 代码 1，**初始化委托生成器**，传入了两个参数。细心观察，发现这两个参数对函数主体并无影响，仅仅用做接收委托生成器的结果，而我们知道委托生成器的执行结果，在于右面表达式最终的返回值(return)；
2. 调用函数执行代码 2，预先激活委托生成器。**让出执行权给委托生成器**，对应伪代码中执行`_i=iter(averager())`(伪代码 1)，获取到平均数函数的生成器\_i。然后伪代码再执行`_y=next(_i)`(伪代码 2)；
3. **此时执行权切换到计算平均值函数(生成器)**，生成器函数执行到代码 g1，产出第一个`average`，并且**让出执行权**；
4. **委托生成器再次获取执行权**。伪代码 2 中\_y 变成了这个`average`(None)，继续执行`_s=yield _y`(伪代码 3)，将\_y 产出给调用者函数，并且**让出执行权**；
5. **调用者获取执行权**，执行到代码 3。代码 3 调用 send(val)函数，发送 val 到\_i，**让出执行权**；
6. **委托生成器获取到执行权**，伪代码 3 中的\_s 变成接收到的 val。继续执行因为\_s 有值，执行到伪代码 5，将值传递给生成器\_i，**并且让出执行权**。
7. **生成器获取执行权**，此时生成器\_i 接收到 val 值之后，执行到代码 g2，算出平均值`average`，并且通过死循环，执行到代码 g1，将`average`再通过`yield`从传递给委托生成器，并且**让出执行权**。
8. **委托生成器获取到执行权**，委托生成器中伪代码 5 的\_y 接收到了值，变成了`average`，委托生成器继续执行，通过死循环来到伪代码 3，伪代码 3 中的`yield`，将`average`产出给调用者函数，并且**让出执行权**。
9. **调用者获得执行权**，调用者继续执行 send(val)，也就是，重复执行过程 5~9。而**每次执行到步骤 7 都会更新`average`**。
10. 当最后调用者函数中内循环结束之后，调用 send(None)。**执行权来到委托生成器**，此时委托生成器接到了 None，执行`_y = next(_i)`(伪代码 4)，**执行权来到生成器**，此时生成器函数中代码 g1 的 term 参数就是 None，从而触发 break，跳出死循环，执行 return 语句。
11. 此时**执行权再次来到委托生成器**，因为`yield from <表达式>`的返回值就是右侧表达式的返回值，所以`results[key]`的值就是生成器最终的值。
12. **委托生成器赋值完成之后，执行权再次来到生成器，在代码 0 处执行外循环。重复执行步骤 1~11。直到外循环也执行完成，向下执行打印语句，打印出 results 的结果。**

这段代码分析中，我把**执行权切换**的过程用显示的标记了出来，所以会略显啰嗦。但是其实它并不复杂，你可以比较明显的发现委托生成器的作用，它就像一个**中间人**一样，在调用者和其他生成器之间进行协调。它把调用者的值给生成器，又把生成器的结果给调用者，在生成器执行结束之后，把 return 的返回值给调用者。在这些过程中调用者不必管任何的异常处理，并且不需要再从`StopIteration`获取返回值。

# 生成器究极进化-->协程的 await

如果大家已经了解协程的概念，会发现生成器本身就是一种协同程序。事实也确实如此，在 python 初期(2.5 版本)确实就把生成器当做协程使用，并为此产生了一个新的提案(PEP342)。

## 为生成器实现协程做的努力

在该提案中为生成器增加了一些功能，让生成器更加的符合协程的特性。

1. 将`yield`定义成表达式，而不是普通的语句。这期间的区别就是表达式可以有返回值，可以接收参数。
2. 增加一个函数`send(arg)`: 发送一个值到生成器作为参数。并返回生成器的下一个值。
3. 增加一个函数`throw()`: 允许调用者获取生成器的值，并在生成器停止时，引发异常。
4. 增加一个函`close()`: 该函数确保生成器可以正确停止。在停止时触发`GeneratorExit`或者`StopIteration`，如果停止时依旧产出值，触发`RuntimeError`。如果引发其他异常，则直接把这个异常给调用者处理。
5. 确保生成器在垃圾回收的时候正确调用上面的`close()`。
6. 因为可以显示的调用`close()`，所以也可以在`try/finally`语句中调用了(无法显示调用 close，就无法正确在 finally 中关闭协程)。

在知道 PEP342 中做过的这些更新之后，我们发现上一小节的例子**报告身高体重平均值**就是一个由生成器实现的协程。

## 生成器实现协程的缺点

生成器虽然可以实现协程，且又有委托生成器这种增强语法，但是它终究不是真正的协程。这种方法有一些缺点：

1. 协程和生成器混淆，他们使用相同的语法，尤其对新手很不友好。
2. 函数是否是协程取决于函数中是否有`yield`或者`yield from`，如果在重构的过程中弄丢了，会出现不明显的事故。
3. 异步调用的时候只支持了`yield`，而忽略了`for`和`with`等特性。

## 真正的协程

为了使协程成为 python 中一个独立的概念，使它更加易用，贴合协程这个概念，并且与生成器区分开，在 PEP492 中，提出了新的语法`async/await`，使用`async`来定义协程，并使用`await`来切换执行权，并且使用一个事件循环来不断的调度协程，在协程遇到 IO 操作的时候切换执行权，并在协程完成之后返回给调用者。

### async

我们都知道`def`关键字可以用来定义一个可调用对象，但是使用`async def`定义的确实一个协程，也就是在实例化之后，它不会被调用，而是转成一个**协程对象**，如下所示：

```python
async def coro():
    pass
# 使用async def 定义的是一个协程对象
print(coro())  # <coroutine object coro...>
```

### await/awaitable

而配合协程使用的是`await <awaitable object>`语句，该语句相较于`yield from <exec>`表达的含义更准确，并且在其他语言中也使用的是这个关键字。

`await`与`yield from`从**作用方面**看是**完全相同**的，但是`await`是为了配合协程使用的，因此与`yield from`不同的是`await`后面跟的表达式必须是**awaitable**对象。

而在该提案(PEP492)中提出了如下三个**awaitable**对象：

1. `coroutine`对象，也就是协程。当一个协程跟在`await`后面，并且`await`把执行权切换给它的时候，它就会执行。
2. `asyncio.Future`也是`abaitable`类型的对象，当它跟在`await`后面并获取执行权的时候，如果已经执行完毕，`await`立刻返回该结果(如果是异常，则立刻引发该异常)，如果没执行完，会持续等待当前 Future 执行完毕，并在执行完毕后返回结果(引发异常)。
3. 实现了魔术方法`__await__`的对象，当这种对象跟在`await`后面并获取执行权的时候，会执行`__await__`函数。

## 实际的例子

这里我想用一个实际的例子，来演示`await`在协程中的应用。在这个例子中，我们**启动一个微型服务器，并且通过常规方式和 async/await 的方式访问它，通过对比执行时间，来看他们的执行效果。**  
(Ps：这个例子只为了简单演示`await`和`yield from`在实际使用中的区别，**并非演示实际操作**，真实工程场景中的运用后续文章详述)

### 服务端

服务端的函数很简单，它的主要功能是接收一个参数`number`并将它原路返回。

```python
import asyncio
from aiohttp import web

async def handler(request):
    data = request.query
    await asyncio.sleep(1)  # 协程 模拟耗时IO
    return web.json_response(text=data['number'])

app = web.Application()
app.router.add_post('/', handler)
web.run_app(app)
```

主要思路就是启动一个支持异步的服务，并且给该服务设置一秒的延时，但是注意这里的延时是使用的`await asyncio.sleep(1)`这样可以让请求和延时操作之间来回切换。

也就是说当服务接到请求并执行到`await`之后，切换执行权到延迟操作，`sleep`会返回一个`await future`，该`future`需要 1s 执行时间，在这时间内如果服务接到了另外的请求，那么执行权就会切换，继续执行新的请求，就好像`sleep`的 1s 延迟叠加了起来一样。

### 客户端

朴实无华的同步请求。三个请求耗时 3s。

```python
start = time.time()
for i in range(3):
    res = requests.request(
        'POST',
        'http://127.0.0.1:8080',
        params={'number': i}
    )
    print(res.text)  # 0， 1， 2
print(time.time() - start)  # 3.0330724716186523
```

使用了`async/await`的异步请求。三个请求耗时 1s。

```python
import aiohttp
import asyncio

async def fetch(client, number):
    async with client.request('POST', 'http://127.0.0.1:8080', params={'number': number}) as resp:
        return await resp.text()
    
    
async def main():
    tasks = []
    async with aiohttp.ClientSession() as client:
        for i in range(3):
            tasks.append(fetch(client, i))
        done, pending = await asyncio.wait(tasks)
        for res in done:
            print(res.result())

start = time.time()
asyncio.run(main())  # 0, 1, 2
print(time.time() - start)  # 1.0144481658935547
```

在异步请求中，我们使用了一个异步的请求库`aiohttp`，该库实现了两个必要的魔术方法`__aenter__，__aexit__`，使它变成了`awaitable`对象。上面的代码将多个请求任务封装到一个事件循环去执行。

# 总结

本文主要目的是引导读者熟悉与协程相关的重要知识**生成器**。分四个小节，介绍了由生成器到协程的演进过程。


**完结**
