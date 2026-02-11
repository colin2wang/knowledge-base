# Financial Data Sources

Comprehensive guide to financial market data sources, APIs, and data acquisition methods for quantitative finance applications.

## Overview

Financial data is the foundation of quantitative analysis and algorithmic trading. The quality, frequency, and reliability of data sources directly impact strategy performance and risk management effectiveness.

## Major Data Providers

### Bloomberg Terminal
Bloomberg Terminal is the industry standard for professional financial data and analytics.

```python
# Bloomberg API Example
import blpapi

class BloombergDataFetcher:
    def __init__(self):
        self.session = blpapi.Session()
        self.session.start()
        
    def get_historical_data(self, security, fields, start_date, end_date):
        """Fetch historical data from Bloomberg"""
        request = self.session.createRequest("HistoricalDataRequest")
        
        request.getElement("securities").appendValue(security)
        for field in fields:
            request.getElement("fields").appendValue(field)
            
        request.set("startDate", start_date.strftime("%Y%m%d"))
        request.set("endDate", end_date.strftime("%Y%m%d"))
        request.set("periodicitySelection", "DAILY")
        
        self.session.sendRequest(request)
        
        # Process response
        data = []
        while True:
            ev = self.session.nextEvent()
            for msg in ev:
                if msg.messageType() == "HistoricalDataResponse":
                    security_data = msg.getElement("securityData")
                    field_data = security_data.getElement("fieldData")
                    
                    for i in range(field_data.numValues()):
                        row = field_data.getValue(i)
                        data_point = {
                            'date': row.getElement("date").getValue(),
                            'px_last': row.getElement("PX_LAST").getValue() if row.hasElement("PX_LAST") else None,
                            'px_open': row.getElement("PX_OPEN").getValue() if row.hasElement("PX_OPEN") else None,
                            'px_high': row.getElement("PX_HIGH").getValue() if row.hasElement("PX_HIGH") else None,
                            'px_low': row.getElement("PX_LOW").getValue() if row.hasElement("PX_LOW") else None,
                            'volume': row.getElement("VOLUME").getValue() if row.hasElement("VOLUME") else None
                        }
                        data.append(data_point)
            if ev.eventType() == blpapi.Event.RESPONSE:
                break
                
        return data
```

### Yahoo Finance
Free and accessible financial data source with extensive market coverage.

```python
import yfinance as yf
import pandas as pd

class YahooFinanceData:
    def __init__(self):
        self.cache = {}
        
    def get_stock_data(self, ticker, period="1y", interval="1d"):
        """Fetch stock data from Yahoo Finance"""
        try:
            stock = yf.Ticker(ticker)
            data = stock.history(period=period, interval=interval)
            return data
        except Exception as e:
            print(f"Error fetching data for {ticker}: {e}")
            return pd.DataFrame()
    
    def get_multiple_stocks(self, tickers, period="1y"):
        """Fetch data for multiple stocks"""
        data_dict = {}
        for ticker in tickers:
            data = self.get_stock_data(ticker, period)
            if not data.empty:
                data_dict[ticker] = data
        return data_dict
    
    def get_financial_ratios(self, ticker):
        """Get key financial ratios"""
        try:
            stock = yf.Ticker(ticker)
            info = stock.info
            
            ratios = {
                'pe_ratio': info.get('trailingPE'),
                'pb_ratio': info.get('priceToBook'),
                'dividend_yield': info.get('dividendYield'),
                'beta': info.get('beta'),
                'market_cap': info.get('marketCap'),
                'forward_pe': info.get('forwardPE'),
                'peg_ratio': info.get('pegRatio')
            }
            return ratios
        except Exception as e:
            print(f"Error fetching ratios for {ticker}: {e}")
            return {}

# Usage example
yahoo_fetcher = YahooFinanceData()
apple_data = yahoo_fetcher.get_stock_data("AAPL", period="2y")
tech_stocks = yahoo_fetcher.get_multiple_stocks(["AAPL", "GOOGL", "MSFT"])
```

### Quandl
Premium financial, economic, and alternative data platform.

```python
import quandl
import pandas as pd

class QuandlDataFetcher:
    def __init__(self, api_key=None):
        if api_key:
            quandl.ApiConfig.api_key = api_key
            
    def get_fred_data(self, series_id, start_date=None, end_date=None):
        """Fetch Federal Reserve Economic Data"""
        try:
            data = quandl.get(f"FRED/{series_id}", 
                            start_date=start_date, 
                            end_date=end_date)
            return data
        except Exception as e:
            print(f"Error fetching FRED data {series_id}: {e}")
            return pd.DataFrame()
    
    def get_wiki_prices(self, ticker, start_date=None, end_date=None):
        """Get Wikipedia stock prices dataset"""
        try:
            data = quandl.get(f"WIKI/{ticker}", 
                            start_date=start_date, 
                            end_date=end_date)
            return data
        except Exception as e:
            print(f"Error fetching WIKI data for {ticker}: {e}")
            return pd.DataFrame()
    
    def get_commodity_data(self, commodity_code):
        """Fetch commodity futures data"""
        datasets = {
            'oil': 'CHRIS/CME_CL1',      # Crude Oil
            'gold': 'CHRIS/CME_GC1',     # Gold
            'silver': 'CHRIS/CME_SI1',   # Silver
            'natural_gas': 'CHRIS/CME_NG1'  # Natural Gas
        }
        
        if commodity_code in datasets:
            try:
                data = quandl.get(datasets[commodity_code])
                return data
            except Exception as e:
                print(f"Error fetching {commodity_code} data: {e}")
                return pd.DataFrame()
        else:
            print(f"Unknown commodity code: {commodity_code}")
            return pd.DataFrame()

# Usage example
quandl_fetcher = QuandlDataFetcher("your_api_key_here")
gdp_data = quandl_fetcher.get_fred_data("GDP", start_date="2020-01-01")
oil_data = quandl_fetcher.get_commodity_data("oil")
```

## Data Quality Assessment

### Data Validation Framework
```python
class DataQualityChecker:
    def __init__(self):
        self.validation_rules = {
            'missing_data_threshold': 0.05,  # 5% missing data acceptable
            'outlier_std_threshold': 3.0,    # 3 standard deviations
            'frequency_consistency': True,
            'value_range_checks': {
                'price': (0, 1000000),      # Reasonable price range
                'volume': (0, 1000000000),   # Reasonable volume range
                'returns': (-0.5, 0.5)       # Reasonable return range
            }
        }
    
    def check_data_quality(self, data, data_type="price"):
        """Comprehensive data quality assessment"""
        quality_report = {
            'total_rows': len(data),
            'missing_data': self.check_missing_data(data),
            'outliers': self.detect_outliers(data, data_type),
            'frequency_issues': self.check_frequency_consistency(data),
            'value_anomalies': self.check_value_ranges(data, data_type),
            'overall_quality_score': 0
        }
        
        # Calculate overall quality score
        quality_score = self.calculate_quality_score(quality_report)
        quality_report['overall_quality_score'] = quality_score
        
        return quality_report
    
    def check_missing_data(self, data):
        """Check for missing data patterns"""
        missing_counts = data.isnull().sum()
        missing_percentages = (missing_counts / len(data)) * 100
        
        return {
            'missing_counts': missing_counts.to_dict(),
            'missing_percentages': missing_percentages.to_dict(),
            'total_missing': missing_counts.sum(),
            'acceptance_status': (missing_percentages <= 
                                self.validation_rules['missing_data_threshold'] * 100).all()
        }
    
    def detect_outliers(self, data, data_type):
        """Detect statistical outliers"""
        outliers = {}
        
        for column in data.select_dtypes(include=[np.number]).columns:
            series = data[column].dropna()
            if len(series) == 0:
                continue
                
            mean = series.mean()
            std = series.std()
            
            # Identify outliers using Z-score method
            z_scores = np.abs((series - mean) / std)
            outlier_mask = z_scores > self.validation_rules['outlier_std_threshold']
            outlier_indices = series[outlier_mask].index.tolist()
            
            outliers[column] = {
                'count': len(outlier_indices),
                'indices': outlier_indices[:10],  # Show first 10 outliers
                'percentage': (len(outlier_indices) / len(series)) * 100
            }
            
        return outliers
    
    def check_frequency_consistency(self, data):
        """Check data frequency consistency"""
        if not isinstance(data.index, pd.DatetimeIndex):
            return {'error': 'Index is not datetime'}
            
        # Calculate time differences
        time_diffs = data.index.to_series().diff().dropna()
        
        # Expected frequency (assuming daily data)
        expected_freq = pd.Timedelta(days=1)
        tolerance = pd.Timedelta(hours=1)
        
        inconsistent_gaps = time_diffs[
            (time_diffs < expected_freq - tolerance) | 
            (time_diffs > expected_freq + tolerance * 5)  # Allow weekends/holidays
        ]
        
        return {
            'inconsistent_gaps_count': len(inconsistent_gaps),
            'gap_details': inconsistent_gaps.head().to_dict() if len(inconsistent_gaps) > 0 else {},
            'frequency_consistent': len(inconsistent_gaps) == 0
        }
```

## Alternative Data Sources

### Social Media Sentiment Data
```python
import tweepy
import requests
from textblob import TextBlob

class SocialMediaSentiment:
    def __init__(self, twitter_credentials=None):
        if twitter_credentials:
            self.twitter_api = tweepy.API(
                tweepy.OAuthHandler(
                    twitter_credentials['consumer_key'],
                    twitter_credentials['consumer_secret']
                )
            )
    
    def get_twitter_sentiment(self, query, count=100):
        """Analyze Twitter sentiment for financial topics"""
        try:
            tweets = tweepy.Cursor(self.twitter_api.search_tweets, 
                                 q=query, 
                                 lang="en",
                                 result_type="recent").items(count)
            
            sentiments = []
            for tweet in tweets:
                analysis = TextBlob(tweet.text)
                sentiments.append({
                    'text': tweet.text,
                    'sentiment_score': analysis.sentiment.polarity,
                    'subjectivity': analysis.sentiment.subjectivity,
                    'created_at': tweet.created_at,
                    'user_followers': tweet.user.followers_count
                })
            
            return self.aggregate_sentiment(sentiments)
        except Exception as e:
            print(f"Error fetching Twitter data: {e}")
            return {}
    
    def aggregate_sentiment(self, sentiments):
        """Aggregate individual sentiments into overall score"""
        if not sentiments:
            return {'overall_sentiment': 0, 'confidence': 0}
            
        # Weight sentiment by follower count
        total_weight = sum(s['user_followers'] for s in sentiments)
        weighted_sentiment = sum(s['sentiment_score'] * s['user_followers'] 
                               for s in sentiments) / total_weight
        
        # Calculate confidence based on sample size and agreement
        sentiment_values = [s['sentiment_score'] for s in sentiments]
        agreement = 1 - np.std(sentiment_values)  # Higher std = lower agreement
        
        return {
            'overall_sentiment': weighted_sentiment,
            'confidence': agreement * min(len(sentiments) / 100, 1),
            'sample_size': len(sentiments),
            'positive_count': sum(1 for s in sentiments if s['sentiment_score'] > 0),
            'negative_count': sum(1 for s in sentiments if s['sentiment_score'] < 0),
            'neutral_count': sum(1 for s in sentiments if s['sentiment_score'] == 0)
        }
```

### News and Event Data
```python
class FinancialNewsAnalyzer:
    def __init__(self):
        self.news_sources = {
            'reuters': 'https://www.reuters.com/',
            'bloomberg': 'https://www.bloomberg.com/',
            'wsj': 'https://www.wsj.com/',
            'cnbc': 'https://www.cnbc.com/'
        }
    
    def fetch_news_headlines(self, ticker, days_back=7):
        """Fetch recent financial news headlines"""
        # This would integrate with news APIs like Alpha Vantage, NewsAPI, etc.
        pass
    
    def extract_market_events(self, news_text):
        """Extract market-moving events from news text"""
        event_keywords = {
            'earnings': ['earnings', 'quarterly results', 'profit', 'loss'],
            'merger': ['merger', 'acquisition', 'takeover', 'buyout'],
            'regulatory': ['SEC', 'regulation', 'compliance', 'fine'],
            'economic': ['GDP', 'inflation', 'interest rate', 'unemployment']
        }
        
        detected_events = []
        text_lower = news_text.lower()
        
        for event_type, keywords in event_keywords.items():
            if any(keyword in text_lower for keyword in keywords):
                detected_events.append(event_type)
                
        return detected_events
```

## Data Storage and Management

### Database Integration
```python
import sqlite3
import pandas as pd
from sqlalchemy import create_engine

class FinancialDataStorage:
    def __init__(self, db_path="financial_data.db"):
        self.engine = create_engine(f'sqlite:///{db_path}')
        self.init_database()
        
    def init_database(self):
        """Initialize database tables"""
        create_tables_sql = """
        CREATE TABLE IF NOT EXISTS stock_prices (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            symbol TEXT NOT NULL,
            date DATE NOT NULL,
            open REAL,
            high REAL,
            low REAL,
            close REAL,
            volume INTEGER,
            adjusted_close REAL,
            UNIQUE(symbol, date)
        );
        
        CREATE TABLE IF NOT EXISTS financial_ratios (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            symbol TEXT NOT NULL,
            date DATE NOT NULL,
            pe_ratio REAL,
            pb_ratio REAL,
            dividend_yield REAL,
            beta REAL,
            market_cap REAL,
            UNIQUE(symbol, date)
        );
        
        CREATE INDEX IF NOT EXISTS idx_stock_symbol ON stock_prices(symbol);
        CREATE INDEX IF NOT EXISTS idx_stock_date ON stock_prices(date);
        """
        
        with self.engine.connect() as conn:
            conn.execute(create_tables_sql)
            conn.commit()
    
    def store_stock_data(self, symbol, data_df):
        """Store stock price data"""
        data_df['symbol'] = symbol
        data_df.to_sql('stock_prices', self.engine, 
                      if_exists='append', index=True, 
                      index_label='date', method='multi')
    
    def get_stock_data(self, symbol, start_date=None, end_date=None):
        """Retrieve stored stock data"""
        query = "SELECT * FROM stock_prices WHERE symbol = ?"
        params = [symbol]
        
        if start_date:
            query += " AND date >= ?"
            params.append(start_date)
        if end_date:
            query += " AND date <= ?"
            params.append(end_date)
            
        query += " ORDER BY date"
        
        return pd.read_sql_query(query, self.engine, params=params, 
                               parse_dates=['date'], index_col='date')
```

## Data Pipeline Automation

### Scheduled Data Fetching
```python
import schedule
import time
from datetime import datetime, timedelta

class DataPipeline:
    def __init__(self):
        self.data_fetchers = {
            'yahoo': YahooFinanceData(),
            'quandl': QuandlDataFetcher(),
            'storage': FinancialDataStorage()
        }
        self.watchlist = ['AAPL', 'GOOGL', 'MSFT', 'AMZN', 'TSLA']
        
    def daily_data_update(self):
        """Daily data update routine"""
        print(f"Starting daily update at {datetime.now()}")
        
        for symbol in self.watchlist:
            try:
                # Fetch latest data
                new_data = self.data_fetchers['yahoo'].get_stock_data(
                    symbol, period="2mo", interval="1d"
                )
                
                if not new_data.empty:
                    # Store in database
                    self.data_fetchers['storage'].store_stock_data(symbol, new_data)
                    print(f"Updated data for {symbol}")
                    
            except Exception as e:
                print(f"Error updating {symbol}: {e}")
        
        print("Daily update completed")
    
    def start_scheduler(self):
        """Start automated scheduling"""
        # Run daily at market close (4 PM EST)
        schedule.every().day.at("16:00").do(self.daily_data_update)
        
        # Weekly fundamental data update
        schedule.every().monday.at("09:00").do(self.weekly_update)
        
        print("Data pipeline scheduler started")
        while True:
            schedule.run_pending()
            time.sleep(60)  # Check every minute
    
    def weekly_update(self):
        """Weekly fundamental data update"""
        print(f"Weekly update started at {datetime.now()}")
        for symbol in self.watchlist:
            try:
                ratios = self.data_fetchers['yahoo'].get_financial_ratios(symbol)
                # Store ratios in database
                print(f"Updated ratios for {symbol}")
            except Exception as e:
                print(f"Error updating ratios for {symbol}: {e}")

# Usage
# pipeline = DataPipeline()
# pipeline.start_scheduler()  # Run in separate thread/process
```

This comprehensive data sources documentation covers major financial data providers, quality assessment frameworks, alternative data integration, storage solutions, and automated data pipelines essential for quantitative finance applications.