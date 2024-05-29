# Python_Arcade_Game
A basic Arcade Game written in Python

#!/usr/bin/env python
# coding: utf-8

# ## Introduction to PyGame

# In[1]:


import subprocess

subprocess.run(['pip3', 'install', '--upgrade', 'pip'])
# Installing PyGame into your virtual environment
subprocess.run(['pip3', 'install', 'pygame'])

from tests import tests

# In[2]:


# Importing PyGame library into the project
import pygame
# Importing random library
import random
# Importing constants for the game controls
from pygame.constants import QUIT, K_DOWN, K_UP, K_LEFT, K_RIGHT

# In[3]:


tests.ch1()

# ## Defining Constants

# In[5]:


# PyGame initialization
pygame.font.init()
pygame.display.init()

# CONSTANTS
FPS = pygame.time.Clock()  # Frame Per Second - used for managing how fast your game runs

HEIGHT = 800
WIDTH = 1200

COLOR_WHITE = (255, 255, 255)
FONT = pygame.font.SysFont('Verdana', 20)

# In[6]:


tests.ch3()

# ## PyGame Window Creation

# In[7]:


# PyGame window Creation

# Display size
main_display = pygame.display.set_mode((WIDTH, HEIGHT))
# Background settings
bg = pygame.transform.scale(pygame.image.load('space.png'), (WIDTH, HEIGHT))
bg_x1 = 0
bg_x2 = bg.get_width()
bg_move = 1

# In[8]:


tests.ch5()

# ## Handling Player Input

# In[9]:


# Handling Player Input

# Load the player image.
player = pygame.transform.scale(pygame.image.load('marvel.png').convert_alpha(), (100, 100))
# Create a rectangle for the player, which will be used for positioning and collision detection.
player_rect = player.get_rect()
# Set the player's starting position
player_rect.center = main_display.get_rect().center

# Define movement vectors for the player.
player_move_down = [0, 4]
player_move_up = [0, -4]
player_move_left = [-4, 0]
player_move_right = [4, 0]

# In[10]:


tests.ch7()


# ## Generating and Managing Enemies

# In[11]:


def create_enemy():
    # Load and resize the enemy image
    enemy = pygame.transform.scale(pygame.image.load('rocket.png'), (100, 50))
    # Create a rectangle for the enemy. It starts off-screen to the right and at a random vertical position.
    enemy_rect = pygame.Rect(WIDTH, random.randint(-enemy.get_height(), HEIGHT - enemy.get_height()), *enemy.get_size())
    # Define the enemy's movement vector. The enemy moves horizontally to the left at a random speed.
    enemy_move = [random.randint(-6, -1), 0]

    return [enemy, enemy_rect, enemy_move]


# In[12]:


tests.ch9()


# ## Creating and Collecting Bonuses

# In[13]:


def create_bonus():
    bonus = pygame.transform.scale(pygame.image.load('crystal.png'), (45, 75))
    bonus_rect = pygame.Rect(random.randint(-bonus.get_height(), WIDTH), 0, *bonus.get_size())
    bonus_move = [0, random.randint(1, 5)]
    return [bonus, bonus_rect, bonus_move]


# In[14]:


tests.ch11()

# ## Managing Game Events

# In[15]:


# Define a custom event for creating new enemies, adding 1 to the base USEREVENT.
CREATE_ENEMY = pygame.USEREVENT + 1
# Set a timer to trigger the CREATE_ENEMY event every 1500 milliseconds (1.5 seconds).
pygame.time.set_timer(CREATE_ENEMY, 1500)

# Define another custom event for creating bonuses, adding 2 to the base USEREVENT.
CREATE_BONUS = pygame.USEREVENT + 2
pygame.time.set_timer(CREATE_BONUS, 3000)

# Initialize empty lists to keep track of all enemies and bonuses on the screen.
enemies = []
bonuses = []
# Initialize the player's score to 0 at the start of the game.
score = 0

# Set the playing flag to True to start the game loop.
playing = True

# In[16]:


tests.ch13()

# ## Handling Game Logic

# In[32]:


while playing:
    # Limit the game to 120 frames per second to ensure smooth gameplay.
    FPS.tick(240)

    # Process all events in the event queue.
    for event in pygame.event.get():
        # If the QUIT event is triggered (e.g., closing the window), stop the game.
        if event.type == QUIT:
            playing = False
            pygame.display.flip()
            pygame.quit()

        # Check for the custom CREATE_ENEMY event to generate a new enemy.
        if event.type == CREATE_ENEMY:
            # Create a new enemy and add it to the list of enemies.
            enemies.append(create_enemy())

        # Check for the custom CREATE_BONUS event to generate a new bonus.
        if event.type == CREATE_BONUS:
            bonuses.append(create_bonus())

    # Move the background images leftward to create a scrolling effect.
    bg_x1 -= bg_move
    bg_x2 -= bg_move

    # If a background image has scrolled completely out of view, reset its position.
    if bg_x1 < -bg.get_width():
        bg_x1 = bg.get_width()

    if bg_x2 < -bg.get_width():
        bg_x2 = bg.get_width()

    # Draw the background images at their updated positions for the scrolling effect.
    main_display.blit(bg, (bg_x1, 0))
    main_display.blit(bg, (bg_x2, 0))

    # Detect which keys are pressed down.
    keys = pygame.key.get_pressed()

    # Player movement conditions. Adjust the player's position based on key inputs.
    if keys[K_DOWN] and player_rect.bottom < HEIGHT:
        player_rect = player_rect.move(player_move_down)

    if keys[K_UP] and player_rect.top > 0:
        player_rect = player_rect.move(player_move_up)

    if keys[K_RIGHT] and player_rect.right < WIDTH:
        player_rect = player_rect.move(player_move_right)

    if keys[K_LEFT] and player_rect.left > 0:
        player_rect = player_rect.move(player_move_left)

    # Enemy logic: move each enemy and check for collisions with the player.
    for enemy in enemies:
        enemy[1] = enemy[1].move(enemy[2])
        main_display.blit(enemy[0], enemy[1])  # Draw the enemy on the screen.

        if player_rect.colliderect(enemy[1]):  # Check for collision with the player.
            playing = False  # End the game if there's a collision.
            break  # Exit the loop early upon collision.

    for bonus in bonuses:
        bonus[1] = bonus[1].move(bonus[2])
        main_display.blit(bonus[0], bonus[1])

        if player_rect.colliderect(bonus[1]):
            score += 1  # Increase score.
            bonuses.pop(bonuses.index(bonus))  # Remove collected bonus.

    # Display the current score in the top right corner and draw the player at its current position.
    main_display.blit(FONT.render(str(score), True, COLOR_WHITE), (WIDTH - 50, 20))
    main_display.blit(player, player_rect)

    # Refresh the screen to show updated positions and score.
    pygame.display.flip()

    # Remove enemies and bonuses that have moved off the screen.
    for enemy in enemies:
        if enemy[1].left < 0:
            enemies.pop(enemies.index(enemy))

    for bonus in bonuses:
        if bonus[1].bottom > HEIGHT:
            bonuses.pop(bonuses.index(bonus))

# In[33]:


tests.ch15()

