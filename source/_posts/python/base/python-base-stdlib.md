{}
date: 2018-01-09 19:52:00
---
Python 标准库概览
=================

操作系统接口
---

[os](https://docs.python.org/3/library/os.html#module-os)
模块提供了很多与操作系统交互的函数:

    >>> import os
    >>> os.getcwd()      # Return the current working directory
    'C:\\Python35'
    >>> os.chdir('/server/accesslogs')   # Change current working directory
    >>> os.system('mkdir today')   # Run the command mkdir in the system shell
    0

应该用 `import os` 风格而非
`from os import *`。这样可以保证随操作系统不同而有所变化的
[os.open()](https://docs.python.org/3/library/os.html#os.open)
不会覆盖内置函数
[open()](https://docs.python.org/3/library/functions.html#open)。

在使用一些像 [os](https://docs.python.org/3/library/os.html#module-os)
这样的大型模块时内置的
[dir()](https://docs.python.org/3/library/functions.html#dir) 和
[help()](https://docs.python.org/3/library/functions.html#help)
函数非常有用:

    >>> import os
    >>> dir(os)
    <returns a list of all module functions>
    >>> help(os)
    <returns an extensive manual page created from the module's docstrings>

针对日常的文件和目录管理任务，[shutil](https://docs.python.org/3/library/shutil.html#module-shutil)
模块提供了一个易于使用的高级接口:

    >>> import shutil
    >>> shutil.copyfile('data.db', 'archive.db')
    'archive.db'
    >>> shutil.move('/build/executables', 'installdir')
    'installdir'

文件通配符
----------

[glob](https://docs.python.org/3/library/glob.html#module-glob)
模块提供了一个函数用于从目录通配符搜索中生成文件列表:

    >>> import glob
    >>> glob.glob('*.py')
    ['primes.py', 'random.py', 'quote.py']

命令行参数
----------

通用工具脚本经常调用命令行参数。这些命令行参数以链表形式存储于
[sys](https://docs.python.org/3/library/sys.html#module-sys) 模块的
*argv* 变量。例如在命令行中执行 `python demo.py one two three`
后可以得到以下输出结果:

    >>> import sys
    >>> print(sys.argv)
    ['demo.py', 'one', 'two', 'three']

[getopt](https://docs.python.org/3/library/getopt.html#module-getopt)
模块使用 Unix
[getopt()](https://docs.python.org/3/library/getopt.html#module-getopt)
函数处理 *sys.argv*。更多的复杂命令行处理由
[argparse](https://docs.python.org/3/library/argparse.html#module-argparse)
模块提供。

错误输出重定向和程序终止
------------------------

[sys](https://docs.python.org/3/library/sys.html#module-sys) 还有
*stdin*， *stdout* 和 *stderr* 属性，即使在 *stdout*
被重定向时，后者也可以用于显示警告和错误信息:

    >>> sys.stderr.write('Warning, log file not found starting a new one\n')
    Warning, log file not found starting a new one

大多脚本的直接终止都使用 `sys.exit()`。

字符串正则匹配
--------------

[re](https://docs.python.org/3/library/re.html#module-re)
模块为高级字符串处理提供了正则表达式工具。对于复杂的匹配和处理，正则表达式提供了简洁、优化的解决方案:

    >>> import re
    >>> re.findall(r'\bf[a-z]*', 'which foot or hand fell fastest')
    ['foot', 'fell', 'fastest']
    >>> re.sub(r'(\b[a-z]+) \1', r'\1', 'cat in the the hat')
    'cat in the hat'

只需简单的操作时，字符串方法最好用，因为它们易读，又容易调试:

    >>> 'tea for too'.replace('too', 'two')
    'tea for two'

数学
----

[math](https://docs.python.org/3/library/math.html#module-math)
模块为浮点运算提供了对底层C函数库的访问:

    >>> import math
    >>> math.cos(math.pi / 4.0)
    0.70710678118654757
    >>> math.log(1024, 2)
    10.0

[random](https://docs.python.org/3/library/random.html#module-random)
提供了生成随机数的工具:

    >>> import random
    >>> random.choice(['apple', 'pear', 'banana'])
    'apple'
    >>> random.sample(range(100), 10)   # sampling without replacement
    [30, 83, 16, 4, 8, 81, 41, 50, 18, 33]
    >>> random.random()    # random float
    0.17970987693706186
    >>> random.randrange(6)    # random integer chosen from range(6)
    4

SciPy \<<http://scipy.org>\> 项目提供了许多数值计算的模块。

互联网访问
----------

有几个模块用于访问互联网以及处理网络通信协议。其中最简单的两个是用于处理从
urls 接收的数据的
[urllib.request](https://docs.python.org/3/library/urllib.request.html#module-urllib.request)
以及用于发送电子邮件的
[smtplib](https://docs.python.org/3/library/smtplib.html#module-smtplib):

    >>> from urllib.request import urlopen
    >>> for line in urlopen('http://tycho.usno.navy.mil/cgi-bin/timer.pl'):
    ...     line = line.decode('utf-8')  # Decoding the binary data to text.
    ...     if 'EST' in line or 'EDT' in line:  # look for Eastern Time
    ...         print(line)

    <BR>Nov. 25, 09:43:32 PM EST

    >>> import smtplib
    >>> server = smtplib.SMTP('localhost')
    >>> server.sendmail('soothsayer@example.org', 'jcaesar@example.org',
    ... """To: jcaesar@example.org
    ... From: soothsayer@example.org
    ...
    ... Beware the Ides of March.
    ... """)
    >>> server.quit()

(注意第二个例子需要在 localhost 运行一个邮件服务器。)

日期和时间
----------

[datetime](https://docs.python.org/3/library/datetime.html#module-datetime)
模块为日期和时间处理同时提供了简单和复杂的方法。支持日期和时间算法的同时，实现的重点放在更有效的处理和格式化输出。该模块还支持时区处理。
:

    >>> # dates are easily constructed and formatted
    >>> from datetime import date
    >>> now = date.today()
    >>> now
    datetime.date(2003, 12, 2)
    >>> now.strftime("%m-%d-%y. %d %b %Y is a %A on the %d day of %B.")
    '12-02-03. 02 Dec 2003 is a Tuesday on the 02 day of December.'

    >>> # dates support calendar arithmetic
    >>> birthday = date(1964, 7, 31)
    >>> age = now - birthday
    >>> age.days
    14368

数据压缩
--------

以下模块直接支持通用的数据打包和压缩格式：[zlib](https://docs.python.org/3/library/zlib.html#module-zlib)，
[gzip](https://docs.python.org/3/library/gzip.html#module-gzip)，
[bz2](https://docs.python.org/3/library/bz2.html#module-bz2)，
[lzma](https://docs.python.org/3/library/lzma.html#module-lzma)，
[zipfile](https://docs.python.org/3/library/zipfile.html#module-zipfile)
以及
[tarfile](https://docs.python.org/3/library/tarfile.html#module-tarfile)。
:

    >>> import zlib
    >>> s = b'witch which has which witches wrist watch'
    >>> len(s)
    41
    >>> t = zlib.compress(s)
    >>> len(t)
    37
    >>> zlib.decompress(t)
    b'witch which has which witches wrist watch'
    >>> zlib.crc32(s)
    226805979

性能度量
--------

有些用户对了解解决同一问题的不同方法之间的性能差异很感兴趣。Python
提供了一个度量工具，为这些问题提供了直接答案。

例如，使用元组封装和拆封来交换元素看起来要比使用传统的方法要诱人的多。[timeit](https://docs.python.org/3/library/timeit.html#module-timeit)
证明了后者更快一些:

    >>> from timeit import Timer
    >>> Timer('t=a; a=b; b=t', 'a=1; b=2').timeit()
    0.57535828626024577
    >>> Timer('a,b = b,a', 'a=1; b=2').timeit()
    0.54962537085770791

相对于
[timeit](https://docs.python.org/3/library/timeit.html#module-timeit)
的细粒度，[profile](https://docs.python.org/3/library/profile.html#module-profile)
和
[pstats](https://docs.python.org/3/library/profile.html#module-pstats)
模块提供了针对更大代码块的时间度量工具。

质量控制
--------

开发高质量软件的方法之一是为每一个函数开发测试代码，并且在开发过程中经常进行测试。

[doctest](https://docs.python.org/3/library/doctest.html#module-doctest)
模块提供了一个工具，扫描模块并根据程序中内嵌的文档字符串执行测试。测试构造如同简单的将它的输出结果剪切并粘贴到文档字符串中。通过用户提供的例子，它发展了文档，允许
doctest 模块确认代码的结果是否与文档一致:

    def average(values):
        """Computes the arithmetic mean of a list of numbers.

        >>> print(average([20, 30, 70]))
        40.0
        """
        return sum(values) / len(values)

    import doctest
    doctest.testmod()   # automatically validate the embedded tests

[unittest](https://docs.python.org/3/library/unittest.html#module-unittest)
模块不像
[doctest](https://docs.python.org/3/library/doctest.html#module-doctest)
模块那么容易使用，不过它可以在一个独立的文件里提供一个更全面的测试集:

    import unittest

    class TestStatisticalFunctions(unittest.TestCase):

        def test_average(self):
            self.assertEqual(average([20, 30, 70]), 40.0)
            self.assertEqual(round(average([1, 5, 7]), 1), 4.3)
            with self.assertRaises(ZeroDivisionError):
                average([])
            with self.assertRaises(TypeError):
                average(20, 30, 70)

    unittest.main() # Calling from the command line invokes all tests

“瑞士军刀”
----------

Python
展现了“瑞士军刀”的哲学。这可以通过它更大的包的高级和健壮的功能来得到最好的展现。列如:

-   [xmlrpc.client](https://docs.python.org/3/library/xmlrpc.client.html#module-xmlrpc.client)
    和
    [xmlrpc.server](https://docs.python.org/3/library/xmlrpc.server.html#module-xmlrpc.server)
    模块让远程过程调用变得轻而易举。尽管模块有这样的名字，用户无需拥有
    XML 的知识或处理 XML。
-   [email](https://docs.python.org/3/library/email.html#module-email)
    包是一个管理邮件信息的库，包括MIME和其它基于 RFC2822 的信息文档。

    不同于实际发送和接收信息的
    [smtplib](https://docs.python.org/3/library/smtplib.html#module-smtplib)
    和
    [poplib](https://docs.python.org/3/library/poplib.html#module-poplib)
    模块，email
    包包含一个构造或解析复杂消息结构（包括附件）及实现互联网编码和头协议的完整工具集。

-   [xml.dom](https://docs.python.org/3/library/xml.dom.html#module-xml.dom)
    和
    [xml.sax](https://docs.python.org/3/library/xml.sax.html#module-xml.sax)
    包为流行的信息交换格式提供了强大的支持。同样，
    [csv](https://docs.python.org/3/library/csv.html#module-csv)
    模块支持在通用数据库格式中直接读写。

    综合起来，这些模块和包大大简化了 Python
    应用程序和其它工具之间的数据交换。

-   国际化由
    [gettext](https://docs.python.org/3/library/gettext.html#module-gettext)，
    [locale](https://docs.python.org/3/library/locale.html#module-locale)
    和
    [codecs](https://docs.python.org/3/library/codecs.html#module-codecs)
    包支持。

