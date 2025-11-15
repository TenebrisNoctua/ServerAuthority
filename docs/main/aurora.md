# Behaviors

!!! warning
    The `Instance` and the Service mentioned in this page cannot be accessed without configuring certain flags. This `Instance` is also in heavy development and is not currently recommended for work.

## What are Behaviors?

In the Server Authority system, you need to simulate the physics and controls for a certain `Instance` on both the server and the client simultaneously. This is what allows the systems explained in the previous pages to work.

To achieve this, there are two ways one can go on about, such as creating a multi-script architecture, where exists a `ModuleScript` which exports a method that runs the simulation for the `Instance`, that is then required and ran by both the server and the client.
Or a single-script architecture, where one or more `Script`s do the same thing. Regardless of which way one can go on about, it is not convenient to do this system in multiple `Script`s just to be able to run the same system on both the server and the client. These `Script`s may also run into certain issues such as yielding.

Luckily for us, there exists a work-in-progress `Instance` that makes it more convenient for us to run the same simulation on both the server and the client, while ensuring that our `Script`s will not run into any yielding issues. It allows us to define Behaviors, which are systems that run the same code on both the server and the client simultaneously, and can be bound to `Instance`s.

-----

## AuroraScript

This `Instance` that I've mentioned above is called `AuroraScript`. This new script `Instance` allows you to define those Behaviors that can be bound to other `Instance`s and operate on them. 

!!! note
    In the future, the class name "AuroraScript" will most likely be replaced with "BehaviorScript".

Unlike the other script `Instance`s, `AuroraScript` has certain limitations placed within its environment. There are a few reasons for this, ranging from optimization, to ensuring the thread does not yield, and to Behaviors to smoothly interact with other Behaviors and run within the world.

## Limitations

* All `AuroraScript`s in the same location (e.g, `workspace`) must have a different name. Otherwise, one of them will not run.
* Certain APIs and global libraries do not work in the environment of an `AuroraScript`. The ones I'm aware of at the moment are:
    * All methods of the `task` library (Except for `task.cancel()`).
    * `RunService:IsClient()` and `RunService:IsServer()`.
    * `coroutine.yield()` (Behaviors cannot yield.)

Besides the limitations, generally `AuroraScript`s should always be parented to a location that can be both accessed from the client and the server, such as `ReplicatedStorage`. If you parent an `AuroraScript` to a location such as `ServerScriptStorage`, this will cause it to only run on the server, and not the client.

## Methods

### `AuroraScript:AddTo(instance: Instance): ()`

This method binds the Behavior `AuroraScript` to the specified `Instance`.
This then calls the (if defined) `.OnStart` method of the Behavior.

!!! note
    Behaviors can also be bound manually, without the need of this method.
    With the Server Authority feature enabled, the Properties widget gain a new section for every `Instance`, called: "Behaviors".
    Clicking the "+" button on this section will allow you to find and bind any Behavior on an `Instance`.

### `AuroraScript:RemoveFrom(instance: Instance): ()`

This method removes the Behavior `AuroraScript` from the specified `Instance`.

### `AuroraScript:IsOnInstance(instance: Instance): ()`

This method allows you to check if the Behavior `AuroraScript` is on the specified `Instance`.

### `AuroraScript:SignalFired(instance: Instance, topic: string): RBXScriptSignal`

This method allows you to create a Signal that is fired when the `AuroraScriptObject` of the `AuroraScript` publishes a value with a topic.

-----

## The Behavior Data Type

Every `AuroraScript` comes with a global data type called `Behavior`, which can be accessed everywhere from the script. We use this data type to define our methods which will run the simulation system.

!!! info
    The term "Behavior" represents an `AuroraScript`. `AuroraScript`s do not have Behaviors, they are the Behaviors themselves. In certain places, for example the "Behaviors" section in the Properties window, this is more apparent.

!!! warning
    Every Behavior must be bound to an `Instance` for them to work.

There are certain methods you have to define in the Behavior for it to start working. They are listed below.

### `Behavior.OnStart(self: AuroraScriptObject): ()`

This is a special method that you can define in the Behavior, which runs when the Behavior has been started. It allows you to connect certain events to certain methods and set certain field values, which are explained below.

```luau
function Behavior.OnStart(self: AuroraScriptObject)
    print(self)
end
```

You might've noticed that `.OnStart` has a parameter called `self`, which is an `AuroraScriptObject`. This is the object that is given to every function that you define in the Behavior. It will be explained in detail in the next section.

!!! info
    Behaviors are started when the physics simulation begins, if they've already been bound to an `Instance` before the simulation.
    If not, then Behaviors are started after `:AddTo()` has been called on them, or if they've been manually bound to an `Instance` in the Properties widget.

### `Behavior.OnStop(self: AuroraScriptObject): ()`

This is a special method that you can define in the Behavior, which runs when the Behavior has been stopped.

```luau
function Behavior.OnStop(self: AuroraScriptObject)
    print(self)
end
```

!!! info
    Behaviors are stopped when the Physics simulation ends, if they've already been started.
    If not, then Behaviors are stopped when `:RemoveFrom()` has been called on them, or if they've been manually removed from an `Instance` in the Properties widget.

### `Behavior.DeclareField(fieldName: string, _: { Type: "boolean" | "cframe" | "color3" | "enum" | "instance" | "number" | "random" | "vector2" | "vector3" }): ()`

This method allows you to define a field with a certain type in the `AuroraScriptObject`. The defined field will appear as a property within the `AuroraScriptObject`.

### `Behavior.ExportMethod(methodName: string)`

This method allows you to export a defined method in the data type. Do note that you must call this after the method with the specified `methodName` has been declared, otherwise, it will error.

-----

## AuroraScriptObject

This is the object given to the every function defined in the Behavior. It has certain properties and methods that allows you to do things such as publishing and sending values and messages across Behaviors, and more. 

```luau
type AuroraScriptObject = {
    Instance: Instance,
    Frame: number,
    LODLevel: number,
    Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
    Subscribe: (self: AuroraScriptObject, topic: string, functionName: string) -> string,
    Publish: (self: AuroraScriptObject, topic: string, ...any) -> any,
    SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
    Delay: (self: AuroraScriptObject, amount: number, functionName: string) -> string,
    SetMaxFrequency: (self: AuroraScriptObject, frequency: number) -> number
}

function Behavior.OnStart(self: AuroraScriptObject)
    print(self) -- prints the AuroraScriptObject
end
```

Now be warned, unlike any other object which is found in the Roblox API, `AuroraScriptObject` does not reflect the properties found in the API dump. Even when you print this object, it does not give you any property names that allows you to directly access a certain value.

An example:

```luau
{
    [Attached Instance] = Part,
    [Behavior] = AuroraScript,
    [Frame ID] = 509,
    [self] =  ▶ {...}
}
```

This is a detailed string that shows you information about the `AuroraScriptObject`, such as which frame it currently is on, the Behavior it is from, the bound instance, and lastly, the `self` table, which is used to show you the declared fields made with `Behavior.DeclareField()` function. (These field properties can be accessed through indexing the `AuroraScriptObject` with their name.)

Unfortunately, you cannot access these properties directly, and most likely this information is only shown for debugging purposes.
In the previous example however, you might have noticed that I've added an `AuroraScriptObject` type which contains information about all of the currently known and actually accessible methods and properties of the `AuroraScriptObject`. 
(Of course, I have found these properties using reverse-engineering methods.) They are described in detail below.

### Properties

### `AuroraScriptObject.Instance`

This is the `Instance` that the Behavior is bound to.

### `AuroraScriptObject.Frame`

The current Frame the world is on. Most likely changes after Heartbeat or PreAnimation.

### `AuroraScriptObject.LODLevel`

The level of distance number of the Behavior. Not sure what it is supposed to be used for at the moment.

### Methods

### `AuroraScriptObject.Connect:(self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection`

This method allows you to connect to a certain given `RBXScriptSignal`. The `functionName` argument must be the name of a function in the Behavior.

```lua

local RunService = game:GetService("RunService")

type AuroraScriptObject = {
    Instance: Instance,
    Frame: number,
    LODLevel: number,
    Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
    Subscribe: (self: AuroraScriptObject, topic: string, functionName: string) -> string,
    Publish: (self: AuroraScriptObject, topic: string, ...any) -> any,
    SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
    Delay: (self: AuroraScriptObject, amount: number, functionName: string) -> string,
    SetMaxFrequency: (self: AuroraScriptObject, frequency: number) -> number
}

function Behavior.OnStart(self: AuroraScriptObject)
    self:Connect(RunService.Heartbeat, "Test")
end

function Behavior.Test(self: AuroraScriptObject, deltaTime: number) -- This method will get called whenever the .Heartbeat event fires.
    print(self, deltaTime)
end
``` 

!!! warning 
    `.Connect` can only be called in the `.OnStart` function. Attempting to call this function anywhere else will cause an error.


### `AuroraScriptObject.Subscribe(self: AuroraScriptObject, topic: string, functionName: string) -> string`

This method allows you to subcribe to a published value in the Behavior with a certain `topic`. The `functionName` must be the name of a function in the Behavior.

The subcribed function will always have 2 arguments by default, the self `AuroraScriptObject` and the bound `Instance`. Additional arguments will come after. Calling this method will return the value of the `topic` parameter.

!!! warning
    This function can only be called in `.OnStart`.

### `AuroraScriptObject.Publish(self: AuroraScriptObject, topic: string, ...any) -> any`

This method allows you to publish a value in the Behavior with a certain `topic`. The last argument given to this method will be returned as a value.

!!! warning 
    This function cannot be called in `.OnStart`.

```lua

local RunService = game:GetService("RunService")

type AuroraScriptObject = {
    Instance: Instance,
    Frame: number,
    LODLevel: number,
    Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
    Subscribe: (self: AuroraScriptObject, topic: string, functionName: string) -> string,
    Publish: (self: AuroraScriptObject, topic: string, ...any) -> any,
    SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
    Delay: (self: AuroraScriptObject, amount: number, functionName: string) -> string,
    SetMaxFrequency: (self: AuroraScriptObject, frequency: number) -> number
}

function Behavior.OnStart(self: AuroraScriptObject)
    self:Subscribe("Test", "GetPublishedValue")
    self:Connect(RunService.Heartbeat, "OnHeartbeat")
end

function Behavior.OnHeartbeat(self: AuroraScriptObject, deltaTime: number)
    self:Publish("Test", "Hello!")
end

function Behavior.GetPublishedValue(self: AuroraScriptObject, boundInstance: Instance, recievedValue: any)
    print(boundInstance, recievedValue) -- Instance, Hello!
end
```

### `AuroraScriptObject.SendMessage(self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any`

This method allows you to communicate with other Behaviors in a world. When sending a message, you must specify the `Instance` that a Behavior is bound to, the name of the Behavior, and the name of the function you want to send this message to.
Then, you can add additional values as arguments to send to the function.

#### Sending and Recieving Messages

Let's assume we have 2 `AuroraScript`s in the `workspace` called: "Test_1", and "Test_2".
And let's also assume we have 2 `Part`s these Behavior `AuroraScript`s are bound to.

Our system would be like this:

Test_1 --> Part<br>
Test_2 --> Part_2

If we wanted to send a message from Test_1 to Test_2, how would we do it?
This is where `:SendMessage()` comes in. Using this method, we can send and recieve messages across different Behaviors.

#### Test_1:

```lua
local RunService = game:GetService("RunService")

type AuroraScriptObject = {
    Instance: Instance,
    Frame: number,
    LODLevel: number,
    Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
    Subscribe: (self: AuroraScriptObject, topic: string, functionName: string) -> string,
    Publish: (self: AuroraScriptObject, topic: string, ...any) -> any,
    SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
    Delay: (self: AuroraScriptObject, amount: number, functionName: string) -> string,
    SetMaxFrequency: (self: AuroraScriptObject, frequency: number) -> number
}

function Behavior.OnStart(self: AuroraScriptObject)
    self:Connect(RunService.Heartbeat, "OnHeartbeat")
end

function Behavior.OnHeartbeat(self: AuroraScriptObject, deltaTime: number)
    self:SendMessage(game.Workspace.Part_2, "Test_2", "Test", "Hello!")
end
```

#### Test_2

```lua
local RunService = game:GetService("RunService")

type AuroraScriptObject = {
    Instance: Instance,
    Frame: number,
    LODLevel: number,
    Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
    Subscribe: (self: AuroraScriptObject, topic: string, functionName: string) -> string,
    Publish: (self: AuroraScriptObject, topic: string, ...any) -> any,
    SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
    Delay: (self: AuroraScriptObject, amount: number, functionName: string) -> string,
    SetMaxFrequency: (self: AuroraScriptObject, frequency: number) -> number
}

function Behavior.OnStart(self: AuroraScriptObject)
    self:Connect(RunService.Heartbeat, "OnHeartbeat")
end

function Behavior.OnHeartbeat(self: AuroraScriptObject, deltaTime: number) end

function Behavior.OnMessageTest(self: AuroraScriptObject, recievedMessage: string)
    warn(recievedMessage) -- "Hello!"
end
```

!!! warning
    All functions recieving a message must have a name that starts with "OnMessage". For example, if we call the `:SendMessage()` with a `functionName` argument called "Test", then the function name must be "OnMessageTest" on the recieving Behavior.

### `AuroraScriptObject.Delay(self: AuroraScriptObject, amount: number, functionName: string): string`

This method allows you to run a function in the Behavior with a certain amount of delay. (In seconds)

### `AuroraScriptObject.SetMaxFrequency(self: AuroraScriptObject, frequency: number): number`

This method allows you to set the maximum amount of frequency the `AuroraScriptObject` can run with. Must be a value between 1 to 60.

-----

# AuroraScriptService

This service is meant to manage and communicate with Behavior `AuroraScript`s.

## Methods

### `AuroraScriptService:SendMessage(instance: Instance, behaviorName: string, functionName: string, ...: any): ()`

This method works exactly the same as the `AuroraScriptObject:SendMessage()`, except it is used outside of Behaviors to send messages to Behaviors. The same rules apply.

### `AuroraScriptService:getBehaviors(): {AuroraScript}`

This method returns a table containing all of the Behaviors in the place.

### `AuroraScriptService:getBehaviorsForInstance(instance: Instance): {AuroraScript}`

This method returns a table containing all of the Behaviors bound to the specified `Instance`.

### `AuroraScriptService:getInstancesForBehavior(behavior: AuroraScript): {Instance}`

This method returns a table containing all of the `Instance`s that the specified Behavior is bound to.

### `AuroraScriptService:FindBindings(instance: Instance): { [string]: any }`

This method returns a table containing all of the `AuroraScriptObject`s from the Behaviors bound to an `Instance`.
Can only be used within `AuroraScript`s.

### `AuroraScriptService:FindBinding(instance: Instance, scriptName: string): AuroraScriptObject`

This method returns the `AuroraScriptObject` from the target Behavior (with the `scriptName`) bound to an `Instance`.
Can only be used within `AuroraScript`s.

### `AuroraScriptService:GetLocalFrameId(): number`

This method returns a number that presumably indicates the current frame of the world.

-----