## python装饰器

```py
1. 修改其他函数功能的函数。可以在不改变函数定义的同时，给函数加功能。OOP里面，可以通过继承时override父函数来达到
同样效果

2. 函数也是对象，所以可以将函数赋值给变量，通过变量来调用函数。函数中可以定义函数，也可以返回函数，函数可作为参数传递

3. 定义好的decorator函数，可以通过@来使用

def log(para="def"):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print "%s call %s" % (para,func.__name__)
            return func(*args, **kwargs)

        return wrapper
    return decorator


@log()
def test_decorator():
    print "normal function"
```



