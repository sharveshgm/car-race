### Car Race
<img width="1203" height="947" alt="img" src="https://github.com/user-attachments/assets/f1c80e1d-6e9b-476c-966d-853becf3d206" />

# Here’s **line-by-line explanation **

***

### Top: Imports, Author Block

```python
import random            # Lets us pick random numbers (for enemy car positions)
from time import sleep   # Lets us pause the game for a while

import pygame            # Main library for creating the game window, drawing, and controls
from pathlib2 import Path # For handling file paths in a cross-platform way
```


***

### The CarRacing Class

Everything happens inside this class.

```python
class CarRacing:
    def __init__(self):
        pygame.init()  # Starts pygame (must be called before anything else)
        self.display_width = 800   # Game window width in pixels
        self.display_height = 600  # Game window height in pixels
        self.black = (0, 0, 0)     # Defines the color black (for background)
        self.white = (255, 255, 255) # Defines the color white (for text)
        self.clock = pygame.time.Clock()  # Used to control the game's speed
        self.gameDisplay = None    # Will hold the pygame game window
        self.root_path = str(Path(__file__).parent) # This is the folder where the script is

        self.initialize()          # Runs the function to set the starting positions & images
```


***

### Game Setup: Positions, Images, First Values

```python
    def initialize(self):

        self.crashed = False   # Game keeps running until crashed becomes True

        self.carImg = pygame.image.load(self.root_path + "/img/car.png")
        # Loads your car image from the img folder
        self.car_x_coordinate = (self.display_width * 0.45)  # Car starts at 45% of the screen width from left
        self.car_y_coordinate = (self.display_height * 0.8)  # Car starts near the bottom (80% down)
        self.car_width = 49  # Car's width (in pixels)

        # Enemy car properties
        self.enemy_car = pygame.image.load(self.root_path + "/img/enemy_car_1.png")
        self.enemy_car_startx = random.randrange(310, 450)   # Enemy starts randomly between these two x positions
        self.enemy_car_starty = -600       # Enemy starts above the screen so it "falls" down
        self.enemy_car_speed = 5           # How fast the enemy moves down
        self.enemy_car_width = 49
        self.enemy_car_height = 100

        # Background image and scrolling setup
        self.bgImg = pygame.image.load(self.root_path + "/img/back_ground.jpg")
        self.bg_x1 = (self.display_width / 2) - (360 / 2)    # Center the background horizontally
        self.bg_x2 = (self.display_width / 2) - (360 / 2)    # For second scrolling background
        self.bg_y1 = 0       # First background's y position
        self.bg_y2 = -600    # Second background sits just above the first (for scrolling)
        self.bg_speed = 3    # How quickly the road scrolls
        self.count = 0       # Your score (counts how many enemies passed)
```


***

### The Car Function

Draws the car at its current position.

```python
    def car(self, car_x_coordinate, car_y_coordinate):
        self.gameDisplay.blit(self.carImg, (car_x_coordinate, car_y_coordinate))
```

*`blit` means "draw this image at these coordinates on the screen."*

***

### Opening the Game Window

```python
    def racing_window(self):
        self.gameDisplay = pygame.display.set_mode((self.display_width, self.display_height))
        # Sets up the game window size
        pygame.display.set_caption('Car Race -- Sharvesh.G')
        # Title at the top of the window
        self.run_car()  # Start the actual game loop
```


***

### Main Game Loop

```python
    def run_car(self):

        while not self.crashed: # Game keeps running until crashed becomes True

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.crashed = True  # If you close the window, end the game

                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT:
                        self.car_x_coordinate -= 50  # Move left by 50 pixels
                        print("CAR X COORDINATES: %s" % self.car_x_coordinate)
                    if event.key == pygame.K_RIGHT:
                        self.car_x_coordinate += 50  # Move right by 50 pixels
                        print("CAR X COORDINATES: %s" % self.car_x_coordinate)
                    print("x: {x}, y: {y}".format(x=self.car_x_coordinate, y=self.car_y_coordinate))
```

*This lets you move your car left or right using arrow keys.*

***

### Drawing Everything

```python
            self.gameDisplay.fill(self.black)  # Clears the screen
            self.back_ground_road()            # Draws the scrolling road

            self.run_enemy_car(self.enemy_car_startx, self.enemy_car_starty)  # Draws the enemy car
            self.enemy_car_starty += self.enemy_car_speed  # Moves the enemy car down the screen each frame

            if self.enemy_car_starty > self.display_height:
                self.enemy_car_starty = 0 - self.enemy_car_height  # If enemy goes off bottom, restart at top
                self.enemy_car_startx = random.randrange(310, 450) # In a new random lane

            self.car(self.car_x_coordinate, self.car_y_coordinate)  # Draws your car at its current position
            self.highscore(self.count)    # Shows your score
            self.count += 1               # Adds to your score each frame

            if self.count % 100 == 0:     # Every 100 frames, speed up the game
                self.enemy_car_speed += 1
                self.bg_speed += 1
```


***

### Collision Detection

```python
            if self.car_y_coordinate < self.enemy_car_starty + self.enemy_car_height:
                # Checks if your car is as high as/below the enemy car
                # Now check if they're overlapping left-right also
                if (
                    self.car_x_coordinate > self.enemy_car_startx and
                    self.car_x_coordinate < self.enemy_car_startx + self.enemy_car_width or
                    self.car_x_coordinate + self.car_width > self.enemy_car_startx and
                    self.car_x_coordinate + self.car_width < self.enemy_car_startx + self.enemy_car_width
                ):
                    self.crashed = True
                    self.display_message("Game Over !!!")
```

*If your car's rectangle and the enemy car's rectangle overlap, you crash!*

***

### Boundary Check

```python
            if self.car_x_coordinate < 310 or self.car_x_coordinate > 460:
                # If your car drives outside the road
                self.crashed = True
                self.display_message("Game Over !!!")
```


***

### Drawing and Timing

```python
            pygame.display.update() # Updates the screen with everything you've drawn
            self.clock.tick(60)     # Limits the game to 60 frames per second
```


***

### Game Over and Restart

```python
    def display_message(self, msg):
        font = pygame.font.SysFont("comicsansms", 72, True)
        text = font.render(msg, True, (255, 255, 255))
        self.gameDisplay.blit(text, (400 - text.get_width() // 2, 240 - text.get_height() // 2))
        self.display_credit()
        pygame.display.update()
        self.clock.tick(60)
        sleep(1)
        car_racing.initialize()
        car_racing.racing_window()
```

*Shows GAME OVER, credits, and after a small pause, restarts the game.*

***

### Scrolling Road Function

```python
    def back_ground_road(self):
        self.gameDisplay.blit(self.bgImg, (self.bg_x1, self.bg_y1))
        self.gameDisplay.blit(self.bgImg, (self.bg_x2, self.bg_y2))

        self.bg_y1 += self.bg_speed    # Both backgrounds move downwards
        self.bg_y2 += self.bg_speed

        if self.bg_y1 >= self.display_height:
            self.bg_y1 = -600         # If it goes off the bottom, start again at top (-600)
        if self.bg_y2 >= self.display_height:
            self.bg_y2 = -600
```

*Creates a continuous moving road effect using two background images.*

***

### Drawing Enemy Car

```python
    def run_enemy_car(self, thingx, thingy):
        self.gameDisplay.blit(self.enemy_car, (thingx, thingy))
```


***

### Displaying the Score

```python
    def highscore(self, count):
        font = pygame.font.SysFont("lucidaconsole", 20)
        text = font.render("Score : " + str(count), True, self.white)
        self.gameDisplay.blit(text, (0, 0))
```


***

### Credits

```python
    def display_credit(self):
        font = pygame.font.SysFont("lucidaconsole", 14)
        text = font.render("Thanks & Regards,", True, self.white)
        self.gameDisplay.blit(text, (600, 520))
        text = font.render("Sharvesh.G", True, self.white)
        self.gameDisplay.blit(text, (600, 540))
        text = font.render("sharveshganesamoorthy@gmail.com", True, self.white)
        self.gameDisplay.blit(text, (600, 560))
```

*Shows "who made this" at the Game Over screen.*

***

### Main Section

```python
if __name__ == '__main__':
    car_racing = CarRacing()
    car_racing.racing_window()
```

*When you run the program, it creates a `CarRacing` object and starts the game.*

***
