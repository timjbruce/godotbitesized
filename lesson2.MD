# Lesson 2

Lesson 2 adds physics to our game world and stops players from walking through walls.

## Generator Update 

**This section uses Godot 4.5.1 and the Generator Project**

[Generator Project](https://github.com/timjbruce/godotbitesized_generator)

In this section, we're going to add physics to our generator and, also, a trick to place the player into a room. On the physics side, our goal is to not allow the player to cross the wall boundary for our game. We also want them to move on a floor, so we'll add some tiles to our output for that, based on the tileset we already did. Finally, the trick that I mentioned. We'll add a function to get a location that we can place the player at within the room.

### First thing - adding a physics layer

Open the Generator project and the Generator scene. Click on the "DefaultTileMap" in the Scene browser.

![Open the Generator scene and select the DefaultTileMap object](/readmeassets/lesson2/generator//2_select_the_defaulttilemap.png)

In the Inspector window, click on the TileSet value to open the TileSet details, expand the "Physics Layer" section, and click the "+ Add Element" button in the Physics Layer section.

![Add the physics layer to the tileset that we created before](/readmeassets/lesson2/generator//2_add_the_physics_layer.png)

We're going to set the Collision Layer to "2" and turn off the Collision Mask (nothing is selected). I typically use Collision Layer 2 for all my wall objects. This allows me to not have to worry about re-defining wall collisions in the future and I can just focus on the game.

![Set the collision layer to 2 and the collision mask to 0](/readmeassets/lesson2/generator//2_set_the_collision_layers.png)

Now we need to define what part of the tileset matchest to the physics layer. Because this it the first physics layer for this tileset, it will be called Physics Layer 0. Click on the TileSet tab in the lower window so we can edit properties of the tileset.

![Edit the properties of the tileset](/readmeassets/lesson2/generator//2_edit_tileset.png)

Next, click the "Paint" tool and select "Physics Layer 0" in the Paint Properties field. Finally, click the upper left hand dark brown square, so that it is highlighted as it is below.

![Select the tile that will be used for Physics Layer 0](/readmeassets/lesson2/generator//2_set_the_collision_tile.png)

Now, save the project and we can move on to the coding!

### Adding code for floors and player start location

We're going to start this work in the generator_node.gd file. Open it up and let's take a look at a couple of items before we add the code.

First, in the `generate` function, note that a `room_listing` is being stored. This room listing is a dictionary and contains the room definition. Eventually, this will hold an entire floor's worth of rooms, but, for now, it's just a single item. We will want to start the player somewhere in that room location.

```
func generate() -> void:
	var grid: Dictionary = {Vector2i(0,0): {"x_start": 0, "y_start": 0}}
	room_listing = _generate_room(grid)
	_draw_rooms()
```

We also need to be aware of our texture size for the walls. If you explore the TileMapLayer, you'll be able to see the rendering quadrant size is 16, meaning a 16x16 square. When the room is built, it starts at Vector 0,0. Since the rendering is 16 x 16, we must account for this when placing the player. If the room is 50 x 50 tiles, we will have 800 x 800 in pixels, but 32 on each edge (up and down, left and right) are taken up by the rendering for our walls. The math needs to be adjusted to 0 + rendering and 800 - rendering, or 32, 768 for where we can place the player.

The rendering quadrant size of a TileMapLayer may change, depending on our graphics and game, so this should be added to our thinking. The lazy way is to add it to the initialization, pass it in from the outside. This, however, can break if you forget to reset it or if you change your TileMapLayer while you're working on your game. The goal for this should be to not have to adjust your code for any changes like that.

To do this, let's declare a script variable right after our vectors for the terrains with this code:

```
var wall_tile_quadrant_size: int = 0
```

In `_get_main_tiles`, we will capture the value right after the `var terrains = {}` line:

```
	wall_tile_quadrant_size = tile_map_layer.rendering_quadrant_size
```

Next, we'll add a new function `get_player_start` to this using the following code to randomly find a location for the Player

```
func get_player_start() -> Vector2i:
	var start_location: Vector2i
	start_location = Vector2i(randi_range(room_listing["x"] + wall_tile_quadrant_size, (room_listing["height"] - 1) * wall_tile_quadrant_size ), 
			randi_range(room_listing["y"] + wall_tile_quadrant_size, (room_listing["width"] - 1) * wall_tile_quadrant_size ))
	return start_location
```

Lastly, in the `generator_node.gd`, we are going to add painting of the floor tiles. Update the `_draw_rooms` function with the following code:

```
func _draw_rooms() -> void:
	if draw_outline:
		for x in range(0, game_width):
			if x % 25 == 0:
				tile_map_layer.set_cell(Vector2i(x,0), 0, floor_vector)
			else:
				tile_map_layer.set_cell(Vector2i(x,0), 0, filler_vector)
		for y in range(0, game_height):
			if y % 25 == 0:
				tile_map_layer.set_cell(Vector2i(0,y), 0, floor_vector)
			else:
				tile_map_layer.set_cell(Vector2i(0,y), 0, filler_vector)

	for x in range(room_listing["x"], (room_listing["x"] + room_listing["width"] + 1)):
		for y in range(room_listing["y"], (room_listing["y"] + room_listing["height"] + 1)):
			if x == room_listing["x"] || x == (room_listing["x"] + room_listing["width"]):
				tile_map_layer.set_cell(Vector2i(x,y), 0, wall_vector)
			elif y == room_listing["y"] || y == (room_listing["y"] + room_listing["height"]):
				tile_map_layer.set_cell(Vector2i(x,y), 0, wall_vector)
			else:
				tile_map_layer.set_cell(Vector2i(x,y), 0, floor_vector)
```

We won't want to call this function from the world to the generator_node, so let's add a wrapper function in the `generator.gd` file to call this.

```
func get_player_start() -> Vector2:
	return $GeneratorNode.get_player_start()
```

And, before we finish up with this build, let's update the Generators `_ready` function to call `player_start` so that we can make sure it is working. Your new ready should look like this:

```
func _ready() -> void:
	$GeneratorNode.initialize(min_room_width, min_room_height, max_room_width, max_room_height, game_width,
		game_height, tile_map_layer, draw_outline)
	if get_parent().name == "root":
		$GeneratorNode.generate()
		print(get_player_start())
		floor_generated.emit()
```

If you run this scene with F6, you should see a window with your room open and you should see a vector in the output window, with dark brown walls and a tan floor.

<details>
<summary>Full code for `generator.gd`</summary>

```
extends Node2D
class_name Generator

@export var min_room_width: int = 3
@export var min_room_height: int = 3
@export var max_room_width: int = 500
@export var max_room_height: int = 500
@export var game_width: int = 1600
@export var game_height: int = 1200
@export var tile_map_layer: TileMapLayer = null
@export var draw_outline: bool = false


signal floor_generated
signal floor_is_cleared


func _ready() -> void:
	$GeneratorNode.initialize(min_room_width, min_room_height, max_room_width, max_room_height, game_width,
		game_height, tile_map_layer, draw_outline)
	if get_parent().name == "root":
		$GeneratorNode.generate()
		print(get_player_start())
		floor_generated.emit()


func generate() -> void:
	$GeneratorNode.generate()
	floor_generated.emit()


func clear_floor() -> void:
	$GeneratorNode.clear_floor()
	floor_is_cleared.emit()


func get_player_start() -> Vector2:
	return $GeneratorNode.get_player_start()
```
</details>
<details>
<summary>Full code for `generator_node.gd`</summary>

```
extends Control
class_name GeneratorNode

var min_room_width: int = 3
var min_room_height: int = 3
var max_room_width: int = 500
var max_room_height: int = 500
var game_width: int = 1600
var game_height: int = 1200
var tile_map_layer: TileMapLayer = null
var draw_outline: bool = false

var wall_vector: Vector2i
var floor_vector: Vector2i
var filler_vector: Vector2i
var exit_vector: Vector2i
var wall_tile_quadrant_size: int = 0

var room_listing: Dictionary


func clear_floor() -> void:
	room_listing = {}
	tile_map_layer.clear()


func generate() -> void:
	var grid: Dictionary = {Vector2i(0,0): {"x_start": 0, "y_start": 0}}
	room_listing = _generate_room(grid)
	_draw_rooms()


func initialize(inc_min_room_width: int, inc_min_room_height: int, inc_max_room_width: int, inc_max_room_height: int,
				inc_game_width: int, inc_game_height: int, inc_tile_map_layer: TileMapLayer, inc_draw_outline: bool) -> void:
	min_room_width = inc_min_room_width
	min_room_height = inc_min_room_height
	max_room_width = inc_max_room_width
	max_room_height = inc_max_room_height
	game_width = inc_game_width
	game_height = inc_game_height
	tile_map_layer = inc_tile_map_layer
	draw_outline = inc_draw_outline
	_get_main_tiles()

func _ready() -> void:
	pass
	
	
func _get_main_tiles() -> void:
	var terrains = {}
	wall_tile_quadrant_size = tile_map_layer.rendering_quadrant_size
	for terrain in tile_map_layer.tile_set.get_terrains_count(0):
		terrains[str(terrain)] = tile_map_layer.tile_set.get_terrain_name(0,terrain)
	var source = tile_map_layer.tile_set.get_source(0)
	#check every tile to find its terrain
	for tile in source.get_tiles_count():
		var tile_data = source.get_tile_data(source.get_tile_id(tile), 0)
		if tile_data.terrain != -1:
			match terrains[str(tile_data.terrain)]:
				"Wall":
					wall_vector = source.get_tile_id(tile)
				"Floor":
					floor_vector = source.get_tile_id(tile)
				"Filler":
					filler_vector = source.get_tile_id(tile)
				"Exit":
					exit_vector = source.get_tile_id(tile)
	

func get_player_start() -> Vector2i:
	var start_location: Vector2i
	start_location = Vector2i(randi_range(room_listing["y"] + wall_tile_quadrant_size, (room_listing["width"] - 1) * wall_tile_quadrant_size),
		randi_range(room_listing["x"] + wall_tile_quadrant_size, (room_listing["height"] - 1) * wall_tile_quadrant_size ))
	return start_location


func _generate_room(grid_location) -> Dictionary:
	var room: Dictionary = {}
	for item in grid_location:
		room["x"] = randi_range(grid_location[item]["x_start"], 
			grid_location[item]["x_start"])
		room["y"] = randi_range(grid_location[item]["y_start"], 
			grid_location[item]["y_start"])
		room["width"] = randi_range(min_room_width, max_room_width - (room["x"] - grid_location[item]["x_start"]))
		room["height"] = randi_range(min_room_height, max_room_height - (room["y"] - grid_location[item]["y_start"]))
	return room


func _draw_rooms() -> void:
	if draw_outline:
		for x in range(0, game_width):
			if x % 25 == 0:
				tile_map_layer.set_cell(Vector2i(x,0), 0, floor_vector)
			else:
				tile_map_layer.set_cell(Vector2i(x,0), 0, filler_vector)
		for y in range(0, game_height):
			if y % 25 == 0:
				tile_map_layer.set_cell(Vector2i(0,y), 0, floor_vector)
			else:
				tile_map_layer.set_cell(Vector2i(0,y), 0, filler_vector)

	for x in range(room_listing["x"], (room_listing["x"] + room_listing["width"] + 1)):
		for y in range(room_listing["y"], (room_listing["y"] + room_listing["height"] + 1)):
			if x == room_listing["x"] || x == (room_listing["x"] + room_listing["width"]):
				tile_map_layer.set_cell(Vector2i(x,y), 0, wall_vector)
			elif y == room_listing["y"] || y == (room_listing["y"] + room_listing["height"]):
				tile_map_layer.set_cell(Vector2i(x,y), 0, wall_vector)
			else:
				tile_map_layer.set_cell(Vector2i(x,y), 0, floor_vector)
```

</details>

## Player Updates

Player updates are going to use an initialize script instead of setting properties for physics using the Inspector. You might be wondering why. The main reason is that we will want to change physics layers for our games. In the generator build, I know walls are always going to be layer 2. Characters might have many different layers, however. We could develop a ghost that goes through walls. We could have multiple players in the game, in which case, we might want them to collide with each other. And, my favorite, we might want to enable or disable friendly fire. If we start with an initialize script now, we start to enable building in a more componentized fashion now.

So, why didn't we do this initially? We didn't have anything to set, initially. However, we're going to make some changes and start that path now!

### Create the Initialize function

Open up the `player.gd` script. The first thing we're going to do is to update the SPEED variable. With capitalization, this variable reads more as a constant and we will want to expose this as a property that can be filled in or set with the initialize function. We also don't specify what type of variable it is, so we'll fix that with the following update:

```
@export var speed: int = 600
```

In your editor, you'll see 4 lines highlighted because we changed the name from `SPEED` to `speed`. Update these lines to use the new variable speed so they look like this:

```
		velocity.x = x_direction * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
	if y_direction:
		velocity.y = y_direction * speed
	else:
		velocity.y = move_toward(velocity.y, 0, speed)
```

Now, we can change the Players speed either via properties, if we paint the Player into a scene.

Next, we'll add the `initialize` function. Our initialize function will take 4 inputs. The first one will be the name of the object. All objects that will be instantiated should have a unique name. This will help when inspecting what the object is at runtime. The second will be the speed of the character. The last two will be the layer(s) the Player will exist on and the final will be the masks that will be used to detect collisions. Within this function, we'll set these values.

I could set these values directly on the `Player` by navigating through the scene tree, as needed. The reason I use the `initialize` function instead is that this allows me to refactor any part or the entire scene without needing to worry about updating any part of my game that calls these items directly. I can change how things are implemented without having to chase down all the parts of my game code that references them. This is using a strategy called "loose coupling," where components are free to change the implementation of a feature without causing a ripple effect to other parts of your game.

If you're starting to wonder about what you can do with loose coupling, that's great! If you're on the fence or still wrapping your head around it, that's ok! In either case, stick around for a while longer and I hope that you'll see how I use this approach in a practical manner and, hopefully, that will spark some ideas.

Let's add our `initialize` function to the `player.gd` file:

```
func initialize(inc_name: String, inc_speed: int, inc_layers: Array[int], inc_mask: Array[int]) -> void:
	name = inc_name
	speed = inc_speed
	for layer in inc_layers:
		set_collision_layer_value(layer, true)
	for mask in inc_mask:
		set_collision_mask_value(mask, true)
```

`name` is an automatically created variable name for objects with Godot. We haven't declared it and do not need to declare it, it just exists. Also, note that we only turn on layers and masks that are passed in and don't turn others off. We're going to make sure that all masks and layers are turned off, by default, in the Inspector in the next step.

We'll also add a function to set the location of the Player, for after the Generator is done.

```
func set_location(new_location: Vector2i) -> void:
	global_position = new_location	
```


<details>
<summary>Full code for player.gd</summary>

```
extends CharacterBody2D
class_name player_2d_body

@export var speed: int = 600


func initialize(inc_name: String, inc_speed: int, inc_layers: Array[int], inc_mask: Array[int]) -> void:
	name = inc_name
	speed = inc_speed
	for layer in inc_layers:
		set_collision_layer_value(layer, true)
	for mask in inc_mask:
		set_collision_mask_value(mask, true)


func set_location(new_location: Vector2i) -> void:
	global_position = new_location	


func _process(_delta: float) -> void:
	var x_direction: float = Input.get_axis("left", "right")
	var y_direction: float = Input.get_axis("up", "down")
	if x_direction:
		velocity.x = x_direction * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
	if y_direction:
		velocity.y = y_direction * speed
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

### Turn off Player Layers and Mask

The last thing we're doing in this build, before saving and checking in code to our repo, is updating the properties for our `Player` object. Open the Player scene, select the "Player" root object in the Scene window, and use the Inspector window. We'll go down to the CollisionObject2D section and unselect all Layers and Masks.

![Unselect the Collision Layers and Masks for the Player object](/readmeassets/lesson2/player/2_unselect_all_layers_and_masks.png)

Now let's save and run the scene with F6. You'll see the Player really has no changes from before, but the code runs and the Player can move.

Next we'll integrate the updated Player and Generator into our World object and see how things work!

## World Updates

for the World Updates, we're to update the Player and the Generator components in your World project. One question you might have is "Why didn't we just do this work in the World project? We already have the Player and the Generator there." Well, yes, you could have. But, if you want to re-use these components, I find it easier to have their own project and folder structures that you can build and test in without having to load up your full game. Some of your games, like mine, will get quite large and this method makes updates to these smaller components faster. It also helps keep me focused on the change that I'm making, versus being distracted by a number of other changes.

Another challenge that updating the components in the World project is that you might accidentally tightly couple your component to the Game World or another component. This practice helps to avoid accidental cases where you reference either a property or a scene node that is not in the component. If you try to copy this component to another project, you might run into small things that need simple debugging to major refactoring to get the feature you want to work.

Before you open the World project in Godot, go to your Explorer, Finder, or other interface and copy the updated Player and Generator components into your World folder. Since our components are in their own sub-folders, this is simply copying the `Player` folder from your Player project and the `Generator` folder from your Generator project to your World project folder. Your file system should look like this:

![Updated file system for your World project should have World, Player, and Generator folders](/readmeassets/build_3/3_updated_file_system.png)

Next, we're going to open up the `world.gd` file and start coding for our updated components! First, we'll add in a resource loader for our player_body.

Now, remove the line that reads `Globals.set_player($Player)` from this file. We're going to dynamically load the player into the game and remove it from the scene shortly.

After removing that line, let's update `_on_floor_generated` to instantiate the player, register it with Globals, initialize it, and set the location. The updated function should look like this.

```
func _on_floor_generated() -> void:
	print("floor generated message received")
	var player_resource: Node2D = preload("res://Player/player.tscn").instantiate()
	Globals.set_player(player_resource)
	Globals.world.add_child(Globals.player)
	Globals.player.initialize("player", 600, [], [2])
	Globals.player.set_location(Globals.generator.get_player_start())

```

---
**NOTE**

A few notes about using preload: Preload loads the object at compile time versus load, which loads at runtime. This allows the player to be ready faster. We are also doing the preload in the function where the player is needed for the time being. As we add enemies and enemy spawn points during a floor, it might be better to preload this one time for all those uses. Also, if we had more than one floor, it might be better to have the player loaded once.

Also, the resource loader returns an object of type "Resource," and not the class of the resource itself. Node2D gives the compiler that this is a Node2D-based object. We could have also used PackedScene for the compiler hint instead of Node2D.

---

Finally, open the World scene and remove the Player node. The updated scene should look like this.

![Remove the player node from your World scene](/readmeassets/lesson2/world/3_player_removed_from_scene.png)

Now save your project and run it with F5. You'll see your player is spawned into a room and can only move on the floor. The player is stopped at the wall.

[Next Lesson](lesson3.MD)
