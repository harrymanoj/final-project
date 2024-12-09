# final-project
import pygame
import sys
import time
import random

# Initialize Pygame
pygame.init()

# Screen dimensions and colors
SCREEN_WIDTH, SCREEN_HEIGHT = 900, 700
WHITE, BLACK, RED, GREEN, GRAY = (255, 255, 255), (0, 0, 0), (255, 0, 0), (0, 255, 0), (50, 50, 50)

# Fonts
FONT = pygame.font.SysFont("Arial", 30)
FONT_SMALL = pygame.font.SysFont("Arial", 24)

# Initialize screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Escape Room - Horror Edition")

# Game variables
lifelines = 3
time_limit = 30

# Utility functions
def display_text(text, x, y, font, color=WHITE, center=False):
    """Display text on the screen."""
    label = font.render(text, True, color)
    rect = label.get_rect()
    if center:
        rect.center = (x, y)
    else:
        rect.topleft = (x, y)
    screen.blit(label, rect)

def reset_game():
    """Reset game variables."""
    global lifelines
    lifelines = 3

# Game over screen
def game_over():
    """Display the game over screen."""
    while True:
        screen.fill(BLACK)
        display_text("Game Over!", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 3, FONT, RED, center=True)
        display_text("Press 'R' to restart or 'Q' to quit.", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2, FONT_SMALL, center=True)
        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    return True
                elif event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()

               

# Login screen
def login_screen():
    """Display the login screen and handle name input."""
    name = ""
    input_box = pygame.Rect(SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2, 300, 50)
    active = False

    while True:
        screen.fill(BLACK)
        display_text("Escape Room - Horror Edition", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 4, FONT, RED, center=True)
        display_text("Enter your name to start:", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 3, FONT_SMALL, center=True)
        pygame.draw.rect(screen, WHITE, input_box, 2)
        display_text(name, input_box.x + 10, input_box.y + 10, FONT_SMALL, WHITE)

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                active = input_box.collidepoint(event.pos)
            if event.type == pygame.KEYDOWN and active:
                if event.key == pygame.K_RETURN and name.strip():
                    return name.strip()  # Return name to start the game
                elif event.key == pygame.K_BACKSPACE:
                    name = name[:-1]  # Remove the last character
                else:
                    name += event.unicode  # Add typed character

# Level functions
def ask_question(question, answer):
    """Handle a single question."""
    global lifelines

    user_input = ""
    start_time = time.time()

    while True:
        elapsed_time = int(time.time() - start_time)
        remaining_time = max(0, time_limit - elapsed_time)

        screen.fill(GRAY)
        display_text(f"Time Left: {remaining_time}s", 20, 20, FONT_SMALL, RED)
        display_text(f"Lifelines: {lifelines}", SCREEN_WIDTH - 150, 20, FONT_SMALL, GREEN)
        display_text(question, SCREEN_WIDTH // 2, SCREEN_HEIGHT // 3, FONT, BLACK, center=True)
        pygame.draw.rect(screen, WHITE, pygame.Rect(SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2, 300, 40))
        display_text(user_input, SCREEN_WIDTH // 2 - 140, SCREEN_HEIGHT // 2 + 5, FONT_SMALL, BLACK)

        pygame.display.flip()

        if remaining_time == 0:
            lifelines -= 1
            display_text("Time's up! Lifeline lost.", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 1.5, FONT, RED, center=True)
            pygame.display.flip()
            time.sleep(2)
            return False

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:
                    if user_input.strip().lower() == answer.lower():
                        display_text("Correct!", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 1.5, FONT, GREEN, center=True)
                        pygame.display.flip()
                        time.sleep(1)
                        return True
                    else:
                        lifelines -= 1
                        display_text("Wrong! Lifeline lost.", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 1.5, FONT, RED, center=True)
                        pygame.display.flip()
                        time.sleep(2)
                        return False
                elif event.key == pygame.K_BACKSPACE:
                    user_input = user_input[:-1]
                else:
                    user_input += event.unicode

def level_1():
    """Level 1: Logical questions."""
    global lifelines
    questions = [
        ("What is the name of the killer in 'Friday the 13th'?", "Jason"),
        ("What year was 'The Shining' released?", "1980"),
        ("Who is the haunted doll in 'The Conjuring'?", "Annabelle"),
    ]
    random.shuffle(questions)

    for question, answer in questions:
        if lifelines == 0:
            return False
        if not ask_question(question, answer):
            if lifelines == 0:
                return False
    return True

def level_2():
    """Level 2: Riddles."""
    global lifelines
    riddles = [
        ("I speak without a mouth and hear without ears. What am I?", "echo"),
        ("The more you take, the more you leave behind. What am I?", "footsteps"),
        ("What has keys but can't open locks?", "piano"),
    ]
    random.shuffle(riddles)

    for riddle, answer in riddles:
        if lifelines == 0:
            return False
        if not ask_question(riddle, answer):
            if lifelines == 0:
                return False
    return True

def level_3():
    """Level 3: Code decryption."""
    global lifelines
    encryptions = [
        ("If B=2, C=3, decode 72 in Caesar cipher shifted by 1.", "g"),
        ("Decode this Morse: .... . .-.. .-.. ---", "hello"),
        ("Decode 1100100 to ASCII.", "d"),
    ]
    random.shuffle(encryptions)

    for task, answer in encryptions:
        if lifelines == 0:
            return False
        if not ask_question(task, answer):
            if lifelines == 0:
                return False
    return True

# Game loop
def main():
    player_name = login_screen()

    while True:
        reset_game()

        if not level_1():
            if not game_over():
                break
        else:
            screen.fill(GRAY)
            display_text("Congratulations! You passed Level 1.", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2, FONT, GREEN, center=True)
            pygame.display.flip()
            time.sleep(2)

        if not level_2():
            if not game_over():
                break
        else:
            screen.fill(GRAY)
            display_text("Great job! You passed Level 2.", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2, FONT, GREEN, center=True)
            pygame.display.flip()
            time.sleep(2)

        if not level_3():
            if not game_over():
                break
        else:
            screen.fill(GRAY)
            display_text("You did it! You escaped!", SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2, FONT, GREEN, center=True)
            pygame.display.flip()
            time.sleep(3)
            break
       
   

# Start game
main()
pygame.quit()
sys.exit()
