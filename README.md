To handle more than 100 unique calculations simultaneously using microservices, you can take advantage of concurrent processing and distributed computing. Here's how you can scale your microservices architecture to achieve this:

1. **Parallel Processing**: Use Python's `concurrent.futures` or similar libraries to parallelize the calculations.
2. **Distributed Microservices**: Deploy multiple instances of your microservices using a container orchestration system like Kubernetes.
3. **Load Balancing**: Use a load balancer to distribute incoming requests across multiple service instances.
4. **Messaging with Kafka**: Use Kafka to handle message queues and distribute tasks among multiple microservices.

Below is an example demonstrating how to implement parallel processing and Kafka for distributing the workload across multiple microservices.

### 1. Technical Indicator Modules

#### `oscillator.py`
```python
import pandas as pd
import numpy as np

class StochasticOscillator:
    def __init__(self, period=14):
        self.period = period

    def calculate(self, high_prices, low_prices, close_prices):
        high = np.array(high_prices)
        low = np.array(low_prices)
        close = np.array(close_prices)

        lowest_low = pd.Series(low).rolling(window=self.period, min_periods=self.period).min()
        highest_high = pd.Series(high).rolling(window=self.period, min_periods=self.period).max()
        k_percent = 100 * ((close - lowest_low) / (highest_high - lowest_low))
        d_percent = k_percent.rolling(window=3).mean()

        return k_percent, d_percent
```

### 2. Flask Microservices

#### `microservice_1.py` (Producer)
This service sends stock data to Kafka.

```python
from flask import Flask, request, jsonify
from kafka import KafkaProducer
import json
import pandas as pd

app = Flask(__name__)

producer = KafkaProducer(bootstrap_servers='localhost:9092', value_serializer=lambda v: json.dumps(v).encode('utf-8'))

@app.route('/send', methods=['POST'])
def send_message():
    data = request.get_json()
    company_id = data.get('company_id')
    file_path = data.get('file_path')
    
    # Read CSV file
    df = pd.read_csv(file_path)
    
    # Send each row to Kafka
    for _, row in df.iterrows():
        message = row.to_dict()
        message['company_id'] = company_id
        producer.send('stock_topic', message)
    
    return jsonify({'status': 'Messages sent to Kafka'}), 200

if __name__ == "__main__":
    app.run(port=5001)
```

#### `microservice_2.py` (Consumer)
This service receives stock data from Kafka and calculates the indicators.

```python
from flask import Flask, jsonify
from kafka import KafkaConsumer
from concurrent.futures import ThreadPoolExecutor
import json
import pandas as pd
import numpy as np
from oscillator import StochasticOscillator

app = Flask(__name__)

consumer = KafkaConsumer(
    'stock_topic',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    group_id='my-group',
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

executor = ThreadPoolExecutor(max_workers=10)
results = []

def calculate_indicators(data):
    # Assuming data is a dictionary with keys: High, Low, Close, company_id
    df = pd.DataFrame([data])
    df['High'] = df['High'].astype(float)
    df['Low'] = df['Low'].astype(float)
    df['Close'] = df['Close'].astype(float)
    
    # Calculate Stochastic Oscillator
    stochastic_oscillator = StochasticOscillator()
    k_percent, d_percent = stochastic_oscillator.calculate(df['High'], df['Low'], df['Close'])
    
    result = {
        'company_id': data['company_id'],
        'Stochastic %K': k_percent.tolist(),
        'Stochastic %D': d_percent.tolist()
    }
    return result

@app.route('/receive', methods=['GET'])
def receive_messages():
    futures = []
    for message in consumer:
        data = message.value
        futures.append(executor.submit(calculate_indicators, data))
        
        if len(futures) >= 100:
            break

    global results
    results = [future.result() for future in futures]
    return jsonify(results), 200

if __name__ == "__main__":
    app.run(port=5002)
```

### 3. Install Dependencies
Install the necessary Python packages using pip:
```sh
pip install flask kafka-python pandas numpy concurrent.futures
```

### 4. Run the Microservices
Start the two Flask microservices in separate terminal windows:

```sh
python microservice_1.py
```

```sh
python microservice_2.py
```

### 5. Sending and Receiving Messages
Send messages to `microservice_1` to distribute tasks to Kafka and calculate indicators using `microservice_2`.

#### Send a message to `microservice_1`:
```sh
curl -X POST -H "Content-Type: application/json" -d '{"company_id": 1, "file_path": "data_company_a.csv"}' http://localhost:5001/send
```

#### Receive and calculate indicators from `microservice_2`:
```sh
curl http://localhost:5002/receive
```

### Notes:
1. **Kafka Topics**: Ensure the Kafka topic (`stock_topic` in this example) exists. You can create it using Kafka CLI tools.
2. **Kafka Configuration**: Adjust `bootstrap_servers` and other Kafka configurations as per your setup.
3. **Parallel Processing**: The `ThreadPoolExecutor` is used for parallel processing. Adjust the `max_workers` parameter based on your system's capacity.
4. **Scaling with Kubernetes**: For a production setup, consider using Kubernetes to deploy and manage multiple instances of your microservices for scalability and fault tolerance.
5. **Load Balancing**: Use a load balancer to distribute incoming requests across multiple instances of your microservices.

This setup allows you to handle more than 100 unique calculations simultaneously by distributing tasks across multiple instances of your microservices and processing them in parallel.
