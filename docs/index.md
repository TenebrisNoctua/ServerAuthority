---
  hide:
    - navigation
    - toc
---

<div class="api-summary-list">
    <h3 class="api-summary-list-h3">Navigation</h3>
    <div class="api-summary-section">
        <h3 class="api-summary-section-h3"><a href="#getting-started">Getting Started</a></h3>
        <h3 class="api-summary-section-h3"><a href="#how-to-access">How to Access?</a></h3>
        <h3 class="api-summary-section-h3"><a href="#server-authority">Server Authority</a></h3>
        <div class="api-summary-section-list">
            <ul style="list-style-type: none;">
                <li><a href="#part-physics">Part Physics</a></li>
                <li><a href="#character-physics">Character Phsyics</a></li>
            </ul>
        </div>
        <h3 class="api-summary-section-h3" style="padding-top: 4px;"><a href="#auroraservice">AuroraService</a></h3>
        <div class="api-summary-section-list">
            <ul style="list-style-type: none;">
                <li><a href="#predictions">Predictions</a></li>
                <li><a href="#methods">Methods</a></li>
                <li><a href="#events">Events</a></li>
            </ul>
        </div>
        <h3 class="api-summary-section-h3" style="padding-top: 4px;"><a href="#aurorascript">AuroraScript</a></h3>
        <div class="api-summary-section-list">
            <ul style="list-style-type: none;">
                <li><a href="#methods">Methods</a></li>
                <li><a href="#behavior">Behavior</a></li>
                <li><a href="#aurorascriptobject">AuroraScriptObject</a></li>
            </ul>
        </div>
    </div>
</div>

# Getting Started

Hello! This post is a detailed documentation about the information I *(and some other people, credited at the bottom)* have found on the new server authority system that Roblox has been developing. This is a new system developed to achieve a more secure game state.
Now, almost all of the information here has been gained through reverse-engineering methods. They will not be discussed here, however.

With all that being said, take every information presented here with a grain of salt, they might be inaccurate, and this feature (at the time of writing) isn't even in beta yet. So please be careful. If you have any additional information that you can present about this feature, please let me know with a message on [Twitter](https://twitter.com/TenebrisNoctua).

Now let's begin.

-----

# How to Access?

If you want to access everything that will be mentioned in the later sections, you must enable certain flags in Roblox Studio.
This documentation will not mention on how you can edit flags in Studio, as there are plenty of resources out there that you can search and learn from on how. And it is simply not my responsibility.

```
DubugAuroraDefaultConfig
DebugHashTableMetrics
NextGenReplicatorEnabledRead3
NextGenReplicatorEnabledWrite4
DebugEnableAuroraService
StudioAuroraEditorSupport
```

Settings these flags to `True` should grant you access to the features mentioned forward.

!!! warning 
    Enabling these flags *MAY CORRUPT YOUR FILES OR PLACES*. Publishing a place with `AuroraScript`s inside may cause your place to never be launched on the client. And you may not be able to access your Team-Create sessions. *BE CAUTIOUS.*

-----

# Server Authority

To give a short summary on what Server Authority is, it is when the server becomes the single source of truth for game actions, logic, and data.
This basically means that the server will dictate how the data and logic such as physics *should* work within your game.
This does cause responsiveness and latency issues however, but those issues are mostly solvable with a prediction and rollback system, such as the one Roblox is utilizing, which I will mention further in later sections.

To enable the server authority system Roblox has implemented, after all the flags above have been enabled, you must go into
`Workspace > Server Authority > AuthorityMode` and set it to `Server`.

!!! warning
    In order to access the features mentioned in the later sections, this step is necessary to do. Otherwise, the features will simply not work.

After this step, server authority should be enabled within your place.

## Part Physics

Enabling this feature completely changes the behavior of all unanchored `Part`s in the `Workspace`. Before; the physics calculation of a `Part` was handled by both the server and the clients in a place. When a client came close to a `Part`, the network ownership would automatically shift from the server to the client, so the client could take the burden of calculating the physics for that `Part`.

However, now, with server authority enabled; this network ownership behavior is completely bypassed, as even if a client comes close to a `Part`, the server will always be the one calculating the physics of that `Part`.

## Character Physics

Enabling this feature also completely changes how characters work and behave within your place. Before; the client, because of network ownership, had full control over their character. This allowed them to change certain properties such as velocity, position, rotation, and many others, to their liking. This, of course, caused many security issues. Using exploits, the client would be able to give themselves an unfair advantage in gameplay. This gave the rise of many exploiting issues such as speed-hacking, fly-hacking, no-clipping, teleporting, and many others, inside popular places on the platform.

However, now, with server authority enabled; mostly any kind of these issues are almost impossible to happen, if not impossible.
Because the server is now predicting where your character should be, it is incredibly hard to do character based exploits now.<br>
In fact, to verify this, you can easily do some tests yourself. Hitting "Play" (or "Test" in the Next Gen Studio Ribbon), should give you the chance to test pretty much anything you want. Be warned however, you will notice some weirdness with animations. This is mostly an animations issue with the new characters, and should be resolved soon.

A good example to test the new system would be to create a new `Part` instance in the `Workspace` on the client, and standing on it.
You would quickly notice that your character is constantly being teleported to the bottom, like the part is not actually there. And if you were to switch to the server view, you would see that the character is standing at the bottom without any issues.  

Or, you can create a wall infront of the character on the server, and delete it on the client. Then, you can try to pass through the place where the wall was prior to deletion. You would quickly notice that the server is constantly teleporting you back, like the wall is still there.

!!! info
    If you have any additional information about character and part physics prediction, please let me know!

-----

# AuroraService

Up until this point, I explained the default behavior of the server authority system on all parts and characters. However, this default behavior can be changed. For example, it is possible to create a system to exclude certain parts and characters from the prediction, so the network ownership system can be applied. And this is the service that allows you to do exactly that. It is made out of methods and events that allows you to manually configure certain parts of the server authority system.

But first, let's talk about predictions in more detail.

## Predictions

The prediction system is what allows the server authority to work. It determines where a part should and should not be. When a `Part` falls down for example, it calculates where that `Part` should land, and should not. It is the same for characters, like it has been described in the previous section.

This prediction system can be configured however, with certain methods exposed through this service. They are described in detail below.

## Methods

### `AuroraService:StartPrediction(target: Instance): ()`

This method allows you to start prediction manually on the target `Instance`. When it is used, the server will start predicting the physics of the said `Instance`, if it's a `BasePart`. Additionally, this method completely bypasses network ownerships.

### `AuroraService:StopPrediction(target: Instance): ()`

This method stops the prediction on the target `Instance`. This method allows you to restore the network ownership behavior of the previous system.

### `AuroraService:GetPredictedInstances(): {Instance}`

This method returns a table which contains the current predicted `Instance`s by the server authority system.

### `AuroraService:IsPredicted(target: Instance): boolean`

This method returns a `boolean`, indicating if the provided `Instance` is being predicted by the server or not.

!!! info
    Like it has been mentioned briefly in the above sections, every `Part` or character in `Workspace` will automatically have their physics predicted by the server, as long as they are not anchored. Anchoring a `Part` will automatically remove it from being predicted. (Or to remove it manually, you can call `AuroraService:StopPrediction()` on that said `Part`.)

### `AuroraService:GetServerView(target: Instance): Instance`

This method will return a completely new `Instance` that most likely shows how the server views the target `Instance`. 
Parenting it to a container such as `Workspace` will show the exact same properties of the target `Instance`.

### `AuroraService:UpdateProperties(target: Instance): ()`

This method most likely is used to manually update an `Instance`'s properties, while its being predicted.
Not sure about what it truly does or what it's really useful for, as all properties should automatically get updated while being predicted.

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

*(This method is also important for `AuroraScript`s.)*

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

This method presumably allows you to set a certain property to input.

## Events

!!! warning 
    Attempting to connect to these events outside of the Behavior of an `AuroraScript` will error.

### `AuroraService.Step`

This event is fired when `AuroraService:StepPhysics()` has been called.

### `AuroraService.FixedRateTick(deltaTime: number, worldStepId: number)`

This event is fired (presumably) when the `worldStepId` changes. 
`deltaTime` should be a constant number, while the `worldStepId` increases.

### `AuroraService.Misprediction(worldStepId: number, mispredictedInstances: {Instance})`

This event is fired when a misprediction occurs on certain `Instance`(s).
Currently unsure of how this event can be fired.

### `AuroraService.Rollback(worldStepId: number)`

This event is (presumably) fired when a rollback occurs.
Currently unsure of how this event can be fired.

!!! info 
    These are all of the current events and the methods of AuroraService. If more information gets found about them, this documentation will be updated.

-----

# AuroraScript

This is a new `Script` object that allows you to connect to the various events of `AuroraService`, and define Behaviors that operate on them. This `Instance` runs both on the Server and Client context. *(This means the code written in this `Instance` will be ran on both the Server and the Client.)*

!!! warning
    All created `AuroraScript`s in the same location (e.g, `workspace`) must have a different name. Otherwise, one of them will not run.

## Methods

### `AuroraScript:AddTo(instance: Instance): ()`

This method adds the Behavior of the `AuroraScript` to the specified `Instance`.
This then calls the (if defined) `.OnStart` method of the Behavior.

Behaviors can also be added manually, without the need of this method.
With the Server Authority feature enabled, the Properties widget gain a new section for every `Instance`, called: "Behaviors".
Clicking the "+" button on this section will allow you to find and add any `AuroraScript`'s Behavior on an `Instance`.

### `AuroraScript:RemoveFrom(instance: Instance): ()`

This method removes the Behavior of the `AuroraScript` from the specified `Instance`.

### `AuroraScript:IsOnInstance(instance: Instance): ()`

This method allows you to check if the Behavior of the `AuroraScript` is on the specified `Instance`.

## Behavior

Every `AuroraScript` comes with a global data type called `Behavior`, which can be accessed everywhere from the script.
This Behavior allows you to connect to certain methods and events that allow you to (presumably) configure how the prediction on a certain `Instance` works.

!!! warning
    Every `AuroraScript`'s Behavior must be bound to an `Instance` for their Behavior to work.

There are certain methods you have to define in the Behavior for it to start working. They are listed below.

### `Behavior.OnStart(self: AuroraScriptObject): ()`

This is a special method that you can define in the Behavior, which runs when the Behavior has been bound to an `Instance`. It allows you to connect to various events of the `AuroraService`, and do some other things.

```luau
function Behavior.OnStart(self: AuroraScriptObject) -- This method is called when :AddTo() is called on an Instance.
	  print(self)
end
```

You might've noticed that `.OnStart` has a parameter called `self`, which is an `AuroraScriptObject`. This is the object that is given to every function that you define in the Behavior. It will be explained in detail in the next section.

### `Behavior.OnStop(self: AuroraScriptObject): ()`

This is a special method that you can define in the Behavior, which runs when the Behavior has been removed from the bound `Instance`.

```luau
function Behavior.OnStop(self: AuroraScriptObject) -- This method is called when :AddTo() is called on an Instance.
	  print(self)
end
```

### `Behavior.DeclareField(fieldName: string, _: { Type: "boolean" | "cframe" | "color3" | "enum" | "instance" | "number" | "random" | "vector2" | "vector3" }): ()`

This method allows you to define a field with a certain type in the `AuroraScriptObject`. Not sure what it is used for at the moment.

## AuroraScriptObject

This is the object given to the every function defined in the Behavior. It has certain properties and methods that allows you to do things such as manual prediction, sending messages across Behaviors, and more. 

```luau
type AuroraScriptObject = {
    Instance: Instance,
    Frame: number,
    LODLevel: number,
    Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
    Subscribe: (self: AuroraScriptObject, name: string, functionName: string) -> string,
    Publish: (self: AuroraScriptObject, name: string, ...any) -> any,
    SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
    Delay: (self: AuroraScriptObject, functionName: string, ...any) -> ...any,
    SetMaxFrequency: (self: AuroraScriptObject, frequency: number) -> number
}

function Behavior.OnStart(self: AuroraScriptObject)
    print(self) -- prints the AuroraScriptObject
end
```

Now be warned, unlike any other object which is found in the Roblox API, `AuroraScriptObject` does not reflect the properties found in the API dump. Even when you print this object, it does not give you any property name that allows you to directly access a certain value.

An example:

```luau
{
    [Attached Instance] = Part,
	  [Behavior] = AuroraScript,
    [Frame ID] = 509,
    [self] =  ▶ {...}
}
```

This is a detailed string that shows you certain information about the `AuroraScriptObject`, such as which frame it currently is on, the Behavior it is from, the bound instance, and lastly, the `self` table, which is used to show you the declared fields made with `Behavior.DeclareField()` function.

Unfortunately, you cannot access these properties directly, and most likely this information is only shown for debugging purposes.
In the previous example however, you might have noticed that I've added an `AuroraScriptObject` type which contains information about all of the currently known and actually accessible methods and properties of the `AuroraScriptObject`. 
I have found these properties using reverse-engineering. They are described in detail below.

### Properties

### `AuroraScriptObject.Instance`

This is the `Instance` that the Behavior is added to.

### `AuroraScriptObject.Frame`

The current Frame the world is on. Most likely changes after Heartbeat or PreAnimation.

### `AuroraScriptObject.LODLevel`

The level of distance level of the Behavior. Not sure what it is supposed to be used for at the moment.

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
	  Subscribe: (self: AuroraScriptObject, name: string, functionName: string) -> string,
	  Publish: (self: AuroraScriptObject, name: string, ...any) -> any,
	  SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
	  Delay: (self: AuroraScriptObject, functionName: string, ...any) -> ...any,
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


### `AuroraScriptObject.Subscribe(self: AuroraScriptObject, name: string, functionName: string) -> string`

This method allows you to subcribe to a published value in the Behavior with a certain `name`. The `functionName` must be the name of a function in the Behavior.
The subcribed function will always have 2 arguments by default, the self `AuroraScriptObject` and the bound `Instance`. Additional arguments will come after. Calling this method will return the value of the `name` parameter.

!!! warning
    This function can only be called in `.OnStart`.

### `AuroraScriptObject.Publish(self: AuroraScriptObject, name: string, ...any) -> any`

This method allows you to publish a value in the Behavior with a certain `name`. The last argument given to this method will be returned as a value.

!!! warning 
    This function cannot be called in `.OnStart`.


```lua
type AuroraScriptObject = {
	  Instance: Instance,
	  Frame: number,
	  LODLevel: number,
	  Connect: (self: AuroraScriptObject, signal: RBXScriptSignal, functionName: string) -> RBXScriptConnection,
	  Subscribe: (self: AuroraScriptObject, name: string, functionName: string) -> string,
	  Publish: (self: AuroraScriptObject, name: string, ...any) -> any,
	  SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
	  Delay: (self: AuroraScriptObject, functionName: string, ...any) -> ...any,
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
And let's also assume we have 2 `Part`s the Behavior of these `AuroraScript`s are bound to.

Our system would be like this:

Test_1 --> Part
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
	  Subscribe: (self: AuroraScriptObject, name: string, functionName: string) -> string,
	  Publish: (self: AuroraScriptObject, name: string, ...any) -> any,
	  SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
	  Delay: (self: AuroraScriptObject, functionName: string, ...any) -> ...any,
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
	  Subscribe: (self: AuroraScriptObject, name: string, functionName: string) -> string,
	  Publish: (self: AuroraScriptObject, name: string, ...any) -> any,
	  SendMessage: (self: AuroraScriptObject, boundInstance: Instance, behaviorName: string, functionName: string, ...any) -> ...any,
	  Delay: (self: AuroraScriptObject, functionName: string, ...any) -> ...any,
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
    All functions recieving a message must start with "OnMessage". For example, if we call the `:SendMessage()` with a `functionName` argument called "Test", then the function name must be "OnMessageTest" on the recieving Behavior.

##### *Unfinished, more coming soon.*



