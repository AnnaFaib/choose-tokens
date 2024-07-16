# choose-tokens
import ccxt
import pandas as pd
import time

# Initialize the exchange
exchange = ccxt.binance({
    'apiKey': 'YOUR_API_KEY',
    'secret': 'YOUR_API_SECRET',
})

# Define the tokens to watch
tokens = ['BTC/USDT', 'ETH/USDT', 'BNB/USDT']

# Define a function to get historical data
def get_historical_data(symbol, timeframe='1d', limit=100):
    ohlcv = exchange.fetch_ohlcv(symbol, timeframe, limit=limit)
    data = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
    data['timestamp'] = pd.to_datetime(data['timestamp'], unit='ms')
    return data

# Define a function to calculate a simple moving average (SMA)
def calculate_sma(data, period=50):
    data[f'SMA_{period}'] = data['close'].rolling(window=period).mean()
    return data

# Define a function to check buy conditions
def check_buy_conditions(data):
    if data['close'].iloc[-1] < data[f'SMA_50'].iloc[-1]:
        return True
    return False

# Define a function to place a market order
def place_order(symbol, side, amount):
    try:
        order = exchange.create_order(symbol, 'market', side, amount)
        return order
    except Exception as e:
        print(f"An error occurred: {e}")
        return None

# Monitor and trade
def monitor_and_trade():
    while True:
        for token in tokens:
            data = get_historical_data(token)
            data = calculate_sma(data)
            
            if check_buy_conditions(data):
                balance = exchange.fetch_balance()
                usdt_balance = balance['total']['USDT']
                
                # Calculate the amount to buy
                amount_to_buy = usdt_balance / data['close'].iloc[-1]
                
                order = place_order(token, 'buy', amount_to_buy)
                if order:
                    print(f"Bought {amount_to_buy} of {token} at {data['close'].iloc[-1]}")
        
        # Wait before the next check
        time.sleep(3600)  # Check every hour

# Start monitoring and trading
monitor_and_trade()
