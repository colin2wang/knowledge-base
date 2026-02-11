# Risk Management

Risk management is the most critical aspect of quantitative trading, determining strategy survival and long-term profitability.

## Core Risk Concepts

### Value at Risk (VaR)
```
# Historical Simulation VaR Calculation
def calculate_historical_var(returns, confidence_level=0.05):
    """
    Calculate VaR using historical simulation method
    returns: historical return series
    confidence_level: confidence level (default 5%)
    """
    var_threshold = np.percentile(returns, confidence_level * 100)
    return abs(var_threshold)

# Parametric VaR Calculation
def calculate_parametric_var(portfolio_value, weights, cov_matrix, confidence_level=0.05):
    """
    Calculate VaR using parametric method
    portfolio_value: total portfolio value
    weights: asset weight vector
    cov_matrix: covariance matrix
    """
    from scipy.stats import norm
    
    # Calculate portfolio volatility
    portfolio_volatility = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights)))
    
    # Calculate VaR
    z_score = norm.ppf(1 - confidence_level)
    var = portfolio_value * portfolio_volatility * z_score
    
    return var

# Monte Carlo VaR Calculation
def calculate_monte_carlo_var(returns, portfolio_value, weights, 
                            simulations=10000, horizon=1, confidence_level=0.05):
    """
    Calculate VaR using Monte Carlo simulation
    """
    # Calculate return statistics
    mean_return = np.mean(returns, axis=0)
    cov_matrix = np.cov(returns.T)
    
    # Generate random return paths
    simulated_returns = np.random.multivariate_normal(
        mean_return * horizon, 
        cov_matrix * horizon, 
        simulations
    )
    
    # Calculate portfolio value for each path
    portfolio_values = []
    for ret in simulated_returns:
        portfolio_return = np.dot(weights, ret)
        final_value = portfolio_value * (1 + portfolio_return)
        portfolio_values.append(final_value)
    
    # Calculate VaR
    var_threshold = np.percentile(portfolio_values, confidence_level * 100)
    var = portfolio_value - var_threshold
    
    return var
```

### Stress Testing
```
# Historical Scenario Stress Testing
def stress_test_historical_scenarios(portfolio_value, weights, historical_data, scenarios):
    """
    Stress testing based on historical extreme events
    scenarios: return data from historical crisis periods
    """
    stress_results = {}
    
    for scenario_name, scenario_returns in scenarios.items():
        # Calculate portfolio loss under this scenario
        portfolio_return = np.dot(weights, scenario_returns)
        stressed_value = portfolio_value * (1 + portfolio_return)
        loss = portfolio_value - stressed_value
        
        stress_results[scenario_name] = {
            'portfolio_return': portfolio_return,
            'loss_amount': loss,
            'loss_percentage': loss / portfolio_value
        }
    
    return stress_results

# Monte Carlo Stress Testing
def monte_carlo_stress_test(portfolio_value, weights, returns, 
                           shock_intensity=2.0, simulations=5000):
    """
    Monte Carlo stress testing
    shock_intensity: shock intensity multiplier
    """
    # Calculate original covariance matrix
    original_cov = np.cov(returns.T)
    
    # Apply stress shock
    stressed_cov = original_cov * (shock_intensity ** 2)
    mean_returns = np.mean(returns, axis=0) * shock_intensity
    
    # Generate stressed return scenarios
    stressed_returns = np.random.multivariate_normal(
        mean_returns, stressed_cov, simulations
    )
    
    # Calculate portfolio performance under stress scenarios
    portfolio_losses = []
    for ret in stressed_returns:
        portfolio_return = np.dot(weights, ret)
        final_value = portfolio_value * (1 + portfolio_return)
        loss = portfolio_value - final_value
        portfolio_losses.append(loss)
    
    # Statistical results
    results = {
        'expected_loss': np.mean(portfolio_losses),
        'worst_case_loss': np.max(portfolio_losses),
        'var_95': np.percentile(portfolio_losses, 95),
        'var_99': np.percentile(portfolio_losses, 99),
        'conditional_var_95': np.mean([loss for loss in portfolio_losses 
                                     if loss >= np.percentile(portfolio_losses, 95)])
    }
    
    return results
```

### Position Sizing
```
# Kelly Criterion Position Management
def kelly_criterion(win_probability, avg_win, avg_loss):
    """
    Kelly criterion for optimal position sizing
    win_probability: win rate
    avg_win: average profit
    avg_loss: average loss
    """
    if avg_loss == 0:
        return 1.0  # Avoid division by zero
    
    kelly_fraction = win_probability - (1 - win_probability) * (avg_loss / avg_win)
    return max(0, min(kelly_fraction, 1))  # Limit between 0-1

# Fixed Fraction Position Sizing
def fixed_fraction_sizing(account_value, risk_per_trade=0.01):
    """
    Fixed risk percentage position sizing
    risk_per_trade: risk percentage per trade
    """
    max_risk_amount = account_value * risk_per_trade
    return max_risk_amount

# Volatility-Adjusted Position Sizing
def volatility_adjusted_sizing(account_value, asset_volatility, 
                             target_volatility=0.15, risk_per_trade=0.01):
    """
    Dynamic position adjustment based on volatility
    asset_volatility: asset volatility
    target_volatility: target volatility
    """
    # Calculate volatility adjustment factor
    vol_adjustment = target_volatility / asset_volatility
    
    # Base position
    base_position = account_value * risk_per_trade
    
    # Adjusted position
    adjusted_position = base_position * vol_adjustment
    
    return min(adjusted_position, account_value * 0.05)  # Maximum 5%

# Correlation-Adjusted Position Sizing
def correlation_adjusted_sizing(weights, correlation_matrix, total_risk_budget):
    """
    Position allocation considering asset correlations
    """
    # Calculate portfolio volatility
    portfolio_variance = np.dot(weights.T, np.dot(correlation_matrix, weights))
    portfolio_volatility = np.sqrt(portfolio_variance)
    
    # Adjust positions based on correlation
    adjusted_weights = weights / (1 + np.diag(correlation_matrix))
    adjusted_weights = adjusted_weights / np.sum(adjusted_weights)
    
    # Allocate risk budget
    positions = adjusted_weights * total_risk_budget / portfolio_volatility
    
    return positions
```

## Risk Monitoring System

### Real-time Risk Metrics
```
class RiskMonitor:
    def __init__(self, portfolio_value, max_drawdown_limit=0.10, 
                 var_limit=0.05, concentration_limit=0.30):
        self.portfolio_value = portfolio_value
        self.max_drawdown_limit = max_drawdown_limit
        self.var_limit = var_limit
        self.concentration_limit = concentration_limit
        self.peak_value = portfolio_value
        self.drawdown_history = []
        
    def update_portfolio_value(self, current_value):
        """Update portfolio value and calculate drawdown"""
        self.portfolio_value = current_value
        self.peak_value = max(self.peak_value, current_value)
        
        drawdown = (self.peak_value - current_value) / self.peak_value
        self.drawdown_history.append(drawdown)
        
        return drawdown
    
    def check_drawdown_risk(self):
        """Check maximum drawdown risk"""
        current_drawdown = self.update_portfolio_value(self.portfolio_value)
        return current_drawdown > self.max_drawdown_limit
    
    def check_concentration_risk(self, position_weights):
        """Check concentration risk"""
        max_weight = np.max(position_weights)
        return max_weight > self.concentration_limit
    
    def check_var_risk(self, returns, confidence_level=0.05):
        """Check VaR risk"""
        var = calculate_historical_var(returns, confidence_level)
        var_ratio = var / self.portfolio_value
        return var_ratio > self.var_limit
    
    def generate_risk_report(self, returns, position_weights):
        """Generate risk report"""
        current_drawdown = self.update_portfolio_value(self.portfolio_value)
        
        report = {
            'timestamp': datetime.now(),
            'portfolio_value': self.portfolio_value,
            'current_drawdown': current_drawdown,
            'drawdown_exceeded': self.check_drawdown_risk(),
            'concentration_exceeded': self.check_concentration_risk(position_weights),
            'var_exceeded': self.check_var_risk(returns),
            'max_position_weight': np.max(position_weights),
            'portfolio_var': calculate_historical_var(returns) / self.portfolio_value
        }
        
        return report
```

### Risk Alert Mechanism
```
class RiskAlertSystem:
    def __init__(self):
        self.alerts = []
        self.thresholds = {
            'high_volatility': 0.03,      # High volatility threshold
            'large_loss': 0.02,           # Large single-day loss
            'position_limit': 0.25,       # Single position upper limit
            'correlation_spike': 0.8      # Correlation abnormal increase
        }
    
    def check_volatility_alert(self, daily_returns):
        """Volatility anomaly alert"""
        recent_vol = np.std(daily_returns[-5:])  # Recent 5-day volatility
        if recent_vol > self.thresholds['high_volatility']:
            alert = {
                'type': 'HIGH_VOLATILITY',
                'level': 'WARNING',
                'message': f'Recent volatility {recent_vol:.4f} exceeds threshold',
                'timestamp': datetime.now()
            }
            self.alerts.append(alert)
            return alert
    
    def check_large_loss_alert(self, daily_return):
        """Large loss alert"""
        if daily_return < -self.thresholds['large_loss']:
            alert = {
                'type': 'LARGE_LOSS',
                'level': 'CRITICAL',
                'message': f'Single-day loss {daily_return:.4f} exceeds limit',
                'timestamp': datetime.now()
            }
            self.alerts.append(alert)
            return alert
    
    def check_position_alert(self, position_weight):
        """Position concentration alert"""
        if position_weight > self.thresholds['position_limit']:
            alert = {
                'type': 'HIGH_CONCENTRATION',
                'level': 'WARNING',
                'message': f'Position weight {position_weight:.4f} exceeds limit',
                'timestamp': datetime.now()
            }
            self.alerts.append(alert)
            return alert
    
    def get_active_alerts(self, severity='WARNING'):
        """Get active alerts by severity level"""
        if severity == 'ALL':
            return self.alerts
        return [alert for alert in self.alerts if alert['level'] == severity]
```

## Risk Management Best Practices

### Diversification Strategies
```
# Portfolio Diversification Analysis
def analyze_diversification(returns_data, sector_classification=None):
    """
    Analyze portfolio diversification
    returns_data: asset return data
    sector_classification: sector classification dictionary
    """
    # Calculate correlation matrix
    correlation_matrix = np.corrcoef(returns_data.T)
    
    # Effective number of bets
    eigenvalues = np.linalg.eigvals(correlation_matrix)
    effective_bets = len(eigenvalues) / np.sum(eigenvalues ** 2)
    
    # Sector concentration (if sector data provided)
    sector_concentration = {}
    if sector_classification:
        for sector, assets in sector_classification.items():
            sector_weights = [weights[asset] for asset in assets if asset in weights]
            sector_concentration[sector] = sum(sector_weights)
    
    return {
        'correlation_matrix': correlation_matrix,
        'effective_bets': effective_bets,
        'sector_concentration': sector_concentration,
        'maximum_correlation': np.max(correlation_matrix - np.eye(len(correlation_matrix))),
        'average_correlation': np.mean(correlation_matrix[np.triu_indices_from(correlation_matrix, k=1)])
    }

# Risk Parity Portfolio Construction
def risk_parity_allocation(cov_matrix, target_risk=0.1):
    """
    Construct risk parity portfolio
    cov_matrix: covariance matrix
    target_risk: target portfolio risk
    """
    n_assets = len(cov_matrix)
    
    def objective(weights):
        # Portfolio risk
        portfolio_risk = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights)))
        
        # Risk contribution of each asset
        marginal_risk = np.dot(cov_matrix, weights) / portfolio_risk
        risk_contribution = weights * marginal_risk
        
        # Minimize deviation from equal risk contribution
        target_contribution = portfolio_risk / n_assets
        return np.sum((risk_contribution - target_contribution) ** 2)
    
    # Constraints
    constraints = [{'type': 'eq', 'fun': lambda x: np.sum(x) - 1}]
    bounds = tuple((0, 1) for _ in range(n_assets))
    
    # Initial guess
    initial_weights = np.array(n_assets * [1. / n_assets])
    
    # Optimization
    result = minimize(objective, initial_weights, method='SLSQP',
                     bounds=bounds, constraints=constraints)
    
    # Scale to target risk level
    optimal_weights = result.x
    current_risk = np.sqrt(np.dot(optimal_weights.T, np.dot(cov_matrix, optimal_weights)))
    scaled_weights = optimal_weights * (target_risk / current_risk)
    
    return scaled_weights
```

### Stop-Loss and Take-Profit
```
# Dynamic Stop-Loss Implementation
class DynamicStopLoss:
    def __init__(self, initial_stop_distance=0.05, trail_factor=0.5):
        self.initial_distance = initial_stop_distance
        self.trail_factor = trail_factor
        self.stop_levels = {}
        self.peak_prices = {}
    
    def update_stop_loss(self, symbol, current_price, entry_price):
        """Update dynamic stop-loss level"""
        # Initialize for new position
        if symbol not in self.stop_levels:
            self.stop_levels[symbol] = entry_price * (1 - self.initial_distance)
            self.peak_prices[symbol] = entry_price
        
        # Update peak price
        if current_price > self.peak_prices[symbol]:
            self.peak_prices[symbol] = current_price
            # Trail stop-loss
            new_stop = current_price * (1 - self.initial_distance * self.trail_factor)
            self.stop_levels[symbol] = max(self.stop_levels[symbol], new_stop)
        
        return self.stop_levels[symbol]
    
    def check_stop_condition(self, symbol, current_price):
        """Check if stop-loss condition is met"""
        if symbol in self.stop_levels:
            return current_price <= self.stop_levels[symbol]
        return False

# Take-Profit Management
class TakeProfitManager:
    def __init__(self, profit_target=0.10, partial_take_profit=0.05):
        self.profit_target = profit_target
        self.partial_take_profit = partial_take_profit
        self.targets = {}
        self.partial_positions = {}
    
    def set_targets(self, symbol, entry_price):
        """Set profit targets"""
        self.targets[symbol] = {
            'partial': entry_price * (1 + self.partial_take_profit),
            'full': entry_price * (1 + self.profit_target)
        }
        self.partial_positions[symbol] = 0.5  # Take 50% at partial target
    
    def check_take_profit(self, symbol, current_price):
        """Check take-profit conditions"""
        if symbol not in self.targets:
            return None
            
        targets = self.targets[symbol]
        
        if current_price >= targets['full']:
            return 'FULL_TAKE_PROFIT'
        elif current_price >= targets['partial']:
            return 'PARTIAL_TAKE_PROFIT'
        
        return None
```

## Regulatory Compliance

### MiFID II Requirements
```
# Transaction Reporting System
class TransactionReporter:
    def __init__(self):
        self.transactions = []
        self.reporting_requirements = {
            'execution_time': True,
            'price': True,
            'volume': True,
            'instrument_identifier': True,
            'client_identification': True
        }
    
    def record_transaction(self, transaction_data):
        """Record transaction for regulatory reporting"""
        required_fields = [
            'timestamp', 'instrument_id', 'side', 'quantity', 
            'price', 'client_id', 'execution_venue'
        ]
        
        # Validate required fields
        for field in required_fields:
            if field not in transaction_data:
                raise ValueError(f"Missing required field: {field}")
        
        # Add to transaction log
        transaction_record = {
            'reporting_timestamp': datetime.now(),
            'transaction_id': str(uuid.uuid4()),
            **transaction_data
        }
        
        self.transactions.append(transaction_record)
        return transaction_record['transaction_id']
    
    def generate_mifid_report(self, start_date, end_date):
        """Generate MiFID II compliant report"""
        filtered_transactions = [
            tx for tx in self.transactions
            if start_date <= tx['timestamp'].date() <= end_date
        ]
        
        report = {
            'reporting_period': {'start': start_date, 'end': end_date},
            'total_transactions': len(filtered_transactions),
            'transaction_details': filtered_transactions,
            'summary_statistics': self._calculate_summary_stats(filtered_transactions)
        }
        
        return report
    
    def _calculate_summary_stats(self, transactions):
        """Calculate summary statistics for reporting"""
        if not transactions:
            return {}
            
        prices = [tx['price'] for tx in transactions]
        quantities = [tx['quantity'] for tx in transactions]
        
        return {
            'total_volume': sum(quantities),
            'average_price': np.mean(prices),
            'price_volatility': np.std(prices),
            'unique_instruments': len(set(tx['instrument_id'] for tx in transactions)),
            'buy_sell_ratio': sum(1 for tx in transactions if tx['side'] == 'BUY') / len(transactions)
        }
```

## Risk Dashboard Implementation

### Real-time Monitoring Dashboard
```
class RiskDashboard:
    def __init__(self, portfolio_manager):
        self.portfolio_manager = portfolio_manager
        self.metrics_history = defaultdict(list)
        self.alert_thresholds = {
            'drawdown': 0.08,
            'var': 0.03,
            'concentration': 0.25,
            'volatility': 0.04
        }
    
    def update_metrics(self):
        """Update all risk metrics"""
        portfolio = self.portfolio_manager.get_portfolio()
        returns = self.portfolio_manager.get_recent_returns(252)  # Annual data
        
        metrics = {
            'timestamp': datetime.now(),
            'portfolio_value': portfolio.total_value,
            'drawdown': self._calculate_drawdown(portfolio),
            'var_95': self._calculate_var(returns, 0.05),
            'max_concentration': self._calculate_max_concentration(portfolio),
            'portfolio_volatility': np.std(returns) * np.sqrt(252),  # Annualized
            'sharpe_ratio': self._calculate_sharpe_ratio(returns),
            'positions': len(portfolio.positions)
        }
        
        # Store historical data
        for key, value in metrics.items():
            if key != 'timestamp':
                self.metrics_history[key].append(value)
        
        return metrics
    
    def _calculate_drawdown(self, portfolio):
        """Calculate current drawdown"""
        peak = max(self.metrics_history['portfolio_value'] or [portfolio.total_value])
        return (peak - portfolio.total_value) / peak
    
    def _calculate_var(self, returns, confidence_level):
        """Calculate Value at Risk"""
        if len(returns) < 30:
            return 0
        return np.percentile(returns, confidence_level * 100)
    
    def _calculate_max_concentration(self, portfolio):
        """Calculate maximum position concentration"""
        if not portfolio.positions:
            return 0
        position_values = [pos.value for pos in portfolio.positions.values()]
        return max(position_values) / portfolio.total_value
    
    def _calculate_sharpe_ratio(self, returns):
        """Calculate Sharpe ratio"""
        if len(returns) < 30 or np.std(returns) == 0:
            return 0
        risk_free_rate = 0.02  # 2% annual
        excess_returns = np.array(returns) - risk_free_rate/252
        return np.mean(excess_returns) / np.std(excess_returns) * np.sqrt(252)
    
    def check_alerts(self, current_metrics):
        """Check all risk alerts"""
        alerts = []
        
        # Drawdown alert
        if current_metrics['drawdown'] > self.alert_thresholds['drawdown']:
            alerts.append({
                'type': 'DRAWDOWN_ALERT',
                'severity': 'HIGH',
                'message': f'Drawdown {current_metrics["drawdown"]:.2%} exceeds threshold'
            })
        
        # VaR alert
        if abs(current_metrics['var_95']) > self.alert_thresholds['var']:
            alerts.append({
                'type': 'VAR_ALERT',
                'severity': 'MEDIUM',
                'message': f'VaR {current_metrics["var_95"]:.2%} exceeds limit'
            })
        
        # Concentration alert
        if current_metrics['max_concentration'] > self.alert_thresholds['concentration']:
            alerts.append({
                'type': 'CONCENTRATION_ALERT',
                'severity': 'MEDIUM',
                'message': f'Max concentration {current_metrics["max_concentration"]:.2%} too high'
            })
        
        return alerts
```

## Key Risk Indicators (KRIs)

### Portfolio-Level KRIs
- **Maximum Drawdown**: Peak-to-trough decline over a specific period
- **Value at Risk**: Potential loss at given confidence level
- **Expected Shortfall**: Average loss beyond VaR threshold
- **Tracking Error**: Deviation from benchmark performance
- **Beta Coefficient**: Systematic risk relative to market

### Position-Level KRIs
- **Individual VaR**: Risk contribution of each position
- **Marginal VaR**: Impact of position changes on portfolio risk
- **Component VaR**: Risk attribution to individual positions
- **Concentration Ratio**: Largest position relative to total portfolio

### Market KRIs
- **Market Volatility**: Overall market uncertainty measure
- **Correlation Changes**: Shifts in asset correlation patterns
- **Liquidity Conditions**: Market depth and trading volume
- **Regulatory Changes**: Impact of new regulations on risk profile

## Risk Management Framework

### Three Lines of Defense
1. **First Line**: Business units and trading desks
2. **Second Line**: Risk management and compliance functions
3. **Third Line**: Internal audit and independent oversight

### Risk Appetite Framework
- Define acceptable risk levels for different strategies
- Establish risk limits and thresholds
- Implement escalation procedures
- Regular risk appetite reviews

### Governance Structure
- Risk Committee oversight
- Clear roles and responsibilities
- Independent risk reporting
- Regular risk culture assessment

## Learning Resources

### Academic Literature
- "Risk Management and Financial Institutions" by John Hull
- "The Handbook of Risk Management" by René Stulz
- "Value at Risk: The New Benchmark" by Philippe Jorion

### Industry Standards
- Basel III regulatory framework
- MiFID II compliance requirements
- ISDA risk management practices
- GARP risk management certifications

### Professional Development
- FRM (Financial Risk Manager) certification
- PRM (Professional Risk Manager) designation
- CFA Institute risk management curriculum
- Industry conferences and workshops