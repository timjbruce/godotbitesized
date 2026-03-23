# Lesson 3 - Extending the Player Component

In this lesson, we'll focus on extending the Player component to 

## First Player Updates!

**This section uses Godot 4.5.1 and the Player Project**

[Player Project](https://github.com/timjbruce/godotbitesized_player)

The next few updates of the Player are going to start adding more features to our Player. Some of these features will grow into components, such as Weapons and Inventory. Other features will be captured within the current scene. For Build 3, we're focusing only on what is contained in the Player scene.

### What Makes a Good Component Versus a Good Feature?

Before we jump into the code, let's talk a bit of philosophy about components versus features. There is no one way to determine if something makes a better component or feature. I tend to look at it this way: is it small and large enough to maintain and does it provide enough functionality by itself?

First, on the size side, I look at the component from the Scene tree and size of code. I tend to lean towards mid-size components. When they get too large, I wonder if I am making something more complicated than it needs to be by combining components into a single package. Sometimes, combining components together can be a good thing. But, when it becomes too large that I spend more time trying to debug something versus adding a feature, then I think it might be time to either re-factor and split the components. If the component is too small, I'm creating more maintenance for myself.

Second, I tend to think about how useful a component can be with other components. If there's only one real use of the component, it won't pass my sniff test especially if it only has a few lines of code. At that point, the main component will be calling it and all I've really done is add a layer of complexity to projects by adding another component that needs to be copied to my main project for no other purpose.

Think about a health bar as an example. Health bars could be a component and show the current value and maximum values of a Players health. It could do this with text and / or a bar. You can add to it and subtract from it. That's probably 5 lines of code and another project. In my perspective, too small for a component and just do it as a feature.

On the other hand, could you create a component that allows people to set a ton of variables and, voila!, they have a game? Sure you could. Should you, is definitely a different question. In my perspective, this is too big for a component. If you decide this is a route you want to go, let me know - I'm interested in where you end up!

Now let's move on to adding some features!

### Switching Between Players, Enemies, and NPCs

The first thing I want to do is to add the ability to use the Player component as an Enemy and as an NPC, as well as a Player. To do this, I'm going to add a `type` to the class that can be set at the component level. You'll remember that we added a `name` value to the `initialize` function and we could use that. However, each object in a Godot game needs a unique name. We could substring and check the `name` for the type of Player this is, but I find it is easier to do with a clearly defined field. Let's update the `player.gd` script header to include this

```
enum PlayerType { Player, Enemy, NPC }

@export var speed: int = 600
@export var type: PlayerType = PlayerType.Player 
@export var animation: AnimatedSprite2D = null
```

The enum provides a list of appropriate values that can be used for type. The variable `type` uses this enum as its data type, allowing us to make sure the correct values are provided for this field and we don't get unintended effects from bad input. We also added a field for an animation to be passed in. We probably don't want our NPCs, players, and enemies to all look the same.

Next, we'll add some member variables to handle the x_direction and y_direction inputs or calculations. We'll use the inputs for actual players, but the NPCs and enemies should probably move based on calculations versus the input. I've put the `_` in front of these variables as a reminder that they should not be accessed directly or `@export`ed later.

```
var _x_direction: float = 0.0
var _y_direction: float = 0.0
```

Next, we're going to update the `_process` function to not handle input from the actual game player to control the enemy or NPC. We still need it to be able to move, and we still want the animations. So, let's wrap the input lines with an if statement that can check if this instance of Player is a player! We also need to reference the new member variables we just created.

```
func _process(_delta: float) -> void:
	if type == PlayerType.Player:
		_x_direction = Input.get_axis("left", "right")
		_y_direction = Input.get_axis("up", "down")
	if _x_direction:
		velocity.x = _x_direction * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
	if _y_direction:
		velocity.y = _y_direction * speed
	else:
		velocity.y = move_toward(velocity.y, 0, speed)
		
	if velocity.x != 0:
		$AnimatedSprite2D.animation = "walk"
		$AnimatedSprite2D.flip_v = false
		$AnimatedSprite2D.flip_h = velocity.x < 0
	elif velocity.y < 0:
		$AnimatedSprite2D.animation = "up"
	elif velocity.y > 0:
		$AnimatedSprite2D.animation = "down"
	else:
		$AnimatedSprite2D.animation = "idle"
	$AnimatedSprite2D.play()
```

Before we make further to code, open the Player scene and change the name of "Label" to "PlayerTypeLabel" as shown below.

![Change the name of Label to PlayerTypeLabel in the Scene Tree](/readmeassets/lesson3/player/3_change_label_name.png)

We'll also change the `initialize` function to take in two new arguments for the type and animation variable. We're going to make animation optional, as we still want to be able to test the Player component without having to deal with multiple animations within the component.

```
func initialize(inc_name: String, inc_speed: int, inc_layers: Array[int], inc_mask: Array[int], 
	inc_type: PlayerType, inc_animation: AnimatedSprite2D = null) -> void:
		name = inc_name
		speed = inc_speed
		type = inc_type
		match type:
			PlayerType.Player:
				$PlayerTypeLabel.text = "Player"
			PlayerType.Enemy:
				$PlayerTypeLabel.text = "Enemy"
			PlayerType.NPC:
				$PlayerTypeLabel.text = "NPC"
			_:
				"!!!!!!!"
		if inc_animation:
			$AnimatedSprite2D.sprite_frames = inc_animation.sprite_frames
		for layer in inc_layers:
			set_collision_layer_value(layer, true)
		for mask in inc_mask:
			set_collision_mask_value(mask, true)
```

Lastly, we are going to use the `_ready` function to do some testing with this component.

```
func _ready() -> void:
	if get_parent().name == "root":
		initialize("Tim", 500, [3], [2], PlayerType.Player, null)
```

Now, if you hit F6 to play the scene, a Player with the Label of "Player" should show up and you can use your controller to move them around. Don't forget to override the ingame camera to see them move on the screen. You can also test this with NPC and Enemies by changing the `PlayerType.Player` values to `PlayerType.NPC` and PlayerType.Enemy` respectively.

### Adding a Test Harness

This is the spot where I normally add a testing scene, as I want to see all the types in one scene. Let's go to the FileSystem and add a new directory called "Test" at the root of the project, like below. This will be our test harness, so we don't have to do crazy stuff like editing and commenting out test cases, like we'd have to if we just used _ready.

![Add a test folder for the Player project](/readmeassets/lesson3/player/3_add_test_harness.png)

Now, create a new scene in the test folder and call it "Test". Keep it as a 2D Scene.

![Add the new testing scene, which will be the scene for the test harness](/readmeassets/lesson3/player/3_add_test_scene.png)

In the Scene window for the "Test" scene, attach a script to the Test node. Accept the default values that appear in the Attach Node Script dialog and click "Create".

![Add a script to the Test Node](/readmeassets/lesson3/player/3_add_script_to_test.png)

Now, add the following code to the `test.gd` file:

```
extends Node2D

var player_node: PackedScene = preload("res://Player/Player.tscn")

func _ready() -> void:
	var layer: Array[int] = [2]
	var mask: Array[int] = [2]
	var test_cases = [["Tim1", 500, layer, mask, player_2d_body.PlayerType.Player, null],
						["Tim2", 500, layer, mask, player_2d_body.PlayerType.NPC, null],
						["Tim3", 500, layer, mask, player_2d_body.PlayerType.Enemy, null]]
	var p1: player_2d_body
	var i: int = 0
	for test_case in test_cases:
		p1 = player_node.instantiate()
		p1.initialize(test_case[0], test_case[1], test_case[2], test_case[3], test_case[4], test_case[5])
		add_child(p1)
		p1.set_location(Vector2i(i * 50, i * 50))
		i+=1
```

This will create a Player, NPC, and Enemy and add them to the Test scene. If you use your controller, you should see that only your Player component moves.

![Testing your player class shows that only your "Player" moves on the screen and the other two stay still](/readmeassets/lesson3/player/3_testing_your_player_class.png)

<details>
<summary>Full listing for player.gd</summary>

```
extends CharacterBody2D
class_name player_2d_body

enum PlayerType { Player, Enemy, NPC }

@export var speed: int = 600
@export var type: PlayerType = PlayerType.Player 
@export var animation: AnimatedSprite2D = null

var _x_direction: float = 0.0
var _y_direction: float = 0.0


func _ready() -> void:
	if get_parent().name == "root":
		initialize("Tim", 500, [3], [2], PlayerType.Player, null)


func initialize(inc_name: String, inc_speed: int, inc_layers: Array[int], inc_mask: Array[int], 
	inc_type: PlayerType, inc_animation: AnimatedSprite2D = null) -> void:
		name = inc_name
		speed = inc_speed
		type = inc_type
		match type:
			PlayerType.Player:
				$PlayerTypeLabel.text = "Player"
			PlayerType.Enemy:
				$PlayerTypeLabel.text = "Enemy"
			PlayerType.NPC:
				$PlayerTypeLabel.text = "NPC"
			_:
				"!!!!!!!"
		if inc_animation:
			$AnimatedSprite2D.sprite_frames = inc_animation.sprite_frames
		for layer in inc_layers:
			set_collision_layer_value(layer, true)
		for mask in inc_mask:
			set_collision_mask_value(mask, true)


func set_location(new_location: Vector2i) -> void:
	global_position = new_location	


func _process(_delta: float) -> void:
	if type == PlayerType.Player:
		_x_direction = Input.get_axis("left", "right")
		_y_direction = Input.get_axis("up", "down")
	if _x_direction:
		velocity.x = _x_direction * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
	if _y_direction:
		velocity.y = _y_direction * speed
	else:
		velocity.y = move_toward(velocity.y, 0, speed)
		
	if velocity.x != 0:
		$AnimatedSprite2D.animation = "walk"
		$AnimatedSprite2D.flip_v = false
		$AnimatedSprite2D.flip_h = velocity.x < 0
	elif velocity.y < 0:
		$AnimatedSprite2D.animation = "up"
	elif velocity.y > 0:
		$AnimatedSprite2D.animation = "down"
	else:
		$AnimatedSprite2D.animation = "idle"
	$AnimatedSprite2D.play()
	
	move_and_slide()
```

</details>

## Updating the World with Player Types

**This section uses Godot 4.5.1 and the World Project**

[World Project](https://github.com/timjbruce/godotbitesized_world)

This section integrates the Player component that introduced Player types. We're going to spawn a player and an enemy into the same room.

### Update the Player

Before you open the project, copy the new Player component into the appropriate folder for your World project. Then, open the project.

If you open the `world.gd` file, you will see that we now get an error on the line for `Globals.player.initialize`. This is because we've added new parameters to the `initialize` function in the "Player" component. We will fix this line of code during our work integrating the new Player component.

### Update of Globals

Before we do, open the `Globals.gd` file. We're going to add a new member variable to Globals to hold the enemies. We want to have many enemies, so we're going to use an array to track them.

```
var enemies: Array[player_2d_body]
```

Next, we'll add a function to allow an enemy to be added to the array.

```
func add_enemy(inc_enemy: player_2d_body) -> void:
	enemies.append(inc_enemy)
```

Save the globals.gd file and let's move to update the World scene.

### Update of World scene

Before we add code to the world.gd file to spawn the Player and Enemy, we want to create an asset that will have the AnimatedSprite2D that we'll use for the Enemy characters. You might make many games and your enemies will probably need to look different. You might even make a boss enemy that looks drastically different for this game. This feature will help you to do that easily.

First, open the World scene and then click on the "World" node in the Scene window. Right click and choose "+ Add Child Node..." Search for "AnimatedSprite2D" and add it to the scene so that it looks like this.

![Add an animatedsprite2d node to the World scene](/readmeassets/lesson3/world/4_add_animated_sprite_2d.png)

Double click on the AnimatedSprite2D and rename it to "EnemySpriteAnimation"

![Rename animatedsprite2d to EnemySpriteAnimation](/readmeassets/lesson3/world/4_rename_animation_to_enemy_sprite_animation.png)

In this repository, locate the folder /World/assets and copy the 4 PNGs located there to your projects World/asset folder. You will need to create this folder before copying the files.

Next, click on the EnemySpriteAnimation node and look at the Inspector window. We will need to add Sprite Frames to this object. Do so by clicking the down arrow that is highlighted below and selecting "New Sprite Frames."

![Add sprite frames to the EnemySpriteAnimation](/readmeassets/lesson3/world/4_add_sprite_frames.png)

Click on the field that reads "SpriteFrames" to open the Animations window on the bottom.

![Load the Animation window by clicking on the SpriteFrames value in the Inspector](/readmeassets/lesson3/world/4_load_the_animation_window.png)

We are going to add the Animations for walk, down, up, and idle to this object using the assets we copied. First, click the "Add Animation" button 3 times so there are four animations listed. Then, rename the animations so it looks like this:

![Add 3 animations to the list and name them idle, walk, up, and down](/readmeassets/lesson3/world/4_add_and_rename_animations.png)

Next, we're going to add frames from sheet sheets. You will click the Add Frames From Sprite Sheet button, locate the appropriate enemy file in your "World/assets" folder and then select the animation frames to add. You might need to update the horizontal and vertical frames for the images. If you need a reminder of this, please see the [Player Build 1](https://github.com/timjbruce/godotbitesized_player/blob/main/readme.MD#build-1). Your end product should look similar to this:

![Add animations to each of the Animations listed using the files that are in the World/assets folder.](/readmeassets/lesson3/world/4_add_sprite_frames.png)

Then, go back to the Inspector window and locate the Texture -> Filter setting. Change this value to "Nearest".

![Change the Texture Filter setting to Nearest for the EnemySpriteAnimation](/readmeassets/lesson3/world/4_set_filter_to_nearest.png)

Finally, we want to hide this asset from within the World scene. In the Scene window, click the eye icon so that this is hidden.

![Hide the animation in the scene by clicking the eye icon so that it shows as closed](/readmeassets/lesson3/world/4_hide_animation.png)

### On to coding! Updates for the world.gd file

Open the `world.gd` file in the editor. We're going to make a few changes here so that we can spawn a player and an enemy in the same room. All of the changes are in `_on_floor_generated`. Let's walk through them

```
func _on_floor_generated() -> void:
	print("floor generated message received")
	var player_resource: PackedScene = preload("res://Player/player.tscn")
	var player: player_2d_body = player_resource.instantiate()
	Globals.set_player(player)
	Globals.world.add_child(Globals.player)
	var player_layers: Array[int] =  [3]
	var player_masks: Array[int] = [2, 4]
	Globals.player.initialize("player", 700, player_layers, player_masks, player_2d_body.PlayerType.Player, null)
	Globals.player.set_location(Globals.generator.get_player_start())
	var enemy: player_2d_body = player_resource.instantiate()
	Globals.add_enemy(enemy)
	Globals.world.add_child(enemy)
	var enemy_layers: Array[int] = [4]
	var enemy_masks: Array[int] = [2, 3]
	Globals.enemies[0].initialize("enemy_0", 300, enemy_layers, enemy_masks, player_2d_body.PlayerType.Enemy, $EnemySpriteAnimation)
	Globals.enemies[0].set_location(Globals.generator.get_player_start())
```

First, we changed the preload to not instantiate a player. This is because we'll use the same resource to create multiple instances. In the first lines, we create a player, add it to Globals, add it as a child, and then initialize it using variables along the way. Finally, we set its location. We do the same process with an enemy, and you can see we use different layers and masks. Basically, this will prevent the player from colliding with the enemy. In the initialization call, we pass the `$EnemySpriteAnimation` to it. As such, our enemy should come into this game with this green animation, versus our standard player animation. Let's try it by saving the project and running it.

There's probably some questions out there on the `initialize` call signature changing between versions and shouldn't components not have this issue. It does seem like this causes a level of coupling between the World and different components. Unfortunately, this is hard to avoid in Godot. Many languages offer what is called "overloading," or offering the same function name with different parameters. With overloading, we could create a second `initialize` function that would take the extra parameters and leave the first one in place. We could manage the missing values in script and this can get a bit messy, over time, if you don't do it well. Godot script, today, does not support overloading so we need to make a call between passing as a dictionary or passing by named variables.

Passing by dictionary can lead to a set of troubleshooting and researching while implementing a component or a component update. You need to know the name of the dictionary elements and the variable types. In the IDE in Godot, if you enter a function name that accepts a dictionary, the tooltip will show up that it needs to receive a dictionary and not all of these details. Additionally, you cannot use type hints today for every type of variable that could be passed in the dictionary. This leads to a subpar developer experience, imho.

I definitely prefer denoting variables and variable types in the function calls directly for these reasons. It allows a much tighter checking at compile time than defining and passing a dictionary, where many values might be incorrect or even missing. If this happens, your players might run into issues playing your games and become frustrated. 

<details>
<summary>Updated world.gd file</summary>

```
extends Node2D

func _ready() -> void:
	Globals.set_world($".")
	Globals.set_global_timer($GlobalTimer)
	Globals.set_generator($Generator)
	Globals.generator.floor_generated.connect(_on_floor_generated)
	Globals.generator.floor_is_cleared.connect(_on_floor_cleared)
	Globals.generator.generate()


func _on_floor_generated() -> void:
	print("floor generated message received")
	var player_resource: PackedScene = preload("res://Player/player.tscn")
	var player: player_2d_body = player_resource.instantiate()
	Globals.set_player(player)
	Globals.world.add_child(Globals.player)
	var player_layers: Array[int] =  [3]
	var player_masks: Array[int] = [2, 4]
	Globals.player.initialize("player", 700, player_layers, player_masks, player_2d_body.PlayerType.Player, null)
	Globals.player.set_location(Globals.generator.get_player_start())
	var enemy: player_2d_body = player_resource.instantiate()
	Globals.add_enemy(enemy)
	Globals.world.add_child(enemy)
	var enemy_layers: Array[int] = [4]
	var enemy_masks: Array[int] = [2, 3]
	Globals.enemies[0].initialize("enemy_0", 300, enemy_layers, enemy_masks, player_2d_body.PlayerType.Enemy, $EnemySpriteAnimation)
	Globals.enemies[0].set_location(Globals.generator.get_player_start())
	

func _on_floor_cleared() -> void:
	print("floor cleared message received")

```

</details>


### Voila!

You might have to move around a bit in your room, depending on the settings and spawn locations, but you should see output similar to this.

![Your game now has an enemy, although it doesn't move!](/readmeassets/lesson3/world/4_your_player_and_enemy.png)

Now let's add more features to the Player!

## Adding movement to the NPCs and Players without Pathfinding

**This section uses Godot 4.5.1 and the Player Project**

[Player Project](https://github.com/timjbruce/godotbitesized_player)

In this build, we'll be leveling off the abilities of the all types of player_2d_body to have the same ability - movement. Currently, this is the only thing that separates our Player type from the others. There will be other things that we will add to the player_2d_body, including inventory, equipment, and interaction. These features will look very similar in nature to what we're going to do for movement.

### Sub Components

This build will feature sub components, or components that we will only use with the player_2d_body but are not included in the Player scene. Sub components are scenes that can be loaded by player_2d_body class. Specifically, we're going to add sub components that allow the Enemy type to "look" for the Player. Also, we want the NPC type to avoid the Enemy. The Enemy looking should be only in the direction they are moving. I tend to use conical or triangular shapes for looking forward. The NPC will be more reactive, always looking around in a panic while moving in one direction. For the NPC, I'll use a circle with them at the center. Both of these will use the `ShapeCast2D` node as their base. 

You might wonder why these sub components aren't just part of the scene if these components are only used by player_2d_body. It seems pretty easy that you could turn them on and off as needed. This is a great question! I tend to look at what the component is doing and if it is a waste of CPU cycles to include in the main component scene. Both of these cases will be handling events with the built-in `_physics_process` function. We don't need this for the Player type. Since this internal function is triggered 60 times per second by default for each node, I like to make sure it only processes code when it is needed.

Your next question might be along the lines of, "but this is used in most of our player_2d_body nodes now. What's the big difference to leave it out of just player?" For this implementation, you're right. We may, however, want games where the NPC stays still. Or wanders but doesn't care about the Enemy. Or something else. Or, what if only your Player type requires physics checking? This approach gives us flexibility to adjust the sub component while leaving the main component mostly unedited.

This might seem like a lot of extra work, especially at first. Thinking about this flexibility and isolating functionality can help with re-use down the road.

### Our First Sub Component - A shared script

Once you start working with sub components, you'll find that some structures and data elements need to be shared across objects. Our `PlayerType` enumeration is a big one that will help with sub components.

Step 1, in the FileSystem window, create a `Scripts` folder under the `Player` folder. We will add a `player_shared.gd` script to that folder. The end result should look like this. The `player_shared.gd` script will act as globals for our player_2d_body class.

![Create a Scripts folder and add a player_shared.gd file to it. We will use this to share global definitions and functions for player_2d_body](/readmeassets/lesson3/player/4_shared_script_created.png)

Next, update the `player_shared.gd` file with the following code:

```
extends Node
class_name PlayerShared

enum PlayerType { Player, Enemy, NPC }
```

The class_name `PlayerShared` gives us a reference to call items in this file by and we are moving just the enum for PlayerType at this time. This way, all of our components in the Player folder can reference the enum values for PlayerType by using the code `PlayerShared.PlayerType.` syntax before the value.

Save this file and close the editor window. We will update the player_2d_body class later, after we get the other sub components setup.

### Our Second Sub Component - SightLine

The SightLine sub component will be used by Enemy types. It will be a line of sight for the enemies and will present a conic shape from the Enemy, expanding outward to a limit. It should also be in the direction the Enemy is moving.

I like to create subdirectories for each sub component, as this keeps it easy to manage. So, create a new folder under the `Player` folder called `Sightline`. The result should look like this.

![Create a new folder for the Sightline scene under the Player folder](/readmeassets/lesson3/player/4_sightline_folder.png)

We're going to create a new scene under this folder called `Sightline`. It should be of type `ShapeCast2D`.

![Create the new Sightline scene using the ShapeCast2D node as the base](/readmeassets/lesson3/player/4_sightline_scene.png)

In the Scene window, select the `SightLine` node. To enable this node, we'll need to set a shape to it. Then we can add in code to support the Enemy type using it to "look" for Players. To set the shape, look in the Inspector window and click on the down arrow by the "Shape" field. Select the "ConvexPolygonShape2D" value.

![Start the process to set the shape of the Sightline 2D shape](/readmeassets/lesson3/player/4_set_shape_of_sightline.png)

Click on the "ConvexPolygonShape2D" field value and then on the "PackedVector2" field value to open the fields where we can build the shape.

For size, enter "3" so we can add 3 points to the shape. We'll use the points (0,0), (135, -45), and (135, 45).

![Set the SightLine points so we get our cone for the enemy to look through](/readmeassets/lesson3/player/4_set_sightline_points.png)

Next, set the Target Position in the Inspect to x = 0.0 px and y = 0.0 px. Also, uncheck any selected Collision Masks.

![Set the SightLine target position to 0 and 0, also uncheck any selected collision masks](/readmeassets/lesson3/player/4_set_sightline_position_and_collision_mask.png)

Add a script to the Sightline scene and name it `Sightline`. We will then place the code below in this file.

```
extends ShapeCast2D
class_name _sightline

signal found_target(player_2d_body)
signal hit_object()

var _has_target: bool = false


func _physics_process(delta):
	# Common Block #
	
	# Per Type Block #
	match get_parent().type:
		PlayerShared.PlayerType.Player:
			return
		PlayerShared.PlayerType.Enemy:
			if is_colliding():
				var collides = get_collision_count()
				for i in collides:
					var obj = get_collider(i)
					match obj.name:
						"player":
							pass
							if not _has_target:
								_has_target = true
								found_target.emit(obj)
						"DefaultTileMap":
							if not _has_target and global_position.distance_to(collision_result[0]["point"]) < 70:
								hit_object.emit()
						_:
							if not _has_target and "enemy" in obj.name and global_position.distance_to(collision_result[0]["point"]) < 50:
								hit_object.emit()
							else:
								print("SightLine: Not looking for ", obj.name)
		PlayerShared.PlayerType.NPC:
			return
		_:
			print("_physics_process Did not have a type")
			return
	#Common Block  

func set_direction(inc_vector2: Vector2) -> void:
	rotation = inc_vector2.angle()


```

This script will run 60 times per second for components it is attached to. If you look through it, you can see there are 2 signals it can emit, `found_target` and `hit_object`. `found_target` is used for when the ShapeCast2D has found a target to follow. It will send the collision target to whatever is connected to it. `hit_object` is used to denote the instance of the class has run into something and cannot move in that direction any further. You've seen those "wall slides," where an enemy or something continues to move slowly along the wall and eventually locks into a corner. This can help prevent that.

Next, look at the `_physics_process` function. This function introduces a technique I normally use in creating components, and that is putting in 3 code blocks for the function. First is a common block, next a per type block, and then finally a close out common block. I try to organize my code so that it fits in this construct, as it makes long term maintenance easier for me. It also helps reduce spaghetti code! I like to use a `match` statement with all the types, even if they aren't being used. The match should never hit these blocks in this implementation, so I just use `return` to skip the rest of the function.

The `Enemy` case is the most interesting. It looks for collisions `player` (who the Enemies will hunt) and `DefaultTileMap` (the walls). If it finds a player, it will emit that the player is found, via `found_target` and allow the listener to do what it will with this information. If it finds a wall, it will emit the `hit_object` signal to let the listener know that the wall is hit. Finally, it will also try to avoid running over other `enemy_` instances. I left in a print statement if it runs into something else that is not explicitly expected. This can be helpful in the future.

Finally, there is a function called `set_direction`. This is used to set the direction of the ShapeCast2D to be in front of where the Enemy is moving. This is not called within this class, but will be called by the player_2d_body class.

Save the script file and scene and we'll move on to our next sub component.

### The Scram Circle, Our Third Sub Component

The Scram Circle will be used by the NPC types, who will constantly be looking for any Enemies to run from. To do this, we'll use a circle shape to simulate the constant scanning, versus an always moving shape.

Start by creating a new folder under the Player folder named `Scramcircle`.

![Create a Scramcircle folder under the Player folder](/readmeassets/lesson3/player/4_scramcircle_folder.png)

Under the `Scramcircle` folder, create a new scene called `Scramcircle`. It should use the `ShapeCast2D` node.

![Create the scene for the Scramcircle using the ShapeCast2D node](/readmeassets/lesson3/player/4_scramcircle_scene.png)

Select the `ScramCircle` node in the Scene window. We need to set the shape, just like we did for the SightLine. Click the down arrow by the "Shape" field and select "CircleShape2D".

![Set the shape of the scramcircle to circleshape2d](/readmeassets/lesson3/player/4_set_scramcircle_shape.png)

Set the radius to 100.0 px, Target Position to 0.0 px, 0.0 px, and unselect the Collision Masks.

![Set the properties for the scramcircle to a 100 px size, located at 0.0, 0.0, with no collision masks](/readmeassets/lesson3/player/4_scramcircle_properties.png)

Next, attach a script to the Scramcircle and call it `scramcircle.gd`. We will use the following code in this file.

```
extends ShapeCast2D
class_name _scramcircle

signal scram_target(player_2d_body)
signal scram_target_clear()
signal hit_object()

var _has_target: bool = false

	
func _physics_process(delta):
	# Common Block #
	
	# Per Type Block #
	match get_parent().type:
		PlayerShared.PlayerType.Player:
			return
		PlayerShared.PlayerType.Enemy:
			return
		PlayerShared.PlayerType.NPC:
			var _scram_target: bool = false
			if is_colliding():
				var collides = get_collision_count()
				for i in collides:
					var obj = get_collider(i)
					match obj.name:
						"player":
							pass
						"DefaultTileMap":
							if not _has_target and global_position.distance_to(collision_result[0]["point"]) < 70:
								hit_object.emit()
						_:
							if not _has_target and "enemy" in obj.name and global_position.distance_to(collision_result[0]["point"]) < 100:
								print("enemy in range ")
								scram_target.emit(obj)
								_has_target = true
								_scram_target = true
							else:
								print("ScramCircle: Not looking for ", obj.name)
			if not _scram_target and _has_target:
				_has_target = false
				scram_target_clear.emit()
		_:
			print("_physics_process Did not have a type")
			return
	#Common Block  

```

This code looks similar to the Sightline code. It has 3 signals it can emit, one to say that it found a target to avoid (`scram_target`), one to clear the target to avoid (`scram_target_clear`), and one to say it hit a wall or other blocker (`hit_object`).

In the `_process_physics` function, we can see that the NPC is avoiding walls. When it locates an enemy it sets a flag internally to denote that it found an enemy and emits the signal that one is in range. The listener can then take action on this. If it escapes from the target, it sends a the clear signal.

Save this script file and scene and we'll update the player code to use these sub components.

### Player Updates

The player_2d_body now just needs to use the sub components we've created. There's some general maintenance and feature updates that we need to make to get this working.

Open up the `player.gd` file. The first step is to remove the `PlayerType enum` line. We've moved that to the `player_shared.gd` script. Next, scan the file for any lines that show with a red highlight. These are lines in error and are broken because we just deleted the enum line. You can fix them by placing `PlayerShared.` in front of each reference to `PlayerType` like so:

```
@export var type: PlayerShared.PlayerType = PlayerShared.PlayerType.Player 
```

There should be seven placed to update this code. If you don't get them all, there is a copy of the entire player.gd file below.

We're also going to add the following member variables to the top of the file, under the `@export lines`.

```
var _sight_line: _sightline = null
var _target: player_2d_body = null

var _scram_circle: _scramcircle = null
var _target_cleared: bool = false
```

`_sight_line` and `_scram_circle` will be used for cases where we need the sub components loaded. The other variables are used to keep state from the ShapeCast2Ds we're using.

Now, we'll update the `initialize` function with the following code:

```
func initialize(inc_name: String, inc_speed: int, inc_layers: Array[int], inc_mask: Array[int], 
	# Common Block #
	inc_type: PlayerShared.PlayerType, inc_animation: AnimatedSprite2D = null) -> void:
		name = inc_name
		speed = inc_speed
		type = inc_type
		if inc_animation:
			$AnimatedSprite2D.sprite_frames = inc_animation.sprite_frames
		for layer in inc_layers:
			set_collision_layer_value(layer, true)
		for mask in inc_mask:
			set_collision_mask_value(mask, true)
		
	# Per Type Block #
		match type:
			PlayerShared.PlayerType.Player:
				$PlayerTypeLabel.text = "Player"
				$Camera2D.enabled = true
			PlayerShared.PlayerType.Enemy:
				$PlayerTypeLabel.text = "Enemy"
				$Camera2D.enabled = false
				_sight_line = preload("res://Player/Sightline/Sightline.tscn").instantiate()
				for mask in inc_mask:
					_sight_line.set_collision_mask_value(mask, true)
				add_child(_sight_line)
				_sight_line.found_target.connect(_on_found_target)
				_sight_line.hit_object.connect(_on_hit_object)
			PlayerShared.PlayerType.NPC:
				$PlayerTypeLabel.text = "NPC"
				$Camera2D.enabled = false
				_scram_circle = preload("res://Player/Scramcircle/Scramcircle.tscn").instantiate()
				for mask in inc_mask:
					_scram_circle.set_collision_mask_value(mask, true)
				add_child(_scram_circle)
				_scram_circle.hit_object.connect(_on_hit_object)
				_scram_circle.scram_target.connect(_on_scram_target)
				_scram_circle.scram_target_clear.connect(_on_clear_scram_target)
			_:
				print("initialize Did not have a type")
				$Camera2D.enabled = false
		
	# Common Block #
```

You will see some errors highlighted on the `connect` lines, since we do not have those functions added yet. The Enemy type will now load the "Sightline" component and look for collisions. It also connects to the signals to handle updates from the sub component. The NPC version does much the same while using the "Scramcircle" component.

Next, we'll update `_process` with the following code. We need the Enemies and NPCs to be able to move

```
func _process(_delta: float) -> void:
	# Common Block #
	
	# Per Type Block #
	match type:
		PlayerShared.PlayerType.Player:
			_x_direction = Input.get_axis("left", "right")
			_y_direction = Input.get_axis("up", "down")		
			if _x_direction:
				velocity.x = _x_direction * speed
			else:
				velocity.x = move_toward(velocity.x, 0, speed)
			if _y_direction:
				velocity.y = _y_direction * speed
			else:
				velocity.y = move_toward(velocity.y, 0, speed)
		PlayerShared.PlayerType.Enemy:
			if _target:
				var cur_vector: Vector2 = (_target.position - global_position).normalized()
				velocity = cur_vector * speed
				_sight_line.set_direction(cur_vector)
			else:
				if velocity == Vector2.ZERO:
					var rand_vector: Vector2 = _random_vector()
					velocity = rand_vector * (speed / 2.0)
					_sight_line.set_direction(rand_vector)
		PlayerShared.PlayerType.NPC:
			if _target:
				var cur_vector: Vector2 = (_target.position - global_position).normalized().rotated(180)
				velocity = cur_vector * speed * 2
			else:
				if velocity == Vector2.ZERO:
					var rand_vector: Vector2 = _random_vector()
					velocity = rand_vector * (speed)
				elif _target_cleared:
					_target_cleared = false
					var rand_vector: Vector2 = _random_vector()
					velocity = rand_vector * (speed)
		_:
			print("_process did not have a type ")
		
	# Common Block
	if velocity.x != 0 and abs(velocity.x) > abs(velocity.y):
		$AnimatedSprite2D.animation = "walk"
		$AnimatedSprite2D.flip_v = false
		$AnimatedSprite2D.flip_h = velocity.x < 0
	elif velocity.y < 0:
		$AnimatedSprite2D.animation = "up"
	elif velocity.y > 0:
		$AnimatedSprite2D.animation = "down"
	else:
		$AnimatedSprite2D.animation = "idle"
	$AnimatedSprite2D.play()
	
	move_and_slide()
```

The Enemies will walk slowly in a random direction. If they detect a Player, they will walk towards the player at their normal pace. NPCs will walk at a normal pace. If they detect an Enemy, they will speed up in a direction away from the Enemy.

Finally, we will add the functions to respond to signals and a helper function to generate a random Vector2. The random Vector2 function will try to locate what directions have collisions and set min and max variables to, hopefully, get a random Vector2 that moves to an open spot.

```
func _on_found_target(inc_target: player_2d_body) -> void:
	_target = inc_target


func _on_scram_target(inc_target: player_2d_body) -> void:
	_target = inc_target


func _on_clear_scram_target() -> void:
	_target_cleared = true
	_target = null

	
func _on_hit_object() -> void:
	velocity = Vector2.ZERO


func _random_vector() -> Vector2:
	#check up, down, left, and right - limit the vector to good locations
	var raycast: RayCast2D = RayCast2D.new()
	var x_min: float = -1.0
	var x_max: float = 1.0
	var y_min: float = -1.0
	var y_max: float = 1.0
	raycast.set_collision_mask_value(2, true)
	raycast.target_position = Vector2i.RIGHT * 50
	if raycast.is_colliding():
		x_max = 0.25
	raycast.target_position = Vector2i.LEFT * 50
	if raycast.is_colliding():
		x_min = -0.25
	raycast.target_position = Vector2i.UP * 50
	if raycast.is_colliding():
		y_max = 0.25
	raycast.target_position = Vector2i.DOWN * 50
	if raycast.is_colliding():
		y_min = -0.25
	raycast.free()
	return Vector2(randf_range(x_min, x_max), randf_range(y_min, y_max))
```

<summary>The entire player.gd file

<details>

```
extends CharacterBody2D
class_name player_2d_body

@export var speed: int = 600
@export var type: PlayerShared.PlayerType = PlayerShared.PlayerType.Player 
@export var animation: AnimatedSprite2D = null

var _x_direction: float = 0.0
var _y_direction: float = 0.0

var _sight_line: _sightline = null
var _target: player_2d_body = null

var _scram_circle: _scramcircle = null
var _target_cleared: bool = false


func _ready() -> void:
	if get_parent().name == "root":
		initialize("Tim", 500, [3], [2], PlayerShared.PlayerType.Player, null)


func initialize(inc_name: String, inc_speed: int, inc_layers: Array[int], inc_mask: Array[int], 
	# Common Block #
	inc_type: PlayerShared.PlayerType, inc_animation: AnimatedSprite2D = null) -> void:
		name = inc_name
		speed = inc_speed
		type = inc_type
		if inc_animation:
			$AnimatedSprite2D.sprite_frames = inc_animation.sprite_frames
		for layer in inc_layers:
			set_collision_layer_value(layer, true)
		for mask in inc_mask:
			set_collision_mask_value(mask, true)
		
	# Per Type Block #
		match type:
			PlayerShared.PlayerType.Player:
				$PlayerTypeLabel.text = "Player"
				$Camera2D.enabled = true
			PlayerShared.PlayerType.Enemy:
				$PlayerTypeLabel.text = "Enemy"
				$Camera2D.enabled = false
				_sight_line = preload("res://Player/Sightline/Sightline.tscn").instantiate()
				for mask in inc_mask:
					_sight_line.set_collision_mask_value(mask, true)
				add_child(_sight_line)
				_sight_line.found_target.connect(_on_found_target)
				_sight_line.hit_object.connect(_on_hit_object)
			PlayerShared.PlayerType.NPC:
				$PlayerTypeLabel.text = "NPC"
				$Camera2D.enabled = false
				_scram_circle = preload("res://Player/Scramcircle/Scramcircle.tscn").instantiate()
				for mask in inc_mask:
					_scram_circle.set_collision_mask_value(mask, true)
				add_child(_scram_circle)
				_scram_circle.hit_object.connect(_on_hit_object)
				_scram_circle.scram_target.connect(_on_scram_target)
				_scram_circle.scram_target_clear.connect(_on_clear_scram_target)
			_:
				print("initialize Did not have a type")
				$Camera2D.enabled = false
		
	# Common Block #


func set_location(new_location: Vector2i) -> void:
	global_position = new_location	


func _process(_delta: float) -> void:
	# Common Block #
	
	# Per Type Block #
	match type:
		PlayerShared.PlayerType.Player:
			_x_direction = Input.get_axis("left", "right")
			_y_direction = Input.get_axis("up", "down")		
			if _x_direction:
				velocity.x = _x_direction * speed
			else:
				velocity.x = move_toward(velocity.x, 0, speed)
			if _y_direction:
				velocity.y = _y_direction * speed
			else:
				velocity.y = move_toward(velocity.y, 0, speed)
		PlayerShared.PlayerType.Enemy:
			if _target:
				var cur_vector: Vector2 = (_target.position - global_position).normalized()
				velocity = cur_vector * speed
				_sight_line.set_direction(cur_vector)
			else:
				if velocity == Vector2.ZERO:
					var rand_vector: Vector2 = _random_vector()
					velocity = rand_vector * (speed / 2.0)
					_sight_line.set_direction(rand_vector)
		PlayerShared.PlayerType.NPC:
			if _target:
				var cur_vector: Vector2 = (_target.position - global_position).normalized().rotated(180)
				velocity = cur_vector * speed * 2
			else:
				if velocity == Vector2.ZERO:
					var rand_vector: Vector2 = _random_vector()
					velocity = rand_vector * (speed)
				elif _target_cleared:
					_target_cleared = false
					var rand_vector: Vector2 = _random_vector()
					velocity = rand_vector * (speed)
		_:
			print("_process did not have a type ")
		
	# Common Block
	if velocity.x != 0 and abs(velocity.x) > abs(velocity.y):
		$AnimatedSprite2D.animation = "walk"
		$AnimatedSprite2D.flip_v = false
		$AnimatedSprite2D.flip_h = velocity.x < 0
	elif velocity.y < 0:
		$AnimatedSprite2D.animation = "up"
	elif velocity.y > 0:
		$AnimatedSprite2D.animation = "down"
	else:
		$AnimatedSprite2D.animation = "idle"
	$AnimatedSprite2D.play()
	
	move_and_slide()
	
func _on_found_target(inc_target: player_2d_body) -> void:
	_target = inc_target


func _on_scram_target(inc_target: player_2d_body) -> void:
	_target = inc_target


func _on_clear_scram_target() -> void:
	_target_cleared = true
	_target = null

	
func _on_hit_object() -> void:
	velocity = Vector2.ZERO


func _random_vector() -> Vector2:
	#check up, down, left, and right - limit the vector to good locations
	var raycast: RayCast2D = RayCast2D.new()
	var x_min: float = -1.0
	var x_max: float = 1.0
	var y_min: float = -1.0
	var y_max: float = 1.0
	raycast.set_collision_mask_value(2, true)
	raycast.target_position = Vector2i.RIGHT * 50
	if raycast.is_colliding():
		x_max = 0.25
	raycast.target_position = Vector2i.LEFT * 50
	if raycast.is_colliding():
		x_min = -0.25
	raycast.target_position = Vector2i.UP * 50
	if raycast.is_colliding():
		y_max = 0.25
	raycast.target_position = Vector2i.DOWN * 50
	if raycast.is_colliding():
		y_min = -0.25
	raycast.free()
	return Vector2(randf_range(x_min, x_max), randf_range(y_min, y_max))
```

</details>

</summary>

### A Quick Test

Open up your Test Scene and the `test.gd` file and update it to the following code:

```
extends Node2D

var player_node: PackedScene = preload("res://Player/Player.tscn")

func _ready() -> void:
	var player_layer: Array[int] = [3]
	var player_mask: Array[int] = [2, 3, 4, 5]
	var enemy_layer: Array[int] = [4]
	var enemy_mask: Array[int] = [2, 3, 4, 5]
	var npc_layer: Array[int] = [5]
	var npc_mask: Array[int] = [2, 3, 4, 5]
	
	var test_cases = [["player", 150, player_layer, player_mask, PlayerShared.PlayerType.Player, null],
						["enemy", 25, enemy_layer, enemy_mask, PlayerShared.PlayerType.Enemy, null],
						["npc", 25, npc_layer, npc_mask, PlayerShared.PlayerType.NPC, null]]
	var p1: player_2d_body
	var i: int = 0
	for test_case in test_cases:
		p1 = player_node.instantiate()
		p1.initialize(test_case[0], test_case[1], test_case[2], test_case[3], test_case[4], test_case[5])
		add_child(p1)
		p1.set_location(Vector2i(i * 50, i * 50))
		i+=1
```

The speed for NPCs and Enemies should be slow enough that you can move in front of the Enemy to get it to follow you. You may need to restart this a few times to get the Enemy to follow you to the NPC scram circle to get it to react. Next, we'll integrate this into the World project!

## Integrating the Moving NPCs and Enemies Into the World

**This section uses Godot 4.5.1 and the World Project**

[World Project](https://github.com/timjbruce/godotbitesized_world)

For this section, we're going to update the player_2d_body component for the World project. To start, do not have either project open and copy the "Player" folder from the Player component to the World folder. Replace the existing Player folder and objects. Next, open the World project.

When the project opens, it should highlight two lines of code that need to be updated. This is from the change of where the Enum for PlayerType was updated. Change these two lines to use `PlayerShared.PlayerType.Player` and `PlayerShared.PlayerType.Enemy` like so:

```
	Globals.player.initialize("player", 700, player_layers, player_masks, PlayerShared.PlayerType.Player, null)
	Globals.enemies[0].initialize("enemy_0", 300, enemy_layers, enemy_masks, PlayerShared.PlayerType.Enemy, $EnemySpriteAnimation)
```

Now, run the project. Move your Player in front of the Enemy to see if you can get it to follow you.

![Get the enemy to chase you](/readmeassets/lesson3/world/5_enemy_chasing.png)

It might be hard to determine what direction the enemy is facing. Also, we don't have an NPC. So, let's update the code where we add more enemies and an NPC to the mix. Update the `_on_floor_generated` function to this:

```
func _on_floor_generated() -> void:
	print("floor generated message received")
	var player_resource: PackedScene = preload("res://Player/player.tscn")
	var player: player_2d_body = player_resource.instantiate()
	Globals.set_player(player)
	Globals.world.add_child(Globals.player)
	var player_layers: Array[int] =  [3]
	var player_masks: Array[int] = [2, 4, 5]
	Globals.player.initialize("player", 700, player_layers, player_masks, PlayerShared.PlayerType.Player, null)
	Globals.player.set_location(Globals.generator.get_player_start())
	var enemy: player_2d_body
	var enemy_layers: Array[int] = [4]
	var enemy_masks: Array[int] = [2, 3, 4, 5]
	for i in range(0,2):
		enemy = player_resource.instantiate()
		Globals.add_enemy(enemy)
		Globals.world.add_child(enemy)
		Globals.enemies[i].initialize("enemy_" + str(i), 300, enemy_layers, enemy_masks, PlayerShared.PlayerType.Enemy, $EnemySpriteAnimation)
		Globals.enemies[i].set_location(Globals.generator.get_player_start())
	var npc: player_2d_body
	var npc_layers: Array[int] = [5]
	var npc_masks: Array[int] = [2, 3, 4, 5]
	for i in range(0,1):
		npc = player_resource.instantiate()
		Globals.add_npc(npc)
		Globals.world.add_child(npc)
		Globals.enemies[i].initialize("npc_" + str(i), 350, npc_layers, npc_masks, PlayerShared.PlayerType.Enemy, $EnemySpriteAnimation)
		Globals.enemies[i].set_location(Globals.generator.get_player_start())
```

This will add two Enemies and one NPC. We also need to add the `add_npc` function to the "Globals.gd" file, as well as a variable to hold the npcs. Add the following line right after the `var enemies:` line at the top of the script.

```
var npcs: Array[player_2d_body]
```

And then add the following function to the file:

```
func add_npc(inc_npc: player_2d_body) -> void:
	npcs.append(inc_npc)
```

Lastly, we're going to turn on debug mode for collisions. This might help you if you want to add more enemies and NPCs to the mix. Go to the Debug Menu and click on "Visible Collision Shapes". Now hit F5 to run the project.

![Enemies have trapped the player!](/readmeassets/lesson3/world/5_enemies_trapped_my_player.png)

Play around with the speeds and numbers of the Enemies, NPCs!