
# Python飞机大战游戏设计
## 1.代码整体框架
代码整体框架如下：
- 设置游戏界面的大小、标题、背景图片、飞机图片（正常、爆炸）、子弹图片；
- 设置两个`list`，分别存储敌机和被击毁的飞机；
- 初始化分数、射击频率、敌机移动频率，并设置游戏循环帧率；
- 进入游戏的主循环部分；
- 在`gameover`后显示最终得分；
- 处理游戏退出。    

在游戏的主循环部分主要包括以下部分：
- 按一定频率发射子弹；
- 按一定频率生成敌机；
- 移动子弹；
- 移动敌机；
- 敌机与玩家飞机相撞处理方法；
- 敌机被子弹击中处理方法；
- 一系列绘制、显示的方法，包括：绘制背景、绘制玩家飞机、显示子弹、显示敌机、绘制得分、更新屏幕；
- 获取、处理键盘事件；
- 处理退出游戏。    

共建立了3个类，分别是：
- 子弹类；
- 玩家飞机类；
- 敌机类。

## 2.写出每个类及每个函数的作用
### 准备工作


```python
#-*- coding: utf-8 -*-
import pygame
from sys import exit
from pygame.locals import *
import random

# 设置游戏屏幕大小
SCREEN_WIDTH = 480
SCREEN_HEIGHT = 800
```

### 定义子弹类
子弹类包含两个函数：  
- 初始化`_init_`函数定义子弹的基本属性，包括：子弹的图片、位置、移动速度；  
- `move`函数计算子弹位置。


```python
# 子弹类
class Bullet(pygame.sprite.Sprite):
    def __init__(self, bullet_img, init_pos):
        pygame.sprite.Sprite.__init__(self)
        self.image = bullet_img
        self.rect = self.image.get_rect()
        self.rect.midbottom = init_pos
        self.speed = 10

    def move(self):
        self.rect.top -= self.speed
```

### 定义玩家飞机类
玩家飞机类包含六个函数：
- 初始化`__init__`函数定义了飞机的基本属性：图片、大小、位置、速度、是否被击中；
- `shoot`函数调用子弹类，给子弹类传递了实参，包括子弹的图片和位置；
- `moveUp` `moveDown` `moveLeft` `moveRight`函数设置了飞机移动的方法，并设置了飞机到边界时不再移动。


```python
# 玩家飞机类
class Player(pygame.sprite.Sprite):
    def __init__(self, plane_img, player_rect, init_pos):
        pygame.sprite.Sprite.__init__(self)
        self.image = []                                 # 用来存储玩家飞机图片的列表
        for i in range(len(player_rect)):
            self.image.append(plane_img.subsurface(player_rect[i]).convert_alpha())
        self.rect = player_rect[0]                      # 初始化图片所在的矩形
        self.rect.topleft = init_pos                    # 初始化矩形的左上角坐标
        self.speed = 8                                  # 初始化玩家飞机速度，这里是一个确定的值
        self.bullets = pygame.sprite.Group()            # 玩家飞机所发射的子弹的集合
        self.is_hit = False                             # 玩家是否被击中

    # 发射子弹
    def shoot(self, bullet_img):
        bullet = Bullet(bullet_img, self.rect.midtop)
        self.bullets.add(bullet)

    # 向上移动，需要判断边界
    def moveUp(self):
        if self.rect.top <= 0:
            self.rect.top = 0
        else:
            self.rect.top -= self.speed

    # 向下移动，需要判断边界
    def moveDown(self):
        if self.rect.top >= SCREEN_HEIGHT - self.rect.height:
            self.rect.top = SCREEN_HEIGHT - self.rect.height
        else:
            self.rect.top += self.speed

    # 向左移动，需要判断边界
    def moveLeft(self):
        if self.rect.left <= 0:
            self.rect.left = 0
        else:
            self.rect.left -= self.speed

    # 向右移动，需要判断边界        
    def moveRight(self):
        if self.rect.left >= SCREEN_WIDTH - self.rect.width:
            self.rect.left = SCREEN_WIDTH - self.rect.width
        else:
            self.rect.left += self.speed
```

### 定义敌机类
敌机类包含两个函数：
- 初始化`__init__`函数定义了敌机的基本属性：敌机图片、坠毁图片、敌机位置、速度；
- `move`函数计算敌机位置。


```python
# 敌机类
class Enemy(pygame.sprite.Sprite):
    def __init__(self, enemy_img, enemy_down_imgs, init_pos):
       pygame.sprite.Sprite.__init__(self)
       self.image = enemy_img
       self.rect = self.image.get_rect()
       self.rect.topleft = init_pos
       self.down_imgs = enemy_down_imgs
       self.speed = 2

    # 敌机移动，边界判断及删除在游戏主循环里处理
    def move(self):
        self.rect.top += self.speed
```

### 其他相关初始化操作


```python
# 初始化 pygame
pygame.init()

# 设置游戏界面大小、背景图片及标题
# 游戏界面像素大小
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))

# 游戏界面标题
pygame.display.set_caption('Python打飞机大战')

# 背景图
background = pygame.image.load('resources/image/background.png').convert()

# Game Over 的背景图
game_over = pygame.image.load('resources/image/gameover.png')

# 飞机及子弹图片集合
plane_img = pygame.image.load('resources/image/shoot.png')

# 设置玩家飞机不同状态的图片列表，多张图片展示为动画效果
player_rect = []
player_rect.append(pygame.Rect(0, 99, 102, 126))        # 玩家飞机图片
player_rect.append(pygame.Rect(165, 234, 102, 126))     # 玩家爆炸图片

player_pos = [200, 600]
player = Player(plane_img, player_rect, player_pos)

# 子弹图片
bullet_rect = pygame.Rect(1004, 987, 9, 21)
bullet_img = plane_img.subsurface(bullet_rect)

# 敌机不同状态的图片列表，包括正常敌机，爆炸的敌机图片
enemy1_rect = pygame.Rect(534, 612, 57, 43)
enemy1_img = plane_img.subsurface(enemy1_rect)
enemy1_down_imgs = plane_img.subsurface(pygame.Rect(267, 347, 57, 43))


#存储敌机，管理多个对象
enemies1 = pygame.sprite.Group()

# 存储被击毁的飞机
enemies_down = pygame.sprite.Group()

# 初始化射击及敌机移动频率
shoot_frequency = 0
enemy_frequency = 0

# 初始化分数
score = 0

# 游戏循环帧率设置
clock = pygame.time.Clock()

# 判断游戏循环退出的参数
running = True
```

### 游戏主循环


```python
# 游戏主循环
while running:
    # 控制游戏最大帧率为 60
    clock.tick(60)

    # 生成子弹，需要控制发射频率
    # 首先判断玩家飞机没有被击中
    # 循环15次发射一个子弹
    if not player.is_hit:
        if shoot_frequency % 15 == 0:
            player.shoot(bullet_img)
        shoot_frequency += 1
        if shoot_frequency >= 15:
            shoot_frequency = 0

    # 生成敌机，需要控制生成频率
    # 循环50次生成一架敌机
    if enemy_frequency % 50 == 0:
        enemy1_pos = [random.randint(0, SCREEN_WIDTH - enemy1_rect.width), 0]
        enemy1 = Enemy(enemy1_img, enemy1_down_imgs, enemy1_pos)
        enemies1.add(enemy1)
    enemy_frequency += 1
    if enemy_frequency >= 100:
        enemy_frequency = 0

    for bullet in player.bullets:
        # 以固定速度移动子弹
        bullet.move()
        # 移动出屏幕后删除子弹
        if bullet.rect.bottom < 0:
            player.bullets.remove(bullet)   

    for enemy in enemies1:
        #2. 移动敌机
        enemy.move()
        #3. 敌机与玩家飞机碰撞效果处理
        if pygame.sprite.collide_circle(enemy, player):
            enemies_down.add(enemy)
            enemies1.remove(enemy)
            player.is_hit = True
            break
        #4. 移动出屏幕后删除敌人
        if enemy.rect.top < 0:
            enemies1.remove(enemy)

    #敌机被子弹击中效果处理
    #将被击中的敌机对象添加到击毁敌机 Group 中
    enemies1_down = pygame.sprite.groupcollide(enemies1, player.bullets, 1, 1)
    for enemy_down in enemies1_down:
        enemies_down.add(enemy_down)

    # 绘制背景
    screen.fill(0)
    screen.blit(background, (0, 0))

    # 绘制玩家飞机
    if not player.is_hit:
        screen.blit(player.image[0], player.rect) #将正常飞机画出来
    else:
        # 玩家飞机被击中后的效果处理
        screen.blit(player.image[1], player.rect) #将爆炸的飞机画出来
        running = False

    # 敌机被子弹击中效果显示
    for enemy_down in enemies_down:
        enemies_down.remove(enemy_down)
        score += 1
        screen.blit(enemy_down.down_imgs, enemy_down.rect) #将爆炸的敌机画出来


    # 显示子弹
    player.bullets.draw(screen)
    # 显示敌机
    enemies1.draw(screen)

    # 绘制得分
    score_font = pygame.font.Font(None, 36)
    score_text = score_font.render('score: '+str(score), True, (128, 128, 128))
    text_rect = score_text.get_rect()
    text_rect.topleft = [10, 10]
    screen.blit(score_text, text_rect)

    # 更新屏幕
    pygame.display.update()

    # 处理游戏退出
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            exit()

    # 获取键盘事件（上下左右按键）
    key_pressed = pygame.key.get_pressed()

    # 处理键盘事件（移动飞机的位置）
    if key_pressed[K_w] or key_pressed[K_UP]:
        player.moveUp()
    if key_pressed[K_s] or key_pressed[K_DOWN]:
        player.moveDown()
    if key_pressed[K_a] or key_pressed[K_LEFT]:
        player.moveLeft()
    if key_pressed[K_d] or key_pressed[K_RIGHT]:
        player.moveRight()
```

### 游戏结束与游戏退出


```python
# 游戏 Game Over 后显示最终得分
font = pygame.font.Font(None, 64)
text = font.render('Final Score: '+ str(score), True, (255, 0, 0))
text_rect = text.get_rect()
text_rect.centerx = screen.get_rect().centerx
text_rect.centery = screen.get_rect().centery + 24
screen.blit(game_over, (0, 0))
screen.blit(text, text_rect)

# 显示得分并处理游戏退出
while 1:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            exit()
    pygame.display.update()
```

## 3.实验结果展示
![Demo](p_1.png)
![Demo](p_2.png)


```python

```
