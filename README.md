To store values of Supertrend calculated by Python libraries in NeDB, follow these steps:

1. **Calculate Supertrend**: Use a Python library to calculate the Supertrend indicator for your data.
2. **Install NeDB**: Set up NeDB in a Node.js environment.
3. **Store the Calculated Values**: Use a Python-to-Javascript bridge like `pyexecjs` or communicate with a Node.js script to store the values in NeDB.

Here’s a detailed guide for each step:

### Step 1: Calculate Supertrend in Python

You can use a library like `pandas-ta` to calculate Supertrend.

```python
import pandas as pd
import pandas_ta as ta

# Example data
data = {
    'date': pd.date_range(start='2022-01-01', periods=100, freq='D'),
    'close': pd.Series(range(100))
}
df = pd.DataFrame(data)

# Calculate Supertrend
df.ta.supertrend(append=True)

# Select relevant columns (Supertrend columns are named by default as 'SUPERT_7_3.0' and 'SUPERTd_7_3.0')
supertrend_data = df[['date', 'SUPERT_7_3.0', 'SUPERTd_7_3.0']]
print(supertrend_data)
```

### Step 2: Set Up NeDB

Install NeDB in your Node.js environment.

```sh
npm install nedb
```

Create a `store_data.js` file to handle data storage.

```javascript
const Datastore = require('nedb');
const db = new Datastore({ filename: 'supertrend.db', autoload: true });

const storeSupertrendData = (data) => {
    db.insert(data, (err, newDoc) => {
        if (err) {
            console.error('Error storing data:', err);
        } else {
            console.log('Data stored successfully:', newDoc);
        }
    });
};

// Make this function available for external calls
module.exports = { storeSupertrendData };
```

### Step 3: Store Calculated Values from Python

Use `pyexecjs` to run the Node.js script from Python.

Install `pyexecjs`:

```sh
pip install pyexecjs
```

Add the following code to call your Node.js script from Python:

```python
import execjs
import json

# Example data to be stored
data_to_store = supertrend_data.to_dict('records')

# Prepare the Node.js code to store data
node_code = """
const { storeSupertrendData } = require('./store_data');

function storeDataFromPython(data) {
    const parsedData = JSON.parse(data);
    storeSupertrendData(parsedData);
}

module.exports = { storeDataFromPython };
"""

# Load the Node.js context
context = execjs.compile(node_code)

# Call the store function with your data
context.call("storeDataFromPython", json.dumps(data_to_store))
```

### Complete Example

Here’s how you can integrate everything:

1. Calculate Supertrend.
2. Store the calculated values using a Node.js script with NeDB.

```python
import pandas as pd
import pandas_ta as ta
import execjs
import json

# Calculate Supertrend
data = {
    'date': pd.date_range(start='2022-01-01', periods=100, freq='D'),
    'close': pd.Series(range(100))
}
df = pd.DataFrame(data)
df.ta.supertrend(append=True)
supertrend_data = df[['date', 'SUPERT_7_3.0', 'SUPERTd_7_3.0']]

# Prepare data for storage
data_to_store = supertrend_data.to_dict('records')

# Node.js code to handle storage
node_code = """
const { storeSupertrendData } = require('./store_data');

function storeDataFromPython(data) {
    const parsedData = JSON.parse(data);
    storeSupertrendData(parsedData);
}

module.exports = { storeDataFromPython };
"""

# Load and execute Node.js code
context = execjs.compile(node_code)
context.call("storeDataFromPython", json.dumps(data_to_store))
```

This setup calculates Supertrend in Python and stores the results in NeDB via a Node.js script. Ensure your Node.js environment is properly configured to run this code.
