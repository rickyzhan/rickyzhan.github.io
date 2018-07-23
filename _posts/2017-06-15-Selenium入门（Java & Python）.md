---
layout: post
title: Selenium入门（Java & Python）
tags: [Selenium, Java, Python, Portfolio]
---

> Selenium，作为一个Web应用程序测试的工具，功能显然更加强大。Selenium的核心Selenium Core基于JsUnit，完全由JavaScript编写，因此可运行于任何支持JavaScript的浏览器上。显然，Selenium非常适合解决上述我们提到的动态网页加载问题。  

### Java 篇

> 这里我们主要讲一下Selenium如何在java平台上使用、以及一些使用过程中出现的坑。

提供api的jar包：

```
commons-codec-1.9.jar
commons-collections-3.2.1.jar
commons-exec-1.1.jar
commons-io-2.4.jar
commons-lang3-3.3.2.jar
commons-logging-1.1.3.jar
cssparser-0.9.14.jar
guava-15.0.jar
httpclient-4.3.4.jar
httpcore-4.3.2.jar
httpmime-4.3.3.jar
jna-3.4.0.jar
phantomjsdriver-1.2.0.jar
selenium-api-2.41.0.jar
selenium-chrome-driver-2.44.0.jar
selenium-firefox-driver-2.44.0.jar
selenium-htmlunit-driver-2.44.0.jar
selenium-ie-driver-2.44.0.jar
selenium-java-2.44.0.jar
selenium-remote-driver-2.41.0.jar
selenium-safari-driver-2.44.0.jar
selenium-support-2.44.0.jar
```
版本兼容：


```
FireFox:
2.25.0   ->  18
2.30.0   ->  19
2.31.0   ->  20
2.42.2   ->  29
2.44.0   ->  33 (不支持31)
2.53.0   ->  43,46(不支持47)
2.41.0   ->  26(绿色版本)
2.44     ->  32.0-35.0
2.53.0-2.53.6 ->  40.0.3

Chrome:
2.29  ->  56-58
2.28  ->  55-57
2.27  ->  54-56
2.26  ->  53-55
2.25  ->  53-55
2.24  ->  52-54
2.23  ->  51-53
2.22  ->  49-52
2.21  ->  46-50
2.20  ->  43-48
2.19  ->  43-47
2.18  ->  43-46
2.17  ->  42-43
2.13  ->  42-45
2.15  ->  40-43
2.14  ->  39-42
2.13  ->  38-41
2.12  ->  36-40
2.11  ->  36-40
2.10  ->  33-36
2.9  ->  31-34
2.8  ->  30-33
2.7  ->  30-33
2.6  ->  29-32
2.5  ->  29-32
2.4  ->  29-32
```

具体实施：

1. Java 环境，IDE 工具 Eclipse or IDEA;
2. 新建 Java 工程，引入依赖包；
3. 使用 Seleium 新建一个浏览器：

```
String browserPath = "/Applications/Firefox.app/Contents/MacOS/firefox";
System.setProperty("webdriver.firefox.bin", browserPath);
WebDriver driver = new FirefoxDriver();
```

4. 为 Selenium 设置代理：

```
FireFox:

String proxyIp = "10.10.10.1";
int proxyPort = 80;
FirefoxProfile profile = new FirefoxProfile();
profile.setPreference("network.proxy.type", 1);
profile.setPreference("network.proxy.http", proxyIp);
profile.setPreference("network.proxy.http_port", proxyPort);
profile.setPreference("network.proxy.ssl", proxyIp);
profile.setPreference("network.proxy.ssl_port", proxyPort);
WebDriver driver = new FirefoxDriver(profile);
```

5. 打开一个 url 并获取对应的网页源码：

```
String url = "http://www.baidu.com";
driver.get(url);
String pageSource = driver.getPageSource();
System.out.println(pageSource);
```

6. 最大化浏览器：

```
driver.manage().window().maximize();
```

7. 定位元素实例：

```
根据ID定位元素:
    WebElement ele1 = driver.findElement(By.id("id"));
根据Class定位元素:
    WebElement ele2 = driver.findElement(By.className("className"));
根据xpath定位元素:
    WebElement ele3 = driver.findElement(By.xpath("xpath"));
```

8. 向表单填数据：

```
WebElement ele4 = driver.findElement(By.id("id"));
ele4.clear();
ele4.sendKeys("str");
```

9. Click

```
WebElement ele5 = driver.findElement(By.id("id"));
ele5.click();
```

常见问题：

1. 使用find_element_by_xxxx()方法查找元素时，如果元素找不到，不会返回null，而是抛出异常，所以需要自己捕获异常。
2. 使用firefox后，很多选项虽然在浏览器中进行了设置，但是通过selenium启动firefox后，设置并没有生效，所以这些设置你需要在代  
码中添加
3. 使用WebDriver点击Button元素时，如果Button元素其他元素遮住了，或没出现在界面中(比如Button在页面底部，但是屏幕只能显示页  
面上半部分)，使用Click()方法可能无法触发Click事件。


### Python

1. 声明浏览器对象：
> Python文件名或者包名不要命名为selenium，会导致无法导入

```
from selenium import webdriver
# webdriver可以认为是浏览器的驱动器，要驱动浏览器必须用到webdriver，支持多种浏览器，这里以Chrome为例
browser = webdriver.Chrome()
```

2. 访问页面并获取网页Html

```
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
print(browser.page_source)  #browser.page_source是获取网页的全部html
browser.close()
```

3. 查找元素

```
单个元素

from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input_first = browser.find_element_by_id('q')
input_second = browser.find_element_by_css_selector('#q')
input_third = browser.find_element_by_xpath('//*[@id="q"]')
print(input_first,input_second,input_third)
browser.close()

常用的查找方法

find_element_by_name
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector

也可以使用通用的方法

from selenium import webdriver
from selenium.webdriver.common.by import By
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input_first = browser.find_element(BY.ID,'q')#第一个参数传入名称，第二个传入具体的参数
print(input_first)
browser.close()

多个元素，elements多个s

input_first = browser.find_elements_by_id('q')
```

4. 元素交互操作，搜索框传入关键字进行自动搜索

```
from selenium import webdriver
import time
browser = webdriver.Chrome()
browser.get('https://www.taobao.com')
input = browser.find_element_by_id('q') #找到搜索框
input.send_keys('iPhone')#传送入关键词
time.sleep(5)
input.clear()   #清空搜索框
input.send_keys('男士内裤')
button = browser.find_element_by_class_name('btn-search')   #找到搜索按钮
button.click()

更多操作：
http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement
可以有属性、截图等等

```
5. 交互动作，驱动浏览器进行动作，模拟拖拽动作，将动作附加到动作链中串行执行

```
from selenium import webdriver
from selenium.webdriver import ActionChains#引入动作链

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult') #切换到iframeResult框架
source = browser.find_element_by_css_selector('#draggable') #找到被拖拽对象
target = browser.find_element_by_css_selector('#droppable') #找到目标
actions = ActionChains(browser) #声明actions对象
actions.drag_and_drop(source, target)
actions.perform()   #执行动作

更多操作:
http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains

```

6. 执行JavaScript
> 有些动作可能没有提供api，比如进度条下拉，这时，我们可以通过代码执行JavaScript

```
from selenium import webdriver
browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
browser.execute_script('window.scrollTo(0, document.body.scrollHeight)')
browser.execute_script('alert("To Bottom")')
```

7. 获取元素信息

```
获取属性
from selenium import webdriver
from selenium.webdriver import ActionChains

browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
logo = browser.find_element_by_id('zh-top-link-logo')#获取网站logo
print(logo)
print(logo.get_attribute('class'))
browser.close()

获取文本值
from selenium import webdriver
browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
input = browser.find_element_by_class_name('zu-top-add-question')
print(input.text)#input.text文本值
browser.close()

# 获取Id，位置，标签名，大小
from selenium import webdriver
browser = webdriver.Chrome()
url = 'https://www.zhihu.com/explore'
browser.get(url)
input = browser.find_element_by_class_name('zu-top-add-question')
print(input.id)#获取id
print(input.location)#获取位置
print(input.tag_name)#获取标签名
print(input.size)#获取大小
browser.close()
```

8. Frame操作
> frame相当于独立的网页，如果在父类网frame查找子类的，则必须切换到子类的frame，子类如果查找父类也需要先切换

```
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException

browser = webdriver.Chrome()
url = 'http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable'
browser.get(url)
browser.switch_to.frame('iframeResult')
source = browser.find_element_by_css_selector('#draggable')
print(source)
try:
    logo = browser.find_element_by_class_name('logo')
except NoSuchElementException:
    print('NO LOGO')
browser.switch_to.parent_frame()
logo = browser.find_element_by_class_name('logo')
print(logo)
print(logo.text)

```

9. 等待
> 隐式等待:
当使用了隐式等待执行测试的时候，如果 WebDriver没有在 DOM中找到元素，将继续等待，超出设定时间后则抛出找不到元素的异常,
换句话说，当查找元素或元素并没有立即出现的时候，隐式等待将等待一段时间再查找 DOM，默认的时间是0

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.implicitly_wait(10) #等待十秒加载不出来就会抛出异常，10秒内加载出来正常返回
browser.get('https://www.zhihu.com/explore')
input = browser.find_element_by_class_name('zu-top-add-question')
print(input)
```
> 显式等待:
指定一个等待条件，和一个最长等待时间，程序会判断在等待时间内条件是否满足，如果满足则返回，如果不满足会继续等待，超过时间就会抛出异常

```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

browser = webdriver.Chrome()
browser.get('https://www.taobao.com/')
wait = WebDriverWait(browser, 10)
input = wait.until(EC.presence_of_element_located((By.ID, 'q')))
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))
print(input, button)

title_is 标题是某内容
title_contains 标题包含某内容
presence_of_element_located 元素加载出，传入定位元组，如(By.ID, 'p')
visibility_of_element_located 元素可见，传入定位元组
visibility_of 可见，传入元素对象
presence_of_all_elements_located 所有元素加载出
text_to_be_present_in_element 某个元素文本包含某文字
text_to_be_present_in_element_value 某个元素值包含某文字
frame_to_be_available_and_switch_to_it frame加载并切换
invisibility_of_element_located 元素不可见
element_to_be_clickable 元素可点击
staleness_of 判断一个元素是否仍在DOM，可判断页面是否已经刷新
element_to_be_selected 元素可选择，传元素对象
element_located_to_be_selected 元素可选择，传入定位元组
element_selection_state_to_be 传入元素对象以及状态，相等返回True，否则返回False
element_located_selection_state_to_be 传入定位元组以及状态，相等返回True，否则返回False
alert_is_present 是否出现Alert

详细内容：
http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.support.expected_conditions
```

10. 前进后退-实现浏览器的前进后退以浏览不同的网页

```
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com/')
browser.get('https://www.taobao.com/')
browser.get('https://www.python.org/')
browser.back()
time.sleep(1)
browser.forward()
browser.close()
```

11. Cookies

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.zhihu.com/explore')
print(browser.get_cookies())
browser.add_cookie({'name': 'name', 'domain': 'www.zhihu.com', 'value': 'germey'})
print(browser.get_cookies())
browser.delete_all_cookies()
print(browser.get_cookies())

选项卡管理 增加浏览器窗口
import time
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.execute_script('window.open()')
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(1)
browser.switch_to_window(browser.window_handles[0])
browser.get('http://www.fishc.com')
```

12. 异常处理

```
from selenium import webdriver

browser = webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.find_element_by_id('hello')

from selenium import webdriver
from selenium.common.exceptions import TimeoutException, NoSuchElementException

browser = webdriver.Chrome()
try:
    browser.get('https://www.baidu.com')
except TimeoutException:
    print('Time Out')
try:
    browser.find_element_by_id('hello')
except NoSuchElementException:
    print('No Element')
finally:
    browser.close()
    
# 详细文档：
http://selenium-python.readthedocs.io/api.html#module-selenium.common.exceptions
```
