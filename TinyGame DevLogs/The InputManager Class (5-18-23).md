Oh god. So I wanted to keep this stuff relatively simple so I could move on to *actually making the game* but as I keep learning again and again: perfection is the enemy of progress (and this code is far from perfection anyways so... chock it up to a learning experience I guess).

### UI and Menus
Let's take a step back and think about the first actions of the game itself. The goal is to be able to accept input from any type of device at any time when the first menus appear to start the game. For *UI* this is relatively simple because we can just accept input from any device anytime that sends the proper input mapping ("ui_right", "ui_left", etc).  

### Game Input
Where things get more complicated is how we manage input during the actual gameplay. The idea is that when we first arrive at our beach scene the playing field will only contain our player (the one that was controlling the menus to get the game started in the first place). This means that initially we'll wanna keep track of the main "player 1" input leading into opening initial game stage to assign it to player 1. 

At this point, the game will be waiting for input from *new input devices* in order to place a player on the field that corresponds to that device. Once this is done the player and device will get added to some list that will only change when either the device is disconnected or the player leaves the game. These devices can be switched between players (as a feature in the future potentially).