Here is the complete code to calculate and plot the Supertrend indicator using pandas-ta:

```python
import pandas as pd
import pandas_ta as ta
import matplotlib.pyplot as plt

# Sample data
data = {
    'date': pd.date_range(start='2020-01-01', periods=100, freq='D'),
    'close': pd.Series(range(100)) + (pd.Series(range(100)) * 0.1).cumsum(),
    'high': pd.Series(range(100)) + (pd.Series(range(100)) * 0.15).cumsum(),
    'low': pd.Series(range(100)) + (pd.Series(range(100)) * 0.05).cumsum()
}

df = pd.DataFrame(data)
df.set_index('date', inplace=True)

# Calculate the Supertrend
supertrend = ta.supertrend(high=df['high'], low=df['low'], close=df['close'], length=10, multiplier=3.0)

# The result is a DataFrame with 'SUPERT_10_3.0', 'SUPERTd_10_3.0', and 'SUPERTl_10_3.0' columns
df = pd.concat([df, supertrend], axis=1)

print(df.head(20))

# Plotting
plt.figure(figsize=(14, 7))

# Plot the closing prices
plt.plot(df.index, df['close'], label='Close Price', color='black')

# Plot the Supertrend
plt.plot(df.index, df['SUPERT_10_3.0'], label='Supertrend', color='green')

# Mark the areas where the trend is up or down
plt.fill_between(df.index, df['close'], df['SUPERT_10_3.0'], where=df['SUPERTd_10_3.0'] == 1, color='lightgreen', alpha=0.5)
plt.fill_between(df.index, df['close'], df['SUPERT_10_3.0'], where=df['SUPERTd_10_3.0'] == -1, color='lightcoral', alpha=0.5)

plt.title('Supertrend Indicator')
plt.xlabel('Date')
plt.ylabel('Price')
plt.legend()
plt.show()
```

### Explanation

1. **Import Libraries:**
   - Import `pandas` for data manipulation.
   - Import `pandas_ta` for technical analysis indicators.
   - Import `matplotlib.pyplot` for plotting.

2. **Sample Data:**
   - Create a dictionary with dates, close, high, and low prices.
   - Convert the dictionary to a pandas DataFrame and set the date column as the index
