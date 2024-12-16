import pygame as pg
import random as rd

clock = pg.time.Clock()  # Создаю объект часов для управления частотой кадров
pg.init()

# Устанавливаю размеры окна
display_width, display_height = 1300, 790
win = pg.display.set_mode((display_width, display_height), flags=pg.NOFRAME)  # Создаю окно без рамки
pg.display.set_caption('Mario')  # Устанавливаю заголовок окна
pg.display.set_icon(pg.image.load('images/Details/icon.png'))  # Устанавливаю иконку окна

# Загружаю изображения фона и опыта
bg_image = pg.image.load('images/details/fon.png').convert_alpha()
xp_image = pg.image.load('images/details/xp.png').convert_alpha()

# Список изображений грибов
mushroom_images = [
    pg.image.load('images/details/mushroom1.png').convert_alpha(),
    pg.image.load('images/details/mushroom2.png').convert_alpha(),
]

# Список изображений облаков
cloud_images = [
    pg.image.load('images/details/cloud1.png').convert_alpha(),
    pg.image.load('images/details/cloud2.png').convert_alpha(),
    pg.image.load('images/details/cloud3.png').convert_alpha(),
]

# Списки изображений Марио при движении вправо
mario_right = [
    pg.image.load('images/mario/mario1.png').convert_alpha(),
    pg.image.load('images/mario/mario2.png').convert_alpha(),
    pg.image.load('images/mario/mario3.png').convert_alpha(),
    pg.image.load('images/mario/mario4.png').convert_alpha(),
    pg.image.load('images/mario/mario5.png').convert_alpha(),
    pg.image.load('images/mario/mario6.png').convert_alpha(),
    pg.image.load('images/mario/mario7.png').convert_alpha(),
]

# и влево
mario_left = [
    pg.image.load('images/mario2/mario1.png').convert_alpha(),
    pg.image.load('images/mario2/mario2.png').convert_alpha(),
    pg.image.load('images/mario2/mario3.png').convert_alpha(),
    pg.image.load('images/mario2/mario4.png').convert_alpha(),
    pg.image.load('images/mario2/mario5.png').convert_alpha(),
    pg.image.load('images/mario2/mario6.png').convert_alpha(),
    pg.image.load('images/mario2/mario7.png').convert_alpha(),
]


font = pg.font.Font('images/Details/text.ttf', 70)  # Шрифт и размер текста
paused_text = font.render('The game is suspended', True, (0, 32, 255))  # Текст и цвет для паузы
time_font = pg.font.Font('images/details/text.ttf', 24)  # Шрифт для времени


# Функция для вывода текста на экран
def print_text(message, x, y, font_color=(255, 255, 255), font_type='images/details/text.ttf', font_size=50):
    font_type = pg.font.Font(font_type, font_size)  # Создание объекта шрифта
    text = font_type.render(message, True, font_color)  # Отрисовка текста
    win.blit(text, (x, y))  # Вывод текста на экран


# Альтернативная функция для вывода текста на экран
def draw_text(surface, text, color, x, y):
    textobj = font.render(text, 1, color)  # Отрисовка текста
    textrect = textobj.get_rect()  # Получение прямоугольника текста
    textrect.topleft = (x, y)  # Установка позиции текста
    surface.blit(textobj, textrect)  # Вывод текста на поверхность


# Функция для отображения экрана паузы
def draw_paused_win():
    win.blit(paused_text, paused_text.get_rect(center=(600, 350)))  # Вывод текста паузы на экран
    pg.display.flip()  # Обновление экрана


# Переменные для анимации и физики Марио
anim_count = 0
mario_speed = 10
mario_x, mario_y = 10, 192
mushroom_x, mushroom_y = 200, 201
mushroom_x2, mushroom_y2 = 700, 464
mushroom_x3, mushroom_y3 = 400, 728
jump_count = 6
border_left = 0
border_right = 495
border_top = 0
border_bottom = 50
mushroom_direction = 1
mushroom_speed = 2
mushroom_frame = 0
animation_speed = 30
star_time = 64
current_time = star_time
fall_border_x = 1210
floor_y = 455


# Параметры текста "Game Over"
text = 'game over'
color = (152, 0, 2)
text_x, text_y = 500, 310

# Загрузка звуков
bg_sound = pg.mixer.Sound('sounds/background.mp3')
bg_sound.play(loops=-1)
jump_sound = pg.mixer.Sound('sounds/jump.mp3')
loss_sound = pg.mixer.Sound('sounds/gameover.mp3')
pause_sound = pg.mixer.Sound('sounds/pause.mp3')
button_sound = pg.mixer.Sound('sounds/pause.mp3')


# Класс кнопки
class Button:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.inactive_color = (255, 0, 0)
        self.active_color = (128, 128, 128)

    def draw(self, x, y, message, action=None):
        mouse = pg.mouse.get_pos()
        click = pg.mouse.get_pressed()
        if x < mouse[0] < x + self.width and y < mouse[1] < y + self.height:
            pg.draw.rect(win, self.active_color, (x, y, self.width, self.height))
            if click[0] == 1:
                pg.mixer.Sound.play(button_sound)
                pg.time.delay(300)
                if action is not None:
                    action()
        else:
            pg.draw.rect(win, self.inactive_color, (x, y, self.width, self.height))
        print_text(message, 200, 320)


# Флаги состояния игры
game_paused = 0
game_over = 0
jump = 0
running = 1


while running:

    win.blit(bg_image, (0, 0))  # Отображаем фон на экране

    xp_positions = [(20, 20), (55, 20), (90, 20)]  # Координаты жизни Марио
    for pos in xp_positions:  # Для каждого значка опыта
        win.blit(xp_image, pos)  # Отображаем значок опыта на соответствующей позиции

    player_rect = mario_left[0].get_rect(topleft=(mario_x, mario_y))  # Определяю прямоугольник для Марио
    mushroom_rect = mushroom_images[0].get_rect(topleft=(mushroom_x, mushroom_y))  # Определяю прямоугольник для гриба

    if player_rect.colliderect(mushroom_rect):  # Если Марио столкнулся с грибом
        draw_text(win, text, color, text_x, text_y)  # Отображаем текст "Game Over"

    keys = pg.key.get_pressed()  # Получаем список нажатых клавиш

    if keys[pg.K_a]:  # Если нажата клавиша A
        win.blit(mario_left[anim_count], (mario_x, mario_y))  # Отображаем Марио идущим влево
    else:  # иначе
        win.blit(mario_right[anim_count], (mario_x, mario_y))  # Отображаем Марио идущим вправо

    if keys[pg.K_a] and mario_x > 1:  # Если нажата клавиша A и Марио не у левого края экрана
        mario_x -= mario_speed  # Перемещаем Марио влево
    elif keys[pg.K_d] and mario_x < 1280:  # Если нажата клавиша D и Марио не у правого края экрана
        mario_x += mario_speed  # Перемещаем Марио вправо

    if not jump:  # Если Марио не прыгает
        if keys[pg.K_SPACE]:  # Если нажата клавиша SPACE
            jump = True  # Начинается процесс прыжка
            jump_sound.play()  # Проигрывается звук прыжка
    else: # Если Марио уже прыгает
        if jump_count >= -6:  # Пока Марио не закончил прыжок
            if jump_count > 0:  # Во время подъема
                mario_y -= (jump_count ** 2) / 2  # Поднимаемся вверх
            else:  # Во время спуска
                mario_y += (jump_count ** 2) / 2  # Опускаемся вниз
            jump_count -= 1  # Увеличиваем или уменьшаем высоту прыжка
        else:  # Когда прыжок завершился
            jump = False  # Сбрасываем флаг прыжка
            jump_count = 6  # Восстанавливаем начальный счетчик прыжка

    if anim_count == 6:  # Если текущая анимационная фаза достигла конца
        anim_count = 0  # Сбрасываем счетчик анимации
    elif keys[pg.K_a] or keys[pg.K_d]:  # Если Марио двигается
        anim_count += 1  # Переходим к следующей фазе анимации

    button = Button(300, 100)  # Создаем кнопку с её размерами
    button.draw(100, 300, 'play')  # Отображаем кнопку с надписью "wow" с её координатами

    mushroom_x += mushroom_speed * mushroom_direction  # Перемещаем гриб в зависимости от направления

    if mushroom_x <= border_left or mushroom_x >= border_right - mushroom_images[0].get_width(): # Если гриб достиг края экрана
        mushroom_direction *= -1  # Меняем направление движения гриба

    mushroom_frame += 1  # Увеличиваем счетчик кадров анимации гриба
    if mushroom_frame >= animation_speed:  # Если достигли максимального количества кадров
        mushroom_frame = 0  # Сбрасываем счетчик

    current_mushroom_image = mushroom_images[mushroom_frame // (animation_speed // len(mushroom_images))]  # Выбираем текущий кадр анимации гриба
    win.blit(current_mushroom_image, (mushroom_x, mushroom_y))  # Отображаем текущий кадр анимации гриба
    win.blit(current_mushroom_image, (mushroom_x2, mushroom_y2))
    win.blit(current_mushroom_image, (mushroom_x3, mushroom_y3))

    dt = clock.tick(140) / 100  # Получаем прошедшее время между кадрами
    current_time -= dt  # Уменьшаем оставшееся время
    if current_time <= 0:  # Если время истекло
        current_time = 0  # Время становится равно нулю

    time_text = time_font.render(f'Time: {int(current_time)}', True, (0, 0, 0)) # Формируем текстовое представление оставшегося времени
    win.blit(time_text, (600, 20))  # Отображаем текст времени на экране

    if mario_x >= fall_border_x:  # Если Марио достиг точки начала падения
        if mario_y < floor_y:  # Если Марио еще не упал на землю
            mario_y += 10  # Продолжаем падать
        else:  # Если Марио уже на земле
            mario_y = floor_y  # Устанавливаем Марио на уровне земли

    for event in pg.event.get():  # Обрабатываем все события
        if event.type == pg.QUIT:  # Если пользователь закрыл окно
            running = False  # Завершаем главный цикл
        elif event.type == pg.KEYDOWN:  # Если была нажата клавиша
            if event.key == pg.K_ESCAPE:  # Если нажата клавиша ESC
                pause_sound.play()  # Проигрываем звук паузы
                game_paused = not game_paused  # Переключаем состояние паузы

    if game_paused:  # Если игра приостановлена и не окончена
        draw_paused_win()  # Отображаем экран паузы

    if not game_over and player_rect.colliderect(mushroom_rect):  # Если игра не окончена и Марио столкнулся с грибом
        loss_sound.play()  # Проигрываем звук проигрыша
        draw_text(win, text, color, text_x, text_y)  # Отображаем текст "Game Over"
        game_over = True  # Устанавливаем флаг окончания игры

    clock.tick(15)  # Ограничиваем частоту кадров до 15 FPS
    pg.display.update()  # Обновляю экран
