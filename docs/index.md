---
  hide:
    - toc
    - navigation
---

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
This does cause responsiveness and latency issues however, but those issues are mostly solvable with a prediction and rollback system, such as the one Roblox is utilizing, which we will mention further in later sections.

To enable the server authority system Roblox has implemented, after all the flags above have been enabled, you must go into
`Workspace > Server Authority > AuthorityMode` and set it to `Server`.

!!! warning
    In order to access the features mentioned in the later sections, this step is necessary to do. Otherwise, the features will simply not work.

After this step, server authority should be enabled within your place.

## Part Physics

Enabling this feature completely changes the behavior of all unanchored `Part`s in the `Workspace`. Before; the physics calculation of a `Part` was handled by both the server and the clients in a place. When a client came close to a `Part`, the network ownership would automatically shift from the server to the client, so the client could take the burden of calculating the physics for that `Part`.

However, now, with server authority enabled; this network ownership behavior is completely bypassed, as even if a client comes close to a `Part`, the server will continue calculating the physics of that `Part`.

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

Up until this point, I explained the default behavior of the server authority system on all parts and characters. However, this default behavior can be changed. For example, it is possible to create a system to exclude certain parts and characters from the prediction, so the network ownership system can be applied.

This is the service that handles the server authority system. It is made out of methods and events that allow you to manually configure certain parts of the server authority system. But first, let's talk about predictions more in detail.

## Predictions

