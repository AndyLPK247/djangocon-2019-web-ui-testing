# djangocon-2019-web-ui-testing
This repository contains the companion project for the
[Hands-On Web UI Testing](https://2019.djangocon.us/tutorials/hands-on-web-ui-testing/) tutorial
taught by [Pandy Knight](https://twitter.com/AutomationPanda)
at [DjangoCon 2019](https://2019.djangocon.us/).
If you are taking the tutorial,
then please clone this repository and follow the instructions below.
The slide deck for the tutorial is also in this repository.

## Project Setup

### System Prerequisites
You can complete this tutorial using any OS: Windows, macOS, Linux, etc.

This tutorial requires Python 3.6 or higher.
You can download the latest Python version from [Python.org](https://www.python.org/downloads/).

This tutorial also requires [pipenv](https://docs.pipenv.org/).
To install pipenv, run `pip install pipenv` from the command line.

You should also have a Python editor/IDE of your choice.
Good choices include [PyCharm](https://www.jetbrains.com/pycharm/)
and [Visual Studio Code](https://code.visualstudio.com/docs/languages/python).

For Web UI testing, you will need to install the latest versions of
[Google Chrome](https://www.google.com/chrome/)
and [Mozilla Firefox](https://www.mozilla.org/en-US/firefox/).
You can use other browsers with Selenium WebDriver,
but this tutorial uses Chrome and Firefox.
Make sure your browser versions are up to date.

You will also need the latest versions of the WebDriver executables for these browsers:

* [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/) for Chrome
* [geckodriver](https://github.com/mozilla/geckodriver/releases) for Firefox

These WebDriver executables will act as a proxy between our test automation code and browser instances.
Make sure their versions match the versions of the browsers, or else problems may happen.
The WebDriver executables must also be on your [system path](https://en.wikipedia.org/wiki/PATH_(variable)).
To verify, simply try to run them from the terminal, and then close them.
The test automation needs these executables to be on the system path
so that it can launch them when tests run.
Please ask if you need help with this configuration.

You will also need [Git](https://git-scm.com/) to copy this project code.
If you are new to Git, [try learning the basics](https://try.github.io/).

### Setup Instructions

1. Clone this repository: `git clone https://github.com/AndyLPK247/djangocon-2019-web-ui-testing.git`
2. Run `cd djangocon-2019-web-ui-testing` to enter the project.
3. Run `pipenv install` to install the dependencies.
4. Run `pipenv run python -m pytest` to verify that the framework can run tests.

### Branching

The `master` branch contains the code for the tutorial's starting point.
The project is basically empty in the `master` branch.

If you want to code along with the tutorial, then create a branch for your work off the `master` branch.
To create your own branch named `tutorial/develop`, run:

    > git checkout master -b tutorial/develop

The `example/*` branches contain the completed code for tutorial parts.
If you get stuck, you can always check the example code.

* [`example/1-first-test`](https://github.com/AndyLPK247/djangocon-2019-web-ui-testing/tree/example/1-first-test)
* `example/2-webdriver-setup`
* `example/3-page-objects`
* `example/4-locators`
* `example/5-webdriver-calls`
* `example/6-browser-config`
* `example/7-race-conditions`
* `example/8-parallel-testing`
* `example/develop` (main development branch for the examples)

## Tutorial Instructions

### Part 1: Writing Our First Test

*Time Estimate: 5 Minutes*

We should always write test *cases* before writing any test *code*.
Test cases exercise an application's behavior to verify goodness and identify badness.
Test code simply automates test cases.
Writing a test case first helps us form our thoughts well.

Consider the following test case:

```gherkin
Scenario: Basic DuckDuckGo Search
    Given the DuckDuckGo home page is displayed
    When the user searches for "panda"
    Then the search result title contains "panda"
    And the search result query is "panda"
    And the search result links pertain to "panda"
```

Let's implement this test using pytest.
Create a new file named `test_search.py` under the `tests` directory,
and add the following code:

```python
"""
These tests cover DuckDuckGo searches.
"""

def test_basic_duckduckgo_search():

    # Given the DuckDuckGo home page is displayed
    # TODO

    # When the user searches for "panda"
    # TODO

    # Then the search result title contains "panda"
    # TODO
    
    # And the search result query is "panda"
    # TODO
    
    # And the search result links pertain to "panda"
    # TODO

    raise NotImplementedError("Incomplete Test")
```

Adding comments to stub each step may seem trivial,
but it's a good first step when you write new test cases.
We can simply add code at each TODO line as we automate.
Once the test is completed, we will remove the exception at the end.
Also, note that pytest expects all test function names to begin with `test_`.

To avoid confusion when we run tests, let's remove the old placeholder test.
Delete `tests/test_fw.py`.

Rerun the tests using `pipenv run python -m pytest`.
The `test_basic_duckduckgo_search` should be the only test that runs,
and it should fail due to the "Incomplete Test" exception.

Finally, commit your code change. Part 1 is complete!

### Part 2: Setting Up Selenium WebDriver

*Time Estimate: 5 Minutes*

[Selenium WebDriver](https://www.seleniumhq.org/projects/webdriver/)
automates user interaction with Web browsers.
It works with several popular programming languages and browsers.

The Selenium WebDriver package for Python is named `selenium`.
Run `pipenv install selenium` to install it for our project.

Every test should use its own WebDriver instance.
This keeps things simple and safe.
The best way to set up the WebDriver instance is to use a
[pytest fixture](https://docs.pytest.org/en/latest/fixture.html).
Fixtures are basically setup and cleanup functions.
By convention, fixtures that multiple tests share are stored in [a file called `conftest.py`](https://docs.pytest.org/en/latest/fixture.html#conftest-py-sharing-fixture-functions).

Create a new file named `tests/conftest.py` and add the following code:

```python
"""
This module contains shared fixtures.
"""

import pytest
import selenium.webdriver


@pytest.fixture
def browser():

  # Initialize the ChromeDriver instance
  b = selenium.webdriver.Chrome()

  # Make its calls wait up to 10 seconds for elements to appear
  b.implicitly_wait(10)

  # Return the WebDriver instance for the setup
  yield b

  # Quit the WebDriver instance for the cleanup
  b.quit()
```

Our fixture uses Chrome as the browser.
Other browser types could be used.
Real-world projects often read browser choice from a config file here.

The implicit wait ensures that WebDriver calls wait for elements to appear before sending calls to them.
10 seconds should be reasonable for our test project's needs.
For larger projects, however, setting explicit waits is a better practice
because different calls need different wait times.
Read more about implicit versus explicit waits [here](https://selenium-python.readthedocs.io/waits.html).

The `yield` statement makes the `browser` fixture a generator.
The first iteration will do the "setup" steps,
while the second iteration will do the "cleanup" steps.
Always make sure to *quit* the WebDriver instance as part of cleanup,
or else zombie processes might lock system resources!

Now, update `test_basic_duckduckgo_search` in `tests/test_search.py` to call the new fixture:

```python
def test_basic_duckduckgo_search(browser):
  # ...
```

Whenever a pytest test function declares a fixture by name as an argument,
pytest will automatically call that fixture before the test runs.
Whatever the fixture returns will be passed into the test function.
Therefore, we can access the WebDriver instance using the `browser` variable!

Re-run the test using `pipenv run python -m pytest` to test the fixture.
Even though the test should still fail,
Chrome should briefly pop up while the test is running.
Make sure Chrome quits after the test is complete.
Then, commit your latest code changes.
Part 2 is now complete!

### Part 3: Defining Page Objects

*Time Estimate: 10 Minutes*

A **page object** represents a component of a Web page or the entire page itself.
They have *locators* for finding elements,
as well as *interaction methods* that interact with the page under test.
Page objects make low-level Selenium WebDriver calls
so that tests can make short, readable calls instead of complex ones.

Since we have our test steps, we know what pages and elements our test needs.
There are two pages under test, each with a few interactions:

1. The DuckDuckGo search page
   * Load the page
   * Search a phrase
2. The DuckDuckGo results page
   * Get the results
   * Get the search query
   * Get the title

Understanding interactions with the Web app is more important than the code.
We can write stubs for page object classes as we figure out the interactions.

Create a new Python package named `pages`.
To do this create a directory under the root directory named `pages`.
Then, put a blank file in it named `__init__.py`.
The `pages` directory should *not* be under the `tests` directory.
Why? When using pytest, the `tests` folder should *not* be a package.

Create a module named `pages/search.py` and add the following code
for the DuckDuckGo search page:

```python
"""
This module contains DuckDuckGoSearchPage,
the page object for the DuckDuckGo search page.
"""


class DuckDuckGoSearchPage:

  def __init__(self, browser):
    self.browser = browser

  def load(self):
    # TODO
    pass

  def search(self, phrase):
    # TODO
    pass
```

Create another module named `pages/result.py` and add the following code
for the DuckDuckGo result page:

```python
"""
This module contains DuckDuckGoResultPage,
the page object for the DuckDuckGo search result page.
"""


class DuckDuckGoResultPage:
  
  def __init__(self, browser):
    self.browser = browser

  def result_count_for_phrase(self, phrase):
    # TODO
    return 0
  
  def search_input_value(self):
    # TODO
    return ""

  def title(self):
    # TODO
    return ""
```

Every page object needs a reference to the WebDriver instance.
That's why the `__init__` methods take in and store a reference to `browser`.

Finally, update `test_basic_duckduckgo_search` in `tests/test_search.py`
with the following code:

```python
"""
These tests cover DuckDuckGo searches.
"""

from pages.result import DuckDuckGoResultPage
from pages.search import DuckDuckGoSearchPage


def test_basic_duckduckgo_search(browser):
  search_page = DuckDuckGoSearchPage(browser)
  result_page = DuckDuckGoResultPage(browser)
  PHRASE = "panda"
  
  # Given the DuckDuckGo home page is displayed
  search_page.load()

  # When the user searches for "panda"
  search_page.search(PHRASE)

  # Then the search result title contains "panda"
  assert PHRASE in result_page.title()
  
  # And the search result query is "panda"
  assert PHRASE == result_page.search_input_value()
  
  # And the search result links pertain to "panda"
  assert result_page.result_count_for_phrase(PHRASE) > 0

  # TODO: Remove this exception once the test is complete
  raise Exception("Incomplete Test")
```

Notice how we are able to write all the test steps using page object calls and assertions.
We also kept the step comments so the code is well-documented.
Even though we haven't made any Selenium WebDriver calls, our test case function is nearly complete!
Our code is readable and understandable.
It delivers clear testing value.

Rerun the test using `pipenv run python -m pytest`.
The test should fail again, but this time, it should fail on one of the assertions.
Then, commit your latest code changes. Part 3 is now complete!

### Part 4: Using Locators

*Time Estimate: 15 Minutes*

Every "thing" on a Web page is a Web *element*:
buttons, labels, dropdowns, input fields, etc.
Elements on the page are specified using HTML.
Tests use page objects to interact with elements.

Interactions typically require three steps:

1. Wait for the target element to appear on the page
2. Get an object representing the target element
3. Send commands to the element object

In our solution, waiting is handled automatically thanks to the browser fixture's `implicitly_wait` call.
Getting the element object, however, requires a locator.

*Locators* are query strings that use HTML attributes to find elements on a Web page.
There are many types of locators:

* ID
* Name
* Class name
* CSS Selector
* XPath
* Link test
* Partial link test
* Tag name

For example, if the page has the following element:

```html
<button id="django_ok">OK</button>
```

Then, an ID locator for "django_ok" could be used to get this element.

Locators are not element objects themselves but instead point to elements. 
The WebDriver instance will use locators to fetch and construct element objects.
Why are locators and elements separate concerns?
Elements on a page are always changing:
they may take time to load, or they may change with user interaction.
Locators, however, are always the same:
they simply specify how to get elements.
For example, a locator could be used to prove that an element does *not* exist.

For our test, we need locators for three elements:

1. The search input on the DuckDuckGo search page
2. The search input on the DuckDuckGo results page
3. The search phrase results on the DuckDuckGo results page

(Note: The page title is not a Web element. It can be fetched as a `browser` property.)

Writing good locators is a bit of an art.
Inspecting the HTML source of a live page makes it easy.
To do this, open the [DuckDuckGo search page](https://duckduckgo.com/) in Chrome.
Then, right-click the page and select "Inspect".
[Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/) will open.
The "Elements" tab shows the HTML source.
As you move the cursor over HTML elements in the source,
Chrome will highlight the elements on the page.
Now, click the icon with the box and cursor in the upper-left corner of the DevTools pane.
Move the cursor over elements on the page, and you will see them highlighted in the source.
Neat!

Try to find the search input element.
Its HTML should look like this:

```html
<input
  id="search_form_input_homepage"
  class="js-search-input search__input--adv"
  type="text"
  autocomplete="off"
  name="q"
  tabindex="1"
  value=""
  autocapitalize="off"
  autocorrect="off">
```

Notice that there is an `id` attribute set to "search_form_input_homepage".
Let's use this as our locator and update `pages/search.py`.

First, import `By` from the `selenium` package so we can write locators:

```python
from selenium.webdriver.common.by import By
```

Then, add the following attribute to the `DuckDuckGoSearchPage` class:

```python
SEARCH_INPUT = (By.ID, 'search_form_input_homepage')
```

`By` contains property keys for each type of locator.
We can write locators as tuples of the locator type and the query string.
(We will use this locator for interaction calls in the next part of the tutorial.)

The full code for `pages/search.py` should now look like this:

```python
"""
This module contains DuckDuckGoSearchPage,
the page object for the DuckDuckGo search page.
"""

from selenium.webdriver.common.by import By


class DuckDuckGoSearchPage:

  SEARCH_INPUT = (By.ID, 'search_form_input_homepage')

  def __init__(self, browser):
    self.browser = browser

  def load(self):
    # TODO
    pass

  def search(self, phrase):
    # TODO
    pass
```

Let's write locators for the `DuckDuckGoResultPage` next.
Perform a search, inspect the page, and try to come up with locators on your own!

Below is the code for `pages/result.py` with locators:

```python
"""
This module contains DuckDuckGoResultPage,
the page object for the DuckDuckGo search result page.
"""

from selenium.webdriver.common.by import By


class DuckDuckGoResultPage:
  
  SEARCH_INPUT = (By.ID, 'search_form_input')

  @staticmethod
  def PHRASE_RESULTS(phrase):
    xpath = f"//div[@id='links']//*[contains(text(), '{phrase}')]"
    return (By.XPATH, xpath)
  
  def __init__(self, browser):
    self.browser = browser

  def result_count_for_phrase(self, phrase):
    # TODO
    return 0
  
  def search_input_value(self):
    # TODO
    return ""

  def title(self):
    # TODO
    return ""
```

Thankfully, the search input element on the result page
uses the same locator as the one on the search page.
The locator for the search phrase results is a bit trickier, though.
It must find all result elements that contain the search phrase in their display texts.
This locator will return a list of elements, not just one.
It is written as a static method so that it can take in the phrase dynamically.
The XPath will return all elements under the "links" div that contain the text of the phrase.
The method returns a tuple so that it can be used like other locators.

We should always try to use the simplest locator that uniquely finds the target elements.
IDs, names, and class names are the easiest,
but sometimes, we must use CSS selectors and XPaths.
To learn more about writing good locators,
take the [Web Element Locator Strategies](https://testautomationu.applitools.com/web-element-locator-strategies/) course
from [Test Automation University](https://testautomationu.applitools.com/).

Although our test will still fail,
rerun it using `pipenv run python -m pytest` to make sure our changes did no harm.
Then, commit your latest code changes. Part 4 is now complete!

### Part 5: Making WebDriver Calls

*Time Estimate: 15 Minutes*

Now we can implement all the page object methods using WebDriver calls.
The [WebDriver API for Python](https://selenium-python.readthedocs.io/api.html)
documents all WebDriver calls.
If you aren't sure how to do something, look it up.
WebDriver can do anything a user can do on a Web page!

Let's start with `DuckDuckGoSearchPage`.
The `load` method is a one-line WebDriver call,
but it's good practice to make the URL a class variable:

```python
URL = 'https://www.duckduckgo.com'

def load(self):
  self.browser.get(self.URL)
```

The `search` method is a bit more complex because it interacts with an element.
We need to use a *locator* to find the search input element,
and then we need to *send keys* to type the search phrase into the element.

First, update the `selenium` package imports:

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
```

The `search` method needs two parts: finding the element and sending the keystrokes.
Thankfully, we already have the locator for the element.

```python
def search(self, phrase):
  search_input = self.browser.find_element(*self.SEARCH_INPUT)
  search_input.send_keys(phrase + Keys.RETURN)
```

The `find_element` method will return the first element found by the locator.
Notice how the locator uses the `*` operator to expand the SEARCH_INPUT locator tuple into arguments.
The `selenium` package offers specific locator type methods (like `find_element_by_name`),
but using the generic `find_element` method with argument expansion is better practice.
If the locator type must be changed due to Web page updates,
then the `find_element` call would not need to be changed.

The `send_keys` method sends the search phrase passed into the `search` method.
This means that the page object can search any phrase!
The addition of `Keys.RETURN` will send the ENTER/RETURN key as well,
which will submit the input value to perform the search and load the results page.

The full code for `pages/search.py` should look like this:

```python
"""
This module contains DuckDuckGoSearchPage,
the page object for the DuckDuckGo search page.
"""

from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys


class DuckDuckGoSearchPage:

  # URL

  URL = 'https://www.duckduckgo.com'

  # Locators

  SEARCH_INPUT = (By.ID, 'search_form_input_homepage')

  # Initializer

  def __init__(self, browser):
    self.browser = browser

  # Interaction Methods

  def load(self):
    self.browser.get(self.URL)

  def search(self, phrase):
    search_input = self.browser.find_element(*self.SEARCH_INPUT)
    search_input.send_keys(phrase + Keys.RETURN)
```

Now, let's do `DuckDuckGoResultPage`.
The `title` method is the easiest one because it just returns a property value:

```python
def title(self):
  return self.browser.title
```

The `search_input_value` method is similar to the `search` method from `DuckDuckGoSearchPage`,
but instead of sending a command, it asks for state from the page.
The "value" attribute contains the text a user types into an "input" element.

```python
def search_input_value(self):
  search_input = self.browser.find_element(*self.SEARCH_INPUT)
  return search_input.get_attribute('value')
```

The `result_count_for_phrase` method is a bit more complex.
The test must verify that the result page displays links relating to the search phrase.
This method should find all result links that contain the search phrase in their display texts.
Then, it should return the count.
Remember, the test asserts that the count is greater than zero.
This assertion may seem weak because it could allow some results to be unrelated to the phrase,
but it is good enough for the nature of a basic search test.
(A more rigorous test could and should more carefully cover search result goodness.)

The `result_count_for_phrase` method should look like this:

```python
def result_count_for_phrase(self, phrase):
  results = self.browser.find_elements(*self.PHRASE_RESULTS(phrase))
  return len(results)
```

Notice that it uses `find_elements` (plural) to get a list of matching elements.

The full code for `pages/result.py` should look like this:

```python
"""
This module contains DuckDuckGoResultPage,
the page object for the DuckDuckGo search result page.
"""

from selenium.webdriver.common.by import By


class DuckDuckGoResultPage:
  
  # Locators

  SEARCH_INPUT = (By.ID, 'search_form_input')

  @staticmethod
  def PHRASE_RESULTS(phrase):
    xpath = f"//div[@id='links']//*[contains(text(), '{phrase}')]"
    return (By.XPATH, xpath)

  # Initializer

  def __init__(self, browser):
    self.browser = browser

  # Interaction Methods

  def result_count_for_phrase(self, phrase):
    results = self.browser.find_elements(*self.PHRASE_RESULTS(phrase))
    return len(results)
  
  def search_input_value(self):
    search_input = self.browser.find_element(*self.SEARCH_INPUT)
    return search_input.get_attribute('value')

  def title(self):
    return self.browser.title
```

Finally, remove the "incomplete" exception from `tests/test_search.py`.
That module's code should look like this:

```python
"""
These tests cover DuckDuckGo searches.
"""

from pages.result import DuckDuckGoResultPage
from pages.search import DuckDuckGoSearchPage


def test_basic_duckduckgo_search(browser):
  search_page = DuckDuckGoSearchPage(browser)
  result_page = DuckDuckGoResultPage(browser)
  PHRASE = "panda"

  # Given the DuckDuckGo home page is displayed
  search_page.load()

  # When the user searches for "panda"
  search_page.search(PHRASE)

  # Then the search result title contains "panda"
  assert PHRASE in result_page.title()
  
  # And the search result query is "panda"
  assert PHRASE == result_page.search_input_value()
  
  # And the search result links pertain to "panda"
  assert result_page.result_count_for_phrase(PHRASE) > 0
```

Rerun the test using `pipenv run python -m pytest`.
Now, finally, it should run to completion and pass!
The test will take a few seconds to run because it must wait for page loads.
Chrome should pop up and automatically go through all test steps.
Try not to interfere with the browser as the test runs.
Make sure pytest doesn't report any failures when it completes.

### Part 6: Configuring Multiple Browsers

*Time Estimate: 15 Minutes*

Our test currently runs on Chrome,
but it should be able to run on other browsers, too.
Any Web UI test should be configurable to run on any compatible browser.
Let's run it on Firefox and Headless Chrome!

Browser choice should be treated as a configuration parameter.
It should not be hard-coded into automation code.
It should also not be written as pytest parameters.
One test session should use one browser.
If another browser needs to be tested, then launch another test session.
This design keeps test code and test executions simpler.

Create a new file named `config.json` in the project's root directory.
JSON files are very easy to use in Python.
The `json` module is part of the standard library,
and JSON files can be parsed into dictionaries with one line.
Add the following lines:

```json
{
  "browser": "Chrome",
  "implicit_wait": 10
}
```

Notice how these inputs correspond to values in the `browser` fixture.
Then, add a new fixture to `tests/conftest.py`:

```python
import json

@pytest.fixture
def config(scope='session'):

  # Read the file
  with open('config.json') as config_file:
    config = json.load(config_file)
  
  # Assert values are acceptable
  assert config['browser'] in ['Firefox', 'Chrome', 'Headless Chrome']
  assert isinstance(config['implicit_wait'], int)
  assert config['implicit_wait'] > 0

  # Return config so it can be used
  return config
```

This fixture reads the `config.json` file.
It also validates the inputs so that tests won't run if the inputs are bad.
The fixture's *scope* is set to "session" so that the fixture is called only one time for all tests.
There is no need to read it repeatedly for every test.

Update the `browser` fixture to use these inputs:

```python
@pytest.fixture
def browser(config):

  # Initialize the WebDriver instance
  if config['browser'] == 'Firefox':
    b = selenium.webdriver.Firefox()
  elif config['browser'] == 'Chrome':
    b = selenium.webdriver.Chrome()
  else:
    opts = selenium.webdriver.ChromeOptions()
    opts.add_argument('headless')
    b = selenium.webdriver.Chrome(options=opts)

  # Make its calls wait up to 10 seconds for elements to appear
  b.implicitly_wait(config['implicit_wait'])

  # Return the WebDriver instance for the setup
  yield b

  # Quit the WebDriver instance for the cleanup
  b.quit()
```

Fixtures can call fixtures.
Here, `browser` calls `config` and then uses its parts to set the browser and implicit wait time.
Notice that Headless Chrome just uses the Chrome WebDriver with extra arguments.

Nothing else needs to be updated in order to change the browser.
Run the test using `pipenv run python -m pytest` with Chrome to verify no harm was done.
You should see the test run successfully.

Then, change the config's *browser* to "Headless Chrome" and rerun the test.
You won't see the browser window appear, but the test should still pass.
Why? "Headless" mode won't render pages visibly.
It's great for automated testing because it's slightly more efficient than "regular" Chrome.

Finally, try "Firefox". Does it work? Warning: it may or may not! Oh no!
Don't panic if it doesn't work. We'll fix it in the next part.

### Part 7: Handling Race Conditions

*Time Estimate: 15 Minutes*

When running the search test using Firefox, you might hit the following failure:

```
      # Then the search result title contains "panda"
>     assert PHRASE in result_page.title()
E     AssertionError: assert 'panda' in 'DuckDuckGo — Privacy, simplified.'
```

Or, the test might pass.
Why would the test fail on Firefox if it passed for Chrome?
And why is there a chance that it *might* fail or *might* pass?
Let's revisit the test case steps:

```gherkin
Scenario: Basic DuckDuckGo Search
    Given the DuckDuckGo home page is displayed
    When the user searches for "panda"
    Then the search result title contains "panda"
    And the search result query is "panda"
    And the search result links pertain to "panda"
```

Step 2 performs the search.
Then, step 3 checks the title of the page.
Unfortunately, step 3 has a *race condition*.
Remember, the browser and the automation are two separate processes.
When the automation triggers the search, the browser will load the new page and title.
At the same time, the automation will continue to execute the test.
If the automation executes the assertion *before* the new page title loads,
then the assertion will fail.
Chrome was fast enough to avoid the race condition,
but Firefox was slow enough to trigger it.

Race conditions are the bane of Web UI testing.
They can be difficult to predict when writing tests.
They can also be difficult to identify in test results
because they typically happen *intermittently*.
Web UI tests gain a bad reputation for being "flaky"
whenever race conditions are not handled appropriately.

Automation must always wait for page components to be ready before interacting with them.
[Implicit waits](https://selenium-python.readthedocs.io/waits.html#implicit-waits)
work well for Web elements,
but they don't work for browser attributes like page title.
They are best for small projects.
[Explicit waits](https://selenium-python.readthedocs.io/waits.html#explicit-waits)
are more customizable, but they require more code.
They are typically the better option for large projects that need different times and conditions.
As a best practice, automation should use only one type of waiting.
Mixing implicit and explicit waits can have unexpected consequences.

Thankfully, there's a shortcut we can use to fix `test_basic_duckduckgo_search`.
The other two assertions use implicit waits for other elements on the page.
By the time those elements are loaded, the title would also be loaded.
Therefore, we can move step 3 to the end of the scenario to be the last thing we check.

The updated code for `tests/test_search.py` should be:

```python
"""
These tests cover DuckDuckGo searches.
"""

from pages.result import DuckDuckGoResultPage
from pages.search import DuckDuckGoSearchPage


def test_basic_duckduckgo_search(browser):
  search_page = DuckDuckGoSearchPage(browser)
  result_page = DuckDuckGoResultPage(browser)
  PHRASE = "panda"
  
  # Given the DuckDuckGo home page is displayed
  search_page.load()

  # When the user searches for "panda"
  search_page.search(PHRASE)

  # Then the search result query is "panda"
  assert PHRASE == result_page.search_input_value()
  
  # And the search result links pertain to "panda"
  assert result_page.result_count_for_phrase(PHRASE) > 0

  # And the search result title contains "panda"
  # (Putting this assertion last guarantees that the page title will be ready)
  assert PHRASE in result_page.title()
```

Rerun the test using `pipenv run python -m pytest` with Firefox to verify the fix.
Then, rerun it again with Chrome and Headless Chrome to make sure those browsers still work.

Always watch out for race conditions,
always wait for things to be ready before interacting with them,
and always run tests multiple times across multiple configurations to identify problems.

### Part 8: Running Tests in Parallel

*Time Estimate: 15 Minutes*

Unfortunately, Web UI tests are very slow compared to unit tests and service API tests.
The best way to speed them up is to run them in parallel.

First, let's parametrize `test_basic_duckduckgo_search` so that we have more than one test to run.
Any pytest test or fixture may be [parametrized](https://docs.pytest.org/en/latest/parametrize.html).
Update the code in `tests/test_search.py` to be:

```python
"""
These tests cover DuckDuckGo searches.
"""

import pytest

from pages.result import DuckDuckGoResultPage
from pages.search import DuckDuckGoSearchPage


@pytest.mark.parametrize('phrase', ['panda', 'python', 'platypus'])
def test_basic_duckduckgo_search(browser, phrase):
  search_page = DuckDuckGoSearchPage(browser)
  result_page = DuckDuckGoResultPage(browser)
  
  # Given the DuckDuckGo home page is displayed
  search_page.load()

  # When the user searches for "panda"
  search_page.search(phrase)

  # Then the search result query is "panda"
  assert phrase == result_page.search_input_value()
  
  # And the search result links pertain to "panda"
  assert result_page.result_count_for_phrase(phrase) > 0

  # And the search result title contains "panda"
  # (Putting this assertion last guarantees that the page title will be ready)
  assert phrase in result_page.title()
```

The test will now run three times with different search phrases.
Rerun the tests to make sure they all work.
You will notice that they run one at a time.

Next, install [pytest-xdist](https://docs.pytest.org/en/3.0.1/xdist.html),
the pytest plugin for parallel testing:

```bash
$ pipenv install pytest-xdist
```

Finally, run the tests using the following command:

```bash
$ pipenv run python -m pytest -n 3
```

The "-n 3" arguments tells pytest to run 3 tests in parallel.
We have 3 example tests, and most machines can handle 3 Web UI tests simultaneously.
When the tests run, notice how 3 browser instances open at once - one per test.

Run the tests a few times using Chrome and Firefox.
Look to see how long the tests typically take per browser.
Also, look to see if any intermittent failures happen.
Then, try using Headless Chrome.
Most likely, Headless Chrome will be significantly faster and more reliable
that regular Chrome and Firefox.

Whenever running tests in parallel,
carefully tune the number of threads to minimize the total test execution time.
More threads does *not* necessarily mean faster testing.
Too many parallel tests will choke system resources.

Anecdotally, for Web UI tests, I found:

* 1 test per processor minimizes total execution time without slowing down individual tests
* 2 tests per processor minimizes total execution time further but warps individual tests
* more than 2 tests per processor does not meaningfully shrink total execution time further
* memory size does not have a significant impact on total execution time

One machine can scale up only so far.
For massive parallel testing, try using
[Selenium Grid](https://github.com/SeleniumHQ/selenium/wiki/Grid2).
There are also many companies that provide cloud-based solutions for parallel WebDriver testing.
Check the *Resources* section below for a list.
Also, to learn more about parallel testing in general, read
[To Infinity and Beyond: A Guide to Parallel Testing](https://automationpanda.com/2018/01/21/to-infinity-and-beyond-a-guide-to-parallel-testing/).

Congrats! You have completed the guided part of this tutorial!

## Independent Exercises

The guided tutorial covered one very basic search test, but DuckDuckGo has many more features.
Write some new tests for DuckDuckGo.
Here are some suggestions:

* search for different phrases
* search by clicking the button instead of typing RETURN
* click a search result
* expand "More Results" at the bottom of the result page
* verify auto-complete suggestions pertain to the search text
* search by selecting an auto-complete suggestion
* search a new phrase from the results page
* do an image search
* do a video search
* do a news search
* change settings
* change region

These tests will require new page objects, locators, and interaction methods.
See how many tests you can automate on your own!
If you get stuck, ask for help.

## Additional Resources

This DjangoCon 2019 *Hands-On Web UI Testing* tutorial is related to other tutorials by Andrew Knight:

* PyOhio 2019: [Hands-On Web UI Testing](https://github.com/AndyLPK247/pyohio-2019-web-ui-testing)
* TestProject: [Web Testing Made Easy with Python, Pytest and Selenium WebDriver](https://blog.testproject.io/2019/07/09/open-source-test-automation-python-pytest-selenium-webdriver/)
* SmartBear: [Hands-On UI Testing with Python](https://automationpanda.com/2019/08/19/hands-on-ui-testing-with-python-smartbear-webinar/)

[Test Automation University](https://testautomationu.applitools.com/)
offers free online courses on several testing and automation topics.
All TAU courses are great, but the following lessons complement this tutorial especially well:

* [Web Element Locator Strategies](https://testautomationu.applitools.com/web-element-locator-strategies/) shows how to write good locators and use Chrome DevTools.
* [Behavior-Driven Python with pytest-bdd](https://testautomationu.applitools.com/behavior-driven-python-with-pytest-bdd/) shows how to use `pytest-bdd` to write BDD-style tests.
* [Setting a Foundation for Successful Test Automation](https://testautomationu.applitools.com/setting-a-foundation-for-successful-test-automation/) shows how to run a testing project the right way.

Many companies provide cloud-based solutions for parallel, multi-browser, multi-platform Web UI testing that work with WebDriver:

* [BrowserStack](https://www.browserstack.com/)
* [LambdaTest](https://www.lambdatest.com/)
* [Sauce Labs](https://saucelabs.com/)
* [SmartBear CrossBrowserTesting](https://crossbrowsertesting.com/)
* [TestProject](https://testproject.io/)

Other helpful links:

* [AutomationPanda.com](https://automationpanda.com/)
  * [Python](https://automationpanda.com/python/)
  * [Testing](https://automationpanda.com/testing/)
  * [Why Python is Great for Test Automation](https://automationpanda.com/2018/07/26/why-python-is-great-for-test-automation/)
  * [Web Element Locators for Test Automation](https://automationpanda.com/2019/01/15/web-element-locators-for-test-automation/)
  * [The Testing Pyramid](https://automationpanda.com/2018/08/01/the-testing-pyramid/)
  * [To Infinity and Beyond: A Guide to Parallel Testing](https://automationpanda.com/2018/01/21/to-infinity-and-beyond-a-guide-to-parallel-testing/)
* [Selenium with Python](https://selenium-python.readthedocs.io/)
  * [WebDriver API](https://selenium-python.readthedocs.io/api.html)
  * [Waits](https://selenium-python.readthedocs.io/waits.html)
  * [Locating Elements](https://selenium-python.readthedocs.io/locating-elements.html)
* [pytest.org](https://docs.pytest.org/)
* [Selenium Grid wiki](https://github.com/SeleniumHQ/selenium/wiki/Grid2)

## About the Author

This tutorial was written and delivered by **Andrew Knight** (aka *Pandy*), the "Automation Panda."
Andy is a Pythonista who specializes in testing and automation.

* Twitter: [@AutomationPanda](https://twitter.com/AutomationPanda)
* Blog: [AutomationPanda.com](https://automationpanda.com/)
* LinkedIn: [andrew-leland-knight](https://www.linkedin.com/in/andrew-leland-knight/)
