# OpenAPI Update Log

## 20/05/21 Updates

- New endpoints:
  7.1.9 Fetch the current transaction fee for a particular coin type
- 7.3.1 Send a withdrawal request
  \- New request parameter: `fee`

## 13/04/21 Updates

- New endpoints:  
  7.1.6. Fetch address-wise balance details by coin type  
  7.1.7. Collect funds from child addresses and transfer to master address when balance is greater than the specified threshold  
  7.1.8. Update auto-collect configuration  
- Other minor fixes


## 26/02/21 Updates

- 7.2.1 Fetch list of transactions 
    \- Sample response fix
- Other minor fixes

## 22/02/21 Updates
  
- 7.2.1 Fetch list of transactions
    \- New request parameters: `address`


## 03/02/21 Updates

- 7.1.1. Fetch details for the assets in a wallet
    \- New response parameter: `alarm_frequency`
- 7.1.2. Fetch wallet details by coin type
    \- New response parameter: `alarm_frequency`
- 7.1.5 Fetch wallet details for master and child addresses
    \- New request parameters: `hd_address`, `coin_type`, `start_balance`, `end_balance`
    \- New response parameters: `alarm_frequency`
- 7.2.1 Fetch list of transactions
    \- New request parameters: `tx_type`, `start_tx_amount`, `end_tx_amount`