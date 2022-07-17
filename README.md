# Scrapy with selenium
Scrapy middleware to handle javascript pages using selenium==4.3.0.

_This was originally a fork of [scrapy-selenium](https://github.com/clemfromspace/scrapy-selenium) but I couldn't use the below command on it._
_Therefore this is actually a rip-off of said repo._
## Installation
```
$ pip install git+https://github.com/mrafee113/selenium_scrapy.git@v0.1.0
```
You should use **python>=3.9**. 
You will also need one of the Selenium [compatible browsers](http://www.seleniumhq.org/about/platforms.jsp).

## Configuration
1. Add the browser to use, the path to the driver executable, and the arguments to pass to the executable to the scrapy settings:
    ```python
    from shutil import which

    SELENIUM_DRIVER_NAME = 'firefox'
    SELENIUM_DRIVER_EXECUTABLE_PATH = which('geckodriver')
    SELENIUM_DRIVER_ARGUMENTS=['-headless']  # '--headless' if using chrome instead of firefox
    SELENIUM_DRIVER_PREFERENCES = {
        # (for firefox) to allow browser data to be cached among sessions.
        "browser.cahce.disk.parent_directory": '/tmp/firefox-data-directory',
   
        # template firefox preferences to customize downloading files.
        "browser.download.folderList": 2,
        "browser.download.manager.showWhenStarting": False,
        "browser.download.dir": '/tmp/firefox-download-directory',
        "browser.helperApps.neverAsk.saveToDisk": "application/x-gzip"
   }
   # WARNING: To support localstorage the github project pyselenium_localstorage was used.
   #  This project has its obvious limitations (read its code).
   #  So be careful for type conversion when setting and getting values.
   SELENIUM_DRIVER_LOCALSTORAGE_DATA = [
        ("https://python.org", {"key1": "value1"}),
        ("https://google.com", {"data": "information"})
   ]
   # or alternatively
   SELENIUM_DRIVER_LOCALSTORAGE_DATA = ("https://github.ccm", {"username": "code"})
    ```

Optionally, set the path to the browser executable:
    ```python
    SELENIUM_BROWSER_EXECUTABLE_PATH = which('firefox')
    ```

In order to use a remote Selenium driver, specify `SELENIUM_COMMAND_EXECUTOR` instead of `SELENIUM_DRIVER_EXECUTABLE_PATH`:
    ```python
    SELENIUM_COMMAND_EXECUTOR = 'http://localhost:4444/wd/hub'
    ```

2. Add the `SeleniumMiddleware` to the downloader middlewares:
    ```python
    DOWNLOADER_MIDDLEWARES = {
        'scrapy_selenium.SeleniumMiddleware': 800
    }
    ```
## Usage
Use the `scrapy_selenium.SeleniumRequest` instead of the scrapy built-in `Request` like below:
```python
from scrapy_selenium import SeleniumRequest

yield SeleniumRequest(url=url, callback=self.parse_result)
```
The request will be handled by selenium, and the request will have an additional `meta` key, named `driver` containing the selenium driver with the request processed.
```python
def parse_result(self, response):
    print(response.request.meta['driver'].title)
```
For more information about the available driver methods and attributes, refer to the [selenium python documentation](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webdriver)

The `selector` response attribute work as usual (but contains the html processed by the selenium driver).
```python
def parse_result(self, response):
    print(response.selector.xpath('//title/@text'))
```

### Additional arguments
The `scrapy_selenium.SeleniumRequest` accept 4 additional arguments:

#### `wait_time` / `wait_until`

When used, selenium will perform an [Explicit wait](http://selenium-python.readthedocs.io/waits.html#explicit-waits) before returning the response to the spider.
```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC

yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    wait_time=10,
    wait_until=EC.element_to_be_clickable((By.ID, 'someid'))
)
```

#### `screenshot`
When used, selenium will take a screenshot of the page and the binary data of the .png captured will be added to the response `meta`:
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    screenshot=True
)

def parse_result(self, response):
    with open('image.png', 'wb') as image_file:
        image_file.write(response.meta['screenshot'])
```

#### `script`
When used, selenium will execute custom JavaScript code.
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    script='window.scrollTo(0, document.body.scrollHeight);',
)
```
