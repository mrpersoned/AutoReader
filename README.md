import requests
import pandas as pd
import matplotlib.pyplot as plt

# 🔐 Add your NewsAPI key here
news_api_key = "84fd62b4c9d94e53917b4fcfba3b008f"  # ← Replace this

def get_top_losers_from_yahoo_json(min_percent_drop=-5.0):
    url = "https://query1.finance.yahoo.com/v1/finance/screener/predefined/saved"
    params = {
        "scrIds": "day_losers",
        "count": "100"
    }
    headers = {
        "User-Agent": "Mozilla/5.0"
    }

    r = requests.get(url, headers=headers, params=params)
    results = r.json()['finance']['result'][0]['quotes']

    data = []
    for quote in results:
        try:
            symbol = quote['symbol']
            name = quote.get('shortName', '')
            price = quote['regularMarketPrice']
            percent_change = quote['regularMarketChangePercent']
            if percent_change <= min_percent_drop:
                data.append({
                    "Symbol": symbol,
                    "Name": name,
                    "Price": price,
                    "% Change": percent_change
                })
        except KeyError:
            continue

    return pd.DataFrame(data)

# 🔍 Get a recent headline and URL for a stock
def get_stock_news(symbol, api_key, max_results=1):
    url = "https://newsapi.org/v2/everything"
    params = {
        "q": symbol,
        "language": "en",
        "sortBy": "publishedAt",
        "pageSize": max_results,
        "apiKey": api_key
    }
    try:
        r = requests.get(url, params=params)
        articles = r.json().get("articles", [])
        if articles:
            return articles[0]["title"], articles[0]["url"]
    except Exception as e:
        return f"News error: {e}", ""
    return "No news found", ""

# --- Main Execution ---

# Fetch and display
top_losers_df = get_top_losers_from_yahoo_json()
top_losers_df = top_losers_df.sort_values(by="% Change").head(10)  # Top 10 losers

# Fetch headlines and URLs
print("\n🔎 Top 10 Stock Losers and Relevant News:")
news_data = []
for symbol in top_losers_df["Symbol"]:
    title, link = get_stock_news(symbol, news_api_key)
    news_data.append({"Symbol": symbol, "Headline": title, "URL": link})
    print(f"\n🔻 {symbol}")
    print(f"📰 {title}")
    print(f"🔗 {link}")

# Plot the chart (no annotations)
plt.figure(figsize=(14, 6))
plt.bar(top_losers_df["Symbol"], top_losers_df["% Change"], color='red')
plt.title("Top Stock Losers (>5% drop)")
plt.xlabel("Symbol")
plt.ylabel("% Change")
plt.xticks(rotation=45)
plt.grid(True, linestyle='--', alpha=0.5)
plt.tight_layout()
plt.show()
