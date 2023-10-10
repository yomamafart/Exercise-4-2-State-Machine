# Exercise 4.2—State Machine

Exercise for MSCH-C220

A demonstration of this exercise is coming soon.

This exercise is designed to explore the possibilities of a 2D Platformer project by creating an animated sprite and a State Machine. The concepts behind this exercise will be outlined in class.

The expectations for this exercise are that you will

 - [ ] Fork and clone this repository
 - [ ] Import the project into Godot
 - [ ] Create an animated sprite for the player
 - [ ] Create the script for a state machine
 - [ ] Create states for idle, moving, jumping, and falling
 - [ ] Edit the LICENSE and README.md
 - [ ] Commit and push your changes back to GitHub. Turn in the URL of your repository on Canvas.

---

## Instructions

Fork this repository. When that process has completed, make sure that the top of the repository reads [your username]/Exercise-4-2-State-Machine. *Edit the LICENSE and replace BL-MSCH-C220 with your full name.* Commit your changes.

Clone the repository to a Local Path on your computer.

Open Godot. Import the project.godot file and open the "State Machine" project.

You should see a very basic 2D Platformer with tiles made from the Godot Logo. A Player (CharacterBody2D) has already been added with a Camera2D, an AnimatedSprite2D and CollisionShape2D, and the skeleton of a state machine. The player also has a label that hovers above his head to display its current state.

Select the AnimatedSprite in the Scene panel. In the Inspector Panel, select Frames->New SpriteFrames. Then select Frames again, and choose Edit.

The Animate Frames panel should appear at the bottom of the window. In the list on the left, rename the "default" animation to Idle (be careful about capitalization). Then press the Sprite Sheet icon (looks like a waffle) and choose res://Assets/player.png. You should now see the player sprite sheet. At the top of the window, change Horizonatal to 10 and Vertical to 5. Select the image in the top left corner, and press the "Add 1 Frame(s)" button.

Again, in the Animations list, create a new animation, and call it "Moving". Open the Sprite Sheet again, and select frames 2–7 on the top row. Add 6 Frame(s).

Create a third animation, and call it "Jumping". This should include frames 8–10 on the top row. After you have added the frames, uncheck the Loop switch.

Finally, create a fourth animation called "Falling". This should include frames 8–10 on row 2. Uncheck the loop switch for this animation, too.

Now, we need to set up the State Machine. Open the res://Player/StateMachine.gd script, and replace it with the following (this would be a good script to add to your Gists):

```
extends Node

var state = null
var previous_state = null
var state_name = ""

func _ready():
	set_state(get_children()[0].name)

func _physics_process(delta):
	if state and state.has_method("physics_process"):
		state.physics_process(delta)

func _process(delta):
	if state and state.has_method("process"):
		state.process(delta)

func _unhandled_input(event):
	if state and state.has_method("unhandled_input"):
		state.unhandled_input(event)

func set_state(s):
	state_name = s
	var new_state = get_node(NodePath(s))
	if state != null:
		if state.has_method("end"):
			state.end()
		previous_state = state
	if new_state != null:
		state = new_state
		if state.has_method("start"):
			state.start()
```

Now, add a new script to the Idle node. Save it as res://Player/Idle.gd:
```
extends Node

@onready var SM = get_parent()
@onready var player = get_node("../..")

func _ready():
	await player.ready

func start():
	player.velocity = Vector2.ZERO
	player.set_animation("Idle")

func physics_process(_delta):
	if not player.is_on_floor():
		SM.set_state("Falling")
	if Input.is_action_pressed("left") or Input.is_action_pressed("right"):
		SM.set_state("Moving")
	if Input.is_action_pressed("jump"):
		SM.set_state("Jumping")
```

Now, add a new script to the Moving node. Save it as res://Player/Moving.gd:
```
extends Node

@onready var SM = get_parent()
@onready var player = get_node("../..")

func _ready():
	await player.ready

func start():
	player.set_animation("Moving")

func physics_process(_delta):
	if not player.is_on_floor():
		SM.set_state("Falling")
	if Input.is_action_pressed("jump"):
		SM.set_state("Jumping")

	var direction = Input.get_axis("left", "right")
	player.set_direction(direction)
	if direction:
		player.velocity.x = player.direction * player.SPEED
	else:
		player.velocity.x = move_toward(player.velocity.x, 0, player.SPEED)
	player.move_and_slide()

	if direction == 0:
		SM.set_state("Idle")
```

Add a script to the Jumping node. Save it as res://Player/Jumping.gd:
```
extends Node

@onready var SM = get_parent()
@onready var player = get_node("../..")

func _ready():
	await player.ready

func start():
	player.set_animation("Jumping")

func physics_process(_delta):
	if Input.is_action_pressed("jump") and player.velocity.y > player.MAX_JUMP:
		player.velocity.y += player.JUMP_VELOCITY
		player.move_and_slide()
	else:
		SM.set_state("Falling")
```

Finally, add a script to the Falling node. Save it as res://Player/Falling.gd:
```
extends Node

@onready var SM = get_parent()
@onready var player = get_node("../..")

func _ready():
	await player.ready

func start():
	player.set_animation("Falling")

func physics_process(_delta):
	player.velocity.y += player.gravity * _delta
	var direction = Input.get_axis("left", "right")
	if direction: 
		player.velocity.x = player.direction * player.SPEED
	else: 
		player.velocity.x = move_toward(player.velocity.x, 0, player.SPEED)

	player.set_direction(direction)
	player.move_and_slide()

	if player.is_on_floor():
		player.velocity.y = 0
		SM.set_state("Idle")
	if player.is_on_ceiling():
		player.velocity.y = 0
```

Test the project. You should be able to see the different states as the character moves, falls, or jumps.

Quit Godot. In GitHub desktop, add a summary message, commit your changes and push them back to GitHub. If you return to and refresh your GitHub repository page, you should now see your updated files with the time when they were changed.

Now edit the README.md file. When you have finished editing, commit your changes, and then turn in the URL of the main repository page (https://github.com/[username]/Exercise-4-2-State-Machine) on Canvas.

The final state of the file should be as follows (replacing the "Created by" information with your name):
```
# Exercise 4.2—State-Machine

Exercise for MSCH-C220

An exercise for the 2D Platformer project, exploring AnimatedSprite and a simple state machine.

## Implementation

Built using Godot 4.1.1

The sprite is an adaptation of [MV Platformer Male](https://opengameart.org/content/mv-platformer-male-32x64) by MoikMellah. CC0 Licensed.

## References

None

## Future Development

None

## Created by 

Jason Francis
```
