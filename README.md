# recipe-app-api

To achieve this, we can update the `update` function to avoid clearing the previous plot. Instead, we'll continuously add the new data to the existing plot. This approach maintains the continuity of the plotted data while updating it every 10 seconds. Here's the updated code:

```python
import pandas as pd
import numpy as np
import pandas_ta as ta
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

# Function to generate new data from the simulator
def get_new_data():
    new_data = {
        'Date': pd.date_range(start='2022-01-01', periods=10, freq='D'),
        'High': np.random.rand(10) * 100 + 100,
        'Low': np.random.rand(10) * 100,
        'Close': np.random.rand(10) * 100 + 50
    }
    return pd.DataFrame(new_data)

# Initialize an empty DataFrame
df = pd.DataFrame(columns=['Date', 'High', 'Low', 'Close'])

# Initialize plot
fig, ax = plt.subplots(figsize=(14, 7))

# Initialize lines
line_close, = ax.plot([], [], label='Close Price', color='blue')
line_supertrend, = ax.plot([], [], label='Supertrend', color='green')

# Function to update the plot
def update(frame):
    global df
    # Get new data
    new_data = get_new_data()
    df = pd.concat([df, new_data]).drop_duplicates().reset_index(drop=True)
    
    # Set the index to Date
    df.set_index('Date', inplace=True)
    
    # Calculate the Supertrend
    supertrend = ta.supertrend(df['High'], df['Low'], df['Close'], length=7, multiplier=3.0)
    
    # Merge the Supertrend results with the original DataFrame
    df = df.join(supertrend)
    
    # Update line data
    line_close.set_data(df.index, df['Close'])
    line_supertrend.set_data(df.index, df['SUPERT_7_3.0'])
    
    # Highlighting uptrends and downtrends
    for i in range(1, len(df)):
        if df['SUPERTd_7_3.0'].iloc[i] == 1:
            ax.plot(df.index[i-1:i+1], df['SUPERT_7_3.0'].iloc[i-1:i+1], color='green')
        else:
            ax.plot(df.index[i-1:i+1], df['SUPERT_7_3.0'].iloc[i-1:i+1], color='red')
    
    ax.set_xlim(df.index.min(), df.index.max())
    ax.set_ylim(min(df['Low'].min(), df['SUPERT_7_3.0'].min()), max(df['High'].max(), df['SUPERT_7_3.0'].max()))
    ax.set_title('Supertrend Indicator')
    ax.set_xlabel('Date')
    ax.set_ylabel('Price')
    ax.legend()

# Create animation
ani = FuncAnimation(fig, update, interval=10000)  # Update every 10 seconds

plt.show()
```

In this updated code:
1. We initialize the plot and create line objects for the close price and Supertrend.
2. The `update` function:
   - Retrieves new data from the simulator.
   - Concatenates it with the existing DataFrame, avoiding duplicates.
   - Sets the index to `Date` and calculates the Supertrend using `pandas_ta`.
   - Updates the data of the line objects (`line_close` and `line_supertrend`) instead of clearing the plot.
   - Adds lines to highlight uptrends and downtrends without clearing previous data.
   - Adjusts the plot's limits dynamically based on the updated data.
3. `FuncAnimation` updates the plot every 10 seconds.

This setup ensures that the plot continuously adds new data while maintaining the previously plotted data, giving a seamless update experience. Replace the `get_new_data` function with your actual simulator data retrieval code as needed.
