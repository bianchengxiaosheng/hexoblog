title: Native_methods_VS_reflection
author: gwl
tags:
  - UEPython
categories:
  - Unreal
date: 2018-01-09 23:32:00
---
## 本地方法与反射
通过默认的UObject class定义的getattr and setattr来作为unreal属性和函数的封装。
这意味着当我们调用： 
self.uobject.bCanBeDamaged = True  
就相当于  
self.uobject.set_property('bCanBeDamaged', True)  
同样的当调用函数时：  
vec = self.uobject.GetActorRightForward()  
就相当于调用  
vec = self.uobject.call_function('GetActorRightForward')  
最为重要的（方便的）K2_ functions也是自动被暴露的：  
vec = self.uobject.GetActorLocation()  
等价于：  
vec = self.uobject.call_function('K2_GetActorLocation')  
很明显的你可以组合方法或属性通过如下方式：  
self.uobject.CharacterMovement.MaxWalkSpeed = 600.0  
尽管系统允许所有unreal api的调用方式，但是使用反射还是比调用本地方法慢。  
所有最好还是尽可能的使用本地方法，所以无论什么时候你认为一个函数应该被暴露为本地方法时，请通过上述的本地方法调用来获取相关的内容。  
如下：  
vec = self.uobject.get_actor_location()   
是比  
vec = self.uobject.GetActorLocation()  
快很多。  
基于反射的方法使用的是骆驼拼写法(第一个首字母大写)。本地方法使用的而是python的命名风格，用小写的字母，下划线分隔方法名。
