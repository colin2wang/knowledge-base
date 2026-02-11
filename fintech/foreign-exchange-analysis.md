# Foreign Exchange Analysis

Comprehensive guide to foreign exchange markets, currency pair analysis, and carry trade strategies for quantitative forex trading.

## Overview

Foreign exchange (FX) analysis involves studying currency pairs, exchange rate dynamics, and international monetary policy to identify trading opportunities and manage currency risk.

## Currency Pair Fundamentals

### Exchange Rate Models
```python
import numpy as np
import pandas as pd
from datetime import datetime, timedelta
import matplotlib.pyplot as plt

class ExchangeRateAnalyzer:
    def __init__(self):
        self.currency_pairs = ['EURUSD', 'GBPUSD', 'USDJPY', 'AUDUSD', 'USDCAD', 'USDCHF']
        
    def purchasing_power_parity(self, domestic_price_level, foreign_price_level, base_rate):
        """Calculate exchange rate using PPP model"""
        ppp_rate = base_rate * (domestic_price_level / foreign_price_level)
        return ppp_rate
    
    def interest_rate_parity(self, domestic_rate, foreign_rate, spot_rate, time_to_maturity):
        """Calculate forward rate using IRP"""
        forward_rate = spot_rate * (1 + domestic_rate * time_to_maturity) / (1 + foreign_rate * time_to_maturity)
        return forward_rate
    
    def calculate_real_exchange_rate(self, nominal_rate, domestic_cpi, foreign_cpi):
        """Calculate real exchange rate"""
        real_rate = nominal_rate * (foreign_cpi / domestic_cpi)
        return real_rate
    
    def analyze_currency_fundamentals(self, economic_data):
        """Analyze fundamental factors affecting currency pairs"""
        analysis = {
            'interest_rate_differential': self.calculate_ir_differential(economic_data),
            'inflation_differential': self.calculate_inflation_differential(economic_data),
            'gdp_growth_differential': self.calculate_gdp_differential(economic_data),
            'current_account_balance': self.analyze_current_account(economic_data),
            'political_risk_index': self.assess_political_risk(economic_data)
        }
        return analysis
    
    def calculate_ir_differential(self, data):
        """Calculate interest rate differential between currency pair"""
        domestic_rate = data['domestic_interest_rate']
        foreign_rate = data['foreign_interest_rate']
        return domestic_rate - foreign_rate
    
    def calculate_inflation_differential(self, data):
        """Calculate inflation rate differential"""
        domestic_inflation = data['domestic_inflation']
        foreign_inflation = data['foreign_inflation']
        return domestic_inflation - foreign_inflation
    
    def calculate_gdp_differential(self, data):
        """Calculate GDP growth differential"""
        domestic_gdp = data['domestic_gdp_growth']
        foreign_gdp = data['foreign_gdp_growth']
        return domestic_gdp - foreign_gdp

# Example usage
fx_analyzer = ExchangeRateAnalyzer()
# fundamentals = fx_analyzer.analyze_currency_fundamentals(economic_data)
```

### Technical Analysis for FX
```python
class FXTechincalAnalyzer:
    def __init__(self):
        pass
    
    def fibonacci_retracement(self, high_price, low_price, retracement_levels=[0.236, 0.382, 0.5, 0.618, 0.786]):
        """Calculate Fibonacci retracement levels"""
        price_range = high_price - low_price
        
        retracement_levels_dict = {}
        for level in retracement_levels:
            retracement_price = high_price - (price_range * level)
            retracement_levels_dict[f'{level*100}%'] = retracement_price
            
        return retracement_levels_dict
    
    def moving_average_envelope(self, prices, ma_period=20, envelope_width=0.02):
        """Calculate moving average envelope for support/resistance"""
        ma = prices.rolling(window=ma_period).mean()
        upper_band = ma * (1 + envelope_width)
        lower_band = ma * (1 - envelope_width)
        
        return {
            'moving_average': ma,
            'upper_envelope': upper_band,
            'lower_envelope': lower_band
        }
    
    def rsi_currency_strength(self, currency_prices, period=14):
        """Calculate relative currency strength using RSI"""
        strength_scores = {}
        
        for currency, prices in currency_prices.items():
            # Calculate returns
            returns = prices.pct_change().dropna()
            
            # Calculate RSI
            avg_gain = returns[returns > 0].rolling(period).mean()
            avg_loss = abs(returns[returns < 0]).rolling(period).mean()
            
            rs = avg_gain / avg_loss
            rsi = 100 - (100 / (1 + rs))
            
            # Convert RSI to strength score (-1 to 1)
            strength_scores[currency] = (rsi - 50) / 50
        
        return strength_scores
    
    def correlation_analysis(self, currency_pairs_data, window=60):
        """Analyze currency pair correlations"""
        # Calculate returns for all pairs
        returns_data = {}
        for pair, prices in currency_pairs_data.items():
            returns_data[pair] = prices.pct_change().dropna()
        
        # Create correlation matrix
        returns_df = pd.DataFrame(returns_data)
        correlation_matrix = returns_df.rolling(window).corr().iloc[-1]
        
        return correlation_matrix
    
    def volatility_clustering(self, currency_returns, lookback_period=30):
        """Analyze volatility clustering in FX markets"""
        # Calculate rolling volatility
        rolling_vol = currency_returns.rolling(lookback_period).std()
        
        # Measure volatility clustering
        volatility_changes = rolling_vol.diff().abs()
        clustering_measure = volatility_changes.rolling(lookback_period).mean()
        
        return {
            'current_volatility': rolling_vol.iloc[-1],
            'volatility_trend': rolling_vol.diff().iloc[-1],
            'clustering_intensity': clustering_measure.iloc[-1]
        }

# Example usage
tech_analyzer = FXTechincalAnalyzer()
# fib_levels = tech_analyzer.fibonacci_retracement(high_price, low_price)
# ma_envelope = tech_analyzer.moving_average_envelope(price_data)
```

## Carry Trade Strategies

### Classic Carry Trade Implementation
```python
class CarryTradeStrategy:
    def __init__(self):
        self.positions = {}
        
    def identify_carry_trade_opportunities(self, interest_rates, volatilities, correlations):
        """Identify optimal carry trade currency pairs"""
        opportunities = []
        
        # Calculate carry trade metrics for each currency pair
        for base_currency in interest_rates:
            for quote_currency in interest_rates:
                if base_currency != quote_currency:
                    # Interest rate differential
                    ir_differential = interest_rates[base_currency] - interest_rates[quote_currency]
                    
                    # Volatility adjustment
                    vol_adjustment = 1 / (volatilities[base_currency] + volatilities[quote_currency])
                    
                    # Correlation adjustment
                    correlation_penalty = 1 - abs(correlations.get((base_currency, quote_currency), 0))
                    
                    # Composite score
                    carry_score = ir_differential * vol_adjustment * correlation_penalty
                    
                    opportunities.append({
                        'pair': f'{base_currency}{quote_currency}',
                        'base_currency': base_currency,
                        'quote_currency': quote_currency,
                        'interest_rate_differential': ir_differential,
                        'carry_score': carry_score,
                        'volatility': volatilities[base_currency] + volatilities[quote_currency],
                        'correlation_risk': correlations.get((base_currency, quote_currency), 0)
                    })
        
        # Sort by carry score
        opportunities.sort(key=lambda x: x['carry_score'], reverse=True)
        return opportunities[:10]  # Top 10 opportunities
    
    def calculate_carry_return(self, position_size, interest_differential, holding_period_days):
        """Calculate expected carry return"""
        daily_carry = position_size * interest_differential / 365
        total_carry = daily_carry * holding_period_days
        return total_carry
    
    def implement_carry_trade(self, pair, position_size, leverage=10, stop_loss_pips=100):
        """Implement carry trade with risk management"""
        # Calculate position metrics
        pip_value = 0.0001  # Standard pip value
        stop_loss_distance = stop_loss_pips * pip_value
        
        # Margin requirement
        margin_required = position_size / leverage
        
        # Risk metrics
        max_loss = position_size * stop_loss_distance
        risk_reward_ratio = self.calculate_carry_potential(pair) / max_loss
        
        return {
            'pair': pair,
            'position_size': position_size,
            'leverage': leverage,
            'margin_required': margin_required,
            'stop_loss_pips': stop_loss_pips,
            'max_loss': max_loss,
            'risk_reward_ratio': risk_reward_ratio,
            'daily_carry': self.calculate_daily_carry(pair, position_size)
        }
    
    def calculate_daily_carry(self, pair, position_size):
        """Calculate daily carry income for position"""
        # This would use actual interest rate data for the currency pair
        base_ir = 0.03  # Example: 3% annual
        quote_ir = 0.01  # Example: 1% annual
        daily_carry = position_size * (base_ir - quote_ir) / 365
        return daily_carry
    
    def dynamic_position_sizing(self, volatility, account_equity, risk_percent=0.02):
        """Calculate position size based on volatility"""
        # ATR-based position sizing
        atr_multiplier = 2  # 2x ATR stop loss
        position_size = (account_equity * risk_percent) / (volatility * atr_multiplier)
        return position_size
    
    def portfolio_carry_optimization(self, opportunities, portfolio_value, max_positions=5):
        """Optimize carry trade portfolio allocation"""
        # Select top opportunities
        selected_opportunities = opportunities[:max_positions]
        
        # Equal risk weighting
        position_risk = portfolio_value * 0.02  # 2% risk per position
        
        portfolio_allocation = {}
        for opp in selected_opportunities:
            pair = opp['pair']
            volatility = opp['volatility']
            
            # Calculate position size
            position_size = self.dynamic_position_sizing(volatility, portfolio_value / max_positions)
            
            portfolio_allocation[pair] = {
                'position_size': position_size,
                'expected_daily_carry': self.calculate_daily_carry(pair, position_size),
                'risk_metrics': self.calculate_position_risk(opp, position_size)
            }
        
        return portfolio_allocation
    
    def calculate_position_risk(self, opportunity, position_size):
        """Calculate comprehensive position risk metrics"""
        return {
            'value_at_risk': position_size * opportunity['volatility'] * 2,  # 2 std devs
            'max_adverse_excursion': position_size * 0.03,  # 3% max drawdown
            'correlation_exposure': abs(opportunity['correlation_risk']),
            'rollover_cost': position_size * abs(opportunity['interest_rate_differential']) / 365
        }

# Example usage
carry_strategy = CarryTradeStrategy()
# opportunities = carry_strategy.identify_carry_trade_opportunities(ir_data, vol_data, corr_data)
# portfolio = carry_strategy.portfolio_carry_optimization(opportunities, 100000)
```

## FX Risk Management

### Value at Risk for Currency Portfolios
```python
class FXRiskManager:
    def __init__(self):
        pass
    
    def calculate_fx_var(self, currency_exposures, correlation_matrix, confidence_level=0.95):
        """Calculate Value at Risk for FX portfolio"""
        # Convert exposures to vector
        exposure_vector = np.array(list(currency_exposures.values()))
        
        # Calculate portfolio variance
        portfolio_variance = exposure_vector.T @ correlation_matrix @ exposure_vector
        
        # Standard deviation
        portfolio_std = np.sqrt(portfolio_variance)
        
        # VaR calculation
        z_score = norm.ppf(confidence_level)
        var = portfolio_std * z_score
        
        return var
    
    def stress_testing(self, portfolio_exposures, stress_scenarios):
        """Perform FX stress testing"""
        stress_results = {}
        
        for scenario_name, scenario_changes in stress_scenarios.items():
            portfolio_value_change = 0
            
            for currency, exposure in portfolio_exposures.items():
                if currency in scenario_changes:
                    currency_change = scenario_changes[currency]
                    portfolio_value_change += exposure * currency_change
            
            stress_results[scenario_name] = {
                'portfolio_impact': portfolio_value_change,
                'percentage_change': portfolio_value_change / sum(portfolio_exposures.values())
            }
        
        return stress_results
    
    def calculate_currency_betas(self, currency_returns, benchmark_returns):
        """Calculate currency betas against benchmark"""
        currency_betas = {}
        
        for currency, returns in currency_returns.items():
            # Align data
            aligned_data = pd.DataFrame({'currency': returns, 'benchmark': benchmark_returns}).dropna()
            
            # Calculate beta
            covariance = np.cov(aligned_data['currency'], aligned_data['benchmark'])[0, 1]
            benchmark_variance = np.var(aligned_data['benchmark'])
            beta = covariance / benchmark_variance
            
            currency_betas[currency] = beta
            
        return currency_betas
    
    def fx_hedge_optimization(self, exposures, hedge_instruments, transaction_costs):
        """Optimize FX hedging strategy"""
        # Simplified optimization - minimize hedging cost while reducing risk
        optimal_hedges = {}
        remaining_exposure = exposures.copy()
        
        # Greedy approach: hedge largest exposures first
        sorted_exposures = sorted(remaining_exposure.items(), key=lambda x: abs(x[1]), reverse=True)
        
        for currency, exposure in sorted_exposures:
            if abs(exposure) > transaction_costs[currency]:
                # Calculate hedge amount considering transaction costs
                hedge_amount = exposure * 0.8  # Hedge 80% of exposure
                optimal_hedges[currency] = {
                    'hedge_amount': hedge_amount,
                    'hedge_cost': abs(hedge_amount) * transaction_costs[currency],
                    'remaining_risk': abs(exposure) - abs(hedge_amount)
                }
                remaining_exposure[currency] -= hedge_amount
        
        return {
            'optimal_hedges': optimal_hedges,
            'total_hedge_cost': sum(hedge['hedge_cost'] for hedge in optimal_hedges.values()),
            'risk_reduction': sum(abs(exp) - hedge.get('remaining_risk', 0) 
                                for exp, hedge in zip(exposures.values(), optimal_hedges.values()))
        }

# Example usage
fx_risk_manager = FXRiskManager()
# var = fx_risk_manager.calculate_fx_var(exposures, correlation_matrix)
# stress_results = fx_risk_manager.stress_testing(portfolio_exposures, scenarios)
```

## Cross-Currency Basis Analysis

### Basis Trading Strategies
```python
class BasisAnalyzer:
    def __init__(self):
        pass
    
    def calculate_cross_currency_basis(self, fx_forward_rate, interest_rates, spot_rate, time_to_maturity):
        """Calculate cross-currency basis"""
        # Theoretical forward rate from IRP
        theoretical_forward = spot_rate * (1 + interest_rates['domestic'] * time_to_maturity) / \
                             (1 + interest_rates['foreign'] * time_to_maturity)
        
        # Basis = Market Forward - Theoretical Forward
        basis = fx_forward_rate - theoretical_forward
        basis_points = basis / spot_rate * 10000  # Convert to basis points
        
        return {
            'basis': basis,
            'basis_points': basis_points,
            'theoretical_forward': theoretical_forward,
            'market_forward': fx_forward_rate
        }
    
    def basis_trading_opportunity(self, basis_data, transaction_costs=0.5):
        """Identify basis trading opportunities"""
        opportunities = []
        
        for currency_pair, data in basis_data.items():
            basis_bp = data['basis_points']
            
            # Consider transaction costs
            net_basis = abs(basis_bp) - transaction_costs
            
            if net_basis > 1:  # Minimum 1bp profit after costs
                opportunities.append({
                    'pair': currency_pair,
                    'basis_points': basis_bp,
                    'net_basis': net_basis,
                    'direction': 'long' if basis_bp > 0 else 'short',
                    'expected_return': net_basis / 10000  # Annualized
                })
        
        return sorted(opportunities, key=lambda x: x['net_basis'], reverse=True)
    
    def covered_interest_arbitrage(self, spot_rate, forward_rate, domestic_rate, foreign_rate, amount):
        """Calculate covered interest arbitrage profit"""
        # Strategy 1: Borrow domestic, invest foreign
        borrow_cost = amount * domestic_rate
        foreign_investment = amount / spot_rate * (1 + foreign_rate)
        forward_sale = foreign_investment * forward_rate
        profit1 = forward_sale - amount - borrow_cost
        
        # Strategy 2: Borrow foreign, invest domestic
        foreign_borrow = amount * foreign_rate / spot_rate
        domestic_investment = amount * (1 + domestic_rate)
        forward_purchase = foreign_borrow * forward_rate
        profit2 = domestic_investment - forward_purchase * spot_rate - foreign_borrow * spot_rate
        
        return {
            'strategy_1_profit': profit1,
            'strategy_2_profit': profit2,
            'arbitrage_opportunity': max(profit1, profit2) > 0,
            'optimal_strategy': 1 if profit1 > profit2 else 2
        }

# Example usage
basis_analyzer = BasisAnalyzer()
# basis_info = basis_analyzer.calculate_cross_currency_basis(forward_rate, ir_data, spot_rate, time_period)
# opportunities = basis_analyzer.basis_trading_opportunity(basis_data)
```

## FX Market Microstructure

### Order Flow Analysis
```python
class FXMicrostructure:
    def __init__(self):
        self.order_book_data = {}
        
    def analyze_order_flow(self, bid_volumes, ask_volumes, trade_volumes):
        """Analyze FX order flow dynamics"""
        # Order book imbalance
        total_bid_volume = sum(bid_volumes)
        total_ask_volume = sum(ask_volumes)
        order_imbalance = (total_bid_volume - total_ask_volume) / (total_bid_volume + total_ask_volume)
        
        # Volume-weighted metrics
        volume_profile = {
            'buy_volume': sum(trade_volumes['buys']),
            'sell_volume': sum(trade_volumes['sells']),
            'net_volume': sum(trade_volumes['buys']) - sum(trade_volumes['sells']),
            'volume_imbalance': (sum(trade_volumes['buys']) - sum(trade_volumes['sells'])) / 
                              (sum(trade_volumes['buys']) + sum(trade_volumes['sells']))
        }
        
        # Market depth analysis
        market_depth = {
            'bid_depth': total_bid_volume,
            'ask_depth': total_ask_volume,
            'depth_ratio': total_bid_volume / total_ask_volume if total_ask_volume > 0 else float('inf'),
            'spread': self.calculate_effective_spread(bid_volumes, ask_volumes)
        }
        
        return {
            'order_imbalance': order_imbalance,
            'volume_profile': volume_profile,
            'market_depth': market_depth,
            'liquidity_measure': self.calculate_liquidity(market_depth)
        }
    
    def calculate_effective_spread(self, bids, asks):
        """Calculate effective bid-ask spread"""
        if not bids or not asks:
            return float('inf')
        
        weighted_bid = sum(price * volume for price, volume in bids) / sum(volume for _, volume in bids)
        weighted_ask = sum(price * volume for price, volume in asks) / sum(volume for _, volume in asks)
        
        return weighted_ask - weighted_bid
    
    def calculate_liquidity(self, market_depth):
        """Calculate market liquidity metrics"""
        return {
            'depth_liquidity': market_depth['bid_depth'] + market_depth['ask_depth'],
            'tightness': 1 / market_depth['spread'] if market_depth['spread'] > 0 else float('inf'),
            'resiliency': self.measure_price_resiliency(),  # Would use historical price data
            'immediacy_cost': market_depth['spread'] / 2
        }
    
    def measure_price_resiliency(self):
        """Measure how quickly prices revert after large trades"""
        # This would require tick data analysis
        # Simplified placeholder
        return 0.7  # Example resiliency score
    
    def detect_market_regimes(self, price_data, volume_data):
        """Detect different FX market regimes"""
        # Calculate volatility and volume metrics
        returns = price_data.pct_change().dropna()
        volatility = returns.rolling(20).std()
        volume_ma = volume_data.rolling(20).mean()
        
        regimes = []
        for i in range(len(price_data)):
            if i < 20:
                regimes.append('normal')
                continue
                
            current_vol = volatility.iloc[i]
            current_vol_ma = volatility.iloc[i-20:i].mean()
            current_volume = volume_data.iloc[i]
            current_volume_ma = volume_ma.iloc[i]
            
            # High volatility, high volume = trending regime
            if current_vol > current_vol_ma * 1.5 and current_volume > current_volume_ma * 1.2:
                regimes.append('trending')
            # Low volatility, low volume = range-bound regime
            elif current_vol < current_vol_ma * 0.7 and current_volume < current_volume_ma * 0.8:
                regimes.append('range_bound')
            else:
                regimes.append('normal')
        
        return regimes

# Example usage
micro_analyzer = FXMicrostructure()
# flow_analysis = micro_analyzer.analyze_order_flow(bid_data, ask_data, trade_data)
# regimes = micro_analyzer.detect_market_regimes(price_series, volume_series)
```

## Currency Portfolio Construction

### Multi-Currency Portfolio Optimization
```python
class CurrencyPortfolioOptimizer:
    def __init__(self):
        pass
    
    def mean_variance_optimization(self, expected_returns, covariance_matrix, risk_aversion=1):
        """Optimize currency portfolio using mean-variance framework"""
        n_assets = len(expected_returns)
        
        def objective(weights):
            portfolio_return = np.dot(weights, expected_returns)
            portfolio_variance = np.dot(weights.T, np.dot(covariance_matrix, weights))
            utility = portfolio_return - risk_aversion * portfolio_variance
            return -utility  # Minimize negative utility
        
        # Constraints
        constraints = [{'type': 'eq', 'fun': lambda x: np.sum(x) - 1}]
        bounds = tuple((-1, 1) for _ in range(n_assets))  # Allow short positions
        
        # Initial guess
        initial_weights = np.array([1/n_assets] * n_assets)
        
        # Optimize
        result = minimize(objective, initial_weights, method='SLSQP', 
                         bounds=bounds, constraints=constraints)
        
        return result.x
    
    def risk_parity_allocation(self, covariance_matrix, target_risk=0.1):
        """Allocate currencies using risk parity approach"""
        n_assets = len(covariance_matrix)
        
        def risk_parity_objective(weights):
            # Portfolio risk
            portfolio_risk = np.sqrt(np.dot(weights.T, np.dot(covariance_matrix, weights)))
            
            # Risk contribution of each asset
            marginal_risk = np.dot(covariance_matrix, weights) / portfolio_risk
            risk_contribution = weights * marginal_risk
            
            # Minimize deviation from equal risk contribution
            target_contribution = portfolio_risk / n_assets
            return np.sum((risk_contribution - target_contribution) ** 2)
        
        # Constraints
        constraints = [{'type': 'eq', 'fun': lambda x: np.sum(x) - 1}]
        bounds = tuple((0, 1) for _ in range(n_assets))  # No short positions
        
        # Initial guess
        initial_weights = np.array([1/n_assets] * n_assets)
        
        # Optimize
        result = minimize(risk_parity_objective, initial_weights, method='SLSQP',
                         bounds=bounds, constraints=constraints)
        
        # Scale to target risk level
        optimal_weights = result.x
        current_risk = np.sqrt(np.dot(optimal_weights.T, np.dot(covariance_matrix, optimal_weights)))
        scaled_weights = optimal_weights * (target_risk / current_risk)
        
        return scaled_weights
    
    def currency_factor_model(self, currency_returns, factors):
        """Build currency factor model for risk attribution"""
        # Align data
        aligned_data = pd.DataFrame(currency_returns)
        aligned_data['factors'] = factors
        
        factor_loadings = {}
        factor_returns = {}
        
        # Calculate factor loadings for each currency
        for currency in currency_returns.columns:
            currency_data = aligned_data[[currency, 'factors']].dropna()
            if len(currency_data) > 10:
                # Simple regression
                X = currency_data['factors'].values.reshape(-1, 1)
                y = currency_data[currency].values
                coefficients = np.linalg.lstsq(X, y, rcond=None)[0]
                factor_loadings[currency] = coefficients[0]
        
        return factor_loadings

# Example usage
portfolio_optimizer = CurrencyPortfolioOptimizer()
# mv_weights = portfolio_optimizer.mean_variance_optimization(expected_returns, cov_matrix)
# rp_weights = portfolio_optimizer.risk_parity_allocation(cov_matrix)
```

This comprehensive foreign exchange analysis documentation covers currency pair fundamentals, technical analysis, carry trade strategies, risk management, basis trading, market microstructure, and portfolio optimization techniques essential for quantitative FX trading.