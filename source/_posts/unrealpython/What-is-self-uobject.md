title: What_is_self.uobject
author: gwl
tags:
  - Unreal4
categories:
  - UE_Python
date: 2018-01-03 22:46:00
---
### 什么是self.uobject
为了让Python被无缝的集成，ue引擎的每个UObject都自动的被映射到一个特殊的Python Object上（ue_PyObject）.  
无论什么时候你通过Python访问UObject(UE的),你都能有效的通过获取一个ue_PyUObject引用去使用UObject上的特征（属性，方法，等等）。  
特殊的python object是被缓存到内存中一个c++的map里。(map的key是UObject的指针，value是ue_PyUObject的指针)  
为了直观的描述，如下代码段:  
	text_render_component = unreal_engine.find_class('TextRenderComponent')
将在内部搜索'TextRenderComponent' class(通过unreal c++的反射原理)，当搜索到的时候，它会在缓存中检查其是否可以通过map获取，如果缓存中没有，它将创建一个新的ue_PyUObject并将相应的对应关系通过map放入内存的缓存中。  
通过上述例子，'text_render_component'会保持一种到UObject的映射(也就是一个UClass)。  
注意：在你的python class中，你映射到的PyActor (or PyPawn, PyCharacter or PyComponent)并不是上述的
ue_PyUObject。它是一个传统的python class,只不过它有一个关联到ue_PyUObject映射的object引用（通过uobject获取）。最准确的通过技术来描述这个这些classes可以称之为'proxy(代理)'。
Pay attention: the python class you map to the PyActor (or PyPawn, PyCharacter or PyComponent), is not a ue_PyUObject. It is a classic python class that holds a reference (via the 'uobject' field) to the related ue_PyUObject mapped object. The best technical term to describe those classes is 'proxy'.

Note about 'uobject' from now on
在下面的介绍中，你所看到的所有'uobject'的引用都是一个ue_PyUObjectobject。
In the following lines, whenever you find a reference to 'uobject' it is meant as a ue_PyUObject object.

添加一个python组件到一个Actor上。
这个的工作原理同PyActor,但是它是一个组件。你可以将它附加（通过Actor蓝图上搜索‘Python’组件的方式）到任何一个Actor上（其实就是ue中组件的概念）。
记住在对于这些组件，self.uobject引用指的是组件本身，并不是Actor。
如果想要去获取actor，你可以使用如下代码：  
	actor = self.uobject.get_owner()
下面的例子将实现使用官方的第三人称蓝图作为一个python组件。

	class Player:
    
    def begin_play(self):
        # get a reference to the owing pawn (a character)
        self.pawn = self.uobject.get_owner()

        # the following two values were originally implemented as blueprint variable
        self.base_turn_rate = 45.0
        self.base_look_up_rate = 45.0

        # bind axis events
        self.pawn.bind_axis('TurnRate', self.turn)
        self.pawn.bind_axis('LookUpRate', self.look_up)
        self.pawn.bind_axis('Turn', self.pawn.add_controller_yaw_input)
        self.pawn.bind_axis('LookUp', self.pawn.add_controller_pitch_input)

        self.pawn.bind_axis('MoveForward', self.move_forward)
        self.pawn.bind_axis('MoveRight', self.move_right)

        # bind actions
        self.pawn.bind_action('Jump', ue.IE_PRESSED, self.pawn.jump)
        self.pawn.bind_action('Jump', ue.IE_RELEASED, self.pawn.stop_jumping)

    def turn(self, axis_value):
        turn_rate = axis_value * self.base_turn_rate * self.uobject.get_world_delta_seconds()
        self.pawn.add_controller_yaw_input(turn_rate)

    def look_up(self, axis_value):
        look_up_rate = axis_value * self.base_look_up_rate * self.uobject.get_world_delta_seconds()
        self.pawn.add_controller_pitch_input(look_up_rate)

    def move_forward(self, axis_value):
        rot = self.pawn.get_control_rotation()
        fwd = ue.get_forward_vector(0, 0, rot[2])
        self.pawn.add_movement_input(fwd, axis_value)

    def move_right(self, axis_value):
        rot = self.pawn.get_control_rotation()
        right = ue.get_right_vector(0, 0, rot[2])
        self.pawn.add_movement_input(right, axis_value)
Adding a python component to an Actor
This works in the same way as the PyActor class, but it is, well, a component. You can attach it (search for the 'Python' component) to any actor.

Remember that for components, the self.uobject field point to the component itself, not the actor.

To access the actor you can use:

actor = self.uobject.get_owner()
The following example implements the third person official blueprint as a python component:

class Player:
    
    def begin_play(self):
        # get a reference to the owing pawn (a character)
        self.pawn = self.uobject.get_owner()

        # the following two values were originally implemented as blueprint variable
        self.base_turn_rate = 45.0
        self.base_look_up_rate = 45.0

        # bind axis events
        self.pawn.bind_axis('TurnRate', self.turn)
        self.pawn.bind_axis('LookUpRate', self.look_up)
        self.pawn.bind_axis('Turn', self.pawn.add_controller_yaw_input)
        self.pawn.bind_axis('LookUp', self.pawn.add_controller_pitch_input)

        self.pawn.bind_axis('MoveForward', self.move_forward)
        self.pawn.bind_axis('MoveRight', self.move_right)

        # bind actions
        self.pawn.bind_action('Jump', ue.IE_PRESSED, self.pawn.jump)
        self.pawn.bind_action('Jump', ue.IE_RELEASED, self.pawn.stop_jumping)

    def turn(self, axis_value):
        turn_rate = axis_value * self.base_turn_rate * self.uobject.get_world_delta_seconds()
        self.pawn.add_controller_yaw_input(turn_rate)

    def look_up(self, axis_value):
        look_up_rate = axis_value * self.base_look_up_rate * self.uobject.get_world_delta_seconds()
        self.pawn.add_controller_pitch_input(look_up_rate)

    def move_forward(self, axis_value):
        rot = self.pawn.get_control_rotation()
        fwd = ue.get_forward_vector(0, 0, rot[2])
        self.pawn.add_movement_input(fwd, axis_value)

    def move_right(self, axis_value):
        rot = self.pawn.get_control_rotation()
        right = ue.get_right_vector(0, 0, rot[2])
        self.pawn.add_movement_input(right, axis_value)