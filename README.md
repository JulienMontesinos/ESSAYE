# ESSAYE
```
import pygame
from pygame import *
import json
import tkinter as tk
from tkinter import filedialog

pygame.init()

#Définition des couleurs
red = (255,0,0)
green = (0,255,0)
blue = (0,0,255)
white = (128,128,128)
black = (0,0,0)
purple = (127,0,255)
orange = (255,165,0)
white_re = (255, 255, 255)
#Définition des variable input pour utilisateur
user_text = ''
user_numero = ''
#Définition de la couleur pour input rectangle
color_active = pygame.Color((135,206,250))
color = color_passive = white_re
active_commande = False
active_numero = False
#Définition de vraiable environnement 
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
        #Si la chaîne est conforme au format, supprimez le "(" au début et ")" à la fin de la chaîne
        #Utilisez ensuite la méthode split(",") pour séparer les chaînes par des virgules et obtenir une liste de trois chaînes
        return tuple(map(int, color_str.strip("()").split(",")))
        #La fonction map applique la fonction int à chaque élément de la liste, convertissant la chaîne en entier
        #Enfin, utilisez la fonction tuple pour convertir l'objet cartographique en tuple. Ce tuple contient trois entiers
    return None  

def manager_progress():
    for name, task_info in tasks.items():
        if owned[name]:  
            task_info['draw'] = True

def draw_task(name, y_coord):
    task_info = tasks[name]
    global score
    global change_color
    # Si la tâche doit actuellement être dessinée et que sa longueur est inférieure à 200, augmentez sa longueur
    if task_info['draw'] and task_info['length'] < 200:
        task_info['length'] += task_info['speed']
    # Si la longueur de la tâche atteint ou dépasse 200, réinitialisez la tâche et augmentez le score
    elif task_info['length'] >= 200:
        task_info['draw'] = False
        task_info['length'] = 0
        score += task_info['value']
        change_color = True
    # Si la tâche a été achetée, réinitialisez sa couleur et sa vitesse
        if owned[name]:
            task_info['color'] = task_info['colors'][0]
            task_info['speed'] = original_speeds[name]
    # Dessinez des cercles de tâches et des barres de progression
    task = pygame.draw.circle(screen, task_info['color'], (30, y_coord), 20, 5)
    pygame.draw.rect(screen, task_info['color'], [70, y_coord - 15, 200, 30])
    pygame.draw.rect(screen, white_re, [75, y_coord - 10, 190, 20])
    pygame.draw.rect(screen, task_info['color'], [70, y_coord - 15, task_info['length'], 30])
    # Afficher la valeur de la tâche à côté du cercle
    value_text = font.render(str(round(task_info['value'],2)), True, black)
    screen.blit(value_text, (20, y_coord -10))
    return task

def current_color_manager(global_color_indices, managers_colors_change):
    # Récupère les informations de couleur pour le gestionnaire actuel
    color_info = managers_colors_change.get(name, {})
    # Définit la couleur par défaut si aucune couleur n'est spécifiée
    default_color_str = color_info.get("color", "(0, 0, 0)")
    default_color = str_to_rgb(default_color_str)
    # Convertit toutes les couleurs spécifiées en tuples RGB
    colors = [str_to_rgb(color_str) for color_str in color_info.get("colors", [])]  
    # Récupère l'indice de la couleur actuelle et détermine la couleur à utiliser
    current_color_index = global_color_indices.get(name, 0)
    current_color = colors[current_color_index] if colors else default_color 
    # Met à jour l'indice de couleur pour le prochain appel de fonction
    global_color_indices[name] = (current_color_index + 1) % len(colors) if colors else 0
    # Retourne la couleur actuelle à utiliser
    return current_color

def draw_buttons(name, x_coord):
    global user_numero, user_text, global_color_indices, managers_colors_change
     
    if name == 'green':
       # Dessine le bouton pour une couleur spécifique (ici 'vert') et son coût
       color_button = pygame.draw.rect(screen, green, [x_coord, 360, 50, 30])
       color_cost = font.render(str(round(costs[name],2)), True, black)
       screen.blit(color_cost, (x_coord + 6, 370))
       # Si le manageur de ce couleur n'est pas possédée par le joueur
       if not owned[name]:
           if user_numero == '1':
            # Si le numéro saisit par l'utilisateur est '1', utilise une couleur décidé par l'utilisateur pour le bouton du manager
              manager_button = pygame.draw.rect(screen, user_text, [x_coord, 440, 55, 30])
              manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
              screen.blit(manager_text, (x_coord + 6, 450))
           else:
               # Si aucune couleur de manager n'est définie ou si l'utilisateur ne donne pas fiche de json, utilise la couleur par défaut
                if not managers_colors_change or name not in managers_colors_change:
                  manager_button = pygame.draw.rect(screen, green, [x_coord, 440, 55, 30])
                  manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                  screen.blit(manager_text, (x_coord + 6, 450))
                else:
                    #Utilise une couleur spécifique pour le manager basée sur le système de rotation des couleurs
                    user_color = current_color_manager(global_color_indices, managers_colors_change)
                    manager_button = pygame.draw.rect(screen, user_color, [x_coord, 440, 55, 30])
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
                if not managers_colors_change or name not in managers_colors_change:
                  manager_button = pygame.draw.rect(screen, green, [x_coord, 440, 55, 30])
                  manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                  screen.blit(manager_text, (x_coord + 6, 450))
                else:
                    user_color = current_color_manager(global_color_indices, managers_colors_change)
                    manager_button = pygame.draw.rect(screen, user_color, [x_coord, 440, 55, 30])
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
                if not managers_colors_change or name not in managers_colors_change:
                  manager_button = pygame.draw.rect(screen, green, [x_coord, 440, 55, 30])
                  manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                  screen.blit(manager_text, (x_coord + 6, 450))
                else:
                    user_color = current_color_manager(global_color_indices, managers_colors_change)
                    manager_button = pygame.draw.rect(screen, user_color, [x_coord, 440, 55, 30])
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
                if not managers_colors_change or name not in managers_colors_change:
                  manager_button = pygame.draw.rect(screen, green, [x_coord, 440, 55, 30])
                  manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                  screen.blit(manager_text, (x_coord + 6, 450))
                else:
                    user_color = current_color_manager(global_color_indices, managers_colors_change)
                    manager_button = pygame.draw.rect(screen, user_color, [x_coord, 440, 55, 30])
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
                if not managers_colors_change or name not in managers_colors_change:
                  manager_button = pygame.draw.rect(screen, green, [x_coord, 440, 55, 30])
                  manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                  screen.blit(manager_text, (x_coord + 6, 450))
                else:
                    user_color = current_color_manager(global_color_indices, managers_colors_change)
                    manager_button = pygame.draw.rect(screen, user_color, [x_coord, 440, 55, 30])
                    manager_text = font.render(str(round(manager_costs[name], 2)), True, black)
                    screen.blit(manager_text, (x_coord + 6, 450))
       else:
           manager_button = pygame.draw.rect(screen, black, [x_coord, 440, 55, 30])
    return color_button, manager_button

def draw_button(screen, position, text):
    #Creation d'un input texte rectangle
    pygame.draw.rect(screen, button_color, position)
    text_render = base_font.render(text, True, black)
    screen.blit(text_render, (position[0] + 10, position[1] + 10))

def is_button_clicked(position, mouse_pos):
    # Décomposer la position du bouton en coordonnées x, y et dimensions
    x, y, width, height = position
    return x < mouse_pos[0] < x + width and y < mouse_pos[1] < y + height

def open_file_dialog():
    # Initialiser Tkinter en créant une fenêtre de base
    root = tk.Tk()
    # Masquer la fenêtre principale de Tkinter
    root.withdraw()
    # Ouvrir une boîte de dialogue pour choisir un fichier, avec un filtre spécifique pour les fichiers JSON  
    file_path = filedialog.askopenfilename(filetypes=[("JSON files", "*.json")])
    # Retourner le chemin du fichier sélectionné
    return file_path

def read_json(file_path):
    try:
        # Tenter d'ouvrir le fichier situé au chemin spécifié en mode lecture
        with open(file_path, 'r') as manager_file:
            # Charger le contenu du fichier JSON dans la variable managers_colors_change
            managers_colors_change = json.load(manager_file)
        # Créer un dictionnaire global_color_indices avec des clés correspondant
        # aux noms trouvés dans managers_colors_change et des valeurs initialisées à 0
        global_color_indices = {name: 0 for name in managers_colors_change}
        # Retourner le contenu du fichier JSON et le dictionnaire des indices de couleur
        return managers_colors_change, global_color_indices
    except Exception as e:
        # Imprimer l'erreur si quelque chose ne va pas lors de la lecture du fichier JSON
        print(e)

    


input_rect = pygame.Rect(450, 390, 140, 28)
input_recta = pygame.Rect(450, 460, 60, 28)

def click(event_pos):
    global active_commande
    global active_numero
    global score
    global change_color
    # Parcourez toutes les tâches et vérifiez si l'événement de clic croise la position de la tâche
    for name, task_info in tasks.items():
        if task_info['task_rect'].collidepoint(event_pos):
            if owned[name]:
                # Si la tâche a été achetée
                while change_color:
                   # Mettre à jour l'index des couleurs et changer la couleur de la tâche
                    task_info['color_index'] = (task_info['color_index'] + 1) % len(task_info['colors'])
                    task_info['color'] = task_info['colors'][task_info['color_index']]
                    change_color = False
               # Augmentez la vitesse en fonction de la couleur de la tâche
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
            # Marquez la tâche comme quelque chose qui doit être dessiné
            task_info['draw'] = True
            return True  
    # Parcourez tous les boutons et vérifiez si l'événement de clic croise la position du bouton
    for name, button_info in button_rects.items():
        # Si le bouton de gestion est cliqué et que le joueur a suffisamment de points à acheter
        if button_info['manager_button'].collidepoint(event_pos) and score >= manager_costs[name] and not owned[name]:
            owned[name] = True
            score -= manager_costs[name]
            return True  
        # Si le bouton de couleur est cliqué et que le joueur a suffisamment de points
        if button_info['color_button'].collidepoint(event_pos) and score >= costs[name]:
            tasks[name]['value'] += 0.15
            costs[name] += 0.1
            score -= costs[name]
            return True 
    # Vérifiez si l'événement de clic se trouve dans la zone de saisie et mettez à jour l'état d'activation en conséquence
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
                if file_path:  
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


