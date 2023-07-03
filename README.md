import pygame
import random
import math
import pygame.font
from pygame import mixer
import sys
import time

pygame.init()

#Criando a janela
screen=pygame.display.set_mode((1000, 630))
fps=60
relogio=pygame.time.Clock()

#Título e sprite
pygame.display.set_caption('AMONG ALIENS')
skate_sprite=pygame.image.load('alienXYZ.png')
pygame.display.set_icon(skate_sprite)
backgound=pygame.image.load('spacebackground1.png')
sprite_menu=pygame.image.load('alienmenuspr.png')

#Jogador principal
pers_principal=pygame.image.load('spacecraftimage3.png')
persoX=0
persoY=215
persoX_mudança=0
persoY_mudança=0

#Nave alienígena
aliendog=[]
alien_dogX=[]
alien_dogY=[]
alien_mudançaX=[]
alien_mudançaY=[]
numerode_aliens=6
alienvivo=[True]*numerode_aliens
velocidade_alien = 2.5

for cont in range(numerode_aliens): #Cria cópias do mesmo inimigo
    aliendog.append(pygame.image.load('alienx.png'))
    aliendog.append(pygame.image.load('alienX4.png'))
    aliendog.append(pygame.image.load('alienX3.png'))
    alien_dogX.append(872)
    alien_dogY.append(random.randint(0, 400))
    alien_mudançaX.append(85)
    alien_mudançaY.append(0.5)

#Mísseis
misseis=[]
immissel=pygame.image.load('bullet2.png')
misselX=0
misselY=265
missel_mudançaX=9
missel_mudançaY=0
missel_condiçao='noponto' #Não é possível ver o míssel
ultimo_disparo=pygame.time.get_ticks()
quantidade_misseis = 16


#Pacote de reposição
pac_reposicao= pygame.image.load('packagebullet.png')
pac_reposicaoX=1500
pac_reposicaoY= (random.randint(0, 400))
pac_mudancaX= 85
pac_mudancaY=4


#Pontuação
pontos=0
fonte= pygame.font.Font('freesansbold.ttf', 32)
fonte2= pygame.font.Font('freesansbold.ttf', 24)
pavraX=10
pavraY=10

#Menu inicial
menu = True
start_pressed = 0
comeco_jogo = False
font = pygame.font.Font(None, 50)
font1 = pygame.font.Font(None, 40)
start_button = pygame.Rect(1000 // 2 - 150, 325, 300, 125)
quit_button = pygame.Rect(1000 // 2 - 75,start_button.y+start_button.height+50, 150, 50)
arredondamento=30
click_botao=pygame.mixer.Sound('lasersound.wav')

#Cronômetro
tempo_inicial = time.time()
tempo_atual=pygame.time.get_ticks()

#Função de Game Over e mensagem
gameover_tipo=pygame.font.Font('freesansbold.ttf', 128)

class Missel:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.velocidade = 5


    def mover(self):
        self.x += self.velocidade

class AlienCL:
    def __init__(self, x, y, velocidade):
        self.x = x
        self.y = y
        self.velocidade = velocidade

def gameover():
    game_fim =gameover_tipo.render('FIM DE JOGO', True, (255, 255, 255))
    screen.blit(game_fim, (80, 170))
    pontuacao_final = fonte.render('Pontuação final: ' + str(pontos), True, (255, 255, 255))
    posicao_pontuacao_final = pontuacao_final.get_rect(center=(screen.get_width() // 2, screen.get_height() // 2 + 100))
    screen.blit(pontuacao_final, posicao_pontuacao_final)
    texto_tempo_final = fonte.render(f"Tempo total: {tempo_final}s", True, (255, 255, 255))
    screen.blit(texto_tempo_final, (400, 560))
    pygame.display.update()
    pygame.time.wait(600000)

#Funções para desenhar personagens na tela
def jogador(x, y):
    for missel in misseis:
        screen.blit(immissel, (missel.x + 30, missel.y + 65))
    screen.blit(pers_principal, (x, y))


#Desenha os aliens na tela
def dogalien(x, y, cont):
    screen.blit(aliendog[cont], (x, y))

#Desenha os pacotes reposição na tela
def pacotesRepo(x, y,):
    global pac_reposicaoX
    screen.blit(pac_reposicao, (x, y))


def atirar(x, y):
    global ultimo_disparo
    global quantidade_misseis
    tempo_atual = pygame.time.get_ticks()
    if tempo_atual - ultimo_disparo >= 500:
        som_missel = mixer.Sound('lasersound.wav')
        som_missel.play()
        novo_missel = Missel(x, y)
        misseis.append(novo_missel)
        quantidade_misseis -= 1
        ultimo_disparo = tempo_atual
        novo_missel.mover()

#Função para desenhar o contador de mísseis na tela
def show_points():
    fonte = pygame.font.Font(None, 36)
    texto = fonte.render(f"Mísseis: {quantidade_misseis}", True, (255, 255, 255))
    screen.blit(texto, (430, 10))

#Função para implementar colisão baseado na distância entre dois pontos
def colisao(alien_dogX, alien_dogY, misselX, misselY):
    distancia=math.sqrt((math.pow(alien_dogX-misselX,2))+(math.pow(alien_dogY-misselY, 2)))
    if distancia<50:
        return True
    else:
        return False

#Função para desenhar a pontuação na tela
def sisPontuacao(x, y):
    pontuaçao=fonte.render('Pontos: ' + str(pontos), True, (255, 255, 255))
    screen.blit(pontuaçao, (x, y))

#Loop para execução do menu
while menu:
    screen.fill((0, 0, 0))

    screen.blit(sprite_menu, (375, 35))

    #Retângulos com bordas redondas
    pygame.draw.rect(screen, (0, 238, 0), start_button, border_radius=arredondamento)
    pygame.draw.rect(screen, (238, 0, 0), quit_button, border_radius=arredondamento)

    # Introduz o botão start
    start_text = font.render("COMEÇAR", True, (0, 0, 0))
    start_text_rect = start_text.get_rect(center=start_button.center)
    screen.blit(start_text, start_text_rect)

    # Indroduz o botão exit
    quit_text = font1.render("SAIR", True, (0, 0, 0))
    quit_text_rect = quit_text.get_rect(center=quit_button.center)
    screen.blit(quit_text, quit_text_rect)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        elif event.type == pygame.MOUSEBUTTONDOWN:
            # Verifica se o botão COMEÇAR foi pressionado
            if start_button.collidepoint(event.pos):
                click_botao=pygame.mixer.Sound('lasersound.wav') #Adiciona som ao clicar o botão
                click_botao.play()
                back_sound = mixer.music.load('backgroundsound.wav')
                mixer.music.play(-1)
                menu = False
            # Verifica se o botão SAIR foi pressionado
            elif quit_button.collidepoint(event.pos):
                click_botao = pygame.mixer.Sound('lasersound.wav') #Adiciona som ao clicar o botão
                click_botao.play()
                pygame.quit()
                sys.exit()

    pygame.display.update()

#Loop infinito que roda o jogo
running=True
while running:
    screen.fill((0, 0, 0))

     #Imagem de fundo
    screen.blit(backgound, (0,0))

    #Artifício para contagem de tempo
    tempo_decorrido = int(time.time() - tempo_inicial)
    if gameover:
        tempo_final = tempo_decorrido
    else:
        tempo_decorrido = int(time.time() - tempo_inicial)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        #Checando se o teclado está precionado para a esquerda ou para a direita
        #Movimentação do personagem, dos vilões e dos mísseis
        if event.type==pygame.KEYDOWN:
            if event.key==pygame.K_LEFT:
                    persoX_mudança=-2
            if event.key==pygame.K_RIGHT:
                    persoX_mudança = 2.5
            if event.key==pygame.K_DOWN:
                    persoY_mudança= 2
            if event.key==pygame.K_UP:
                    persoY_mudança = -2
            if event.key==pygame.K_SPACE:
                if quantidade_misseis > 0:
                    if missel_condiçao is 'noponto':
                        tempo_atual = pygame.time.get_ticks()
                        if tempo_atual - ultimo_disparo >= 500:
                            som_missel=mixer.Sound('lasersound.wav')
                            som_missel.play()
                            misselY=persoY
                            atirar(persoX, misselY)
                            ultimo_disparo = tempo_atual #Conta o momento do último disparo com relação ao tempo atual.

        if event.type==pygame.KEYUP:
            if event.key==pygame.K_LEFT or event.key==pygame.K_RIGHT:
                    persoX_mudança = 0
            elif event.key==pygame.K_DOWN or event.key==pygame.K_UP:
                    persoY_mudança = 0

    #variável para aumentar a velocidade do alien
    velocidade_alien+= 0.0005 * tempo_decorrido//450

    persoX+=persoX_mudança
    persoY+=persoY_mudança

    #Desenhando o tempo na tela
    fonte = pygame.font.SysFont(None, 36)
    texto_tempo = fonte.render(f"Tempo: {tempo_decorrido}s", True, (255, 255, 255))
    screen.blit(texto_tempo, (850, 10))

    #Delimitando o espaço de atuação da personagem
    if persoX<=-1:
        persoX=-1
    elif persoX>=872:
        persoX=872
    elif persoY<=-40:
        persoY=-40
    elif persoY>=430:
        persoY=430

    # Delimitando o espaço de atuação do inimigo
    for cont in range(numerode_aliens):
        alien_dogY[cont] += alien_mudançaY[cont]
        #GameOver
        if colisao(alien_dogX[cont], alien_dogY[cont], persoX, persoY):
            som_bateu = mixer.Sound('explosionsound.wav')
            som_bateu.play()
            gameover()
            som_bateu.stop()
            break

        if alien_dogX[cont]<-180:
            for cont2 in range(numerode_aliens):
                alien_dogX[cont2]=80
                gameover()
                break

        if alien_dogY[cont] <= -15:
            alien_mudançaY[cont] = 0.5 * velocidade_alien
            alien_dogX[cont] -= alien_mudançaX[cont]
        elif alien_dogY[cont] >= 400:
            alien_mudançaY[cont] = -0.5 * velocidade_alien
            alien_dogX[cont] -= alien_mudançaX[cont]

        # Colisão
        toque = colisao(alien_dogX[cont], alien_dogY[cont], misselX, misselY)
        if toque:
            som_bateu = mixer.Sound('explosionsound.wav')
            som_bateu.play()
            misselX = 0
            misselY = 265
            missel_condiçao = 'noponto'
            pontos += 1
            print(pontos)
            alien_dogX[cont] = 872
            alien_dogY[cont] = random.randint(0, 400)

        dogalien(alien_dogX[cont], alien_dogY[cont], cont)

    #Delimitando o espaço de atuação e a movimentação do pacote reposição
    pac_reposicaoY += pac_mudancaY
    if pac_reposicaoY <= 5:
        pac_mudancaY = 4.5
        pac_reposicaoX -= pac_mudancaX
    elif pac_reposicaoY >= 400:
        pac_mudancaY = -4.5
        pac_reposicaoX -= pac_mudancaX
    elif pac_reposicaoX<=-180:
        pac_reposicaoX=2000

    #Colisão entre o pacote e a nave para incrementar o nº de mísseis
    toque = colisao(pac_reposicaoX, pac_reposicaoY, persoX, persoY)
    if toque:
        quantidade_misseis += 4
        pac_reposicaoX = 2500
        pac_reposicaoY = random.randint(0, 400)

    #Movimentação do míssel
    for missel in misseis:
        missel.mover()

        #Adapta a colisão aos múltiplos mísseis disparados
        for cont in range(numerode_aliens):
            if colisao(alien_dogX[cont], alien_dogY[cont], missel.x + 30, missel.y + 65):
                som_bateu = mixer.Sound('explosionsound.wav')
                som_bateu.play()
                pontos += 1
                print(pontos)
                alien_dogX[cont] = 900
                alien_dogY[cont] = random.randint(0, 400)
                misseis.remove(missel)
                break

    if misselX>=850:
        misselX=0
        missel_condiçao = 'noponto'
    if missel_condiçao is 'atirar':
        atirar(misselX, misselY)
        misselX+=missel_mudançaX

    show_points()
    pacotesRepo(pac_reposicaoX, pac_reposicaoY)
    jogador(persoX, persoY)
    for cont in range(numerode_aliens):
        dogalien(alien_dogX[cont], alien_dogY[cont], cont)
    sisPontuacao(pavraX, pavraY)
    pygame.display.update()
