# 模拟可调用对象

如果类有称作 `__call__`的`#[pymethod]`，那么它是可调用的。这允许类的实例表现得像函数一样。

这个方法的签名必须是`__call__(<self>, ...) -> object`


## 例子：实现一个可调用的计数器

下面的 pyclass 是一个基本的装饰器（构造器将一个Python对象作为参数，当被调用时调用该对象），本节最后有一个等价的Python实现的链接。

[这里](https://github.com/PyO3/pyo3/tree/main/examples/decorator)可以看到一个包含这个 pyclass 的示范包。


```rust
#include ../../../examples/decorator/src/lib.rs
```

Python code:

```python
#include ../../../examples/decorator/tests/example.py
```

Output:

```text
say_hello has been called 1 time(s).
hello
say_hello has been called 2 time(s).
hello
say_hello has been called 3 time(s).
hello
say_hello has been called 4 time(s).
hello
```

## 纯Python实现

一个和Rust版本类似的Python实现

```python
class Counter:
    def __init__(self, wraps):
        self.count = 0
        self.wraps = wraps

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"{self.wraps.__name__} has been called {self.count} time(s)")
        self.wraps(*args, **kwargs)
```

注意到它也可以用高阶函数实现

```python
def Counter(wraps):
    count = 0
    def call(*args, **kwargs):
        nonlocal count
        count += 1
        print(f"{wraps.__name__} has been called {count} time(s)")
        return wraps(*args, **kwargs)
    return call
```

## `Cell` 什么？

[previous implementation](https://github.com/PyO3/pyo3/discussions/2598)使用了`u64`，这表示它需要一个`&mut self`接收者（receiver）来更新计数

```rust
#[pyo3(signature = (*args, **kwargs))]
fn __call__(&mut self, py: Python<'_>, args: &PyTuple, kwargs: Option<&PyDict>) -> PyResult<Py<PyAny>> {
    self.count += 1;
    let name = self.wraps.getattr(py, "__name__")?;

    println!("{} has been called {} time(s).", name, self.count);

    // After doing something, we finally forward the call to the wrapped function
    let ret = self.wraps.call(py, args, kwargs)?;

    // We could do something with the return value of
    // the function before returning it
    Ok(ret)
}
```

这里的问题是`&mut self`这样的接收者意味着PyO3必须唯一地借用它，并在`self.wraps.call(py, args, kwargs)`调用中保持该借用。这个调用会将控制权返回给一个可以任意调用的Python代码，包括装饰器函数。如果这个发生了，PyO3就不能创建第二个唯一的借用并会抛出异常。


```py
@Counter
def say_hello():
    if say_hello.count < 2:
        print(f"hello from decorator")

say_hello()
# RuntimeError: Already borrowed
```

本章给出的实现通过永远不做唯一的借用避免了这个错误，所有方法将`&self`作为接受者，all the methods take `&self` as receivers, of which multiple may exist simultaneously，这需要一个共享的计数器，而最简单的方式就是这里所使用的[`Cell`](https://doc.rust-lang.org/std/cell/struct.Cell.html)。

这显示了运行任意Python代码的危险，
- Python 的异步执行器（asynchronous executor）可能在Python代码的中途挂起当前线程，即使是你在控制的Python代码，而运行其他Python代码
- 丢弃任意的Python对象可能会引起（invoke）在Python中通过`__del__` methods定义的自毁器（destructor）
- 调用Python的 C-API（绝大多数PyO3 api在内部调用了 C-API）可能会抛出异常，可能让信号处理器（signal handler）中的Python代码运行

所以在写 unsafe 的代码时要格外慎重
