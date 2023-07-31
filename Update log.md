# OpenAPI Update Log
## 12/12/23
- New endpoint: 7.6.9. Fetch Transaction Details by Request ID
  
## 12/01/23
- Modified endpoint: 7.5.1. Transaction Notification API Callback
  - New request parameter: `processTime` 
  - Callbacks will be sent in sequence based on the generic order of the transaction status 
  - Callbacks for transactions will be sent independently per transaction and in parallel sequence when concurrent transactions occur, instead of sending batch transactions in a single callback 
  - Callbacks for separate transaction statuses will be sent in minimum intervals of 1 second

## 13/12/22
- New coin type: `SGD-POS-BRIDGED`

## 11/08/22
- New coin type: `SOL`, `AVAX` (C-Chain)

## 27/07/22
- New coin type: `MATIC`, `USDC-POS`, `XIDR-POS`, `XSGD-POS`
- Updated coin type name: `MATIC-ERC20`

## 12/05/22
- Modified endpoint: 7.4.1. Generate Child Addresses For a Master Address
  - Change: the number of child addresses that can be created per request has changed

## 18/03/22
- New coin type: `FIL`, `GXT-BEP20`

## 07/03/22
- 7.1.10. Specify amount to collect from child address to master address  
   \- New response parameter: `transaction ID`

## 16/02/22
- New endpoint: 7.5.2 HTS Coin Association Notification API Callback

## 09/02/22
- New endpoint: 7.1.10 Specify amount to collect from child address to master address

## 24/01/22
- New coin type: `SAC`, `GXT`

## 14/01/22 Updates
- New coin type: `XSGD-HTS`

## 07/01/22 Updates
- New endpoint: 7.5.2 HTS Coin Association Notification API Callback
- New endpoint: 7.6.2 HBAR Address Association With Multiple HTS Coins
- New endpoint: 7.6.3 HTS Coin Association With Multiple HBAR Addresses
- New error code: 106069

## 10/12/21 Updates
- Added description for HBAR child address creation: 7.4.1. Generate Child Addresses For a Master Address 
- Modified endpoint: 7.6.1 Single HBAR address association with a single HTS coin
- Removed endpoint: 7.6.2 HTS coin association with multiple HBAR addresses
- Added new error code: 106068

## 30/11/21 Updates
- Added new coin type: `BNB`, `FIL-BEP20`, `USDT-BEP20`, `BUSD-BEP20`, `USDC-BEP20`, `HBAR`
- New endpoint: 7.6.1 HBAR address association with multiple HTS coins
- New endpoint: 7.6.2 HTS coin association with multiple HBAR addresses
- Added new error codes 106061~106067

## 22/10/21 Updates
- Increased API request limits for various API operations

## 20/10/21 Updates
- Added new error codes 106025~106060

## 23/08/21 Updates
- Cancelled API key expiration 
- Increased supported coin types
- Other minor fixes

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
