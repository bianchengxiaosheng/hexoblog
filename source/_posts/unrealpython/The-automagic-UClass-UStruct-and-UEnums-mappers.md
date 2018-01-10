title: The_automagic_UClass_UStruct_and_UEnums_mappers
author: gwl
date: 2018-01-09 23:59:08
tags:
---
The automagic UClass, UStruct and UEnums mappers
------------------------------------------------

Instead of doing a gazilion of unreal_engine.find_class(name) calls, the plugin adds three 'magic' modules called unreal_engine.classes, unreal_engine.structs and unreal_engine.enums. They allows to import unreal classes/structs/enums like python classes:

```py
from unreal_engine.classes import ActorComponent, ForceFeedbackEffect, KismetSystemLibrary

...
components = self.uobject.get_owner().GetComponentsByClass(ActorComponent)

...
self.force_feedback = ue.load_object(ForceFeedbackEffect, '/Game/vibrate')
self.uobject.get_player_controller().ClientPlayForceFeedback(self.force_feedback)

...
name = KismetSystemLibrary.GetObjectName(self.actor)
```

the last example, shows another magic feature: static classes function calls. Obviously in this specific case using self.actor.get_name() would have been the best approach, but this feature allows you to access your blueprint function libraries too.

Another example for adding a widget:

```py
from unreal_engine.classes import WidgetBlueprintLibrary

class PythonFunnyActor:
    def begin_play(self):
        WidgetBlueprintLibrary.Create(self.uobject, ue.find_class('velocity_C'))
```

And another complex example using enums, keyword arguments and output values (output values are appended after the return value):
```py

import unreal_engine as ue
from unreal_engine import FVector, FRotator, FTransform, FHitResult
from unreal_engine.classes import ActorComponent, ForceFeedbackEffect, KismetSystemLibrary, WidgetBlueprintLibrary
from unreal_engine.enums import EInputEvent, ETraceTypeQuery, EDrawDebugTrace

...

is_hitting_something, hit_result = KismetSystemLibrary.LineTraceSingle_NEW(self.actor, self.actor.get_actor_location(), FVector(300, 300, 300), ETraceTypeQuery.TraceTypeQuery1, DrawDebugType=EDrawDebugTrace.ForOneFrame)
if is_hitting_something:
    ue.log(hit_result)
```

Remember that structs are passed by value (not by ref like UObject's), so a dedicated unreal_engine.UScriptStruct python class is exposed.

To create a new struct instance you can do:

```python
from unreal_engine.structs import TerrificStruct

ts = TerrificStruct()
```

or (to initialize some of its fields)

```python
from unreal_engine.structs import TerrificStruct

ts = TerrificStruct(Foo='Bar', Test=17.22)
```

To access the fields of a struct just call the fields() method.

A good example of struct usage is available here: https://github.com/20tab/UnrealEnginePython/blob/master/docs/Settings.md
