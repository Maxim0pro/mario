import pygame as pg  # импорт библиотеки pygame
import random as rd  # импорт библиотеки random

clock = pg.time.Clock()  # обращение к классу

pg.init()  # инициализация игры
width = 1300  # ширина
height = 790  # высота
win = pg.display.set_mode((1300, 790), flags=pg.NOFRAME)  # дисплэй
pg.display.set_caption('Mario')  # название для игры
pg.display.set_icon(pg.image.load('images/Details/incon.png'))  # иконка игры
bg = pg.image.load('images/Details/fon.png').convert_alpha()  # задний фон

mushroom_images = [
    pg.image.load('images/details/mushroom1.png').convert_alpha(),
    pg.image.load('images/details/mushroom2.png').convert_alpha()
]  # добавление гриба

cloud_images = [
    pg.image.load('images/Cloud/cloud1.png').convert_alpha(),
    pg.image.load('images/Cloud/cloud2.png').convert_alpha(),
    pg.image.load('images/Cloud/cloud3.png').convert_alpha()
]  # добавление облаков

walk_right = [
    pg.image.load('images/mario/mario1.png').convert_alpha(),
    pg.image.load('images/mario/mario2.png').convert_alpha(),
    pg.image.load('images/mario/mario3.png').convert_alpha(),
    pg.image.load('images/mario/mario4.png').convert_alpha(),
    pg.image.load('images/mario/mario5.png').convert_alpha(),
    pg.image.load('images/mario/mario6.png').convert_alpha(),
    pg.image.load('images/mario/mario7.png').convert_alpha()
]  # Марио повернутый в право

walk_left = [
    pg.image.load('images/mario2/mario1.png').convert_alpha(),
    pg.image.load('images/mario2/mario2.png').convert_alpha(),
    pg.image.load('images/mario2/mario3.png').convert_alpha(),
    pg.image.load('images/mario2/mario4.png').convert_alpha(),
    pg.image.load('images/mario2/mario5.png').convert_alpha(),
    pg.image.load('images/mario2/mario6.png').convert_alpha(),
    pg.image.load('images/mario2/mario7.png').convert_alpha()
]  # Марио повернутый в лево


text = 'Game Over'  # текст
color = (152, 0, 2)  # цвет
x = 500
y = 310

font = pg.font.Font('images/Details/text.ttf', 70)  # шрифт и размер текста
game_paused = False
paused_text = font.render('The game is suspended', True, (0, 32, 255))  # надпись и размер текста


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
player_x = 10  # координаты по х для Марио
player_y = 192  # координаты по y для Марио
mushroom_x = 300  # координаты по х для гриба
mushroom_y = 203  # координаты по у для гриба

jump = False
jump_count = 5  # на 5 позиций вверх Марио поднимается и отпускается


bg_sound = pg.mixer.Sound('sounds/background.mp3')  # звук игры
bg_sound.play(loops=-1)  # звук играет бесконечно

jump_sound = pg.mixer.Sound('sounds/jump.mp3')  # звук прыжка

loss_sound = pg.mixer.Sound('sounds/gameover.mp3')  # звук проигрыша

pause_sound = pg.mixer.Sound('sounds/pause.mp3')

game_over = False
running = True

while running:  # цикл игры

    win.blit(bg, (0, 0))  # вывод заднего фона

    win.blit(mushroom_images[0], (mushroom_x, mushroom_y))  # вывод гриба на экран
    win.blit(mushroom_images[1], (840, 464))  # вывод гриба на экран
    win.blit(mushroom_images[1], (400, 728))  # вывод гриба на экран

    #win.blit(cloud_images[0], (400, 5))  # вывод облака на экран

    player_rect = walk_left[0].get_rect(topleft=(player_x, player_y))  # отрисовка квадрата для игрока
    mushroom_rect = mushroom_images[0].get_rect(topleft=(mushroom_x, mushroom_y))  # отрисовка квадрата для гриба

    if player_rect.colliderect(mushroom_rect):
        draw_text(win, text, color, x, y)

    keys = pg.key.get_pressed()  # проверяем нажал ли пользователь на кнопку
    if keys[pg.K_a]:  # если пользователь нажал на "a" то
        win.blit(walk_left[anim_count], (player_x, player_y))  # выводим игрока на экран
    else:  # иначе
        win.blit(walk_right[anim_count], (player_x, player_y))  # выводим игрока на экран

    if keys[pg.K_a] and player_x > 1:  # если пользователь нажал на клавишу 'a' то
        player_x -= player_speed  # Марио идет назад
    elif keys[pg.K_d] and player_x < 1280:  # если пользователь нажал на клавишу 'd' то
        player_x += player_speed  # игрок идет вперед

    if not jump:  # проверка на значение
        if keys[pg.K_SPACE]:  # если пользователь нажал на пробел
            jump = True  # то Марио прыгает
            jump_sound.play()
    else:  # иначе
        if jump_count >= -5:  # условие будет выполняться до тех пор пока значение не будет -5
            if jump_count > 0:  # если переменная будет больше 0
                player_y -= (jump_count ** 2) / 2  # то поднимаем  игрока
            else:  # иначе
                player_y += (jump_count ** 2) / 2  # отпускаем игрока
            jump_count -= 1  # переменную уменьшаем на 1
        else:  # иначе
            jump = False  # завершение прыжка
            jump_count = 5  # обнуляем до первоночального числа

    if anim_count == 6:  # если переменная == 6 значений
        anim_count = 0  # то оно обнавляется и обратно начанает с нуля
    elif keys[pg.K_a] or keys[pg.K_d]:  # иначе если пользователь нажал на 'a' или 'd' то
        anim_count += 1  # меняем элементы из списка

    for event in pg.event.get():
        if event.type == pg.QUIT:
            running = False
        elif event.type == pg.KEYDOWN:  #
            if event.key == pg.K_ESCAPE:
                pause_sound.play()  #
                game_paused = not game_paused  #

    if game_paused:
        draw_paused_win()
    else:
        pass

    mushroom_x -= 2  # скорость движения гриба

    pg.display.update()  # обновление дисплэя

    for event in pg.event.get():  # получаем список из всех возможных событий
        if event.type == pg.QUIT:  # если закрываем приложения то
            running = False  # установливаем для переменной значение False

    if not game_over and player_rect.colliderect(mushroom_rect):
        loss_sound.play()  # звук проигрыша
        draw_text(win, text, color, x, y)
        game_over = True  # флаг окончания игры

    clock.tick(10)  # сколько кадров в секунду будет меняться картинки игрока
