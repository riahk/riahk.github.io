---
layout: post
title:  "Basic Web Scraping with Python (and BeautifulSoup)"
date:   2015-08-18
published: false
categories: python
---

The internet is full of data. Settling an argument with your friend over the height of the world's tallest mountain is as easy as a simple search query and voila! Instant information, directly to your brain. But what if we want that information in a more easily useable and malleable format? Sure, there are APIs for some databases, but maybe we just want to go directly to the source ourselves.

Enter the web scraper: a script that reads raw HTML from a webpage and extracts the data we need, converting it into an easy to use format such as raw text or JSON. This sounds like a long, complicated process, and the thought of writing a web scraper intimidated me before I actually tried it. Let's code one!

Our goal: write a simple web scraper that loads the HTML from [japaneseemoticons.me](http://japaneseemoticons.me/triumph-emoticons/), extract the unicode emoticons and write them to a raw text file. We're going to do this with the help of python's urllib module as well as BeautifulSoup4, a super useful module for traversing the HTML data of a webpage. 

#### Setup ####

This tutorial assumes you already have python installed and some experience with the language. First thing we'll do is install [BeautifulSoup4](http://www.crummy.com/software/BeautifulSoup/bs4/doc/#installing-beautiful-soup), which is easiest if you already have pip:

```
pip install beautifulsoup4
```

For other install options, check out the link above (which also has a very useful guide on beautifulsoup's features).

Now make a new directory and open up a fresh new .py file. We'll be importing BeautifulSoup (obviously) as well as urllib, which is python's built-in url reader.

```python
from urllib import urlopen
from bs4 import BeautifulSoup
```

#### Getting Raw HTML ####

Before we write more of our scraper, it's important to have an idea about the form of the data we're working with and what elements on the page contain the relevant information we need. In this case we need to know what elements on the page contain the emoticons, and how to best filter these out. An easy way to play around with the data is running code through the python interpreter. Note that we're looking specifically at the 'excited' emoji page.

```python
url = 'http://japaneseemoticons.me/excited-emoticons/'
html = urlopen(url).read()
print html
```

Here's a snippet of what you might see:
```html
<table style="width: 100%;">
<tbody>
<tr>
<td style="text-align: center;">☆*･゜ﾟ･*(^O^)/*･゜ﾟ･*☆</td>
</tr>
<tr>
<td style="text-align: center;">☆*:.｡. o(≧▽≦)o .｡.:*☆</td>
</tr>
<tr>
<td style="text-align: center;">*✲ﾟ*｡✧٩(･ิᴗ･ิ๑)۶*✲ﾟ*｡✧</td>
</tr>
<tr>
<td style="text-align: center;">｡.ﾟ+:((ヾ(｡･ω･)ｼ)).:ﾟ+｡</td>
</tr>
<tr>
<td style="text-align: center;">＼（　●　⌒　∇　⌒　●　）／</td>
</tr>
...
```

From here we can see that all the emoticons are contained in <td> elements on the page. We'll want to cycle through each <td> element and extract the inner text.

#### Parsing HTML ####

Enter BeautifulSoup! Let's take a look at what we get when we 'soupify' our raw HTML:

```python
soup = BeautifulSoup(html, 'html.parser')
print soup
```

Printing soup gives us the same output as printing the raw html; however, when we check type(soup) we'll see that we have a class 'bs4.BeautifulSoup' object. This gives us access to some useful functions, such as find_all() which we'll use to get an array of the <td> elements on the page.

```python
tdElems = soup.find_all('td')
print tdElems[0]
```

You'll see the first <td> element on the page:

```html
<td style="width: 33%; text-align: center;">(((o(*ﾟ▽ﾟ*)o)))</td>
```

Now all we have to do to extract the emoticon from our <td> element is use the get_text() function:

```python
emoji = tdElems[0].get_text()
print emoji
```

Ta-da! We have our data! Now we can manipulate it however we want.

#### Writing to a file ###

There are multiple ways we could use our data; we can further manipulate it in our script, save it as a json object, or as a raw text file. We'll be doing the latter, using python's file reading and writing capabilities; Let's start by creating a new file object to save our emoticons to:

```python
f = open('emoji.txt', 'w')
```

Our first argument is the path and name of our file; if it exists, python will open the existing file; otherwise it will create the file. In this example we're creating our file in whatever directory we run our script from. The second argument specifies the user permissions, in this case we are allowing the user to write to the file. (Note: we have not given it read permissions, so if we tried to run f.read() we would get an error).

Say we just want to save the single emoticon from above into our file:

```python
f.write(emoji.encode('utf-8')
f.close()
```

Don't forget the encoding, or you'll get an error! If we find and open our file, we'll see that our emoticon has been saved into it. We can now put all these parts together to create a working web scraper that gets all the emojis from the page:

```python
from urllib import urlopen
from bs4 import BeautifulSoup

url = 'http://japaneseemoticons.me/excited-emoticons/'
html = urlopen(url).read()

soup = BeautifulSoup(html, 'html.parser')
emojis = ''
f = open('emojis.txt', 'w')

for td in soup.find_all('td'):
  emoji = td.get_text()
  emojis += emoji + '\n'

f.write(emojis.encode('utf8')
f.close()
```

That's it! This is an extremely simple example, and we were lucky that our data was in such an easy to filter format. There are multiple ways we could add to this script: for example, scraping multiple URLs and saving to multiple files or saving to a JSON object as opposed to raw text. However, knowing the fundamentals of a web scraper is the first step to tapping into the massive amounts of data available on the net. 
