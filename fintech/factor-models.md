# Factor Models

Comprehensive guide to multi-factor models used in quantitative finance for risk management, portfolio construction, and alpha generation.

## Overview

Factor models explain asset returns through systematic risk factors, enabling better risk attribution, portfolio optimization, and performance evaluation in quantitative investing.

## Classic Factor Models

### Capital Asset Pricing Model (CAPM)
The foundational single-factor model linking expected returns to market risk.

```python
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt

class CAPMModel:
    def __init__(self, risk_free_rate=0.02):
        self.risk_free_rate = risk_free_rate
        self.alpha = None
        self.beta = None
        self.r_squared = None
    
    def fit(self, asset_returns, market_returns):
        """Fit CAPM model using linear regression"""
        # Calculate excess returns
        asset_excess = asset_returns - self.risk_free_rate/252
        market_excess = market_returns - self.risk_free_rate/252
        
        # Linear regression: Asset Excess Return = α + β × Market Excess Return + ε
        slope, intercept, r_value, p_value, std_err = stats.linregress(
            market_excess.dropna(), 
            asset_excess.dropna()
        )
        
        self.alpha = intercept * 252  # Annualized alpha
        self.beta = slope
        self.r_squared = r_value ** 2
        self.std_err = std_err
        
        return {
            'alpha': self.alpha,
            'beta': self.beta,
            'r_squared': self.r_squared,
            't_statistic': slope / std_err,
            'p_value': p_value
        }
    
    def predict_expected_return(self, market_return):
        """Predict expected return using CAPM"""
        market_excess = market_return - self.risk_free_rate
        expected_excess_return = self.alpha + self.beta * market_excess
        expected_return = self.risk_free_rate + expected_excess_return
        return expected_return
    
    def calculate_jensen_alpha(self, asset_returns, market_returns):
        """Calculate Jensen's alpha"""
        fitted_results = self.fit(asset_returns, market_returns)
        return fitted_results['alpha']
    
    def calculate_treynor_ratio(self, portfolio_return, market_returns):
        """Calculate Treynor ratio (risk-adjusted return per unit of systematic risk)"""
        if self.beta is None:
            raise ValueError("Model must be fitted first")
        
        excess_return = portfolio_return - self.risk_free_rate
        return excess_return / self.beta

# Example usage
capm = CAPMModel(risk_free_rate=0.02)
# capm_results = capm.fit(stock_returns, market_returns)
```

### Fama-French Three-Factor Model
Extension of CAPM incorporating size and value factors.

```python
class FamaFrenchThreeFactor:
    def __init__(self, risk_free_rate=0.02):
        self.risk_free_rate = risk_free_rate
        self.coefficients = None
        self.r_squared = None
    
    def prepare_factors(self, market_returns, size_factor, value_factor):
        """Prepare Fama-French factors"""
        # Market factor (excess market return)
        market_factor = market_returns - self.risk_free_rate/252
        
        # Size factor (SMB - Small Minus Big)
        smb_factor = size_factor
        
        # Value factor (HML - High Minus Low)
        hml_factor = value_factor
        
        return pd.DataFrame({
            'MKT': market_factor,
            'SMB': smb_factor,
            'HML': hml_factor
        }).dropna()
    
    def fit(self, asset_returns, market_returns, size_factor, value_factor):
        """Fit Fama-French three-factor model"""
        # Prepare factors
        factors = self.prepare_factors(market_returns, size_factor, value_factor)
        
        # Align dates
        common_dates = asset_returns.index.intersection(factors.index)
        asset_excess = (asset_returns - self.risk_free_rate/252).loc[common_dates]
        factors_aligned = factors.loc[common_dates]
        
        # Multiple regression
        X = factors_aligned.values
        y = asset_excess.values
        
        # Add intercept
        X_with_intercept = np.column_stack([np.ones(len(X)), X])
        
        # Ordinary least squares
        coefficients = np.linalg.lstsq(X_with_intercept, y, rcond=None)[0]
        
        self.coefficients = {
            'alpha': coefficients[0] * 252,  # Annualized
            'beta_mkt': coefficients[1],
            'beta_smb': coefficients[2],
            'beta_hml': coefficients[3]
        }
        
        # Calculate R-squared
        y_pred = X_with_intercept @ coefficients
        ss_res = np.sum((y - y_pred) ** 2)
        ss_tot = np.sum((y - np.mean(y)) ** 2)
        self.r_squared = 1 - (ss_res / ss_tot)
        
        return self.coefficients
    
    def predict_returns(self, market_factor, smb_factor, hml_factor):
        """Predict returns using fitted model"""
        if self.coefficients is None:
            raise ValueError("Model must be fitted first")
        
        excess_return = (
            self.coefficients['alpha'] +
            self.coefficients['beta_mkt'] * market_factor +
            self.coefficients['beta_smb'] * smb_factor +
            self.coefficients['beta_hml'] * hml_factor
        )
        
        return self.risk_free_rate + excess_return

# Example usage
ff_model = FamaFrenchThreeFactor()
# ff_results = ff_model.fit(stock_returns, market_returns, smb_data, hml_data)
```

## Multi-Factor Risk Models

### Barra Risk Factor Model
Industry-standard multi-factor risk model for portfolio risk management.

```python
class BarraRiskModel:
    def __init__(self, factors_config=None):
        if factors_config is None:
            self.factors_config = self.default_factors()
        else:
            self.factors_config = factors_config
        
        self.factor_loadings = {}
        self.factor_returns = {}
        self.specific_risk = {}
    
    def default_factors(self):
        """Default Barra-style factors"""
        return {
            'style_factors': [
                'BETA', 'MOMENTUM', 'SIZE', 'EARNINGS_YIELD', 
                'VOLATILITY', 'GROWTH', 'LEVERAGE', 'LIQUIDITY'
            ],
            'industry_factors': [
                'ENERGY', 'MATERIALS', 'INDUSTRIALS', 'CONSUMER_DISCRETIONARY',
                'CONSUMER_STAPLES', 'HEALTHCARE', 'FINANCIALS', 'INFORMATION_TECHNOLOGY',
                'COMMUNICATION_SERVICES', 'UTILITIES', 'REAL_ESTATE'
            ],
            'country_factors': ['US', 'INTERNATIONAL']
        }
    
    def calculate_style_factors(self, stock_data):
        """Calculate style factor exposures"""
        factors = {}
        
        # Beta (market sensitivity)
        factors['BETA'] = self.calculate_beta(stock_data['returns'], stock_data['market_returns'])
        
        # Momentum (12-1 month momentum)
        factors['MOMENTUM'] = self.calculate_momentum(stock_data['prices'])
        
        # Size (log market capitalization)
        factors['SIZE'] = np.log(stock_data['market_cap'])
        
        # Earnings yield (E/P ratio)
        factors['EARNINGS_YIELD'] = stock_data['earnings'] / stock_data['market_cap']
        
        # Volatility (realized volatility)
        factors['VOLATILITY'] = stock_data['returns'].rolling(252).std()
        
        # Growth (earnings growth rate)
        factors['GROWTH'] = stock_data['earnings'].pct_change(4)  # Quarterly growth
        
        # Leverage (debt-to-equity ratio)
        factors['LEVERAGE'] = stock_data['total_debt'] / stock_data['total_equity']
        
        # Liquidity (turnover ratio)
        factors['LIQUIDITY'] = stock_data['volume'] / stock_data['shares_outstanding']
        
        return pd.DataFrame(factors)
    
    def calculate_industry_factors(self, stock_data):
        """Calculate industry factor exposures"""
        industries = pd.get_dummies(stock_data['sector'])
        return industries
    
    def calculate_beta(self, stock_returns, market_returns, window=252):
        """Calculate rolling beta"""
        betas = []
        for i in range(window, len(stock_returns)):
            stock_window = stock_returns.iloc[i-window:i]
            market_window = market_returns.iloc[i-window:i]
            
            covariance = np.cov(stock_window, market_window)[0, 1]
            market_variance = np.var(market_window)
            
            beta = covariance / market_variance if market_variance != 0 else 0
            betas.append(beta)
        
        # Pad with NaN for initial period
        return pd.Series([np.nan] * window + betas, index=stock_returns.index)
    
    def calculate_momentum(self, prices, lookback=252, skip=21):
        """Calculate momentum factor (12-1 month)"""
        if len(prices) < lookback + skip:
            return pd.Series([np.nan] * len(prices), index=prices.index)
        
        momentum = []
        for i in range(lookback + skip, len(prices)):
            past_return = (prices.iloc[i-skip] / prices.iloc[i-lookback-skip]) - 1
            momentum.append(past_return)
        
        # Pad with NaN
        padding = [np.nan] * (lookback + skip)
        return pd.Series(padding + momentum, index=prices.index)
    
    def estimate_factor_returns(self, returns, factor_loadings, method='OLS'):
        """Estimate factor returns using cross-sectional regression"""
        # Align data
        common_dates = returns.index.intersection(factor_loadings.index)
        returns_aligned = returns.loc[common_dates]
        factors_aligned = factor_loadings.loc[common_dates]
        
        factor_returns = {}
        
        for date in common_dates:
            date_returns = returns_aligned.loc[date].dropna()
            date_factors = factors_aligned.loc[date].dropna()
            
            # Get common assets
            common_assets = date_returns.index.intersection(date_factors.index)
            
            if len(common_assets) > len(self.factors_config['style_factors']) + 5:
                X = date_factors.loc[common_assets]
                y = date_returns.loc[common_assets]
                
                # Add intercept
                X_matrix = np.column_stack([np.ones(len(X)), X])
                
                # Regression
                try:
                    coefficients = np.linalg.lstsq(X_matrix, y, rcond=None)[0]
                    factor_returns[date] = coefficients[1:]  # Exclude intercept
                except:
                    factor_returns[date] = np.zeros(len(self.factors_config['style_factors']))
            else:
                factor_returns[date] = np.zeros(len(self.factors_config['style_factors']))
        
        return pd.DataFrame(factor_returns).T
    
    def calculate_portfolio_risk(self, portfolio_weights, factor_loadings, factor_covariance):
        """Calculate portfolio risk using factor model"""
        # Factor risk
        portfolio_factor_exposures = portfolio_weights @ factor_loadings
        factor_risk = portfolio_factor_exposures @ factor_covariance @ portfolio_factor_exposures
        
        # Specific risk (assuming diagonal)
        specific_variances = np.diag(self.specific_risk)
        specific_risk = portfolio_weights @ specific_variances @ portfolio_weights
        
        total_risk = np.sqrt(factor_risk + specific_risk)
        return total_risk

# Example usage
barra_model = BarraRiskModel()
# style_factors = barra_model.calculate_style_factors(stock_data)
```

## Smart Beta and Factor-Based Investing

### Multi-Factor Portfolio Construction
```python
class MultiFactorPortfolio:
    def __init__(self, factors_weights=None):
        if factors_weights is None:
            # Equal weighting by default
            self.factors_weights = {
                'momentum': 0.25,
                'value': 0.25,
                'quality': 0.25,
                'low_volatility': 0.25
            }
        else:
            self.factors_weights = factors_weights
    
    def calculate_factor_scores(self, stock_data):
        """Calculate composite factor scores"""
        factor_scores = {}
        
        # Momentum factor
        factor_scores['momentum'] = self.calculate_momentum_score(stock_data['prices'])
        
        # Value factor
        factor_scores['value'] = self.calculate_value_score(
            stock_data['pe_ratio'], 
            stock_data['pb_ratio']
        )
        
        # Quality factor
        factor_scores['quality'] = self.calculate_quality_score(
            stock_data['roa'], 
            stock_data['debt_ratio']
        )
        
        # Low volatility factor
        factor_scores['low_volatility'] = self.calculate_volatility_score(
            stock_data['returns']
        )
        
        return pd.DataFrame(factor_scores)
    
    def calculate_momentum_score(self, prices, lookback=252):
        """Calculate momentum score"""
        momentum = (prices / prices.shift(lookback)) - 1
        # Rank and normalize to 0-1
        return (momentum.rank() - 1) / (len(momentum) - 1)
    
    def calculate_value_score(self, pe_ratio, pb_ratio):
        """Calculate value score (lower ratios = higher value)"""
        pe_score = 1 - (pe_ratio.rank() / len(pe_ratio))
        pb_score = 1 - (pb_ratio.rank() / len(pb_ratio))
        return (pe_score + pb_score) / 2
    
    def calculate_quality_score(self, roa, debt_ratio):
        """Calculate quality score"""
        roa_score = roa.rank() / len(roa)
        debt_score = 1 - (debt_ratio.rank() / len(debt_ratio))
        return (roa_score + debt_score) / 2
    
    def calculate_volatility_score(self, returns, window=252):
        """Calculate low volatility score (lower volatility = higher score)"""
        volatility = returns.rolling(window).std()
        return 1 - (volatility.rank() / len(volatility))
    
    def construct_portfolio(self, stock_data, portfolio_size=50):
        """Construct multi-factor portfolio"""
        # Calculate factor scores
        factor_scores = self.calculate_factor_scores(stock_data)
        
        # Calculate composite score
        composite_score = sum(
            factor_scores[factor] * weight 
            for factor, weight in self.factors_weights.items()
        )
        
        # Select top stocks
        top_stocks = composite_score.nlargest(portfolio_size)
        
        # Equal weighting
        weights = pd.Series(1/portfolio_size, index=top_stocks.index)
        
        return {
            'weights': weights,
            'selected_stocks': top_stocks.index.tolist(),
            'factor_exposures': factor_scores.loc[top_stocks.index].mean(),
            'expected_return': self.estimate_expected_return(stock_data['returns'], weights)
        }
    
    def estimate_expected_return(self, returns, weights):
        """Estimate portfolio expected return"""
        portfolio_returns = (returns * weights).sum(axis=1)
        return portfolio_returns.mean() * 252  # Annualized

# Example usage
multi_factor = MultiFactorPortfolio()
# portfolio = multi_factor.construct_portfolio(stock_data)
```

## Risk Parity and Factor-Based Optimization

### Risk Parity Portfolio
```python
class RiskParityOptimizer:
    def __init__(self, target_risk=0.1):
        self.target_risk = target_risk
    
    def optimize_risk_parity(self, expected_returns, covariance_matrix):
        """Optimize risk parity portfolio"""
        n_assets = len(expected_returns)
        
        def objective(weights):
            # Portfolio risk
            portfolio_risk = np.sqrt(weights @ covariance_matrix @ weights)
            
            # Risk contribution of each asset
            marginal_risk = (covariance_matrix @ weights) / portfolio_risk
            risk_contribution = weights * marginal_risk
            
            # Target: equal risk contribution
            target_contribution = portfolio_risk / n_assets
            return np.sum((risk_contribution - target_contribution) ** 2)
        
        # Constraints
        constraints = [{'type': 'eq', 'fun': lambda x: np.sum(x) - 1}]
        bounds = tuple((0, 1) for _ in range(n_assets))
        
        # Initial guess
        initial_weights = np.array(n_assets * [1. / n_assets])
        
        # Optimize
        result = minimize(objective, initial_weights, method='SLSQP',
                         bounds=bounds, constraints=constraints)
        
        # Scale to target risk level
        optimal_weights = result.x
        current_risk = np.sqrt(optimal_weights @ covariance_matrix @ optimal_weights)
        scaled_weights = optimal_weights * (self.target_risk / current_risk)
        
        return scaled_weights
    
    def factor_risk_parity(self, factor_returns, factor_covariance):
        """Risk parity across factors"""
        n_factors = len(factor_returns.columns)
        
        def factor_objective(weights):
            portfolio_factor_exposures = weights
            portfolio_risk = np.sqrt(
                portfolio_factor_exposures @ factor_covariance @ portfolio_factor_exposures
            )
            
            marginal_risk = (factor_covariance @ weights) / portfolio_risk
            risk_contribution = weights * marginal_risk
            
            target_contribution = portfolio_risk / n_factors
            return np.sum((risk_contribution - target_contribution) ** 2)
        
        constraints = [{'type': 'eq', 'fun': lambda x: np.sum(x) - 1}]
        bounds = tuple((0, 1) for _ in range(n_factors))
        initial_weights = np.array(n_factors * [1. / n_factors])
        
        result = minimize(factor_objective, initial_weights, method='SLSQP',
                         bounds=bounds, constraints=constraints)
        
        return result.x

# Example usage
risk_parity_optimizer = RiskParityOptimizer(target_risk=0.15)
# optimal_weights = risk_parity_optimizer.optimize_risk_parity(expected_returns, cov_matrix)
```

## Performance Attribution

### Brinson Model for Performance Attribution
```python
class PerformanceAttribution:
    def __init__(self):
        pass
    
    def brinson_attribution(self, portfolio_returns, benchmark_returns, 
                           portfolio_weights, benchmark_weights, sectors):
        """Brinson performance attribution model"""
        
        # Calculate sector returns
        sector_portfolio_returns = self.calculate_sector_returns(
            portfolio_returns, portfolio_weights, sectors
        )
        sector_benchmark_returns = self.calculate_sector_returns(
            benchmark_returns, benchmark_weights, sectors
        )
        
        # Overall portfolio and benchmark returns
        portfolio_return = (portfolio_returns * portfolio_weights).sum()
        benchmark_return = (benchmark_returns * benchmark_weights).sum()
        
        # Attribution components
        allocation_effect = self.calculate_allocation_effect(
            portfolio_weights, benchmark_weights, sector_benchmark_returns
        )
        
        selection_effect = self.calculate_selection_effect(
            portfolio_weights, benchmark_weights, 
            sector_portfolio_returns, sector_benchmark_returns
        )
        
        interaction_effect = self.calculate_interaction_effect(
            portfolio_weights, benchmark_weights,
            sector_portfolio_returns, sector_benchmark_returns
        )
        
        return {
            'total_active_return': portfolio_return - benchmark_return,
            'allocation_effect': allocation_effect,
            'selection_effect': selection_effect,
            'interaction_effect': interaction_effect
        }
    
    def calculate_sector_returns(self, returns, weights, sectors):
        """Calculate returns by sector"""
        sector_returns = {}
        for sector in sectors.unique():
            sector_mask = sectors == sector
            sector_weights = weights[sector_mask]
            sector_returns_data = returns[sector_mask]
            
            if len(sector_weights) > 0:
                sector_return = (sector_returns_data * sector_weights).sum() / sector_weights.sum()
                sector_returns[sector] = sector_return
            else:
                sector_returns[sector] = 0
                
        return pd.Series(sector_returns)
    
    def calculate_allocation_effect(self, port_weights, bench_weights, sector_bench_returns):
        """Calculate allocation effect"""
        weight_difference = port_weights.groupby(sectors).sum() - bench_weights.groupby(sectors).sum()
        return (weight_difference * sector_bench_returns).sum()
    
    def calculate_selection_effect(self, port_weights, bench_weights, 
                                 sector_port_returns, sector_bench_returns):
        """Calculate selection effect"""
        return_effect = sector_port_returns - sector_bench_returns
        bench_weights_sector = bench_weights.groupby(sectors).sum()
        return (bench_weights_sector * return_effect).sum()
    
    def calculate_interaction_effect(self, port_weights, bench_weights,
                                   sector_port_returns, sector_bench_returns):
        """Calculate interaction effect"""
        weight_diff = port_weights.groupby(sectors).sum() - bench_weights.groupby(sectors).sum()
        return_diff = sector_port_returns - sector_bench_returns
        return (weight_diff * return_diff).sum()

# Example usage
attribution = PerformanceAttribution()
# results = attribution.brinson_attribution(port_returns, bench_returns, port_weights, bench_weights, sectors)
```

This comprehensive factor models documentation covers classic models like CAPM and Fama-French, advanced risk models like Barra, smart beta strategies, risk parity optimization, and performance attribution techniques essential for quantitative portfolio management.