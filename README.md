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
           ⬤
