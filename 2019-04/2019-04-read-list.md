# 【2019/04】UI Automation Test Tutorial 

[TOC]

## 1. 环境部署

### 1.1 Appium 环境部署

**命令行安装:**

```shell
npm install -g appium
```

**dmg安装：**

[Appium-1.6.3 download link](https://github.com/appium/appium-desktop/releases/tag/v1.6.3)

> Note: 这里放置各个平台的Release版本，本篇教程时为1.6.3，但是最新的是1.12.1，所有有一定延迟。

### 1.2 Python

python 通常都是内置的，但是也可以用 brew 安装最新版本

```shell
brew install python
```

安装 `Appium-Python-Client`

```shell
pip install Appium-Python-Client
# or
pip3 install Appium-Python-Client
```

<details>
  <summary>点击展开安装成功的日志</summary>
```shell
Collecting Appium-Python-Client
  Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1056)'))': /simple/appium-python-client/
  Downloading https://files.pythonhosted.org/packages/26/f1/f932791ec73be6e13539fb201f6923305b8e67b2b47078fd2efc3ad4f865/Appium-Python-Client-0.40.tar.gz (41kB)
    100% |████████████████████████████████| 51kB 43kB/s 
Collecting selenium>=3.14.1 (from Appium-Python-Client)
  Downloading https://files.pythonhosted.org/packages/80/d6/4294f0b4bce4de0abf13e17190289f9d0613b0a44e5dd6a7f5ca98459853/selenium-3.141.0-py2.py3-none-any.whl (904kB)
    100% |████████████████████████████████| 911kB 28kB/s 
Requirement already satisfied: urllib3 in /usr/local/lib/python3.7/site-packages (from selenium>=3.14.1->Appium-Python-Client) (1.24.1)
Building wheels for collected packages: Appium-Python-Client
  Building wheel for Appium-Python-Client (setup.py) ... done
  Stored in directory: /Users/pmst/Library/Caches/pip/wheels/3e/33/3e/c5c3e241611d77f8a4c723213c10b3e49553766bc1d6734f64
Successfully built Appium-Python-Client
Installing collected packages: selenium, Appium-Python-Client
Successfully installed Appium-Python-Client-0.40 selenium-3.141.0
```
</details>

接着安装 `pytest` —— 一个为测试而生的框架，起码能格式化测试日志信息。

```shell
pip install -U pytest
```

<details>
	<summary>点击展开安装成功的日志</summary>
  ```shell
    Downloading https://files.pythonhosted.org/packages/7e/16/83b2a35c427b838df9836c9e7e4ae6dfbcbdea643db44652f693b1c57d70/pytest-4.4.0-py2.py3-none-any.whl (223kB)
    100% |████████████████████████████████| 225kB 9.6kB/s 
Collecting py>=1.5.0 (from pytest)
  Downloading https://files.pythonhosted.org/packages/76/bc/394ad449851729244a97857ee14d7cba61ddb268dce3db538ba2f2ba1f0f/py-1.8.0-py2.py3-none-any.whl (83kB)
    100% |████████████████████████████████| 92kB 6.9kB/s 
Collecting atomicwrites>=1.0 (from pytest)
  Downloading https://files.pythonhosted.org/packages/52/90/6155aa926f43f2b2a22b01be7241be3bfd1ceaf7d0b3267213e8127d41f4/atomicwrites-1.3.0-py2.py3-none-any.whl
Collecting six>=1.10.0 (from pytest)
  Downloading https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
Requirement already satisfied, skipping upgrade: setuptools in /usr/local/lib/python3.7/site-packages (from pytest) (40.8.0)
Collecting pluggy>=0.9 (from pytest)
  Downloading https://files.pythonhosted.org/packages/84/e8/4ddac125b5a0e84ea6ffc93cfccf1e7ee1924e88f53c64e98227f0af2a5f/pluggy-0.9.0-py2.py3-none-any.whl
Collecting attrs>=17.4.0 (from pytest)
  Downloading https://files.pythonhosted.org/packages/23/96/d828354fa2dbdf216eaa7b7de0db692f12c234f7ef888cc14980ef40d1d2/attrs-19.1.0-py2.py3-none-any.whl
Collecting more-itertools>=4.0.0; python_version > "2.7" (from pytest)
  Downloading https://files.pythonhosted.org/packages/b3/73/64fb5922b745fc1daee8a2880d907d2a70d9c7bb71eea86fcb9445daab5e/more_itertools-7.0.0-py3-none-any.whl (53kB)
    100% |████████████████████████████████| 61kB 18kB/s 
Installing collected packages: py, atomicwrites, six, pluggy, attrs, more-itertools, pytest
Successfully installed atomicwrites-1.3.0 attrs-19.1.0 more-itertools-7.0.0 pluggy-0.9.0 py-1.8.0 pytest-4.4.0 six-1.12.0
  ```
</details>
安装 Carthage 为了唤起 WebDriveAgent ?

```shell
brew install carthage
```

> TODO: 待补充



## 2. CI Step1: Build Project for product(app/ipa format)

编译工程，生成app文件

```shell
xcodebuild -sdk iphonesimulator12.2

# /Users/pmst/Downloads/appium-ios-basic-master/final/build/Release-iphonesimulator/AppiumTest.app
```



## 3. CI Step2: Python用例编写

[AppiumTest 工程配套的 python 用例](https://www.appcoda.com/automated-ui-testing-appium/)

```python
import unittest
import os
from random import randint
from appium import webdriver
from time import sleep
from selenium.webdriver.common.keys import Keys

class LoginTests(unittest.TestCase):

    def setUp(self):
        app = ('/Users/pmst/Downloads/appium-ios-basic-master/final/build/Release-iphonesimulator/AppiumTest.app')
        self.driver = webdriver.Remote(
            command_executor='http://127.0.0.1:4723/wd/hub',
            desired_capabilities={
                'app': app,
                'platformName': 'iOS',
                'platformVersion': '12.2',
                'deviceName': 'iPhone 6'
            }
        )

    def tearDown(self):
        self.driver.quit()

    def testEmailField(self):
        emailTF = self.driver.find_element_by_accessibility_id('emailTextField')
        emailTF.send_keys("validEmail")
        emailTF.send_keys(Keys.RETURN)
        sleep(1)
        self.assertEqual(emailTF.get_attribute("value"), "validEmail")

    def testPasswordField(self):
        passwordTF = self.driver.find_element_by_accessibility_id('passwordTextField')
        passwordTF.send_keys("validPW")
        passwordTF.send_keys(Keys.RETURN)
        sleep(1)
        self.assertNotEqual(passwordTF.get_attribute("value"), "validPW")

    def testLogin(self):
        self.testEmailField()
        self.testPasswordField()
        self.driver.find_element_by_accessibility_id('loginButton').click()
        sleep(1)
        smiley = self.driver.find_element_by_accessibility_id('smileyImage')
        self.assertTrue(smiley.get_attribute('wdVisible'))


if __name__ == '__main__':
    suite = unittest.TestLoader().loadTestsFromTestCase(LoginTests)
    unittest.TextTestRunner(verbosity=2).run(suite)
```

`setup` 函数中创建了一个 `webdriver`，用于连接 `appium` 开启的后台服务(**http://127.0.0.1:4723**)，通过http协议传递"测试命令”，我们的测试用例本质就是一条条测试命令，比如“点击一个按钮”，“往TextFiled框输入文字”等等，因此appium就像一个测试员等待着我们的指令，而他作为执行者在真机/模拟器操作，然后再将操作结果通过http协议返回给我们，此处的信息交换方式采用约定好的 JSON 格式，(Note:**而非将python脚本直接发送给appium后台执行**)。

既然约定了命令提供方和appium测试方的信息交换为JSON格式，为此命令提供方编写用例的语言是什么就无所谓了，可以是 `*C#, .NET, Java, node, Perl, PHP, Python, Ruby*` 等等，只要它们最后输出的一串串约定好的JSON格式的命令就可以了。

```python
self.driver = webdriver.Remote(
            command_executor='http://127.0.0.1:4723/wd/hub',
            desired_capabilities={
                'app': app,
                'platformName': 'iOS',
                'platformVersion': '12.2',
                'deviceName': 'iPhone 6'
            }
        )
```

这种方式非常简单，实例化一个 driver 需要提供必要的接口信息，如上述展示。之后可能还有其他方式，待补充。



## 4. CI Step3: 运行测试用例

一、命令行开启 Appium 服务器(or 直接点击桌面版应用程序的run)

```shell
pmst:final pmst$ appium 
[Appium] Welcome to Appium v1.12.1
[Appium] Appium REST http interface listener started on 0.0.0.0:4723
```

二、使用 `pytest` 运行脚本

```shell
pytest test_login.py
```

测试结果很顺利，所有以 `test` 前缀的都会作为一个case跑一遍，因此上面要跑三个case；另外键盘的输入使用 ` emailTF.send_keys` 方式，尝试改为中文，发现弹出的英文键盘也没关系，依然神速般填充了内容。