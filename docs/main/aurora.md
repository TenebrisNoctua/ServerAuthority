# Services and Behaviors

!!! warning
    The services and `Instance`s mentioned in this page cannot be accessed without configuring certain flags.
    If you have enabled the flags from the previous page, you may continue. However, if you've just enrolled into the program without changing the flags, then it is not recommended for you to read this page.

In the previous pages, I've explained the default behavior of the Server Authority system. While this behavior is mostly sufficient, there are cases where you might need more control. For example, what if you want to opt-out of the prediction system manually on certain `Part`s?
Or another example, what if you want to get a list of all currently predicted `Instance`s?

Fortunately, in the Server Authority system, there exists new services and `Instance`s allowing you to control the system much greater than ever before.

-----

# AuroraService

This is the first new main service in Server Authority, where you're able to manually configure the prediction and physics. Its methods and events are listed below.

## Methods

### `AuroraService:StartPrediction(target: Instance): ()`

This method allows you to start prediction manually on the target `Instance`. When it is used, the engine will start predicting the physics of the said `Instance`.

### `AuroraService:StopPrediction(target: Instance): ()`

This method stops the prediction on the target `Instance`. This method also allows you to restore the network ownership behavior of the previous system.

### `AuroraService:GetPredictedInstances(): {Instance}`

This method returns a table which contains the current predicted `Instance`s by the Server Authority system.

### `AuroraService:IsPredicted(target: Instance): boolean`

This method returns a `boolean`, indicating if the provided `Instance` is being predicted by the engine or not.

### `AuroraService:GetServerView(target: Instance): Instance`

This method will return a completely new `Instance` that most likely shows how the server views the target `Instance`. 
Parenting it to a container such as `Workspace` will show the exact same properties of the target `Instance`.

### `AuroraService:UpdateProperties(target: Instance): ()`

This method most likely is used to manually update an `Instance`'s properties, while its being predicted.

### `AuroraService:ShowDebugVisaulizer(state: boolean): ()`

This method, when called on the server in a play-test, shows you information about the current state of the prediction system.
It contains many elements such as the prediction success rate on both `Instance`s and scripts. Or how many predicted `Instance`s there are, live.

### `AuroraService:GetWorldStepId(): number`

This method returns the current world step id, which is a number.
Step id is presumably used for rolling back to a previous state of the prediction.

### `AuroraService:GetRemoteWorldStepId(): number`

There's not much information about this method, other than it always returns 0.

### `AuroraService:StepPhysics(worldsteps: number, instances: {Instance}): ()`

This method steps the physics of the given `Instance`s by the given `worldsteps` amount. 
All the given `Instance`s must be a `BasePart`. If an `Instance` is not a `BasePart` in this table, then it will be ignored for the physics step. 

### `AuroraService:SetIncomingReplicationLag(seconds: number): ()`

This method allows you to set the incoming replication lag. It may be used while debugging.

### Input Recording

Currently unsure of what input recording does, as all of the methods described below seemingly do nothing.
Will update when more information has been found.

### `AuroraService:StartInputRecording(): ()`

This method allows you to start recording input.

### `AuroraService:StopInputRecording(): ()`

This method allows you to stop recording input.

### `AuroraService:PlayInputRecording(): ()`

This method allows you to play the recorded input.

### `AuroraService:SetPropertyIsInput(target: Instance, propertyName: string, isInput: bool): ()`

This method presumably allows you to set `isInput` to a property of a target `Instance`.

## Events

!!! warning 
    Attempting to connect to these events outside of a Behavior `AuroraScript` will error.

### `AuroraService.Step`

This event is fired when `AuroraService:StepPhysics()` has been called.

### `AuroraService.FixedRateTick(deltaTime: number, worldStepId: number)`

This event is the same as `RunService.FixedHeartbeat`, but provides an additional parameter called `worldStepId`.

### `AuroraService.Misprediction(worldStepId: number, mispredictedInstances: {Instance})`

This event is the same as `RunService.Misprediction`, and is fired when a misprediction occurs on certain `Instance`(s). This happens when there's a mismatch between the client and the server's position on an `Instance`'s physics and movement.

### `AuroraService.Rollback(worldStepId: number)`

This event is fired when a rollback occurs. Rollbacks generally occur after a misprediction, which causes an `Instance` to be reverted back to its previous state.

!!! info 
    These are all of the current events and the methods of AuroraService. If more information gets found about them, this documentation will be updated.

-----

# AuroraScript

!!! info
    It is most likely that Roblox will be changing the name "AuroraScript" to "BehaviorScript" in the near future, to more accurately represent what it does.

This is a new unique `Script` object that allows you to connect to the various events of `AuroraService`, and define Behaviors that operate on them. 

Unlike the other `Script` types, `AuroraScript` is pretty limited in terms of what is possible to do with it. This is due to its purpose mainly being related to server authority and prediction, and not the main game logic. Here are the limitions that I'm currently aware of:

* All `AuroraScript`s in the same location (e.g, `workspace`) must have a different name. Otherwise, one of them will not run.
* Certain APIs and global libraries do not work in the environment of an `AuroraScript`. The ones I'm aware of at the moment are:
    * All methods of the `task` library (Except for `task.cancel()`).
    * `RunService:IsClient()` and `RunService:IsServer()`.
    * `coroutine.yield()` (Behaviors cannot yield.)

Besides the limitations, generally `AuroraScript`s should always be parented to a location that can be both accessed from the client and the server, such as `ReplicatedStorage`. If you parent an `AuroraScript` to a location such as `ServerScriptStorage`, this will cause it to only run on the server, and not the client. This defeats the whole purpose of `AuroraScript`s, as they run both on the server and the client simultaneously.

## Methods

### `AuroraScript:AddTo(instance: Instance): ()`

This method binds the Behavior `AuroraScript` to the specified `Instance`.
This then calls the (if defined) `.OnStart` method of the Behavior.

Behaviors can also be bound manually, without the need of this method.
With the Server Authority feature enabled, the Properties widget gain a new section for every `Instance`, called: "Behaviors".
Clicking the "+" button on this section will allow you to find and bind any Behavior on an `Instance`.

### `AuroraScript:RemoveFrom(instance: Instance): ()`

This method removes the Behavior `AuroraScript` from the specified `Instance`.

### `AuroraScript:IsOnInstance(instance: Instance): ()`

This method allows you to check if the Behavior `AuroraScript` is on the specified `Instance`.

### `AuroraScript:SignalFired(instance: Instance, topic: string): RBXScriptSignal`

This method allows you to create a Signal that is fired when the `AuroraScriptObject` of the `AuroraScript` publishes a value with a topic.

## Behavior

Every `AuroraScript` comes with a global data type called `Behavior`, which can be accessed everywhere from the script.
This Behavior allows you to connect to certain methods and events that allow you to (presumably) configure how the prediction on a certain `Instance` works.

!!! info
    The term "Behavior" actually represents an `AuroraScript`. `AuroraScript`s do not have Behaviors, they are the Behaviors themselves. In certain places, for example the "Behaviors" section in the Properties window, this is more apparent.

!!! warning
    Every Behavior must be bound to an `Instance` for them to work.

There are certain methods you have to define in the Behavior for it to start working. They are listed below.

### `Behavior.OnStart(self: AuroraScriptObject): ()`

This is a special method that you can define in the Behavior, which runs when the Behavior has been started. It allows you to connect to various events of the `AuroraService`, and do some other things.

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

## AuroraScriptObject

This is the object given to the every function defined in the Behavior. It has certain properties and methods that allows you to do things such as manual prediction, sending messages across Behaviors, and more. 

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

This method allows you to connect to a certain given `RBXScriptSignal`. It is mostly used to connecting to certain `AuroraService` events.
The `functionName` argument must be the name of a function in the Behavior.

```lua
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
    self:Connect(AuroraService.FixedRateTick, "Predict")
end

function Behavior.Predict(self: AuroraScriptObject, deltaTime: number, worldStepId: number) -- This method will get called whenever the .FixedRateTick event fires.
    print(self, deltaTime, worldStepId)
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
    self:Connect(AuroraService.FixedRateTick, "OnFixedRateTick")
end

function Behavior.OnFixedRateTick(self: AuroraScriptObject, deltaTime: number, worldStepId: number)
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
local AuroraService = game:GetService("AuroraService")

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
    self:Connect(AuroraService.FixedRateTick, "OnFixedRateTick")
end

function Behavior.OnFixedRateTick(self: AuroraScriptObject, deltaTime: number, worldStepId: number)
    self:SendMessage(game.Workspace.Part_2, "Test_2", "Test", "Hello!")
end
```

#### Test_2

```lua
local AuroraService = game:GetService("AuroraService")

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
    self:Connect(AuroraService.FixedRateTick, "OnFixedRateTick")
end

function Behavior.OnFixedRateTick(self: AuroraScriptObject, deltaTime: number, worldStepId: number) end

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

This service is meant to manage and communicate with Behavior `AuroraScript`s. Unlike AuroraService however, it does not have special properties or events, just methods.

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