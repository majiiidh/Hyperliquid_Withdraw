# Withdraw Staked $HYPE Using Hyperliquid Python SDK

This repository provides simple Python examples for interacting with the [Hyperliquid](https://hyperliquid.xyz) network using the [Official Python SDK](https://github.com/hyperliquid-dex/hyperliquid-python-sdk).  

Currently, it includes a script to **undelegate from a validator** and **withdraw from the staking account back to your spot account**.

---

## ⚠️ Security Warning
- Hard‑coding your private key in a source file is **NOT SECURE**.
- **NEVER** commit it to a public repository or share it with others! **REMOVE** the Python files after running the codes.
- This repository is for **educational purposes only**. 

---
## Installation

Install the required dependencies:

```
pip install eth-account hyperliquid-python-sdk
```

---
## Undelegate stake from a validator

After delegating HYPE to a validator, there is a 1-day lockup period. Once this period ends, you can undelegate your tokens. They will then be available in your staking account immediately after undelegation.

1. Create a Python file:
```
nano undelegate.py
```

2. Copy the script below into the file, replacing `PRIVATE_KEY`, `VALIDATOR_ADDRESS`, and `AMOUNT_HYPE` with your own values. You can find validator addresses in the Staking Action History on the Hyperliquid platform. Save the file (ctrl+x, y, Enter).


```
from eth_account import Account
from hyperliquid.api import API
from hyperliquid.exchange import Exchange  # Exchange inherits API
from hyperliquid.utils.constants import MAINNET_API_URL
from hyperliquid.utils.signing import get_timestamp_ms

# --- BEGIN CONFIGURATION ---
# Replace the private key below with your own.  Keep it secret!
PRIVATE_KEY = "0xREPLACE_WITH_YOUR_PRIVATE_KEY"
VALIDATOR_ADDRESS = "0xREPLACE_VALIDATOR_ADDRESS"
AMOUNT_HYPE = 1.0  # 1 HYPE
IS_UNDELEGATE = True
# --- END CONFIGURATION ---

def to_wei(hype_amount: float) -> int:
    return int(hype_amount * 10**8)

# Set up wallet and exchange client
wallet = Account.from_key(PRIVATE_KEY)
exchange = Exchange(wallet=wallet, base_url=MAINNET_API_URL)

# Convert amount to wei
undelegate_amount = to_wei(AMOUNT_HYPE)

# Perform the undelegation
undelegate_result = exchange.token_delegate(
    validator=VALIDATOR_ADDRESS,
    wei=undelegate_amount,
    is_undelegate=IS_UNDELEGATE
)

print(undelegate_result)
```

3. Run the script:
```
python3 undelegate.py
```

A successful response will look like:
```
{'status': 'ok', 'response': {'type': 'default'}}
```

---

## Withdraw from staking

This script allows you to transfer tokens from staking to your spot account. Note that transfers go through a 7-day unstaking queue.


1. Create a Python file:
```
nano withdraw.py
```
2. Copy the script below into the file, replace `PRIVATE_KEY` and `AMOUNT_HYPE` with your own values, then save the file (ctrl+x, y, Enter).

```
from eth_account import Account

from hyperliquid.api import API
from hyperliquid.utils.constants import MAINNET_API_URL, TESTNET_API_URL
from hyperliquid.utils.signing import sign_user_signed_action, get_timestamp_ms

# --- BEGIN CONFIGURATION ---
# Replace the private key below with your own.  Keep it secret!
PRIVATE_KEY = "0xREPLACE_WITH_YOUR_PRIVATE_KEY"
# Amount of HYPE to withdraw from staking (float). 1.0 means 1 HYPE.
AMOUNT_HYPE = 1.0
# Set to False to target mainnet; True uses the testnet endpoint.
IS_TESTNET = False
# --- END CONFIGURATION ---

C_WITHDRAW_SIGN_TYPES = [
    {"name": "hyperliquidChain", "type": "string"},
    {"name": "wei", "type": "uint64"},
    {"name": "nonce", "type": "uint64"},
]


def to_wei(hype_amount: float) -> int:
    """
    Convert an amount of HYPE into wei.  HYPE has 8 decimals, so one HYPE is
    10**8 wei.
    """
    return int(hype_amount * 10 ** 8)


def withdraw_from_staking(private_key: str, amount_hype: float, *, is_testnet: bool = False) -> dict:
    """
    Submit a staking withdrawal (cWithdraw) using the supplied private key.

    Parameters
    ----------
    private_key : str
        The private key for the account performing the withdrawal.
    amount_hype : float
        The amount of HYPE to withdraw from staking.
    is_testnet : bool, optional
        If True, use Hyperliquid’s testnet; otherwise, use mainnet.

    Returns
    -------
    dict
        The JSON response from the API call.
    """
    base_url = TESTNET_API_URL if is_testnet else MAINNET_API_URL
    wallet = Account.from_key(private_key)
    timestamp = get_timestamp_ms()
    wei_amount = to_wei(amount_hype)

    action = {
        "type": "cWithdraw",
        "wei": wei_amount,
        "nonce": timestamp,
    }

    is_mainnet = not is_testnet
    signature = sign_user_signed_action(
        wallet,
        action,
        C_WITHDRAW_SIGN_TYPES,
        "HyperliquidTransaction:CWithdraw",
        is_mainnet,
    )

    payload = {
        "action": action,
        "nonce": timestamp,
        "signature": signature,
    }

    api = API(base_url=base_url)
    response = api.post("/exchange", payload)
    return response


if __name__ == "__main__":
    result = withdraw_from_staking(PRIVATE_KEY, AMOUNT_HYPE, is_testnet=IS_TESTNET)
    print(result)
```

3. Run the script:
```
python3 withdraw.py
```

A successful response will look like:
```
{'status': 'ok', 'response': {'type': 'default'}}
```

---

## ⚠️ Cleanup

After running the scripts, remove them to protect your private key:

```
rm undelegate.py withdraw.py
```


---


**Transfer funds to you desired network**

After the 7-day unstaking period, you can transfer $HYPE from Hyperliquid to HyperEVM using a bridge like [Hyperdash](https://hyperdash.info/bridge), and then move tokens to your target network via (JumperExchange)[https://jumper.exchange] or any other bridge supporting HyperEVM.



