import pygame
import cv2
import numpy as np

pygame.init()
WIDTH, HEIGHT = 650, 650
WINDOW = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Cover Cat =^-^=")


BG = cv2.VideoCapture("Untitled_Artwork.mp4")


CHAR_IMG = pygame.image.load("cat_char.png").convert_alpha()
CHAR_IMG = pygame.transform.scale(CHAR_IMG, (500, 500))


PLAYER_WIDTH = 400
PLAYER_HEIGHT = 250


def show_title_screen():
    font = pygame.font.SysFont('comicsans', 60)
    label = font.render('Cover Cat', True, "white", "pink")
    
    showing = True
    while showing:
        WINDOW.fill("pink")  
        WINDOW.blit(label, (200, 200))  

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.MOUSEBUTTONDOWN:  
                showing = False

        pygame.display.update()


def main():
    show_title_screen() 

    clock = pygame.time.Clock()
    run = True

  
    player = pygame.Rect(230, HEIGHT - PLAYER_HEIGHT - 20, PLAYER_WIDTH, PLAYER_HEIGHT)
    speed = 5  # How fast the character moves

    while run:
        ret, frame = BG.read()
        if not ret:
            BG.set(cv2.CAP_PROP_POS_FRAMES, 0)
            continue

        frame = cv2.resize(frame, (WIDTH, HEIGHT))
        frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        frame = np.rot90(frame)
        bg_surface = pygame.surfarray.make_surface(frame)
        bg_surface = pygame.transform.rotate(bg_surface, -360)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

       
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] or keys[pygame.K_a]:
            player.x -= speed
        if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            player.x += speed

        # Keep cat on screen
        if player.x < 0:
            player.x = 0
        if player.x > WIDTH - PLAYER_WIDTH:
            player.x = WIDTH - PLAYER_WIDTH

        WINDOW.blit(bg_surface, (0, 0))  
        WINDOW.blit(CHAR_IMG, (player.x, player.y))  

        pygame.display.update()
        clock.tick(10)

    BG.release()
    pygame.quit()

if __name__ == "__main__":
    main()
