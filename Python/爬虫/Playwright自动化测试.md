### Playwright 简介
Playwright 是由 微软公司 2020年初 发布的新一代自动化测试工具，相较于目前最常用的 Selenium，它仅用一个 API 即可自动执行 Chromium、Firefox、WebKit 等主流浏览器自动化操作。作为针对 Python 语言纯自动化的工具，在回归测试中可更快的实现自动化


# 安装Playwright
~~~shell
# Playwright 需要 Python >=3.7 版本
pip install playwright

# 安装 Chromium、Firefox、WebKit等浏览器的驱动文件（内置浏览器）和 ffmpeg 用于视频录制
# 全部驱动都安装方式
python -m playwright install

# 按需安装，重新安装一下（chromium 是其中一个浏览器插件，看自己用什么就装什么，有 chromium, chrome, chrome-beta, msedge, msedge-beta, msedge-dev, firefox, webkit 
# PLAYWRIGHT_BROWSERS_PATH=0 是为了支持 pyinstaller 打包报错 Please run the following command to download new browsers 问题
PLAYWRIGHT_BROWSERS_PATH=0 playwright install chromium
~~~

# Playwright自动化网页测试
~~~python
from playwright.sync_api import Playwright, sync_playwright # 导入Playwright 的API

def run (playwright: Playwright) -> None:
  # 创建chromium浏览器实例，headless=False表示浏览器有界面显示
  browser = playwright.chromium.launch(headless=False)

  # 通过浏览器实例创建content上下文，即一个浏览器会话窗口，并且可以设置窗口大小
  content = browser.new_context(viewport={"width": 1920, "height": 1080})

  # 通过浏览器会话窗口可以创建多个页面，也就是多个 tab 栏页面
  page = content.new_page()

  # 通过浏览器页面打开指定网址的网页
  page.goto('https://www.baidu.com')

  # 通过 css选择器 或者 xpath选择器 获取输入标签并且输入内容
  # page.locator('//input[@id="kw"]').fill('周杰伦') 也可以直接写下面这种方式
  page.fill('//input[@id="kw"]', '周杰伦')

  # 通过 css选择器 或者 xpath选择器 获取百度搜索按钮进行点击
  page.click('//input[@id="su"]')

  # 暂停以允许手动继续
  # page.pause()  

  # 保存浏览器网页截图到指定路径
  page.screenshot(path='search_result.png')  # 截图搜索结果

  # playwright 自带的延迟等待，单位是毫秒，防止直接执行下面的代码关闭浏览器
  # 也可以通过一些其他方式延迟关闭，例如sleep
  page.wait_for_timeout(30000)

  # 使用完成关闭上下文（也就是会话窗口）
  content.close()
  # 关闭浏览器
  browser.close()


def main():
  # 获取playwright实例，并将其传递给run函数，设置函数执行完自动关闭playwright实例资源
  with sync_playwright() as playwright:
    run(playwright)


# 这是脚本的入口点。
# 它开始执行main函数。
if __name__ == "__main__":
    main()

~~~










