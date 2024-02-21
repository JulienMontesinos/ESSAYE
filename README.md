# ESSAYE
```
import pygame
from pygame import *

pygame.init()

red = (255,0,0)
green = (0,255,0)
blue = (0,0,255)
white = (128,128,128)
black = (0,0,0)
purple = (127,0,255)
orange = (255,165,0)
white_re = (255, 255, 255)

screen = pygame.display.set_mode([800,500])
pygame.display.set_caption('Petit Jeu sur internet')
#background = black
framerate = 60
font = pygame.font.Font('freesansbold.ttf', 16)
timer = pygame.time.Clock()
change_color = True
score = 1000000

costs = {'green': 1, 'red': 2, 'orange': 3, 'white': 4, 'purple': 5}
owned = {'green': False, 'red': False, 'orange': False, 'white': False, 'purple': False}
manager_costs = {'green': 100, 'red': 500, 'orange': 1800, 'white': 4000, 'purple': 10000}
tasks = {
    'green': {'color': green, 'value': 1, 'draw': False, 'length': 0, 'speed': 5, 'color_index': 0, 'colors': [(144, 238, 144), (34, 139, 34), (60, 179, 113)]},
    'red': {'color': red, 'value': 2, 'draw': False, 'length': 0, 'speed': 4, 'color_index': 0, 'colors': [(255, 69, 0), (128, 0, 0), (255, 102, 102)]},
    'orange': {'color': orange, 'value': 3, 'draw': False, 'length': 0, 'speed': 3, 'color_index': 0, 'colors': [(255, 239, 213), (255, 140, 0), (255, 220, 180)]},
    'white': {'color': white, 'value': 4, 'draw': False, 'length': 0, 'speed': 2, 'color_index': 0, 'colors': [(169, 169, 169), (238, 216, 174), (139, 131, 120)]},
    'purple': {'color': purple, 'value': 5, 'draw': False, 'length': 0, 'speed': 1, 'color_index': 0, 'colors': [(138, 43, 226), (153, 50, 204), (216, 191, 216)]}
}
original_speeds = {'green': 5, 'red': 4, 'orange': 3, 'white': 2, 'purple': 1}
#les images
fond = image.load('fond-ecran.jpg')
fond = fond.convert()

def manager_progress():
    for name, task_info in tasks.items():
        if owned[name]:  
            task_info['draw'] = True

def draw_task(name, y_coord):
    task_info = tasks[name]
    global score
    global change_color
    if task_info['draw'] and task_info['length'] < 200:
        task_info['length'] += task_info['speed']
    elif task_info['length'] >= 200:
        task_info['draw'] = False
        task_info['length'] = 0
        score += task_info['value']
        change_color = True
        if owned[name]:
            task_info['color'] = task_info['colors'][0]
            task_info['speed'] = original_speeds[name]

    task = pygame.draw.circle(screen, task_info['color'], (30, y_coord), 20, 5)
    pygame.draw.rect(screen, task_info['color'], [70, y_coord - 15, 200, 30])
    pygame.draw.rect(screen, white_re, [75, y_coord - 10, 190, 20])
    pygame.draw.rect(screen, task_info['color'], [70, y_coord - 15, task_info['length'], 30])
    value_text = font.render(str(round(task_info['value'],2)), True, black)
    screen.blit(value_text, (20, y_coord -10))
    return task

def draw_buttons(name, x_coord):
    if name == 'green':
       color_button = pygame.draw.rect(screen, green, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       if not owned[name]:
           manager_button = pygame.draw.rect(screen, green, [x_coord, 440, 55, 30])
           manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
           screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    if name == 'red':
       color_button = pygame.draw.rect(screen, red, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       if not owned[name]:
           manager_button = pygame.draw.rect(screen, red, [x_coord, 440, 55, 30])
           manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
           screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    if name == 'orange':
       color_button = pygame.draw.rect(screen, orange, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       if not owned[name]:
           manager_button = pygame.draw.rect(screen, orange, [x_coord, 440, 55, 30])
           manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
           screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    if name == 'white':
       color_button = pygame.draw.rect(screen, white, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       if not owned[name]:
           manager_button = pygame.draw.rect(screen, white, [x_coord, 440, 55, 30])
           manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
           screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    if name == 'purple':
       color_button = pygame.draw.rect(screen, purple, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       if not owned[name]:
           manager_button = pygame.draw.rect(screen, purple, [x_coord, 440, 55, 30])
           manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
           screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    return color_button, manager_button

def click(event_pos):
    global score
    global change_color
    for name, task_info in tasks.items():
        if task_info['task_rect'].collidepoint(event_pos):
            if owned[name]:
                while change_color:
                    task_info['color_index'] = (task_info['color_index'] + 1) % len(task_info['colors'])
                    task_info['color'] = task_info['colors'][task_info['color_index']]
                    change_color = False
                    
                if task_info['color'] =='green':
                   task_info['speed'] += 5
                elif task_info['color'] =='red':
                    task_info['speed'] += 4
                elif task_info['color'] =='orange':
                    task_info['speed'] += 3
                elif task_info['color'] =='white':
                    task_info['speed'] += 2
                elif task_info['color'] =='purple':
                    task_info['speed'] += 1
            task_info['draw'] = True
            return True  
    for name, button_info in button_rects.items():
        if button_info['manager_button'].collidepoint(event_pos) and score >= manager_costs[name] and not owned[name]:
            owned[name] = True
            score -= manager_costs[name]
            return True  
        if button_info['color_button'].collidepoint(event_pos) and score >= costs[name]:
            tasks[name]['value'] += 0.15
            costs[name] += 0.1
            score -= costs[name]
            return True  
    return False  

running = True
button_rects = {}
while running:
    manager_progress()
    timer.tick(framerate)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            click(event.pos)

    #screen.fill(background)
    screen.blit(fond, (0,0))

    y_coords = [50, 110, 170, 230, 290]
    for i, name in enumerate(tasks.keys()):
        task_rect = draw_task(name, y_coords[i])
        tasks[name]['task_rect'] = task_rect  

    x_coords = [10, 70, 130, 190, 250]
    for i, name in enumerate(tasks.keys()):
        color_button, manager_button = draw_buttons(name, x_coords[i])
        button_rects[name] = {'color_button': color_button, 'manager_button': manager_button}

    #display_score = font.render('Money: $'+str(round(score, 2)), True, white, black)
    display_score = font.render('Money: $'+str(round(score, 2)), True, black)
    screen.blit(display_score, (10,5))
    vitesse = font.render('Vitesste', True, black)
    screen.blit(vitesse, (10,335))
    manager = font.render('Manager', True, black)
    screen.blit(manager, (10,415))
    pygame.display.flip()

pygame.quit()
```


