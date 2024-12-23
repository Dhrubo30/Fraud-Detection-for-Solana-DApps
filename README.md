import requests
import pandas as pd
from sklearn.ensemble import IsolationForest


SOLANA_RPC_URL = "https://api.mainnet-beta.solana.com"


def fetch_transactions(address):
    headers = {"Content-Type": "application/json"}
    payload = {
        "jsonrpc": "2.0",
        "id": 1,
        "method": "getSignaturesForAddress",
        "params": [address, {"limit": 100}]
    }
    response = requests.post(SOLANA_RPC_URL, json=payload, headers=headers)
    if response.status_code == 200:
        return response.json()["result"]
    else:
        print("Error fetching transactions:", response.status_code, response.text)
        return []


def analyze_transactions(transactions):
    
    data = pd.DataFrame(transactions)
    if data.empty:
        print("No transactions found.")
        return None
    
    
    data['blockTime'] = pd.to_datetime(data['blockTime'], unit='s')
    data['time_diff'] = data['blockTime'].diff().dt.total_seconds().fillna(0)
    data['slot_diff'] = data['slot'].diff().fillna(0)
    features = data[['time_diff', 'slot_diff']]
    model = IsolationForest(contamination=0.1, random_state=42)
    data['anomaly'] = model.fit_predict(features)

    anomalies = data[data['anomaly'] == -1]
    return anomalies

if __name__ == "__main__":

    wallet_address = "YOUR_WALLET_ADDRESS_HERE"


    print("Fetching transactions...")
    transactions = fetch_transactions(wallet_address)


    if transactions:
        print("Analyzing transactions...")
        anomalies = analyze_transactions(transactions)

        if anomalies is not None and not anomalies.empty:
            print("Suspicious transactions detected:")
            print(anomalies)
        else:
            print("No suspicious transactions detected.")
    else:
        print("No transactions to analyze.")
