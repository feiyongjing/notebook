### 使用 aiohttp 库发起异步HTTP请求
~~~python
# 导入异步IO库
import asyncio

# 安装aiohttp库 pip install -i https://pypi.tuna.tsinghua.edu.cn/simple aiohttp
import aiohttp

# 安装aiofiles库 pip install -i https://pypi.tuna.tsinghua.edu.cn/simple aiofiles
import aiofiles

# 时间库
import time

import platform


# 异步下载函数
async def aiodownload(**img):
    # 异步操作的资源中必须添加 async 关键字
    async with aiohttp.ClientSession() as session:
        async with session.get(img["url"]) as resp:
            content = await resp.content.read() # aiohttp读取响应是异步操作必须添加 await 关键字
            async with aiofiles.open(img["name"], mode="wb") as f:
                await f.write(content)          # aiofiles写入文件是异步操作必须添加 await 关键字


# 创建协程任务
async def main(*images):
    tasks = [asyncio.create_task(aiodownload(**img)) for img in images]
    await asyncio.wait(tasks)


if __name__ == '__main__':
    # 检查平台，windows平台需要注意设置事件循环策略
    if platform.system() == 'Windows':
        asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    t1 = time.time()
    # 需要下载的图片名称和链接地址
    images = [
        {"name": "沙漠皇帝阿兹尔.jpg",
         "url": "https://game.gtimg.cn/images/lol/act/img/skinloading/41d4cdd8-0f29-4fd7-a849-2b8f478da4b5.jpg"},
        {"name": "疾风剑豪亚索.jpg",
         "url": "https://game.gtimg.cn/images/lol/act/img/skinloading/23b8e9d6-e81c-47fa-9b96-491dd2d1f2d2.jpg"},
        {"name": "青钢影卡蜜尔.jpg",
         "url": "https://game.gtimg.cn/images/lol/act/img/skinloading/1418505c-d1fe-4deb-9566-03856ee618cc.jpg"}
    ]
    asyncio.run(main(*images))
    t2 = time.time()
    print("一共使用了", t2 - t1)
~~~
---