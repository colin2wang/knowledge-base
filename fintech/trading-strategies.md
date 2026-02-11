# Trading Strategies

Trading strategies form the core of algorithmic trading systems, defining the logic for entering and exiting positions based on market conditions, technical indicators, or fundamental factors.

## Strategy Classification

### Momentum Strategies
Momentum strategies capitalize on the tendency for assets that have performed well to continue performing well, and vice versa.

```python
import numpy as np
import pandas as pd

class MomentumStrategy:
    def __init__(self, lookback_period=20, momentum_threshold=0.05):
        self.lookback_period = lookback_period
        self.momentum_threshold = momentum_threshold
        self.positions = {}
        
    def calculate_momentum(self, prices):
        """Calculate price momentum over lookback period"""
        if len(prices) < self.lookback_period:
            return 0
            
        current_price = prices[-1]
        lookback_price = prices[-self.lookback_period]
        momentum = (current_price - lookback_price) / lookback_price
        return momentum
    
    def generate_signal(self, symbol, price_history):
        """Generate buy/sell/hold signal"""
        momentum = self.calculate_momentum(price_history)
        
        if momentum > self.momentum_threshold:
            return 'BUY'
        elif momentum < -self.momentum_threshold:
            return 'SELL'
        else:
            return 'HOLD'
    
    def update_position(self, symbol, signal, current_price, capital=10000):
        """Update position based on signal"""
        position_size = 0
        
        if signal == 'BUY' and symbol not in self.positions:
            # Calculate position size (2% of capital per position)
            position_size = (capital * 0.02) / current_price
            self.positions[symbol] = position_size
        elif signal == 'SELL' and symbol in self.positions:
            # Close position
            position_size = -self.positions[symbol]
            del self.positions[symbol]
            
        return position_size
```

### Mean Reversion Strategies
Mean reversion strategies assume that prices tend to revert to their historical average over time.

```python
class MeanReversionStrategy:
    def __init__(self, window=20, z_score_threshold=2.0):
        self.window = window
        self.z_score_threshold = z_score_threshold
        self.positions = {}
        
    def calculate_z_score(self, prices):
        """Calculate Z-score for mean reversion"""
        if len(prices) < self.window:
            return 0
            
        recent_prices = prices[-self.window:]
        mean_price = np.mean(recent_prices)
        std_price = np.std(recent_prices)
        
        if std_price == 0:
            return 0
            
        current_price = prices[-1]
        z_score = (current_price - mean_price) / std_price
        return z_score
    
    def generate_signal(self, symbol, price_history):
        """Generate mean reversion signal"""
        z_score = self.calculate_z_score(price_history)
        
        if z_score < -self.z_score_threshold:
            return 'BUY'  # Oversold - buy
        elif z_score > self.z_score_threshold:
            return 'SELL'  # Overbought - sell
        else:
            return 'HOLD'
```

### Statistical Arbitrage
Statistical arbitrage exploits pricing inefficiencies between related securities.

```python
class StatisticalArbitrage:
    def __init__(self, hedge_ratio_lookback=60, entry_zscore=2.0, exit_zscore=0.5):
        self.hedge_ratio_lookback = hedge_ratio_lookback
        self.entry_zscore = entry_zscore
        self.exit_zscore = exit_zscore
        self.positions = {}
        self.hedge_ratios = {}
        
    def calculate_hedge_ratio(self, asset1_prices, asset2_prices):
        """Calculate optimal hedge ratio using OLS regression"""
        if len(asset1_prices) < self.hedge_ratio_lookback:
            return 1.0
            
        # Simple linear regression to find hedge ratio
        x = np.array(asset2_prices[-self.hedge_ratio_lookback:])
        y = np.array(asset1_prices[-self.hedge_ratio_lookback:])
        
        # Calculate hedge ratio (slope of regression line)
        hedge_ratio = np.cov(x, y)[0, 1] / np.var(x)
        return hedge_ratio
    
    def calculate_spread_zscore(self, asset1_price, asset2_price, hedge_ratio):
        """Calculate Z-score of the spread"""
        spread = asset1_price - hedge_ratio * asset2_price
        # In practice, you'd use historical spread data for mean/std calculation
        return spread  # Simplified version
    
    def generate_pair_trading_signal(self, asset1_data, asset2_data):
        """Generate pair trading signal"""
        asset1_prices, asset1_current = asset1_data
        asset2_prices, asset2_current = asset2_data
        
        hedge_ratio = self.calculate_hedge_ratio(asset1_prices, asset2_prices)
        spread_zscore = self.calculate_spread_zscore(
            asset1_current, asset2_current, hedge_ratio
        )
        
        if spread_zscore < -self.entry_zscore:
            return 'LONG_SPREAD'  # Buy asset1, sell asset2
        elif spread_zscore > self.entry_zscore:
            return 'SHORT_SPREAD'  # Sell asset1, buy asset2
        elif abs(spread_zscore) < self.exit_zscore:
            return 'CLOSE_POSITION'
        else:
            return 'HOLD'
```

## Multi-Factor Strategies

### Factor-Based Portfolio Construction
```python
class MultiFactorStrategy:
    def __init__(self, factors_weights=None):
        if factors_weights is None:
            # Equal weighting by default
            self.factors_weights = {
                'momentum': 0.25,
                'value': 0.25,
                'quality': 0.25,
                'volatility': 0.25
            }
        else:
            self.factors_weights = factors_weights
            
    def calculate_factor_scores(self, stock_data):
        """Calculate composite factor scores for stocks"""
        factor_scores = {}
        
        for symbol, data in stock_data.items():
            scores = {}
            
            # Momentum factor (12-month return)
            scores['momentum'] = self.calculate_momentum_factor(data['prices'])
            
            # Value factor (P/E ratio inverse)
            scores['value'] = self.calculate_value_factor(data['pe_ratio'])
            
            # Quality factor (ROE)
            scores['quality'] = self.calculate_quality_factor(data['roe'])
            
            # Volatility factor (inverse of price volatility)
            scores['volatility'] = self.calculate_volatility_factor(data['prices'])
            
            # Weighted composite score
            composite_score = sum(
                scores[factor] * self.factors_weights[factor] 
                for factor in self.factors_weights
            )
            factor_scores[symbol] = composite_score
            
        return factor_scores
    
    def calculate_momentum_factor(self, prices):
        """Calculate momentum factor score"""
        if len(prices) < 252:  # Need at least 1 year of data
            return 0
        return (prices[-1] / prices[-252]) - 1
    
    def calculate_value_factor(self, pe_ratio):
        """Calculate value factor score (lower P/E is better)"""
        if pe_ratio <= 0:
            return 0
        return 1 / pe_ratio  # Inverse relationship
    
    def calculate_quality_factor(self, roe):
        """Calculate quality factor score"""
        return max(0, roe)  # Higher ROE is better
    
    def calculate_volatility_factor(self, prices):
        """Calculate volatility factor score (lower volatility is better)"""
        if len(prices) < 30:
            return 1
        returns = np.diff(np.log(prices))  # Log returns
        volatility = np.std(returns) * np.sqrt(252)  # Annualized
        return 1 / (1 + volatility)  # Inverse relationship
```

## Risk Management Integration

### Position Sizing Strategies
```python
class PositionSizer:
    def __init__(self, max_position_size=0.05, risk_per_trade=0.01):
        self.max_position_size = max_position_size  # 5% of portfolio per position
        self.risk_per_trade = risk_per_trade  # 1% risk per trade
        
    def kelly_criterion_position_size(self, win_rate, avg_win, avg_loss, account_value):
        """Calculate position size using Kelly Criterion"""
        if avg_loss == 0:
            return 0
            
        kelly_fraction = win_rate - (1 - win_rate) * (avg_loss / avg_win)
        kelly_fraction = max(0, min(kelly_fraction, 1))  # Clamp between 0 and 1
        
        position_value = account_value * kelly_fraction
        return min(position_value, account_value * self.max_position_size)
    
    def volatility_adjusted_position_size(self, asset_volatility, target_volatility, account_value):
        """Adjust position size based on asset volatility"""
        if asset_volatility == 0:
            return account_value * self.max_position_size
            
        adjustment_factor = target_volatility / asset_volatility
        position_value = account_value * self.risk_per_trade * adjustment_factor
        return min(position_value, account_value * self.max_position_size)
    
    def equal_risk_position_size(self, stop_loss_distance, account_value):
        """Equal risk position sizing"""
        if stop_loss_distance <= 0:
            return account_value * self.max_position_size
            
        position_value = (account_value * self.risk_per_trade) / stop_loss_distance
        return min(position_value, account_value * self.max_position_size)
```

### Stop-Loss and Take-Profit Implementation
```python
class TradeManager:
    def __init__(self, stop_loss_pct=0.05, take_profit_pct=0.10, trailing_stop=False):
        self.stop_loss_pct = stop_loss_pct
        self.take_profit_pct = take_profit_pct
        self.trailing_stop = trailing_stop
        self.positions = {}
        self.trade_history = []
        
    def enter_position(self, symbol, entry_price, position_size, timestamp):
        """Enter a new position"""
        stop_loss = entry_price * (1 - self.stop_loss_pct)
        take_profit = entry_price * (1 + self.take_profit_pct)
        
        self.positions[symbol] = {
            'entry_price': entry_price,
            'position_size': position_size,
            'stop_loss': stop_loss,
            'take_profit': take_profit,
            'entry_time': timestamp,
            'peak_price': entry_price
        }
        
    def update_position(self, symbol, current_price, timestamp):
        """Update position with current market data"""
        if symbol not in self.positions:
            return None
            
        position = self.positions[symbol]
        
        # Update trailing stop
        if self.trailing_stop:
            position['peak_price'] = max(position['peak_price'], current_price)
            position['stop_loss'] = position['peak_price'] * (1 - self.stop_loss_pct)
        
        # Check exit conditions
        exit_reason = None
        exit_price = None
        
        if current_price <= position['stop_loss']:
            exit_reason = 'STOP_LOSS'
            exit_price = position['stop_loss']
        elif current_price >= position['take_profit']:
            exit_reason = 'TAKE_PROFIT'
            exit_price = position['take_profit']
            
        if exit_reason:
            pnl = (exit_price - position['entry_price']) * position['position_size']
            trade_record = {
                'symbol': symbol,
                'entry_price': position['entry_price'],
                'exit_price': exit_price,
                'position_size': position['position_size'],
                'pnl': pnl,
                'exit_reason': exit_reason,
                'entry_time': position['entry_time'],
                'exit_time': timestamp
            }
            self.trade_history.append(trade_record)
            del self.positions[symbol]
            
            return trade_record
            
        return None
    
    def get_portfolio_exposure(self):
        """Calculate current portfolio exposure"""
        total_exposure = sum(pos['position_size'] * pos['entry_price'] 
                           for pos in self.positions.values())
        return total_exposure
```

## Strategy Optimization

### Parameter Optimization
```python
class StrategyOptimizer:
    def __init__(self, strategy_class, parameter_ranges):
        self.strategy_class = strategy_class
        self.parameter_ranges = parameter_ranges
        
    def grid_search_optimization(self, data, objective_function, cv_folds=5):
        """Perform grid search optimization"""
        best_params = None
        best_score = float('-inf')
        
        # Generate parameter combinations
        param_combinations = self.generate_parameter_grid()
        
        for params in param_combinations:
            # Cross-validation
            cv_scores = []
            fold_size = len(data) // cv_folds
            
            for fold in range(cv_folds):
                # Split data
                train_start = fold * fold_size
                train_end = train_start + int(fold_size * 0.8)  # 80% training
                test_start = train_end
                test_end = min(train_end + int(fold_size * 0.2), len(data))  # 20% testing
                
                train_data = data.iloc[train_start:train_end]
                test_data = data.iloc[test_start:test_end]
                
                # Test parameters
                score = self.test_parameters(train_data, test_data, params, objective_function)
                cv_scores.append(score)
            
            # Average CV score
            avg_score = np.mean(cv_scores)
            
            if avg_score > best_score:
                best_score = avg_score
                best_params = params
                
        return best_params, best_score
    
    def test_parameters(self, train_data, test_data, params, objective_function):
        """Test specific parameter combination"""
        strategy = self.strategy_class(**params)
        
        # Run strategy on training data
        train_signals = self.run_strategy(strategy, train_data)
        
        # Run strategy on test data
        test_signals = self.run_strategy(strategy, test_data)
        
        # Calculate objective score
        score = objective_function(test_signals, test_data)
        return score
```

## Performance Monitoring

### Real-time Strategy Analytics
```python
class StrategyMonitor:
    def __init__(self, strategy):
        self.strategy = strategy
        self.performance_metrics = {}
        self.drawdown_tracker = []
        
    def update_metrics(self, current_portfolio_value, benchmark_returns=None):
        """Update performance metrics"""
        # Calculate returns
        if hasattr(self, 'previous_value'):
            returns = (current_portfolio_value - self.previous_value) / self.previous_value
        else:
            returns = 0
            
        self.previous_value = current_portfolio_value
        
        # Track drawdown
        if not hasattr(self, 'peak_value'):
            self.peak_value = current_portfolio_value
            
        self.peak_value = max(self.peak_value, current_portfolio_value)
        current_drawdown = (self.peak_value - current_portfolio_value) / self.peak_value
        self.drawdown_tracker.append(current_drawdown)
        
        # Update metrics
        self.performance_metrics.update({
            'current_value': current_portfolio_value,
            'current_returns': returns,
            'max_drawdown': max(self.drawdown_tracker),
            'current_drawdown': current_drawdown,
            'volatility': np.std(self.get_recent_returns()) * np.sqrt(252),
            'sharpe_ratio': self.calculate_sharpe_ratio()
        })
        
        return self.performance_metrics
    
    def calculate_sharpe_ratio(self, risk_free_rate=0.02):
        """Calculate Sharpe ratio"""
        returns = self.get_recent_returns()
        if len(returns) < 2:
            return 0
            
        excess_returns = np.array(returns) - risk_free_rate/252
        return np.mean(excess_returns) / np.std(excess_returns) * np.sqrt(252)
    
    def get_recent_returns(self, lookback=30):
        """Get recent returns for analysis"""
        # This would typically pull from a returns history
        return [0.001] * min(lookback, len(getattr(self, 'return_history', [])))
```

## Strategy Implementation Best Practices

### Code Structure Guidelines
1. **Modular Design**: Separate signal generation, risk management, and execution
2. **Parameter Validation**: Ensure parameters are within reasonable bounds
3. **Error Handling**: Robust exception handling for market data issues
4. **Logging**: Comprehensive logging for debugging and monitoring
5. **Backward Compatibility**: Maintain compatibility when updating strategies

### Testing Framework
```python
def test_strategy_robustness(strategy, test_data):
    """Comprehensive strategy testing"""
    test_results = {
        'basic_functionality': test_basic_operations(strategy, test_data),
        'edge_cases': test_edge_cases(strategy),
        'performance_stress': test_performance_under_stress(strategy, test_data),
        'parameter_sensitivity': test_parameter_sensitivity(strategy, test_data)
    }
    return test_results

def test_basic_operations(strategy, data):
    """Test basic strategy operations"""
    try:
        signals = strategy.generate_signals(data)
        return len(signals) > 0 and all(isinstance(s, str) for s in signals.values())
    except Exception as e:
        return False

def test_edge_cases(strategy):
    """Test edge cases"""
    edge_cases = [
        # Empty data
        [],
        # Single data point
        [100],
        # Extreme values
        [100, 10000, 0.01],
        # Missing data
        [100, None, 105]
    ]
    
    results = []
    for case in edge_cases:
        try:
            result = strategy.handle_edge_case(case)
            results.append(result is not None)
        except:
            results.append(False)
            
    return all(results)
```

This comprehensive trading strategies documentation covers the fundamental approaches, implementation examples, risk management integration, and best practices for developing robust algorithmic trading strategies.