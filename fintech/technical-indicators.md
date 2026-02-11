# Technical Indicators

Comprehensive guide to technical analysis indicators used in quantitative trading and algorithmic strategies.

## Overview

Technical indicators are mathematical calculations based on price, volume, or open interest data that help traders identify trends, momentum, volatility, and potential reversal points in financial markets.

## Trend Indicators

### Moving Averages
Moving averages smooth price data to identify trend direction and potential support/resistance levels.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

class MovingAverages:
    def __init__(self):
        pass
    
    def simple_moving_average(self, prices, window):
        """Calculate Simple Moving Average"""
        return pd.Series(prices).rolling(window=window).mean()
    
    def exponential_moving_average(self, prices, span):
        """Calculate Exponential Moving Average"""
        return pd.Series(prices).ewm(span=span, adjust=False).mean()
    
    def weighted_moving_average(self, prices, weights):
        """Calculate Weighted Moving Average"""
        weights = np.array(weights)
        weights = weights / weights.sum()  # Normalize weights
        
        wma = []
        for i in range(len(weights) - 1, len(prices)):
            window = prices[i - len(weights) + 1:i + 1]
            wma_value = np.sum(window * weights)
            wma.append(wma_value)
            
        # Pad with NaN for initial values
        wma = [np.nan] * (len(weights) - 1) + wma
        return pd.Series(wma, index=prices.index if hasattr(prices, 'index') else None)
    
    def hull_moving_average(self, prices, window):
        """Calculate Hull Moving Average - reduces lag"""
        half_length = window // 2
        sqrt_length = int(np.sqrt(window))
        
        # Calculate weighted moving averages
        wma_half = self.weighted_moving_average(prices, list(range(1, half_length + 1)))
        wma_full = self.weighted_moving_average(prices, list(range(1, window + 1)))
        
        # Difference series
        diff_series = 2 * wma_half - wma_full
        
        # Final HMA
        hma = self.weighted_moving_average(diff_series, list(range(1, sqrt_length + 1)))
        return hma
    
    def moving_average_crossover_signal(self, prices, fast_window=10, slow_window=30):
        """Generate signals based on moving average crossovers"""
        fast_ma = self.exponential_moving_average(prices, fast_window)
        slow_ma = self.exponential_moving_average(prices, slow_window)
        
        signals = pd.Series(0, index=prices.index)
        signals[fast_ma > slow_ma] = 1   # Bullish signal
        signals[fast_ma < slow_ma] = -1  # Bearish signal
        
        return signals, fast_ma, slow_ma

# Example usage
ma_calculator = MovingAverages()
prices = pd.Series([100, 102, 101, 103, 105, 104, 106, 108, 107, 109])

sma = ma_calculator.simple_moving_average(prices, 5)
ema = ma_calculator.exponential_moving_average(prices, 5)
signals, fast_ma, slow_ma = ma_calculator.moving_average_crossover_signal(prices)
```

### MACD (Moving Average Convergence Divergence)
MACD identifies trend changes and momentum shifts.

```python
class MACDIndicator:
    def __init__(self, fast_period=12, slow_period=26, signal_period=9):
        self.fast_period = fast_period
        self.slow_period = slow_period
        self.signal_period = signal_period
    
    def calculate_macd(self, prices):
        """Calculate MACD line"""
        ema_fast = pd.Series(prices).ewm(span=self.fast_period, adjust=False).mean()
        ema_slow = pd.Series(prices).ewm(span=self.slow_period, adjust=False).mean()
        macd_line = ema_fast - ema_slow
        return macd_line
    
    def calculate_signal_line(self, macd_line):
        """Calculate MACD signal line"""
        signal_line = macd_line.ewm(span=self.signal_period, adjust=False).mean()
        return signal_line
    
    def calculate_histogram(self, macd_line, signal_line):
        """Calculate MACD histogram"""
        histogram = macd_line - signal_line
        return histogram
    
    def generate_signals(self, prices):
        """Generate buy/sell signals"""
        macd_line = self.calculate_macd(prices)
        signal_line = self.calculate_signal_line(macd_line)
        histogram = self.calculate_histogram(macd_line, signal_line)
        
        # Generate signals
        signals = pd.Series(0, index=prices.index)
        
        # Bullish crossover
        crossover_up = (macd_line > signal_line) & (macd_line.shift(1) <= signal_line.shift(1))
        signals[crossover_up] = 1
        
        # Bearish crossover
        crossover_down = (macd_line < signal_line) & (macd_line.shift(1) >= signal_line.shift(1))
        signals[crossover_down] = -1
        
        return signals, macd_line, signal_line, histogram

# Example usage
macd_indicator = MACDIndicator()
prices_sample = pd.Series(np.random.randn(100).cumsum() + 100)  # Simulated price data
signals, macd_line, signal_line, histogram = macd_indicator.generate_signals(prices_sample)
```

## Momentum Indicators

### Relative Strength Index (RSI)
RSI measures the speed and change of price movements to identify overbought/oversold conditions.

```python
class RSIIndicator:
    def __init__(self, period=14):
        self.period = period
    
    def calculate_rsi(self, prices):
        """Calculate RSI indicator"""
        # Calculate price changes
        delta = prices.diff()
        
        # Separate gains and losses
        gain = delta.where(delta > 0, 0)
        loss = -delta.where(delta < 0, 0)
        
        # Calculate average gains and losses
        avg_gain = gain.rolling(window=self.period).mean()
        avg_loss = loss.rolling(window=self.period).mean()
        
        # Calculate RS and RSI
        rs = avg_gain / avg_loss
        rsi = 100 - (100 / (1 + rs))
        
        return rsi
    
    def calculate_smoothed_rsi(self, prices):
        """Calculate smoothed RSI using EMA"""
        delta = prices.diff()
        gain = delta.where(delta > 0, 0)
        loss = -delta.where(delta < 0, 0)
        
        # Use EMA for smoothing
        avg_gain = gain.ewm(alpha=1/self.period, adjust=False).mean()
        avg_loss = loss.ewm(alpha=1/self.period, adjust=False).mean()
        
        rs = avg_gain / avg_loss
        rsi = 100 - (100 / (1 + rs))
        
        return rsi
    
    def generate_rsi_signals(self, prices, overbought=70, oversold=30):
        """Generate trading signals based on RSI levels"""
        rsi = self.calculate_smoothed_rsi(prices)
        
        signals = pd.Series(0, index=prices.index)
        signals[rsi < oversold] = 1    # Buy signal (oversold)
        signals[rsi > overbought] = -1 # Sell signal (overbought)
        
        return signals, rsi

# Example usage
rsi_indicator = RSIIndicator()
sample_prices = pd.Series([50, 52, 51, 53, 55, 54, 56, 58, 57, 55, 53, 51, 49, 50, 52])
rsi_signals, rsi_values = rsi_indicator.generate_rsi_signals(sample_prices)
```

### Stochastic Oscillator
Stochastic oscillator compares a particular closing price to a range of prices over a specific period.

```python
class StochasticOscillator:
    def __init__(self, k_period=14, d_period=3):
        self.k_period = k_period
        self.d_period = d_period
    
    def calculate_stochastic(self, high_prices, low_prices, close_prices):
        """Calculate %K and %D lines"""
        # Calculate %K
        lowest_low = low_prices.rolling(window=self.k_period).min()
        highest_high = high_prices.rolling(window=self.k_period).max()
        
        k_percent = 100 * (close_prices - lowest_low) / (highest_high - lowest_low)
        
        # Calculate %D (signal line)
        d_percent = k_percent.rolling(window=self.d_period).mean()
        
        return k_percent, d_percent
    
    def generate_signals(self, high_prices, low_prices, close_prices, 
                        overbought=80, oversold=20):
        """Generate buy/sell signals"""
        k_percent, d_percent = self.calculate_stochastic(high_prices, low_prices, close_prices)
        
        signals = pd.Series(0, index=close_prices.index)
        
        # Bullish signals
        bullish_crossover = (k_percent > d_percent) & (k_percent.shift(1) <= d_percent.shift(1))
        oversold_reversal = (k_percent < oversold) & (k_percent > k_percent.shift(1))
        signals[bullish_crossover | oversold_reversal] = 1
        
        # Bearish signals
        bearish_crossover = (k_percent < d_percent) & (k_percent.shift(1) >= d_percent.shift(1))
        overbought_reversal = (k_percent > overbought) & (k_percent < k_percent.shift(1))
        signals[bearish_crossover | overbought_reversal] = -1
        
        return signals, k_percent, d_percent
```

## Volatility Indicators

### Bollinger Bands
Bollinger Bands consist of a moving average and upper/lower bands that adapt to market volatility.

```python
class BollingerBands:
    def __init__(self, period=20, num_std=2):
        self.period = period
        self.num_std = num_std
    
    def calculate_bollinger_bands(self, prices):
        """Calculate Bollinger Bands"""
        # Middle band (SMA)
        middle_band = prices.rolling(window=self.period).mean()
        
        # Standard deviation
        std_dev = prices.rolling(window=self.period).std()
        
        # Upper and lower bands
        upper_band = middle_band + (std_dev * self.num_std)
        lower_band = middle_band - (std_dev * self.num_std)
        
        return middle_band, upper_band, lower_band
    
    def calculate_bandwidth(self, upper_band, lower_band, middle_band):
        """Calculate bandwidth indicator"""
        bandwidth = (upper_band - lower_band) / middle_band
        return bandwidth
    
    def calculate_percent_b(self, prices, upper_band, lower_band):
        """Calculate %B indicator"""
        percent_b = (prices - lower_band) / (upper_band - lower_band)
        return percent_b
    
    def generate_bb_signals(self, prices):
        """Generate signals based on Bollinger Bands"""
        middle_band, upper_band, lower_band = self.calculate_bollinger_bands(prices)
        percent_b = self.calculate_percent_b(prices, upper_band, lower_band)
        
        signals = pd.Series(0, index=prices.index)
        
        # Buy signals
        signals[percent_b < 0] = 1      # Price below lower band
        signals[(percent_b > 0) & (percent_b < 0.1) & 
                (percent_b > percent_b.shift(1))] = 1  # Bounce from lower area
        
        # Sell signals
        signals[percent_b > 1] = -1     # Price above upper band
        signals[(percent_b < 1) & (percent_b > 0.9) & 
                (percent_b < percent_b.shift(1))] = -1  # Pullback from upper area
        
        return signals, middle_band, upper_band, lower_band, percent_b

# Example usage
bb_indicator = BollingerBands()
sample_data = pd.Series(np.random.randn(50).cumsum() + 100)
signals, mb, ub, lb, percent_b = bb_indicator.generate_bb_signals(sample_data)
```

### Average True Range (ATR)
ATR measures market volatility by decomposing the entire range of an asset price for a given period.

```python
class ATRIndicator:
    def __init__(self, period=14):
        self.period = period
    
    def calculate_true_range(self, high, low, close):
        """Calculate True Range"""
        tr1 = high - low
        tr2 = abs(high - close.shift(1))
        tr3 = abs(low - close.shift(1))
        true_range = pd.concat([tr1, tr2, tr3], axis=1).max(axis=1)
        return true_range
    
    def calculate_atr(self, high, low, close):
        """Calculate Average True Range"""
        true_range = self.calculate_true_range(high, low, close)
        atr = true_range.rolling(window=self.period).mean()
        return atr
    
    def calculate_normalized_atr(self, high, low, close):
        """Calculate normalized ATR (ATR as percentage of price)"""
        atr = self.calculate_atr(high, low, close)
        normalized_atr = (atr / close) * 100
        return normalized_atr

# Example usage
atr_indicator = ATRIndicator()
# Assuming you have high, low, close price series
# atr_values = atr_indicator.calculate_atr(high_prices, low_prices, close_prices)
```

## Volume Indicators

### On-Balance Volume (OBV)
OBV uses volume flow to predict changes in stock price.

```python
class OBVIndicator:
    def __init__(self):
        pass
    
    def calculate_obv(self, close_prices, volumes):
        """Calculate On-Balance Volume"""
        obv = pd.Series(0, index=close_prices.index)
        
        for i in range(1, len(close_prices)):
            if close_prices.iloc[i] > close_prices.iloc[i-1]:
                obv.iloc[i] = obv.iloc[i-1] + volumes.iloc[i]
            elif close_prices.iloc[i] < close_prices.iloc[i-1]:
                obv.iloc[i] = obv.iloc[i-1] - volumes.iloc[i]
            else:
                obv.iloc[i] = obv.iloc[i-1]
                
        return obv
    
    def calculate_obv_signal(self, close_prices, volumes, signal_period=10):
        """Generate signals based on OBV divergence"""
        obv = self.calculate_obv(close_prices, volumes)
        
        # Calculate OBV moving average
        obv_ma = obv.rolling(window=signal_period).mean()
        
        signals = pd.Series(0, index=close_prices.index)
        
        # Bullish divergence
        bullish_div = (obv > obv_ma) & (close_prices < close_prices.rolling(window=signal_period).mean())
        signals[bullish_div] = 1
        
        # Bearish divergence
        bearish_div = (obv < obv_ma) & (close_prices > close_prices.rolling(window=signal_period).mean())
        signals[bearish_div] = -1
        
        return signals, obv, obv_ma
```

## Multi-Indicator Strategy

### Combined Indicator System
```python
class MultiIndicatorStrategy:
    def __init__(self):
        self.indicators = {
            'ma': MovingAverages(),
            'rsi': RSIIndicator(),
            'macd': MACDIndicator(),
            'bb': BollingerBands()
        }
        self.weights = {
            'trend': 0.4,
            'momentum': 0.3,
            'volatility': 0.3
        }
    
    def calculate_composite_signal(self, ohlcv_data):
        """Calculate composite signal from multiple indicators"""
        close = ohlcv_data['close']
        high = ohlcv_data['high']
        low = ohlcv_data['low']
        volume = ohlcv_data['volume']
        
        # Calculate individual signals
        ma_signals, _, _ = self.indicators['ma'].moving_average_crossover_signal(close)
        rsi_signals, _ = self.indicators['rsi'].generate_rsi_signals(close)
        macd_signals, _, _, _ = self.indicators['macd'].generate_signals(close)
        bb_signals, _, _, _, _ = self.indicators['bb'].generate_bb_signals(close)
        
        # Weighted combination
        composite_signal = (
            ma_signals * self.weights['trend'] +
            rsi_signals * self.weights['momentum'] +
            macd_signals * self.weights['momentum'] * 0.5 +
            bb_signals * self.weights['volatility']
        )
        
        # Normalize to -1, 0, 1
        composite_signal = np.sign(composite_signal)
        
        return composite_signal
    
    def backtest_strategy(self, ohlcv_data, initial_capital=100000):
        """Backtest the multi-indicator strategy"""
        signals = self.calculate_composite_signal(ohlcv_data)
        close_prices = ohlcv_data['close']
        
        # Simple position tracking
        positions = pd.Series(0, index=close_prices.index)
        portfolio_value = pd.Series(initial_capital, index=close_prices.index)
        
        current_position = 0
        cash = initial_capital
        
        for i in range(1, len(signals)):
            signal = signals.iloc[i]
            price = close_prices.iloc[i]
            
            if signal == 1 and current_position == 0:  # Buy signal
                shares_to_buy = int(cash / price)
                if shares_to_buy > 0:
                    cash -= shares_to_buy * price
                    current_position = shares_to_buy
                    
            elif signal == -1 and current_position > 0:  # Sell signal
                cash += current_position * price
                current_position = 0
            
            positions.iloc[i] = current_position
            portfolio_value.iloc[i] = cash + current_position * price
        
        return {
            'signals': signals,
            'positions': positions,
            'portfolio_value': portfolio_value,
            'total_return': (portfolio_value.iloc[-1] - initial_capital) / initial_capital
        }

# Example usage with sample data
# strategy = MultiIndicatorStrategy()
# results = strategy.backtest_strategy(sample_ohlcv_data)
```

This comprehensive technical indicators documentation covers the most widely used indicators in quantitative trading, including implementation examples, signal generation methods, and practical applications for algorithmic strategies.