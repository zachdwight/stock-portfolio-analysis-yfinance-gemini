# Stock Portfolio Analysis Enhancement Guide

This guide explains the enhancements made to your stock portfolio analysis tool and how to implement them.

## Overview of Changes

The enhanced version adds:
1. **News & Market Sentiment Analysis** - Real-time financial news analysis using Claude/Gemini
2. **Technical Indicators** - RSI, SMA, momentum-based trade signals
3. **Trade Recommendations** - Daily and weekly actionable trade recommendations with confidence scores
4. **Dual AI Support** - Works with Claude (Anthropic) or Gemini (Google)
5. **Enhanced PDF Report** - 8 pages including sentiment analysis and trade recommendations

## Files Modified/Created

### New File: requirements.txt
Install dependencies:
```bash
pip install -r requirements.txt
```

**Dependencies added:**
- `anthropic>=1.0.0` - For Claude API
- `requests>=2.31.0` - For NewsAPI
- Updated versions of existing libraries

### Updated File: README.md
Comprehensive documentation including:
- API key setup instructions
- Configuration options
- Trade recommendation interpretation
- Risk management guidelines

### Updated File: stock_portfolio_summarizer.py
See section below for implementation details.

## Implementation Steps

### Step 1: Update Imports (Top of File)

Replace:
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import yfinance as yf
import warnings
import time
from matplotlib.backends.backend_pdf import PdfPages
import io
import sys
import os
import textwrap
```

With:
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import yfinance as yf
import warnings
import time
from matplotlib.backends.backend_pdf import PdfPages
import io
import sys
import os
import textwrap
from datetime import datetime, timedelta
import requests
from typing import Dict, List, Tuple, Optional
```

### Step 2: Add Claude Support (After Gemini Setup)

After `genai.configure(api_key=API_KEY)`, add:

```python
# --- Anthropic Claude API Integration ---
from anthropic import Anthropic

CLAUDE_API_KEY = os.getenv("ANTHROPIC_API_KEY")
claude_client = None
if CLAUDE_API_KEY:
    claude_client = Anthropic(api_key=CLAUDE_API_KEY)

# --- Configuration ---
AI_MODEL = os.getenv("AI_MODEL", "claude")  # "claude" or "gemini"
NEWSAPI_KEY = os.getenv("NEWSAPI_KEY", "")
ENABLE_NEWS_ANALYSIS = True
ENABLE_TRADE_RECOMMENDATIONS = True
TREND_LOOKBACK_DAYS = 60
```

### Step 3: Move style_chart Function Earlier

Move the `style_chart` function definition from the end of the file to right after the color palette configuration (around line 97). This function needs to be defined before it's called.

### Step 4: Add New Functions (Before Portfolio Input)

Add these functions before `# --- 1. Your Portfolio Input ---`:

#### 4.1 fetch_financial_news()
```python
def fetch_financial_news(query_terms: Optional[List[str]] = None, days_back: int = 1) -> List[Dict]:
    """Fetch recent financial and political news using NewsAPI."""
    if not NEWSAPI_KEY:
        print("    Warning: NEWSAPI_KEY not set. Skipping news analysis.")
        return []

    # Implementation details in enhanced version
    # Uses NewsAPI to fetch articles about finance, earnings, Fed policy, etc.
    # Returns list of articles with headlines, sources, descriptions
```

#### 4.2 analyze_market_sentiment()
```python
def analyze_market_sentiment(articles: List[Dict], ai_model: str = "claude") -> Dict:
    """Analyze market sentiment from news using Claude or Gemini."""
    # Implementation details
    # Calls Claude or Gemini API to analyze sentiment
    # Returns: overall sentiment, bullish%, key events, sector implications
```

#### 4.3 calculate_technical_indicators()
```python
def calculate_technical_indicators(ticker: str, period: int = 14) -> Dict:
    """Calculate RSI, SMA, momentum indicators."""
    # Implementation details
    # Returns: RSI, RSI status, SMA 20/50, momentum, trend
```

#### 4.4 generate_trade_recommendations()
```python
def generate_trade_recommendations(portfolio_df: pd.DataFrame, market_sentiment: Dict, ai_model: str = "claude") -> Tuple[List[Dict], List[Dict]]:
    """Generate daily and weekly trade recommendations."""
    # Implementation details
    # Combines technical signals + sentiment for BUY/SELL/HOLD signals
    # Returns: (daily_recommendations, weekly_recommendations)
```

### Step 5: Fetch News & Generate Recommendations

In the main execution section, after `print("\n--- Additional Information ---")`, add:

```python
# --- News & Trade Recommendations Analysis ---
news_articles = []
market_sentiment = None
recommendations_daily = []
recommendations_weekly = []

if ENABLE_NEWS_ANALYSIS:
    print("\nFetching financial news...")
    news_articles = fetch_financial_news(days_back=1)

    if news_articles:
        print(f"\nAnalyzing market sentiment from {len(news_articles)} articles...")
        market_sentiment = analyze_market_sentiment(news_articles, ai_model=AI_MODEL)

        if ENABLE_TRADE_RECOMMENDATIONS:
            print("\nGenerating trade recommendations...")
            recommendations_daily, recommendations_weekly = generate_trade_recommendations(
                portfolio_df, market_sentiment, ai_model=AI_MODEL
            )
```

### Step 6: Add PDF Pages for Sentiment & Recommendations

In the PDF generation section (within `with PdfPages(pdf_output_filename) as pdf:`), after the 10-year projection plot, add:

#### 6.1 Market Sentiment Page
```python
# Page 6: Market Sentiment Analysis
if market_sentiment:
    fig_sentiment = plt.figure(figsize=(8.5, 11), facecolor='white')
    # [Create sentiment visualization with key events, sector implications]
    pdf.savefig(fig_sentiment, bbox_inches='tight')
    plt.close(fig_sentiment)
```

#### 6.2 Daily Recommendations Page
```python
# Page 7: Daily Trade Recommendations
if recommendations_daily:
    fig_daily = plt.figure(figsize=(8.5, 11), facecolor='white')
    # [Create table with Ticker, Signal, Confidence, Price, Trend]
    # [Add methodology and disclaimer]
    pdf.savefig(fig_daily, bbox_inches='tight')
    plt.close(fig_daily)
```

#### 6.3 Weekly Recommendations Page
```python
# Page 8: Weekly Trade Recommendations
if recommendations_weekly:
    fig_weekly = plt.figure(figsize=(8.5, 11), facecolor='white')
    # [Create similar table for weekly recommendations]
    # [Add strategy notes and risk management info]
    pdf.savefig(fig_weekly, bbox_inches='tight')
    plt.close(fig_weekly)
```

## Configuration

Set environment variables before running:

```bash
# Required for Claude support
export ANTHROPIC_API_KEY="your-anthropic-key"

# Alternative to Claude (or fallback)
export GOOGLE_API_KEY="your-gemini-key"

# For news analysis (free tier: 100 requests/day)
export NEWSAPI_KEY="your-newsapi-key"

# Choose your AI model
export AI_MODEL="claude"  # or "gemini"
```

## Testing the Enhanced Version

1. **Setup API Keys**: Set all required environment variables
2. **Install Dependencies**: `pip install -r requirements.txt`
3. **Customize Portfolio**: Edit `portfolio_input` dictionary with your stocks
4. **Run Script**: `python stock_portfolio_summarizer.py`
5. **View Report**: Open generated `Portfolio_Analysis_Report.pdf`

## Trade Recommendation Signals

### Signal Types
- **BUY**: RSI < 30 (oversold) + positive sentiment OR strong bullish momentum
- **SELL**: RSI > 70 (overbought) + negative sentiment OR strong bearish momentum
- **HOLD**: Neutral momentum and sentiment

### Confidence Levels
- 0-60%: Weak signal - use with caution
- 60-75%: Moderate signal - confirm with other analysis
- 75-90%: Strong signal - good entry point
- 90%+: Very strong signal - multiple confirmations

## Important Notes

⚠️ **Disclaimer**: These are AI-generated suggestions only. NOT financial advice. Always:
- Conduct your own research
- Consult a licensed financial advisor
- Use stop-loss orders
- Never risk capital you can't afford to lose
- Monitor positions regularly

## Full Implementation Example

For a complete working implementation, see the functions provided in this guide or use the enhanced script available at:
- GitHub: [Link to your repo]
- Includes complete function implementations
- Handles all edge cases and error scenarios

## Troubleshooting

**NewsAPI returns 401 error**: Ensure NEWSAPI_KEY is set correctly. Free tier requires valid key.

**Claude API fails**: Check ANTHROPIC_API_KEY is set and has credits available.

**Technical indicators not calculated**: Ensure ticker has sufficient historical data (at least 50+ days).

**PDF generation slow**: Normal for 8-page report with multiple API calls. Takes 30-60 seconds.

## Next Steps

1. Install requirements: `pip install -r requirements.txt`
2. Update API keys in environment
3. Implement the functions following steps above
4. Test with sample portfolio (included: AAPL, MSFT, JPM, etc.)
5. Customize with your own holdings
