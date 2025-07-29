import time
import requests

# ====== CONFIGURATION =======
client_id = " Dhan client ID "
access_token = "Your acces token of DHAN"
BASE_URL = "https://api.dhan.co"
headers = {
    "Accept": "application/json",
    "access-token": access_token,
    "client-id": client_id
}

# ====== LOGIC VARIABLES =======
locked_profit = 0
lock_trigger = 1000
lock_step = 500

# ====== FUNCTIONS =========
def fetch_positions():
    try:
        url = f"{BASE_URL}/positions"
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
        else:
            print(f"Error fetching positions: {response.status_code} {response.text}")
            return []
    except Exception as e:
        print(f"Exception fetching positions: {e}")
        return []

def calculate_mtm(positions):
    mtm = 0
    for pos in positions:
        if pos.get("netQty", 0) != 0:
            realized = float(pos.get("realizedProfit", 0))
            unrealized = float(pos.get("unrealizedProfit", 0))
            mtm += realized + unrealized
    return mtm

def close_all_positions(positions):
    for pos in positions:
        net_qty = pos.get("netQty", 0)
        if net_qty != 0:
            side = "SELL" if net_qty > 0 else "BUY"
            quantity = abs(net_qty)
            security_id = pos["securityId"]

            order_payload = {
                "securityId": security_id,
                "exchangeSegment": pos["exchangeSegment"],
                "transactionType": side,
                "productType": pos["productType"],
                "orderType": "MARKET",
                "quantity": quantity,
                "price": 0,
                "triggerPrice": 0,
                "afterMarketOrder": False,
                "amoTime": "OPEN",
                "disclosedQuantity": 0,
                "validity": "DAY",
                "offlineOrder": False
            }

            response = requests.post(f"{BASE_URL}/orders", headers=headers, json=order_payload)
            print(f"Closing {side} {quantity} of {pos['tradingSymbol']} → Status: {response.status_code}")

def monitor():
    global locked_profit, lock_trigger

    while True:
        positions = fetch_positions()
        mtm = calculate_mtm(positions)
        print(f"\n MTM P&L: ₹{mtm:.2f} | Locked Profit: ₹{locked_profit}")

        if mtm <= -3000:
            print("Max loss hit. Closing all positions.")
            close_all_positions(positions)
            break

        elif mtm >= 7000:
            print(" Target profit hit. Closing all positions.")
            close_all_positions(positions)
            break

        elif mtm >= lock_trigger + lock_step:
            locked_profit += lock_step
            lock_trigger += lock_step
            print(f" Trailing profit lock updated → New Locked Profit: ₹{locked_profit}")

        elif mtm <= locked_profit and locked_profit > 0:
            print(" MTM dropped below locked profit. Closing all positions.")
            close_all_positions(positions)
            break

        time.sleep(5)  # Check every 10 seconds

# ====== RUN MONITORING LOOP =======
if __name__ == "__main__":
    monitor()
