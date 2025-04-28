# Web Scraping with Pydoll

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to use Pydoll for scraping JavaScript-heavy websites, bypass Cloudflare, and scale with rotating proxies like Bright Data.

- [An Introduction to Pydoll](#an-introduction-to-pydoll)
- [Using Pydoll for Web Scraping: Complete Tutorial](#using-pydoll-for-web-scraping-complete-tutorial)
- [Bypassing Cloudflare With Pydoll](#bypassing-cloudflare-with-pydoll)
- [Limitations of This Approach to Web Scraping](#limitations-of-this-approach-to-web-scraping)
- [Integrating Pydoll with Bright Data's Rotating Proxies](#integrating-pydoll-with-bright-datas-rotating-proxies)
- [Alternatives to Pydoll for Web Scraping](#alternatives-to-pydoll-for-web-scraping)

## An Introduction to Pydoll

This guide only explains the basics of Pydoll. You can learn more about its functionality and what makes it stand out as a Python web scraping library in [this blog post](https://brightdata.com/blog/web-data/python-web-scraping-libraries).

### What It Is

[Pydoll](https://autoscrape-labs.github.io/pydoll/) is a Python browser automation library built for web scraping, testing, and automating repetitive tasks. What sets it apart is that it eliminates the need for traditional web drivers. In detail, it connects directly to browsers through the DevTools Protocol—no external dependencies required.

### Features

- **Zero webdrivers**: No browser driver dependency for easier setup and fewer version issues.
- **Async-first**: Fully asynchronous, built on `asyncio` for high concurrency and efficiency.
- **Human-like interactions**: Realistic typing, mouse movements, and clicks to evade bot detection.
- **Event-driven**: React to browser, DOM, network, and lifecycle events in real-time.
- **Multi-browser support**: Works with Chrome, Edge, and other Chromium browsers via a unified API.
- **Screenshot & PDF export**: Capture pages or elements, and generate high-quality PDFs.
- **Native Cloudflare bypass**: Bypass Cloudflare without third-party tools (given good IP reputation).
- **Concurrent scraping**: Scrape multiple pages/sites in parallel to speed up tasks.
- **Advanced keyboard control**: Simulate real typing with control over keys and timing.
- **Powerful event system**: Monitor and handle network and page events dynamically.
- **File upload support**: Automate uploads via inputs or file chooser dialogs.
- **Proxy integration**: Rotate IPs and geotarget using proxies.
- **Request interception**: Intercept, modify, or block HTTP requests and responses.

Learn more in the [official documentation](https://autoscrape-labs.github.io/pydoll/features/).


## Using Pydoll for Web Scraping: Complete Tutorial

In this section, you'll discover how to utilize Pydoll to extract data from the asynchronous, JavaScript-powered version of "[Quotes to Scrape](https://quotes.toscrape.com/js-delayed/?delay=2000)":

![The target site loading the data after 2 seconds](https://media.brightdata.com/2025/04/The-target-site-loading-the-data-after-2-seconds.gif)

This webpage dynamically renders quote elements using JavaScript after a brief delay. Consequently, conventional scraping tools won't function properly. To extract content from this page, you need a browser automation solution like Pydoll.

### Step #1: Project Setup

Before you start, ensure you have Python 3+ installed on your system. If not, [download it](https://www.python.org/downloads/) and follow the installation guide.

Next, run this command to create a directory for your scraping project:

```sh
mkdir pydoll-scraper
```

The `pydoll-scraper` folder will serve as your project directory.

Navigate to the folder in your terminal and initialize a Python [virtual environment](https://docs.python.org/3/library/venv.html) within it:

```sh
cd pydoll-scraper
python -m venv venv
```

Open the project folder in your preferred Python IDE. [Visual Studio Code with the Python extension](https://code.visualstudio.com/docs/languages/python) or [PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/#section=windows) are excellent choices.

Create a `scraper.py` file in the project folder, which should now contain:

![The project file structure for web scraping with Pydoll](https://media.brightdata.com/2025/04/The-project-file-structure-for-web-scraping-with-Pydoll.png)

At this point, `scraper.py` is just an empty Python script, but it will soon contain the [data parsing logic](https://brightdata.com/blog/web-data/what-is-data-parsing).

Next, activate the virtual environment in your IDE's terminal. On Linux or macOS, execute:

```sh
source venv/bin/activate
```

Similarly, on Windows, run:

```sh
venv/Scripts/activate
```

Great! Your Python environment is now configured for web scraping with Pydoll.

### Step #2: Set Up Pydoll

In your activated virtual environment, install Pydoll through the [`pydoll-python`](https://pypi.org/project/pydoll-python/) package:

```sh
pip install pydoll-python
```

Now, add the following code to the `scraper.py` file to begin using Pydoll:

```python
import asyncio
from pydoll.browser.chrome import Chrome

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # scraping logic...

# Execute the async scraping function
asyncio.run(main())
```

Note that Pydoll provides an asynchronous API for web scraping and requires the use of Python's [`asyncio`](https://docs.python.org/3/library/asyncio.html) standard library.

### Step #3: Connect to the Target Site

Invoke the [`go_to()`](https://autoscrape-labs.github.io/pydoll/deep-dive/page-domain/#navigation-system) method available through the `page` object to browse to the target website:

```python
await page.go_to("https://quotes.toscrape.com/js-delayed/?delay=2000")
```

The `?delay=2000` query parameter instructs the page to load the desired data dynamically after a 2-second delay. This is a feature of the target sandbox site, designed to help test dynamic scraping behavior.

Now, try executing the above script. If everything works correctly, Pydoll will:

1.  Launch a Chrome instance
2.  Navigate to the target site
3.  Close the browser window immediately—since there's no additional logic in the script yet

This is what you should briefly see before it closes:

![The target page being loaded by the Chrome instance controlled by Pydoll](https://media.brightdata.com/2025/04/The-target-page-being-loaded-by-the-Chrome-instance-controlled-by-Pydoll.png)

### Step #4: Wait for the HTML Elements to Appear

Examine the last image from the previous step. It shows the content of the page controlled by Pydoll in the Chrome instance. You'll notice it's completely empty—no data has loaded yet.

This occurs because the target site dynamically renders data after a 2-second delay. While this delay is specific to the example site, waiting for page elements to render is a common requirement when [scraping SPAs (single-page applications)](https://hackernoon.com/how-to-scrape-modern-spas-pwas-and-ai-driven-dynamic-sites) and other dynamic websites that depend on AJAX.

Learn more in our article about [scraping dynamic websites with Python](https://brightdata.com/blog/how-tos/scrape-dynamic-websites-python).

To handle this common scenario, Pydoll offers [built-in waiting mechanisms](https://autoscrape-labs.github.io/pydoll/deep-dive/find-elements-mixin/#waiting-mechanisms) via this method:

- `wait_element()`: Waits for a single element to appear (with timeout support)

This method supports CSS selectors, XPath expressions, and more—similar to how [Selenium's `By` object works](https://brightdata.com/blog/how-tos/using-selenium-for-web-scraping).

Let's study the HTML structure of the target page. Open it in your browser, wait for the quotes to load, right-click one of the quotes, and select the "Inspect" option:

![The HTML of the quote elements](https://media.brightdata.com/2025/04/The-HTML-of-the-quote-elements.png)

In the DevTools panel, you'll observe that each quote is wrapped in a `<div>` with the class `quote`. This means you can target them using the CSS selector:

```python
.quote
```

Now, use Pydoll to wait for these elements to appear before proceeding:

```python
await page.wait_element(By.CSS_SELECTOR, ".quote", timeout=3)
```

Don't forget to import `By`:

```python
from pydoll.constants import By
```

Run the script again, and this time you'll notice that Pydoll waits for the quote elements to load before closing the browser.

### Step #5: Prepare for Web Scraping

Remember, the target page contains multiple quotes. Since you want to extract all of them, you need a data structure to store this information. A simple array works perfectly, so initialize one:

```python
quotes = []
```

To [locate elements on the page](https://autoscrape-labs.github.io/pydoll/deep-dive/find-elements-mixin/#findelementsmixin-architecture), Pydoll provides two useful methods:

- `find_element()`: Locates the first matching element
- `find_elements()`: Locates all matching elements

Just like with `wait_element()`, these methods accept a selector using the `By` object.

So, select all quote elements on the page with:

```python
quote_elements = await page.find_elements(By.CSS_SELECTOR, ".quote")
```

Next, iterate through the elements and prepare to apply your scraping logic:

```python
for quote_element in quote_elements:
  # Scraping logic...
```

### Step #6: Implement the Data Parsing Logic

Begin by examining a single quote element:

![The HTML of a quote element](https://media.brightdata.com/2025/04/The-HTML-of-a-quote-element.png)

As evident from the HTML above, each quote element contains:

- The text quote in a `.text` node
- The author in the `.author` element
- A list of tags in the `.tag` elements

Implement the scraping logic to select these elements and extract the relevant data:

```python
# Extract the quote text (and remove curly quotes)
text_element = await quote_element.find_element(By.CSS_SELECTOR, ".text")
text = (await text_element.get_element_text()).replace(""", "").replace(""", "")

# Extract the author name
author_element = await quote_element.find_element(By.CSS_SELECTOR, ".author")
author = await author_element.get_element_text()

# Extract all associated tags
tag_elements = await quote_element.find_elements(By.CSS_SELECTOR, ".tag")
tags = [await tag_element.get_element_text() for tag_element in tag_elements]
```

**Note**: The [`replace()`](https://docs.python.org/3/library/stdtypes.html#str.replace) method removes the unnecessary curly double quotes from the extracted quote text.

Now, use the scraped data to create a new dictionary object and add it to the `quotes` array:

```python
# Populate a new quote with the scraped data
quote = {
    "text": text,
    "author": author,
    "tags": tags
}
# Append the extracted quote to the list
quotes.append(quote)
```

### Step #7: Export to CSV

Currently, the scraped data resides in a Python list. Make it easier to share and analyze by exporting it to a human-readable format like CSV.

Use Python to generate a new file named `quotes.csv` and populate it with the extracted data:

```python
with open("quotes.csv", "w", newline="", encoding="utf-8") as csvfile:
            # Add the header
            fieldnames = ["text", "author", "tags"]
            writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

            # Populate the output file with the scraped data
            writer.writeheader()
            for quote in quotes:
                writer.writerow(quote)
```

Remember to import [`csv`](https://docs.python.org/3/library/csv.html) from the Python Standard Library:

```python
import csv
```

### Step #8: Put It All Together

The complete `scraper.py` file should now contain:

```python
import asyncio
from pydoll.browser.chrome import Chrome
from pydoll.constants import By
import csv

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # Navigate to the target page
        await page.go_to("https://quotes.toscrape.com/js-delayed/?delay=2000")

        # Wait up to 3 seconds for the quote elements to appear
        await page.wait_element(By.CSS_SELECTOR, ".quote", timeout=3)

        # Where to store the scraped data
        quotes = []

        # Select all quote elements
        quote_elements = await page.find_elements(By.CSS_SELECTOR, ".quote")

        # Iterate over them and scrape data from them
        for quote_element in quote_elements:
            # Extract the quote text (and remove curly quotes)
            text_element = await quote_element.find_element(By.CSS_SELECTOR, ".text")
            text = (await text_element.get_element_text()).replace(""", "").replace(""", "")

            # Extract the author
            author_element = await quote_element.find_element(By.CSS_SELECTOR, ".author")
            author = await author_element.get_element_text()

            # Extract all tags
            tag_elements = await quote_element.find_elements(By.CSS_SELECTOR, ".tag")
            tags = [await tag_element.get_element_text() for tag_element in tag_elements]

            # Populate a new quote with the scraped data
            quote = {
                "text": text,
                "author": author,
                "tags": tags
            }
            # Append the extracted quote to the list
            quotes.append(quote)

    # Export the scraped data to CSV
    with open("quotes.csv", "w", newline="", encoding="utf-8") as csvfile:
                # Add the header
                fieldnames = ["text", "author", "tags"]
                writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

                # Populate the output file with the scraped data
                writer.writeheader()
                for quote in quotes:
                    writer.writerow(quote)

# Execute the async scraping function
asyncio.run(main())
```

Test the script by executing:

```sh
python scraper.py
```

Once completed, a `quotes.csv` file will appear in your project folder.

![The output data in quotes.csv](https://media.brightdata.com/2025/04/The-output-data-in-the-quotes-csv.png)

## Bypassing Cloudflare With Pydoll

When interacting with websites through browser automation tools, one of the [major challenges](https://brightdata.com/blog/web-data/web-scraping-challenges) you'll encounter is [web application firewalls (WAFs)](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/), such as Cloudflare.

When your requests are identified as coming from an automated browser, these systems often display a CAPTCHA. In certain cases, they present it to all visitors during their initial visit to the site.

[Bypassing CAPTCHAs in Python](https://brightdata.com/blog/web-data/bypass-captchas-with-python) is challenging. However, techniques exist to convince Cloudflare you're a legitimate user, preventing CAPTCHA challenges from appearing initially. Pydoll addresses this by providing a dedicated API for exactly this purpose.

To demonstrate this functionality, we'll use the "[Antibot Challenge](https://www.scrapingcourse.com/antibot-challenge)" test page from the ScrapingCourse website:

![Automatic Cloudflare verification on the target page](https://media.brightdata.com/2025/04/Automatic-Cloudflare-verification-on-the-target-page.gif)

As shown, the page consistently performs the [Cloudflare JavaScript Challenge](https://hackernoon.com/bypassing-javascript-challenges-for-effective-web-scraping). After bypassing it, sample content appears confirming that the anti-bot protection has been defeated.

Pydoll offers two approaches for handling Cloudflare:

1. [Context manager approach](https://autoscrape-labs.github.io/pydoll/features/#context-manager-approach-synchronous): Manages the anti-bot challenge synchronously, pausing script execution until the challenge is resolved.
2. [Background processing approach](https://autoscrape-labs.github.io/pydoll/features/#background-processing-approach): Handles the anti-bot asynchronously in the background.

We'll cover both methods. However, as mentioned in the official documentation, be aware that Cloudflare bypassing isn't guaranteed. Factors like IP reputation or browsing history can affect success.

For more sophisticated techniques, check our comprehensive tutorial on [scraping Cloudflare-protected sites](https://brightdata.com/blog/web-data/bypass-cloudflare-for-web-scraping).

### Context Manager Approach

To let Pydoll automatically handle the Cloudflare anti-bot challenge, use the `expect_and_bypass_cloudflare_captcha()` method like this:

```python
import asyncio
from pydoll.browser.chrome import Chrome
from pydoll.constants import By

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # Wait for the Cloudflare challenge to be executed
        async with page.expect_and_bypass_cloudflare_captcha():
            # Connect to the Cloudflare-protected page:
            await page.go_to("https://www.scrapingcourse.com/antibot-challenge")
            print("Waiting for Cloudflare anti-bot to be handled...")

        # This code runs only after the anti-bot is successfully bypassed
        print("Cloudflare anti-bot bypassed! Continuing with automation...")

        # Print the text message on the success page
        await page.wait_element(By.CSS_SELECTOR, "#challenge-title", timeout=3)
        success_element = await page.find_element(By.CSS_SELECTOR, "#challenge-title")
        success_text = await success_element.get_element_text()
        print(success_text)

asyncio.run(main())
```

When you execute this script, the Chrome window will automatically overcome the challenge and load the target page.

The output will be:

```
Waiting for Cloudflare anti-bot to be handled...
Cloudflare anti-bot bypassed! Continuing with automation...
You bypassed the Antibot challenge! :D
```

### Background Processing Approach

If you prefer not to halt script execution while Pydoll addresses the Cloudflare challenge, you can utilize the `enable_auto_solve_cloudflare_captcha()` and `disable_auto_solve_cloudflare_captcha()` methods like this:

```python
import asyncio
from pydoll.browser import Chrome
from pydoll.constants import By

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # Enable automatic captcha solving before navigating
        await page.enable_auto_solve_cloudflare_captcha()

        # Connect to the Cloudflare-protected page:
        await page.go_to("https://www.scrapingcourse.com/antibot-challenge")
        print("Page loaded, Cloudflare anti-bot will be handled in the background...")

        # Disable anti-bot auto-solving when no longer needed
        await page.disable_auto_solve_cloudflare_captcha()

        # Print the text message on the success page
        await page.wait_element(By.CSS_SELECTOR, "#challenge-title", timeout=3)
        success_element = await page.find_element(By.CSS_SELECTOR, "#challenge-title")
        success_text = await success_element.get_element_text()
        print(success_text)

asyncio.run(main())
```

This approach enables your scraper to perform other operations while Pydoll resolves the Cloudflare anti-bot challenge in the background.

This time, the output will be:

```
Page loaded, Cloudflare anti-bot will be handled in the background...
You bypassed the Antibot challenge! :D
```

## Limitations of This Approach to Web Scraping

With Pydoll—or any [scraping tool](https://brightdata.com/blog/web-data/best-web-scraping-tools)—sending too many requests can likely result in being blocked by the target server. This happens because most websites implement rate limiting to prevent bots (like your scraping script) from overwhelming their servers with excessive requests.

This represents a [standard anti-scraping](https://brightdata.com/blog/web-data/anti-scraping-techniques) and anti-DDoS strategy. Understandably, website owners want to protect their sites from automated traffic floods.

Even when following best practices such as [respecting `robots.txt`](https://brightdata.com/blog/how-tos/robots-txt-for-web-scraping-guide), making numerous requests from a single IP address can still trigger suspicion. Consequently, you might encounter [`403 Forbidden`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/403) or [`429 Too Many Requests`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/429) errors.

The most effective solution is rotating your IP address using a web proxy.

For those unfamiliar with this concept, a [web proxy](https://brightdata.com/blog/proxy-101/what-is-proxy-server) functions as an intermediary between your scraper and the target website. It forwards your requests and returns responses, making it appear to the target site that traffic originates from the proxy—not your actual device.

This technique not only helps conceal your real IP but also assists in bypassing geo-restrictions and [numerous other applications](https://brightdata.com/use-cases).

Various [proxy types exist](https://brightdata.com/blog/proxy-101/ultimate-guide-to-proxy-types). To avoid blocking, you need a premium provider offering authentic rotating proxies like Bright Data.

In the following section, you'll learn how to combine [Bright Data's rotating proxies](https://brightdata.com/solutions/rotating-proxies) with Pydoll for more effective web scraping—particularly at scale.

## Integrating Pydoll with Bright Data's Rotating Proxies

Let's implement Bright Data's residential proxies with Pydoll.

If you don't have an account yet, [register for Bright Data](https://brightdata.com/cp/start). Otherwise, proceed and sign in to access your dashboard:

![The Bright Data dashboard](https://media.brightdata.com/2025/04/The-Bright-Data-dashboard-1.png)

From the dashboard, select the "Get proxy products" button:

![Get proxy products](https://media.brightdata.com/2025/04/Clicking-the-Get-proxy-products-button.png)

You'll be directed to the "Proxies & Scraping Infrastructure" page:

![The "Proxies & Scraping Infrastructure" page](https://media.brightdata.com/2025/04/The-Proxies-Scraping-Infrastructure-page-1.png)

In the table, locate the "Residential" row and click it:

![Clicking the "residential" row](https://media.brightdata.com/2025/04/Clicking-the-residential-row.png)

You'll arrive at the residential proxy configuration page:

![The "residential" page](https://media.brightdata.com/2025/04/The-residential-page.png)

For first-time users, follow the setup wizard to configure the proxy according to your requirements.

Navigate to the "Overview" tab and find your proxy's host, port, username, and password:

![The proxy credentials](https://media.brightdata.com/2025/04/The-proxy-credentials.png)

Utilize those details to construct your proxy URL:

```
proxy_url = "<brightdata_proxy_username>:<brightdata_proxy_password>@<brightdata_proxy_host>:<brightdata_proxy_port>";
```

Replace the placeholders (`<brightdata_proxy_username>`, `<brightdata_proxy_password>`, `<brightdata_proxy_host>`, `<brightdata_proxy_port>`) with your actual proxy credentials.

Ensure you activate the proxy product by switching the toggle from "Off" to "On":

![Clicking the activation toggle](https://media.brightdata.com/2025/04/Clicking-the-activation-toggle.png)

With your proxy configured, here's how to incorporate it into Pydoll using its [built-in proxy configuration capabilities](https://autoscrape-labs.github.io/pydoll/features/#proxy-integration):

```python
import asyncio
from pydoll.browser.chrome import Chrome
from pydoll.browser.options import Options
from pydoll.constants import By
import traceback

async def main():
    # Create browser options
    options = Options()

    # The URL of your Bright Data proxy
    proxy_url = "<brightdata_proxy_username>:<brightdata_proxy_password>@<brightdata_proxy_host>:<brightdata_proxy_port>" # Replace it with your proxy URL

    # Configure the proxy integration option
    options.add_argument(f"--proxy-server={proxy_url}")

    # To avoid potential SSL errors
    options.add_argument("--ignore-certificate-errors")

    # Start browser with proxy configuration
    async with Chrome(options=options) as browser:
        await browser.start()
        page = await browser.get_page()

        # Visit a special page that returns the IP of the caller
        await page.go_to("https://httpbin.io/ip")

        # Extract the page content containing only the IP of the incoming
        # request and print it
        body_element = await page.find_element(By.CSS_SELECTOR, "body")
        body_text = await body_element.get_element_text()
        print(f"Current IP address: {body_text}")

# Execute the async scraping function
asyncio.run(main())
```

Each time you run this script, you'll observe a different exit IP address, thanks to Bright Data's proxy rotation.

> **Note**:
> 
> Typically, Chrome's `--proxy-server` flag doesn't support authenticated proxies directly. However, [Pydoll's advanced proxy manager](https://autoscrape-labs.github.io/pydoll/deep-dive/browser-domain/#proxy-manager) overcomes this limitation, allowing password-protected proxy server usage.

## Alternatives to Pydoll for Web Scraping

While Pydoll is certainly a powerful web scraping library, especially for browser automation with built-in anti-bot circumvention features, other valuable tools exist.

Here are several strong Pydoll alternatives worth exploring:

- [**SeleniumBase**](https://brightdata.com/blog/web-data/web-scraping-with-seleniumbase): A Python framework built upon Selenium/WebDriver APIs, providing a professional-grade toolkit for web automation. It supports everything from end-to-end testing to sophisticated scraping workflows.
- [**Undetected ChromeDriver**](https://brightdata.com/blog/web-data/web-scraping-with-undetected-chromedriver): A modified version of ChromeDriver engineered to avoid detection by popular anti-bot services like Imperva, DataDome, and Distil Networks. Ideal for stealthy scraping when using Selenium.

## Conclusion

Using Pydoll without IP rotation mechanisms can lead to inconsistent results. To make web scraping reliable and scalable, try Bright Data's [proxy networks](https://brightdata.com/proxy-types) that include datacenter proxies, residential proxies, ISP proxies, and mobile proxies.

Create an account and begin testing our proxies at no cost today!