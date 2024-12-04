import pygame as pg
import random as rd

clock = pg.time.Clock()

pg.init()
display_width = 1300
display_height = 790
win = pg.display.set_mode((1300, 790), flags=pg.NOFRAME)
pg.display.set_caption('Mario')
pg.display.set_icon(pg.image.load('images/Details/incon.png'))
bg = pg.image.load('images/Details/fon.png').convert_alpha()

mushroom_images = [
    pg.image.load('images/details/mushroom1.png').convert_alpha(),
    pg.image.load('images/details/mushroom2.png').convert_alpha()
]

cloud_images = [
    pg.image.load('images/Cloud/cloud1.png').convert_alpha(),
    pg.image.load('images/Cloud/cloud2.png').convert_alpha(),
    pg.image.load('images/Cloud/cloud3.png').convert_alpha()
]

walk_right = [
    pg.image.load('images/mario/mario1.png').convert_alpha(),
    pg.image.load('images/mario/mario2.png').convert_alpha(),
    pg.image.load('images/mario/mario3.png').convert_alpha(),
    pg.image.load('images/mario/mario4.png').convert_alpha(),
    pg.image.load('images/mario/mario5.png').convert_alpha(),
    pg.image.load('images/mario/mario6.png').convert_alpha(),
    pg.image.load('images/mario/mario7.png').convert_alpha()
]

walk_left = [
    pg.image.load('images/mario2/mario1.png').convert_alpha(),
    pg.image.load('images/mario2/mario2.png').convert_alpha(),
    pg.image.load('images/mario2/mario3.png').convert_alpha(),
    pg.image.load('images/mario2/mario4.png').convert_alpha(),
    pg.image.load('images/mario2/mario5.png').convert_alpha(),
    pg.image.load('images/mario2/mario6.png').convert_alpha(),
    pg.image.load('images/mario2/mario7.png').convert_alpha()
]


text = 'Game Over'
color = (152, 0, 2)
x = 500
y = 310

font = pg.font.Font('images/Details/text.ttf', 70)
paused_text = font.render('The game is suspended', True, (0, 32, 255))


def draw_text(surface, text, color, x, y):
    textobj = font.render(text, 1, color)
    textrect = textobj.get_rect()
    textrect.topleft = (x, y)
    surface.blit(textobj, textrect)


def draw_paused_win():
    win.blit(paused_text, paused_text.get_rect(center=(600, 350)))
    pg.display.flip()


anim_count = 0  # картинка из списка не меняется
player_speed = 8  # скорость передвижение Марио
player_x, player_y = 10, 192
mushroom_x, mushroom_y = 300, 203


jump_count = 5


bg_sound = pg.mixer.Sound('sounds/background.mp3')
bg_sound.play(loops=-1)
jump_sound = pg.mixer.Sound('sounds/jump.mp3')
loss_sound = pg.mixer.Sound('sounds/gameover.mp3')
pause_sound = pg.mixer.Sound('sounds/pause.mp3')


game_paused = False
jump = False
game_over = False
running = True

while running:

    win.blit(bg, (0, 0))

    win.blit(mushroom_images[0], (mushroom_x, mushroom_y))
    win.blit(mushroom_images[1], (840, 464))
    win.blit(mushroom_images[1], (400, 728))

    #win.blit(cloud_images[0], (400, 5))

    player_rect = walk_left[0].get_rect(topleft=(player_x, player_y))
    mushroom_rect = mushroom_images[0].get_rect(topleft=(mushroom_x, mushroom_y))

    if player_rect.colliderect(mushroom_rect):
        draw_text(win, text, color, x, y)

    keys = pg.key.get_pressed()
    if keys[pg.K_a]:
        win.blit(walk_left[anim_count], (player_x, player_y))
    else:
        win.blit(walk_right[anim_count], (player_x, player_y))

    if keys[pg.K_a] and player_x > 1:
        player_x -= player_speed
    elif keys[pg.K_d] and player_x < 1280:
        player_x += player_speed

    if not jump:
        if keys[pg.K_SPACE]:
            jump = True
            jump_sound.play()
    else:
        if jump_count >= -5:
            if jump_count > 0:
                player_y -= (jump_count ** 2) / 2
            else:
                player_y += (jump_count ** 2) / 2
            jump_count -= 1
        else:
            jump = False
            jump_count = 5

    if anim_count == 6:
        anim_count = 0
    elif keys[pg.K_a] or keys[pg.K_d]:
        anim_count += 1

    for event in pg.event.get():
        if event.type == pg.QUIT:
            running = False
        elif event.type == pg.KEYDOWN:
            if event.key == pg.K_ESCAPE:
                pause_sound.play()
                game_paused = not game_paused

    if game_paused:
        draw_paused_win()
    else:
        pass

    mushroom_x -= 2

    pg.display.update()

    for event in pg.event.get():
        if event.type == pg.QUIT:
            running = False

    if not game_over and player_rect.colliderect(mushroom_rect):
        loss_sound.play()
        draw_text(win, text, color, x, y)
        game_over = True

    clock.tick(10)
