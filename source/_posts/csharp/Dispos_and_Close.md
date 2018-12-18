title: C#资源释放及Dispose、Close
author: gwl
tags:
  - c#
categories:
  - c#
date: 2018-01-09 19:41:00
---

## C#资源释放及Dispose、Close和析构方法
---
 

备注：此文的部分观点有误，之所以仍旧保留本文，是需要在后期给出一个勘误版。正确的版本在这里“C#中标准Dispose模式的实现”



一：什么是资源

在开始本文前，需要一些准备知识。首先要提出“什么是资源”。在CLR出来之后，Windows系统资源开始分为“非托管资源”和“托管资源”。

         非托管资源是指：所有的Window内核对象（句柄）都是非托管资源，如对于Stream，数据库连接，GDI+的相关对象，还有Com对象等等，这些资源并不是受到CLR管理；

         托管资源是指：由CLR管理分配和释放的资源，即由CLR里new出来的对象。

其次再来讲，资源的释放方式。

         非托管资源：需要显式释放的，也即需要你写代码释放；

         托管资源：并不需要显式释放，但是如果引用类型本身含有非托管资源，则需要进行现实释放；

二：显式释放的C#实现

显式释放的C#实现，由C#语法支持的有：

         1：实现IDisposable接口的Dispose方法；

         2：析构方法（终结器）；

         不由C#语法支持，但是约定支持的显式释放是：

         3：提供显示释放方法，比如常用的Close方法；

三：Dispose、Close和析构方法异同点

但是，还需要区分这3种方式的异同点。首先，你无法调用析构方法。析构方法是由垃圾回收机制进行调用的。换句话来说，就是你不知道析构方法被调用的时机。严格意义上来说，它只是作为资源释放的一个补救措施。

资源释放的一个正确的措施是为类型实现IDisposable接口的Dispose。当你需要释放类型的资源的时候，应该显示的调用Dipose方法。当然，这里还有一个C#的语法糖，就是使用using程序块，在离开using程序块的时候，CLR会自动调用类型所创建对象的Dipose方法。

可能有人会问道，既然可以通过Dispose方法的方式来进行资源的释放，为什么有些类型还需要提供一个Close方法。这里面的区别，或者说约定在于，如果你仔细观察这些类型：他们基本都只公开了Close方法，他们都实现了IDisposable，但都隐藏了Dispose方法。以Socket这个类为例，它：

1：提供public void Close()

        public void Close()
        {
            //….
            ((IDisposable)this).Dispose();
            //…. 
        }
2：提供显式void IDisposable.Dispose()

        void IDisposable.Dispose() 
        { 
            this.Dispose(true);
            GC.SuppressFinalize(this);
        }
3：提供protected virtual void Dispose(bool disposing)。真正的资源释放的代码放在这里。

所以理论上来将，提供Close方法最终还是使用的Dispose方法，之所以这么做，是因为这些类型出于显式实现IDisposable的因素，在调用这些Dispose方法的时候，必须完成一次转型，如： 

           ((IDisposable)new A()).Dispose(); 

为了避免转型，同时也为了避免不熟悉C#语法的开发人员更直观的释放资源，提供了Close方法。

在上文的例子中，你可能已经注意到IDisposable.Dispose这个方法中，包含一句： 

      GC.SuppressFinalize(this); 

这是告诉CLR，在进行垃圾回收的时候，不用再继续调用析构方法（终结器）了。是的，因为你已经手动释放资源了。这也从另一个方面验证了析构方法只是作为资源释放的补救机制。因为假设你忘记Close或者Dispose了，CLR会在垃圾回收的时候为你做这件事。查看Socket的析构函数，你会很好的理解这一点。

        ~Socket()
        {
            this.Dispose(false);
        }
是的，析构方法调用的也是Dispose。

备注1：本文带来几个争论

1：托管资源本身是否需要显式释放。答案显然是：不需要；

2：如果引用类型对象不再需要，是否需要显式=null；答案是：即使不这样做，GC也会进行垃圾回收。

3：将托管资源分为引用类型资源和值类型资源这种分类方法是有问题的，或者说是错误的。正确的分类法应该是栈资源和堆资源。线程栈中存放的是方法的实参和方法内部的局部变量。堆上存放的是类型对象本身及对象的两个额外成员：类型对象指针和同步块索引。

4：Dispose方法本身是用来让你放置资源清理代码的。显然，一个空方法并不代表清理工作本身，真正执行清理工作的是你具体的代码。

备注2：推荐Dipose模式实现

如：基类

复制代码
代码
    class ClassShouldDisposeBase : IDisposable
    {
        public void Dispose()
        {
            this.Dispose(true);
            GC.SuppressFinalize(this);
        }

        protected virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                //执行基本的清理代码
            }
        }

        ~ClassShouldDisposeBase()
        {
            this.Dispose(false);
        }

    }
复制代码
子类：

复制代码
代码
    class ClassShouldDispose: ClassShouldDisposeBase
    {
        protected virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                // 执行子类清理代码
                // 如有必要，执行base.Dispose(disposing);
            }
            else
            {
                // 如有必要，执行base.Dispose(disposing);
            }
        }

        public void Close()
        {
            //调用本类或者基类的Dispose方法
            //其它代码
        }
    }
复制代码