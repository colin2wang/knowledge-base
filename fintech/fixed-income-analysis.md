# Fixed Income Analysis

Comprehensive guide to bond pricing, yield curve modeling, and fixed income quantitative analysis for institutional investors and portfolio managers.

## Overview

Fixed income analysis involves the valuation and risk management of debt securities, including government bonds, corporate bonds, mortgage-backed securities, and other fixed-income instruments.

## Bond Pricing Fundamentals

### Present Value Pricing Model
```python
import numpy as np
import pandas as pd
from scipy.optimize import newton
import matplotlib.pyplot as plt

class BondPricer:
    def __init__(self):
        pass
    
    def price_bond(self, cash_flows, discount_rates, settlement_date=None):
        """Price bond using present value of cash flows"""
        if settlement_date is None:
            settlement_date = 0
            
        # Calculate time periods from settlement
        time_periods = np.array(range(len(cash_flows))) + 1 - settlement_date
        
        # Present value calculation
        present_values = cash_flows / ((1 + discount_rates) ** time_periods)
        bond_price = np.sum(present_values)
        
        return bond_price
    
    def price_zero_coupon_bond(self, face_value, yield_to_maturity, time_to_maturity):
        """Price zero-coupon bond"""
        return face_value / ((1 + yield_to_maturity) ** time_to_maturity)
    
    def price_coupon_bond(self, face_value, coupon_rate, yield_to_maturity, 
                         time_to_maturity, frequency=2):
        """Price coupon-paying bond"""
        # Calculate periodic coupon payment
        coupon_payment = face_value * coupon_rate / frequency
        periods = int(time_to_maturity * frequency)
        
        # Cash flows
        cash_flows = np.full(periods, coupon_payment)
        cash_flows[-1] += face_value  # Add principal repayment
        
        # Discount rates (convert annual YTM to periodic)
        periodic_ytm = yield_to_maturity / frequency
        
        # Time periods
        time_periods = np.arange(1, periods + 1) / frequency
        
        # Present value
        present_values = cash_flows / ((1 + periodic_ytm) ** (np.arange(1, periods + 1)))
        bond_price = np.sum(present_values)
        
        return bond_price
    
    def calculate_yield_to_maturity(self, market_price, face_value, coupon_rate, 
                                  time_to_maturity, frequency=2, guess=0.05):
        """Calculate YTM using numerical methods"""
        def price_error(ytm):
            calculated_price = self.price_coupon_bond(
                face_value, coupon_rate, ytm, time_to_maturity, frequency
            )
            return calculated_price - market_price
        
        try:
            ytm = newton(price_error, guess, maxiter=100)
            return ytm
        except:
            # Fallback to bisection method
            return self.bisection_method(price_error, 0.001, 1.0)
    
    def bisection_method(self, func, a, b, tolerance=1e-8, max_iterations=1000):
        """Bisection method for finding roots"""
        if func(a) * func(b) >= 0:
            raise ValueError("Function must have opposite signs at endpoints")
        
        for _ in range(max_iterations):
            c = (a + b) / 2
            if abs(func(c)) < tolerance:
                return c
            if func(c) * func(a) < 0:
                b = c
            else:
                a = c
        
        return (a + b) / 2

# Example usage
bond_pricer = BondPricer()
# bond_price = bond_pricer.price_coupon_bond(1000, 0.05, 0.04, 5, 2)
# ytm = bond_pricer.calculate_yield_to_maturity(1025, 1000, 0.05, 5, 2)
```

### Duration and Convexity Analysis
```python
class BondRiskMeasures:
    def __init__(self):
        pass
    
    def calculate_macaulay_duration(self, cash_flows, discount_rates, time_periods=None):
        """Calculate Macaulay duration"""
        if time_periods is None:
            time_periods = np.arange(1, len(cash_flows) + 1)
        
        # Present value of cash flows
        pv_cash_flows = cash_flows / ((1 + discount_rates) ** time_periods)
        bond_price = np.sum(pv_cash_flows)
        
        # Weighted average time
        weighted_times = time_periods * pv_cash_flows
        macaulay_duration = np.sum(weighted_times) / bond_price
        
        return macaulay_duration
    
    def calculate_modified_duration(self, macaulay_duration, yield_to_maturity, frequency=2):
        """Calculate modified duration"""
        periodic_ytm = yield_to_maturity / frequency
        modified_duration = macaulay_duration / (1 + periodic_ytm)
        return modified_duration
    
    def calculate_convexity(self, cash_flows, discount_rates, time_periods=None):
        """Calculate convexity"""
        if time_periods is None:
            time_periods = np.arange(1, len(cash_flows) + 1)
        
        # Present value of cash flows
        pv_cash_flows = cash_flows / ((1 + discount_rates) ** time_periods)
        bond_price = np.sum(pv_cash_flows)
        
        # Convexity calculation
        convexity_terms = (time_periods * (time_periods + 1) * pv_cash_flows) / ((1 + discount_rates) ** 2)
        convexity = np.sum(convexity_terms) / bond_price
        
        return convexity
    
    def calculate_dv01(self, modified_duration, bond_price):
        """Calculate DV01 (dollar value of 01 basis point)"""
        dv01 = modified_duration * bond_price * 0.0001
        return dv01
    
    def price_change_approximation(self, modified_duration, convexity, bond_price, yield_change):
        """Approximate price change using duration and convexity"""
        duration_effect = -modified_duration * bond_price * yield_change
        convexity_effect = 0.5 * convexity * bond_price * (yield_change ** 2)
        total_change = duration_effect + convexity_effect
        
        return {
            'duration_effect': duration_effect,
            'convexity_effect': convexity_effect,
            'total_change': total_change,
            'approximate_new_price': bond_price + total_change
        }

# Example usage
risk_measures = BondRiskMeasures()
# duration = risk_measures.calculate_macaulay_duration(cash_flows, discount_rates)
# price_impact = risk_measures.price_change_approximation(duration, convexity, price, 0.0025)
```

## Yield Curve Modeling

### Bootstrapping Yield Curve
```python
class YieldCurveConstructor:
    def __init__(self):
        self.zero_rates = {}
        
    def bootstrap_yield_curve(self, bond_data):
        """Construct zero-coupon yield curve from bond prices"""
        # Sort bonds by maturity
        sorted_bonds = sorted(bond_data, key=lambda x: x['maturity'])
        
        zero_rates = {}
        
        for bond in sorted_bonds:
            maturity = bond['maturity']
            price = bond['price']
            coupon = bond['coupon']
            face_value = bond['face_value']
            
            # Calculate spot rate
            spot_rate = self.calculate_spot_rate(
                price, coupon, face_value, maturity, zero_rates
            )
            zero_rates[maturity] = spot_rate
            
        self.zero_rates = zero_rates
        return zero_rates
    
    def calculate_spot_rate(self, price, coupon, face_value, maturity, known_rates):
        """Calculate spot rate for specific maturity"""
        # For zero-coupon bonds
        if coupon == 0:
            spot_rate = (face_value / price) ** (1/maturity) - 1
            return spot_rate
        
        # For coupon bonds, need to strip out known forward rates
        periods = int(maturity * 2)  # Semi-annual
        coupon_payment = face_value * coupon / 2
        
        # Calculate present value of known cash flows
        known_pv = 0
        for i in range(1, periods):
            time = i / 2
            if time in known_rates:
                rate = known_rates[time]
                known_pv += coupon_payment / ((1 + rate/2) ** i)
        
        # Solve for final spot rate
        remaining_value = price - known_pv - coupon_payment
        final_spot = (face_value + coupon_payment) / remaining_value
        spot_rate = (final_spot ** (2/maturity)) - 1
        
        return spot_rate
    
    def interpolate_rates(self, target_maturities):
        """Interpolate rates for intermediate maturities"""
        if not self.zero_rates:
            raise ValueError("Yield curve not constructed yet")
        
        maturities = list(self.zero_rates.keys())
        rates = list(self.zero_rates.values())
        
        interpolated_rates = {}
        for target in target_maturities:
            if target in self.zero_rates:
                interpolated_rates[target] = self.zero_rates[target]
            else:
                # Linear interpolation
                rate = np.interp(target, maturities, rates)
                interpolated_rates[target] = rate
                
        return interpolated_rates

# Example usage
curve_constructor = YieldCurveConstructor()
# yield_curve = curve_constructor.bootstrap_yield_curve(bond_market_data)
```

### Nelson-Siegel Model
```python
class NelsonSiegelModel:
    def __init__(self):
        self.parameters = None
    
    def fit_nelson_siegel(self, maturities, yields):
        """Fit Nelson-Siegel model to yield curve data"""
        from scipy.optimize import minimize
        
        def objective(params):
            beta0, beta1, beta2, tau = params
            fitted_yields = self.nelson_siegel_formula(maturities, beta0, beta1, beta2, tau)
            sse = np.sum((yields - fitted_yields) ** 2)
            return sse
        
        # Initial parameter guesses
        initial_params = [0.03, -0.02, 0.01, 2.0]
        
        # Parameter bounds
        bounds = [(0, 0.2), (-0.1, 0.1), (-0.1, 0.1), (0.1, 10)]
        
        # Optimize
        result = minimize(objective, initial_params, bounds=bounds, method='L-BFGS-B')
        
        self.parameters = result.x
        return self.parameters
    
    def nelson_siegel_formula(self, maturities, beta0, beta1, beta2, tau):
        """Nelson-Siegel yield formula"""
        maturities = np.array(maturities)
        tau = max(tau, 0.001)  # Avoid division by zero
        
        factor = (1 - np.exp(-maturities/tau)) / (maturities/tau)
        factor2 = factor - np.exp(-maturities/tau)
        
        yields = beta0 + beta1 * factor + beta2 * factor2
        return yields
    
    def get_yield(self, maturity):
        """Get yield for specific maturity"""
        if self.parameters is None:
            raise ValueError("Model not fitted yet")
        
        beta0, beta1, beta2, tau = self.parameters
        return self.nelson_siegel_formula([maturity], beta0, beta1, beta2, tau)[0]
    
    def get_forward_rate(self, start_maturity, end_maturity):
        """Calculate forward rate between two maturities"""
        if self.parameters is None:
            raise ValueError("Model not fitted yet")
        
        # Integral approach for forward rates
        def integral_rate(maturity):
            return self.get_yield(maturity) * maturity
        
        forward_rate = (integral_rate(end_maturity) - integral_rate(start_maturity)) / (end_maturity - start_maturity)
        return forward_rate

# Example usage
ns_model = NelsonSiegelModel()
# ns_params = ns_model.fit_nelson_siegel(observed_maturities, observed_yields)
# yield_5y = ns_model.get_yield(5)
```

## Credit Risk Analysis

### Credit Spreads and Default Probability
```python
class CreditRiskAnalyzer:
    def __init__(self):
        pass
    
    def calculate_credit_spread(self, corporate_bond_yield, risk_free_rate):
        """Calculate credit spread"""
        return corporate_bond_yield - risk_free_rate
    
    def estimate_default_probability(self, credit_spread, recovery_rate=0.4):
        """Estimate default probability using credit spread"""
        # Simplified approach: PD = Spread / (1 - Recovery Rate)
        default_probability = credit_spread / (1 - recovery_rate)
        return default_probability
    
    def calculate_expected_loss(self, exposure, default_probability, recovery_rate):
        """Calculate expected credit loss"""
        loss_given_default = 1 - recovery_rate
        expected_loss = exposure * default_probability * loss_given_default
        return expected_loss
    
    def credit_value_at_risk(self, portfolio_exposures, correlations, confidence_level=0.99):
        """Calculate credit VaR for portfolio"""
        # Simplified portfolio credit risk model
        total_exposure = sum(portfolio_exposures.values())
        
        # Assume correlated defaults
        avg_pd = np.mean(list(self.calculate_portfolio_pds(portfolio_exposures).values()))
        credit_var = total_exposure * avg_pd * (1 - 0.4)  # Assuming 40% recovery
        
        return credit_var
    
    def calculate_portfolio_pds(self, exposures):
        """Calculate individual default probabilities"""
        # This would typically use credit ratings, financial ratios, etc.
        pds = {}
        for issuer, exposure in exposures.items():
            # Simplified rating-based PDs
            rating_pds = {
                'AAA': 0.0001, 'AA': 0.0005, 'A': 0.001, 'BBB': 0.005,
                'BB': 0.02, 'B': 0.05, 'CCC': 0.15
            }
            # In practice, would map issuer to actual rating
            pds[issuer] = rating_pds.get('BBB', 0.005)  # Default to BBB
            
        return pds

# Example usage
credit_analyzer = CreditRiskAnalyzer()
# credit_spread = credit_analyzer.calculate_credit_spread(corp_yield, treasury_yield)
# default_prob = credit_analyzer.estimate_default_probability(credit_spread)
```

## Interest Rate Derivatives

### Bond Futures Pricing
```python
class BondFuturesPricer:
    def __init__(self):
        pass
    
    def price_bond_future(self, cheapest_to_deliver_price, conversion_factor, 
                         risk_free_rate, time_to_expiry):
        """Price bond futures contract"""
        # Fair value = (CTD price / Conversion Factor) * e^(-rt)
        fair_value = (cheapest_to_deliver_price / conversion_factor) * np.exp(-risk_free_rate * time_to_expiry)
        return fair_value
    
    def calculate_basis(self, futures_price, cash_price, conversion_factor):
        """Calculate futures-cash basis"""
        theoretical_futures = cash_price / conversion_factor
        basis = futures_price - theoretical_futures
        return basis
    
    def identify_cheapest_to_deliver(self, deliverable_bonds, futures_price):
        """Identify cheapest-to-deliver bond"""
        implied_repo_rates = {}
        
        for bond in deliverable_bonds:
            # Calculate implied repo rate
            invoice_price = futures_price * bond['conversion_factor']
            implied_repo = (invoice_price / bond['clean_price'] - 1) * (360 / bond['days_to_delivery'])
            implied_repo_rates[bond['cusip']] = implied_repo
        
        # Cheapest to deliver has lowest implied repo rate
        cheapest_cusip = min(implied_repo_rates, key=implied_repo_rates.get)
        return cheapest_cusip, implied_repo_rates[cheapest_cusip]

# Example usage
futures_pricer = BondFuturesPricer()
# future_price = futures_pricer.price_bond_future(ctd_price, conv_factor, rf_rate, time_to_expiry)
```

### Interest Rate Swaps
```python
class InterestRateSwap:
    def __init__(self):
        self.fixed_leg = []
        self.floating_leg = []
    
    def price_interest_rate_swap(self, notional, fixed_rate, floating_curve, 
                               start_date, end_date, payment_frequency=0.5):
        """Price interest rate swap"""
        # Calculate fixed leg present value
        fixed_pv = self.calculate_fixed_leg_pv(notional, fixed_rate, end_date, start_date, payment_frequency)
        
        # Calculate floating leg present value
        floating_pv = self.calculate_floating_leg_pv(notional, floating_curve, start_date, end_date, payment_frequency)
        
        # Swap value = Floating PV - Fixed PV
        swap_value = floating_pv - fixed_pv
        return swap_value
    
    def calculate_fixed_leg_pv(self, notional, fixed_rate, end_date, start_date, frequency):
        """Calculate present value of fixed leg payments"""
        # Simplified calculation
        payment_dates = self.generate_payment_dates(start_date, end_date, frequency)
        time_periods = [(date - start_date).days / 365.25 for date in payment_dates]
        
        pv = 0
        for t in time_periods:
            payment = notional * fixed_rate * frequency
            discount_factor = np.exp(-0.03 * t)  # Simplified discounting
            pv += payment * discount_factor
            
        return pv
    
    def calculate_floating_leg_pv(self, notional, floating_curve, start_date, end_date, frequency):
        """Calculate present value of floating leg payments"""
        # Assume forward rates from yield curve
        payment_dates = self.generate_payment_dates(start_date, end_date, frequency)
        time_periods = [(date - start_date).days / 365.25 for date in payment_dates]
        
        pv = 0
        prev_time = 0
        for t in time_periods:
            # Get forward rate for period
            forward_rate = floating_curve.get_forward_rate(prev_time, t)
            payment = notional * forward_rate * (t - prev_time)
            discount_factor = np.exp(-0.03 * t)
            pv += payment * discount_factor
            prev_time = t
            
        return pv
    
    def generate_payment_dates(self, start_date, end_date, frequency):
        """Generate payment dates"""
        # Simplified - would use actual business day conventions
        days = (end_date - start_date).days
        periods = int(days * frequency / 365.25)
        dates = [start_date + pd.Timedelta(days=int(i * 365.25 / frequency)) for i in range(1, periods + 1)]
        return dates

# Example usage
irs_pricer = InterestRateSwap()
# swap_value = irs_pricer.price_interest_rate_swap(notional, fixed_rate, yield_curve, start_dt, end_dt)
```

## Portfolio Management Applications

### Fixed Income Portfolio Optimization
```python
class FixedIncomeOptimizer:
    def __init__(self):
        pass
    
    def optimize_bond_portfolio(self, bonds, target_duration, target_yield, constraints=None):
        """Optimize bond portfolio for target duration and yield"""
        from scipy.optimize import minimize
        
        n_bonds = len(bonds)
        
        def objective(weights):
            # Minimize tracking error from target metrics
            portfolio_duration = sum(weights[i] * bonds[i]['duration'] for i in range(n_bonds))
            portfolio_yield = sum(weights[i] * bonds[i]['yield'] for i in range(n_bonds))
            
            duration_error = (portfolio_duration - target_duration) ** 2
            yield_error = (portfolio_yield - target_yield) ** 2
            
            return duration_error + yield_error
        
        # Constraints
        cons = [{'type': 'eq', 'fun': lambda w: np.sum(w) - 1}]  # Weights sum to 1
        
        if constraints:
            cons.extend(constraints)
        
        # Bounds (weights between 0 and 1)
        bounds = tuple((0, 1) for _ in range(n_bonds))
        
        # Initial guess (equal weights)
        initial_weights = np.array([1/n_bonds] * n_bonds)
        
        # Optimize
        result = minimize(objective, initial_weights, method='SLSQP', bounds=bounds, constraints=cons)
        
        return result.x
    
    def calculate_portfolio_metrics(self, weights, bonds):
        """Calculate portfolio-level metrics"""
        portfolio_duration = sum(weights[i] * bonds[i]['duration'] for i in range(len(bonds)))
        portfolio_yield = sum(weights[i] * bonds[i]['yield'] for i in range(len(bonds)))
        portfolio_convexity = sum(weights[i] * bonds[i]['convexity'] for i in range(len(bonds)))
        
        # Credit metrics
        portfolio_avg_rating = sum(weights[i] * bonds[i]['rating_numeric'] for i in range(len(bonds)))
        portfolio_credit_spread = sum(weights[i] * bonds[i]['credit_spread'] for i in range(len(bonds)))
        
        return {
            'duration': portfolio_duration,
            'yield': portfolio_yield,
            'convexity': portfolio_convexity,
            'avg_credit_rating': portfolio_avg_rating,
            'weighted_avg_credit_spread': portfolio_credit_spread
        }

# Example usage
optimizer = FixedIncomeOptimizer()
# optimal_weights = optimizer.optimize_bond_portfolio(bond_list, target_duration=5, target_yield=0.04)
```

This comprehensive fixed income analysis documentation covers bond pricing fundamentals, risk measures, yield curve modeling, credit risk analysis, interest rate derivatives, and portfolio optimization techniques essential for fixed income quantitative analysis.