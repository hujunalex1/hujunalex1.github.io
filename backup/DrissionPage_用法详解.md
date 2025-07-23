

## 概述
DrissionPage 是一个功能强大的 Python 网页自动化库，它将 requests 和 selenium 的优点结合在一起，提供了更加灵活高效的网页操作方式。

## 安装
```bash
pip install DrissionPage
```

## 主要特点
- 同时支持无头模式和有头模式
- 可以在 requests 模式和 selenium 模式之间切换
- 提供更简洁的 API
- 支持多标签页操作
- 内置元素等待机制

## 基本用法

### 1. 导入和初始化
```python
from DrissionPage import WebPage

# 创建页面对象
page = WebPage()
# 或者指定模式
page = WebPage(mode='s')  # selenium 模式
page = WebPage(mode='d')  # requests 模式
```

### 2. 页面导航
```python
# 访问网页
page.get('https://www.example.com')

# 获取当前网址
current_url = page.url

# 页面前进后退
page.back()
page.forward()
page.refresh()
```

### 3. 元素定位
```python
# 通过不同方式定位元素
element = page.ele('#id')           # ID
element = page.ele('.class')        # 类名
element = page.ele('tag:div')       # 标签
element = page.ele('text:搜索')      # 文本内容
element = page.ele('@value=submit') # 属性
element = page.ele('xpath://div[@id="content"]') # XPath

# 定位多个元素
elements = page.eles('.item')
```

### 4. 元素操作
```python
# 点击元素
element.click()

# 输入文本
element.input('文本内容')

# 获取文本
text = element.text

# 获取属性
value = element.attr('value')

# 获取HTML
html = element.html
```

### 5. 表单操作
```python
# 填写表单
page.ele('#username').input('用户名')
page.ele('#password').input('密码')
page.ele('#login-btn').click()

# 选择下拉框
page.ele('#select').select('选项值')

# 上传文件
page.ele('#file-input').input('/path/to/file.txt')
```

### 6. 等待机制
```python
# 等待元素出现
element = page.wait.ele_displayed('#loading')

# 等待元素消失
page.wait.ele_deleted('#loading')

# 等待页面加载完成
page.wait.load_complete()

# 自定义等待时间
page.wait(3)  # 等待3秒
```

### 7. 多标签页操作
```python
# 打开新标签页
page.new_tab('https://www.example.com')

# 切换标签页
page.to_tab(1)  # 切换到第2个标签页

# 关闭当前标签页
page.close_tab()

# 获取所有标签页
tabs = page.tab_ids
```

### 8. 模式切换

**ChromiumPage模式**：专门用于控制浏览器，适合需要动态渲染或用户交互的场景，如登录、表单提交等

```
from DrissionPage import ChromiumPage
page = ChromiumPage()  # 默认有头模式
page = ChromiumPage headless=True  # 无头模式
```

**SessionPage模式**：基于requests的会话对象，适合静态页面或API请求，无需启动浏览器即可高效获取数据

```
from DrissionPage import SessionPage
page = SessionPage()  # 创建会话对象
```

**WebPage模式**：结合前两种模式的优势，既可控制浏览器进行交互，又可收发数据包，特别适合需要先登录再爬取数据的场景

```
from DrissionPage import WebPage
page = WebPage()  # 默认使用ChromiumPage模式
page = WebPage 's'  # 使用SessionPage模式
```
```python
# 切换到 selenium 模式（有头浏览器）
page.change_mode('s')

# 切换到 requests 模式（无头请求）
page.change_mode('d')
```

### 9. 截图和下载
```python
# 页面截图
page.screenshot('/path/to/screenshot.png')

# 元素截图
element.screenshot('/path/to/element.png')

# 下载文件
page.download('https://example.com/file.pdf', '/path/to/save/')
```

### 10. 处理弹窗
```python
# 处理JavaScript弹窗
page.handle_alert('accept')  # 接受
page.handle_alert('dismiss') # 拒绝
```

## 实际应用示例

### 示例1：搜索操作
```python
from DrissionPage import WebPage

page = WebPage()
page.get('https://www.baidu.com')

# 输入搜索关键词
search_box = page.ele('#kw')
search_box.input('DrissionPage')

# 点击搜索按钮
search_btn = page.ele('#su')
search_btn.click()

# 获取搜索结果
results = page.eles('.result')
for result in results:
    title = result.ele('h3').text
    print(title)
```

### 示例2：登录操作
```python
from DrissionPage import WebPage

page = WebPage()
page.get('https://example.com/login')

# 填写登录表单
page.ele('#username').input('your_username')
page.ele('#password').input('your_password')
page.ele('#login-button').click()

# 等待登录完成
page.wait.url_change()
print('登录成功')
```

### 示例3：数据抓取
```python
from DrissionPage import WebPage
import time

page = WebPage()
page.get('https://example.com/products')

# 滚动加载更多内容
for i in range(5):
    page.scroll.to_bottom()
    time.sleep(2)

# 抓取商品信息
products = page.eles('.product-item')
for product in products:
    name = product.ele('.product-name').text
    price = product.ele('.product-price').text
    print(f'商品: {name}, 价格: {price}')
```

### 示例4：文件下载
```python
from DrissionPage import WebPage

page = WebPage()
page.get('https://example.com/files')

# 点击下载链接
download_links = page.eles('.download-link')
for link in download_links:
    file_url = link.attr('href')
    page.download(file_url, './downloads/')
```

## 高级功能

### 1. 配置选项
```python
from DrissionPage import WebPage, ChromiumOptions

# 配置浏览器选项
options = ChromiumOptions()
options.add_argument('--headless')  # 无头模式
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')
options.set_user_agent('自定义User-Agent')
options.set_window_size(1920, 1080)

# 使用配置创建页面
page = WebPage(chromium_options=options)
```

### 2. Cookie 操作
```python
# 获取所有 Cookie
cookies = page.cookies()

# 添加 Cookie
page.set_cookies({'name': 'value'})

# 删除 Cookie
page.remove_cookies('cookie_name')
```

### 3. 执行 JavaScript
```python
# 执行 JavaScript 代码
result = page.run_js('return document.title')

# 异步执行 JavaScript
page.run_js_loaded('console.log("页面加载完成")')
```

### 4. 代理设置
```python
from DrissionPage import ChromiumOptions, WebPage

# 设置代理
options = ChromiumOptions()
options.add_argument('--proxy-server=http://proxy.example.com:8080')

page = WebPage(chromium_options=options)
```

### 5. 处理iframe
```python
# 切换到iframe
iframe = page.ele('iframe')
page.to_frame(iframe)

# 在iframe中操作
element = page.ele('#element-in-iframe')
element.click()

# 切换回主页面
page.to_main_frame()
```

## 最佳实践

### 1. 错误处理
```python
from DrissionPage import WebPage
from DrissionPage.errors import ElementNotFoundError

page = WebPage()
try:
    page.get('https://example.com')
    element = page.ele('#element-id')
    element.click()
except ElementNotFoundError:
    print("元素未找到")
except Exception as e:
    print(f"发生错误: {e}")
```

### 2. 智能等待
```python
# 使用智能等待，避免硬编码时间
page.wait.load_complete()  # 等待页面加载完成
element = page.wait.ele_displayed('#dynamic-element', timeout=10)
```

### 3. 性能优化
```python
# 在不需要渲染的情况下使用 requests 模式
page = WebPage(mode='d')  # 更快的请求模式

# 需要 JavaScript 时切换到 selenium 模式
page.change_mode('s')
```

## 常见问题

### Q1: 如何处理动态加载的内容？
```python
# 等待元素出现
element = page.wait.ele_displayed('.dynamic-content')

# 或者使用滚动触发加载
page.scroll.to_bottom()
page.wait(2)  # 等待内容加载
```

### Q2: 如何处理验证码？
```python
# 截图查看验证码
captcha_element = page.ele('#captcha')
captcha_element.screenshot('./captcha.png')

# 手动输入或使用 OCR 识别
captcha_code = input("请输入验证码: ")
page.ele('#captcha-input').input(captcha_code)
```

### Q3: 如何批量处理多个页面？
```python
urls = ['https://example.com/page1', 'https://example.com/page2']

for url in urls:
    page.get(url)
    # 处理页面内容
    title = page.title
    print(f"页面标题: {title}")
```

## 与其他库的对比

| 特性 | DrissionPage | Selenium | Requests |
|------|-------------|----------|----------|
| 简易性 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 性能 | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| JavaScript支持 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ❌ |
| 模式切换 | ⭐⭐⭐⭐⭐ | ❌ | ❌ |
| 文档完整性 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

## 总结

DrissionPage 是一个非常优秀的网页自动化库，特别适合：

1. **网页爬虫项目** - 既能处理静态内容，也能处理 JavaScript 渲染的动态内容
2. **自动化测试** - 提供比 Selenium 更简洁的 API
3. **数据采集** - 支持模式切换，在性能和功能间取得平衡
4. **Web 自动化** - 丰富的功能集合，满足各种自动化需求

