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
            <ul>
                <li><a href="#part-physics">Part Physics</a></li>
                <li><a href="#character-physics">Character Phsyics</a></li>
            </ul>
        </div>
        <h3 class="api-summary-section-h3"><a href="#auroraservice">AuroraService</a></h3>
        <div class="api-summary-section-list">
            <ul>
                <li><a href="#predictions">Predictions</a></li>
                <li><a href="#methods">Methods</a></li>
                <li><a href="#events">Events</a></li>
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

Up until this point, I explained the default behavior of the server authority system on all parts and characters. However, this default behavior can be changed. For example, it is possible to create a system to exclude certain parts and characters from the prediction, so the network ownership system can be applied. And this is the service that allows you to do so. It is made out of methods and events that allows you to manually configure certain parts of the server authority system.

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
    These events will not work unless connected to the Behavior of an `AuroraScript`.
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








