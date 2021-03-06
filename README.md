# Selpy-Python

Selpy-Python is a Page Object Model (POM) framework for selenium automation with python `pytest`. This framework uses 'pytest-xdist' module to run multiple threads to run the tests at the same time depending on the number of available processor cores (2X of number of available cores).

This framework also uses _**[selpy](https://pypi.org/project/selpy/) custom built pypi module**_ to implement snapshot feature (which is available in jest framework). If there are any change in UI we can directly change this in the test data file during the test run with ease. Manual maintenance of test data can be reduced drastically with this feature.

More details of this module can be refered in [selpy](https://pypi.org/project/selpy/) or [repo](https://github.com/nareshnavinash/selpy)


[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Made with Python](https://img.shields.io/badge/Made%20with-Python-yellow.svg)](https://www.python.org/)
[![StackOverflow](http://img.shields.io/badge/Stack%20Overflow-Ask-blue.svg)]( https://stackoverflow.com/users/10505289/naresh-sekar )
[![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg)](CONTRIBUTING.md)
[![email me](https://img.shields.io/badge/Contact-Email-green.svg)](mailto:nareshnavinash@gmail.com)


![alt text](Library/selpy_framework.png)


## Supports
* Multiple browser automation
* Multi browser automation
* Allure reports
* Jenkins Integration
* Modes of run via CLI command
* Headless run
* Docker Execution
* Driver Screenshots
* Testdata driven tests
* Multi Thread run
* Snap
* Static code analyser
* Reports e-mailer using shovel

## Setup
* Clone this repository
* Navigate to the cloned folder
* To install the dependencies in MAC we use Homebrew version manager install (using)[https://brew.sh/]
* Once brew is installed install python by `brew install python3`
* To get additional dependencies of python like pip3, do `brew postinstall python3`
* Install the required packages needed for this framework using `pip3 install -r requirements.txt`

## To Run the tests
For a simple run of all the test files in normal mode, try

```
pytest
```

To run the tests in snap mode (to save the UI texts to the dynamic file)
```
snap=1 pytest
```
Once the changes are saved to the file run the tests with `pytest` to get the test running against the saved data. To verify this feature I intentionally added two locator texts which will be changing continuously.

To Run the tests in parallel mode or multi thread run for the available test files, try (To have parallel run you need to have atleast 2 tests inside your folder structure)

```
pytest -s -v -n=2
```

To Run the tests in parallel mode for the available test files along with browser specification, try

```
browser=chrome pytest -s -v -n=2
```

To Run the tests in parallel mode for the available test files in headless mode, try

```
headless=1 browser=chrome pytest -s -v -n=2
```

This will run the tests in headless mode

## To open allure results
Allure is a open source framework for reporting the test runs. To install allure in mac, use the following steps

```
brew cask install adoptopenjdk
brew install allure
```

To view the results for the test run, use

```
allure serve reports/allure
```

## Multiple Browser
Currently supports for Chrome, Firefox and Safari browser, but handled in such a way that framework can be easily configured to support multiple browsers. I used webdriver manager to resolve the driver-browser compatibility issues, use the same to add your designated browser (firefox, edge, ie, safari etc.,).

## Multi Browser
Initiate the driver class inside support package multiple times with different WebDriver objects. You can execute the actions in multiple browsers at the same time by executing actions against each driver object. Screenshots for all the drivers has been handled in the `conftest.py` file.

## Reports
For better illustration on the testcases, allure reports has been integrated. Allure reports can also be integrated with jenkins to get a dashboard view. Apart from allure, pytest's default reporting such as html file has been added to the `reports/` folder.

## Jenkins Integration with Docker images
Get any of the linux with python docker image as the slaves in jenkins and use the same for executing the UI automation with this framework (Sample docker image - `https://hub.docker.com/_/python`). From the jenkins bash Execute the following to get the testcases to run,

```
#!/usr/bin/python3
python --version
cd <path_to_the_project>
pip3 install -r requirements.txt
headless=1 pytest -s -v -n 4
```

In Jenkins pipeline, try to add the following snippet to execute the tests,

```
pipeline {
    agent { docker { image 'python:3.7.6' } }
    stages {
        stage('test') {
            steps {
                sh 'python --version'
                sh 'cd project/'
                sh 'pip3 install -r requirements.txt'
                sh 'headless=1 pytest -s -v -n 4'
            }
        }
    }
}
```

## Headless Run
In `Data/GlobalData/global_data.yml` file, if the headless is `1`, the chrome will be initialized in headless mode which can be used to run in server. Screenshots will be added even if the browser runs in headless mode.

## Break down into end to end tests

### Adding Locators to the project

1. Add Locators to the that are going to be used inside the project inside the `Locators` folder.

2. Import `from selpy.locator import Locator` inside each locator file to use the ```Locator``` method from ```selpy``` module.

3. For each page add a new class and declare the locators. Static locators can be class variables. Dynamic locators can be separate methods. 

```
class AmazonHomePageLocator:
    amazon_logo = Locator("css selector", "div#nav-logo a[aria-label='Amazon']")
    amazon_search_categories = Locator("css selector", "div.nav-search-scope select.nav-search-dropdown")

    def __init__(self):
        print("Locators for Amazon home page")

    @staticmethod
    def amazon_search_category_list(string):
        return Locator("xpath", "//select[contains(@class,'nav-search-dropdown')]//option[text()='%s']" % string)
```

4. To use the Locator method we need to pass the type of locator and the actual locator element. Type of locator has to be mentioned in the following way to allow `selpy` to process the locator.

```
CSS - 'css selector'
XPATH - 'xpath'
ID - 'id'
NAME - 'name'
LINK TEXT - 'link text'
PARTIAL LINK TEXT - 'partial link text'
TAG NAME - 'tag name'
CLASS NAME - 'class name'
``` 

5. Ideally each web page should have a new file inside locators folder (with the same name as the web page) and all the locators inside a web page has to be declared inside a page class(Class name also should be same as the web page name).
* If the web page name is `home page` then the locator file name should be `home_page.rb` inside `locators` folder and the class name should be `HomePageLocator` inside `Locators` module.

### Adding page methods to the project

1. Add page specific methods inside the `Pages` folder.

2. Import the corresponding locator method inside the page file. This is to use the locators inside the page methods seamlessly. Ideal way is to import only one locator file inside the page file.

3. For each page add a new class and each page class should inherit the locators class of the same page

```
from Locators.amazon_home_page import AmazonHomePageLocator


class AmazonHomePage(AmazonHomePageLocator):

    def __init__(self):
        super().__init__()

    @classmethod
    def is_home_page_displayed(cls):
        return AmazonHomePageLocator.amazon_logo.is_displayed_with_wait()
```

4. Ideally each web page should have a new page file inside `pages` folder with the class name same as the web page name.
* If the web page name is `home page` then the pages file name should be `home_page.rb` inside `pages` folder and the class name should be `HomePage` inside `Pages` module.

### Creating a new test file in the project

1. Define the tests inside the Tests folder. Create a new `.py` file and import the required modules inside (depending on the requirement). Mainly require the page modules inside the test file. It is not recommended to import locator modules since we can access the locators from the page module.

```
import allure
import pytest
import time
from selpy.driver import Driver
from Pages.amazon_home_page import AmazonHomePage
from Pages.amazon_search_result import AmazonSearchResultPage
from Pages.amazon_product_page import AmazonProductPage
from selpy.variable import Var
```

2. It is suggested to mention the allure feature name, severity, pytest's markers to the test. This allows us to have better reporting and dynamic way to run in the future.

```
@allure.feature("Feature name")
@allure.severity('Critical')
@pytest.mark.regression
@pytest.mark.ui
def test_amazon_book_search_001():
    with allure.step("Initialize the UI dynamic data"):
        ui_dynamic_data = {}
    with allure.step("Set the test data file needed for this test run"):
        static_variable = Var("amazon.yml", "static")
        dynamic_variable = Var("amazon_book_search_result_dynamic.yml", "dynamic")
```

To run the test with marker you can execute as

```
pytest -v -m regression 
# or
pytest -v -m ui
```

Use `allure.step("step name")` to have a detailed reporting in allure.

3. Append the method name for the test as `test_` only then it will be taken as a test case. This has been configured in `pytest.ini` as,

```
markers =
    sanity: sanity tests marker
    regression: regression tests marker
    ui: UI tests marker
    api: API tess marker
python_files=*.py
python_functions=test_*
addopts = -rsxX
          -q
          -v
          --self-contained-html
          --html=reports/html_report.html
          --cov=Tests
          --alluredir reports/allure
          --clean-alluredir
```

I have created markers to have distinguished marker for automation purpose. The `python_funtions` param is where we need to mention the test files. `addopts` param used to take the values that are used to give in command line along with pytest.

Allure configurations and pytest's default report has been wired here.

4. A file `conftest.py` should be created inside the Tests folder. In this file we can have the run before each, run before each module, run during and after pytest setup methods. Adding screenshot to the testcases is handled by,

```
@pytest.fixture(autouse=True)
def before_each():
    print('*-* Before each INITIALIZATION')
    try:
        yield
        for driver in Store.drivers:
            root_dir = os.path.dirname(os.path.abspath(__file__)).replace("/Tests", "")
            name = 'img%s.png' % datetime.datetime.now().strftime("%Y%m%d-%H%M%S")
            config_path = root_dir + '/reports/screenshots/' + name
            driver.save_screenshot(config_path)
            allure.attach.file(config_path, name=name, attachment_type=allure.attachment_type.PNG)
    except Exception as e:
        print(e)
    print('*-* After each END')
```

The fixture param `autouse=True` ensures that this block is invoked only once for each test method. 

Closing of all the drivers has been handled like,

```
@pytest.fixture(scope='module', autouse=True)
def before_module():
    print('*-* Before module INITIALIZATION')
    yield
    for driver in Store.drivers:
        driver.quit()
    print('*-* After module END')
```

The param `scope='module'`ensures that this block is invoked only once for each test file.

5. We used home grown pypi published module `selpy` for Page Object Model support as well as snap support. To use that module data files path has to be set, this is done by,

```
from selpy.store import Store
def pytest_configure(config):
    Store.global_data_path = os.path.dirname(os.path.abspath(__file__)).replace("/Tests", "") + '/Data/GlobalData/global_data.yml'
    Store.static_data_path = os.path.dirname(os.path.abspath(__file__)).replace("/Tests", "") + '/Data/TestData/'
    Store.dynamic_data_path = os.path.dirname(os.path.abspath(__file__)).replace("/Tests", "") + '/Data/DynamicData/'
```

This ensures that this data has been set before pytest is being invoked only once. More about `selpy` module can be seen at [pypi page](https://pypi.org/project/selpy/)

6. Assert using pytest's default assertion method. Make sure you have a proper description to the assertion, so that once it is failed the failure message is proper.

```
assert (AmazonHomePage.is_home_page_displayed() is True), "Amazon home page is not displayed"
```

### Data sets:

In order to have distinguished set of data I have used three types of data.

* **Global** - Global configuration for the whole project. Here mode of run, browsers to use, browser configurations etc., are specified.
* **Test Data** - This is to store the module level data. Ideally for each test file we need to have a test data file, but that depends on the requirement.
* **Dynamic Data** - This is to store the dynamic data. Files in this folder are supposed to change when we run with `snap=1 pytest`. This is separated from the other data files so that other static files are not disturbed during the run.

## Detailing snap Mode:

Snap mode is created to reduce the burden in maintaining the test data. While we test an application with data heavy validations, there is a high chance that the data in the application may change over the course of time.

For example: If you are testing an application which tracks nasdaq or any other market related stuff, the data will change on daily basis. We need to have a capability to embrace to the change that is happening in the application with ease.

To use the snap mode, you need to use `selpy` [selpy](https://pypi.org/project/selpy/) module, and make use of `Var` methods as follows.

1. Declare the dictionary where the UI texts/data is to be stored during the test run.

```
ui_dynamic_data = {}
ui_dynamic_data["amazon_product_title"] = AmazonProductPage.amazon_product_title.texts_as_string()
```

2. Initiate a class variable with the file name against which you need to verify the UI data.

```
dynamic_variable = Var("amazon_book_search_result_dynamic.yml", "dynamic")
``` 

3. To compare the UI data with the file use the `compare` method that comes along with the `dynamic_variable`

```
dynamic_variable.compare(ui_dynamic_data)
```

This will compare and report it to allure and assertion has been done within that method.

4. While running the suite for first time where no data is saved within the test file, run the suite with `snap=1 pytest` this will ensure the UI data is being saved to the file.

5. Use only YML files for this purpose, since it is easier to handle key value pair with yaml file.

## Static code analyser:

For static code analyser I used flake8. To check the configurations view (.flake8)[.flake8] file. To check on the code status execte,

```
flake8
```

currently there are `0` vulnerabilities with this project.

## Shovel

Shovel has been added to mail the report status once the run is completed. This will be useful once we integrate with CI and helpful in debugging which commit has caused the actual problem. For more details you could visit the [Shovel file](Shovel/reports.py). (Code refactoring is yet to be done in that file). In Jenkins you could get the following values as environment variables.

```
$BUILD_URL #Directly fetched from ENV variable
$GIT_URL #Directly fetched from ENV variable
$GIT_BRANCH #Directly fetched from ENV variable
$GIT_COMMIT #Directly fetched from ENV variable
$GIT_COMMITER_EMAIL=$(git --no-pager show -s --format='%ae' $GIT_COMMIT)
$REPORTS_EMAIL_RECEIVERS="nareshnavinash@gmail.com;nareshsekar@zoho.com" #Or can be taken from parameterized build

#Run the rake task by,
shovel reports.email $REPORTS_EMAIL_RECEIVERS $BUILD_URL $GIT_BRANCH $GIT_COMMIT $GIT_COMMITER_EMAIL
```

## Built With

* [pytest](https://docs.pytest.org/en/latest/) - Core test framework
* [flake8](https://pypi.org/project/flake8/) - Static code analyser
* [pytest-xdist](https://pypi.org/project/pytest-xdist/) - To run pytest in parallel mode
* [selpy](https://pypi.org/project/selpy/) - Page Object Model for python
* [Allure pytest](https://pypi.org/project/allure-pytest/) - For Detailed reporting
* [Selenium](https://www.seleniumhq.org/) - For web browser automation.

## Contributing

1. Clone the repo!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Create a pull request.

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on code of conduct, and the process for submitting pull requests.

## Authors

* **[Naresh Sekar](https://github.com/nareshnavinash)**

## License

This project is licensed under the GNU GPL-3.0 License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

* To all the open source contributors whose code has been referred in this project.
