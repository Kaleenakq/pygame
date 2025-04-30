import pygame
import cv2
import numpy as np
import random
import time

# Initialize Pygame
pygame.init()

# Audio background
pygame.mixer.init()
pygame.mixer.music.load("CHEETO - THE GAME (1).wav")
pygame.mixer.music.set_volume(0.5)
pygame.mixer.music.play(-1)

#end game sound
pygame.mixer.init()
explosion_sound = pygame.mixer.Sound("explosion-80108.mp3")
explosion_played = False

# Set window size
WIDTH, HEIGHT = 650, 650
WINDOW = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Cover Cat =^-^=")

# Load background video
BG = cv2.VideoCapture("Untitled_Artwork (1).mp4")

# Load character animation video
CHAR_VIDEO = cv2.VideoCapture("Untitled_Artwork (2).mp4")
char_frames = []
while True:
    ret, frame = CHAR_VIDEO.read()
    if not ret:
        break
    frame = cv2.resize(frame, (100, 100))
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    frame = np.rot90(frame)
    surface = pygame.surfarray.make_surface(frame)
    surface = pygame.transform.rotate(surface, 360)
    char_frames.append(surface)
CHAR_VIDEO.release()

# Load enemy image
ENEMY_IMG = pygame.image.load("Untitled_Artwork (4).png")
ENEMY_IMG = pygame.transform.scale(ENEMY_IMG, (60, 60))

# Character and hitbox size
PLAYER_WIDTH = 60
PLAYER_HEIGHT = 80

# Hitbox settings
HITBOX_WIDTH = 30
HITBOX_HEIGHT = 30
HITBOX_OFFSET_X = 50
HITBOX_OFFSET_Y = 40

# Enemy class
class Enemy:
    def __init__(self, x, y, speed):
        self.rect = pygame.Rect(x, y, 60, 60)
        self.speed = speed

    def update(self):
        self.rect.y += self.speed

    def draw(self, surface):
        surface.blit(ENEMY_IMG, (self.rect.x, self.rect.y))

# Title screen
def show_title_screen():
    font = pygame.font.SysFont('comicsans', 80)
    small_font = pygame.font.SysFont('comicsans', 30)
    label = font.render('Cover Cat', True, "white", "pink")
    click_label = small_font.render('click anywhere to start', True, "orange")

    instructions_btn = pygame.Rect(200, 300, 250, 50)
    lore_btn = pygame.Rect(200, 370, 250, 50)

    instructions_text = [
        "Use LEFT and RIGHT arrow keys",
        "or A and D to move.",
        "Take cover from falling fish!",
        "Survie as long as possible"
        
    ]

    lore_text = [
        "The cat named Cheeto is made of Cheeto dust.",
        "He only eats Cheetos. He evolved into a Cheeto",
        "because his mom only fed him Cheetos as a kitten.",
        "After using 8 of his 9 lives, he must take cover",
        "from fish that rain from the sky.",
        "If he eats a fish, he uses his last life.",
        "Don't kill Cheeto!",
        "Take cover. Stay alive."
    ]

    showing = True
    show_instructions = False
    show_lore = False

    while showing:
        WINDOW.fill("pink")
        WINDOW.blit(label, (150, 100))
        if not (show_instructions or show_lore):
            WINDOW.blit(click_label, (150, 180))

        pygame.draw.rect(WINDOW, "white", instructions_btn)
        pygame.draw.rect(WINDOW, "white", lore_btn)

        inst_text = small_font.render("Instructions", True, "pink")
        lore_text_label = small_font.render("Lore", True, "pink")
        WINDOW.blit(inst_text, (instructions_btn.x + 30, instructions_btn.y + 10))
        WINDOW.blit(lore_text_label, (lore_btn.x + 90, lore_btn.y + 10))
        tiny_font = pygame.font.SysFont('comicsans', 20)
        if show_instructions:
            draw_modal(WINDOW, small_font, instructions_text)
        elif show_lore:
            draw_modal(WINDOW, tiny_font, lore_text)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                mouse_pos = pygame.mouse.get_pos()
                if instructions_btn.collidepoint(mouse_pos):
                    show_instructions = not show_instructions
                    show_lore = False
                elif lore_btn.collidepoint(mouse_pos):
                    show_lore = not show_lore
                    show_instructions = False
                elif not (instructions_btn.collidepoint(mouse_pos) or lore_btn.collidepoint(mouse_pos)):
                    if not (show_instructions or show_lore):
                        showing = False

        pygame.display.update()

def draw_modal(surface, font, lines):
    modal_rect = pygame.Rect(50, 220, 550, 300)
    pygame.draw.rect(surface, "white", modal_rect)
    pygame.draw.rect(surface, "orange", modal_rect, 3)

    for i, line in enumerate(lines):
        text = font.render(line, True, "orange")
        surface.blit(text, (modal_rect.x + 10, modal_rect.y + 20 + i * 30))

def show_game_over_screen():
    font = pygame.font.SysFont('comicsans', 40)
    small_font = pygame.font.SysFont('comicsans', 20)

    game_over_text = font.render('Oh no! You killed Cheeto!', True, "white")
    retry_text = small_font.render('click anywhere to restart', True, "orange")

    showing = True
    explosion_sound.play()
    while showing:
        WINDOW.fill("red")
        WINDOW.blit(game_over_text, (50, 200))
        WINDOW.blit(retry_text, (120, 300))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                showing = False

        pygame.display.update()

def main():
    while True:
        show_title_screen()

        clock = pygame.time.Clock()
        player = pygame.Rect(250, HEIGHT - PLAYER_HEIGHT - 20, PLAYER_WIDTH, PLAYER_HEIGHT)
        speed = 5

        enemies = []
        enemy_spawn_rate = 875  # ms
        enemy_speed = 5
        last_enemy_spawn_time = pygame.time.get_ticks()
        last_difficulty_increase_time = time.time()

        font = pygame.font.SysFont('comicsans', 27)

        char_frame_index = 0
        char_frame_tick = 0
        char_frame_delay = 4

        run = True
        start_time = time.time()

        while run:
            ret, frame = BG.read()
            if not ret:
                BG.set(cv2.CAP_PROP_POS_FRAMES, 0)
                continue

            frame = cv2.resize(frame, (WIDTH, HEIGHT))
            frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            frame = np.rot90(frame)
            bg_surface = pygame.surfarray.make_surface(frame)

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    exit()

            keys = pygame.key.get_pressed()
            if keys[pygame.K_LEFT] or keys[pygame.K_a]:
                player.x -= speed
            if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
                player.x += speed

            # Keep player inside screen
            player.x = max(0, min(player.x, WIDTH - player.width))

            # Handle difficulty scaling
            elapsed_since_difficulty = time.time() - last_difficulty_increase_time
            if elapsed_since_difficulty >= 5:
                last_difficulty_increase_time = time.time()
                enemy_spawn_rate = max(100, enemy_spawn_rate - 100)
                enemy_speed = min(10, enemy_speed + .5)

            # Handle enemy spawning
            current_time = pygame.time.get_ticks()
            if current_time - last_enemy_spawn_time >= enemy_spawn_rate:
                x_pos = random.randint(0, WIDTH - 60)
                enemies.append(Enemy(x_pos, -60, enemy_speed))
                last_enemy_spawn_time = current_time

            # Update enemies
            for enemy in enemies[:]:
                enemy.update()
                if enemy.rect.top > HEIGHT:
                    enemies.remove(enemy)

            # Update character animation
            char_frame_tick += 1
            if char_frame_tick >= char_frame_delay:
                char_frame_tick = 0
                char_frame_index = (char_frame_index + 1) % len(char_frames)

            # Draw everything
            WINDOW.blit(bg_surface, (0, 0))
            WINDOW.blit(char_frames[char_frame_index], (player.x, player.y))

            hitbox = pygame.Rect(
                player.x + HITBOX_OFFSET_X,
                player.y + HITBOX_OFFSET_Y,
                HITBOX_WIDTH,
                HITBOX_HEIGHT
            )
            #pygame.draw.rect(WINDOW, "orange", hitbox, 2)  # hitbox for testing

            for enemy in enemies:
                enemy.draw(WINDOW)
                if hitbox.colliderect(enemy.rect):
                    run = False

            # Display timer
            elapsed_time = int(time.time() - start_time)
            timer_label = font.render(f"Time: {elapsed_time}s", True, "red")
            WINDOW.blit(timer_label, (10, 10))

            pygame.display.update()
            clock.tick(60)

        show_game_over_screen()

if __name__ == "__main__":
    main()
