Sure, here's the complete code:

```python
import pandas as pd
import numpy as np
from scipy.optimize import minimize

# Load your data (example with CSV)
data = pd.read_csv('your_stock_data.csv', parse_dates=['Date'], index_col='Date')

# Compute Stochastic Oscillator
def stochastic_oscillator(data, n=14):
    L14 = data['Low'].rolling(window=n).min()
    H14 = data['High'].rolling(window=n).max()
    K = 100 * ((data['Close'] - L14) / (H14 - L14))
    D = K.rolling(window=3).mean()
    return K, D

# Optimize parameters (n and smoothing)
def optimize_stochastic(data):
    def objective(params):
        n, smoothing = int(params[0]), int(params[1])
        K, D = stochastic_oscillator(data, n)
        D = K.rolling(window=smoothing).mean()
        
        # Example: minimize the variance of %D as an objective
        return D.var()

    initial_guess = [14, 3]
    bounds = [(1, 50), (1, 50)]  # realistic bounds for n and smoothing
    
    result = minimize(objective, initial_guess, bounds=bounds, method='L-BFGS-B')
    optimized_params = result.x
    return optimized_params

# Run optimization
optimized_params = optimize_stochastic(data)
print(f"Optimized parameters: n={int(optimized_params[0])}, smoothing={int(optimized_params[1])}")

# Calculate Stochastic Oscillator with optimized parameters
n_optimized = int(optimized_params[0])
smoothing_optimized = int(optimized_params[1])
K_optimized, D_optimized = stochastic_oscillator(data, n_optimized)
D_optimized = K_optimized.rolling(window=smoothing_optimized).mean()

# Output first few values as example
print("First few %K values with optimized parameters:")
print(K_optimized.head())
print("First few %D values with optimized parameters:")
print(D_optimized.head())
```
