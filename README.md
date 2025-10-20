# Web Scraping Practice Pages

This repository contains a small static website you can host on GitHub Pages for teaching web scraping with Python `requests` and `BeautifulSoup`.

Currently, it includes a single practice page focused on basic text (multiple h1/h2/h3 headings and paragraphs). You can add more pages over time as lessons progress.

## Pages

- `index.html`: hub
- `pages/basic-text.html`: headings and paragraphs (multiple instances for counting/grouping)
- `pages/simple-table.html`: one simple table with 10 rows
- `pages/many-tables.html`: multiple tables with varying shapes and edge cases

## Local preview

You can open `index.html` directly in the browser, or run a simple local server to test relative links:

```bash
python3 -m http.server 8000
```

Then visit http://localhost:8000/

## Host on GitHub Pages

1. Push this repo to GitHub.
2. In your repository settings, go to "Pages".
3. Under "Build and deployment", select "Deploy from a branch".
4. Select the `main` branch and `/ (root)`.
5. Save; your site will be published at `https://<your-username>.github.io/<repo-name>/`.

If you use a custom domain, configure DNS and set it in Pages settings.

## Starter Python snippet

Install dependencies:

```bash
pip install requests beautifulsoup4
```

Fetch and parse a page:

```python
import requests
from bs4 import BeautifulSoup

url = "https://<your-username>.github.io/<repo-name>/pages/basic-text.html"
r = requests.get(url, timeout=10)
r.raise_for_status()
soup = BeautifulSoup(r.text, "html.parser")

# Examples
title = soup.find("h1", {"id": "main-title"}).get_text(strip=True)
paragraphs = [p.get_text(strip=True) for p in soup.select("p")]
```

Select by attributes and classes:

```python
url = "https://<your-username>.github.io/<repo-name>/pages/basic-text.html"
soup = BeautifulSoup(requests.get(url, timeout=10).text, "html.parser")

# Count headings and paragraphs
h1s = soup.select("#content h1")
h2s = soup.select("#content h2")
h3s = soup.select("#content h3")
paras = soup.select("#content p")
print(len(h1s), len(h2s), len(h3s), len(paras))

# Parse simple table
import pandas as pd  # optional but handy
url = "https://<your-username>.github.io/<repo-name>/pages/simple-table.html"
soup = BeautifulSoup(requests.get(url, timeout=10).text, "html.parser")

table = soup.select_one("#products")
headers = [th.get_text(strip=True) for th in table.select("thead th")]
rows = []
for tr in table.select("tbody tr"):
	rows.append([td.get_text(strip=True) for td in tr.select("td")])

data = [dict(zip(headers, r)) for r in rows]
print(data[0])

# Iterate over many tables
url = "https://<your-username>.github.io/<repo-name>/pages/many-tables.html"
soup = BeautifulSoup(requests.get(url, timeout=10).text, "html.parser")

tables = soup.select("table")
print("Found", len(tables), "tables")
def parse_table(table):
	# Try to read headers from thead, else fall back to first row
	headers = [th.get_text(strip=True) for th in table.select("thead th")]
	if not headers:
		first = table.find("tr")
		headers = [th.get_text(strip=True) for th in first.find_all(["th","td"])]
		body_rows = first.find_next_siblings("tr")
	else:
		body_rows = table.select("tbody tr") or table.find_all("tr")[1:]
	rows = [[td.get_text(strip=True) for td in tr.find_all("td")] for tr in body_rows]
	return headers, rows

for t in tables:
	headers, rows = parse_table(t)
	print(t.get("id") or "(no id)", "->", len(headers), "cols,", len(rows), "rows")
```

Tables to dicts:

```python
url = "https://<your-username>.github.io/<repo-name>/pages/tables.html"
soup = BeautifulSoup(requests.get(url, timeout=10).text, "html.parser")

def table_to_dicts(table):
	headers = [th.get_text(strip=True) for th in table.select("thead th")]
	rows = []
	for tr in table.select("tbody tr"):
		cells = [td.get_text(strip=True) for td in tr.select("td")]
		rows.append(dict(zip(headers, cells)))
	return rows

simple = table_to_dicts(soup.select_one("#simple"))
```

Paginate across pages:

```python
import urllib.parse as up

base = "https://<your-username>.github.io/<repo-name>/pages/pagination/page1.html"
while True:
	res = requests.get(base, timeout=10)
	res.raise_for_status()
	soup = BeautifulSoup(res.text, "html.parser")
	items = [li.get_text(strip=True) for li in soup.select("#results li")]
	print(items)

	next_link = soup.select_one("nav a[rel=next]")
	if not next_link:
		break
	base = up.urljoin(base, next_link["href"])  # follow relative link
```

JS-rendered caveat:

```python
url = "https://<your-username>.github.io/<repo-name>/pages/javascript-content.html"
html = requests.get(url, timeout=10).text
print("#items present?", "#items" in html)  # True (the container)
print("Gamma present?", "Gamma" in html)   # False (rendered by JS)
```

## Teaching ideas

- Start with counting and scoping selections inside `#content`.
- Ask students to compute paragraph counts per heading.
- Discuss robots.txt and ethics; add delays and headers to mimic polite clients.

## License

MIT
