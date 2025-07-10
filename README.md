<p align="center">
    <img src="https://github.com/jpjacobpadilla/Stealth-Requests/blob/173df6b8a8ef53bd1fd514b85291c5f98530a462/logo.png?raw=true">
</p>

<h1 align="center">The Easiest Way to Crawl the Web</h1>

<p align="center">
    <a href="https://github.com/jpjacobpadilla/stealth-requests/blob/main/LICENSE">
        <img src="https://img.shields.io/github/license/jpjacobpadilla/stealth-requests.svg?color=green">
    </a> 
    <a href="https://www.python.org/">
        <img src="https://img.shields.io/badge/python-3.9%2B-green" alt="Python 3.8+">
    </a> 
    <a href="https://pypi.org/project/stealth-requests/">
        <img alt="PyPI" src="https://img.shields.io/pypi/v/stealth-requests.svg?color=green">
    </a> 
    <a href="https://pepy.tech/project/stealth-requests">
        <img alt="PyPI installs" src="https://img.shields.io/pepy/dt/stealth-requests?label=pypi%20installs&color=green">
    </a>
</p>


### Features
- **Realistic HTTP Requests:**
    - Mimics the Chrome browser for undetected scraping using the [curl_cffi](https://curl-cffi.readthedocs.io/en/latest/) package
    - Automatically rotates User Agents between requests
    - Tracks and updates the `Referer` header to simulate realistic request chains
    - Built-in retry logic for failed requests (e.g. 429, 503, 522)
- **Faster and Easier Parsing:**
    - Extract emails, phone numbers, images, and links with simple methods
    - Automatically extract metadata (title, description, author, etc.) from HTML-based responses
    - Seamlessly convert responses into [Lxml](https://lxml.de/apidoc/lxml.html) and [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/) objects for more parsing
    - Easily convert full or specific sections of HTML to Markdown


### Install

```
$ pip install stealth_requests
```


### Table of Contents

- [Sending Requests](#sending-requests)
- [Sending Requests With Asyncio](#sending-requests-with-asyncio)
- [Accessing Page Metadata](#accessing-page-metadata)
- [Extracting Emails, Phone Numbers, Images, and Links](#extracting-emails-phone-numbers-images-and-links)
- [More Parsing Options](#more-parsing-options)
- [Converting Responses to Markdown](#converting-responses-to-markdown)
- [Using Proxies](#using-proxies)


### Sending Requests

Stealth-Requests mimics the API of the [requests](https://requests.readthedocs.io/en/latest/) package, allowing you to use it in nearly the same way.

You can send one-off requests like such:

```python
import stealth_requests as requests

resp = requests.get('https://link-here.com')
```

Or you can use a `StealthSession` object which will keep track of certain headers for you between requests such as the `Referer` header.

```python
from stealth_requests import StealthSession

with StealthSession() as session:
    resp = session.get('https://link-here.com')
```

When sending a request, or creating a `StealthSession`, you can specify the type of browser that you want the request to mimic - either `chrome`, which is the default, or `safari`. If you want to change which browser to mimic, set the `impersonate` argument, either in `requests.get` or when initializing `StealthSession` to `safari` or `chrome`.

You can also specify how many times to retry the request on failed requests!

```python
import stealth_requests as requests

resp = requests.get('https://link-here.com', retry=3)
```

### Sending Requests With Asyncio

This package supports Asyncio in the same way as the `requests` package:

```python
from stealth_requests import AsyncStealthSession

async with AsyncStealthSession(impersonate='safari') as session:
    resp = await session.get('https://link-here.com')
```

or, for a one-off request, you can make a request like this:

```python
import stealth_requests as requests

resp = await requests.get('https://link-here.com', impersonate='safari')
```


### Accessing Page Metadata

The response returned from this package is a `StealthResponse`, which has all of the same methods and attributes as a standard [requests response object](https://requests.readthedocs.io/en/latest/api/#requests.Response), with a few added features. One of these extra features is automatic parsing of header metadata from HTML-based responses. The metadata can be accessed from the `meta` property, which gives you access to the following metadata:

- title: `str | None`
- author: `str | None`
- description: `str | None`
- thumbnail: `str | None`
- canonical: `str | None`
- twitter_handle: `str | None`
- keywords: `tuple[str] | None`
- robots: `tuple[str] | None`

Here's an example of how to get the title of a page:

```python
import stealth_requests as requests

resp = requests.get('https://link-here.com')
print(resp.meta.title)
```


### Extracting Emails, Phone Numbers, Images, and Links

If you would like to get all of the webpage URLs (`a` tags) from an HTML-based response, you can use the `links` property. If you'd like to get all image URLs (`img` tags) you can use the `images` property from a response object.

```python
import stealth_requests as requests

resp = requests.get('https://link-here.com')
for image_url in resp.images:
    # ...
```

Stealth-Requests also includes built-in methods for extracting emails and phone numbers from the raw HTML of a page. These methods use simple regex patterns to return a de-duplicated list of matches.

You can access them with the `.emails` and `.phone_numbers` properties:

```python
import stealth_requests as requests

resp = requests.get('https://link-here.com')

print(resp.emails)
# ('hello@example.com', 'contact@site.com')

print(resp.phone_numbers)
# ('(212) 555-1234', '+1-800-123-4567')
```


### More Parsing Options

To make parsing HTML faster, I've also added two popular parsing packages to Stealth-Requests - Lxml and BeautifulSoup4. To use these add-ons you need to install the `parsers` extra: 

```
$ pip install 'stealth_requests[parsers]'
```

To easily get an Lxml tree, you can use `resp.tree()` and to get a BeautifulSoup object, use the `resp.soup()` method.

For simple parsing, I've also added the following convenience methods, from the Lxml package, right into the `StealthResponse` object:

- `text_content()`: Get all text content in a response
- `xpath()` Go right to using XPath expressions instead of getting your own Lxml tree.


### Converting Responses to Markdown

In some cases, it’s easier to work with a webpage in Markdown format rather than HTML. After making a GET request that returns HTML, you can use the `resp.markdown()` method to convert the response into a Markdown string, providing a simplified and readable version of the page content!

`markdown()` has two optional parameters:

1. `content_xpath` An XPath expression, in the form of a string, which can be used to narrow down what text is converted to Markdown. This can be useful if you don't want the header and footer of a webpage to be turned into Markdown.
2. `ignore_links` A boolean value that tells Html2Text whether to include links in the Markdown output.


### Using Proxies


Stealth-Requests supports proxy usage through a `proxies` dictionary, similar to the standard requests package.


You can pass both http and https proxy URLs when making a request:

```python
import stealth_requests as requests

proxies = {
    "http": "http://username:password@proxyhost:port",
    "https": "http://username:password@proxyhost:port",
}

resp = requests.get('https://link-here.com', proxies=proxies)
```

[↑ Back to top](#table-of-contents)