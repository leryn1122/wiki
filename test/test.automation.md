
# 自动化测试
自动化测试这里主要介绍的 Selenium Python 框架的教程。

## Selenium 框架
参考文档：

- [1. Installation — Selenium Python Bindings 2 documentation](https://selenium-python.readthedocs.io/installation.html)
- [https://sites.google.com/chromium.org/driver/downloads?authuser=0](https://sites.google.com/chromium.org/driver/downloads?authuser=0)
- [Chrome for Testing availability](https://googlechromelabs.github.io/chrome-for-testing/)

Selenium 是一款用于 Web 应用自动化测试的测试框架，可以利用浏览器驱动模拟用户在浏览器中的操作。
可以通过 Python 带来启动一个浏览器实例来自动化测试 Web 应用，对于 iOS 和 Android 移动端应用也可以很好的支持。同样也可以用于模拟 JS 运行环境来爬取 SPA 网页应用。
```bash
pipenv install selenium
```
下载对应的浏览器驱动，Selenium 支持多种浏览器，其他浏览器有：

- Chrome
- Firefox
- IE
- Edge
- Opera
- PhantomJS

我选择 ChromeDriver，需要根据当前浏览器和框架来下载对应版本的浏览器驱动。参考上面的 Chrome 官方驱动下载页面。
```python
from selenium import webdriver
from selenium.webdriver.chrome.webdriver import WebDriver

def create_webdriver_instance(path: str = None,
                              enable_proxy: bool = False) -> WebDriver:
    """
    Use `http://127.0.0.1:7890` as local http_proxy.
    """
    option = webdriver.ChromeOptions()
    option.add_experimental_option('excludeSwitches', ['enable-automation'])
    option.add_experimental_option('useAutomationExtension', False)
    option.add_argument('--headless')
    option.add_argument('--no-sandbox')
    option.add_argument('--disable-gpu')
    option.add_argument('--disable-dev-shm-usage')
    option.add_argument('--disable-blink-features=AutomationControlled')
    option.add_argument('--use-agent=%s' % user_agent())
    if path:
        option.binary_location(path)
    if enable_proxy:
        option.add_argument('--proxy-server=http://127.0.0.1:7890')
    return webdriver.Chrome(options=option)
```
如果有的网页由 Cloudflare 人机检测守护，使用 `stealth.min.js` 隐藏 Selenium 特征越过人机检测：

- [stealth.min.js - jsDelivr CDN](https://cdn.jsdelivr.net/gh/requireCool/stealth.min.js/stealth.min.js)
```python
from selenium_stealth import stealth

def skip_cloudflare_waf(driver: webdriver.Chrome):
    """
    Use `stealth.min.js` to skip Cloudflare bot verification.
    """
    with open('resources/stealth.min.js', 'r') as f:
        js = f.read()
    driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {
        'source': js
    })
    stealth(driver,
            languages=['en-US', 'en'],
            vendor='Google Inc.',
            platform='Win32',
            webgl_vendor='Intel Inc.',
            renderer='Intel Iris OpenGL Engine',
            fix_hairline=True,
            )
```

### 定位元素
然后可以用过定位器定位元素：
```python
driver.get("https://github.com")
driver.maximize_window()
driver.execute_script("window.scrollTo(0, document.body.scrollHeight)")

# 用定位器查找标签
locator = (By.XPATH, '//*[@id="read-only-cursor-text-area"]')
button = driver.find_element(locator[0], locator[1])

# 用 API 查找标签
driver.find_element_by_id()
driver.find_element_by_name()
driver.find_element_by_class_name()
driver.find_element_by_tag_name()
driver.find_element_by_link_text()
driver.find_element_by_partial_link_text()
driver.find_element_by_xpath()
driver.find_element_by_css_selector()
```

### 发送键盘事件
```python
element = driver.find_element_by_id("kw")

element.send_keys(Keys.BACK_SPACE)
element.send_keys(Keys.SPACE)
element.send_keys(Keys.TAB)
element.send_keys(Keys.ESCAPE)
element.send_keys(Keys.ENTER)
element.send_keys(Keys.CONTROL,'a')
element.send_keys(Keys.CONTROL,'c')
element.send_keys(Keys.CONTROL,'x')
element.send_keys(Keys.CONTROL,'v')
element.send_keys(Keys.F1) 键盘 F1
element.send_keys(Keys.F12) 键盘 F12
```
