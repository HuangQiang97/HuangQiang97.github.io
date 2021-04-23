# Python



[toc]

参考： [廖雪峰][GeneralLink]

## 1:面向对象

```python
# class 定义
class My_class(object):
    # 类属性，所有实例共有,优先级低于实例属性,只能通过My_class.count修改，修改后该类的其它实力和子类都会同步变化。
    # stu.count=1只是再stu下新建一个高优先级的count=1的属性，对My_class.count无影响。
    # del stu.count 后My_class.count==stu.count
    # 实例属性属于各个实例所有，互不干扰；
    # 类属性属于类所有，所有实例共享一个属性；
    # 不要对实例属性和类属性使用相同的名字，否则将产生难以发现的错误。
    count = -1

    # 特殊的__slots__变量，来限制该class实例能添加的属性,
    # 注意私有属性的写法，class方法不必写出，但通过类实例加入方法要写入,
    # 且slots会对类属性设为只读，并且slots对继承的子类是不起作用的.
    # __slots__ = ('__name', '__age','__score','args','kw','gender','set_gender','param')

    # 初始化方法
    def __init__(self, name, score, age=21, *args, **kwargs):
        # 实例属性
        self.__name = name
        self.__score = score
        self.__age = age
        self.args = args
        self.kw = kwargs

        self.index = 0

    # get/set访问私有变量。
    def set(self, name, score, age):
        self.__name = name
        self.__score = score
        self.__age = age

    def get(self):
        return [self.__name, self.__score, self.__age]

    def print_score(self):
        print('name:%s,score:%s,age:%s,args:%s,kwargs:%s' % (self.__name, self.__score, self.__age, self.args, self.kw))

    # 简化get/set ,可通过stu.name实现访问/修改__name功能。
    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, name):
        self.__name = name

    # 只定义get，__age就变为只读属性。
    @property
    def age(self):
        return self.__age

    # 实现len(stu)
    def __len__(self):
        return len(self.args)

    # toString
    def __str__(self):
        return 'name:%s,score:%s,age:%s,args:%s,kwargs:%s' % (self.__name, self.__score, self.__age, self.args, self.kw)

    __repr__ = __str__

    # 实现遍历就必须实现一个__iter__()方法，该方法返回一个迭代对象，
    # 然后，Python的for循环就会不断调用该迭代对象的__next__()方法拿到循环的下一个值，
    # 直到遇到StopIteration错误时退出循环。
    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.args[0]):
            raise StopIteration
        else:
            self.index = self.index + 1
            return self.args[0][self.index - 1]

    # 下标访问
    def __getitem__(self, index):
        if isinstance(index, slice):
            start = index.start
            stop = index.stop
            step = index.step
            return self.args[0][start:step:stop]
        else:
            return self.args[0][index]

    # 当调用不存在的属性时，Python解释器会试图调用__getattr__(self, name)来尝试获得属性，
    # 只有在没有找到属性的情况下，才调用__getattr__，已有的属性，不会在__getattr__中查找。
    def __getattr__(self, name):
        if name == 'weight':
            return 68

    # 可以实现针对完全动态的情况作调用，如网站地址。
    # class Chain(object):
    #
    #     def __init__(self, path=''):
    #         self._path = path
    #
    #     def __getattr__(self, path):
    #         return Chain('%s/%s' % (self._path, path))
    #   >>Chain().status.user.timeline.list
    #       '/status/user/timeline/list'

    # 实现直接对实例进行调用stu(args)
    def __call__(self, *args, **kwargs):
        print('i\'m callable,%s,%s' % (args, kwargs))

    def saySomething(self):
        print('i\'m super class')


# 创建实例
stu = My_class('huang', 88, 33, [0, 1, 2, 3, 4, 5, 6, 7, 8, 9], key_0='vale_0', key_1='vale_1')
stu.print_score()

# 对实例变量绑定任何数据
stu.gender = 'male'
print(stu.gender)


# 对实例绑定方法，只对该实例起作用
def set_gender(self, gender):
    self.gender = gender


from types import MethodType

stu.set_gender = MethodType(set_gender, stu)
stu.set_gender('female')
print(stu.gender)

# 对类绑定方法，对该类所有实例起作用
My_class.set_gender = set_gender
foo = My_class('huang', 88, 33, [1, 2, 3], key_0='vale_0', key_1='vale_1')
foo.set_gender('female')
print(foo.gender)

# 访问修改私有属性
print(stu.get())
stu.set('hu', 98, 20)
print(stu.get())


# stu.__name 无法访问,但可通过 _classname__propertyname 强制访问私有变量，不建议这么做。
# stu._My_class__name='qiang'
# print(stu._My_class__name)

# 子类
class Subclass(My_class):

    # 复写父类方法
    def saySomething(self):

        # 子类调用父类方法
        # super().saySomething

        print('i\'m subclass')


sub_stu = Subclass('huang', 88, 33, [1, 2, 3], key_0='vale_0', key_1='vale_1')
sub_stu.print_score()
stu.saySomething()
sub_stu.saySomething()

# 实例与继承类关系
print(isinstance(stu, My_class),
      isinstance(sub_stu, My_class),
      isinstance(stu, Subclass)
      )


# 有saysomething功能的任何类
class Other_class(object):
    def saySomething(self):
        print('i\'m other class')


others = Other_class()


# 继承与多态，不管实现只管调用。
def say_twice(stu):
    stu.saySomething()


say_twice(stu)
say_twice(sub_stu)
# 鸭子类型，不管他是否是鸭子，他的行为像鸭子就行。
say_twice(others)

import types

# 实例类型判断
print(type(123) == type(456),
      type(123) == int,
      type('abc') == type('123'),
      type('abc') == str,
      type('abc') == type(123))

print(
    type(say_twice) == types.FunctionType,
    type(abs) == types.BuiltinFunctionType,
    type(lambda x: x) == types.LambdaType,
    type((x for x in range(10))) == types.GeneratorType,
)

print(
    isinstance('a', str),
    isinstance(123, int),
    isinstance(b'a', bytes)
)

# 判断[1,2,3]是否是list或者 tuple
print(isinstance([1, 2, 3], (list, tuple)))

# 获得stu所有的属性与方法
print(dir(stu))

# 造作对象属性,只有在不知道对象信息的时候，我们才会去获取对象信息,优先使用object.property操作属性。
print(hasattr(stu, 'kw'), hasattr(stu, 'get'))

# none为可选参数，如果没有 kw属性才返回None
print(getattr(stu, 'kw', None))

# 获得函数，print_stu()==print_score()
print_stu = getattr(stu, 'print_score')
print_stu()

# 设置属性，等价于stu.param=20
setattr(stu, 'param', 20)
print(stu.param)

other_stu = My_class('huang', 88, 33, [1, 2, 3], key_0='vale_0', key_1='vale_1')

print(My_class.count, stu.count, sub_stu.count, other_stu.count)

stu.count = 1
# 只有stu.count 变化
print(My_class.count, stu.count, sub_stu.count, other_stu.count)

My_class.count = 2
# My_class 其它实例和子类都同步变化
print(My_class.count, stu.count, sub_stu.count, other_stu.count)

del stu.count
# stu.count值变回my_class.count值
print(My_class.count, stu.count, sub_stu.count, other_stu.count)

# 简化get/set实现读写
print(stu.name)
stu.name = 'huangqiang'
print(stu.get())

# 简化get实现只读
print(stu.age)


# stu.age=99999
# print(stu.get())

# 多重继承
class Anaimal(object):
    def eat(self):
        print('i can eat')


class Flyable(object):
    def fly(self):
        print('i can fly')


class Runable(object):
    def run(self):
        print('i can run')


class Dog(Anaimal, Runable):
    pass


dog = Dog()
dog.eat()
dog.run()

# 定制类
# 获得长度
print(len(stu))

# toString
print(stu)

# 遍历对象
for i in stu:
    print(i)

# 下标访问
print(stu[3])
print(stu[1:3:9])

# 动态属性
print(stu.weight)

# 直接调用对象
stu([1, 2, 3, 4], key='value')
print(callable(stu), callable(dog))

from enum import Enum

# Month类型的枚举类，可以直接使用Month.Jan来引用一个常量，或者枚举它的所有成员
# value属性则是自动赋给成员的int常量，默认从1开始计数.
Month = Enum('Mon', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

print(Month(1), Month.Jan, Month.Jan.value)

for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)

from enum import Enum, unique


# unique保证value唯一
@unique
class Weekday(Enum):
    Sun = 0  # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6


print(Weekday(0), Weekday.Sun, Weekday.Sun.name, Weekday.Sun.value)
for name, member in Weekday.__members__.items():
    print(name, '=>', member, ',', member.value)

```

## 2:异常处理，代码测试

```python
# 当我们认为某些代码可能会出错时，就可以用try来运行这段代码，
# 如果执行出错，则后续代码不会继续执行，而是直接跳转至错误处理代码，即except语句块，
# 执行完except后，如果有finally语句块，则执行finally语句块，至此，执行完毕。

try:
    re = 10 / int('a')
    # re=10/int('23')
    print('runing')

except ValueError as e:
    print('value error', e)

# Python的错误其实也是class，所有的错误类型都继承自BaseException，
# exception 间有继承关系，子类要先写在父类异常前面，否则异常会被父类捕捉，子类无法捕捉到异常
# 如 UnicodeError父类为ValueError,这里UnicodeError无法捕捉到异常。
except UnicodeError as e:
    print('UnicodeError')

except ZeroDivisionError as e:
    print('ZeroDivisionError', e)

# 如果没有错误发生，可以在except语句块后面加一个else，当没有错误发生时，会自动执行else语句
else:
    print('no error')

finally:
    print('finish')

print('do something')

# 使用try...except捕获错误还有一个巨大的好处，就是可以跨越多层调用，即不断上报错误。
# 比如函数main()调用foo()，foo()调用bar()，结果bar()出错了，这时，只要main()捕获到了，就可以处理。
# 不需要在每个可能出错的地方去捕获错误，只要在合适的层次去捕获错误就可以了。

def bar(parm):
    return 10 / int(parm)

def foo():
    try:
        print(bar('a'))
    except BaseException as e:
        print(e)

foo()

# 如果错误没有被捕获，它就会一直往上抛，最后被Python解释器捕获，
# 打印一个错误信息，错误信息为从顶层到底层，然后程序退出.

# 可以用log把错误堆栈打印出来，(log信息与普通print信息会混叠，并不是按程序执行顺序打印)
# 然后分析错误原因，同时，让程序继续执行下去。

import logging

def fun_0(parm):
    if parm == 0:
        # 抛出错误
        raise ValueError('fuck msg')
    return 10 / parm

def fun_1():
    try:
        fun_0(0)
    except Exception as e:

        # 写log文件,level :过滤日志级别，只有达到该级别及以上的才会记录，
        # a:append，w:rewrite,message:由下面的logging.exception/debag/info/...('msg')决定
        logging.basicConfig(level=logging.DEBUG, filename='error.log', filemode='w',
                            format='%(asctime)s -> %(pathname)s->%(filename)s->[line:%(lineno)d] -> %(levelname)s: %(message)s')

        # 加入logging.basicConfig写信息到文件，不加打印到控制台
        # debug->critical严重因依次递增，
        # 日常记录信息
        logging.debug('debug msg')
        logging.info('info msg')
        # 错误信息
        logging.warning('waring msg')
        logging.error('error msg')
        logging.critical('critical msg')

        # 异常栈信息
        logging.exception(e)

        # 记录日志后，可以将异常原样继续上抛。
        # raise
        # 或者把错误细化后上抛。
        # raise ZeroDivisionError

    print('------finish')

fun_1()

# 单元测试
# 说明我们测试的这个函数能够正常工作。如果单元测试不通过，要么函数有bug，要么测试条件输入不正确，总之，需要修复使单元测试能够通过。
# 如果我们对abs()函数代码做了修改，只需要再跑一遍单元测试，如果通过，说明我们的修改不会对abs()函数原有的行为造成影响,
# 如果测试不通过，说明我们的修改与原有行为不一致，要么修改代码，要么修改测试。
# 这种以测试为驱动的开发模式最大的好处就是确保一个程序模块的行为符合我们设计的测试用例。在将来修改的时候，可以极大程度地保证该模块行为仍然是正确的。

class My_dict(dict):

    def __init__(self, **kwargs):
        super().__init__(kwargs)

    # A.a式访问
    def __getattr__(self, item):
        try:
            return self[item]
        except KeyError as e:
            raise

    # A.a式设置值
    def __setattr__(self, key, value):
        self[key] = value

import unittest

# 编写一个测试类，从unittest.TestCase继承,
# 以test开头的方法就是测试方法，不以test开头的方法不被认为是测试方法，测试的时候不会被执行

class TestDict(unittest.TestCase):

    # 可以在单元测试中编写两个特殊的setUp()和tearDown()方法。
    # 这两个方法会分别在每调用一个测试方法的前后分别被执行。
    # 可用于做前期准备或者收尾工作
    def setUp(self):
        print('up')

    def tearDown(self):
        print('down')

    def test_init(self):
        # test init
        my_dict = My_dict(A='a', B='b')
        self.assertEqual(my_dict.A, 'a')
        self.assertEqual(my_dict['A'], 'a')
        self.assertTrue(isinstance(my_dict, dict))
        # test attr
        my_dict.C = 'c'
        self.assertEqual(my_dict.C, 'c')
        # test key
        my_dict['D'] = 'd'
        self.assertEqual(my_dict['D'], 'd')

    #异常处理测试
    def test_errors(self):
        my_dict = My_dict(A='a')
        with self.assertRaises(KeyError):
            value = my_dict.B

# 运行单元测试。
if __name__ == '__main__':
    unittest.main()

# 另一种方法是在命令行通过参数-m unittest直接运行单元测试：
# $ python -m unittest mydict_test

```

```python
# 文档测试
# Python内置的“文档测试”（doctest）模块可以直接提取注释中的代码并执行测试。
# doctest严格按照Python交互式命令行的输入和输出来判断测试结果是否正确。
# 只有测试异常的时候，可以用...表示中间一大段烦人的输出
#注意命令行与输出间不能有空行，>>>符号后要加一个空格

class My_dict_Doctest(dict):
    '''

    >>> my_dict = My_dict_Doctest(A='a', B='b')

    >>> my_dict.A
    'a'
    >>> my_dict['A']
    'a'

    >>> isinstance(my_dict, dict)
    True

    >>> my_dict.C = 'c'
    >>> my_dict.C
    'c'

    >>> my_dict['D'] = 'd'
    >>> my_dict['D']
    'd'

    >>> my_dict['E']
    Traceback (most recent call last):
      ...
    KeyError: 'E'

    '''

    def __init__(self,**kwargs):
        super().__init__(kwargs)

    #A.a式访问
    def __getattr__(self, item):
        try:
            return self[item]
        except KeyError as e:
            raise

     #A.a式设置值
    def __setattr__(self, key, value):
        self[key]=value

def my_power(x,n):
    '''
    :param x:
    :param n:
    :return:x^n

    >>> my_power(2,3)
    8

    >>> my_power(2,-1)
    0.5

    >>> my_power(2,'a')
    Traceback (most recent call last):
      ...
    ValueError

    '''

    try:
        return x**n
    except Exception as e:
        raise ValueError

if __name__=='__main__':
    import doctest
    doctest.testmod()

```

## 3:IO

```python
#读写文件就是请求操作系统打开一个文件对象（通常称为文件描述符），
# 然后，通过操作系统提供的接口从这个文件对象中读取数据（读文件），或者把数据写入这个文件对象（写文件）。

try:
    #如果文件不存在，open()函数就会抛出一个IOError的错误
    #相对路径为当前py文件路径
    file=open('error.log','r')
    #read()方法可以一次读取文件的全部内容，
    # Python把内容读到内存，用一个str对象表示：
    text=file.read()
    print(type(text))
    print(text)
except IOError as e:
    print(e)
finally:
    #文件使用完毕后必须关闭
    if file:
        file.close()

#简化写法,自动执行close
#读取文本文件，并且是UTF-8编码的文本文件
with open('error.log','r') as file:
    # read()会一次性读取文件的全部内容
    # read(size)方法，每次最多读取size个字节的内容。
    # 调用readline()可以每次读取一行内容，
    # 调用readlines()一次读取所有内容并按行返回list
    #再同一个文件下以上read/write方法共享一个文件指针，当前read方法从前一个read方法的结束指针处开始读文件，
    #当text=='' 时表示文件已经读完。
    text = file.read()
    print(type(text))
    print(text)

    text = file.read(1024)
    print(type(text))
    print(text)

    text = file.readline()
    print(type(text))
    print(text)

    text = file.readlines()
    print(type(text))
    print(text)

#open()函数返回的这种有个read()方法的对象,不要求从特定类继承，只要写个read()方法就行。

#要读取二进制文件，比如图片、视频等等,用'rb'模式
#要读取非UTF-8编码的文本文件，需要给open()函数传入encoding参数
#在文本文件中可能夹杂了一些非法编码的字符。遇到这种情况，open()函数还接收一个errors参数，表示如果遇到编码错误后如何处理。

with open('pink.png','rb') as file:
    pass

with open('error.log','r',encoding='GBK',errors='ignore') as file:
    pass

#调用open()函数时，传入标识符要可写文件权限
#读写都会移动文件指针。获得指针位置：file.tell() ，设置指针位置：file.seek(0)
# r 只读文件,打开文件后指针位于0，文件不存在会报错。rb 二进制读文件。
# r+ 可读可写，打开文件后指针位于0，读为从当前指针出开始到结尾，写为从当前指针出开始写逐个字符覆盖原先内容，读写文件时不会创建不存在的文件，会报错.
# rb+ 二进制格式读写文件，打开文件后指针位于0，文件不存在会报错。

# w只写文件，打开文件后指针位于0，在打开文件后原先内容即被清空，从头开始写。如果该文件不存在，创建新文件。 wb+ 二进制读写文件。
# w+ 读写文件，打开文件后指针位于0，在打开文件后原先内容即被清空，从头开始写，读为从当前指针出开始到结尾。写文件时如果该文件不存在，创建新文件。

# a 只追写文件，打开文件后指针位于end,写为从end开始写.不存在则创建 ，ab+ 追读写二进制。
# a+ 读 追写文件。打开文件后指针位于end,写为从end开始写，读为从当前指针出开始到结尾， 读写文件时不存在则创建。



with open('error.log','a+') as file:
    print('--------------------------------')
    print(file.tell())
    file.seek(0)
    print(file.readlines())
    file.write('fuck\n')
    print(file.tell())

#当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。
# 只有调用close()/flush()方法时，操作系统才保证把没有写入的数据全部写入磁盘。
# 忘记调用close()的后果是数据可能只写了一部分到磁盘，剩下的丢失了。所以，还是用with语句来得保险：

print('--------------------------')
#，数据读写不一定是文件，也可以在内存中读写。
from io import StringIO ,BytesIO
#写str
f = StringIO('Hello!\nHi!\nGoodbye!')

#逐字覆盖式写入
#f.write('fuck')
while True:
    s = f.readline()
    if s == '':
        break
    print(s)
#写bytes
f = BytesIO('中文'.encode('utf-8'))

#逐字覆盖式写入
#f.write(bytes(22))
print(f.getvalue())

import os
#操作系统类型
print(os.name)
#环境变量
env_info=os.environ
print(type(env_info))
print(env_info.get('ALLUSERSPROFILE','default value'))

# 查看当前目录的绝对路径:
current_path=os.path.abspath('.')
print(current_path)

#把两个路径合成一个时，不要直接拼字符串，而要通过os.path.join()函数，这样可以正确处理不同操作系统的路径分隔符。
full_path=os.path.join(current_path, 'testdir')
print(type(full_path),full_path)
# 然后创建一个目录,若存在则报错
os.mkdir(full_path)
# 删掉一个目录:
os.rmdir(full_path)
# 对文件重命名:
os.rename('G:\\Python\\PycharmProject\\learn_python\\FileIO\\fuck.py', 'fuckme.py')
# 删掉文件:
os.remove('G:\\Python\\PycharmProject\\learn_python\\FileIO\\fuckme.py')

#把一个路径拆分为两部分，后一部分总是最后级别的目录或文件名，前一部分为剩余信息：
print(os.path.split(full_path))
#得到文件扩展名:('/path/to/file', '.txt')
print(os.path.splitext('/path/to/file.txt'))


import shutil

#复制文件，fsrc(可读)，fdst(可写）都是文件对象，都需要打开后才能进行复制操作
f1=open('error.log','r')
f2=open('name_copy.log','w+')
shutil.copyfileobj(f1,f2)

#复制文件，fsrcpath，fdstpath,只修改文件内容，不修改书写权限与文件状态。
shutil.copyfile('error.log','name_copy_2.log')

#仅copy 读写权限，不更改文件内容，组和用户。frspath,dstpath
#shutil.copymode('error.log','name_copy.log')

# 复制所有的状态信息，包括权限，组，用户，时间等
#shutil.copystat(srcpath,dstpath)

#复制文件的内容以及权限，先copyfile后copymode
#shutil.copy(srcpath,dstpath)
# 复制文件的内容以及文件的所有状态信息。先copyfile后copystat
#shutil.copy2(srcpath, dstpath)

#递归的复制文件夹内容及读写权限，文件状态
shutil.copytree('src', 'dst')
#递归地删除文件夹
shutil.rmtree('dst')

# 递归的移动文件夹/文件，目标文件夹/文件不存在就创建
shutil.move('src', 'dst')

# 压缩打包
#base_name：压缩打包后的文件名或者路径名,忽略压缩扩展名。
#format：压缩或者打包格式    "zip", "tar", "bztar"or "gztar"
#root_dir : 将哪个目录或者文件打包（也就是源文件）
shutil.make_archive('dst', 'zip', root_dir='dst')
#列出路径下所有文件夹与文件,返回list[str]
type(os.listdir('.'))
#判断是否是文件夹
print(os.path.isdir('dst'))
#判断是否是文件
print(os.path.isfile('dst.zip'))

#列出当前路径下所有.py文件
print([x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py'])

#把变量从内存中变成可存储或传输的过程称之为序列化，在Python中叫pickling
#序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。
#反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling。

#pickle.dumps()方法把任意对象序列化成一个bytes，然后，就可以把这个bytes写入文件。或者用另一个方法pickle.dump()直接把对象序列化后写入一个file-like Object：

import pickle
d=dict(A='a',B='b')
f = open('dump.txt', 'wb')
pickle.dump(d, f)
f.close()

import pickle

# pickle.dumps(object)方法把任意对象序列化成一个bytes
# pickle.dump(object,f)直接把对象序列化后写入一个file-like Object：
with open('dump.txt', 'wb') as f:
    d = dict(A='a', B='b')
    print(d)
    pickle.dump(d, f)

# pickle.loads(bytes)方法反序列化出对象
# 用pickle.load(f)方法从一个file-like Object中直接反序列化出对象。
with open('dump.txt', 'rb') as f:
    d = pickle.load(f)
    print(d)

# 这个变量和原来的变量是完全不相干的对象，它们只是内容相同而已。
# 可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据。

# 更好的方法是序列化为JSON，因为JSON表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输。
#   JSON类型	Python类型
#   {}	        dict
#   []	        list
#   "string"	str
#   1234.56	    int或float
#   true/false	True/False
#   null   	    None

# obj-(obj2dict())->dict-->str
# str-->dict-(dict2obj())->obj
# 这个变量和原来的变量是完全不相干的对象，它们只是内容相同而已。
# json.dumps(object)方法可以把一个可序列化对象返回一个str，内容就是标准的JSON。类似的，dump(object,file)方法可以直接把JSON写入一个file-like Object。
# 要把JSON反序列化为Python对象，用loads(str)或者对应的load(file)方法，前者把JSON的字符串反序列化，后者从file-like Object中读取字符串并反序列化：

import json

print('---------------------')
d = dict(A='a', B='b')
print(d)
dict_str = json.dumps(d)
print(dict_str)

d = json.loads(dict_str)
print(d)


# 非dict类序列化
class Stu(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

    def __str__(self):
        return 'name:%s,age:%s,score:%s' % (self.name, self.age, self.score)


# 对于非dict对象要定义一个函数将对象转化为一个dict,或者使用对象自带方法，object.__dict__方法(也有少数例外，比如定义了__slots__的class。)
def stu2dict(stu):
    return dict(name=stu.name, age=stu.age, score=stu.score)


stu = Stu('huang', 22, 88)
print(stu)

stu_str = json.dumps(stu, default=stu2dict)
print(stu_str)

stu_str = json.dumps(stu, default=lambda ob: ob.__dict__)
print(stu_str)


# 同样要定义一个方法完成dist->obj
def dict2stu(d):
    return Stu(d['name'], d['age'], d['score'])


stu = json.loads(stu_str, object_hook=dict2stu)
print(stu)
```

## 4:进程，线程

参考：[Process/Threading][Proc_Thre_Link]

```python
# -------------------------------多进程

# 
# Unix/Linux操作系统提供了一个fork()系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。
# 子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。
# windows不支持fork
# 有了fork调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务。

import multiprocessing
import os

# Only works on Unix/Linux/Mac:
# pid = os.fork()
# if pid == 0:
#     print('I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid()))
# else:
#     print('I (%s) just created a child process (%s).' % (os.getpid(), pid))


from multiprocessing import Process, Pool, Queue, Pipe
import time

#getpid()获得当前进程ID，getppid()获得父进程ID

def run_process(args):
    time.sleep(1)
    print('Ass we can!,parms:%s,super:%s,subs:%s'%(args,os.getppid(),os.getpid()))

if __name__=='__main__':
    print('start')
    print('super:%s'%(os.getpid()))
    #创建进程对象，target：子进程要处理的函数，args：函数参数。
    proc=Process(target=run_process,args=(12,))
    #启动子进程
    proc.start()
    #让父进程等待子进程执行完毕，父进程才继续执行,通常用于进程间的同步。
    proc.join()
    print('finish')

import random
# -------------------------------进程池

# 如果要启动大量的子进程，可以用进程池的方式批量创建子进程.
# 由于进程启动的开销比较大，使用多进程的时候会导致大量内存空间被消耗。
# 为了防止这种情况发生可以使用进程池，（由于启动线程的开销比较小，所以不需要线程池这种概念，
# 多线程只会频繁得切换cpu导致系统变慢，并不会占用过多的内存空间）
# 进程池中常用方法：
# `apply()` 同步执行（串行）,多个进程按启动顺序挨个执行，一个执行完再到下一个进程。
# `apply_async()` 异步执行（并行），启动多个进程同步执行，并行数目按Pool大小和CPU 线程数决定。
# `terminate()` 立刻关闭进程池
# `close()` 等待所有进程结束后，才关闭进程池。
# `join()` 主进程等待所有子进程执行完毕。必须在close或terminate()之后。

#示例1
def subs_proc(args):
    print('i\'m %s ,id"%s'%(args,os.getpid()))
    start=time.time()
    time.sleep(random.random()*3)
    print('%s:run time %s s'%(args,time.time()-start))

if __name__=='__main__':
    #线程池。默认大小为CPU线程数，本机为8
    pool=Pool(8)
    print('id:%s'%os.getpid())
    for i in range(10):
        #添加任务,如线程池满，要等待其中有线程结束，资源空闲才会获得资源开始执行。
        pool.apply_async(subs_proc,args=(i,))
    pool.close()
    #会等待所有子进程执行完毕，调用join()之前必须先调用close()/terminate()，调用close()/terminate()之后就不能继续添加新的Process了。
    pool.join()
    print('finish')
#示例1
def Foo(i):
    time.sleep(2)
    print(i)
    return i + 100

def Bar(arg):
    print('-->exec done:', arg)
# 允许进程池同时放入5个进程
#进程池内部维护一个进程序列，当使用时，去进程池中获取一个进程，
# 如果进程池序列中没有可供使用的进程，那么程序就会等待，直到进程池中有可用进程为止。
# 在上面的程序中产生了10个进程，但是只能有5同时被放入进程池，剩下的都被暂时挂起，并不占用内存空间，等前面的五个进程执行完后，再执行剩下5个进程。

if __name__=='__main__':
    pool = Pool(5)
    for i in range(10):
        #异步执行。
        # func子进程执行完后，才会执行callback，否则callback不执行（而且callback是由父进程来执行了）
        # Bar(Foo(i))
        pool.apply_async(func=Foo, args=(i,), callback=Bar)
        #pool.apply(func=Foo, args=(i,))
    print('end')
    pool.close()
    # 主进程等待所有子进程执行完毕。必须在close()或terminate()之后。
    pool.join()


#---------------------------------子进程
# 示例一
#很多时候，子进程并不是自身，而是一个外部进程。我们创建了子进程后，还需要控制子进程的输入和输出。
#import subprocess
#等价于 cmd输入：ping 127.0.0.1
# r = subprocess.call(['ping', '127.0.0.1'])
# print('Exit code:', r)
#实例2
# 如果子进程还需要输入，则可以通过communicate()方法输入
#等价于 cmd输入：my_cmd
#p = subprocess.Popen(['my_cmd'], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
#相当于在命令行执行命令my_cmd，然后手动输入：
# a=1
# b=2
# c=3
# output, err = p.communicate(b'a=1\nb=2\nc=3\n')
# print(output.decode('utf-8'))
# print('Exit code:', p.returncode)
#实例三
# p = subprocess.Popen('cmd.exe',shell=True, stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
# output, err = p.communicate("echo HELLW_WORLD".encode("GBK"))
# print(output.decode('GBK'))
# print(p.returncode)
#---------------------------------子进程间通信
def proc_write(queue):
    for i in ['A','B','C','D','E','F']:
        #写数据put(True,waittime)
        #True:表示在queue满下可以等待，waittime 若为None则一直等待，非None则在等待规定事件后还没有数据抛出Queue.Full
        #False:表示不等待，queue满就抛出Queue.Full
        try:
            queue.put(i,True,3)
            print('put %s to queue'%i)
            time.sleep(random.random()*2)
        except Exception as e:
            print(e)
            break

def proc_read(queue):
    while True:
        #读数据，get(True,waittime):
        # True表示queue空可以等待，waittime 若为None则一直等待，非None则在等待规定事件后还没有数据抛出Queue.Empty
        # False表示queue空不等待，没有数据就抛出Queue.Empty
        try:
            msg=queue.get(True,3)
            print('get %s from queue'%msg)
        except Exception as e:
            print(e)
            break

if __name__=='__main__':

    queue=Queue()
    pip=Pipe()
    #创建进程实例
    p_w=Process(target=proc_write,args=(queue,))
    p_r=Process(target=proc_read,args=(queue,))
    p_w.start()
    p_r.start()
    #等待write结束
    p_w.join()
    #强制结束read
    p_r.terminate()


    
 
#pipe
#创建子进程时候，就是把一个端口复制一份再给子线程，各个子线程的端口相互独立。
#同时父线程仍然保有一对端口，可实现父线程与各个子线程间通信。
#设置False，实现单向传输。

#pipe()返回两个连接对象分别表示管道的两端，每端都有send()和recv()方法。
#如果两个进程试图在同一时间的同一端进行读取和写入那么，这可能会损坏管道中的数据。


#父线程与多个子线程间双向通信。
def pipe_write(pipe):
    while True:
        try:
            pipe.send('msg from write')
            time.sleep(random.random() * 2)
            msg = pipe.recv()
            print('write get %s' % msg)
        except Exception as e:
            print(e)
            break


def pipe_read(pipe):
    while True:
        try:
            pipe.send('msg from read')
            time.sleep(random.random() * 2)
            msg = pipe.recv()
            print('read get %s' % msg)
        except Exception as e:
            print(e)
            break


if __name__ == '__main__':
    #True:双向通信
    #False,单向通行，A只能用来接收消息,B只能用来发送消息
    pipe_A, pipe_B = Pipe(True)
    #将B传递给多个子进程，会把B端复制后再创递，建立链接： A<---->B,A<----->C
    proc_read = Process(target=pipe_read, args=(pipe_B,))
    proc_write = Process(target=pipe_write, args=(pipe_B,))
    proc_read.start()
    proc_write.start()

    while True:
        try:
            pipe_A.send('msg from super')
            time.sleep(random.random() * 2)
            msg = pipe_A.recv()
            print('super get %s' % msg)
        except Exception as e:
            print(e)
            break

            
#通过Manager可实现进程间数据的共享。Manager()返回的manager对象会通过一个服务进程，来使其他进程通过代理的方式操作python对象。
# manager对象支持 `list`, `dict`, `Namespace`, `Lock`, `RLock`, `Semaphore`, `BoundedSemaphore`, `Condition`, `Event`, `Barrier`, `Queue`, `Value` ,`Array`.
from multiprocessing import Process, Lock

def f(d, l):
    d['1'] = 1
    l.append(1)

if __name__ == '__main__':
    with Manager() as manager:
        #构建同步数据对象
        d = manager.dict()
        l = manager.list()
        p_list = []
        for i in range(10):
            p = Process(target=f, args=(d, l))
            p.start()
            p_list.append(p)
        for res in p_list:
            res.join()

        print(d)
        print('-----------------')
        print(l)
#---------------------------------子进程同步锁       
#进程同步锁，进程同步。
#数据输出的时候保证不同进程的输出内容在同一块屏幕正常显示，防止数据乱序的情况。
def f(l, i):
    l.acquire()
    try:
        print('hello world', i)
    finally:
        l.release()

if __name__ == '__main__':
    lock =Lock()
    for num in range(10):
        Process(target=f, args=(lock, num)).start()
            
 #---------------------------------多线程           

#多任务可以由多进程完成，也可以由一个进程内的多线程完成。
#我们前面提到了进程是由若干线程组成的，一个进程至少有一个线程。
#由于线程是操作系统直接支持的执行单元，

import threading

#任何进程默认就会启动一个线程，我们把该线程称为主线程，主线程又可以启动新的线程,
# Python的threading模块有个current_thread()函数，它永远返回当前线程的实例。
# 主线程实例的名字叫MainThread，子线程的名字在创建时指定，我们用LoopThread命名子线程。
# 名字仅仅在打印时用来显示，完全没有其他意义，
# 如果不起名字Python就自动给线程命名为Thread-1，Thread-2……
def subs_thread():
    #current_thread()函数，它永远返回当前线程的实例
    print('%s is running!'%threading.current_thread().name)
    for i in range(3):
        print(i)
        time.sleep(random.random())
    print('%s is finish!' % threading.current_thread().name)

print('%s is running!'%threading.current_thread().name)
#创建子线程，子线程的名字在创建时指定(可选参数）
sub=threading.Thread(target=subs_thread,name='subs_thread')
#启动子线程
sub.start()
sub.join()
print('%s is finish!' % threading.current_thread().name)

 #---------------------------------多线程同步锁

# 多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，
# 而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，
# 因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。
#修改z资源值需要多条语句，而执行这几条语句时，线程可能中断，即执行顺序与逻辑顺序不同，从而导致多个线程把同一个对象的内容改乱了。

#同步锁：线程获得了锁，因此其他线程不能同时执行枷锁操作，只能等待，直到锁被释放后，获得该锁以后才执行。
# 由于锁只有一个，无论多少线程，同一时刻最多只有一个线程持有该锁

#阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。
# 其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，
# 导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。

count=0
#互斥锁
lock=threading.Lock()
#RLcok类的用法和Lock类一模一样，但它支持嵌套，，在多个锁没有释放的时候一般会使用使用RLcok类。
#lock = threading.RLock()
#互斥锁同时只允许一个线程更改数据，而Semaphore是同时允许一定数量的线程更改数据 
#semaphore = threading.BoundedSemaphore(5)  # 最多允许5个线程同时运行

def change(step):
    #加锁
    lock.acquire()
    # 告知Python这个变量名不是局部的，而是 全局 的，要在外部寻找它的定义。
    global  count
    try:
        for i in range(100000):
            count=count+step
            count=count-step
    finally:
        # 用finally 保证改完了一定会释放锁:
        lock.release()

for i in range(50):
    t_0=threading.Thread(target=change,args=(5,))
    t_1=threading.Thread(target=change,args=(8,))
    t_0.start()
    t_1.start()
    #join 很重要，如果没有join ,子线程还在运行时，主线程就开始执行下面的语句，取到的count就可能是个中间值，而非终值。
    t_0.join()
    t_1.join()

    if count != 0:
        print('wrong')
        print(count)
        break
print('----------------')


#ython的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，
# 任何Python线程执行前，必须先获得GIL锁，然后，每执行100条字节码，解释器就自动释放GIL锁，
# 让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，
# 所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。
#Python解释器由于设计时有GIL全局锁，导致了多线程无法利用多核。

#Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。
# 多个Python进程有各自独立的GIL锁，互不影响。

def loop():
    x = 0
    while True:
        x = x ^ 1

#对线程方式，只能使用一个核。
for i in range(multiprocessing.cpu_count()):
    t = threading.Thread(target=loop)
    t.start()

#对进程方式，能使用所有核。
if __name__=='__main__':
    for i in range(multiprocessing.cpu_count()):
        t = Process(target=loop)
        t.start()

 #---------------------------------多线程局部变量

#在多线程环境下，每个线程都有自己的数据。一个线程使用自己的局部变量比使用全局变量好，
# 因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。
#但是局部变量也有问题，就是在函数调用的时候，传递起来很麻烦。

#一个ThreadLocal变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，各个线程数据相互隔离，互不干扰。
# ThreadLocal解决了参数在一个线程中各个函数之间互相传递的问题

#ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，
# 这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

thread_data=threading.local()

def stu_read():
    #取数据
    name=thread_data.stu.name
    print('thread %s:%s'%(threading.current_thread().name,name))

def stu_write(name):
    #写数据
    thread_data.stu=Stu(name)
    stu_read()

class Stu(object):
    def __init__(self,name):
        self.name=name
#各个线程间数据独立
t_0=threading.Thread(target=stu_write,name='thread_A',args=('A',))
t_1=threading.Thread(target=stu_write,name='thread_B',args=('B',))
t_0.start()
t_1.start()


# 多进程和多线程，这是实现多任务最常用的两种方式。

# 首先，要实现多任务，通常我们会设计Master-Worker模式，Master负责分配任务，Worker负责执行任务，
# 因此，多任务环境下，通常是一个Master，多个Worker。
# 如果用多进程实现Master-Worker，主进程就是Master，其他进程就是Worker。
# 如果用多线程实现Master-Worker，主线程就是Master，其他线程就是Worker。

# 多进程模式最大的优点就是稳定性高，因为一个子进程崩溃了，不会影响主进程和其他子进程。
# 主进程挂了所有进程就全挂了，但是Master进程只负责分配任务，挂掉的概率低。
# 多进程模式的缺点是创建进程的代价大，在Unix/Linux系统下，用fork调用还行，在Windows下创建进程开销巨大。
# 另外，操作系统能同时运行的进程数也是有限的。

# 多线程模式通常比多进程快一点，多线程模式致命的缺点就是任何一个线程挂掉都可能直接造成整个进程崩溃，因为所有线程共享进程的内存。
# 在Windows下，多线程的效率比多进程要高。但多线程存在稳定性的问题。

# 线程切换

# 无论是多进程还是多线程，切换时候都要进行保存当前环境，准备下一个任务环境。
# 单任务模型:挨个处理完毕任务。
# 多任务模型：在多任务间频繁切换。
# 多任务一旦多到一个限度，切换操作就会消耗掉系统所有的资源，效率急剧下降。

# 计算密集型 vs. IO密集型

# 计算密集型任务的特点是要进行大量的计算，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，
# 但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。
# 计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。最好用C语言编写。
#
# 第二种任务的类型是IO密集型，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。
# 对于IO密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是IO密集型任务，比如Web应用。
# IO密集型任务执行期间，99%的时间都花在IO上，花在CPU上的时间很少，对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言。

# 异步IO

# 考虑到CPU和IO之间巨大的速度差异，一个任务在执行的过程中大部分时间都在等待IO操作，单进程单线程模型会导致别的任务无法并行执行，
# 因此，需要多进程模型或者多线程模型来支持多任务并发执行。
# 现代操作系统支持异步IO。如果充分利用操作系统提供的异步IO支持，就可以用单进程单线程模型来执行多任务，
# 这种全新的模型称为事件驱动模型，在多核CPU上，可以运行多个进程（数量与CPU核心数相同），充分利用多核CPU。
# 由于系统总的进程数量十分有限，因此操作系统调度非常高效。
# 对应到Python语言，单线程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。

```

### 4.1:分布式多进程

```python
#master
# 分布式系统

# 在Thread和Process中，应当优选Process，因为Process更稳定，
# 而且，Process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。
# Python的multiprocessing模块不但支持多进程，其中managers子模块还支持把多进程分布到多台机器上。
# 一个服务进程可以作为调度者，将任务分布到其他多个进程中，依靠网络通信。

# 通过managers模块把Queue通过网络暴露出去，就可以让其他机器的进程访问Queue了,
# 其它程序必须通过manager.get_task_queue()获得的Queue接口添加

import random,  queue,time
from multiprocessing.managers import BaseManager
from queue import Queue
from multiprocessing import freeze_support

task_size=10
# 发送任务的队列:
task_queue = queue.Queue(task_size)
# 接收结果的队列:
result_queue = queue.Queue(task_size)

# 从BaseManager继承的QueueManager:
class QueueManager(BaseManager):
    pass

# 服务进程负责启动Queue，把Queue注册到网络上，然后往Queue里面写入任务
# 把两个Queue都注册到网络上, callable参数关联了Queue对象，
# 由于QueueManager管理的不止一个Queue，所以，要给每个Queue的网络调用接口起个名字

def get_task_q():
    return task_queue
def get_result_q():
    return result_queue

def func():
    # 把两个Queue都注册到网络上, callable参数关联了Queue对象:
    QueueManager.register('get_task_queue', callable=get_task_q)
    QueueManager.register('get_result_queue', callable=get_result_q)

    # 绑定端口5000, 设置验证码'abc':
    # 保证两台机器正常通信，不被其他机器恶意干扰。
    manager = QueueManager(address=('127.0.0.1', 5000), authkey=b'abc')
    # 启动Queue:
    manager.start()
    try:
        # 获得通过网络访问的Queue对象:
        task = manager.get_task_queue()
        result = manager.get_result_queue()


        # 放几个任务进去:
        for i in range(task_size):
            n = random.randint(0, 10000)
            print('Put task %d...' % n)
            try:
                task.put(n)
            except Queue.Full:
                print('task queue is full.')

        # while not result_queue.full():
        #     time.sleep(1)

        # 从result队列读取结果:
        print('Try get results...')
        for i in range(task_size):
        #while not result.empty():
            try:
                r = result.get(timeout=10)
                print('Result: %s' % r)
            except Queue.Empty:
                print('task queue is empty.')
    except Exception:
        print('master fucked!')
    finally:
        # 关闭:
        manager.shutdown()
        print('master exit.')


if __name__ == '__main__':
    #windows下多进程可能会炸，添加这句可以缓解
    freeze_support()

    func()



```



```python
#worker
# # 分布式系统


import time, sys
from multiprocessing.managers import BaseManager

# 创建类似的QueueManager:
class QueueManager(BaseManager):
    pass

# 由于这个QueueManager只从网络上获取Queue，所以注册时只提供名字:
QueueManager.register('get_task_queue')
QueueManager.register('get_result_queue')

# 连接到服务器，也就是运行task_master.py的机器:
server_addr = '127.0.0.1'
print('Connect to server %s...' % server_addr)
# 端口和验证码注意保持与task_master.py设置的完全一致:
m = QueueManager(address=(server_addr, 5000), authkey=b'abc')
try:
    # 从网络连接:
    m.connect()
except:
    print('connect fucked!')
    sys.exit(-1)
# task_worker.py中根本没有创建Queue的代码，
# Queue对象存储在task_master.py进程中,其它程序通过master暴露接口访问。
# 获取Queue的对象:
task = m.get_task_queue()
result = m.get_result_queue()
# 从task队列取任务,并把结果写入result队列:
while not task.empty():
    try:
        n = task.get(timeout=1)
        print('run task %d * %d...' % (n, n))
        r = '%d * %d = %d' % (n, n, n*n)
        time.sleep(1)
        result.put(r)
    except Exception:
        print('task queue is empty.')
# 处理结束:
print('worker exit.')
```

### 4.2:Evevt

事件用于主线程控制其他线程的执行，事件是一个简单的线程同步对象，,全局定义了一个“Flag”，当flag值为“False”，那么event.wait()就会阻塞，当flag值为“True”，那么event.wait()便不再阻塞。

```python
#利用Event类模拟红绿灯
import threading
import time
import random
start=time.time()
event = threading.Event()
car_list=[]
car_count=0
lock=threading.Lock()

def lighter():
    time_count = 0
    # 设置标志位，初始值为绿灯
    event.set()     
    while True:
        #5~5.5s:黄灯，5.5~20s:红灯
        if 5 < time_count <=20 :
            # 红灯，清除标志位
            event.clear()  
            print("\33[41;1m<---------------------------->\033[0m")
        # 绿灯，设置标志位
        elif time_count > 20:
            # 设置标志位
            event.set()  
            time_count = 0
        #0~5s：绿灯
        else:
            print("\33[42;1m<---------------------------->\033[0m")
        #等待一秒
        time.sleep(1)
        time_count += 1

def car_out():
    while True:
        global car_list
        # 判断是否设置了标志位
        if event.is_set():
            if len(car_list)!=0:
                lock.acquire()
                #队列里的第一辆车出去
                pop_car=car_list.pop(0)
                print('<<<<------------%.2f s car %s,queue size:%s' % (time.time()-start,pop_car, len(car_list)))
                lock.release()
                #一辆车出队列耗时0.5s
                time.sleep(0.5)
        else:
            #红灯，队列等待
            if len(car_list)!=0:
                lock.acquire()
                print('----------------%.2f s car >= %s waiting, queue size:%s'%(time.time()-start,car_list[0],len(car_list)))
                lock.release()
            #等待设置标志位，没什么意义，只是为了防止不断输出等待信息。
            event.wait()

def car_in():
    global car_count
    global car_list
    while True:
        #平均2s来一辆车
        time.sleep(random.random()*4)
        lock.acquire()
        #新来的加到队尾
        car_count=car_count+1
        car_list.append(car_count)
        print('------------>>>>%.2f s car %s,queue size:%s' % (time.time() - start, car_count, len(car_list)))
        lock.release()
#一个红绿灯周期内平均来车：20/2=10，出车容量：5/0.5+1=11，不会堵塞。
light = threading.Thread(target=lighter,)
light.start()

car_in_thread = threading.Thread(target=car_in)
car_in_thread.start()

car_out_thread = threading.Thread(target=car_out)
car_out_thread.start()
```

### 4.3:定时器

```python
from threading import Timer
def hello():
    print("hello, world")

t = Timer(1, hello)
t.start()  # after 1 seconds, "hello, world" will be printed
```

### 4.4:协程

```python
#协程
#线程和进程的操作是由程序触发系统接口，最后的执行者是系统，它本质上是操作系统提供的功能。而协程的操作则是程序员指定的，在python中通过yield，人为的实现并发处理。
# 协程存在的意义：对于多线程应用，CPU通过切片的方式来切换线程间的执行，线程切换时需要耗时。协程，则只使用一个线程，分解一个线程成为多个“微线程”，在一个线程中规定某个代码块的执行顺序。
# 协程的适用场景：当程序中存在大量不需要CPU的操作时（IO）。
# 常用第三方模块gevent和greenlet。（本质上，gevent是对greenlet的高级封装，因此一般用它就行，这是一个相当高效的模块。）
# greenlet就是通过switch方法在不同的任务之间进行切换,以swich处为跳转节点.
# gevent通过joinall将任务和它的参数进行统一调度，实现单线程中的协程。

from greenlet import greenlet

def test1():
    print(12)
    #执行gr2
    gr2.switch()
    print(34)
    #执行gr2
    gr2.switch()

def test2():
    print(56)
    # 执行gr1
    gr1.switch()
    print(78)
#创建对象
gr1 = greenlet(test1)
gr2 = greenlet(test2)
#执行gr1
gr1.switch()


import gevent
def f(url):
    print('GET: %s' % url)
    # do something

gevent.joinall([
        gevent.spawn(f, 'https://www.python.org/'),
        gevent.spawn(f, 'https://www.yahoo.com/'),
        gevent.spawn(f, 'https://github.com/'),
])
```

## 5:正则表达式

```python
# 正则表达式
import  re
# 用\d可以匹配一个数字，\w可以匹配一个字母或数字，.可以匹配任意字符，\s可以匹配一个空格（也包括Tab等空白符）

# 用*表示任意个（包括0个），用+表示至少一个字符，用?表示0个或1个字符，用{n}表示n个字符，用{n,m}表示[n~m]个字符
limit=r'\d{3}\s+\d{3,8}'
result=re.match(limit,'123  74575545')
if result:
    print('matched')

# - , ; _ : 是特殊字符，在正则表达式中，要用'\'转义：\- \, \; \_ \:

# 要做更精确地匹配，可以用[]表示范围:[ABC]===>A or B or C
# [ABC]:表示满足表示至少满足三个条件中的一个的单个字符。
# [ABC]+ :表示由至少一个 满足A/B/C条件的单个字符 构成。
# [ABC]* 表示任意个 满足A/B/C条件的单个字符 构成。
# [ABC]{0,5}: 表示由0~5个 满足A/B/C条件的单个字符 构成
print('---------------------------------------------')
limit=r'[a-zA-Z\_][0-9a-zA-Z\_]*'
if re.match(limit,'result_1_A'):
    print('match')
if re.match(limit,'9_result_1_A'):
    print('match')
# A|B可以匹配A或B,|运算符优先级很低，AB|CD :满足条件AB 获得CD
print('---------------------------')
result=re.match(r'\d\_|[a-z]+','4_')
print(result)

result=re.match(r'\d\_|[a-z]+','erererer')
print(result)
# ^表示行的开头，^\d表示必须以数字开头。
# $表示行的结束，\d$表示必须以数字结束。
print(re.match(r'^\d+[a-z]','111a1a'))

# 由于Python的字符串本身也用\转义，'ABC\\-001'==>ABC_001
# 建议使用Python的r前缀，就不用考虑转义的问题:r'ABC\-001'==>ABC_001

# re.match(limit,str)方法判断是否匹配，如果匹配成功，返回一个Match对象，否则返回None。
# re.split(limit,str)返回切割后的str数组

print(re.split(r'[\,\;\s]+','1,2,;3 ,4; 5'))
# 提取满足条件的子串的强大功能
# m=re.match(^(limit_0)limit_1(limit_2)$，str):把满足条件0，2的字串提取出来
# m.group(0):str本身，group(1)开始才为目标字符串，groups():返回目标子串的元组。

m=re.match(r'(^\d{3})[\s-]+(\d+)$','123 - 74575120')
print(m.groups())

# 贪婪匹配
# 正则匹配默认是贪婪匹配，也就是匹配尽可能多的字符。
# r'^(\d+)(0*)$' ：数字0会被划入第一个条件。
# 加个?就可以采用非贪婪匹配：r'^(\d+?)(0*)$'
# '.*?'表示最短匹配任意长度字符串，长度可以为零
m=re.match(r'(\d+)(0*)','123700000')
print(m.groups())

m=re.match(r'^(\d+?)(0*)$','123700000')
print(m.groups())

# 编译
# 当我们在Python中使用正则表达式时，re模块内部会干两件事情：
# 编译正则表达式，如果正则表达式的字符串本身不合法，会报错；
# 用编译后的正则表达式去匹配字符串。
# 如果一个正则表达式要重复使用几千次，出于效率的考虑，我们可以预编译该正则表达式，接下来重复使用时就不需要编译这个步骤了，直接匹配

# compiled_limit=re.compile(limit)
# complied_limit.match(str)
# complied_limit.split(str)

temp=re.compile(r'(^\d{3})[\s-]+(\d*)$')
print(temp.match('545 - 4546455').groups())

# 邮箱验证
def email_match(email):

    '''
    >>>
    >>> email_match('tom@voyager.org')
    (True, 'tom')
    >>> email_match('bill.gates@microsoft.com')
    (True, 'bill.gates')
    >>> email_match('mr-bob@example.com')
    False

    :param email:
    :return: [true,name]or false
    '''

    limit=r'^[\w][\w\.\_]+@[\w]+\.[a-z]+$'
    result=re.match(limit,email)
    if result:
        #
        limit=r'.*?([\w\.]+)'
        name=re.match(limit,email)
        return (True,name[1])
    else:
        return False


if __name__=='__main__':
    import doctest
    doctest.testmod()
```



## 6:内建模块

### 6.1:日期

```python

# 详细格式时间：datetime
# 注意到datetime是模块，datetime模块还包含一个datetime类.
from datetime import datetime ,timedelta,timezone

#获得当前时间
# 返回当前日期和时间，其类型是datetime。
current=datetime.now()
print(type(current),current)
# 自定义时间，不带时区，需要自己设置
current=datetime(2019,11,11,11,11,11,1111)
print(current)
# timestamp:把 1970年1月1日 00:00:00 UTC+00:00时区的时刻称为epoch time，记为0（1970年以前的时间timestamp为负数），
# 当前时间就是相对于epoch time（把epoch的时区转换为当地时区后做差）的秒数，称为timestamp。
# timestamp是一个浮点数。如果有小数位，小数位表示毫秒数。
# timestamp 是与时区无关的变量。
# datetime表示的时间需要时区信息才能确定一个特定的时间，否则只能视为本地时间。
# 如果要存储datetime，最佳方法是将其转换为timestamp再存储，因为timestamp的值与时区完全无关。
dt=current.timestamp()
print(dt)
#转换为当地时区
print(datetime.fromtimestamp(dt))
#转换为0时区标准时间。
print(datetime.utcfromtimestamp(dt))
# str<==>datetime
current = datetime.strptime('2019-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')
print(current)
date_str=current.strftime('%Y-%I-%d %H:%M:%S %A')
print(date_str)

# 日期运算
current=current+timedelta(days=2,hours=5,minutes=4)
print(current)

# 时区转换
# 利用带时区的datetime，通过astimezone()方法，可以转换到任意时区。
#获得UTC 0
# 构建时区：timezone(timedelta(hours=8))
utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc)
print(utc_dt)
#转换为UTC 8
bj_dt = utc_dt.astimezone(timezone(timedelta(hours=8)))
print(bj_dt)
# 不是必须从UTC+0:00时区转换到其他时区，任何带时区的datetime都可以正确转换
# 转换为UTC 9
tokyo_dt = bj_dt.astimezone(timezone(timedelta(hours=9)))
print(tokyo_dt)


```

### 6.2:Collections

```python
# namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，
# 并可以用属性而不是索引来引用tuple的某个元素。
from collections import namedtuple, deque,OrderedDict,ChainMap,Counter
Point=namedtuple('Point',['x','y'])
point=Point(2,4)
print(point.x,point.y)
# 验证创建的Point对象是tuple的一种子类：
print(isinstance(point,tuple))

# 使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低。
# deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈
# deque实现list的append()和pop(),appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素。
my_list=deque([1,2,3,4])
my_list.append(5)
my_list.appendleft(0)
print(my_list)
print(my_list.popleft(),my_list.pop())
print(my_list)

# 使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。
# 如果要保持Key的顺序，可以用OrderedDict
# OrderedDict的Key会按照插入的顺序排列，不是Key本身排序
my_dict=dict([('a', 1), ('b', 2), ('c', 3)])
print(my_dict)
my_dict=OrderedDict([('a', 1), ('b', 2), ('c', 3)])
print(my_dict)

# ChainMap可以把一组dict串起来并组成一个逻辑上的dict。
# ChainMap本身也是一个dict，但是查找的时候，会按照顺序在内部的dict依次查找。
# combine_dict=ChainMap(dict_0, dict_1, dict_1)
# 顺序：dict_0->dicy_1->dict_2
# value=combine_dict['key']

# Chainmap 环境参数，默认参数，命令行参数：组合可实现多参数输入

# 解析命令行参数
import os, argparse
# 解析对象
parser = argparse.ArgumentParser()
# python file.py  -u bob => user=bob
parser.add_argument('-u', '--user')
parser.add_argument('-c', '--color')
namespace = parser.parse_args()
# 构建参数字典 (v非空)
command_line_args = { k: v for k, v in vars(namespace).items() if v }

# 环境变量
# user=bob python file.py
# 环境参数字典
env_args_dict=os.environ

# Counter是一个简单的计数器，例如，统计字符出现的个数：

counter=Counter()
for ch in 'fuck you leather man!':
    counter[ch]=counter[ch]+1
print(counter)
```

### 6.3:Hash校验

```python
# 摘要算法又称哈希算法、散列算法。它通过一个函数，把任意长度的数据转换为一个长度固定的数据串（通常用16进制的字符串表示）。
# 摘要算法就是通过摘要函数f()对任意长度的数据data计算出固定长度的摘要digest，目的是为了发现原始数据是否被人篡改过。

import hashlib

# MD5算法:生成结果是固定的128 bit字节，通常用一个32位的16进制字符串表示。
md5 = hashlib.md5()
md5.update('how to use md5 in python hashlib?'.encode('utf-8'))
# 多次添加消息
md5.update('fuck'.encode('UTF-8'))
print(md5.hexdigest())

#sha1 :结果是160 bit字节，通常用一个40位的16进制字符串表示。
sha1 = hashlib.sha1()
sha1.update('how to use sha1 in '.encode('utf-8'))
# 多次添加消息
sha1.update('python hashlib?'.encode('utf-8'))
print(sha1.hexdigest())

# 在计算哈希的时候，不能仅针对原始输入计算，需要增加一个salt来使得相同的输入也能得到不同的哈希，
# 加salt的哈希就是：计算一段message的哈希时，根据不通口令计算出不同的哈希。要验证哈希值，必须同时提供正确的口令。
# Hmac算法针对所有哈希算法都通用，无论是MD5还是SHA-1
import hmac
import random
# 传入的key和message都是bytes类型，str类型需要首先编码为bytes
msg=b'fuck'
key=(''.join([chr(random.randint(48, 122)) for i in range(20)])).encode('utf-8')
#h=hmac.new(key,msg,digestmod='MD5')
h=hmac.new(key,msg,digestmod='sha1')
# 多次添加消息
h.update('lether man'.encode('utf-8'))
print(h.hexdigest())

```

### 6.4:Base64 转换

```python
# Base64是一种用64个字符来表示任意二进制数据的方法。
# 首先，准备一个包含64个字符的数组,对二进制数据进行处理，每3个字节一组，一共是3x8=24bit，划为4组，每组正好6个bit,2^6=64
# 得到4个数字作为索引，然后查表，获得相应的4个字符，就是编码后的字符串。
# Base64编码会把3字节的二进制数据编码为4字节的文本数据，长度增加33%，
# 如果要编码的二进制数据不是3的倍数，最后会剩下1个或2个字节,
# Base64用\x00字节在末尾补足后，再在编码的末尾加上1个或2个=号，表示补了多少字节，解码的时候，会自动去掉。

import base64
msg=b'fuck you leather man!'
print(msg)
code_msg=base64.b64encode(msg)
print(code_msg)
decode_mag=base64.b64decode(code_msg)
print(decode_mag)

# 由于标准的Base64编码后可能出现字符+和/，在URL中就不能直接作为参数，
# 所以又有一种"url safe"的base64编码，其实就是把字符+和/分别变成-和_

code_msg=base64.urlsafe_b64encode(msg)
print(code_msg)
decode_mag=base64.urlsafe_b64decode(code_msg)
print(decode_mag)

#由于=字符也可能出现在Base64编码中，但=用在URL、Cookie里面会造成歧义，所以，很多Base64编码后会把=去掉
#因为Base64是把3个字节变为4个字节,=号为尾部补全内容，所以，Base64编码的长度永远是4的倍数，因此，需要加上=把Base64字符串的长度变为4的倍数，就可以正常解码了
```

### 6.5:struct 二进制转换

```python
# struct模块来解决bytes和其他二进制数据类型的转换
import struct

# struct的pack函数把任意数据类型变成bytes
# >表示字节顺序是big-endian，也就是网络序，
# ? :bool   1
# c :char   1  b :unsigned char 1
# h :short  2  H :unsigned short 2
# i :int    4  I :unsigned int 4
# l :long   4  L :unsigned long 4
# f :float  4
# d :double 8

byte_data=struct.pack('>IH', 102400,99)
# unpack把bytes变成相应的数据类型：
data=struct.unpack('>IH', byte_data)
print(data)
```

### 6.6:Contex

```python
# Python的with语句允许我们非常方便地使用资源，而不必担心资源没有关闭
# 实际上，任何对象，只要正确实现了上下文管理，就可以用于with语句
# with语句不仅可以管理文件，还可以管理锁、连接等等

# import  threading
# lock = threading.lock()
# with lock:
#     #执行一些操作
#     pass

# 实现上下文管理是通过__enter__和__exit__这两个方法实现的

# with语句中的[as variable]是可选的，如果指定了as variable说明符，则variable是上下文管理器expression调用__enter__()函数返回的对象。
# 所以，f并不一定就是expression，而是expression.__enter__()的返回值
# def __enter__(self):
#     print('Begin')
#     return self
#
# 分别为异常类型、异常信息和堆栈
# def __exit__(self, exc_type, exc_value, traceback):
#     if exc_type:
#         print('Error')
#     else:
#         print('End')

# @contextmanager 实现上下文管理

from contextlib import contextmanager
# 自定义类
class MyClass(object):
    def __init__(self,name):
        self.__name=name
    def tell(self):
        return self.__name
    def close(self):
        print('closed')
# 管理器
@contextmanager
def manager(name):
    # yield之前的代码等同于上下文管理器中的__enter__函数。
    print('begin')
    # yield的返回值等同于__enter__函数
    try:
        yield MyClass(name)
    except:
        pass
    # 等同于上下文管理器的__exit__函数
    print('end')
# print('begin')==> print(man.tell())==> print('end')
with manager('fuck you') as man:
    print(man.tell())

# 我们希望在某段代码执行前后自动执行特定代码，也可以用@contextmanager实现

@contextmanager
def manager():
    print('begin')
    yield
    print('end')
# print('begin')==> print('do something')==> print('end')
with manager() :
    print('do something')

# 如果一个对象没有实现上下文，我们就不能把它用于with语句。可以用closing()来把该对象变为上下文对象。
# 它的作用就是把任意对象变为上下文对象，并支持with语句。
# closeing上下文管理器仅使用于具有close()方法的资源对象
from contextlib import closing
# print(myclass.tell())===>myclass.close()
with closing(MyClass('hello world')) as myclass:
    print(myclass.tell())

```

### 6.7:Itertools

```python
import  itertools

# count()会创建一个无限的迭代器，
# 所以上述代码会打印出自然数序列，根本停不下来，
counts=itertools.count(2)
print(type(counts))
for item in counts:
    print(item)

# repeat()负责把一个元素无限重复下去，不过如果提供第二个参数就可以限定重复次数
repeats=itertools.repeat('msg',10)
print(type(repeats))
for item in repeats:
    print(item)

# 通过takewhile()等函数根据条件判断来截取出一个有限的序列
counts=itertools.takewhile(lambda x:x<15,counts)
print(list(counts))

# chain()可以把一组迭代对象串联起来，形成一个更大的迭代器
for item in itertools.chain('fuck','you'):
    print(item)


def pi(n):
    counts=list(map(lambda x:4/((-1)**(1+x)*(2*x-1)),itertools.takewhile(lambda x:x<n+1,  itertools.count(1))))
    return sum(counts)

assert 3.04 < pi(10) < 3.05
assert 3.13 < pi(100) < 3.14
assert 3.140 < pi(1000) < 3.141
assert 3.1414 < pi(10000) < 3.1415
print('ok')
```

### 6.8:Urllib

```python
from urllib import request, parse

# 模拟浏览器发送GET请求，就需要使用Request对象，通过往Request对象添加HTTP头，我们就可以把请求伪装成浏览器。
req = request.Request('http://www.douban.com/')
# 添加Http请求头
req.add_header('User-Agent',
               'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.16 Safari/537.36 Edg/79.0.309.12')

with request.urlopen(req) as f:
    print('Status:', f.status, f.reason)
    # 获取Http请求头
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:\n', f.read().decode('utf-8'))
    htmls = f.read().decode('utf-8')

# 操作XML有两种方法：DOM和SAX。DOM会把整个XML读入内存，解析为树，因此占用内存大，解析慢，优点是可以任意遍历树的节点。
# SAX是流模式，边读边解析，占用内存小，解析快，缺点是我们需要自己处理事件
# 使用SAX解析XML事件是start_element，end_element和char_data

# SAX模式
from xml.parsers.expat import ParserCreate

class DefaultSaxHandler(object):
    # 解析起始标签<a href="/python"> ===> name=a ,attrs: {'href': '/python'}
    def start_element(self, name, attrs):
        print('sax:start_element: %s, attrs: %s' % (name, str(attrs)))

    # 解析终止标签
    def end_element(self, name):
        print('sax:end_element: %s' % name)

    # 解析内容
    def char_data(self, text):
        print('sax:char_data: %s' % text)

xml = r'''<?xml version="1.0"?>
<ol>
    <li><a href="/python">Python</a></li>
    <li><a href="/ruby">Ruby</a></li>
</ol>
'''
# 绑定事件
handler = DefaultSaxHandler()
parser = ParserCreate()
parser.StartElementHandler = handler.start_element
parser.EndElementHandler = handler.end_element
parser.CharacterDataHandler = handler.char_data
# 开始解析，碰到一个标签就开始调用相应函数，嵌套解析内容
parser.Parse(xml)

# 解析html
from html.parser import HTMLParser

class MyHTMLParser(HTMLParser):
    # # 解析起始标签<a href="/python"> ===> name=a ,attrs: {'href': '/python'}
    def handle_starttag(self, tag, attrs):
        print('starttag-----------------------------------')
        print('<%s>' % tag)

    # 解析终止标签
    def handle_endtag(self, tag):
        print('endtag-----------------------------------')
        print('</%s>' % tag)

    def handle_startendtag(self, tag, attrs):
        print('startendtag-----------------------------------')
        print('<%s/>' % tag)

    # 解析内容
    def handle_data(self, data):
        print('data-----------------------------------')
        print(data)

    # 解析注释
    def handle_comment(self, data):
        print('commit-----------------------------------')
        print('<!--', data, '-->')

    def handle_entityref(self, name):
        print('entityref-----------------------------------')
        print('&%s;' % name)

    def handle_charref(self, name):
        print('charref-----------------------------------')
        print('&#%s;' % name)

parser = MyHTMLParser()
# feed()方法可以多次调用。碰到一个标签就开始调用相应函数，嵌套解析内容
parser.feed(htmls)
```

## 7:第三方模块

### 7.1:Requests

```python

import requests
# 对于带参数的URL，传入一个dict作为params参数
params={'q':'python','cat':'1001'}
# 要传入HTTP Header时，我们传入一个dict作为headers参数
my_header={'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit'}
# 请求中传入Cookie，只需准备一个dict传入cookies参数
# my_cookie={'kay':'value'}
# requests.get(...,cookies=my_cookie,...)
# 指定超时，传入以秒为单位的timeout参数：
r=requests.get('https://www.douban.com/search',params=params,headers=my_header,timeout=10)
print(r.url,r.status_code,r.reason)
# requests自动检测内容编码，可以使用encoding属性查看
print(r.encoding)
# 获得反馈json
# print(r.json())

# 获取响应头
print(r.headers)
print(r.headers['Content-Type'])

# 获取Cookie,把cookie对象转换为dict
cookies = requests.utils.dict_from_cookiejar(r.cookies)
print(cookies)
print(r.cookies['bid'])

# 用content属性获得bytes对象
print(r.content)

# 获取网页代码
print(r.text)

# 要发送POST请求，只需要把get()方法变成post()，然后传入dict形式的data参数作为POST请求的数据
# 把post()方法替换为put()，delete()等，就可以以PUT或DELETE方式请求资源。
# # 如果要传递JSON数据，可以直接传入dict形式的json参数
# 上传文件需要更复杂的编码格式，但是requests把它简化成files参数，
# 在读取文件时，注意务必使用'rb'即二进制模式读取，这样获取的bytes长度才是文件的长度
# my_data={'form_email': 'abc@example.com', 'form_password': '123456'}
# params = {'key': 'value'}
# upload_files = {'file': open('report.xls', 'rb')}
# r = requests.post(url, json=params,data=my_data,files=upload_files)

```

### 7.2:Psutils

```python
import psutil
#CPU线程；核心数；CPU的用户／系统／空闲时间：
print(psutil.cpu_count(),psutil.cpu_count(logical=False),psutil.cpu_times())
# CPU线程使用率，每3秒刷新一次
for i in range(5):
    print(psutil.cpu_percent(interval=3,percpu=True))
# 获取物理内存和交换内存信息
print(psutil.virtual_memory())
print(psutil.swap_memory())
# # 磁盘分区信息
print(psutil.disk_partitions())
# F磁盘使用情况
psutil.disk_usage('F:\\')
# 磁盘IO
print(psutil.disk_io_counters())
# 获取网络读写字节／包的个数
print(psutil.net_io_counters())
# 获取网络接口信息
print(psutil.net_if_addrs())
 # 获取网络接口状态
print(psutil.net_if_stats())

'''
#需要管理员权限
psutil.pids() # 所有进程ID
psutil.Process(8080) # 获取指定进程ID=3776，其实就是当前Python交互环境
p.name() # 进程名称
p.exe() # 进程exe路径
p.cwd() # 进程工作目录
p.cmdline() # 进程启动的命令行
p.ppid() # 父进程ID
p.parent() # 父进程
p.children() # 子进程列表
p.status() # 进程状态
p.username() # 进程用户名
p.create_time() # 进程创建时间
p.terminal() # 进程终端
p.cpu_times() # 进程使用的CPU时间
p.memory_info() # 进程使用的内存
p.open_files() # 进程打开的文件
p.connections() # 进程相关网络连接
p.num_threads() # 进程的线程数量
p.threads() # 所有线程信息
p.environ() # 进程环境变量
p.terminate() # 结束进程
'''

#实时监测进程信息
psutil.test()
```

## 8:Gui

```python
import tkinter as tk
from tkinter import filedialog, messagebox

# 参考： https://www.cnblogs.com/shwee/p/9427975.html

#根窗体就是画板，在tkinter中则是Toplevel，画布就是tkinter中的容器（Frame），
# 画板上可以放很多张画布（Convas），tkinter中的容器中也可以放很多个容器，
# 绘画中的构图布局则是tkinter中的布局管理器（几何管理器），绘画的内容就是tkinter中的一个个小组件

def gui():
    # 创建窗口
    window = tk.Tk()
    window.title('window title')
    # w*h
    window.geometry('1000x1000')
    # weight数据
    click_count=None
    entry_input=None
    text=None
    list_items=None
    radio_button=None
    check_button=None

    # 更改label内容
    def click():
        click_count.set(int(click_count.get()) + 1)
        print(click_count.get())
    #构建插件
    click_count = tk.StringVar()
    click_count.set(0)
    # height=2,就是标签有2个字符这么高
    # 动态显示文本，静态文本直接text='msg
    label = tk.Label(window,textvariable=click_count, font=('Arial', 12), width=30, height=2, bg='gray')
    #左上角位置坐标，label.pack()默认放在中间。
    label.place(x=10, y=0)
    # 构建按钮
    b = tk.Button(window, text='click', font=('Arial', 12), width=10, height=1, command=click)
    b.place(x=10, y=50)

    # 编辑text
    def insert():
        input=entry_input.get()
        #在鼠标处插入，'end':在尾部插入
        text.insert('insert',input)
        # x.y ：第x行，第y列，行从1 开始，列从0开始。
        # 1.2-->end
        print(text.get(1.2,'end'))

    # 单行文本输入域
    # show='*' 密码格式，普通文本：show:None
    entry_input = tk.Entry(window, show='*', font=('Arial', 14))
    entry_input.place(x=10, y=100)

    b = tk.Button(window, text='insert', font=('Arial', 12), width=10, height=1, command=insert)
    b.place(x=10, y=150)

    # 多行文本输入
    text = tk.Text(window, height=3,width=20)
    text.place(x=10, y=200)

    # 获得选中项
    def print_list_selection():
        print(list_items.get(list_items.curselection()))

    # 构建下拉列表
    var=tk.StringVar()
    var.set((1,2,3,4))
    list_items = tk.Listbox(window, listvariable=var,height=3)
    # lb.insert(1, 'first')       # 在第一个位置加入'first'字符
    # lb.delete(2)  #删除选项
    list_items.place(x=10, y=250)

    v=tk.Button(window,text='show selected', width=15, height=1, command=print_list_selection)
    v.place(x=10, y=450)

    #输出单选框内容
    def print_button_selection():
        print(radio_button.get())
    # 构建单选框
    radio_button=tk.StringVar()
    r1 = tk.Radiobutton(window, text='A', variable=radio_button, value='A', command=print_button_selection)
    r1.place(x=10, y=500)
    r2 = tk.Radiobutton(window, text='B', variable=radio_button, value='B', command=print_button_selection)
    r2.place(x=10, y=550)

    # 输出确认项
    def print_check_selection():
        print(check_button.get())

    # 构建check button
    check_button = tk.IntVar()
    c = tk.Checkbutton(window, text='check',variable=check_button, onvalue=1, offvalue=0, command=print_check_selection)    # 传值原理类似于radiobutton部件
    c.place(x=10, y=600)

    # 输出滑动条
    # 默认传参v
    def print_scale_selection(v):
        print(v)

    # 构建滑动条
    s = tk.Scale(window, label='scale msg', from_=0, to=10, orient=tk.HORIZONTAL, length=200, showvalue=1,tickinterval=2, resolution=0.1, command=print_scale_selection)
    s.place(x=10, y=650)

    # 构建帧
    frame = tk.Frame(window,bg='green',height=300,width=300)
    frame.place(x=600,y=10)
    # 在帧上构建画布
    canvas = tk.Canvas(frame, bg='red', height=200, width=200)
    # 说明图片位置，并导入图片到画布上
    image_file = tk.PhotoImage(file='pic.gif')  # 图片位置（相对路径，与.py文件同一文件夹下，也可以用绝对路径，需要给定图片具体绝对路径）
    image = canvas.create_image(0, 0, anchor='nw', image=image_file)  # 图片锚定点（n图片顶端的中间点位置）放在画布（250,0）坐标处
    canvas.place(x=10,y=10)

    # 文件操作
    def OpenFile():
        f = filedialog.askopenfilename(title='打开文件', filetypes=[('Python', '*.py *.pyw'), ('All Files', '*')])
        print(f)
        # 可使用os 模块运行文件
    def SaveFile():
        f = filedialog.asksaveasfilename(title='保存文件', initialdir='d:\mywork', initialfile='hello.py')
        print(f)

    def do_job():
        print('do something')

    # 构建按钮栏
    menubar = tk.Menu(window)
    # 菜单项（默认不下拉 tearoff=0，下拉内容包括Open，Save，Exit功能项）
    filemenu = tk.Menu(menubar, tearoff=0)
    # 将上面定义的空菜单命名为File，放在菜单栏中
    menubar.add_cascade(label='File', menu=filemenu)
    # 在File中加入Open、Save等小菜单，即我们平时看到的下拉菜单，每一个小菜单对应命令操作。
    filemenu.add_command(label='Open', command=OpenFile)
    filemenu.add_command(label='Save', command=SaveFile)
    # 添加一条分隔线
    filemenu.add_separator()
    # 用tkinter里面自带的quit()函数
    filemenu.add_command(label='Exit', command=window.quit)

    # 创建一个Edit菜单项（默认不下拉，下拉内容包括Cut，Copy，Paste功能项）
    editmenu = tk.Menu(menubar, tearoff=0)
    # 将上面定义的空菜单命名为 Edit，放在菜单栏中，就是装入那个容器中
    menubar.add_cascade(label='Edit', menu=editmenu)
    # 同样的在 Edit 中加入Cut、Copy、Paste等小命令功能单元，
    editmenu.add_command(label='Cut', command=do_job)
    editmenu.add_command(label='Copy', command=do_job)
    editmenu.add_command(label='Paste', command=do_job)

    #创建第二级菜单，即菜单项里面的菜单
    submenu = tk.Menu(filemenu)
    # 给放入的菜单submenu命名为Import
    filemenu.add_cascade(label='Import', menu=submenu, underline=0)  #

    # 创建第三级菜单命令
    submenu.add_command(label='Submenu', command=do_job)

    # 创建菜单栏完成后，配置让菜单栏menubar显示出来
    window.config(menu=menubar)

    # grid 实现布局
    # row为行，colum为列，padx就是单元格左右间距，pady就是单元格上下间距，
    # ipadx是单元格内部元素与单元格的左右间距，ipady是单元格内部元素与单元格的上下间距。
    canvas = tk.Canvas(window, bg='gray', height=200, width=200)
    canvas.place(x=600,y=400)
    for i in range(3):
        for j in range(3):
            tk.Label(canvas, text=1).grid(row=i, column=j, padx=10, pady=10, ipadx=10, ipady=10)

    # 创建消息框 bool
    result = messagebox.askokcancel('Python Tkinter', 'ok/cancel')
    print(result)
    # str yes/no
    result = messagebox.askquestion('Python Tkinter', "yes/no?")
    print(result)
    # bool
    result = messagebox.askyesno('Python Tkinter', 'true/flase？')
    print(result)
    # 警告，错误
    messagebox.showinfo('Python Tkinter', 'info')
    messagebox.showwarning('Python Tkinter', 'warning')
    messagebox.showerror('Python Tkinter', 'error')

    #创建一个全新的window
    window_up = tk.Toplevel(window)
    window_up.geometry('300x200')
    window_up.title('Sign up window')

    # window.mainloop就会让window不断的刷新，如果没有mainloop,就是一个静态的window
    # 一个gui程序只能有一个mainloop
    window.mainloop()

if __name__=='__main__':
    gui()

```

## 9: Net

### 9.1:TCP_web

```python

import socket
#  创建一个socket
# AF_INET指定使用IPv4协议，如果要用更先进的IPv6，就指定为AF_INET6。SOCK_STREAM指定使用面向流的TCP协议，
with socket.socket(socket.AF_INET,socket.SOCK_STREAM) as s:
    # 注意参数是一个tuple，包含地址和端口号。
    s.connect(('www.sina.com',80))
    # 发送数据:
    s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')
    buffer=[]
    while True:
        #一次最多接收指定的字节数
        temp=s.recv(1024)
        if temp:
            buffer.append(temp)
        # recv()返回空数据，表示接收完毕
        else:
            break
buffer=b''.join(buffer)
header, html = buffer.split(b'\r\n\r\n', 1)
print(header.decode('utf-8'))
with open('sina.html','w') as f:
    f.write(html.decode('utf-8'))

```

### 9.2:TCP_client

````python

import socket
import os
import struct
import json
# AF_INET指定使用IPv4协议，如果要用更先进的IPv6，就指定为AF_INET6。
# SOCK_STREAM指定使用面向流的TCP协议，收发都要使用二进制数据，注意转换。
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    # 建立连接:参数是一个tuple，包含地址和端口号。
    s.connect(('127.0.0.1', 9999))
    # 接收消息,1024 :一次最多接收指定的字节数
    print(s.recv(1024).decode('utf-8'))

    file_path='sina.html'
    # 得到文件的大小,字节
    filesize_bytes = os.path.getsize(file_path)
    filename = 'recv_'+file_path
    dirc = {
        'filename': filename,
        'filesize_bytes': filesize_bytes,
    }
    # 将字典转换成字符串
    head_info = json.dumps(dirc)
    # 将字符串的长度打包为2进制
    head_info_len = struct.pack('i', len(head_info))
    # 发送头文件
    s.send(head_info_len)
    s.send(head_info.encode('utf-8'))
    #发送文件
    with open(file_path,'rb') as f:
        s.sendall(f.read())
    # 反馈
    msg=s.recv(1024).decode('utf-8')
    print(msg)

````

### 9.3:TCP_server

```python
import socket
import threading
import struct
import json
import sys

# 进度条
def process_bar(precent, width=50):
    use_num = int(precent*width)
    space_num = int(width-use_num)
    precent = precent*100
    print('[%s%s]%d%%'%(use_num*'#', space_num*' ',precent),file=sys.stdout,flush=True, end='\r')

def tcplink(client,addr):
    client.send(b'welcome',)
    # 接受头文件数据
    head_length=struct.unpack('i', client.recv(4))[0]
    head=client.recv(head_length)
    head_dir = json.loads(head.decode('utf-8'))
    filesize = head_dir['filesize_bytes']
    filename = head_dir['filename']
    # 接受文件
    recv_size=0
    with open(filename,'w') as f:
        while True:
            temp=filesize-recv_size
            #recv() 方法在未收到消息下会阻塞当前线程
            process_bar(recv_size/filesize)
            if temp>1024:
                data=client.recv(1024)
                recv_size+=1024
                # 收发都是二进制数据流，要注意转换
                f.write(data.decode('utf-8'))
            else:
                data=client.recv(temp)
                # 收发都是二进制数据流，要注意转换
                f.write(data.decode('utf-8'))
                break
    client.send('trans finished'.encode('utf-8'))
    # 关闭连接
    client.close()
    print(addr,' closed')

# 一个Socket依赖4项：服务器地址、服务器端口、客户端地址、客户端端口来唯一确定一个Socket。
with socket.socket(socket.AF_INET,socket.SOCK_STREAM) as s:
    # 监听端口
    # # 要绑定监听的地址和端口。服务器可能有多块网卡，可以绑定到某一块网卡的IP地址上，也可以用0.0.0.0绑定到所有的网络地址，还可以用127.0.0.1绑定到本机地址。
    s.bind(('127.0.0.1', 9999))
    # 调用listen()方法开始监听端口，
    # 传入的参数指定等待连接的最大数量,实现链接多个对象
    s.listen(5)
    print('Waiting for connection...')
    while True:
        # 接受链接,accept()会等待并返回一个客户端的连接
        client,addr=s.accept()
        # 创建新线程来处理TCP连接:
        t=threading.Thread(target=tcplink,args=(client,addr))
        t.start()
```

### 9.4:UDP_client

```python
import socket
#SOCK_DGRAM指定了这个Socket的类型是UDP。
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    for data in [b'Michael', b'Tracy', b'Sarah']:
        # 不需要调用connect()，直接通过sendto()给服务器发数据：
        # 发送数据,无连接，发送时指明收信方
        s.sendto(data, ('127.0.0.1', 9999))
        # 接收数据:
        print(s.recv(1024).decode('utf-8'))

```

### 9.5:UDP_server

```python

import socket
import threading 

# 使用UDP协议时，不需要建立连接，只需要知道对方的IP地址和端口号，就可以直接发数据包。
#SOCK_DGRAM指定了这个Socket的类型是UDP。
with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    # 绑定端口,服务器绑定UDP端口和TCP端口互不冲突.
    s.bind(('127.0.0.1', 9999))
    while True:
        # 需要调用listen()方法，而是直接接收来自任何客户端的数据：
        # recvfrom()方法返回数据和客户端的地址与端口
        data,addr=s.recvfrom(1024)
        print('Received from %s:%s.' % addr)
        s.sendto(b'Hello, %s!' % data, addr)


```

## 10:Mail

参考：[Send_Recv_Mail][Mail_Link]

### 10.1:Send

```python
# 流程： 发件人 -> MUA ---(SMTP)---> MTA---(SMTP)--->MTA ->  MDA <--(POP3/IMAP)--- MUA <- 收件人
# 要编写程序来发送和接收邮件，本质上就是：编写MUA把邮件发到MTA；编写MUA从MDA上收邮件。

# SMTP是发送邮件的协议，Python内置对SMTP的支持，可以发送纯文本邮件、HTML邮件以及带附件的邮件。
# Python对SMTP支持有smtplib和email两个模块，email负责构造邮件，smtplib负责发送邮件。

from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.utils import parseaddr, formataddr

import smtplib
    
# 编码
def _format_addr(s):
    name, addr = parseaddr(s)
     # 如果包含中文，需要通过Header对象进行编码
    return formataddr((Header(name, 'utf-8').encode(), addr))

# 发件人信息
from_addr = '6781@qq.com'
password = 'password'
# 收件人信息
to_addr = '6781@qq.com'
# SMTP 服务器
smtp_server = 'smtp.qq.com'
# 163：465或者994 QQ: 465或587
SSL_part=587

# 一个邮件对象就是一个Messag对象，如果构造一个MIMEText对象，就表示一个文本邮件对象，
# 如果构造一个MIMEImage对象，就表示一个作为附件的图片，要把多个对象组合起来，就用MIMEMultipart对象，
# 而MIMEBase可以表示任何对象。
# Message
# +- MIMEBase
#    +- MIMEMultipart
#    +- MIMENonMultipart
#       +- MIMEMessage
#       +- MIMEText
#       +- MIMEImage

# ---------------------------纯文本-------------------------------------
# 第一个参数就是邮件正文，第二个参数是MIME的subtype，plain表示纯文本,html表示HTML页面，最终的MIME就是'text/plain'，用utf-8编码保证多语言兼容性。
# 发送HTML邮件，在构造MIMEText对象时，把HTML字符串传进去，再把第二个参数由plain变为html 
# msg = MIMEText('hello, send by Python...', 'plain', 'utf-8')
# msg = MIMEText('<html><body><h1>Hello</h1>'+'<p>send by <a href="http://www.python.org">Python</a>...</p>'+'</body></html>', 'html', 'utf-8')

# ---------------------------含附件-------------------------------------
# 创建MIMEMultipart()实例，然后构造附件,如果有多个附件，可依次构造.
# MIMEMultipart(alternative) :表示在收件方如果html无法正常解析，就是用纯文本。发件方要同时加入plain html
# MIMEMultipart()            :只发送plain或者html
msg = MIMEMultipart('alternative')
# 邮件正文是MIMEText
msg.attach(MIMEText('send with file...', 'plain', 'utf-8'))
# HTML内嵌图片要把图片当附件传入，利用编号调用，编号在附件的文件头中确定
msg.attach(MIMEText('<html><body><h1>Hello</h1>' +'<p><img src="cid:1"></p>' +'</body></html>', 'html', 'utf-8'))



# 添加附件就是加上一个MIMEBase，从本地读取文件:
with open('Mail\\pdf.pdf', 'rb') as f:
    # 设置附件的MIME和文件名。
    mime = MIMEBase('doc', 'pdf', filename='show_file_name.pdf')
    # 加上必要的头信息:
    mime.add_header('Content-Disposition', 'attachment', filename='show_file_name.pdf')
    # 文件ID，便于HTML调用
    mime.add_header('Content-ID', '<0>')
    mime.add_header('X-Attachment-Id', '0')
    # 把附件的内容读进来:
    mime.set_payload(f.read())
    # 用Base64编码:
    encoders.encode_base64(mime)
    # 添加到MIMEMultipart:
    msg.attach(mime)

# 读取本地文本文件
with open('Mail\\send.py', 'rb') as f:
    # 构造附件
    att = MIMEText(f.read(), 'base64', 'utf-8')
    # 文件头
    att["Content-Type"] = 'application/octet-stream'
    att["Content-Disposition"] = 'attachment; filename="show_file_name.py"'
    msg.attach(att)

# 添加图片
with open('Mail\\img.png', 'rb') as fp:
    msgImage = MIMEImage(fp.read())
    # 定义图片 ID，在 HTML 文本中引用
    msgImage.add_header('Content-ID', '<1>')
    mime.add_header('X-Attachment-Id', '1')
    # 文件头
    msgImage["Content-Disposition"] = 'attachment; filename="show_file_name.png"'
    msg.attach(msgImage)

# 邮件主题、如何显示发件人、收件人等信息并不是通过SMTP协议发给MTA，而是包含在发给MTA的文本中的
msg['From'] = _format_addr('2812 <%s>' % from_addr)
# 接收的是字符串而不是list，如果有多个邮件地址，用,分隔即可。SSL_pa
msg['To'] = _format_addr('2750 <%s>' % to_addr)

msg['Subject'] = Header('SMTP send', 'utf-8').encode()

# SSL加密传输
# Version 1
# server=smtplib.SMTP(smtp_server,SSL_part)
# server.starttls()
# Version 2
server =smtplib.SMTP_SSL(smtp_server)

# 打印出和SMTP服务器交互的所有信息
server.set_debuglevel(1)
# 登录SMTP服务器
server.login(from_addr, password)
# 可以一次发给多个人，所以传入一个list，邮件正文是一个str，as_string()把MIMEText对象变成str。
# ['****@mail.com','****@mail.com']
server.sendmail(from_addr, [to_addr], msg.as_string())
server.quit()
```

### 10.2:Recv

```python
# POP3协议收取的不是一个已经可以阅读的邮件本身，而是邮件的原始文本，这和SMTP协议很像，SMTP发送的也是经过编码后的一大段文本。
# 要把POP3收取的文本变成可以阅读的邮件，还需要用email模块提供的各种类来解析原始文本，变成可阅读的邮件对象。
# 第一步：用poplib把邮件的原始文本下载到本地；
# 第二部：用email解析原始文本，还原为邮件对象。

from email.parser import Parser
from email.header import decode_header
from email.utils import parseaddr

import poplib

# 输入邮件地址, 口令和POP3服务器地址:
email = '6781@qq.com'
password = 'password'
pop3_server ='pop.qq.com'

# 猜测文字编码
# 首先在message中寻找编码，如果没有，就在header的Content-Type中寻找
def guess_charset(msg):
    charset = msg.get_charset()
    if charset is None:
        content_type = msg.get('Content-Type', '').lower()
        pos = content_type.find('charset=')
        if pos >= 0:
            charset = content_type[pos + 8:].strip()
    return charset

# 解析文本
def decode_str(s):
    # decode_header()返回一个list,这里只返回了第一个元素
    value, charset = decode_header(s)[0]
    if charset:
        value = value.decode(charset)
    return value
# Message对象本身可能是一个MIMEMultipart对象，即包含嵌套的其他MIMEBase对象，
# 递归地打印出Message对象的层次结构
def print_info(msg, indent=0):
    # indent用于缩进显示。
    if indent == 0:
        # 第一层缩进，包含邮件基本信息
        for header in ['From', 'To', 'Subject']:
            value = msg.get(header, '')
            if value:
                if header=='Subject':
                    # 解析主题
                    value = decode_str(value)
                else:
                    # 解析收发件人信息，然后转换成unicode对象。
                    hdr, addr = parseaddr(value)
                    name = decode_str(hdr)
                    # u表示将后面跟的字符串以unicode格式存储
                    value = u'%s <%s>' % (name, addr)
            print('%s%s: %s' % ('  ' * indent, header, value))
    # 解析复合体的载荷
    if (msg.is_multipart()):
        parts = msg.get_payload()
        for n, part in enumerate(parts):
            print('%spart %s' % ('  ' * indent, n))
            print('%s--------------------' % ('  ' * indent))
            # 递归解析只载荷
            print_info(part, indent + 1)
    # 解析单体载荷
    else:
        content_type = msg.get_content_type()
        # html /plain
        if content_type=='text/plain' or content_type=='text/html':
            # 得到文本
            content = msg.get_payload(decode=True)
            # 猜测编码
            charset = guess_charset(msg)
            # 解析
            if charset:
                content = content.decode(charset)
            print('%sText: %s' % ('  ' * indent, content + '...'))

        # 附件
        else:
            print('%sAttachment: %s' % ('  ' * indent, content_type))
            # 下载附件
            file_name = msg.get_filename()
            file_name='Mail\\Recv_'+file_name
            with open(file_name,'wb') as f:
                data=msg.get_payload(decode=True)
                f.write(data)


# 连接到POP3服务器:
server = poplib.POP3_SSL(pop3_server)
# 可以打开或关闭调试信息:
server.set_debuglevel(1)
# 可选:打印POP3服务器的欢迎文字:
print(server.getwelcome().decode('utf-8'))
# 身份认证:
server.user(email)
server.pass_(password)
# stat()返回邮件数量和占用空间:
print('Messages: %s. Size: %s' % server.stat())
# list()返回所有邮件的编号:
# mails包含返回的所有邮件
# 返回一个3元祖(返回信息, 消息列表, 消息的大小)，如果指定index，就只返回指定消息的数据
resp, mails, octets = server.list()
# 可以查看返回的列表类似[b'1 82923', b'2 2184', ...]
print(mails)
# 获取最新一封邮件, 注意索引号从1开始:
index = len(mails)
# lines存储了邮件的原始文本的每一行,
# 可以获得整个邮件的原始文本:
# 获取详细index，设置为已读，返回3元组(返回信息, 消息msgnum的所以内容, 消息的字节数)，
# 如果指定index，就只返回指定消息的数据
resp, lines, octets = server.retr(index)
msg_content = b'\r\n'.join(lines).decode('utf-8')
# 稍后解析出邮件:
msg = Parser().parsestr(msg_content)
print_info(msg)
# 可以根据邮件索引号直接从服务器删除邮件:
# server.dele(index)
# 关闭连接:
server.quit()
```

## 11:SQLite

参考：[SQLite][SQL_Link0]	[SQLite][SQL_Link1]

```python



# 基于表（Table）的一对多的关系就是关系数据库的基础。
# 表是数据库中存放关系数据的集合，一个数据库里面通常都包含多个表，表和表之间通过外键关联。
# 首先连接到数据库，一个数据库连接称为Connection；
# 连接到数据库后，需要打开游标，称之为Cursor，通过Cursor执行SQL语句，然后，获得执行结果。

import sqlite3 as SQL
# 如果文件不存在，会自动在当前目录创建，timeout默认为5
# Connection和Cursor对象，打开后一定记得关闭
conn=SQL.connect('SQL\\test.db',timeout=5)
# 创建在内存上面,执行完任何操作后，都不需要提交事务的(commit)
# conn = sqlite3.connect('"memory:')

# 创建一个Cursor.
cursor=conn.cursor() 
# 执行语句
# 使用Cursor对象执行insert，update，delete语句时，执行结果由rowcount返回影响的行数，就可以拿到执行结果。
# 使用Cursor对象执行select语句时，通过featchall()可以拿到结果集。结果集是一个list，每个元素都是一个tuple，对应一行记录。
# 如果SQL语句带有参数，那么需要把参数按照位置传递给execute()方法，有几个?占位符就必须对应几个参数，参数用元组方式传入。
# REAL:浮点数，存储为IEEE 8byte浮点数
# TEXT:文本字符串，缺省的编码为utf-8
# INT:带符号的整数，根据值的大小，自动存储为1,2,3,4,5,8字节6种
# char :字符

cursor.execute('create table user (id INT  primary key , name varchar(20) not null)')
# 添加列
#cursor.execute('ALTER TABLE user ADD COLUMN age int')

# 插入数据
cursor.execute(r"insert into user (id, name) values (1, 'huang1')")
cursor.execute(r"insert into user (id, name) values (2, 'huang2')")
# 通过rowcount获得插入的行数:
print(cursor.rowcount)
# 获取查询结果集中所有（剩余）的行，返回一个列表。当没有可用的行时，则返回一个空的列表。
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
cursor.execute(r"UPDATE user set name = 'qiang' where ID=2")
# 通过rowcount获得插入的行数:
print(cursor.rowcount)
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
cursor.execute('DELETE from user where ID=2')
# 通过rowcount获得插入的行数:
print(cursor.rowcount)
cursor.execute('select * from user')
values = cursor.fetchall()
print(values)
# 关闭游标
cursor.close()
# 该方法回滚自上一次调用 commit() 以来对数据库所做的更改。
# conn.rollback()
# commit-->close
# 提交事务:
conn.commit()
conn.close()
```

## 12:Web

### 12.1:HTTP

>HTTP请求的流程：
>
>步骤1：浏览器首先向服务器发送HTTP请求，请求包括：
>
>方法：GET还是POST，GET仅请求资源，POST会附带用户数据；
>
>路径：/full/url/path；
>
>域名：由Host头指定：Host: [www.sina.com.cn](www.sina.com.cn)
>
>以及其他相关的Header；
>
>如果是POST，那么请求还包括一个Body，包含用户数据。
>
>步骤2：服务器向浏览器返回HTTP响应，响应包括：
>
>响应代码：200表示成功，3xx表示重定向，4xx表示客户端发送的请求有错误，5xx表示服务器端处理时发生了错误；
>
>响应类型：由Content-Type指定，例如：Content-Type: text/html;charset=utf-8表示响应类型是HTML文本，并且编码是UTF-8，Content-Type: image/jpeg表示响应类型是JPEG格式的图片；
>
>以及其他相关的Header；
>
>通常服务器的HTTP响应会携带内容，也就是有一个Body，包含响应的内容，网页的HTML源码就在Body中。
>
>步骤3：如果浏览器还需要继续向服务器请求其他资源，比如图片，就再次发出HTTP请求，重复步骤1、2。

### 12.2:WSGI

>##### hello.py
>
>```python
># environ：一个包含所有HTTP请求头信息的dict对象；
># start_response：一个发送HTTP响应头的函数。
>def application(environ, start_response):
>    # Header只能发送一次
>    # 函数接收两个参数，一个是HTTP响应码，一个是一组list表示的HTTP Header，每个Header用一个包含两个str的tuple表示。
>    start_response('200 OK', [('Content-Type', 'text/html')])
>    body = '<h1>Hello, %s!</h1>' % (environ['PATH_INFO'][1:] or 'web')
>    return [body.encode('utf-8')]
>    return [b'<h1>Hello, web!</h1>']
>
>```
>
>##### Server
>
>``` ### python
>#负责启动WSGI服务器，加载application()函数：
>from wsgiref.simple_server import make_server
># 导入我们自己编写的application函数:
>from Web.hello import application
>
># 创建一个服务器，IP地址为空，端口是8848，处理函数是application:
>httpd = make_server('', 8000, application)
>print('Serving HTTP on port 8000...')
># 开始监听HTTP请求:
>httpd.serve_forever()
>```

### 12.3:Flask_Web

``` python
### from flask import Flask
from flask import request

app = Flask(__name__)

# Flask通过Python的装饰器在内部自动地把URL和函数给关联起来
@app.route('/', methods=['GET', 'POST'])
def home():
    return '<h1>Home</h1>'

@app.route('/signin', methods=['GET'])
def signin_form():
    return '''<form action="/signin" method="post">
              <p><input name="username"></p>
              <p><input name="password" type="password"></p>
              <p><button type="submit">Sign In</button></p>
              </form>'''

@app.route('/signin', methods=['POST'])
def signin():
    # 需要从request对象读取表单内容：
    if request.form['username']=='admin' and request.form['password']=='password':
        return '<h3>Hello, admin!</h3>'
    return '<h3>Bad username or password.</h3>'

if __name__ == '__main__':
    app.run()
```

### 12.4:MVC

>##### App
>
>```python
>
># MVC：Model-View-Controller，中文名“模型-视图-控制器”。
># Python处理URL的函数就是C：Controller，Controller负责业务逻辑
># 包含变量的模板就是V：View，View负责显示逻辑，通过简单地替换一些变量，View最终输出的就是用户看到的HTML。
># 包含要用来替换的数据就是M,Model是用来传给View的，这样View在替换变量的时候，就可以从Model中取出相应的数据。
># 把Python代码和HTML代码最大限度地分离了。
># 一定要把模板放到正确的templates目录下，templates和flask_web.py在同级目录下：
>from flask import Flask, request, render_template
>
>app = Flask(__name__)
>
>@app.route('/', methods=['GET', 'POST'])
>def home():
>    return render_template('home.html')
>
>@app.route('/signin', methods=['GET'])
>def signin_form():
>    return render_template('form.html')
>
>@app.route('/signin', methods=['POST'])
>def signin():
>    username = request.form['username']
>    password = request.form['password']
>    if username=='admin' and password=='password':
>    # 通过render_template()函数来实现模板的渲染。
>        return render_template('signin-ok.html', username=username)
>    return render_template('form.html', message='Bad username or password', username=username)
>
>if __name__ == '__main__':
>    app.run()
>```
>
>##### templates
>
>>###### form.html
>>
>>```html
>><html>
>><head>
>>  <title>Please Sign In</title>
>></head>
>><body>
>>  {% if message %}
>>  <p style="color:red">{{ message }}</p>
>>  {% endif %}
>>  <form action="/signin" method="post">
>>    <legend>Please sign in:</legend>
>>    <p><input name="username" placeholder="Username" value="{{ username }}"></p>
>>    <p><input name="password" placeholder="Password" type="password"></p>
>>    <p><button type="submit">Sign In</button></p>
>>  </form>
>></body>
>></html>
>>```
>>
>>##### home.html
>>
>>```html
>><html>
>><head>
>>  <title>Home</title>
>></head>
>><body>
>>  <h1 style="font-style:italic">Home</h1>
>></body>
>></html>
>>```
>>
>>##### signin-ok.html
>>
>>```html
>><html>
>><head>
>>  <title>Welcome, {{ username }}</title>
>></head>
>><body>
>>  <p>Welcome, {{ username }}!</p>
>></body>
>></html>
>>```

## 13:Asynchronous IO

参考：[同步/异步 阻塞/非阻塞][asy_link0]	

​				[iterator / generator][asy_link1]	

​				[asyio/cortine][asy_link2]	

```python

# -------------------------------->同步/异步 阻塞/非阻塞------------------------------------

#
# 在一个线程中，CPU执行代码的速度极快，然而，一旦遇到IO操作，如读写文件、发送网络数据时，就需要等待IO操作完成，才能继续进行下一步操作。这种情况称为同步IO。
#
# 使用多线程或者多进程来并发执行代码，为多个用户服务。每个用户都会分配一个线程，如果遇到IO导致线程被挂起，其他用户的线程不受影响，但是系统不能无上限地增加线程。由于系统切换线程的开销也很大，一旦线程数量过多，CPU的时间就花在线程切换上了，结果导致性能严重下降。
#
# loop = get_event_loop()
# while True:
#     event = loop.get_event()
#     process_event(event)
# 当代码需要执行一个耗时的IO操作时，它只发出IO指令，并不等待IO结果，然后就去执行其他代码了。一段时间后，当IO返回结果时，再通知CPU进行处理。
#
# 当遇到IO操作时，代码只负责发出IO请求，不等待IO结果，然后直接结束本轮消息处理，进入下一轮消息处理过程。当IO操作完成后，将收到一条“IO完成”的消息，处理该消息时就可以直接获取IO操作结果。
#
# 在“发出IO请求”到收到“IO完成”的这段时间里，同步IO模型下，主线程只能挂起，但异步IO模型下，主线程并没有休息，而是在消息循环中继续处理其他消息。这样，在异步IO模型下，一个线程就可以同时处理多个IO请求，并且没有切换线程的操作。
#
# 烧水：
#  1 老张把水壶放到火上，立等水开。（同步阻塞）
#  2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）
#  3 老张把水开了会响的水壶放到火上，立等水开。（异步阻塞）
#  4 老张把水开了会响的水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）
#
#  所谓同步异步，只是对于水壶而言：普通水壶，同步，能没有结束前，一直等结果；响水壶，异步，不需要知道该功能结果，该功能有结果后通知（回调通知）。 响水壶可以在自己完工之后，提示老张水开了。 普通水壶同步只能让调用者去轮询自己。
#  所谓阻塞非阻塞，仅仅对于老张而言。 立等的老张，阻塞，（函数）没有接收完数据或者没有得到结果之前，不会返回；看电视的老张，非阻塞，（函数）立即返回，通过select通知调用者。

# -------------------------------->COROUTINE-------------------------------------------

# 子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。
# 所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。
# 子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。
# 协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。有点类似CPU的中断.

# 执行有点像多线程，但协程的特点在于是一个线程执行
# 协程极高的执行效率。子程序切换不是线程切换，而是由程序自身控制，没有线程切换的开销。
# 不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态。
# 协程是一个线程执行，利用多核CPU:多进程+协程，既充分利用多核，又充分发挥协程的高效率。

# yield 与yield
# from:
# yield list:返回整个列表，yield from list:一次返回一个元素。
# yield from:会自动处理大量错误。
# yield from后面必须是子生成器函数
#
# # 子生成器
# def ger_0():
#   loop:
#     x = yield
#     do
#     somthing
#   return result
#
# # 委托生成器
# def ger_1():
#     result = yield from ger_0()
#
# # 调用方
# g = ger_1()
# g.send(0) -->ger_0: x = 0
# g.send(1) -->ger_0: x = 1
# g.send(2) -->ger_0: x = 2
# ger_1: result = ger_0:result

# consumer函数是一个generator
def consumer():
    print('2---------')
    r = ''
    while True:
        print('3---------')
        # 通过yield拿到消息n处理，又通过yield把结果r传回
        # 赋值语句先计算= 右边，由于右边是 yield 语句，
        # 所以yield语句执行完以后，进入暂停，而赋值语句在下一次启动生成器的时候首先被执行；
        # P:sned(None)->C:yield r=''->P:send(1)->C:n=1->C:yield r='200 OK'->P:send(2)
        n = yield r
        if not n:
            print('6---------')
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    print('1---------')
    # 启动生成器
    # 在一个生成器函数未启动之前，是不能传递值进去。
    # 也就是说在使用c.send(n)之前，必须先使用c.send(None)或者next(c)来返回生成器的第一个值
    c.send(None)
    n = 0
    while n < 5:
        n = n + 1
        print('4---------')
        print('[PRODUCER] Producing %s...' % n)
        # 切换到consumer执行
        r = c.send(n)
        print('5---------')
        # 拿到consumer处理的结果，继续生产下一条消息
        print('[PRODUCER] Consumer return: %s' % r)
    # 关闭consumer，整个过程结束。
    c.close()

#   1->2->3 ->  4->3->5  ->  4->3->5  ->   4->3->5
c = consumer()
produce(c)


# -------------------------------->ASYNCIO-------------------------------------------

# https://blog.csdn.net/SL_World/article/details/86597738
# asyncio的编程模型就是一个消息循环。我们从asyncio模块中直接获取一个EventLoop的引用，然后把需要执行的协程扔到EventLoop中执行，就实现了异步IO。
# 异步操作需要在coroutine中通过yield from完成；
# 多个coroutine可以封装成一组Task然后并发执行。
import threading
import asyncio
# 把一个generator标记为coroutine类型
@asyncio.coroutine
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    # yield from语法可以让我们方便地调用另一个generator
    # asyncio.sleep()也是一个coroutine，所以线程不会等待asyncio.sleep()，而是直接中断并执行下一个消息循环。
    # 当asyncio.sleep()返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。
    yield from asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
# 两个coroutine是由同一个线程并发执行的。
# 直到循环事件的所有事件都处理完才能完整结束。
loop.run_until_complete(asyncio.wait(tasks))
loop.close()


# 引入了新的语法async和await，可以让coroutine的代码更简洁易读
# @asyncio.coroutine替换为async；
# 把yield from替换为await。

import threading
import asyncio

async def hello():
    print('Hello world! (%s)' % threading.currentThread())
    await asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()


# -------------------------------->调用方-子生成器-委托生成器<-------------------------------


import time
import asyncio

async def taskIO_1():
    print('开始运行IO任务1...')
    await asyncio.sleep(3)
    print('IO任务1已完成，耗时3s')
    return taskIO_1.__name__

async def taskIO_2():
    print('开始运行IO任务2...')
    await asyncio.sleep(2)
    print('IO任务2已完成，耗时2s')
    return taskIO_2.__name__

# 调用方
async def main():
    # 把所有任务添加到task中
    tasks = [taskIO_1(), taskIO_2()]

    # 返回已经完成的任务，完成一个返回一个
    for completed_task in asyncio.as_completed(tasks):
        # 子生成器
        resualt = await completed_task
        print('协程无序返回值：'+resualt)

    # # done：已经完成的任务，pending：未完成任务
    # # 等待任务全部完成才返回
    # done, pending = await asyncio.wait(tasks)
    # for r in done:
    #     print('协程无序返回值：' + r.result())

if __name__ == '__main__':
    start = time.time()
    # 创建一个事件循环对象loop
    loop = asyncio.get_event_loop()
    try:
        # 完成事件循环，直到最后一个任务结束
        loop.run_until_complete(main())
    finally:
        # 结束事件循环
        loop.close()
    print('所有IO任务总耗时%.5f秒' % float(time.time()-start))

# -------------------------------->AIOHTTP-------------------------------------------

# 把asyncio用在服务器端，例如Web服务器，由于HTTP连接就是IO操作，因此可以用单线程+coroutine实现多用户的高并发支持。
# asyncio实现了TCP、UDP、SSL等协议

from aiohttp import web

routes = web.RouteTableDef()

@routes.get('/')
async def index(request):
    await asyncio.sleep(2)
    return web.json_response({
        'name': 'index'
    })

@routes.get('/about')
async def about(request):
    await asyncio.sleep(0.5)
    return web.Response(text="<h1>about us</h1>")

def init():
    app = web.Application()
    app.add_routes(routes)
    web.run_app(app)

init()
```



## 14:其它

### Package

>新建包：在该包文件夹下新建 \_\_init\_\_.py
>
>同文件夹下相互引用：from file_0 import func_0
>
> 在pycharm下： make_directory as-->sources Root

## 15:链接

[GeneralLink]:https://www.liaoxuefeng.com/wiki/1016959663602400
[Proc_Thre_Link]:https://www.cnblogs.com/whatisfantasy/p/6440585.html
[Mail_link]: https://www.jianshu.com/p/abb2d6e91c1f
[SQL_Link0]:https://www.runoob.com/sqlite/sqlite-python.html
[SQL_Link1]:https://www.liaoxuefeng.com/wiki/1016959663602400/1017801751919456
[asy_link0]:https://www.cnblogs.com/Anker/p/5965654.html
[asy_link1]:https://blog.csdn.net/SL_World/article/details/86507872
[asy_link2]:https://github.com/SparksFly8/Learning_Python/blob/master/coroutine/README.md

