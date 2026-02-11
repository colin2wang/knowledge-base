# Time Series Analysis

Advanced time series analysis techniques for financial market modeling, forecasting, and quantitative finance applications.

## Overview

Time series analysis is fundamental to quantitative finance, enabling the modeling of temporal dependencies, volatility clustering, and the identification of predictable patterns in financial data.

## Classical Time Series Models

### ARIMA (AutoRegressive Integrated Moving Average)
ARIMA models capture linear temporal dependencies in time series data.

```python
import numpy as np
import pandas as pd
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.stattools import adfuller
import warnings
warnings.filterwarnings('ignore')

class ARIMAModel:
    def __init__(self, order=(1, 1, 1)):
        self.order = order
        self.model = None
        self.fitted_model = None
    
    def check_stationarity(self, time_series, significance_level=0.05):
        """Check if time series is stationary using Augmented Dickey-Fuller test"""
        result = adfuller(time_series.dropna())
        
        adf_statistic = result[0]
        p_value = result[1]
        critical_values = result[4]
        
        is_stationary = p_value < significance_level
        
        return {
            'adf_statistic': adf_statistic,
            'p_value': p_value,
            'critical_values': critical_values,
            'is_stationary': is_stationary,
            'recommended_differencing': 0 if is_stationary else 1
        }
    
    def difference_series(self, time_series, order=1):
        """Apply differencing to make series stationary"""
        return time_series.diff(periods=order).dropna()
    
    def fit(self, time_series, exog_vars=None):
        """Fit ARIMA model to time series data"""
        # Check stationarity
        stationarity_check = self.check_stationarity(time_series)
        
        # Apply differencing if needed
        if not stationarity_check['is_stationary']:
            diff_order = stationarity_check['recommended_differencing']
            stationary_series = self.difference_series(time_series, diff_order)
            self.order = (self.order[0], diff_order, self.order[2])
        else:
            stationary_series = time_series
        
        # Fit ARIMA model
        self.model = ARIMA(stationary_series, order=self.order, exog=exog_vars)
        self.fitted_model = self.model.fit()
        
        return self.fitted_model
    
    def forecast(self, steps=10, exog_future=None):
        """Generate forecasts"""
        if self.fitted_model is None:
            raise ValueError("Model must be fitted before forecasting")
            
        forecast_result = self.fitted_model.forecast(steps=steps, exog=exog_future)
        return forecast_result
    
    def get_model_summary(self):
        """Get detailed model summary"""
        if self.fitted_model is None:
            return "Model not fitted yet"
        return self.fitted_model.summary()

# Example usage
arima_model = ARIMAModel(order=(1, 1, 1))
# Assuming you have price data
# fitted_model = arima_model.fit(log_returns)
# forecasts = arima_model.forecast(steps=30)
```

### GARCH (Generalized AutoRegressive Conditional Heteroskedasticity)
GARCH models capture volatility clustering and time-varying volatility in financial time series.

```python
from arch import arch_model
import numpy as np

class GARCHModel:
    def __init__(self, p=1, q=1, dist='normal'):
        self.p = p
        self.q = q
        self.dist = dist
        self.model = None
        self.fitted_model = None
    
    def fit(self, returns):
        """Fit GARCH model to return series"""
        # Ensure returns are stationary (usually they are)
        self.model = arch_model(returns, p=self.p, q=self.q, dist=self.dist)
        self.fitted_model = self.model.fit(update_freq=5, disp='off')
        return self.fitted_model
    
    def forecast_volatility(self, horizon=10):
        """Forecast conditional volatility"""
        if self.fitted_model is None:
            raise ValueError("Model must be fitted before forecasting")
            
        forecast = self.fitted_model.forecast(horizon=horizon)
        return np.sqrt(forecast.variance.iloc[-1])  # Convert variance to volatility
    
    def simulate_paths(self, n_simulations=1000, horizon=252):
        """Simulate future price paths"""
        if self.fitted_model is None:
            raise ValueError("Model must be fitted before simulation")
            
        # Get standardized residuals and conditional volatility
        std_resids = self.fitted_model.std_resid
        cond_vol = self.fitted_model.conditional_volatility
        
        # Simulate
        simulations = []
        last_vol = cond_vol.iloc[-1]
        last_return = std_resids.iloc[-1] * last_vol
        
        for _ in range(n_simulations):
            path_vol = [last_vol]
            path_return = [last_return]
            
            for _ in range(horizon):
                # Forecast next volatility
                next_vol = np.sqrt(
                    self.fitted_model.params['omega'] +
                    self.fitted_model.params['alpha[1]'] * path_return[-1]**2 +
                    self.fitted_model.params['beta[1]'] * path_vol[-1]**2
                )
                
                # Generate next return
                if self.dist == 'normal':
                    next_return = np.random.normal(0, next_vol)
                elif self.dist == 't':
                    df = self.fitted_model.params['nu']
                    next_return = np.random.standard_t(df) * next_vol
                
                path_vol.append(next_vol)
                path_return.append(next_return)
            
            simulations.append({
                'volatility': path_vol[1:],  # Exclude initial value
                'returns': path_return[1:]   # Exclude initial value
            })
        
        return simulations
    
    def value_at_risk(self, confidence_level=0.05, horizon=1):
        """Calculate Value at Risk using GARCH model"""
        simulations = self.simulate_paths(n_simulations=10000, horizon=horizon)
        returns = [sim['returns'][0] for sim in simulations]  # 1-day returns
        var = np.percentile(returns, confidence_level * 100)
        return abs(var)

# Example usage
garch_model = GARCHModel(p=1, q=1)
# Assuming you have return data
# fitted_garch = garch_model.fit(daily_returns)
# volatility_forecast = garch_model.forecast_volatility(horizon=30)
# var_95 = garch_model.value_at_risk(confidence_level=0.05)
```

## Advanced Time Series Techniques

### State Space Models
State space models provide flexible framework for time series decomposition and filtering.

```python
import pandas as pd
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.tsa.seasonal import seasonal_decompose

class StateSpaceAnalyzer:
    def __init__(self):
        pass
    
    def decompose_series(self, time_series, model='additive', period=None):
        """Decompose time series into trend, seasonal, and residual components"""
        if period is None:
            # Auto-detect seasonality period
            period = self.detect_seasonality(time_series)
        
        decomposition = seasonal_decompose(time_series, model=model, period=period)
        
        return {
            'trend': decomposition.trend,
            'seasonal': decomposition.seasonal,
            'residual': decomposition.resid,
            'observed': decomposition.observed
        }
    
    def detect_seasonality(self, time_series, max_lag=100):
        """Detect seasonality period using autocorrelation"""
        from statsmodels.tsa.stattools import acf
        
        autocorr = acf(time_series.dropna(), nlags=max_lag)
        
        # Find peaks in autocorrelation
        peaks = []
        for i in range(1, len(autocorr) - 1):
            if autocorr[i] > autocorr[i-1] and autocorr[i] > autocorr[i+1]:
                peaks.append((i, autocorr[i]))
        
        # Return the lag with highest autocorrelation (excluding lag 0)
        if peaks:
            return max(peaks, key=lambda x: x[1])[0]
        else:
            return 12  # Default to monthly seasonality
    
    def fit_sarimax(self, endog, exog=None, order=(1,1,1), seasonal_order=(1,1,1,12)):
        """Fit SARIMAX model for seasonal time series"""
        model = SARIMAX(endog, exog=exog, order=order, seasonal_order=seasonal_order)
        fitted_model = model.fit(disp=False)
        return fitted_model
    
    def kalman_filter_smoothing(self, observations, transition_matrix, observation_matrix, 
                              process_noise, observation_noise, initial_state, initial_covariance):
        """Implement Kalman filter for state estimation"""
        n_states = len(initial_state)
        n_obs = len(observations)
        
        # Initialize arrays
        states = np.zeros((n_obs, n_states))
        covariances = np.zeros((n_obs, n_states, n_states))
        predictions = np.zeros(n_obs)
        
        # Initial state
        states[0] = initial_state
        covariances[0] = initial_covariance
        
        for t in range(1, n_obs):
            # Prediction step
            predicted_state = transition_matrix @ states[t-1]
            predicted_cov = transition_matrix @ covariances[t-1] @ transition_matrix.T + process_noise
            
            # Update step
            innovation = observations[t] - observation_matrix @ predicted_state
            innovation_cov = observation_matrix @ predicted_cov @ observation_matrix.T + observation_noise
            kalman_gain = predicted_cov @ observation_matrix.T @ np.linalg.inv(innovation_cov)
            
            states[t] = predicted_state + kalman_gain @ innovation
            covariances[t] = (np.eye(n_states) - kalman_gain @ observation_matrix) @ predicted_cov
            
            # Predicted observation
            predictions[t] = observation_matrix @ states[t]
        
        return states, predictions, covariances

# Example usage
ssa = StateSpaceAnalyzer()
# decomposition = ssa.decompose_series(price_series)
```

### Machine Learning Approaches

### LSTM Neural Networks for Time Series
```python
import numpy as np
import pandas as pd
from sklearn.preprocessing import MinMaxScaler
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense, Dropout
from tensorflow.keras.optimizers import Adam

class LSTMTimeSeries:
    def __init__(self, sequence_length=60, lstm_units=50, dropout=0.2):
        self.sequence_length = sequence_length
        self.lstm_units = lstm_units
        self.dropout = dropout
        self.model = None
        self.scaler = MinMaxScaler(feature_range=(0, 1))
        
    def prepare_sequences(self, data):
        """Prepare sequences for LSTM training"""
        scaled_data = self.scaler.fit_transform(data.reshape(-1, 1))
        
        X, y = [], []
        for i in range(self.sequence_length, len(scaled_data)):
            X.append(scaled_data[i-self.sequence_length:i, 0])
            y.append(scaled_data[i, 0])
            
        X, y = np.array(X), np.array(y)
        X = np.reshape(X, (X.shape[0], X.shape[1], 1))
        
        return X, y, scaled_data
    
    def build_model(self, input_shape):
        """Build LSTM model architecture"""
        self.model = Sequential([
            LSTM(units=self.lstm_units, return_sequences=True, input_shape=input_shape),
            Dropout(self.dropout),
            LSTM(units=self.lstm_units, return_sequences=True),
            Dropout(self.dropout),
            LSTM(units=self.lstm_units),
            Dropout(self.dropout),
            Dense(units=1)
        ])
        
        self.model.compile(optimizer=Adam(learning_rate=0.001), loss='mean_squared_error')
        return self.model
    
    def train(self, data, epochs=50, batch_size=32, validation_split=0.1):
        """Train LSTM model"""
        X, y, scaled_data = self.prepare_sequences(data)
        
        if self.model is None:
            self.build_model((X.shape[1], 1))
        
        history = self.model.fit(
            X, y,
            epochs=epochs,
            batch_size=batch_size,
            validation_split=validation_split,
            verbose=1
        )
        
        return history
    
    def predict(self, data, steps=30):
        """Make multi-step predictions"""
        if self.model is None:
            raise ValueError("Model must be trained before prediction")
        
        # Prepare last sequence
        scaled_data = self.scaler.transform(data.reshape(-1, 1))
        last_sequence = scaled_data[-self.sequence_length:].reshape(1, self.sequence_length, 1)
        
        predictions = []
        current_sequence = last_sequence.copy()
        
        for _ in range(steps):
            # Predict next value
            next_pred = self.model.predict(current_sequence, verbose=0)
            predictions.append(next_pred[0, 0])
            
            # Update sequence
            current_sequence = np.roll(current_sequence, -1, axis=1)
            current_sequence[0, -1, 0] = next_pred[0, 0]
        
        # Inverse transform predictions
        predictions_array = np.array(predictions).reshape(-1, 1)
        predictions_original = self.scaler.inverse_transform(predictions_array)
        
        return predictions_original.flatten()

# Example usage
# lstm_model = LSTMTimeSeries(sequence_length=60)
# lstm_model.train(training_data, epochs=100)
# future_predictions = lstm_model.predict(historical_data, steps=30)
```

## Volatility Modeling

### Stochastic Volatility Models
```python
class StochasticVolatilityModel:
    def __init__(self, mu=0, phi=0.98, sigma=0.2):
        self.mu = mu      # Long-term volatility mean
        self.phi = phi    # Persistence parameter
        self.sigma = sigma # Volatility of volatility
        self.volatility_process = []
        
    def simulate_sv_process(self, n_periods, initial_vol=0.2):
        """Simulate stochastic volatility process"""
        log_vol = np.zeros(n_periods)
        log_vol[0] = np.log(initial_vol**2)
        
        shocks = np.random.normal(0, self.sigma, n_periods)
        
        for t in range(1, n_periods):
            log_vol[t] = self.mu + self.phi * (log_vol[t-1] - self.mu) + shocks[t]
        
        volatility = np.exp(log_vol/2)
        self.volatility_process = volatility
        
        return volatility
    
    def generate_price_paths(self, n_paths, n_periods, initial_price=100):
        """Generate price paths with stochastic volatility"""
        price_paths = []
        
        for _ in range(n_paths):
            # Simulate volatility path
            vol_path = self.simulate_sv_process(n_periods)
            
            # Generate returns with time-varying volatility
            returns = np.random.normal(0, vol_path, n_periods)
            
            # Convert to price path
            price_path = [initial_price]
            for ret in returns[1:]:
                price_path.append(price_path[-1] * np.exp(ret))
            
            price_paths.append(price_path)
        
        return np.array(price_paths)
    
    def estimate_parameters(self, observed_returns):
        """Estimate SV model parameters using quasi-maximum likelihood"""
        # Simplified estimation using method of moments
        log_returns_sq = np.log(observed_returns**2)
        
        # Estimate persistence (phi) using autocorrelation
        autocorr = np.corrcoef(log_returns_sq[:-1], log_returns_sq[1:])[0, 1]
        phi_hat = autocorr
        
        # Estimate long-term mean
        mu_hat = np.mean(log_returns_sq)
        
        # Estimate volatility of volatility
        residuals = log_returns_sq[1:] - mu_hat - phi_hat * (log_returns_sq[:-1] - mu_hat)
        sigma_hat = np.std(residuals)
        
        return {
            'mu': mu_hat,
            'phi': phi_hat,
            'sigma': sigma_hat
        }

# Example usage
sv_model = StochasticVolatilityModel()
# simulated_vols = sv_model.simulate_sv_process(252)
# price_paths = sv_model.generate_price_paths(1000, 252)
```

## Time Series Feature Engineering

### Technical Features Extraction
```python
class TimeSeriesFeatures:
    def __init__(self):
        self.feature_functions = {
            'returns': self.calculate_returns,
            'volatility': self.calculate_realized_volatility,
            'autocorrelation': self.calculate_autocorrelation,
            'hurst_exponent': self.calculate_hurst_exponent,
            'detrended_fluctuation': self.calculate_dfa
        }
    
    def calculate_returns(self, prices, period=1):
        """Calculate logarithmic returns"""
        return np.log(prices / prices.shift(period)).dropna()
    
    def calculate_realized_volatility(self, returns, window=22):
        """Calculate realized volatility (standard deviation of returns)"""
        return returns.rolling(window=window).std() * np.sqrt(252)  # Annualized
    
    def calculate_autocorrelation(self, returns, lags=range(1, 11)):
        """Calculate autocorrelation at different lags"""
        from statsmodels.tsa.stattools import acf
        autocorrelations = {}
        for lag in lags:
            if len(returns) > lag:
                autocorr = acf(returns.dropna(), nlags=lag)[-1]
                autocorrelations[f'autocorr_{lag}'] = autocorr
        return autocorrelations
    
    def calculate_hurst_exponent(self, ts, max_lag=20):
        """Calculate Hurst exponent for long-range dependence"""
        lags = range(2, max_lag)
        tau = [np.std(np.subtract(ts[lag:], ts[:-lag])) for lag in lags]
        
        # Remove zeros
        lags = [lag for lag, t in zip(lags, tau) if t > 0]
        tau = [t for t in tau if t > 0]
        
        if len(lags) < 2:
            return 0.5  # Random walk
            
        # Calculate Hurst exponent
        poly = np.polyfit(np.log(lags), np.log(tau), 1)
        hurst = poly[0] / 2.0
        return hurst
    
    def calculate_dfa(self, ts, window_sizes=None):
        """Calculate Detrended Fluctuation Analysis"""
        if window_sizes is None:
            window_sizes = [10, 20, 40, 80, 160]
        
        fluctuations = []
        
        for window_size in window_sizes:
            if len(ts) < window_size * 2:
                continue
                
            # Integrate time series
            y = np.cumsum(ts - np.mean(ts))
            
            # Local trend removal and fluctuation calculation
            fluctuation = 0
            segments = len(y) // window_size
            
            for i in range(segments):
                start_idx = i * window_size
                end_idx = start_idx + window_size
                segment = y[start_idx:end_idx]
                
                # Fit local trend
                x_local = np.arange(window_size)
                coeffs = np.polyfit(x_local, segment, 1)
                trend = np.polyval(coeffs, x_local)
                
                # Calculate fluctuation
                fluctuation += np.mean((segment - trend) ** 2)
            
            fluctuations.append(np.sqrt(fluctuation / segments))
        
        if len(fluctuations) < 2:
            return 0.5
            
        # Fit power law
        poly = np.polyfit(np.log(window_sizes[:len(fluctuations)]), 
                         np.log(fluctuations), 1)
        return poly[0]  # DFA exponent
    
    def extract_all_features(self, price_series):
        """Extract comprehensive set of time series features"""
        returns = self.calculate_returns(price_series)
        
        features = {
            'mean_return': np.mean(returns),
            'volatility': np.std(returns) * np.sqrt(252),
            'skewness': self.calculate_skewness(returns),
            'kurtosis': self.calculate_kurtosis(returns),
            'hurst_exponent': self.calculate_hurst_exponent(returns),
            'dfa_exponent': self.calculate_dfa(returns),
        }
        
        # Add autocorrelations
        auto_corr_features = self.calculate_autocorrelation(returns)
        features.update(auto_corr_features)
        
        # Add volatility clustering measures
        volatility = self.calculate_realized_volatility(returns)
        features['volatility_persistence'] = np.corrcoef(volatility[:-1], volatility[1:])[0, 1]
        
        return features
    
    def calculate_skewness(self, data):
        """Calculate skewness"""
        return np.mean(((data - np.mean(data)) / np.std(data)) ** 3)
    
    def calculate_kurtosis(self, data):
        """Calculate kurtosis"""
        return np.mean(((data - np.mean(data)) / np.std(data)) ** 4) - 3

# Example usage
feature_extractor = TimeSeriesFeatures()
# features = feature_extractor.extract_all_features(price_data)
```

This comprehensive time series analysis documentation covers classical econometric models, advanced machine learning approaches, volatility modeling, and feature engineering techniques essential for quantitative finance applications.