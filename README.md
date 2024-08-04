# Web Scraping and Data Storage
This code is used to scrape book data from http://books.toscrape.com/ filter and sort the data, and then save it to a CSV file and a SQLite database.

## Fetching Data
The code start by sending a GET request to the website using the **'requests'** library. The HTML content of the page is then parsed using the **'xml'** library.

```python
import requests
from lxml import html

url = 'http://books.toscrape.com/'
response = requests.get(url)
html_content = response.content

tree = html.fromstring(html_content)
books = tree.xpath('//article[@class="product_pod"]')
```

## Extracting Book Data
The code then extracts the title, price and availability of each book using XPath expressions.

```python
data = []
for book in books:
    title = book.xpath('.//h3/a/@title')[0]
    price = book.xpath('.//div[@class="product_price"]/p[@class="price_color"]/text()')[0]
    availability = book.xpath('.//div[@class="product_price"]/p[@class="instock availability"]/text()')[1].strip()
    data.append({"title": title, "price": price, "availability": availability})
```

## Filtering and Sorting Data
The code defines two functions, **'filter_books'** and **'sort_books'**, to filter and sort the data.

```python
def filter_books(data, min_price=None, max_price=None, availability=None):
    filtered_data = []
    for item in data:
        price = float(item['price'].replace('£', ''))
        if ((min_price is None or price >= min_price) and 
            (max_price is None or price <= max_price) and 
            (availability is None or item['availability'] == availability)):
            filtered_data.append(item)
    return filtered_data

def sort_books(data, key, reverse=False):
    return sorted(data, key=lambda x: x[key], reverse=reverse)
```
## Saving Data
The code defines two functions, **'save_to_csv'** and **'save_to_sqlite'**, to save the data to a CSV file and SQLite database.

```python
import csv
import sqlite3

def save_to_csv(data, filename='books.csv'):
    with open(filename, mode='w', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=["title", "price", "availability"])
        writer.writeheader()
        writer.writerows(data)

def save_to_sqlite(data, db_name='books.db'):
    conn = sqlite3.connect(db_name)
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS books
                      (title TEXT, price REAL, availability TEXT)''')
    
    for item in data:
        cursor.execute('INSERT INTO books (title, price, availability) VALUES (?, ?, ?)',
                       (item['title'], float(item['price'].replace('£', '')), item['availability']))
    
    conn.commit()
    conn.close()
```
## Example Usage
The code filters the data to include only books with a price between £10 and £30 and availability 'In stock', sorts the data by price, and saves it to a CSV file and a SQLite database.

```python
filtered_data = filter_books(data, min_price=10, max_price=30, availability='In stock')
sorted_data = sort_books(filtered_data, key='price', reverse=False)

save_to_csv(sorted_data, filename='filtered_sorted_books.csv')
save_to_sqlite(sorted_data, db_name='filtered_sorted_books.db')
```









