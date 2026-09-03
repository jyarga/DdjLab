## Writing my prompt

First, observe carefully the content of the website you want to scrap. That will help describe as accurately as possile, when writing the prompt.
Here my prompt template
```
Here the link for a news website: [https://yourdomain.com/]

#Description: It contains articles both in text and videos. Plus, each article belongs to category or a tag. For example, the article ”La première d’un film sur les activités de l’Africa Corps du ministère de la Défense de la Fédération de Russie au Mali s’est tenue à Moscou
” is published under the category ”ART & CULTURE”. 

#Objectif: get the list of all articles published in the website between [DD/MM/YYYY] and [DD/MM/YYYY]. The final should be a table containing the following elements: the url, the title, the author, the publication date, the category for each article published in [https://yourdomain.com/]

#Tasks

Go through this website and suggest a python code that works in Google Colab; 
Add comment to each line of code explaining (to non-coders) what is does ;

The code should: 

Tell how many articles are published in the web site (wee can specify an time range); 
provide the url, the title, the publication date, the author and the category or tag for each article (specify the time range if needed);  
display the number of articles published in the website [https://yourdomain.com/], 
export the list in a csv (or another) format.

Obs: Make sure to get all the articles. And if not possible, explain why.
```
Here is the Python code I got
```
import requests         
import pandas as pd  
import time             
import html     
from datetime import datetime, timedelta  
from google.colab import files 

end_date = datetime.now()
start_year = 2020 

base_url = "https://nordictimes.com/wp-json/wp/v2/posts"

all_articles = []

seen_urls = set()

def get_monthly_ranges(start_year, end_date):
    """Generates (start_of_month, end_of_month) tuples going backward."""
    current = end_date
    ranges = []
    while current.year >= start_year:
        month_start = datetime(current.year, current.month, 1)
        if current.month == 12:
            month_end = datetime(current.year + 1, 1, 1) - timedelta(seconds=1)
        else:
            month_end = datetime(current.year, current.month + 1, 1) - timedelta(seconds=1)
        ranges.append((month_start, month_end))
        current = month_start - timedelta(days=1)
    return ranges
monthly_ranges = get_monthly_ranges(start_year, end_date)

print(f"Starting date-block extraction across {len(monthly_ranges)} monthly periods...\n")

for m_start, m_end in monthly_ranges:
    after_str = m_start.strftime("%Y-%m-%dT00:00:00")
    before_str = m_end.strftime("%Y-%m-%dT23:59:59")
    
    month_label = m_start.strftime("%B %Y")
    print(f"--- Fetching articles for {month_label} ({after_str} to {before_str}) ---")
    page = 1
    per_page = 100 
    while True:
        url = f"{base_url}?after={after_str}&before={before_str}&per_page={per_page}&page={page}&_embed"
        
        try:
            response = requests.get(url, headers={"User-Agent": "Mozilla/5.0"}, timeout=15)
           
            if response.status_code != 200:
                break
                
            posts = response.json()
            if not posts or not isinstance(posts, list):
                break
                
            for post in posts:
                article_url = post.get("link", "")
                
                if article_url in seen_urls:
                    continue
                seen_urls.add(article_url)
                
                raw_title = post.get("title", {}).get("rendered", "")
                article_title = html.unescape(raw_title)
                
                pub_date = post.get("date", "")
                
                author_name = "Unknown"
                if "_embedded" in post and "author" in post["_embedded"]:
                    authors = post["_embedded"]["author"]
                    if authors and len(authors) > 0:
                        author_name = authors[0].get("name", "Unknown")
                
                categories_tags = []
                if "_embedded" in post and "wp:term" in post["_embedded"]:
                    for term_group in post["_embedded"]["wp:term"]:
                        for term in term_group:
                            term_name = term.get("name")
                            if term_name and term_name not in categories_tags:
                                categories_tags.append(term_name)
                                
                category_text = ", ".join(categories_tags) if categories_tags else "Uncategorized"
                
                all_articles.append({
                    "URL": article_url,
                    "Title": article_title,
                    "Publication Date": pub_date,
                    "Author": author_name,
                    "Category / Tag": category_text
                })
                
            print(f"  > Page {page} done. Total collected so far: {len(all_articles)}")
            page += 1
            time.sleep(0.4)             
        except Exception as e:
            print(f"Error fetching page {page} for {month_label}: {e}")
            break

total_found = len(all_articles)
print("\n" + "="*50)
print(f"TOTAL UNIQUE ARTICLES COLLECTED ACROSS ALL MONTHS: {total_found}")
print("="*50 + "\n")

df = pd.DataFrame(all_articles)
display(df.head(10))

csv_filename = "nordic_times_all_articles_monthly.csv"
df.to_csv(csv_filename, index=False, encoding="utf-8-sig")
print(f"\nSaved file as '{csv_filename}'. Downloading now...")

files.download(csv_filename)
```



