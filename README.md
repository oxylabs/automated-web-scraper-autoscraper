[![Oxylabs promo code](https://raw.githubusercontent.com/oxylabs/product-integrations/refs/heads/master/Affiliate-Universal-1090x275.png)](https://oxylabs.go2cloud.org/aff_c?offer_id=7&aff_id=877&url_id=112)

# Automated Web Scraper With Python AutoScraper

This tutorial will show you how to automate your web scraping processes using AutoScaper – one of the several Python web scraping libraries available. 

Check out a more detailed tutorial on [our blog](https://oxylabs.io/blog/automated-web-scraper-autoscraper).

* [Methods to install AutoScraper](#methods-to-install-autoscraper)
  * [Scraping products with AutoScraper](#scraping-products-with-autoscraper)
    + [Scraping product category URLs](#scraping-product-category-urls)
    + [Scraping product information from a single webpage](#scraping-product-information-from-a-single-webpage)
    + [Scraping all the products on a specific category](#scraping-all-the-products-on-a-specific-category)
  * [How to use AutoScraper with proxies](#how-to-use-autoscraper-with-proxies)
  * [Saving and loading an AutoScraper model](#saving-and-loading-an-autoscraper-model)

## Methods to install AutoScraper

First things first, let’s install the AutoScraper library. There are actually several ways to install and use this library, but for this tutorial, we’re going to use the Python package index (PyPI) repository using the following pip command:

```pip install autoscraper```

## Scraping products with AutoScraper

This section showcases an example to auto scrape public data with the AutoScraper module in Python using the [Oxylabs Scraping Sandbox](https://sandbox.oxylabs.io/products?_gl=1*10z7xfg*_gcl_au*NzUxMjYzNDQ4LjE3MjY1NzMwNjc.) website as a target.

The target website has three thousand products in different categories. 

### Scraping product category URLs

Now, if you want to scrape the links to the category pages, you can do it with the following trivial code:

```
from autoscraper import AutoScraper

UrlToScrape = "https://sandbox.oxylabs.io/products"

WantedList = [
    "https://sandbox.oxylabs.io/products/category/nintendo",
    "https://sandbox.oxylabs.io/products/category/dreamcast"
]

Scraper = AutoScraper()
data = Scraper.build(UrlToScrape, wanted_list=WantedList)
print(data)
```

Note that the Oxylabs Sandbox uses JavaScript to load some elements dynamically, such as the category buttons. Since AutoScraper doesn’t support JavaScript rendering, you won’t be able to scrape all category links. For instance, you can access the “Xbox platform” category but not the subcategories inside it.

With that in mind, the code above first imports AutoScraper from the autoscraper library. Then, we provide the URL from which we want to scrape the information in the ```UrlToScrape```. 

The ```WantedList``` assigns sample data that we want to scrape from the given subject URL. To get the category page links from the target page, you need to provide two example URLs to the ```WantedList```. One link is a data sample of a JavaScript-rendered category button, while another link is a data sample of a static category button that doesn’t have any subcategories. Try running the code with only one category URL in the ```WantedList``` to see the difference.

The AutoScraper() creates an AutoScraper object to initiate different functions of the autoscraper library. The Scraper.build() method scrapes the data similar to the ```WantedList``` from the target URL.

After executing the Python script above, the ```data``` list will have the category page links available at https://sandbox.oxylabs.io/products. The output of the script should look like this: 

```['https://sandbox.oxylabs.io/products/category/nintendo', 'https://sandbox.oxylabs.io/products/category/xbox-platform', 'https://sandbox.oxylabs.io/products/category/playstation-platform', 'https://sandbox.oxylabs.io/products/category/dreamcast', 'https://sandbox.oxylabs.io/products/category/pc', 'https://sandbox.oxylabs.io/products/category/stadia']```

### Scraping product information from a single webpage

Say that we want to get the title of the product along with its price; we can train and build an AutoScraper model as follows:

```
from autoscraper import AutoScraper

UrlToScrape = "https://sandbox.oxylabs.io/products/3"
WantedList = ["Super Mario Galaxy 2", "91,99 €"]
 
InfoScraper = AutoScraper()
InfoScraper.build(UrlToScrape, wanted_list=WantedList)
```

The script above feeds a URL of a product page and a sample of required information from that page to the AutoScraper model. The ```build()``` method learns the rules for scraping information and preparing our ```InfoScraper``` for future use.

Now, let’s apply this ```InfoScraper``` tactic to a different product’s URL and see if it returns the desired information.

```
another_product_url = "https://sandbox.oxylabs.io/products/39"

data = InfoScraper.get_result_similar(another_product_url)
print(data)
```

Output:

```['Super Mario 64', '91,99 €']```

The script above applies ```InfoScraper``` to ```another_product_url``` and prints the ```data```. Depending on the target website you want to scrape, you may want to use the ```get_result_exact()``` function instead of ```get_result_similar()```. This should ensure that AutoScraper returns an accurate product title and price as defined by the ```WantedList```.

Additionally, it’s very important to provide a ```UrlToScrape``` that doesn’t have duplicate data that may match some unwanted elements. Consider this example:

```
from autoscraper import AutoScraper

UrlToScrape = "https://sandbox.oxylabs.io/products/1"
WantedList = ["The Legend of Zelda: Ocarina of Time", "91,99 €"]
 
InfoScraper = AutoScraper()
InfoScraper.build(UrlToScrape, wanted_list=WantedList)

another_product_url = "https://sandbox.oxylabs.io/products/39"

data = InfoScraper.get_result_exact(another_product_url)
print(data)
```

Here, the ```UrlToScrape``` has the price of ```91,99 €``` twice on the page, as highlighted in the screenshot:

![](screenshot_1.png) 

Hence, the code also matches the unwanted element with the price of ```91,99 €``` and additionally returns the price of a related product like this:

```['Super Mario 64', '87,99 €', '91,99 €']```

One way to solve this problem is to use the ```grouped=True``` parameter to return the data points with their corresponding AutoScraper rule names. Next, use the ```keep_rules()``` function and pass the rules you want to keep. In our code, we have to turn the data ```dictionary``` into a ```list``` to access and pass over the first and the last rule, containing accurate product title and price:

```
from autoscraper import AutoScraper

UrlToScrape = "https://sandbox.oxylabs.io/products/1"
WantedList = ["The Legend of Zelda: Ocarina of Time", "91,99 €"]
 
InfoScraper = AutoScraper()
InfoScraper.build(UrlToScrape, wanted_list=WantedList)

another_product_url = "https://sandbox.oxylabs.io/products/39"

data = InfoScraper.get_result_exact(another_product_url, grouped=True)
print(data)
print()

InfoScraper.keep_rules([list(data)[0], list(data)[-1]])
filtered_data = InfoScraper.get_result_exact(another_product_url)
print(filtered_data)
```

Please note that this method may return incorrect results if the actual price of the product isn’t the last item returned by the ```data``` object.

### Scraping all the products on a specific category

Install the [pandas](https://pypi.org/project/pandas/) and [openpyxl](https://pypi.org/project/openpyxl/) libraries via the terminal, which we’ll use to save the data to an Excel file:

```pip install pandas openpyxl```

Then, use the following Python script:

```
#ProductsByCategoryScraper.py
from autoscraper import AutoScraper
import pandas as pd
 
#ProductUrlScraper section
Playstation_5_Category = "https://sandbox.oxylabs.io/products/category/playstation-platform/playstation-5"
WantedList = ["https://sandbox.oxylabs.io/products/246"]
Product_Url_Scraper = AutoScraper()
Product_Url_Scraper.build(Playstation_5_Category, wanted_list=WantedList)
 
#ProductInfoScraper section
Product_Page_Url = "https://sandbox.oxylabs.io/products/246" 
WantedList = ["Ratchet & Clank: Rift Apart", "87,99 €"]
 
Product_Info_Scraper = AutoScraper()
Product_Info_Scraper.build(Product_Page_Url, wanted_list=WantedList)
 
#Scraping info of each product and storing into an Excel file
Products_Url_List = Product_Url_Scraper.get_result_similar(Playstation_5_Category) 
Products_Info_List = []
for Url in Products_Url_List:
  product_info = Product_Info_Scraper.get_result_exact(Url)
  Products_Info_List.append(product_info)
df = pd.DataFrame(Products_Info_List, columns =["Title", "Price"])
df.to_excel("products_playstation_5.xlsx", index=False)
```

The script above has three main constituents: two sections for building the scrapers and the third one to scrape data from all the products in the Playstation 5 category and save it as an Excel file. 

For this step, we’ve built ```Product_Url_Scraper``` to scrape all the similar product links on the [Playstation 5 Category](https://sandbox.oxylabs.io/products/category/playstation-platform/playstation-5?_gl=1*1frsqtt*_gcl_au*NzUxMjYzNDQ4LjE3MjY1NzMwNjc.) page. These thirteen links are stored in the ```Products_Url_List```. Now, for each URL in the ```Products_Url_List```, we apply the ```Product_Info_Scraper``` and append the scraped information to the ```Products_Info_List```. Finally, the ```Products_Info_List```  is converted to a data frame and then exported as an Excel file for future use.

Output:

![](screenshot_2.png) 

The output reflects achieving the initial goal – scraping titles and prices of all the products in the Playstation 5 category. 

Now, we know how to use a combination of multiple AutoScraper models to scrape data in bulk. You can re-formulate the script above to scrape all the products from all the categories and save them in different Excel files for each category. 

## How to use AutoScraper with proxies

The ```build```, ```get_result_similar```, and ```get_result_exact``` functions of AutoScraper accept request-related arguments in the request_args parameter. 

Here’s what testing and using AutoScraper with proxy IPs looks like:

```
from autoscraper import AutoScraper

UrlToScrape = "https://ip.oxylabs.io/"
WantedList = ["YOUR_REAL_IP_ADDRESS"]

proxy = {
    "http": "proxy_endpoint",
    "https": "proxy_endpoint",
}
 
InfoScraper = AutoScraper()
InfoScraper.build(UrlToScrape, wanted_list=WantedList)

data = InfoScraper.get_result_similar(UrlToScrape, request_args={"proxies": proxy})
print(data)
```

Visit https://ip.oxylabs.io/, copy the displayed IP address, and paste it instead of YOUR_REAL_IP_ADDRESS in the ```WantedList```. This information will be used to tell AutoScraper what kind of data to look for.

The proxy_endpoint refers to the address of a [proxy server](https://oxylabs.io/products/paid-proxy-servers) in the correct format (e.g., ```http://customer-USERNAME:PASSWORD@pr.oxylabs.io:7777```). The script above should work fine when proper proxy endpoints are added to the proxy dictionary. Every time you run the code, it outputs a proxy IP address that should be different from your actual IP. 

After you’ve successfully tested the proxy server connection, you can then use the [proxies](https://oxylabs.io/products/residential-proxy-pool) with the initial request like so:

```
from autoscraper import AutoScraper

UrlToScrape = "https://sandbox.oxylabs.io/products/3"
WantedList = ["Super Mario Galaxy 2", "91,99 €"]

proxy = {
    "http": "proxy_endpoint",
    "https": "proxy_endpoint",
}
 
InfoScraper = AutoScraper()
InfoScraper.build(UrlToScrape, wanted_list=WantedList, request_args={"proxies": proxy})
# Remaining code...
```

## Saving and loading an AutoScraper model

AutoScraper provides the ability to save and load a pre-trained scraper. We can use the following script to save the InfoScraper object to a file:

```InfoScraper.save("file_name")```

Similarly, we can load a scraper using:

```
SavedScraper = AutoScraper()
SavedScraper.load("file_name")
```
