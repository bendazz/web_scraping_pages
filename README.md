# Web Scraping Practice Pages

This repository contains a small static website you can host on GitHub Pages for teaching web scraping with Python `requests` and `BeautifulSoup`.

Each page focuses on a concept to practice parsing HTML and extracting data. Because it's a static site, it's ideal for deterministic, repeatable exercises.

## Pages

- `index.html`: hub with links to all practice pages
- `pages/basic-text.html`: headings and paragraphs
- `pages/lists-links-images.html`: lists, links, and images (including a lazy-loaded image pattern)
- `pages/tables.html`: simple and complex tables (colspan/rowspan)
- `pages/nested-structure.html`: nested elements and hierarchy (articles, comments)
- `pages/attributes-selectors.html`: attributes, classes, data-*, and attribute selectors
- `pages/forms.html`: forms and input fields (static only)
- `pages/javascript-content.html`: JS-rendered elements (not visible to `requests`)
- `pages/pagination/page1.html`, `page2.html`, `page3.html`: pagination demo with prev/next links

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

## Starter Python snippets

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
url = "https://<your-username>.github.io/<repo-name>/pages/attributes-selectors.html"
soup = BeautifulSoup(requests.get(url, timeout=10).text, "html.parser")

on_sale = [li["data-sku"] for li in soup.select("#products li.on-sale")]
featured_name = soup.select_one("#products li.featured .name").get_text(strip=True)
users = [
	{
		"username": a.get_text(strip=True),
		"href": a["href"],
		"active": tr.get("data-active") == "true",
	}
	for tr in soup.select("#users tbody tr")
	for a in [tr.select_one("a.user")]
]
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

- Assign each page as a small exercise; progress from selectors to pagination.
- Ask students to extract attributes (href, src, data-*) vs text content.
- Discuss robots.txt and ethics; add delays and headers to mimic polite clients.
- For advanced topics, compare `requests` results vs a headless browser (e.g., Playwright) on the JS page.

## License

MIT
