# Getting Started

Hello. This is a detailed documentation about the new server authority system that Roblox has showcased recently.
This is a new system aiming to achieve a better physics simulation, while also providing better security.<br>
This documentation will try to give you almost all of the available information about this new system. However, not all information may be accurate, as this system is still a work-in-progress.

With all that being said, this documentation may recieve updates whenever there's a new feature added, or a feature removed. If you want to stay up-to-date, make sure to visit this documentation again later, and if you have any additional information that you can provide about this system, let me know with a message on my [Twitter](https://twitter.com/TenebrisNoctua).

Let's begin.

-----

# How to Access?

If you want to access everything that will be mentioned in later sections, you must either:

**(A) Enroll in the Early Access Program for the Server Authority Core API.**<br>
**(B) Enable certain flags in Roblox Studio.\***

<h6>*: This documentation will not mention on how you can edit flags in Roblox Studio, as it is not my responsibility.</h6>

## Enrolling to the Early Access Program for the Server Authority Core API

This is the simplest way you can gain access to the new features that will be mentioned in this documentation. You can send an application [here](http://rblx.co/server-authority) to apply for the program. 

If you're accepted, go to your [account settings](https://www.roblox.com/my/account), and scroll down to the "Early Access Programs" section.
Then, select the *"Early Access: Server Authority Core API Early Access"* program. This will now enroll you to the program, and following the steps below, you can access the features.

### If using vanilla Studio

Restart your Studio. Upon the restart, there will be an "Update Studio" button on the top right. Clicking this button will automatically update your Studio and install the version meant for the program. After this, you should be good to go.

### If using a custom bootstrapper

If your custom bootstrapper application to launch Studio does not have a built-in system for automatically opening the version meant for the early access program, then follow the steps below.

Go to your [creator page](https://create.roblox.com). And re-download the Studio application. After install, it should automatically open. Click on the "Update Studio" button on the top right. This will automatically update your Studio and install the version meant for the program.
After this, you should be good to go.

Do not forget to launch the Studio application through the versions folder for the Studio application. Using your custom bootstrapper may not work.

## Enabling certain flags in Roblox Studio

If you're not enrolled in the early access program, you can set the flags below to `True` to access the features.

```
DebugAuroraDefaultConfig
DebugHashTableMetrics
NextGenReplicatorEnabledRead3
NextGenReplicatorEnabledWrite4
DebugEnableAuroraService
StudioAuroraEditorSupport
```

!!! warning
    Modifying Roblox Studio flags *is not recommended*, as it can corrupt your places, or even the Studio app itself. I heavily recommend you to try to gain access through the early access program instead. **Regardless, I will not accept any responsibility that comes from you modifying your Studio application.**

-----

# Server Authority

To give a short summary on what Server Authority is, it is when the server becomes the single source of truth for game actions, logic, and data. This basically means that the server will dictate how the data and logic such as physics will work within your game.

To fully enable this feature in Roblox Studio, you must go into the `Workspace` properties and change some values. They are listed below:

* `Server Authority > AuthorityMode`: `Server`
* `Behavior > PhysicsSteppingMethod`: `Fixed`

!!! warning
    Setting the `AuthorityMode` to `Server` will automatically change certain other workspace properties. They are listed below:

    * `StreamingEnabled`: `true`
    * `SignalBehavior`: `Deferred`

    These properties cannot be changed while the `AuthorityMode` is set to `Server`.

## Part Physics

Enabling this feature completely changes the behavior of all unanchored `Part`s in the `Workspace`. Before; the physics calculation of a `Part` was handled by both the server and the clients in a place. When a client came close to a `Part`, the network ownership would automatically shift from the server to the client, so the client could take the burden of calculating the physics for that `Part`.

However, now, with server authority enabled; this network ownership behavior is disabled by default. The server and the client now calculates the physics for the said part at the same time, while the server being the source of truth.

## Character Physics

Before, because of network ownership, the client had full control over their character. This allowed them to change certain properties of their character such as velocity, position, rotation, and many others to their liking. This of course, caused many security issues. Using exploits, the client would be able to give themselves an unfair advantage in gameplay by changing these properties. This gave the rise of many exploiting issues such as speed-hacking, fly-hacking, no-clipping, teleporting, and many others inside popular places on the platform.

The issues are not only limited to exploiting. In many competitive games, such as racing, the cars would most of the time be misaligned, or their position would constantly jitter, or look very different. That is because of client calculated physics of the previous system. Because the network ownership of each car is set to each client, rather than the server, the car positions would end up different than expected.

With the new server authority system, most of these issues, if not all of them, are automatically resolved.

An example from the first case, where a client would modify their character to gain an unfair advantage such as speedhacking, would no longer be possible, as the client no longer holds ownership of their character, and all that is given to the server are inputs.

For the second case, since the server now calculates the positions of the cars, they now move more consistently. Maneuvers, drifts, hits, become more accurate on each client, and the system works more reliably.

If you want to see this system in action, you can easily do some tests yourself. Hitting "Play" (or "Test" in the Next Gen Studio Ribbon), should give you the chance to test pretty much anything you want.

A good way to test the new system would be to create a wall infront of the character on the server, and then delete it on the client. Then, you can try to pass through the place where the wall was prior to deletion. You will quickly notice that you're unable to move past the place where the wall once was, like the wall is still there, but invisible.

-----

# The Prediction System

!!! warning
    Information in this section may be inaccurate in certain places, if you have any valid corrections, please don't hesitate to reach out to me. 

The server authority system is powered by a netcode system with predictions and rollback. Under this system, all characters and parts have their network ownership set to the server. By default, the server calculates the physics or movement of all of the parts and characters, and the only thing client sends to the server are the inputs for the movement. Normally, a system like this would result in choppy physics and movement, and it wouldn't feel responsive. This was the main issue with implementing a server authoritative system. However, to solve this issue, a prediction system was implemented.

On the client, the character is simulated just like it's on the server. However, for the character movement to feel responsive and smooth, it is simulated a little bit ahead of the server. This is called "prediction". The client "predicts" where the character might be in the next second ahead of the server, to make sure the experience feels a lot smoother and responsive, while the server dictates where the character actually will be. However, if the client and server disagrees on where the character will be (a misprediction), then a rollback occurs. This rollback allows the client to resimulate the physics and the state of the character, and to catch up where it is supposed to be.

Mispredictions are normal and expected. They should be small and the resulting correction should be imperceptible to clients.

!!! example
    Your client thinks you’ve moved forward. However, the server registered that you were hit by a stun grenade and can’t move for a few  seconds. The client and server now have different states.

This divergence is called a misprediction. It can occur for several reasons: the network latency has shifted, other players acted in ways the client didn’t anticipate, the experience runs certain logic exclusively on the Server, etc. While you can’t prevent every misprediction, you can keep gameplay feeling smooth and responsive by using the right techniques.

Basically, the client can be slightly incorrect and make a "misprediction." Things like latency variance and other players inputs can cause the client to be slightly incorrect.

After misprediction, the client rolls back to the authoritative update by reverting its current state and time. Then, the client resimulates automatically after rollback to speed back to its predicted frame. The number of frames to resimulate is based on the latency between the client and the server. The client tries to stay far enough ahead of the server so that its own inputs arrive on the server just in time to be processed on the frame the user intended to perform them.

!!! example 
    Let’s say there is 100ms of latency between the client and server, and the experience is a 60Hz game.

Each frame is 1/60s (or 16.67ms). Since 100ms of latency is equivalent to ~6 frames (100 ms divided by 16.67 ms/frame), we know the client will be 6 frames ahead of the server. This means that, when the client detects it made a misprediction, it will rollback to the server's state and then resimulate frames ahead. In general, the player should hardly notice this.

-----

# The Input Action System

The Input Action System (IAS) is a major part of the server authority system. It allows you to reliably transfer inputs from client to the server, while allowing you to customize them easily. I will not be explaining this system and how it works however. You can check out the main post talking about it through [here](https://devforum.roblox.com/t/client-beta-input-action-system-is-now-available-to-publish-in-experiences/3890979).

Player inputs like joystick movement and button presses are sent from client to server using [`InputAction`](https://create.roblox.com/docs/reference/engine/classes/InputAction)s. You have to use `InputAction` for all inputs that affect the core game simulation, because they are the only client authoritative data in the core rollback system.

You can send any continuous stream of data from the client to server using this API, and the server will trust what the client has sent. It is up to you to validate that the clients have sent legitimate inputs. Be sure to enforce maximum and minimum ranges on numbers sent from the client, just like how you would validate values sent from remotes.

!!! note 
    The server will automatically ignore input data from the client if it arrives far too late or far too early, which can happen if there are sudden changes in a client's network conditions.

-----

# RunService

There are new properties and events added to `RunService` which you can utilize to work with the server authority system.

## Properties

### `RunService.FrameNumber`

This value determines the current frame number. This value is rolled back on the client when a resimulation occurs. It is the foundation upon which features like `FixedHeartbeat` are built. Printing this number can be helpful for understanding what exactly is happening when on the client and server.

## Events

### `RunService.FixedHeartbeat(deltaTime: number)`

This signal is fired just like the normal [`Heartbeat`](https://create.roblox.com/docs/reference/engine/classes/RunService#Heartbeat) signal, but is also fired again for each frame of resimulation after a misprediction is detected. This is the right place to put your core game logic, including processing input and updating your synchronized game data.

### `RunService.Misprediction(remoteWorldStepId: number, mispredictedInstances: {any})`

This signal is fired when a misprediction occurs on the client. This means the client has received data from the server that did not match what it had previously simulated. The client is about to go back to the server state and call `FixedHeartbeat` to bring itself back up to its present frame. The second argument contains information about which properties on which `Instance`s were incorrect, and how many frames will be simulated. In general, if playing by yourself, you should only see a small number of mispredictions when your ping to the server changes.

-----

# Instance

Alongside `RunService`, there are new methods implemented to control the prediction state for `Instance`s.

## Methods

### `Instance:SetPredictionMode(mode: Enum.PredictionMode)`

Determines whether the engine will rollback and resimulate the `Instance`.

* If `mode` is `Enum.PredictionMode.Off`: Disables rollback and resimulation for the Instance. When a place's `Workspace.AuthorityMode` is set to `Server`, the `Instance` will be owned by the server with no client-side prediction.

* If `mode` is `Enum.PredictionMode.Automatic` (default value): Allows the engine to determine whether to rollback and resimulate the `Instance`. For `Instance`s that derive from `BasePart`, the engine uses the player’s simulation radius to determine if an `Instance` should be predicted. This helps limit expensive client-side prediction to only the relevant `Instance`s. At the moment, Non-`BasePart`s will not rollback when set to `Automatic`.

* If `mode` is `Enum.PredictionMode.On`: Will ensure the `Instance` is always rolled back when a misprediction occurs. For `Instance`s critical to your experience, use this setting. Otherwise, do not overuse `On` for `Instance`s, as it’ll have significant performance implications for low-end devices.

### `Instance:GetPredictionMode(): Enum.PredictionMode`

Returns the prediction mode enum that indicates whether the engine will rollback and resimulate the `Instance`.

### `Instance:IsPredicted(): boolean`

Returns a boolean indicating whether the `Instance` is being predicted or not.

## Attribute Rollback

Attributes on predicted `Instance`s are fully synchronized with the rollback netcode model. For Non-`BasePart` `Instance`s, you must set the `Instance`'s `PredictionMode` to `Enum.PredictionMode.On` to have it be fully synchronized. This means that on the client, attributes are compared against the server’s version and mismatches will cause a full rollback and resimulation. You should use attributes to store the game data that affect your core simulation. Examples include scores, health, ammunition, inventory, or custom game rules (like which player last touched a ball).

-----

# Debugging and Tooling

Press CTRL + SHIFT + F6 on Windows or CMD + SHIFT + F6 on macOS to enable the Server Authority Visualizer. When it’s on, you’ll see a new debug pane appear at the bottom-right of your viewport. Along with showing statistics about server authority, this tool shows PV mispredictions for all predicted `Instance`s in the workspace. Pairs of boxes connected by pink lines show mispredictions: for each pair, the blue box represents the client’s misprediction and the green box represents the server’s authoritative simulation. Each box emits a vector representing the `CFrame`'s facing direction at the given location. You can clear the mispredictions from the world with CTRL/CMD + SHIFT + F7 and sort them by proximity to the player with CTRL/CMD + SHIFT + F8.

-----