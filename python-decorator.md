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
    
4. 常见场景
    #1 网站的授权检查，相当于中间件功能
    #2 日志功能
    
5. 可以用类实现，类里可以实现更丰富的功能，而且可以通过继承扩展

from functools import wraps

class logit(object):
    def __init__(self, logfile='out.log'):
        self.logfile = logfile

    def __call__(self, func):
        @wraps(func)
        def wrapped_function(*args, **kwargs):
            log_string = func.__name__ + " was called"
            print(log_string)
            # 打开logfile并写入
            with open(self.logfile, 'a') as opened_file:
                # 现在将日志打到指定的文件
                opened_file.write(log_string + '\n')
            # 现在，发送一个通知
            self.notify()
            return func(*args, **kwargs)
        return wrapped_function

    def notify(self):
        # logit只打日志，不做别的
        pass
```



