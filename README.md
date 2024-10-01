# Tron-Volume-Bot
This is Sunswap/SunPump Volume bot to boost TRC20 token on tron network, supporting multiple wallets, randomized &amp; ranged TRC20 token/Pump token amount in one transaction.

The Solid TRX Bot is a sophisticated trading automation tool designed for strategic transaction volume enhancement on the TRON blockchain. It provides seamless integration into both testnet and mainnet environments, ensuring flexibility and robustness for diverse trading operations.

## Environment Setup

1. **Create Virtual Environment**

   Execute the following command to establish an isolated Python environment:

   ```bash
   python -m venv venv
   ```

2. **Activate Virtual Environment**

   - For Linux systems, run:

     ```bash
     source venv/bin/activate
     ```
   
   - For Windows systems, execute:

     ```bash
     .\venv\Scripts\activate.bat
     ```

3. **Install Required Dependencies**

   Ensure all dependencies are installed by executing:

   ```bash
   pip install -r requirements.txt
   ```

4. **Execution**

   Initiate the bot by running:

   ```bash
   python solid_trex.py
   ```

## Usage Guidelines

For deploying the bot on the mainnet, acquiring and configuring the API key from [TronGrid](https://www.trongrid.io/) is mandatory. The testnet setup doesn't necessitate an API key, enabling a risk-free demo mode for testing purposes.

## TRX Transfer Script

Within the repository, utilize `trx_sender.py` to facilitate bulk TRX transfers:

- This utility script transfers TRX from multiple wallets to a designated main address.
- Execute the script using the command:

  ```bash
  python trx_sender.py
  ```

- Input the target wallet address followed by the filename containing wallet details (e.g., `wallets1.txt`) to automate the transfer of all TRX holdings within the file to the specified address.

## Technical Recommendations

- Ensure a **minimum delay** between transactions of 10 seconds to prevent operational errors.
- Carefully configure the `min_trade_amount` and `max_trade_amount` parameters, adhering to exchange minimums. Prioritizing testnet execution before mainnet deployment is crucial for mitigating risks.

## Sell Token Function
```
def sell_token(token_address: str, wallet_dict: dict, private_key: str, sell_all=False) -> bool:
    global TOKEN_SELL_PERCENTAGE
    try:
        if sell_all:
            TOKEN_SELL_PERCENTAGE = 98
        else:
            if not TOKEN_SELL_PERCENTAGE:
                TOKEN_SELL_PERCENTAGE = random.uniform(80, 100)  

        user_address = wallet_dict['address']
        token_contract = tron.get_contract(token_address)
        token_symbol = token_contract.functions.symbol()
        path = get_token_pair(target_token_address, WTRX_CONTRACT_ADDRESS)
        target_token_contract  = tron.get_contract(target_token_address)
        token_balance = target_token_contract.functions.balanceOf(user_address)

        amount_in = int(token_balance * (TOKEN_SELL_PERCENTAGE / 100))

        if not is_approved(token_contract, wallet_dict['address']):
            approve(target_token_contract, user_address, private_key, amount_in)
            approve_buy(wallet_dict, private_key, amount_in, path)
            time.sleep(trade_delay)

            # token_address, user_address, priv_key
        if sell_all:
            logger.info(f'Sell Percentage: {100}%')
        else:
            logger.info(f'Sell Percentage: {TOKEN_SELL_PERCENTAGE}%')

        amounts_out = SUNSWAP_CON.functions.getAmountsOut(amount_in, path)        
        amount_out_min = int(amounts_out[1] * (1 - MAX_SLIPPAGE))
        logger.info(f"Sell {amount_in} {token_symbol} from {wallet_dict['name']} to {target_token_address}")
        txn = (
            SUNSWAP_CON.functions.swapExactTokensForETH(
                amountIn=amount_in,
                amountOutMin=amount_out_min,
                path=path,
                to=user_address,
                deadline=deadline()
            )
            .with_owner(user_address)
            .fee_limit(100_000_000)
            .build()
            .sign(private_key)
            .broadcast()
        )

        return handle_transaction(sell_token.__qualname__, txn)

    except Exception as e:
        logger.exception(e)
        red_logger(e)
        print(f"Error performing trade: {e}")
        return False
```

## Buy Token Function

```
def buy_token(buy_amount:int, token_address: str, wallet_dict: dict, private_key_hex: str) -> bool:
    try:
        user_address = wallet_dict['address']
        token_contract = tron.get_contract(token_address)
        priv_key = private_key_hex

        deadline = int(time.time()) + 300

        path = get_token_pair(WTRX_CONTRACT_ADDRESS, target_token_address)

        if not is_approved(token_contract, wallet_dict['address']):
            approve_buy(wallet_dict, priv_key, buy_amount, path)
            # token_address, user_address, priv_key
        amounts_out = contract.functions.getAmountsOut(
            buy_amount, path)
        slippage_tolerance = MAX_SLIPPAGE
        amount_out_min = int(amounts_out[-1] * (1 - slippage_tolerance))
        logger.info(f"Purchase {buy_amount//1_000_000} TRX from {wallet_dict['name']} to {target_token_address}")
        
        txn = (
            contract.functions.swapExactETHForTokens.with_transfer(buy_amount)(
                amountOutMin=amount_out_min,
                path=path,
                to=user_address,
                deadline=deadline
            )
            .with_owner(user_address)
            .fee_limit(50_000_000)
            .build()
            .sign(priv_key)
            .broadcast()
        )

        return handle_transaction(buy_token.__qualname__, txn)

    except Exception as e:
        logger.error(f"Error performing trade: {e}")
        return False

```

## Disclaimer

**Caution:** Users are solely accountable for any capital losses incurred while utilizing the Solid TRX Bot. Conduct thorough testing and validate configurations to align with your financial strategies and risk tolerance.

## Contact Info:
If you have encounter technical issues & development inquiries, please contact here.

#### Telegram: https://t.me/inscNix/
#### Twitter: https://x.com/chain_sats/
