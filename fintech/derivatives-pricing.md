# Derivatives Pricing

Comprehensive guide to derivatives pricing, Greeks calculation, and risk management for options, futures, and other derivative instruments.

## Overview

Derivatives pricing involves sophisticated mathematical models to value complex financial instruments whose payoffs depend on underlying assets, interest rates, or other market variables.

## Options Pricing Theory

### Black-Scholes-Merton Model
```python
import numpy as np
from scipy.stats import norm
from scipy.optimize import brentq
import matplotlib.pyplot as plt

class BlackScholesModel:
    def __init__(self):
        pass
    
    def black_scholes_call(self, S, K, T, r, sigma, q=0):
        """Calculate European call option price using Black-Scholes"""
        d1 = (np.log(S/K) + (r - q + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
        d2 = d1 - sigma * np.sqrt(T)
        
        call_price = S * np.exp(-q * T) * norm.cdf(d1) - K * np.exp(-r * T) * norm.cdf(d2)
        return call_price
    
    def black_scholes_put(self, S, K, T, r, sigma, q=0):
        """Calculate European put option price using Black-Scholes"""
        d1 = (np.log(S/K) + (r - q + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
        d2 = d1 - sigma * np.sqrt(T)
        
        put_price = K * np.exp(-r * T) * norm.cdf(-d2) - S * np.exp(-q * T) * norm.cdf(-d1)
        return put_price
    
    def calculate_greeks(self, S, K, T, r, sigma, option_type='call', q=0):
        """Calculate option Greeks"""
        d1 = (np.log(S/K) + (r - q + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
        d2 = d1 - sigma * np.sqrt(T)
        
        # Delta
        if option_type == 'call':
            delta = np.exp(-q * T) * norm.cdf(d1)
        else:
            delta = np.exp(-q * T) * (norm.cdf(d1) - 1)
        
        # Gamma
        gamma = np.exp(-q * T) * norm.pdf(d1) / (S * sigma * np.sqrt(T))
        
        # Theta (per year)
        if option_type == 'call':
            theta = (-S * np.exp(-q * T) * norm.pdf(d1) * sigma / (2 * np.sqrt(T)) 
                    - r * K * np.exp(-r * T) * norm.cdf(d2)
                    + q * S * np.exp(-q * T) * norm.cdf(d1))
        else:
            theta = (-S * np.exp(-q * T) * norm.pdf(d1) * sigma / (2 * np.sqrt(T)) 
                    + r * K * np.exp(-r * T) * norm.cdf(-d2)
                    - q * S * np.exp(-q * T) * norm.cdf(-d1))
        
        # Vega
        vega = S * np.exp(-q * T) * norm.pdf(d1) * np.sqrt(T)
        
        # Rho (per 1% change in rate)
        if option_type == 'call':
            rho = K * T * np.exp(-r * T) * norm.cdf(d2)
        else:
            rho = -K * T * np.exp(-r * T) * norm.cdf(-d2)
        
        return {
            'delta': delta,
            'gamma': gamma,
            'theta': theta / 365,  # Per day
            'vega': vega / 100,    # Per 1% vol change
            'rho': rho / 100       # Per 1% rate change
        }
    
    def implied_volatility(self, market_price, S, K, T, r, option_type='call', q=0, 
                          initial_guess=0.2, tolerance=1e-8):
        """Calculate implied volatility using Newton-Raphson method"""
        def price_error(sigma):
            if option_type == 'call':
                model_price = self.black_scholes_call(S, K, T, r, sigma, q)
            else:
                model_price = self.black_scholes_put(S, K, T, r, sigma, q)
            return model_price - market_price
        
        try:
            implied_vol = brentq(price_error, 0.001, 5.0, xtol=tolerance)
            return implied_vol
        except:
            # Fallback to bisection method
            return self.bisection_method(price_error, 0.001, 5.0, tolerance)
    
    def bisection_method(self, func, a, b, tolerance=1e-8, max_iterations=1000):
        """Bisection method for finding roots"""
        if func(a) * func(b) >= 0:
            raise ValueError("Function must have opposite signs at endpoints")
        
        for _ in range(max_iterations):
            c = (a + b) / 2
            if abs(func(c)) < tolerance or (b - a) / 2 < tolerance:
                return c
            if func(c) * func(a) < 0:
                b = c
            else:
                a = c
        
        return (a + b) / 2

# Example usage
bs_model = BlackScholesModel()
# call_price = bs_model.black_scholes_call(100, 105, 1, 0.05, 0.2)
# greeks = bs_model.calculate_greeks(100, 105, 1, 0.05, 0.2, 'call')
# implied_vol = bs_model.implied_volatility(8.5, 100, 105, 1, 0.05, 'call')
```

### Binomial Option Pricing
```python
class BinomialOptionPricer:
    def __init__(self, steps=100):
        self.steps = steps
    
    def binomial_call_option(self, S, K, T, r, sigma, steps=None):
        """Price European call option using binomial model"""
        if steps is None:
            steps = self.steps
            
        dt = T / steps
        u = np.exp(sigma * np.sqrt(dt))
        d = 1 / u
        p = (np.exp(r * dt) - d) / (u - d)
        
        # Initialize option values at maturity
        option_values = np.zeros(steps + 1)
        stock_prices = np.zeros(steps + 1)
        
        # Calculate stock prices at maturity
        for i in range(steps + 1):
            stock_prices[i] = S * (u ** (steps - i)) * (d ** i)
            option_values[i] = max(0, stock_prices[i] - K)
        
        # Backward induction
        for step in range(steps - 1, -1, -1):
            for i in range(step + 1):
                option_values[i] = np.exp(-r * dt) * (p * option_values[i] + (1 - p) * option_values[i + 1])
        
        return option_values[0]
    
    def binomial_american_put(self, S, K, T, r, sigma, steps=None):
        """Price American put option using binomial model"""
        if steps is None:
            steps = self.steps
            
        dt = T / steps
        u = np.exp(sigma * np.sqrt(dt))
        d = 1 / u
        p = (np.exp(r * dt) - d) / (u - d)
        
        # Initialize option values
        option_values = np.zeros((steps + 1, steps + 1))
        stock_prices = np.zeros((steps + 1, steps + 1))
        
        # Calculate stock prices and option values at maturity
        for i in range(steps + 1):
            for j in range(i + 1):
                stock_prices[j, i] = S * (u ** (i - j)) * (d ** j)
                option_values[j, i] = max(0, K - stock_prices[j, i])
        
        # Backward induction with early exercise
        for i in range(steps - 1, -1, -1):
            for j in range(i + 1):
                continuation_value = np.exp(-r * dt) * (
                    p * option_values[j, i + 1] + (1 - p) * option_values[j + 1, i + 1]
                )
                exercise_value = max(0, K - stock_prices[j, i])
                option_values[j, i] = max(continuation_value, exercise_value)
        
        return option_values[0, 0]
    
    def calculate_binomial_greeks(self, S, K, T, r, sigma, option_type='call', steps=None):
        """Calculate Greeks using binomial model"""
        if steps is None:
            steps = self.steps
            
        # Base price
        if option_type == 'call':
            base_price = self.binomial_call_option(S, K, T, r, sigma, steps)
        else:
            base_price = self.binomial_american_put(S, K, T, r, sigma, steps)
        
        # Delta (finite difference)
        dS = S * 0.01
        if option_type == 'call':
            price_up = self.binomial_call_option(S + dS, K, T, r, sigma, steps)
            price_down = self.binomial_call_option(S - dS, K, T, r, sigma, steps)
        else:
            price_up = self.binomial_american_put(S + dS, K, T, r, sigma, steps)
            price_down = self.binomial_american_put(S - dS, K, T, r, sigma, steps)
        
        delta = (price_up - price_down) / (2 * dS)
        
        # Gamma
        gamma = (price_up - 2 * base_price + price_down) / (dS ** 2)
        
        # Theta
        dt = T / (steps * 10)  # Small time increment
        if option_type == 'call':
            price_future = self.binomial_call_option(S, K, T - dt, r, sigma, steps)
        else:
            price_future = self.binomial_american_put(S, K, T - dt, r, sigma, steps)
        
        theta = (price_future - base_price) / dt
        
        return {
            'delta': delta,
            'gamma': gamma,
            'theta': theta
        }

# Example usage
binomial_model = BinomialOptionPricer(steps=100)
# american_put = binomial_model.binomial_american_put(100, 105, 1, 0.05, 0.2)
```

## Advanced Derivatives Models

### Stochastic Volatility Models
```python
class HestonModel:
    def __init__(self):
        pass
    
    def heston_characteristic_function(self, phi, S, K, T, r, v0, kappa, theta, sigma, rho):
        """Heston model characteristic function"""
        # Model parameters
        a = kappa * theta
        b = kappa + rho * sigma * phi * 1j
        c = sigma ** 2 / 2
        
        d = np.sqrt((rho * sigma * phi * 1j - b) ** 2 - c * (2j * phi - phi ** 2))
        g = (b - rho * sigma * phi * 1j + d) / (b - rho * sigma * phi * 1j - d)
        
        # Characteristic function
        C = (r * phi * 1j * T + 
             (a / c) * ((b - rho * sigma * phi * 1j + d) * T - 2 * np.log((1 - g * np.exp(d * T)) / (1 - g))))
        D = (b - rho * sigma * phi * 1j + d) / c * ((1 - np.exp(d * T)) / (1 - g * np.exp(d * T)))
        
        return np.exp(C + D * v0)
    
    def heston_call_price(self, S, K, T, r, v0, kappa, theta, sigma, rho, integration_points=100):
        """Price call option using Heston model"""
        # Integration using FFT-like approach
        phi = np.linspace(0.0001, 100, integration_points)
        dphi = phi[1] - phi[0]
        
        # Characteristic function values
        char_func_values = np.array([
            self.heston_characteristic_function(p, S, K, T, r, v0, kappa, theta, sigma, rho) 
            for p in phi
        ])
        
        # Integration
        integrand = np.real(np.exp(-1j * phi * np.log(K)) * char_func_values / (1j * phi))
        option_price = S / 2 - np.exp(-r * T) * K / np.pi * np.trapz(integrand, phi)
        
        return max(0, option_price)
    
    def simulate_heston_paths(self, S0, v0, kappa, theta, sigma, rho, r, T, n_steps, n_paths):
        """Simulate paths using Heston model"""
        dt = T / n_steps
        sqrt_dt = np.sqrt(dt)
        
        # Pre-allocate arrays
        S_paths = np.zeros((n_paths, n_steps + 1))
        v_paths = np.zeros((n_paths, n_steps + 1))
        
        S_paths[:, 0] = S0
        v_paths[:, 0] = v0
        
        # Cholesky decomposition for correlated Brownian motions
        cov_matrix = np.array([[1, rho], [rho, 1]])
        L = np.linalg.cholesky(cov_matrix)
        
        for i in range(1, n_steps + 1):
            # Generate correlated random numbers
            Z = np.random.normal(0, 1, (n_paths, 2))
            W_S = L[0, 0] * Z[:, 0] + L[0, 1] * Z[:, 1]
            W_v = L[1, 0] * Z[:, 0] + L[1, 1] * Z[:, 1]
            
            # Update variance (ensure positivity)
            dv = kappa * (theta - np.maximum(v_paths[:, i-1], 0)) * dt + sigma * np.sqrt(np.maximum(v_paths[:, i-1], 0)) * W_v * sqrt_dt
            v_paths[:, i] = np.maximum(v_paths[:, i-1] + dv, 0)
            
            # Update stock price
            dS = r * S_paths[:, i-1] * dt + np.sqrt(np.maximum(v_paths[:, i-1], 0)) * S_paths[:, i-1] * W_S * sqrt_dt
            S_paths[:, i] = S_paths[:, i-1] * np.exp(dS)
        
        return S_paths, v_paths

# Example usage
heston_model = HestonModel()
# call_price = heston_model.heston_call_price(S, K, T, r, v0, kappa, theta, sigma, rho)
```

## Greeks Calculation and Risk Management

### Comprehensive Greeks Calculator
```python
class GreeksCalculator:
    def __init__(self):
        self.bs_model = BlackScholesModel()
        self.binomial_model = BinomialOptionPricer()
    
    def calculate_all_greeks(self, S, K, T, r, sigma, option_type='call', model='black_scholes', q=0):
        """Calculate comprehensive Greeks using specified model"""
        if model == 'black_scholes':
            return self.bs_model.calculate_greeks(S, K, T, r, sigma, option_type, q)
        elif model == 'binomial':
            return self.binomial_model.calculate_binomial_greeks(S, K, T, r, sigma, option_type)
        else:
            raise ValueError("Unsupported model type")
    
    def calculate_second_order_greeks(self, S, K, T, r, sigma, option_type='call', q=0):
        """Calculate second-order Greeks (Gamma, Vanna, Volga, etc.)"""
        # Small perturbations for finite difference
        dS = S * 0.01
        dsigma = sigma * 0.01
        dr = 0.0001
        
        # Base Greeks
        base_greeks = self.calculate_all_greeks(S, K, T, r, sigma, option_type, 'black_scholes', q)
        
        # Vanna (∂²P/∂S∂σ)
        greeks_up_vol = self.calculate_all_greeks(S + dS, K, T, r, sigma + dsigma, option_type, 'black_scholes', q)
        greeks_down_vol = self.calculate_all_greeks(S - dS, K, T, r, sigma + dsigma, option_type, 'black_scholes', q)
        vanna = (greeks_up_vol['delta'] - greeks_down_vol['delta']) / (2 * dS)
        
        # Volga (∂²P/∂σ²)
        greeks_up_sigma = self.calculate_all_greeks(S, K, T, r, sigma + dsigma, option_type, 'black_scholes', q)
        greeks_down_sigma = self.calculate_all_greeks(S, K, T, r, sigma - dsigma, option_type, 'black_scholes', q)
        volga = (greeks_up_sigma['vega'] - greeks_down_sigma['vega']) / (2 * dsigma * 100)
        
        # Charm (∂Δ/∂t)
        dt = 1/365
        greeks_future = self.calculate_all_greeks(S, K, T - dt, r, sigma, option_type, 'black_scholes', q)
        charm = (greeks_future['delta'] - base_greeks['delta']) / dt
        
        # Speed (∂³P/∂S³)
        greeks_up2 = self.calculate_all_greeks(S + 2*dS, K, T, r, sigma, option_type, 'black_scholes', q)
        greeks_down2 = self.calculate_all_greeks(S - 2*dS, K, T, r, sigma, option_type, 'black_scholes', q)
        speed = (greeks_up2['gamma'] - 2 * base_greeks['gamma'] + greeks_down2['gamma']) / (dS ** 2)
        
        return {
            'gamma': base_greeks['gamma'],
            'vanna': vanna,
            'volga': volga,
            'charm': charm,
            'speed': speed,
            'color': (greeks_future['gamma'] - base_greeks['gamma']) / dt,  # ∂Γ/∂t
            'ultima': (greeks_up_sigma['volga'] - greeks_down_sigma['volga']) / (2 * dsigma * 100)  # ∂Volga/∂σ
        }
    
    def greek_surface_plot(self, S_range, sigma_range, K, T, r, option_type='call'):
        """Generate 3D surface plots of Greeks"""
        S_grid, sigma_grid = np.meshgrid(S_range, sigma_range)
        delta_surface = np.zeros_like(S_grid)
        gamma_surface = np.zeros_like(S_grid)
        vega_surface = np.zeros_like(S_grid)
        
        for i in range(len(sigma_range)):
            for j in range(len(S_range)):
                greeks = self.calculate_all_greeks(
                    S_grid[i, j], K, T, r, sigma_grid[i, j], option_type
                )
                delta_surface[i, j] = greeks['delta']
                gamma_surface[i, j] = greeks['gamma']
                vega_surface[i, j] = greeks['vega']
        
        return {
            'S_grid': S_grid,
            'sigma_grid': sigma_grid,
            'delta_surface': delta_surface,
            'gamma_surface': gamma_surface,
            'vega_surface': vega_surface
        }

# Example usage
greeks_calc = GreeksCalculator()
# all_greeks = greeks_calc.calculate_all_greeks(100, 105, 1, 0.05, 0.2, 'call')
# second_order = greeks_calc.calculate_second_order_greeks(100, 105, 1, 0.05, 0.2, 'call')
```

## Exotic Options Pricing

### Barrier Options
```python
class BarrierOptionPricer:
    def __init__(self):
        pass
    
    def up_and_out_call(self, S, K, H, T, r, sigma, q=0):
        """Price up-and-out call barrier option"""
        # Using Black-Scholes with barrier adjustment
        mu = (r - q - 0.5 * sigma**2) / sigma**2
        lam = np.sqrt(mu**2 + 2*r/sigma**2)
        
        x1 = (np.log(S/H) + (mu + 1) * sigma**2 * T) / (sigma * np.sqrt(T))
        y1 = (np.log(H/S) + (mu + 1) * sigma**2 * T) / (sigma * np.sqrt(T))
        
        # Standard call price
        bs_call = BlackScholesModel().black_scholes_call(S, K, T, r, sigma, q)
        
        # Barrier adjustment
        barrier_adj = (H/S) ** (2*mu) * BlackScholesModel().black_scholes_call(
            H**2/S, K, T, r, sigma, q
        )
        
        return bs_call - barrier_adj
    
    def down_and_in_put(self, S, K, H, T, r, sigma, q=0):
        """Price down-and-in put barrier option"""
        # Relationship: DI put = Vanilla put - DO put
        vanilla_put = BlackScholesModel().black_scholes_put(S, K, T, r, sigma, q)
        do_put = self.down_and_out_put(S, K, H, T, r, sigma, q)
        return vanilla_put - do_put
    
    def down_and_out_put(self, S, K, H, T, r, sigma, q=0):
        """Price down-and-out put barrier option"""
        if S <= H:  # Barrier already hit
            return 0
            
        mu = (r - q - 0.5 * sigma**2) / sigma**2
        lam = np.sqrt(mu**2 + 2*r/sigma**2)
        
        y = (np.log(H/S) + (mu + 1) * sigma**2 * T) / (sigma * np.sqrt(T))
        x2 = (np.log(S/K) + (mu + 1) * sigma**2 * T) / (sigma * np.sqrt(T))
        
        # Standard put price
        bs_put = BlackScholesModel().black_scholes_put(S, K, T, r, sigma, q)
        
        # Barrier adjustment
        barrier_term = (H/S) ** (2*mu) * BlackScholesModel().black_scholes_put(
            H**2/S, K, T, r, sigma, q
        )
        
        return bs_put - barrier_term

# Example usage
barrier_pricer = BarrierOptionPricer()
# barrier_call = barrier_pricer.up_and_out_call(100, 105, 120, 1, 0.05, 0.2)
```

## Risk Management Applications

### Options Portfolio Risk Management
```python
class OptionsPortfolioManager:
    def __init__(self):
        self.greeks_calculator = GreeksCalculator()
        self.positions = {}
    
    def add_position(self, option_id, S, K, T, r, sigma, quantity, option_type='call', q=0):
        """Add option position to portfolio"""
        greeks = self.greeks_calculator.calculate_all_greeks(
            S, K, T, r, sigma, option_type, 'black_scholes', q
        )
        
        self.positions[option_id] = {
            'underlying_price': S,
            'strike': K,
            'time_to_expiry': T,
            'risk_free_rate': r,
            'volatility': sigma,
            'quantity': quantity,
            'option_type': option_type,
            'dividend_yield': q,
            'greeks': greeks
        }
    
    def calculate_portfolio_greeks(self):
        """Calculate aggregate portfolio Greeks"""
        portfolio_greeks = {
            'delta': 0, 'gamma': 0, 'theta': 0, 'vega': 0, 'rho': 0
        }
        
        for position in self.positions.values():
            qty = position['quantity']
            for greek in portfolio_greeks:
                portfolio_greeks[greek] += qty * position['greeks'][greek]
        
        return portfolio_greeks
    
    def calculate_var(self, confidence_level=0.95, time_horizon=1):
        """Calculate Value at Risk for options portfolio"""
        portfolio_greeks = self.calculate_portfolio_greeks()
        
        # Simplified delta-normal VaR
        delta = portfolio_greeks['delta']
        gamma = portfolio_greeks['gamma']
        vega = portfolio_greesks['vega']
        
        # Assumptions for VaR calculation
        underlying_vol = 0.02  # 2% daily vol
        vol_vol = 0.1  # 10% vol of vol
        
        # Delta component
        delta_var = abs(delta) * underlying_vol * np.sqrt(time_horizon)
        
        # Gamma component (approximate)
        gamma_var = 0.5 * abs(gamma) * (underlying_vol ** 2) * time_horizon
        
        # Vega component
        vega_var = abs(vega) * vol_vol * np.sqrt(time_horizon)
        
        # Total VaR
        total_var = delta_var + gamma_var + vega_var
        var_at_confidence = total_var * norm.ppf(confidence_level)
        
        return var_at_confidence
    
    def hedge_portfolio(self, target_delta=0, target_gamma=0):
        """Calculate hedging requirements"""
        portfolio_greeks = self.calculate_portfolio_greeks()
        
        # Current exposure
        current_delta = portfolio_greeks['delta']
        current_gamma = portfolio_greeks['gamma']
        
        # Hedging requirements
        delta_hedge_needed = target_delta - current_delta
        gamma_hedge_needed = target_gamma - current_gamma
        
        return {
            'delta_hedge_required': delta_hedge_needed,
            'gamma_hedge_required': gamma_hedge_needed,
            'hedging_strategy': self.determine_hedging_approach(delta_hedge_needed, gamma_hedge_needed)
        }
    
    def determine_hedging_approach(self, delta_req, gamma_req):
        """Determine appropriate hedging approach"""
        strategy = {}
        
        if abs(delta_req) > 0:
            strategy['underlying_hedge'] = delta_req
            
        if abs(gamma_req) > 0:
            # Need options to hedge gamma
            strategy['atm_options_needed'] = gamma_req / 0.05  # Approximate gamma of ATM options
            
        return strategy

# Example usage
portfolio_manager = OptionsPortfolioManager()
# portfolio_manager.add_position('option1', 100, 105, 1, 0.05, 0.2, 100, 'call')
# portfolio_greeks = portfolio_manager.calculate_portfolio_greeks()
```

This comprehensive derivatives pricing documentation covers Black-Scholes model, binomial pricing, stochastic volatility models, Greeks calculation, exotic options, and risk management applications essential for derivatives quantitative analysis.