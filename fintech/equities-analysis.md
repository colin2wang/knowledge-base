# Equities Analysis

Comprehensive guide to equity analysis, stock screening, and fundamental analysis techniques for quantitative investment strategies.

## Overview

Equity analysis encompasses both quantitative and qualitative methods to evaluate stock investments, combining financial statement analysis, valuation metrics, and market data to make informed investment decisions.

## Fundamental Analysis Framework

### Financial Statement Analysis
```python
import pandas as pd
import numpy as np
from datetime import datetime

class FinancialStatementAnalyzer:
    def __init__(self):
        self.ratios = {}
        
    def analyze_income_statement(self, income_statement):
        """Analyze key income statement metrics"""
        analysis = {
            'revenue_growth': self.calculate_revenue_growth(income_statement),
            'gross_margin': self.calculate_gross_margin(income_statement),
            'operating_margin': self.calculate_operating_margin(income_statement),
            'net_margin': self.calculate_net_margin(income_statement),
            'earnings_quality': self.assess_earnings_quality(income_statement)
        }
        return analysis
    
    def analyze_balance_sheet(self, balance_sheet):
        """Analyze balance sheet health"""
        analysis = {
            'current_ratio': self.calculate_current_ratio(balance_sheet),
            'debt_to_equity': self.calculate_debt_to_equity(balance_sheet),
            'interest_coverage': self.calculate_interest_coverage(balance_sheet),
            'working_capital': self.calculate_working_capital(balance_sheet),
            'asset_quality': self.assess_asset_quality(balance_sheet)
        }
        return analysis
    
    def analyze_cash_flow(self, cash_flow_statement):
        """Analyze cash flow generation"""
        analysis = {
            'operating_cash_flow': self.calculate_operating_cash_flow(cash_flow_statement),
            'free_cash_flow': self.calculate_free_cash_flow(cash_flow_statement),
            'cash_flow_margin': self.calculate_cash_flow_margin(cash_flow_statement),
            'cash_conversion_cycle': self.calculate_cash_conversion_cycle(cash_flow_statement),
            'cash_flow_stability': self.assess_cash_flow_stability(cash_flow_statement)
        }
        return analysis
    
    def calculate_revenue_growth(self, income_stmt):
        """Calculate revenue growth rates"""
        revenues = income_stmt['revenue']
        growth_rates = revenues.pct_change().dropna()
        return {
            'annual_growth': growth_rates.mean() * 4,  # Quarterly to annual
            'growth_stability': growth_rates.std(),
            'recent_growth': growth_rates.iloc[-4:].mean(),  # Last year
            'acceleration': growth_rates.iloc[-2] - growth_rates.iloc[-6]  # 6 quarters ago
        }
    
    def calculate_gross_margin(self, income_stmt):
        """Calculate gross margin metrics"""
        gross_profit = income_stmt['gross_profit']
        revenue = income_stmt['revenue']
        gross_margins = gross_profit / revenue
        
        return {
            'current_margin': gross_margins.iloc[-1],
            'average_margin': gross_margins.mean(),
            'margin_trend': np.polyfit(range(len(gross_margins)), gross_margins, 1)[0],
            'margin_volatility': gross_margins.std()
        }
    
    def calculate_operating_margin(self, income_stmt):
        """Calculate operating margin"""
        operating_income = income_stmt['operating_income']
        revenue = income_stmt['revenue']
        operating_margins = operating_income / revenue
        
        return {
            'current_operating_margin': operating_margins.iloc[-1],
            'operating_leverage': operating_margins.std() / operating_margins.mean(),
            'margin_expansion': operating_margins.iloc[-1] - operating_margins.iloc[0]
        }

# Example usage
financial_analyzer = FinancialStatementAnalyzer()
# income_analysis = financial_analyzer.analyze_income_statement(income_statement_data)
```

### Valuation Metrics
```python
class EquityValuation:
    def __init__(self):
        self.valuation_methods = ['dcf', 'comparables', 'asset_based', 'precedent_transactions']
        
    def discounted_cash_flow_valuation(self, free_cash_flows, wacc, terminal_growth=0.025):
        """DCF valuation model"""
        # Project cash flows for next 5-10 years
        projection_years = 10
        projected_fcf = self.project_free_cash_flows(free_cash_flows, projection_years)
        
        # Calculate present value of projected cash flows
        discount_factors = [(1 + wacc) ** i for i in range(1, projection_years + 1)]
        pv_fcf = sum(cf / df for cf, df in zip(projected_fcf, discount_factors))
        
        # Terminal value calculation
        terminal_value = (projected_fcf[-1] * (1 + terminal_growth)) / (wacc - terminal_growth)
        pv_terminal = terminal_value / ((1 + wacc) ** projection_years)
        
        # Total enterprise value
        enterprise_value = pv_fcf + pv_terminal
        
        return {
            'enterprise_value': enterprise_value,
            'equity_value': enterprise_value,  # Adjust for net debt if needed
            'fair_value_per_share': enterprise_value,  # Divide by shares outstanding
            'margin_of_safety': 0.25  # 25% margin of safety
        }
    
    def comparables_valuation(self, company_metrics, peer_group_metrics):
        """Relative valuation using comparable companies"""
        valuation_multiples = {
            'pe_ratio': company_metrics['net_income'],
            'price_to_sales': company_metrics['revenue'],
            'price_to_book': company_metrics['book_value'],
            'ev_to_ebitda': company_metrics['ebitda'],
            'price_to_free_cash_flow': company_metrics['free_cash_flow']
        }
        
        # Calculate peer group averages
        peer_averages = {}
        for multiple, denominator in valuation_multiples.items():
            if denominator != 0:
                peer_averages[multiple] = peer_group_metrics[multiple].mean()
        
        # Apply multiples to company metrics
        valuations = {}
        for multiple, average_multiple in peer_averages.items():
            if multiple == 'pe_ratio':
                valuations[multiple] = company_metrics['net_income'] * average_multiple
            elif multiple == 'price_to_sales':
                valuations[multiple] = company_metrics['revenue'] * average_multiple
            # Add other multiples...
        
        # Weighted average valuation
        weights = {'pe_ratio': 0.4, 'price_to_sales': 0.3, 'price_to_book': 0.3}
        weighted_valuation = sum(valuations.get(m, 0) * w for m, w in weights.items())
        
        return {
            'individual_valuations': valuations,
            'weighted_valuation': weighted_valuation,
            'valuation_range': (weighted_valuation * 0.8, weighted_valuation * 1.2)
        }
    
    def project_free_cash_flows(self, historical_fcf, projection_years):
        """Project future free cash flows"""
        # Calculate historical growth rate
        if len(historical_fcf) < 2:
            return [historical_fcf.iloc[-1]] * projection_years
            
        growth_rates = historical_fcf.pct_change().dropna()
        avg_growth = growth_rates.mean()
        
        # Project with declining growth
        projections = []
        current_fcf = historical_fcf.iloc[-1]
        
        for year in range(1, projection_years + 1):
            # Assume growth declines by 10% each year until reaching terminal growth
            year_growth = max(avg_growth * (0.9 ** (year - 1)), 0.025)  # Min 2.5%
            current_fcf = current_fcf * (1 + year_growth)
            projections.append(current_fcf)
            
        return projections

# Example usage
valuator = EquityValuation()
# dcf_value = valuator.discounted_cash_flow_valuation(fcf_data, wacc=0.10)
```

## Stock Screening Systems

### Quantitative Screening Framework
```python
class StockScreener:
    def __init__(self):
        self.screening_criteria = {
            'financial_health': {
                'minimum_market_cap': 1000000000,  # $1B minimum
                'maximum_debt_to_equity': 0.5,
                'minimum_current_ratio': 1.5,
                'minimum_roe': 0.10,
                'minimum_roa': 0.05
            },
            'valuation': {
                'maximum_pe_ratio': 25,
                'maximum_price_to_sales': 5,
                'minimum_dividend_yield': 0.01
            },
            'growth': {
                'minimum_revenue_growth': 0.05,
                'minimum_earnings_growth': 0.08,
                'minimum_book_value_growth': 0.05
            },
            'momentum': {
                'minimum_3_month_return': 0.05,
                'minimum_12_month_return': 0.15,
                'relative_strength_vs_sector': 0.5
            }
        }
    
    def screen_universe(self, stock_data, universe='large_cap'):
        """Screen stock universe based on criteria"""
        # Filter by universe
        if universe == 'large_cap':
            filtered_data = stock_data[stock_data['market_cap'] >= 10000000000]  # $10B+
        elif universe == 'mid_cap':
            filtered_data = stock_data[
                (stock_data['market_cap'] >= 2000000000) & 
                (stock_data['market_cap'] < 10000000000)
            ]
        else:
            filtered_data = stock_data
        
        # Apply financial health screens
        healthy_stocks = self.apply_financial_health_filters(filtered_data)
        
        # Apply valuation screens
        valued_stocks = self.apply_valuation_filters(healthy_stocks)
        
        # Apply growth screens
        growth_stocks = self.apply_growth_filters(valued_stocks)
        
        # Apply momentum screens
        momentum_stocks = self.apply_momentum_filters(growth_stocks)
        
        # Rank by composite score
        ranked_stocks = self.rank_by_composite_score(momentum_stocks)
        
        return ranked_stocks
    
    def apply_financial_health_filters(self, data):
        """Apply financial health screening criteria"""
        criteria = self.screening_criteria['financial_health']
        
        filtered = data[
            (data['market_cap'] >= criteria['minimum_market_cap']) &
            (data['debt_to_equity'] <= criteria['maximum_debt_to_equity']) &
            (data['current_ratio'] >= criteria['minimum_current_ratio']) &
            (data['roe'] >= criteria['minimum_roe']) &
            (data['roa'] >= criteria['minimum_roa'])
        ]
        return filtered
    
    def apply_valuation_filters(self, data):
        """Apply valuation screening criteria"""
        criteria = self.screening_criteria['valuation']
        
        filtered = data[
            (data['pe_ratio'] <= criteria['maximum_pe_ratio']) &
            (data['price_to_sales'] <= criteria['maximum_price_to_sales']) &
            (data['dividend_yield'] >= criteria['minimum_dividend_yield'])
        ]
        return filtered
    
    def rank_by_composite_score(self, data):
        """Rank stocks by composite score"""
        # Normalize factors to 0-1 scale
        normalized_data = data.copy()
        
        # Financial health score (higher is better)
        normalized_data['health_score'] = (
            (data['current_ratio'] / 3) +  # Max 3
            (0.2 / (1 + data['debt_to_equity'])) +  # Lower debt better
            data['roe'] + 
            data['roa']
        ) / 4
        
        # Valuation score (lower ratios better for some metrics)
        normalized_data['value_score'] = (
            (1 / (1 + data['pe_ratio'] / 25)) +  # Lower PE better
            (1 / (1 + data['price_to_sales'] / 5)) +  # Lower P/S better
            data['dividend_yield']  # Higher yield better
        ) / 3
        
        # Growth score
        normalized_data['growth_score'] = (
            data['revenue_growth'] +
            data['earnings_growth'] +
            data['book_value_growth']
        ) / 3
        
        # Momentum score
        normalized_data['momentum_score'] = (
            data['three_month_return'] +
            data['twelve_month_return'] / 3 +
            data['relative_strength']
        ) / 3
        
        # Composite score
        normalized_data['composite_score'] = (
            normalized_data['health_score'] * 0.3 +
            normalized_data['value_score'] * 0.25 +
            normalized_data['growth_score'] * 0.25 +
            normalized_data['momentum_score'] * 0.2
        )
        
        # Sort by composite score
        ranked = normalized_data.sort_values('composite_score', ascending=False)
        return ranked

# Example usage
screener = StockScreener()
# screened_stocks = screener.screen_universe(financial_data, universe='large_cap')
```

## Sector and Industry Analysis

### Sector Rotation Strategy
```python
class SectorAnalyzer:
    def __init__(self):
        self.sectors = [
            'Consumer Discretionary', 'Consumer Staples', 'Energy',
            'Financials', 'Healthcare', 'Industrials',
            'Information Technology', 'Materials', 'Real Estate',
            'Utilities', 'Communication Services'
        ]
        
    def analyze_sector_rotation(self, sector_returns, economic_indicators):
        """Analyze optimal sector rotation timing"""
        rotation_signals = {}
        
        # Economic regime identification
        economic_regime = self.identify_economic_regime(economic_indicators)
        
        # Sector momentum analysis
        sector_momentum = self.calculate_sector_momentum(sector_returns)
        
        # Sector valuation analysis
        sector_valuations = self.analyze_sector_valuations()
        
        # Generate rotation signals
        for sector in self.sectors:
            signal_strength = self.calculate_rotation_signal(
                sector, economic_regime, sector_momentum, sector_valuations
            )
            rotation_signals[sector] = signal_strength
            
        return rotation_signals
    
    def identify_economic_regime(self, indicators):
        """Identify current economic regime"""
        # Simplified economic regime model
        gdp_growth = indicators.get('gdp_growth', 0)
        inflation = indicators.get('inflation', 0)
        unemployment = indicators.get('unemployment', 0)
        
        if gdp_growth > 0.03 and inflation < 0.03:
            return 'expansion'
        elif gdp_growth < 0:
            return 'recession'
        elif inflation > 0.05:
            return 'inflationary'
        else:
            return 'stagflation'
    
    def calculate_sector_momentum(self, returns_data, lookback_periods=[1, 3, 6, 12]):
        """Calculate sector momentum across multiple timeframes"""
        momentum_scores = {}
        
        for sector in self.sectors:
            if sector in returns_data.columns:
                sector_returns = returns_data[sector]
                momentum_score = 0
                
                for period in lookback_periods:
                    if len(sector_returns) >= period:
                        period_return = (sector_returns.iloc[-period:] + 1).prod() - 1
                        weight = 1 / period  # Longer periods get lower weight
                        momentum_score += period_return * weight
                
                momentum_scores[sector] = momentum_score
        
        return momentum_scores
    
    def calculate_rotation_signal(self, sector, regime, momentum, valuations):
        """Calculate sector rotation signal strength"""
        # Regime preferences
        regime_preferences = {
            'expansion': {
                'Technology': 1.2, 'Consumer Discretionary': 1.1, 'Industrials': 1.0
            },
            'recession': {
                'Consumer Staples': 1.2, 'Utilities': 1.1, 'Healthcare': 1.0
            },
            'inflationary': {
                'Energy': 1.2, 'Materials': 1.1, 'Real Estate': 1.0
            },
            'stagflation': {
                'Utilities': 1.1, 'Consumer Staples': 1.0, 'Healthcare': 0.9
            }
        }
        
        # Base signal from momentum
        base_signal = momentum.get(sector, 0)
        
        # Adjust for economic regime
        regime_adjustment = regime_preferences.get(regime, {}).get(sector, 1.0)
        
        # Adjust for valuations (cheaper sectors get boost)
        valuation_adjustment = 1.0  # Simplified
        
        final_signal = base_signal * regime_adjustment * valuation_adjustment
        return final_signal

# Example usage
sector_analyzer = SectorAnalyzer()
# rotation_signals = sector_analyzer.analyze_sector_rotation(sector_returns, economic_data)
```

## ESG Integration

### Environmental, Social, and Governance Analysis
```python
class ESGAnalyzer:
    def __init__(self):
        self.esg_metrics = {
            'environmental': ['carbon_emissions', 'energy_efficiency', 'waste_management'],
            'social': ['employee_satisfaction', 'community_relations', 'product_safety'],
            'governance': ['board_independence', 'executive_compensation', 'audit_quality']
        }
    
    def calculate_esg_score(self, company_data):
        """Calculate comprehensive ESG score"""
        esg_components = {}
        
        # Environmental score
        esg_components['environmental'] = self.calculate_environmental_score(company_data)
        
        # Social score
        esg_components['social'] = self.calculate_social_score(company_data)
        
        # Governance score
        esg_components['governance'] = self.calculate_governance_score(company_data)
        
        # Overall ESG score
        overall_score = (
            esg_components['environmental'] * 0.3 +
            esg_components['social'] * 0.3 +
            esg_components['governance'] * 0.4
        )
        
        return {
            'overall_esg_score': overall_score,
            'component_scores': esg_components,
            'esg_quartile': self.determine_esg_quartile(overall_score)
        }
    
    def calculate_environmental_score(self, data):
        """Calculate environmental component score"""
        carbon_score = 1 - min(data.get('carbon_intensity', 1), 1)  # Lower is better
        energy_score = data.get('renewable_energy_percentage', 0)
        waste_score = 1 - min(data.get('waste_per_employee', 1), 1)
        
        return (carbon_score + energy_score + waste_score) / 3
    
    def calculate_social_score(self, data):
        """Calculate social component score"""
        employee_score = data.get('employee_satisfaction_score', 0) / 100
        diversity_score = data.get('board_diversity_percentage', 0)
        safety_score = 1 - min(data.get('workplace_accidents', 1), 1)
        
        return (employee_score + diversity_score + safety_score) / 3
    
    def calculate_governance_score(self, data):
        """Calculate governance component score"""
        board_score = data.get('independent_directors_percentage', 0)
        compensation_score = 1 - min(data.get('ceo_pay_ratio', 1) / 300, 1)  # Normalize
        audit_score = data.get('audit_committee_quality', 0)
        
        return (board_score + compensation_score + audit_score) / 3
    
    def determine_esg_quartile(self, score):
        """Determine ESG quartile ranking"""
        if score >= 0.75:
            return 'First Quartile (Top 25%)'
        elif score >= 0.50:
            return 'Second Quartile (25-50%)'
        elif score >= 0.25:
            return 'Third Quartile (50-75%)'
        else:
            return 'Fourth Quartile (Bottom 25%)'

# Example usage
esg_analyzer = ESGAnalyzer()
# esg_results = esg_analyzer.calculate_esg_score(company_esg_data)
```

## Performance Monitoring

### Equity Portfolio Analytics
```python
class EquityPortfolioAnalytics:
    def __init__(self, benchmark='SPY'):
        self.benchmark = benchmark
        
    def analyze_portfolio_performance(self, portfolio_returns, benchmark_returns, risk_free_rate=0.02):
        """Comprehensive portfolio performance analysis"""
        analysis = {
            'absolute_performance': self.calculate_absolute_metrics(portfolio_returns, risk_free_rate),
            'relative_performance': self.calculate_relative_metrics(portfolio_returns, benchmark_returns),
            'risk_metrics': self.calculate_risk_metrics(portfolio_returns),
            'attribution_analysis': self.perform_attribution_analysis(portfolio_returns),
            'drawdown_analysis': self.analyze_drawdowns(portfolio_returns)
        }
        return analysis
    
    def calculate_absolute_metrics(self, returns, risk_free_rate):
        """Calculate absolute performance metrics"""
        annualized_return = (1 + returns.mean()) ** 252 - 1
        volatility = returns.std() * np.sqrt(252)
        sharpe_ratio = (annualized_return - risk_free_rate) / volatility
        
        cumulative_return = (1 + returns).prod() - 1
        
        return {
            'annualized_return': annualized_return,
            'volatility': volatility,
            'sharpe_ratio': sharpe_ratio,
            'cumulative_return': cumulative_return,
            'best_month': returns.max(),
            'worst_month': returns.min()
        }
    
    def calculate_relative_metrics(self, portfolio_returns, benchmark_returns):
        """Calculate relative performance metrics"""
        excess_returns = portfolio_returns - benchmark_returns
        tracking_error = excess_returns.std() * np.sqrt(252)
        information_ratio = excess_returns.mean() / excess_returns.std() * np.sqrt(252)
        
        # Calculate beta
        covariance = np.cov(portfolio_returns, benchmark_returns)[0, 1]
        benchmark_variance = np.var(benchmark_returns)
        beta = covariance / benchmark_variance
        
        # Calculate alpha
        portfolio_return = portfolio_returns.mean() * 252
        benchmark_return = benchmark_returns.mean() * 252
        alpha = portfolio_return - (risk_free_rate + beta * (benchmark_return - risk_free_rate))
        
        return {
            'tracking_error': tracking_error,
            'information_ratio': information_ratio,
            'beta': beta,
            'alpha': alpha,
            'up_capture': self.calculate_up_capture(portfolio_returns, benchmark_returns),
            'down_capture': self.calculate_down_capture(portfolio_returns, benchmark_returns)
        }
    
    def calculate_risk_metrics(self, returns):
        """Calculate comprehensive risk metrics"""
        # Value at Risk
        var_95 = np.percentile(returns, 5)
        var_99 = np.percentile(returns, 1)
        
        # Maximum drawdown
        cumulative = (1 + returns).cumprod()
        running_max = cumulative.expanding().max()
        drawdown = (cumulative - running_max) / running_max
        max_drawdown = drawdown.min()
        
        # Downside deviation
        negative_returns = returns[returns < 0]
        downside_deviation = negative_returns.std() * np.sqrt(252)
        
        # Sortino ratio
        annualized_return = (1 + returns.mean()) ** 252 - 1
        sortino_ratio = (annualized_return - 0.02) / downside_deviation
        
        return {
            'value_at_risk_95': var_95,
            'value_at_risk_99': var_99,
            'maximum_drawdown': max_drawdown,
            'downside_deviation': downside_deviation,
            'sortino_ratio': sortino_ratio,
            'skewness': self.calculate_skewness(returns),
            'kurtosis': self.calculate_kurtosis(returns)
        }

# Example usage
portfolio_analytics = EquityPortfolioAnalytics()
# performance_analysis = portfolio_analytics.analyze_portfolio_performance(port_returns, bench_returns)
```

This comprehensive equities analysis documentation covers fundamental analysis, valuation methods, stock screening systems, sector analysis, ESG integration, and portfolio performance monitoring essential for quantitative equity investing.