import requests
from datetime import datetime
from decimal import Decimal

API_URL = "https://api.etherscan.io/v2/api"

API_KEY = "YOUR_ETHERSCAN_API_KEY"
WALLET = "0x0000000000000000000000000000000000000000"

TOKEN_CONTRACT = "0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
TOKEN_SYMBOL = "USDC"
TOKEN_DECIMALS = 6


def get_token_transfers(wallet):
    params = {
        "chainid": 1,
        "module": "account",
        "action": "tokentx",
        "contractaddress": TOKEN_CONTRACT,
        "address": wallet,
        "page": 1,
        "offset": 20,
        "sort": "desc",
        "apikey": API_KEY
    }

    response = requests.get(
        API_URL,
        params=params,
        timeout=15
    )

    response.raise_for_status()
    data = response.json()

    if data.get("status") != "1":
        raise RuntimeError(
            data.get("message", "API request failed")
        )

    return data["result"]


def format_amount(raw_value):
    return Decimal(raw_value) / (
        Decimal(10) ** TOKEN_DECIMALS
    )


def main():
    transfers = get_token_transfers(WALLET)

    print(f"Wallet: {WALLET}")
    print(f"Token: {TOKEN_SYMBOL}")
    print(f"Transfers found: {len(transfers)}\n")

    for tx in transfers:
        amount = format_amount(tx["value"])

        timestamp = datetime.fromtimestamp(
            int(tx["timeStamp"])
        )

        is_outgoing = (
            tx["from"].lower() == WALLET.lower()
        )

        direction = "OUT" if is_outgoing else "IN"

        print("-" * 65)
        print(f"Time:      {timestamp}")
        print(f"Direction: {direction}")
        print(f"Amount:    {amount:,.2f} {TOKEN_SYMBOL}")
        print(f"From:      {tx['from']}")
        print(f"To:        {tx['to']}")
        print(f"Block:     {tx['blockNumber']}")
        print(f"Hash:      {tx['hash']}")


if __name__ == "__main__":
    main()
