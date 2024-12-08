# Simulador de Bungee Jump 
### Descrição do Projeto
Esse projeto é um jogo criado para representar fenômenos específicos da física, com o objetivo de demonstrar de forma interativa e divertida os conceitos de queda livre em um Bungee Jump (Bungee jumping é um esporte radical praticado por muitos aventureiros corajosos, que consiste em saltar de uma altura num vazio, amarrado aos tornozelos ou cintura a uma corda elástica),os conteiros trabalhados são gravidade e viscosidade, e elasticidade. O jogo permite que o jogador selecione um planeta, cada um com níveis de gravidades diferentes, e observar como o personagem cai de uma queda livre amarrado em uma corda, com isso ele pode observar como a força elástica interage com o corpo. Também é possível personalizar a simulação escolhendo a massa do objeto, a força gravitacional do planeta personalizado e a Viscocidade do movimento que neste caso a viscosidade simula a resistência do ar no sistema
#### Escolha dos planetas e personalização dos planetas
[FOTO DA ESCOLHA DE PLANETAS E PERSONALIZAÇÃO]
#### Animação do personagem no Bungee Jump no planeta selecionado:
[ANIMAÇÃO DO PERSONAGEM CAINDO EM QUALQUER PLANETA]
### Conceitos de Física e Modelo
Este projeto utiliza um modelo físico simplificado para simular o movimento de um personagem preso a uma corda de bungee jump. A seguir, descrevemos os principais conceitos físicos e como eles foram implementados no código.
### Conceitos Físicos
#### Força Gravitacional
A simulação começa com a Força Gravitacional agindo no começo ela funciona como uma queda livre,A queda livre como o movimento quando algum objeto é solto ou abandonado do repouso (velocidade inicial igual a zero) a partir de uma certa altura em relação ao solo, em uma região onde haja aceleração gravitacional.

    m: massa do personagem.
    g: aceleração da gravidade do planeta selecionado.
#### Força Viscosa
Logo após a Força Viscosa age na simulação, ela  funciona para simular a resistencia do ar no sistema. Essa força é proporcional à velocidade do personagem: 
Fviscosa = −b⋅v 

    b: coeficiente de viscosidade do ambiente.
    v: velocidade do personagem.
#### Força Elástica
A corda de bungee jump exerce uma força elástica que tende a restaurar o personagem para a posição inicial,
Essa força é descrita pela Lei de Hooke: Felástica = −k⋅x

    k: constante de elasticidade da corda.
    x: deformação da corda, definida como a diferença entre a posição atual e o comprimento natural da corda.


#### Personagem atinge sua altura mínima
Quando a força elástica atinge seu máximo o objeto começa o movimento contra a aceleração de gravidade e a mesma começa a agir como força dissipativa junto com a força viscosa.
Até o momento em que as forças dissipativas zeram o movimento e o movimento volta para o estado inicial da simulação de Queda livre.

#### Força Resultante e Aceleração

A soma das forças determina a aceleração do personagem, conforme a Segunda Lei de Newton:

Ftotal = Fgravidade + Felástica + Fviscosa

Ftotal​ = Fgravidade​ + Felástica​ + Fviscosa​

a = Ftotal / massa

Atualização de Velocidade e Posição

### Informações sobre os autores: 
### Alunos
|        Nome                         |    NUSP   |       
|:-----------------------------------:|:---------:|  
|   Artur Domitti Camargo             |  15441661 |   
|   Lucas Mello Ciosaki       	      |  14591305 |   
|   Lucas Alves da Silva		         |  11819553  |
|   Renato Calacina Spessotto           |  14605824 |   

Essa simulação foi feita como parte do processo avaliativo da disciplina 7600105 - Física Básica I (2024) da USP-São Carlos ministrada pela Prof. Krissia de Zawadzki e pelo Prof. Esmerindo de Sousa Bernardes.



código da main 
```
import pygame as pg
from menu import draw_menu, handle_menu_events, buttons
from game import initialize_scene, run_simulation, handle_custom_config

pg.init()

# Configuração da tela
screen = pg.display.set_mode((1280, 720))
clock = pg.time.Clock()

# Controle do estado
running = True
menu_active = True
current_scene = None

while running:
    if menu_active:
        draw_menu(screen, buttons)
        for event in pg.event.get():
            if event.type == pg.QUIT:
                running = False
            elif event.type == pg.MOUSEBUTTONDOWN:
                result = handle_menu_events(event.pos, buttons)
                if result == "custom":
                    custom_config, is_custom = handle_custom_config(screen, clock)
                    if custom_config:
                        current_scene = initialize_scene(screen, custom_config, is_custom=is_custom)
                        menu_active = False
                elif result:
                    current_scene = initialize_scene(screen, result)
                    menu_active = False
    else:
        running, menu_active = run_simulation(screen, clock, current_scene)



pg.quit() 
```

Codigo do menu
```
import pygame as pg

pg.font.init()
my_font = pg.font.Font('assets/8bitsFont.ttf', 28)

# Planetas disponíveis com viscosidade
planets = {
    "Terra": {"gravity": 9.81, "background": "assets/earth.png", "character": "assets/human.png", "viscosity": 10},
    "Lua": {"gravity": 1.62, "background": "assets/moon.png", "character": "assets/astronaut.png", "viscosity": 1},
    "Júpiter": {"gravity": 24.79, "background": "assets/jupiter.png", "character": "assets/astronaut.png", "viscosity": 50},
}

# Classe de botões
class Button:
    def __init__(self, x, y, width, height, text, color, action=None):
        self.rect = pg.Rect(x, y, width, height)
        self.text = text
        self.color = color
        self.action = action

    def draw(self, screen):
        pg.draw.rect(screen, self.color, self.rect)
        text_surf = my_font.render(self.text, True, (255, 255, 255))
        text_rect = text_surf.get_rect(center=self.rect.center)
        screen.blit(text_surf, text_rect)

    def is_clicked(self, pos):
        return self.rect.collidepoint(pos)


# Configuração dos botões
buttons = []
start_y = 200
button_height = 50
button_width = 300
button_spacing = 20
swidth = 1280

for idx, (name, planet) in enumerate(planets.items()):
    button = Button(
        x=swidth // 2 - button_width // 2,
        y=start_y + idx * (button_height + button_spacing),
        width=button_width,
        height=button_height,
        text=name,
        color=(0, 0, 255),
        action=planet,
    )
    buttons.append(button)


# Função para desenhar o menu
def draw_menu(screen, buttons):
    screen.fill((0, 0, 0))
    title_text = my_font.render("Escolha um planeta:", True, (255, 255, 255))
    screen.blit(title_text, (screen.get_width() // 2 - title_text.get_width() // 2, 50))
    for button in buttons:
        button.draw(screen)
    pg.display.flip()

# Adicionar o botão de configurações personalizadas
custom_button = Button(
    x=swidth // 2 - button_width // 2,
    y=start_y + len(planets) * (button_height + button_spacing),
    width=button_width,
    height=button_height,
    text="Configuração Personalizada",
    color=(255, 0, 0),
    action="custom"
)
buttons.append(custom_button)


# Função para tratar eventos do menu
def handle_menu_events(mouse_pos, buttons):
    for button in buttons:
        if button.is_clicked(mouse_pos):
            if button.action == "custom":
                return "custom"  # Identificador para configuração personalizada
            return button.action  # Retorna os dados do planeta selecionado
    return None
```

Codigo do game
```
import pygame as pg
from math import *
import numpy as np

# Inicializa Pygame e fontes
pg.init()
pg.font.init()
my_font = pg.font.Font('assets/8bitsFont.ttf', 28)

# Classe para representar a cena (planeta, gravidade, fundo e personagem)
class Scene:
    def __init__(self, background, character, gravity, viscosity):
        self.background = pg.transform.scale(pg.image.load(background), (1280, 720))
        self.character = character
        self.gravity = gravity
        self.viscosity = viscosity

# Classe para representar o personagem
class Character:
    def __init__(self, sprite, massa, x, y, width, height):
        self.sprite = pg.image.load(sprite)
        self.massa = massa
        self.x = x
        self.y = y
        self.height = height
        self.width = width

    def scale(self):
        self.sprite = pg.transform.scale(self.sprite, (self.width, self.height))

    def getPos(self):
        posX = self.x - self.width / 2
        posY = self.y - self.height / 2
        return posX, posY

    def render(self, screen):
        self.scale()
        screen.blit(self.sprite, self.getPos())

# Classe para desenhar a corda
class Corda:
    def __init__(self, x, y, tamanho, k):
        self.initialX = x
        self.initialY = y
        self.x = x
        self.y = y
        self.tamanho = tamanho
        self.k = k

    def render(self, screen):
        def draw_line_round_corners_polygon(surf, p1, p2, c, w):
            p1v = pg.math.Vector2(p1)
            p2v = pg.math.Vector2(p2)
            lv = (p2v - p1v).normalize()
            lnv = pg.math.Vector2(-lv.y, lv.x) * w // 2
            pts = [p1v + lnv, p2v + lnv, p2v - lnv, p1v - lnv]
            pg.draw.polygon(surf, c, pts)
            pg.draw.circle(surf, c, p1, round(w / 2))
            pg.draw.circle(surf, c, p2, round(w / 2))

        draw_line_round_corners_polygon(screen, (self.initialX, self.initialY), (self.x, self.y), "aquamarine4", 10)

# Função para renderizar a interface de usuário
def renderUI(screen, g, m, b, deformacao):
    swidth, sheight = screen.get_size()
    x = 5 / 8 * swidth
    y = 1 / 4 * sheight

    # Renderiza textos
    text_gravidade = my_font.render(f"Gravidade: {g:.2f}", False, (255, 255, 255))
    text_massa = my_font.render(f"Massa: {m:.2f}", False, (255, 255, 255))
    text_viscosidade = my_font.render(f"Viscosidade: {b:.2f}", False, (255, 255, 255))  # Exibindo viscosidade
    text_deformacao = my_font.render(f"Deformação: {deformacao:.2f}", False, (255, 255, 255))

    # Fundo da UI
    uiBackground = pg.Surface((330, 155), pg.SRCALPHA)
    uiBackground.fill((0, 0, 0, 100))

    # Renderiza textos e fundo na tela
    screen.blit(uiBackground, (x - 20, y - 15))
    screen.blit(text_massa, (x, y))
    screen.blit(text_gravidade, (x, y + 30))
    screen.blit(text_viscosidade, (x, y + 60))  
    screen.blit(text_deformacao, (x, y + 90))

# Função para inicializar a cena baseada nos dados do planeta
def initialize_scene(screen, planet_data, is_custom=False):

    massa = planet_data["massa"] if is_custom else 80  # Usa a massa personalizada apenas no cenário customizado
    character = Character(
        planet_data["character"],
        massa=massa,
        x=screen.get_width() / 2,
        y=screen.get_height() / 4,
        width=96,
        height=96
    )
    return Scene(
        background=planet_data["background"],
        character=character,
        gravity=planet_data["gravity"],
        viscosity=planet_data["viscosity"]  # Passando viscosidade
    )

# Função principal para rodar a simulação
def run_simulation(screen, clock, current_scene):
    running = True
    menu_active = False

    # Reseta variáveis ao começar
    vel = 0
    fator = 15
    b = current_scene.viscosity
    
    posIni = current_scene.character.y
    corda = Corda(
        x=current_scene.character.x, 
        y=current_scene.character.y, 
        tamanho=30, 
        k=100
    )
    dt = 0.1


    
    while running:
        print(dt)
        for event in pg.event.get():
            if event.type == pg.QUIT:
                return False, menu_active

        # Referências rápidas
        character = current_scene.character
        background = current_scene.background
        pos = character.y
        m = character.massa
        g = current_scene.gravity

        # Física da simulação
        if pos - posIni < corda.tamanho:
            deformacao = 0
        else:
            deformacao = (pos - posIni) - corda.tamanho

        if deformacao > 0:
            F_elastica = -corda.k * deformacao
        else:
            F_elastica = 0
        F_gravidade = m * g * fator 
        deformacao = max(0, (pos - posIni) - corda.tamanho)
        F_elastica = -corda.k * deformacao
        F_viscosa = -b * vel
        F_total = F_gravidade + F_elastica + F_viscosa
        a = F_total / m

        vel += a * dt
        pos += vel * dt

        # Atualiza posições
        character.y = pos
        corda.x = character.x
        corda.y = character.y

        # Renderiza elementos na tela
        screen.blit(background, (0, 0))
        corda.render(screen)
        character.render(screen)
        renderUI(screen, g, m, b, deformacao)

        # Atualiza a tela
        pg.display.flip()
        dt = clock.tick(60) / 1000
        if(dt > 1):
            dt = 0.1
        

    return running, menu_active

def handle_custom_config(screen, clock):
    input_boxes = [
        {"label": "Gravidade:", "value": "", "rect": pg.Rect(500, 200, 280, 50)},
        {"label": "Viscosidade:", "value": "", "rect": pg.Rect(500, 300, 280, 50)},
        {"label": "Massa:", "value": "", "rect": pg.Rect(500, 400, 280, 50)}
    ]
    active_box = None
    font = pg.font.Font(None, 36)
    running = True

    # Configuração padrão da Terra
    default_data = {
        "gravity": 9.8,  # Gravidade da Terra
        "viscosity": 0.5,  # Exemplo de viscosidade na Terra
        "massa": 80,  # Massa padrão
        "background": "assets/terra_background.jpg",  # Fundo da Terra
        "character": "assets/terra_character.png"  # Personagem da Terra
    }
    custom_data = default_data

    

    while running:
        for event in pg.event.get():
            if event.type == pg.QUIT:
                return None
            elif event.type == pg.MOUSEBUTTONDOWN:
                active_box = None
                for box in input_boxes:
                    if box["rect"].collidepoint(event.pos):
                        active_box = box
            elif event.type == pg.KEYDOWN and active_box:
                if event.key == pg.K_RETURN:
                    if all(box["value"] for box in input_boxes):
                        try:
                            custom_data = {
                                "gravity": float(input_boxes[0]["value"]),
                                "viscosity": float(input_boxes[1]["value"]),
                                "massa": float(input_boxes[2]["value"]),
                                "background": "assets/earth.png",  # Fundo fixo da Terra
                                "character": "assets/human.png"  # Personagem fixo da Terra
                            }
                            running = False
                        except ValueError:
                            pass  # Ignore entradas inválidas
                elif event.key == pg.K_BACKSPACE:
                    active_box["value"] = active_box["value"][:-1]
                else:
                    active_box["value"] += event.unicode

        # Renderizar a tela
        screen.fill((0, 0, 0))
        for box in input_boxes:
            pg.draw.rect(screen, (255, 255, 255), box["rect"], 2)
            label = font.render(box["label"], True, (255, 255, 255))
            value = font.render(box["value"], True, (255, 255, 255))
            screen.blit(label, (box["rect"].x - 150, box["rect"].y + 10))
            screen.blit(value, (box["rect"].x + 10, box["rect"].y + 10))

        pg.display.flip()
        clock.tick(30)

    return custom_data, True

```


