Certainly! Let's create the Supertrend indicator calculation manually without using the `pandas-ta` library. We will plot and update the chart every 10 seconds with new data.

Here's how you can do it:

1. Define the Supertrend calculation function.
2. Use `matplotlib.animation` to update the plot every 10 seconds without clearing the previous plot.

First, install the necessary libraries:

```sh
pip install pandas numpy matplotlib
```

Here's the complete code:

```python
import pandas as pd
import numpy as np
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

# Function to calculate Supertrend
def calculate_supertrend(df, period=7, multiplier=3):
    # Compute the ATR (Average True Range)
    df['H-L'] = df['High'] - df['Low']
    df['H-Cp'] = np.abs(df['High'] - df['Close'].shift(1))
    df['L-Cp'] = np.abs(df['Low'] - df['Close'].shift(1))
    
    df['TR'] = df[['H-L', 'H-Cp', 'L-Cp']].max(axis=1)
    df['ATR'] = df['TR'].rolling(window=period).mean()
    
    # Compute the basic upper and lower bands
    df['Upper Band'] = ((df['High'] + df['Low']) / 2) + (multiplier * df['ATR'])
    df['Lower Band'] = ((df['High'] + df['Low']) / 2) - (multiplier * df['ATR'])
    
    # Initialize Supertrend columns
    df['Supertrend'] = np.nan
    df['In Uptrend'] = True

    for current in range(1, len(df.index)):
        previous = current - 1

        if df['Close'][current] > df['Upper Band'][previous]:
            df['Supertrend'][current] = df['Lower Band'][current]
            df['In Uptrend'][current] = True
        elif df['Close'][current] < df['Lower Band'][previous]:
            df['Supertrend'][current] = df['Upper Band'][current]
            df['In Uptrend'][current] = False
        else:
            df['Supertrend'][current] = df['Supertrend'][previous]
            df['In Uptrend'][current] = df['In Uptrend'][previous]
            if df['In Uptrend'][current] and df['Lower Band'][current] < df['Supertrend'][previous]:
                df['Supertrend'][current] = df['Lower Band'][current]
            if not df['In Uptrend'][current] and df['Upper Band'][current] > df['Supertrend'][previous]:
                df['Supertrend'][current] = df['Upper Band'][current]
                
    return df

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
    df = calculate_supertrend(df)
    
    # Update line data
    line_close.set_data(df.index, df['Close'])
    line_supertrend.set_data(df.index, df['Supertrend'])
    
    # Highlighting uptrends and downtrends
    for i in range(1, len(df)):
        if df['In Uptrend'].iloc[i]:
            ax.plot(df.index[i-1:i+1], df['Supertrend'].iloc[i-1:i+1], color='green')
        else:
            ax.plot(df.index[i-1:i+1], df['Supertrend'].iloc[i-1:i+1], color='red')
    
    ax.set_xlim(df.index.min(), df.index.max())
    ax.set_ylim(min(df['Low'].min(), df['Supertrend'].min()), max(df['High'].max(), df['Supertrend'].max()))
    ax.set_title('Supertrend Indicator')
    ax.set_xlabel('Date')
    ax.set_ylabel('Price')
    ax.legend()

# Create animation
ani = FuncAnimation(fig, update, interval=10000)  # Update every 10 seconds

plt.show()
```

In this code:
1. `get_new_data` simulates new data retrieval.
2. `calculate_supertrend` manually calculates the Supertrend indicator.
3. The `update` function appends new data, calculates the Supertrend, and updates the plot without clearing previous data.
4. `FuncAnimation` updates the plot every 10 seconds.

Replace `get_new_data` with your actual simulator data retrieval function to use this setup in your application.
