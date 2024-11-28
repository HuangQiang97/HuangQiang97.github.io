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

## 15:链接

[GeneralLink]:https://www.liaoxuefeng.com/wiki/1016959663602400
[Proc_Thre_Link]:https://www.cnblogs.com/whatisfantasy/p/6440585.html
