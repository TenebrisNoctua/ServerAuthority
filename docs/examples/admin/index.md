# Basic Admin System

This is an example basic admin system that you can create and use with Server Authority.

## The Design

We will create a simple modular admin system, that contains some also simple command modules which allows the player to commit certain actions in the game. We can begin by creating a folder in `ReplicatedStorage`, which both the Server and the Client can access.

![hierarchyimage_1](../../img/tutorial/admin/hierarchyimage_1.png)

Next, we will create these instances:

* "CommandEvents" `Folder`: This will hold our events which will allow the Server and the Client to communicate for the command modules. Such as when the command has been ran on the Server, the Server will fire an event which will tell the Client to also run the command.

* "CommandScripts" `Folder`: This will hold our Server and Client scripts which will run the command modules. You can edit these in any way you wish to customize the behavior of when and how will the command be ran.

* "Commands" `Folder`: This will hold our main command modules which will contain our simulation code for running the command. These modules will be ran on both the Server and the Client.

* "Main" `ModuleScript`: This is a module which essentially requires all the command modules and puts them in a simple API for usage. Instead of requiring each and individual command module, one can simply require this module and access the functions and data of all command modules through it. However, this module is purely for ease of use, and not necessary. You can also require the modules individually if you wish to customize that behavior, too.

![hierarchyimage_2](../../img/tutorial/admin/hierarchyimage_2.png)

After setting our hierarchy, we can now set the Main module code:

### Main

```luau
-- Written by @TenebrisNoctua
-- Initializes and loads the command modules.

--// Types

type ModuleReturnType = {
	Begin: (player: Player, ...any) -> (...any),
	End: (player: Player, ...any) -> (...any)?
}

type function GetCommandsType(commandsFolder: type)
	assert(commandsFolder:is("class"), "commandsFolder argument is not an extern type.")
	local commandsTable = types.newtable()
	commandsTable:setindexer(types.string, types.any)
	
	local properties = commandsFolder:properties()
	for i, property in properties do
		local name = i:value()
		if name == "Parent" then continue end
		commandsTable:setproperty(types.singleton(name), types.optional(ModuleReturnType))
	end
	
	return commandsTable
end

--// Main Initialization

local CommandsFolder = script.Parent:FindFirstChild("Commands")
assert(CommandsFolder, "Cannot find the commands folder.")

local CommandModules = CommandsFolder:GetChildren()
local GlobalCommands: GetCommandsType<typeof(CommandsFolder)> = {}

for _, moduleScript in CommandModules do
	if not moduleScript:IsA("ModuleScript") then continue end
	local success, commandModule = pcall(require, moduleScript)
	if success then
		GlobalCommands[moduleScript.Name] = commandModule
	else
		warn("Failed to require command module:", moduleScript.Name)
	end
end

return GlobalCommands
```

This module will now allow us to easily access the command modules with a simple to use API system.

Now that we have set-up our main admin system, we can begin adding our command modules and scripts. To do so, you can check out the basic command tutorial pages on the left side. You may use them to learn how to implement some basic general commands.