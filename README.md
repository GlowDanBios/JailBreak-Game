# JailBreak-Game
Заключенный (ГГ) вырывается на свободу, и ведет перестрелку с охранниками на 3 локациях (тюремный двор, кухня, качалка ). Для перехода на следущую локацию необходимо собрать достаточное кол-во ключей, выпадающих из мертвых охранников. На карте есть несколько точек спавна оружия, аптечек и брони. В начале локации у игрока нет оружия и стандартное кол-во жизней. 
from tkinter import Tk, Canvas, mainloop, NW
from PIL import Image, ImageTk
import random,time,winsound,threading


window_width = 720
window_height = 530

tk = Tk()
c = Canvas(tk, width = window_width, height = window_height, bg = 'black')
c.pack()

'''corner_map = [
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 0, 0, 1],
    [1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1],
    [1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 1],
    [1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1],
    [1, 0, 0, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
]'''

game_map = [
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1],
    [1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 1],
    [1, 0, 1, 1, 0, 0, 0, 0, 1, 1, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 1, 1, 0, 0, 0, 0, 1, 1, 0, 1],
    [1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 1],
    [1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
]

block_width  = window_width // 12
block_height = (window_height-50) // 12

bullets = []
people = []
images = {
    "brick": ImageTk.PhotoImage(Image.open("brick.png").resize((block_width, block_height))),
    "grass": ImageTk.PhotoImage(Image.open("grass.png").resize((block_width, block_height))),
    "player_up": ImageTk.PhotoImage((Image.open("player.png").convert('RGBA').resize((block_width+5, block_height+5)))),
    "player_down": ImageTk.PhotoImage(
        (Image.open("player.png").convert('RGBA').resize((block_width, block_height-10)).rotate(180))),
    "player_left": ImageTk.PhotoImage(
        (Image.open("player.png").convert('RGBA').resize((block_width-10, block_height-10)).rotate(90))),
    "player_right": ImageTk.PhotoImage(
        (Image.open("player.png").convert('RGBA').resize((block_width-10, block_height-10)).rotate(270)))
    }
ammo = 10
hp = 3

for i in range(12):
    for j in range(12):
        if game_map[i][j] == 0:
            c.create_image(i * block_width, j * block_height+50, image=images['grass'], anchor=NW)
        if game_map[i][j] == 1:
            c.create_image(i * block_width, j * block_height+50, image=images['brick'], anchor=NW)

def rotate(game_object, direction):
    keys = ['up', 'down', 'left', 'right']
    for key in keys:
        image = game_object[key]
        c.itemconfigure(image, state = 'hidden')
    image = game_object[direction]
    c.itemconfigure(image, state = 'normal')
    game_object['direction'] = direction
    pass


def move(game_object):
    obg = game_object
    keys = ['up', 'down', 'left', 'right'] # в этих ключах лежат картинки
    for key in keys:
         c.move(obg[key], dx, dy)
    pass

def delete(game_object):
    keys = ['up', 'down', 'left', 'right'] # в этих ключах лежат картинки
    for key in keys:
            image = game_object[key]
            c.delete(image)
    
    pass


def coords(game_object):
    x, y = c.coords(game_object[direction])
    return (int(x // block_width), int(y // block_height))
    pass


def get_person(x, y, direction,u):
  if u == 1:
    person = {
        "life": 3,
        "direction": direction,
        "up": c.create_image(x * block_width, y * block_height, image=images['player_up'], anchor=NW, state='normal'),
        "down": c.create_image(x * block_width, y * block_height, image=images['player_down'], anchor=NW, state='hidden'),
        "left": c.create_image(x * block_width, y * block_height, image=images['player_left'], anchor=NW, state='hidden'),
        "right": c.create_image(x * block_width, y * block_height, image=images['player_right'], anchor=NW,
                                state='hidden')
    }
  else:
    person ={
        "life": 1,
        "direction": direction,
        "up": c.create_image(x * block_width, y * block_height, image=images['enemy_up'], anchor=NW, state='normal'),
        "down": c.create_image(x * block_width, y * block_height, image=images['enemy_down'], anchor=NW, state='hidden'),
        "left": c.create_image(x * block_width, y * block_height, image=images['enemy_left'], anchor=NW, state='hidden'),
        "right": c.create_image(x * block_width, y * block_height, image=images['enemy_right'], anchor=NW,
                                state='hidden')
    }
  rotate(person, direction)
  return person

def get_bullet(x, y, direction, weapon):
    if direction == 'left':
        # выбираем точку на 10 пикселей левее границы клетки с танком
        # по высоте - середина блока
        rx = block_width * x - 10
        ry = block_height * y + block_height // 2
    elif direction == 'right':
        # выбираем точку на 10 пикселей правее правой границы клетки с танком
        rx = block_width * (x + 1) + 10
        ry = block_height * y + block_height // 2
    elif direction == 'up':
        # выбираем точку на 10 пикселей выше верхней границы
        rx = block_width * x + block_width // 2
        ry = block_height * y - 10
    else:
        # выбираем точку на 10 пикселей нижней границы
        rx = block_width * x + block_width // 2
        ry = block_height * (y + 1) + 10
        
    if weapon == 1:
      bullet = {
        "direction": 'up',
        "up": c.create_image(rx, ry, image=images['bullet1_up'], state='normal'),
        "down": c.create_image(rx, ry, image=images['bullet1_down'], state='hidden'),
        "left": c.create_image(rx, ry, image=images['bullet1_left'], state='hidden'),
        "right": c.create_image(rx, ry, image=images['bullet1_right'], state='hidden')
      }

    rotate(bullet, direction)
    return bullet


def is_available(i, j):
    if i < 1 or i >= 12 or j < 0 or j >= 12:
        return False
    if game_map[i][j] == 1:
        return False
    for person in people:
        if coords(person,person['direction']) == (i,j):
            return False
    return True

def get_action(person):
    actions = []
    equstr = ["go_up","go_left","go_down","go_up"]
    direct = person['direction']
    height = len(game_map)
    width = len(game_map[1])
    x, y = coords(tank, direct)
    for i in range(1,x):
        dx,dy = coords(player,'up')
        if (dx,dy) == (i,y):
            return 'fire_left'
    for j in range(width-1, x, -1):
        dx, dy = coords(my_tank, 'up')
        if (dx,dy) == (j,y):
            return 'fire_right'
    for u in range(y, -1, -1):
        dx, dy = coords(my_tank, 'up') 
        if (dx,dy) == (x,u):
            return 'fire_up'
    for k in range(y-1,height):
        dx,dy = coords(my_tank, 'up')
        if (dx, dy) == (x,k):
            return 'fire_down'
    if is_available(x+1,y):
        actions.append("go_right")
    if is_available(x,y-1):
        actions.append("go_up")
    if is_available(x-1,y):
        actions.append('go_left')
    if is_available(x,y+1):
        actions.append('go_down')
    if len(actions)!=0:
        return random.choice(actions)
    else:
         return random.choice(equstr)


x=0
y=0
while is_available(x,y) == False:
    x = random.randint(1,12)
    y = random.randint(1,12)
player = get_person(x,y,'up',1)

        
