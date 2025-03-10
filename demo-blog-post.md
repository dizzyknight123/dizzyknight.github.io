---
Title: Data Scrapy Process (by “Group 9”)
Date: 2025-03-10 10:00
Category: Reflective Report
Tags: Group FIN-TANSTIC WIZARDS
---

By Group "Fintech Disruption"

This is a demo blog post. Its purpose is to show how to use the basic
functionality of Markdown in the context of a blog post.

## How to Include a Link and Python Code

We chose [Investing.com](http://www.investing.com) to get the whole
year data of XRP and recalculated the return and 30 days volatility.

The code we use is as follows:
```python
import nltk
import pandas as pd
myvar = 8
DF = pd.read_csv('XRP-data.csv')
```


## How to Include a Quote

As a famous hedge fund manager once said:
>Fed watching is a great tool to make money. I have been making all my
>gazillions using this technique.



## How to Include an Image

Fed Chair Powell is working hard:

![Picture showing Powell]({static}/images/group-Fintech-Disruption_Powell.jpeg)
# Web Scraping BitcoinTalk Forum: A Developer's Journey
*How I Built a Scalable Scraper to Archive 5,000+ Forum Posts*

---

## The Mission
[BitcoinTalk.org](https://bitcointalk.org/) stands as the primordial soup of cryptocurrency discourse. When tasked with archiving historical discussions about blockchain's early days, I faced three core challenges:
1.	Pagination Complexity - Forum URLs use non-sequential numbering (1.3240, 1.3280, etc.)
2.	Nested Content - Each post contains replies requiring multi-level scraping
3.	Ethical Scraping - Maintaining respectful request rates and data organization

---

## Architectural Blueprint
*Key Components*
1.	Page Crawler - Traverses forum board pages
2.	Post Parser - Extracts thread details and replies
3.	Data Batcher - Saves in CSV chunks to prevent memory overload
 
---

## Core Implementation
### 1. Intelligent Pagination Handling
```python
url_end = '1.40'  # Starting point
while url_end != '1.6360':  # Termination condition
    url = f'https://bitcointalk.org/index.php?board={url_end}'
    # ... scraping logic ...
    # Increment by 40 for next page
    current_page, offset = url_end.split('.')
    url_end = f"{current_page}.{str(int(offset) + 40)}"  
```
**Why It Works:**  
•	Mimics observed URL pattern jumps (40 increments)  
•	Avoids hardcoded page counts through dynamic URL generation


---

### 2. Dual-Layer Content Extraction

**Sample Post Structure:**
```html
<td class="windowbg">
  <a href="/index.php?topic=54321">Bitcoin Pizza Day Discussion</a>
</td>
<td class="windowbg2">
  <a href="/index.php?action=profile">SatoshiNakamoto</a>
</td>
```

**Extraction Logic:**
```python
for cell in page.find_all('td', class_='windowbg'):
    title = cell.find('a').text.strip()
    author = cell.find_next_sibling('td').find('a').text.strip()
    post_url = cell.find('a')['href']
    
    # Dive into post page
    post_page_content = requests.get(post_url).text
    soup = BeautifulSoup(post_page_content, 'html.parser')
```

---

### 3. Temporal Data Sanitization

**Raw Date String:**
"June 05, 2024, 03:14:07 PM - Last edit: June 05, 2024, 03:15:00 PM"

*Cleaning Pipeline:*
```python
def clean_date(dirty_str):
    try:
        # Remove edit history
        clean = dirty_str.split('Last edit:')[0].strip()  
        # Convert to datetime object
        dt = datetime.strptime(clean, "%B %d, %Y, %I:%M:%S %p")  
        return dt.strftime("%Y-%m-%d %H:%M:%S")
    except ValueError:
        return None
```

---

### 4. Reply Thread Processing

**Nested Structure Handling:**
```python
replies_content = []
    reply_blocks = post_page_content.find_all('td', class_='msgcl1')
    for reply_block in reply_blocks[1:]:
        reply_author_td = reply_block.find('td', class_='poster_info')
        if reply_author_td:
            reply_author_a = reply_author_td.find('a')
            if reply_author_a:
                reply_author = reply_author_a.get_text(strip=True)
            else:
                reply_author = "Unknown"
        else:
            reply_author = "Unknown"

        reply_date_div = reply_block.find('td', class_='td_headerandpost').find('div', class_='smalltext')
        if reply_date_div:
            reply_date_str = reply_date_div.get_text(strip=True)
            try:
                reply_date_str = reply_date_str.split('Last edit:')[0].strip()
                reply_date = datetime.strptime(reply_date_str, "%B %d, %Y, %I:%M:%S %p")
                reply_date = reply_date.strftime("%Y-%m-%d %H:%M:%S")
            except ValueError as e:
                print(f"Error parsing reply date: {reply_date_str}. Error: {e}")
                reply_date = None
        else:
            reply_date = None

        reply_text_div = reply_block.find('div', class_='post')

        for elem in reply_text_div.find_all(['div'], class_=['quoteheader', 'quote']):
            if elem.name == 'div' and elem.get('class') and 'quoteheader' in elem.get('class'):
                elem.extract()
            elif elem.name == 'div' and elem.get('class') and 'quote' in elem.get('class'):
                elem.extract()
        
        if reply_text_div:
            # reply_text = clean_text(reply_text_div.get_text(strip=True))
            reply_text = reply_text_div.get_text(strip=True)
        else:
            reply_text = "No content"

        if reply_text:
            replies_content.append((reply_author, reply_date, reply_text))
```
**Key Decisions:**  
•	Skip first msgcl1 element (original post)  
•	Remove quote blocks to avoid duplicate content  
•	Preserve reply hierarchy through tuple storage

---

## Storage Strategy
CSV Chunking System
```python
POSTS_PER_FILE = 520  # ~7MB per CSV

if len(final_list) >= POSTS_PER_FILE:
    save_to_csv(final_list, current_file_count)
    final_list = []
    current_file_count += 1
```
**File Structure:**
| Post_Title | Post_Date | Post_Author | Post_Content | Reply_Author | Reply_Date | Reply_Content |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| "Bitcoin Pizza Day" | 2024-06-05 13:11:51 | "Satoshi" | "Never sell BTC..." | "PizzaLover" | 2024-06-05 15:57:29 | "I actually bought..." |

**Advantages:**  
•	Prevents memory bloat with large datasets  
•	Enables parallel processing of chunks(kaggle)  
•	Maintains relational data integrity

---

## Anti-Block Measures
**Rate Limiting:**
```python
time.sleep(1)  # Between page requests
```
---

**Final Stats:**  
✅ 5,477 posts archived  
✅ 104,074 replies processed  
✅ 12 CSV files generated  
⏱️ Total runtime: 3h22m  

Would you implement this differently? Let's discuss in the comments! 🚀
