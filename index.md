# Foosbar: A computer game based on Foosball.

Controls:

| Action        | Key |
| ------------- |:---:|
| Right Player Switch Handles | ↔ |
| Right Player Move Handle | ↕ |
| Left Player Switch Handles | A/D |
| Left Player Move Handle | W/S |

Source code:
```
'''Foosbar - a computer game based on foosball'''

import pygame, random, time
from pygame.locals import *

pygame.init()
screenw = 900
screenh = 500
screen = pygame.display.set_mode((screenw, screenh))
pygame.display.set_caption('Foosbar')

'''
Handle Class - handles the handles and the players.
Arguments:
players    - the number of players on the handle.
side       - the side the handle is on. Either 'left' or 'right'.
slot       - the number of handles from the left side that the handle is located on.
totalslots - the total number of handles in the game.
colorset   - the color of the players on the handle. Must be a list with two RGB tuples.
'''
class handle():
    def __init__(self, players, side, slot, totalslots, colorset = [(100,255,100),(100,100,255)]):
        self.players = players
        self.side = side
        self.totalslots = totalslots
        self.slot = slot

        self.spacing = screenh/(players+1)
        self.topspace = self.spacing

        if players == 3:
            self.state = 'ready'
        else:
            self.state = 'idle'

        if self.side == 'left':
            self.color = colorset[0]
        elif self.side == 'right':
            self.color = colorset[1]

        self.idleradius = 10
        self.readyradius = 12
        self.activeradius = 18

        self.rects = []
        for i in range(self.players):
            self.rects.append(pygame.rect.Rect(((screenw/(self.totalslots+1))*(self.slot+1))-self.readyradius,self.spacing*i+self.topspace-self.readyradius,self.readyradius*2,self.readyradius*2))
        print self.rects
    def move(self, direction):
        if direction == 'up' and self.topspace >= 0:
            self.topspace -= 20
        elif self.topspace <= self.spacing*2:
            self.topspace += 20
        self.rects = []
        for i in range(self.players):
            self.rects.append(pygame.rect.Rect(((screenw/(self.totalslots+1))*(self.slot+1))-self.readyradius,self.spacing*i+self.topspace-self.readyradius,self.readyradius*2,self.readyradius*2))

    def render(self):
        handlex = (screenw/(self.totalslots+1))*(self.slot+1)
        if self.state == 'idle':
            radius = self.idleradius
        if self.state == 'ready':
            radius = self.readyradius
        if self.state == 'active':
            radius = self.activeradius
        #for x in self.rects:
        #   pygame.draw.rect(screen,(255,255,255),x)
        pygame.draw.line(screen,(50,50,50),(handlex,0),(handlex,screenh),5)
        for i in range(self.players):
            if self.state == 'idle':
                if self.side == 'left':
                    color = (100,150,100)
                if self.side == 'right':
                    color = (100,100,150)
            else:
                color = self.color
            pygame.draw.circle(screen,color,(handlex,self.spacing*i+self.topspace),radius)

'''
Balls class
Controls the foosball.
methods:
playerCollide - manage collisions with players
wallCollide   - manage collisions with edges
render        - draw ball onto surface, and moves it.
reset         - reset the ball.
'''
class balls():
    def __init__(self, radius):
        self.radius = radius
        self.x = screenw/2
        self.y = screenh/2
        self.xspeed = random.choice([random.randint(-10,-5),random.randint(5,10)])
        self.yspeed = random.choice([random.randint(-10,-5),random.randint(5,10)])
        self.rect = pygame.rect.Rect(self.x-radius,self.y-radius,radius*2,radius*2)

    def playerCollide(self,playerrects,side):
        if side == 'left':
            force = 1
        if side == 'right':
            force = -1
        for rect in playerrects:
            if self.rect.colliderect(rect):
                self.xspeed = self.xspeed*(-1)+force
                print 'hit player'

    def wallCollide(self):
        if self.x+self.radius >= screenw or self.x-self.radius <= 0:
            print 'hit sides'
            self.xspeed=self.xspeed*(-1)
        if self.y+self.radius >= screenh or self.y-self.radius <= 0:
            print 'hit top/bottom'
            print self.yspeed
            self.yspeed=self.yspeed*(-1)
            print self.yspeed
        
    def reset(self):
        self.x = screenw/2
        self.y = screenh/2
        self.xspeed = random.choice([random.randint(-10,-5),random.randint(5,10)])
        self.yspeed = random.choice([random.randint(-10,-5),random.randint(5,10)])

    def render(self):
        pygame.draw.circle(screen,(255,255,255),(self.x,self.y),self.radius)
        self.x+=self.xspeed
        self.y+=self.yspeed
        self.rect = pygame.rect.Rect(self.x-self.radius,self.y-self.radius,self.radius*2,self.radius*2)
        #pygame.draw.rect(screen,(100,100,100),self.rect)

'''
GOAL class - controls the goals.
Arguments:
side - the side that the goal is located on.
length - the length of the goal.
Attributes:
score - the number of times that the ball has collided with the ball.
side - the side that the goal is located on.
Methods:
scoreKeep - Detect whether a player has scored, and increments the score accordingly. 
render - Render the goal. 
'''
class goal():
    def __init__(self, side, length, width=10):
        self.side = side
        self.length = length
        self.width = width
        self.score = 0
        
        if self.side == 'left':
            xcoord = 0
        if self.side == 'right':
            xcoord = screenw-self.width
        ycoord = (screenh-self.length)/2
        self.rect = pygame.rect.Rect(xcoord,ycoord,self.width,self.length)

    def scoreKeep(self, ballrect):
        if self.rect.colliderect(ballrect):
            self.score += 1
            return True
    
    def render(self, color):
        pygame.draw.rect(screen,color,self.rect)

'''MAIN FUNCTION'''
def main():
    playerlayout = [['left',3],['right',2],['left',4],['right',4],['left',2],['right',3]]
    handles = []
    for i in range(len(playerlayout)):
        handles.append(handle(playerlayout[i][1],playerlayout[i][0],i,len(playerlayout)))
    ball = balls(14)
    rightdirection = ''
    leftdirection = ''
    leftGoal = goal('left',180)
    rightGoal = goal('right',180)
    scoreFont = pygame.font.Font(None,150)
    winFont = pygame.font.Font(None,100)
    t = pygame.time.Clock()
    while True:
        screen.fill((20,20,20))
        pygame.draw.circle(screen, (50,50,50),(screenw/2,screenh/2), 20, 3)
        '''RENDER HANDLES AND PLAYERS'''
        for item in handles:
            item.render()
            if item.state == 'ready':
                ball.playerCollide(item.rects,item.side)
        '''HANDLE GOALS'''
        if leftGoal.scoreKeep(ball.rect) or rightGoal.scoreKeep(ball.rect):
            time.sleep(0.1)
            ball.reset()
            pygame.display.flip()
            time.sleep(0.3)
        leftGoal.render((255,255,255))
        rightGoal.render((255,255,255))
        rightScoreText = scoreFont.render(str(leftGoal.score),1,(150,150,150))
        leftScoreText  = scoreFont.render(str(rightGoal.score),1,(150,150,150))
        screen.blit(rightScoreText,(screenw*3/4,screenh/8))
        screen.blit(leftScoreText, (screenw/4-60,screenh/8))
        end = False
        if leftGoal.score == 10:
            leftWinText = winFont.render('Right Player Wins!',1,(100,100,255),(50,50,50))
            screen.blit(leftWinText, ((screenw-winFont.size('Right Player Wins!')[0])/2,(screenh-winFont.size('Right Player Wins!')[1])/2))
            end = True
        if rightGoal.score == 10:
            rightWinText = winFont.render('Left Player Wins!',1,(100,255,100),(50,50,50))
            screen.blit(rightWinText, ((screenw-winFont.size('Left Player Wins!')[0])/2,(screenh-winFont.size('Left Player Wins!')[1])/2))
            end = True
        while end:
            pygame.display.flip()
            t.tick(10)
            for event in pygame.event.get():
                if event.type == KEYDOWN:
                    end = False
                    rightGoal.score = 0
                    leftGoal.score = 0
                if event.type == QUIT:
                    pygame.quit()
                        
        '''RENDER BALL'''
        ball.render()
        ball.wallCollide()
        '''MOVE HANDLES'''
        for item in handles:
            if (item.state == 'ready' or item.state == 'active') and item.side == 'right':
                if rightdirection == 'up':
                    item.move('up')
                if rightdirection == 'down':
                    item.move('down')
            if (item.state == 'ready' or item.state == 'active') and item.side == 'left':
                if leftdirection == 'up':
                    item.move('up')
                if leftdirection == 'down':
                    item.move('down')
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
            '''DETECT KEYPRESSES'''
            if event.type == KEYDOWN:
                '''RIGHT UP/DOWN'''
                if event.key == K_UP:
                    for item in handles:
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'right':
                            rightdirection = 'up'
                if event.key == K_DOWN:
                    for item in handles:
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'right':
                            rightdirection = 'down'
                '''RIGHT SWITCH HANDLES (Left/right)'''
                if event.key == K_RIGHT:
                    for i in range(len(handles)):
                        item = handles[i]
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'right' and item.slot != item.totalslots-1 and item.slot != item.totalslots-2:
                            item.state = 'idle'
                            handles[i+2].state = 'ready'
                            break
                if event.key == K_LEFT:
                    for i in range(len(handles)):
                        item = handles[i]
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'right' and item.slot != 0 and item.slot != 1:
                            item.state = 'idle'
                            handles[i-2].state = 'ready'
                            break
                '''LEFT SWITCH HANDLES (a/d)'''
                if event.key == K_d:
                    for i in range(len(handles)):
                        item = handles[i]
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'left' and item.slot != item.totalslots-1 and item.slot != item.totalslots-2:
                            item.state = 'idle'
                            handles[i+2].state = 'ready'
                            break
                if event.key == K_a:
                    for i in range(len(handles)):
                        item = handles[i]
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'left' and item.slot != 0 and item.slot != 1:
                            item.state = 'idle'
                            handles[i-2].state = 'ready'
                            break
                '''LEFT UP/DOWN (w/s)'''
                if event.key == K_w:
                    for item in handles:
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'left':
                            leftdirection = 'up'
                if event.key == K_s:
                    for item in handles:
                        if (item.state == 'ready' or item.state == 'active') and item.side == 'left':
                            leftdirection = 'down'
            if event.type == KEYUP:
                if event.key == K_UP or event.key == K_DOWN:
                    rightdirection = ''
                if event.key == K_w or event.key == K_s:
                    leftdirection = ''
        pygame.display.flip()
        t.tick(30)

if __name__ == '__main__':
    main()
```
Released under the MIT License
