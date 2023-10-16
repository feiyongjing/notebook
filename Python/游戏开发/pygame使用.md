### 按照pygame pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pygame 
### 测试是否按照完成 python -m pygame.examples.aliens 执行会出现一个小游戏窗口

### 创建游戏窗口并加载图片素材
~~~python
import pygame

if __name__ == '__main__':

    pygame.init()

    print("开始游戏")

    sereen = pygame.display.set_mode(size=(480, 700), flags=pygame.RESIZABLE | pygame.DOUBLEBUF | pygame.HWSURFACE)

    backgroundImg = pygame.image.load("./images/background.jpg")
    warcraftImg = pygame.image.load("./images/warcraft.png")

    sereen.blit(backgroundImg, (0, 0))
    sereen.blit(warcraftImg, (150, 500))
    pygame.display.update()

    # 定义一个矩形对象，设置矩形在窗口的地址和矩形的宽高
    # 矩形对象的x、y属性表示矩形在窗口的地址
    # 矩形对象的size属性表示矩形的宽高
    hero_reck = pygame.Rect(150, 500, 160, 180)

    # 创建游戏时间对象
    clock = pygame.time.Clock()

    # 游戏循环正式开始
    while True:
        # 设置游戏每秒的帧率
        clock.tick(60)

        event_list = pygame.event.get()

        if len(event_list) >0 :
            print(event_list)

            for event in event_list:
                if event.type==pygame.QUIT:
                    pygame.quit()
                    exit()

        # 设置矩形向上移动
        hero_reck.y -= 1

        sereen.blit(backgroundImg, (0, 0))
        sereen.blit(warcraftImg, hero_reck)

        pygame.display.update()

    pygame.quit()

~~~
---

# 游戏运行步骤
~~~python
import pygame

if __name__ == '__main__':
    # 游戏开始之前需要使用 pygame.init() 导入并加载所有的pygame模块，使用其他模块之前必须先调用init()方法
    pygame.init()

    print("游戏窗口创建")

    print("游戏窗口内图片的绘制，即各种素材加载到游戏窗口中")

    print("应用更新窗口绘制图片位置的显示，即刷新游戏窗口更新游戏窗口内素材的位置")

    print("创建游戏时间对象")

    print("开始游戏的循环，并且使用创建的游戏事件对象控制游戏帧率")

    print("游戏循环中监听各种用户事件，通过事件来做出响应的操作，如下例子")

    print("游戏循环中监听素材移动事件，做出对应的素材重新在窗口绘制图片位置的显示，注意一般都是先绘制背景覆盖原有窗口")

    print("游戏循环中监听退出游戏事件，进行退出游戏关闭窗口")

    # 游戏结束之前需要使用 pygame.quit() 卸载pygame模块
    pygame.quit()
~~~
---


# 游戏窗口的创建和窗口背景图片的绘制以及其他素材的绘制

### pygame.display模块负责创建、管理游戏窗口
### pygame.image模块负责素材图片模块
~~~python
import pygame

if __name__ == '__main__':
 
    pygame.init()

    # set_mode()函数用于创建游戏窗口，返回窗口对象
    # size参数控制窗口的宽高，
    # flags参数表示窗口的特性，可以是以下常量的按位或组合：
    # pygame.RESIZABLE：窗口可以被调整大小。
    # pygame.NOFRAME：窗口没有边框和标题栏。
    # pygame.FULLSCREEN：窗口全屏显示。
    # pygame.DOUBLEBUF：使用双缓冲技术，可以避免屏幕闪烁。
    # pygame.HWSURFACE：使用硬件加速，可以提高渲染速度。
    sereen = pygame.display.set_mode(size=(480, 700), flags=pygame.RESIZABLE | pygame.DOUBLEBUF | pygame.HWSURFACE)

    
    # load()函数用于加载背景图片和其他素材图片
    backgroundImg = pygame.image.load("./images/background.jpg")
    warcraftImg = pygame.image.load("./images/warcraft.png")

    # blit()函数绘制窗口内各种素材图片的位置
    # source参数设置图片资源图片
    # dest参数设置图片在窗口中的位置
    sereen.blit(backgroundImg, (0, 0))
    sereen.blit(warcraftImg, (150, 500))

    # update()函数应用更新窗口绘制所有图片位置的显示，即刷新游戏窗口更新游戏窗口内素材的位置
    pygame.display.update()

    pygame.quit()
~~~
---


# 游戏时间对象控制游戏循环的帧率
~~~python
import pygame

if __name__ == '__main__':

    pygame.init()

    # 创建游戏时间对象
    clock = pygame.time.Clock()

    i=0

    # 游戏循环正式开始
    while True:
        # 设置游戏每秒的帧率，即每秒循环60次
        clock.tick(60)

        print(i)
        
    pygame.quit()
~~~
---


# 游戏循环中监听各种用户事件
### pygame.event是用户事件监听模块
~~~python
import pygame

if __name__ == '__main__':

    pygame.init()

    # 游戏循环正式开始
    while True:
        # 设置游戏每秒的帧率
        clock.tick(60)

        # get()函数获取同一时间用户做的事件列表
        event_list = pygame.event.get()

        if len(event_list) >0 :
            print(event_list)

            for event in event_list:
                # 检测退出游戏事件
                if event.type==pygame.QUIT:
                    # 游戏结束之前需要卸载pygame模块
                    pygame.quit()
                    # 退出程序
                    exit()
    
    pygame.quit()
~~~
---

# Rect矩形对象：其本质是一个矩形表示在窗口中的位置
~~~python
import pygame

if __name__ == '__main__':
 
    pygame.init()

    # 创建游戏窗口
    sereen = pygame.display.set_mode(size=(480, 700), flags=pygame.RESIZABLE | pygame.DOUBLEBUF | pygame.HWSURFACE)

    
    # 加载背景图片和其他素材图片
    backgroundImg = pygame.image.load("./images/background.jpg")
    warcraftImg = pygame.image.load("./images/warcraft.png")

    # 定义一个矩形对象，设置矩形在窗口的地址和矩形的宽高
    # 矩形对象的x、y属性表示矩形在窗口的地址
    # 矩形对象的size属性表示矩形的宽高
    hero_reck = pygame.Rect(150, 500, 160, 180)

    # 绘制窗口内各种素材图片的位置
    sereen.blit(backgroundImg, (0, 0))
    # 直接通过矩形对象表示图片素材在窗口中的位置，注意矩形对象的宽高需要和图片的宽高一致
    sereen.blit(warcraftImg, hero_reck)

    # 刷新游戏窗口更新游戏窗口内素材的位置
    pygame.display.update()

    pygame.quit()
~~~

# 精灵和精灵组
### 上述各种素材的加载、位置绘制、刷新窗口位置比较麻烦，所以引进了 精灵类 和 精灵组类 简化操作
### 精灵类在存储了 加载的素材 和 素材在窗口中的位置（Rect矩形对象）并且提供了 update()函数直接刷新游戏窗口内素材的位置，以及 kill()函数从所有精灵组中移除
### 精灵组类是用于管理一组精灵类，提供了 add() 函数向精灵组中添加精灵对象，sprites()函数查询精灵组中的精灵列表，update()函数让精灵组中的全部精灵调用 update() 函数刷新游戏窗口内素材的位置，draw()函数让精灵组中全部精灵内的素材都绘制到指定的窗口中，注意精灵组对象调用完 draw() 函数之后还是需要使用 pygame.display.update() 刷新游戏窗口更新游戏窗口内素材的位置

## 自定义精灵类使用和精灵组
### 精灵类是 pygame.sprite.Sprite 这个类，所有自定义精灵类需要继承这个类
~~~python
import pygame
import time

class GameSprite(pygame.sprite.Sprite):

    def __init__(self, image_name: str, speed=1):
        '''
        加载素材和初始化矩形对象、移动速度等自定义属性
        '''
        super().__init__()
        self.image = pygame.image.load(image_name)
        self.rect = self.image.get_rect()
        self.speed = speed

    def update(self, *args: Any, **kwargs: Any) -> None:
        '''
        重写update函数，手动实现素材的移动
        '''
        self.rect.y += self.speed

if __name__ == '__main__':
    
    pygame.init()

    # 创建窗口
    sereen = pygame.display.set_mode(size=(480, 700), flags=pygame.RESIZABLE | pygame.DOUBLEBUF | pygame.HWSURFACE)

    # 创建精灵组，并创建精灵添加进精灵组
    group = pygame.sprite.Group()

    gameSprite = GameSprite()
    group.add(gameSprite)

    # 移动精灵的位置，并重新绘制精灵组中的精灵位置
    group.update()
    group.draw(screen)
    pygame.display.update()

    # 睡眠30秒
    time.sleep(30)

    # 从精灵组中移除精灵，并重新绘制精灵组中的精灵位置
    gameSprite.kill()

    group.update()
    group.draw(screen)
    pygame.display.update()

    pygame.quit()

~~~
---

# 检查精灵碰撞
~~~python
import pygame
# groupcollide()函数的第一参数和第二参数都是精灵组，第三和第四参数是布尔值，第五参数是碰撞检测函数
# 检查两个精灵组中是否有精灵产生碰撞了,默认不传碰撞检测函数就是按照精灵中的 Rect矩形对象判断是否碰撞，
# 如果有第五参数碰撞检测函数，就按照碰撞检测函数检查碰撞，碰撞检测函数应该接受两个精灵作为值，并返回一个bool值来表示是否碰撞
# 第三和第四参数的布尔值表示是否将产生碰撞的精灵从第一和第二参数的精灵组中移除
pygame.sprite.groupcollide(bullet_group, enemy_group, True, True, collision)
~~~
---

# 定时器事件
~~~python
import pygame

if __name__ == '__main__':
    
    pygame.init()
    # 设置定时器事件
    # set_timer()函数的第一参数是自定义事件的ID，其实就是一个int数字
    # 第二参数是每隔多少时间触发触发一次事件
    pygame.time.set_timer(CREATE_ENEMY_EVENT, 1000)

    # 游戏循环正式开始
    while True:
        # 设置游戏每秒的帧率
        clock.tick(60)

        # pygame.event.get() 获取监听的全部事件，在其中找到指定的自定义事件
        for event in pygame.event.get():
            print(event)
            if event.type == CREATE_ENEMY_EVENT:
                # 捕获敌机创建事件
                print("创建敌机")
    
    pygame.quit()

~~~
---

# 键盘事件捕获
~~~python
import pygame

if __name__ == '__main__':
    
    pygame.init()

    # 游戏循环正式开始
    while True:
        # 键盘事件捕获方式一
        # pygame.event.get() 获取监听的全部事件，在其中找到指定的键盘事件
        for event in pygame.event.get():
            print(event)
            if event.type == pygame.K_DOWN and event.key == pygame.K_RIGHT:
                print("判断按下了向右方向键，缺点是即使按着键盘也只会触发一次事件")

        # 键盘事件捕获方式二
        # 获取键盘字典列表，如果按下了某个键盘则这个键盘key的value是1，直到松开键盘变为0
        keys_pressed = pygame.key.get_pressed()
        if keys_pressed[pygame.K_RIGHT]:
            print("判断按下了向右方向键，不放手会一直触发")
        elif keys_pressed[pygame.K_LEFT]:
            print("判断按下了向左方向键，不放手会一直触发")
        else:
            print("判断按下了其他键，不放手会一直触发")

    pygame.quit()

~~~