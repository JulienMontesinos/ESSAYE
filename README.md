# ESSAYE
```
import pygame
from pygame import *
import json
import tkinter as tk
from tkinter import filedialog

pygame.init()

red = (255,0,0)
green = (0,255,0)
blue = (0,0,255)
white = (128,128,128)
black = (0,0,0)
purple = (127,0,255)
orange = (255,165,0)
white_re = (255, 255, 255)
user_text = ''
user_numero = ''
color_active = pygame.Color((135,206,250))
color = color_passive = white_re
active_commande = False
active_numero = False
base_font = pygame.font.Font(None, 28)
button_color = (139, 0, 0) 
button_position = (480, 280, 140, 30)
button_text = 'Upload JSON'
managers_colors_change ={}
global_color_indices = {name: 0 for name in managers_colors_change}

screen = pygame.display.set_mode([800,500])
pygame.display.set_caption('Petit Jeu sur internet')
#background = black
framerate = 60
font = pygame.font.Font('freesansbold.ttf', 16)
clock = pygame.time.Clock()
change_color = True
score = 100000

costs = {'green': 1, 'red': 2, 'orange': 3, 'white': 4, 'purple': 5}
#colors_managers_numeros = {'1': green,'2': red, '3': orange, '4': white, '5': purple }
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

def str_to_rgb(color_str):
    if color_str.startswith("(") and color_str.endswith(")"):
        return tuple(map(int, color_str.strip("()").split(",")))
    return None  

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
    global user_numero, user_text, global_color_indices, managers_colors_change
    color_info = managers_colors_change.get(name, {})
    default_color_str = color_info.get("color", "(0, 0, 0)")
    default_color = str_to_rgb(default_color_str)
    colors = [str_to_rgb(color_str) for color_str in color_info.get("colors", [])]  
    current_color_index = global_color_indices[name]
    current_color = colors[current_color_index] if colors else default_color  

    if name == 'green':
       color_button = pygame.draw.rect(screen, green, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       if not owned[name]:
           if user_numero == '1':
              manager_button = pygame.draw.rect(screen, user_text, [x_coord, 440, 55, 30])
              manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
              screen.blit(manager_text, (x_coord + 6, 450))
           else:
                #color_info['color_index'] = (color_info['color_indices'] + 1) % len(color_info['colors'])
                #color_info['color'] = color_info['colors'][color_info['color_indices']]
                #use_color = current_color
                #global_color_indices[name] = (current_color_index + 1) % len(colors) if colors else 0
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
           if user_numero == '2':
              manager_button = pygame.draw.rect(screen, user_text, [x_coord, 440, 55, 30])
              manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
              screen.blit(manager_text, (x_coord + 6, 450))
           else:
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
           if user_numero == '3':
              manager_button = pygame.draw.rect(screen, user_text, [x_coord, 440, 55, 30])
              manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
              screen.blit(manager_text, (x_coord + 6, 450))
           else:
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
           if user_numero == '4':
                manager_button = pygame.draw.rect(screen, user_text, [x_coord, 440, 55, 30])
                manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                screen.blit(manager_text, (x_coord + 6, 450))
           else:
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
           if user_numero == '5':
                manager_button = pygame.draw.rect(screen, user_text, [x_coord, 440, 55, 30])
                manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                screen.blit(manager_text, (x_coord + 6, 450))
           else: 
                manager_button = pygame.draw.rect(screen, purple, [x_coord, 440, 55, 30])
                manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    return color_button, manager_button

def draw_button(screen, position, text):
    pygame.draw.rect(screen, button_color, position)
    text_render = base_font.render(text, True, black)
    screen.blit(text_render, (position[0] + 10, position[1] + 10))

def is_button_clicked(position, mouse_pos):
    x, y, width, height = position
    return x < mouse_pos[0] < x + width and y < mouse_pos[1] < y + height

def open_file_dialog():
    root = tk.Tk()
    root.withdraw()  
    file_path = filedialog.askopenfilename(filetypes=[("JSON files", "*.json")])
    return file_path

def read_json(file_path):
    try:
        with open(file_path, 'r') as manager_file:
            managers_colors_change = json.load(manager_file)
        global_color_indices = {name: 0 for name in managers_colors_change}
        return managers_colors_change, global_color_indices
    except Exception as e:
        print(e)
    


input_rect = pygame.Rect(450, 390, 140, 28)
input_recta = pygame.Rect(450, 460, 60, 28)

def click(event_pos):
    global active_commande
    global active_numero
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
    if input_rect.collidepoint(event.pos): 
         active_commande = True
    else: 
         active_commande = False
    if input_recta.collidepoint(event.pos): 
         active_numero = True
    else: 
         active_numero = False
    return False  

running = True
button_rects = {}
while running:
    manager_progress()
    clock.tick(framerate)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            click(event.pos)
            mouse_pos = event.pos
            if is_button_clicked(button_position, mouse_pos):
                file_path = open_file_dialog()
                if file_path:  # 确保用户选择了文件
                    managers_colors_change, global_color_indices = read_json(file_path) 
                
        if event.type == pygame.KEYDOWN: 
            # Check for backspace 
    # Check for backspace
           if event.key == pygame.K_BACKSPACE:
               if active_commande:
                   user_text = user_text[:-1]
               elif active_numero:
                    user_numero = user_numero[:-1]
           else:
                if active_commande:
                   user_text += event.unicode
                elif active_numero:
                   user_numero += event.unicode
    
    # it will set background color of screen 
    screen.fill((255, 255, 255)) 
  
    if active_commande: 
        color = color_active 
    else: 
        color = color_passive 

    if active_numero: 
        color = color_active 
    else: 
        color = color_passive 
    #screen.fill(background)
    screen.blit(fond, (0,0))
    if active_commande: 
        color = color_active 
        pygame.draw.rect(screen, color, input_rect)
    else: 
        color = color_passive 
        pygame.draw.rect(screen, color, input_rect, 2)
    if active_numero: 
        color = color_active 
        pygame.draw.rect(screen, color, input_recta)
    else: 
        color = color_passive 
        pygame.draw.rect(screen, color, input_recta, 2)
    #pygame.draw.rect(screen, white_re, [460, 390, 200,80]).set_alpha(200)
    text_surface = base_font.render(user_text, True, black)
    text_surfacea = base_font.render(user_numero, True, black)
    

 
    # render at position stated in arguments 
    screen.blit(text_surface, (input_rect.x+5, input_rect.y+5)) 
    input_rect.w = max(220, text_surface.get_width()+10)
    input_rect.h = max(28, text_surface.get_height()+5)
    screen.blit(text_surfacea, (input_recta.x+5, input_recta.y+5))
    input_recta.w = max(220, text_surfacea.get_width()+10)
    input_recta.h = max(28, text_surfacea.get_height()+5) 
    draw_button(screen, button_position, button_text)

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
    commande_color = font.render('Avoir des changements?', True, black)
    screen.blit(commande_color,(460, 365))
    commande_numero = font.render('Indiquer la numéro', True, black)
    screen.blit(commande_numero,(460, 435))
    screen.blit(manager, (10,415))
    pygame.display.flip()

pygame.quit()
```


