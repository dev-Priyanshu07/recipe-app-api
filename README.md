The Supertrend indicator is a popular technical analysis tool used in trading to determine the trend direction. To create a class for the Supertrend function in Python, we need to calculate the average true range (ATR) and then use it to compute the Supertrend.

Here's a Python class that calculates the Supertrend indicator:

```python
import pandas as pd
import numpy as np

class Supertrend:
    def __init__(self, period, multiplier):
        self.period = period
        self.multiplier = multiplier
    
    def compute_atr(self, data):
        high_low = data['High'] - data['Low']
        high_close = np.abs(data['High'] - data['Close'].shift())
        low_close = np.abs(data['Low'] - data['Close'].shift())
        
        ranges = pd.concat([high_low, high_close, low_close], axis=1)
        true_range = np.max(ranges, axis=1)
        atr = true_range.rolling(self.period).mean()
        
        return atr
    
    def compute_supertrend(self, data):
        atr = self.compute_atr(data)
        hl2 = (data['High'] + data['Low']) / 2
        upperband = hl2 + (self.multiplier * atr)
        lowerband = hl2 - (self.multiplier * atr)
        
        supertrend = pd.Series(index=data.index)
        in_uptrend = True
        
        for current in range(1, len(data.index)):
            if data['Close'][current] > upperband[current - 1]:
                in_uptrend = True
            elif data['Close'][current] < lowerband[current - 1]:
                in_uptrend = False

            if in_uptrend:
                supertrend[current] = lowerband[current]
            else:
                supertrend[current] = upperband[current]
        
        return supertrend

# Example usage:
# Create a DataFrame with your data
data = pd.DataFrame({
    'High': [127.01, 127.62, 126.59, 126.57, 126.39],
    'Low': [125.36, 125.78, 125.07, 124.80, 124.90],
    'Close': [125.65, 127.35, 126.41, 125.84, 125.52]
})

# Initialize the Supertrend class
supertrend_indicator = Supertrend(period=3, multiplier=3)

# Compute the Supertrend
data['Supertrend'] = supertrend_indicator.compute_supertrend(data)

# Print the data with the Supertrend column
print(data)
```

### Explanation:

1. **Initialization**:
   ```python
   def __init__(self, period, multiplier):
       self.period = period
       self.multiplier = multiplier
   ```
   The `__init__` method initializes the Supertrend class with a specified period and multiplier.

2. **ATR Calculation**:
   ```python
   def compute_atr(self, data):
       high_low = data['High'] - data['Low']
       high_close = np.abs(data['High'] - data['Close'].shift())
       low_close = np.abs(data['Low'] - data['Close'].shift())
       
       ranges = pd.concat([high_low, high_close, low_close], axis=1)
       true_range = np.max(ranges, axis=1)
       atr = true_range.rolling(self.period).mean()
       
       return atr
   ```
   The `compute_atr` method calculates the Average True Range (ATR) using the high, low, and close prices.

3. **Supertrend Calculation**:
   ```python
   def compute_supertrend(self, data):
       atr = self.compute_atr(data)
       hl2 = (data['High'] + data['Low']) / 2
       upperband = hl2 + (self.multiplier * atr)
       lowerband = hl2 - (self.multiplier * atr)
       
       supertrend = pd.Series(index=data.index)
       in_uptrend = True
       
       for current in range(1, len(data.index)):
           if data['Close'][current] > upperband[current - 1]:
               in_uptrend = True
           elif data['Close'][current] < lowerband[current - 1]:
               in_uptrend = False

           if in_uptrend:
               supertrend[current] = lowerband[current]
           else:
               supertrend[current] = upperband[current]
       
       return supertrend
   ```
   The `compute_supertrend` method calculates the Supertrend indicator based on the ATR and the multiplier. It determines whether the current trend is up or down and sets the Supertrend value accordingly.

### Example Usage:
After defining the class, you can use it to calculate the Supertrend for a given dataset.

This example initializes the Supertrend class with a period of 3 and a multiplier of 3 and computes the Supertrend for a sample dataset.
