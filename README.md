# 2021_1_learningfair
import random, sys, time, math, pygame
from pygame.locals import *

pygame.init()


display_width = 800
display_height = 600
gameDisplay = pygame.display.set_mode((display_width, display_height))
clock = pygame.time.Clock()


FPS = 30 
WINWIDTH = 640 
WINHEIGHT = 480 
HALF_WINWIDTH = int(WINWIDTH / 2)
HALF_WINHEIGHT = int(WINHEIGHT / 2)

GRASSCOLOR = (98, 171, 101)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BULE=(1,0,255)
BLACK=(0,0,0)

CAMERASLACK = 90     
MOVERATE = 9         
BOUNCERATE = 6     
BOUNCEHEIGHT = 30  
STARTSIZE = 25     
WINSIZE = 300      
INVULNTIME = 2     
GAMEOVERTIME = 4  
MAXHEALTH = 5    


LEFT = 'left'
RIGHT = 'right'

NUMGRASS = 100       
NUMDEERS = 30  
DEERMINSPEED = 3 
DEERMAXSPEED = 7 
DIRCHANGEFREQ = 2    


def main():
    global FPSCLOCK, BASICFONT, L_DEER_IMG, R_DEER_IMG, GRASSIMAGES, SCORE

    pygame.init()
    FPSCLOCK = pygame.time.Clock()
    pygame.display.set_icon(pygame.image.load('gameicon.png'))
    gameDisplay = pygame.display.set_mode((WINWIDTH, WINHEIGHT))
    pygame.display.set_caption('수뭉이 키우기♡')
    BASICFONT = pygame.font.Font(None, 40)

   
    L_DEER_IMG = pygame.image.load('deer.png')
    R_DEER_IMG = pygame.transform.flip(L_DEER_IMG, True, False)
    
   
    GRASSIMAGES = []
    for i in range(1, 5):
        GRASSIMAGES.append(pygame.image.load('grass%s.png' % i))

   
    while True:
        runGame()
        
        

def runGame():
    
    invulnerableMode = False  
    invulnerableStartTime = 0 
    gameOverMode = False      
    gameOverStartTime = 0    
    winMode = False          

    score = 0;  

       
   
    winSurf = BASICFONT.render('You have become KING DEER!', True, BULE)
    winRect = winSurf.get_rect()
    winRect.center = (HALF_WINWIDTH, HALF_WINHEIGHT)
    
   
    winSurf2 = BASICFONT.render('(Press "r" to restart.)', True, BULE)
    winRect2 = winSurf2.get_rect()
    winRect2.center = (HALF_WINWIDTH, HALF_WINHEIGHT + 30)

   
    gameOverSurf = BASICFONT.render('Game Over', True, RED)
    gameOverRect = gameOverSurf.get_rect()
    gameOverRect.center = (HALF_WINWIDTH, HALF_WINHEIGHT) 

    
   
    camerax = 0
    cameray = 0

    grassObjs = []    
    deerObjs = [] 
    
  
    playerObj = {'surface': pygame.transform.scale(L_DEER_IMG, (STARTSIZE, STARTSIZE)),
                 'facing': LEFT,
                 'size': STARTSIZE,
                 'x': HALF_WINWIDTH,
                 'y': HALF_WINHEIGHT,
                 'bounce':0,
                 'health': MAXHEALTH}

    moveLeft  = False
    moveRight = False
    moveUp    = False
    moveDown  = False

 
    for i in range(10):
        grassObjs.append(makeNewGrass(camerax, cameray))
        grassObjs[i]['x'] = random.randint(0, WINWIDTH)
        grassObjs[i]['y'] = random.randint(0, WINHEIGHT)

    while True:

        
       
     
        if invulnerableMode and time.time() - invulnerableStartTime > INVULNTIME:
            invulnerableMode = False

       
        for dObj in deerObjs:
           
            dObj['x'] += dObj['movex']
            dObj['y'] += dObj['movey']
            dObj['bounce'] += 1
            if dObj['bounce'] > dObj['bouncerate']:
                dObj['bounce'] = 0 


        while len(grassObjs) < NUMGRASS:
            grassObjs.append(makeNewGrass(camerax, cameray))
        while len(deerObjs) < NUMDEERS:
            deerObjs.append(makeNewDeer(camerax, cameray))

        

        for i in range(len(grassObjs) - 1, -1, -1):
            if isOutsideActiveArea(camerax, cameray, grassObjs[i]):
                del grassObjs[i]
        for i in range(len(deerObjs) - 1, -1, -1):
            if isOutsideActiveArea(camerax, cameray, deerObjs[i]):
                del deerObjs[i]

  
        playerCenterx = playerObj['x'] + int(playerObj['size'] / 2)
        playerCentery = playerObj['y'] + int(playerObj['size'] / 2)
        if (camerax + HALF_WINWIDTH) - playerCenterx > CAMERASLACK:
            camerax = playerCenterx + CAMERASLACK - HALF_WINWIDTH
        elif playerCenterx - (camerax + HALF_WINWIDTH) > CAMERASLACK:
            camerax = playerCenterx - CAMERASLACK - HALF_WINWIDTH
        if (cameray + HALF_WINHEIGHT) - playerCentery > CAMERASLACK:
            cameray = playerCentery + CAMERASLACK - HALF_WINHEIGHT
        elif playerCentery - (cameray + HALF_WINHEIGHT) > CAMERASLACK:
            cameray = playerCentery - CAMERASLACK - HALF_WINHEIGHT

    
        gameDisplay.fill(GRASSCOLOR)

    
        for gObj in grassObjs:
            gRect = pygame.Rect( (gObj['x'] - camerax,
                                  gObj['y'] - cameray,
                                  gObj['width'],
                                  gObj['height']) )
            gameDisplay.blit(GRASSIMAGES[gObj['grassImage']], gRect)



       

        flashIsOn = round(time.time(), 1) * 10 % 2 == 1
        if not gameOverMode and not (invulnerableMode and flashIsOn):
            playerObj['rect'] = pygame.Rect( (playerObj['x'] - camerax,
                                              playerObj['y'] - cameray - getBounceAmount(playerObj['bounce'], BOUNCERATE, BOUNCEHEIGHT),
                                              playerObj['size'],
                                              playerObj['size']) )
            gameDisplay.blit(playerObj['surface'], playerObj['rect'])


        drawHealthMeter(playerObj['health'])

        drawScore(score)
       
   
        for dObj in deerObjs:
            dObj['rect'] = pygame.Rect( (dObj['x'] - camerax,
                                         dObj['y'] - cameray - getBounceAmount(dObj['bounce'], dObj['bouncerate'], dObj['bounceheight']),
                                         dObj['width'],
                                         dObj['height']) )
            gameDisplay.blit(dObj['surface'], dObj['rect'])    

      
            if random.randint(0, 99) < DIRCHANGEFREQ:
                dObj['movex'] = getRandomVelocity()
                dObj['movey'] = getRandomVelocity()
                if dObj['movex'] > 0: 
                    dObj['surface'] = pygame.transform.scale(R_DEER_IMG, (dObj['width'], dObj['height']))
                else: 
                    dObj['surface'] = pygame.transform.scale(L_DEER_IMG, (dObj['width'], dObj['height']))

        for event in pygame.event.get():
            if event.type == QUIT:
                terminate()

            elif event.type == KEYDOWN:
                if event.key in (K_UP, K_w):
                    moveDown = False
                    moveUp = True
                elif event.key in (K_DOWN, K_s):
                    moveUp = False
                    moveDown = True
                elif event.key in (K_LEFT, K_a):
                    moveRight = False
                    moveLeft = True
                    if playerObj['facing'] != LEFT: 
                        playerObj['surface'] = pygame.transform.scale(L_DEER_IMG, (playerObj['size'], playerObj['size']))
                    playerObj['facing'] = LEFT
                elif event.key in (K_RIGHT, K_d):
                    moveLeft = False
                    moveRight = True
                    if playerObj['facing'] != RIGHT:
                        playerObj['surface'] = pygame.transform.scale(R_DEER_IMG, (playerObj['size'], playerObj['size']))
                    playerObj['facing'] = RIGHT
                elif winMode and event.key == K_r:
                    return

                
            elif event.type == KEYUP:
               
                if event.key in (K_LEFT, K_a):
                    moveLeft = False
                elif event.key in (K_RIGHT, K_d):
                    moveRight = False
                elif event.key in (K_UP, K_w):
                    moveUp = False
                elif event.key in (K_DOWN, K_s):
                    moveDown = False

                elif event.key == K_ESCAPE:
                    terminate()

        if not gameOverMode:
        
            if moveLeft:
                playerObj['x'] -= MOVERATE
            if moveRight:
                playerObj['x'] += MOVERATE
            if moveUp:
                playerObj['y'] -= MOVERATE
            if moveDown:
                playerObj['y'] += MOVERATE

            if (moveLeft or moveRight or moveUp or moveDown) or playerObj['bounce'] != 0:
                playerObj['bounce'] += 1

            if playerObj['bounce'] > BOUNCERATE:
                playerObj['bounce'] = 0 

      
            for i in range(len(deerObjs)-1, -1, -1):
                dObj = deerObjs[i]
                if 'rect' in dObj and playerObj['rect'].colliderect(dObj['rect']):
                 

                    if dObj['width'] * dObj['height'] <= playerObj['size']**2:
                       
                        playerObj['size'] += int( (dObj['width'] * dObj['height'])**0.2 ) + 1
                        del deerObjs[i]
                       

                        if playerObj['facing'] == LEFT:
                            playerObj['surface'] = pygame.transform.scale(L_DEER_IMG, (playerObj['size'], playerObj['size']))
                        if playerObj['facing'] == RIGHT:
                            playerObj['surface'] = pygame.transform.scale(R_DEER_IMG, (playerObj['size'], playerObj['size']))

                        score += 10

                        if playerObj['size'] > WINSIZE:
                            winMode = True 

                    elif not invulnerableMode:
                   
                        invulnerableMode = True
                        invulnerableStartTime = time.time()
                        playerObj['health'] -= 1
                        if playerObj['health'] == 0:
                            gameOverMode = True 
                            gameOverStartTime = time.time()

        else:
           
            gameDisplay.blit(gameOverSurf, gameOverRect)
            if time.time() - gameOverStartTime > GAMEOVERTIME:
                return 


                            

   
        if winMode:
            gameDisplay.blit(winSurf, winRect)
            gameDisplay.blit(winSurf2, winRect2)                     
                

       

        pygame.display.update()
        FPSCLOCK.tick(FPS)




def terminate():
    pygame.quit()
    sys.exit()


def getBounceAmount(currentBounce, bounceRate, bounceHeight):
 
    return int(math.sin( (math.pi / float(bounceRate)) * currentBounce ) * bounceHeight)


def makeNewGrass(camerax, cameray):
   
    gr = {}
    gr['grassImage'] = random.randint(0, len(GRASSIMAGES) - 1)
    gr['width']  = GRASSIMAGES[0].get_width()
    gr['height'] = GRASSIMAGES[0].get_height()
    gr['x'], gr['y'] = getRandomOffCameraPos(camerax, cameray, gr['width'], gr['height'])
    gr['rect'] = pygame.Rect( (gr['x'], gr['y'], gr['width'], gr['height']) )
    return gr

def getRandomOffCameraPos(camerax, cameray, objWidth, objHeight):
  
    cameraRect = pygame.Rect(camerax, cameray, WINWIDTH, WINHEIGHT)
    while True:
        x = random.randint(camerax - WINWIDTH, camerax + (2 * WINWIDTH))
        y = random.randint(cameray - WINHEIGHT, cameray + (2 * WINHEIGHT))
        objRect = pygame.Rect(x, y, objWidth, objHeight)
        if not objRect.colliderect(cameraRect):
            return x, y

def makeNewDeer(camerax, cameray):
   
    deer = {}
    generalSize = random.randint(5, 25)
    multiplier = random.randint(1, 3)
    deer['width']  = (generalSize + random.randint(0, 10)) * multiplier
    deer['height'] = (generalSize + random.randint(0, 10)) * multiplier
    deer['x'], deer['y'] = getRandomOffCameraPos(camerax, cameray, deer['width'], deer['height'])
    deer['movex'] = getRandomVelocity()
    deer['movey'] = getRandomVelocity()
    if deer['movex'] < 0: 
        deer['surface'] = pygame.transform.scale(L_DEER_IMG, (deer['width'], deer['height']))
    else: 
        deer['surface'] = pygame.transform.scale(R_DEER_IMG, (deer['width'], deer['height']))
    deer['bounce'] = 0
    deer['bouncerate'] = random.randint(10, 18)
    deer['bounceheight'] = random.randint(10, 50)
    return deer

def getRandomVelocity():
    speed = random.randint(DEERMINSPEED, DEERMAXSPEED)
    if random.randint(0, 1) == 0:
        return speed
    else:
        return -speed

def drawHealthMeter(currentHealth):
    for i in range(currentHealth): 
        pygame.draw.rect(gameDisplay, RED,   (15, 5 + (10 * MAXHEALTH) - i * 10, 20, 10))
    for i in range(MAXHEALTH): 
        pygame.draw.rect(gameDisplay, WHITE, (15, 5 + (10 * MAXHEALTH) - i * 10, 20, 10), 1)

def isOutsideActiveArea(camerax, cameray, obj):
    
    boundsLeftEdge = camerax - WINWIDTH
    boundsTopEdge = cameray - WINHEIGHT
    boundsRect = pygame.Rect(boundsLeftEdge, boundsTopEdge, WINWIDTH * 3, WINHEIGHT * 3)
    objRect = pygame.Rect(obj['x'], obj['y'], obj['width'], obj['height'])
    return not boundsRect.colliderect(objRect)        

def drawScore(count):
   

    scoreSurf = BASICFONT.render('Score: %s' % count, True, BLACK)
    scoreRect = scoreSurf.get_rect()
    scoreRect.topleft = (50, 20)
    gameDisplay.blit(scoreSurf, scoreRect)
    


if __name__ == '__main__':
    main()

