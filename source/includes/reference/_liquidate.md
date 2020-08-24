# liquidate #

A request from Simplex telling that you can liquidate the received crypto.

## Synopsis ##

API name: **`liquidate`**  
Direction: **Simplex &rarr; You**

## Parameters ##

> Example request:

```json
{
  "liquidation_order_id": "af492cb2-4318-5b07-8ece-be34f479e23b",
  "txn_id": "af492cb2-5b07-4318-8ece-be34f479e23b",
  "quote_id": "bb4fbdef-9abc-41c1-94d9-a670413c4d02",
  "destination_crypto_address": "1EmXYy57z71H8J5jrxXsdjuJXZnPZgHnjh",
  "crypto_currency": "BTC",
  "crypto_amount": 500000,
  "user_id": "595b88bea687c5dd444f99e0004a45d3",
  "user_aka_ids": ["1504241c7d83476aa3adcd54e2272d25", "38b583c7ccd246ffaed4ab0232b71647"]
}
```

Name                       | Type           |   |
-------------------------- | -------------- | - |
liquidation_order_id       | Id             | **required**
txn_id                     | Id             | **required**
quote_id                   | Id             | **required**
destination_crypto_address | CryptoAddress  | **required**
crypto_currency            | CryptoCurrency | **required**
crypto_amount              | MoneyAmount    | **required**
user_id                    | Id             | **required**
user_aka_ids               | List<Id>       | **required**
commission_crypto_amount   | Id             |
commission_crypto_address  | CryptoAddress  | **required**, if `commission_crypto_amount`

### liquidation_order_id ###
#### (Id, **required**)

The POST liquidate operation must be idempotent, meaning that if Simplex does not get a response (i.e. network error) the method invocation will be repeated with the same liquidation_order_id to avoid double liquidation.

### txn_id ###
#### (Id, **required**)

The identifier of the Simplex transaction involved.

### crypto_currency ###
#### (CryptoCurrency, **required**)

The crypto currency (the currency, not the amount) to be received.

This will match the quote's `quote_currency`, and is supplied as a convenience.

### crypto_amount ###
#### (MoneyAmount, **required**)

How much cryptocurrency of type `crypto_currency` is to be liquidated.

This will match the quote, and is supplied as a convenience.

**Note:** you should always check the actual amount received, as well as report it back to Simplex. End-users' wallets may, for example, subtract a small "blockchain fee" to help the blockchain transaction go through quickly.

If the amount you receive is only slightly different, and you can still honor the quote's rate with the slightly different amount, then you should do so.

Otherwise, you may reply with an `"amount_mismatch"` to `liquidate`.

In any case, settlements between Simplex and you and always based on actual amounts received.

### destination_crypto_address ###
#### (CryptoAddress, **required**)

The crypto that is accepted to the given destination_crypto_address should be liquidated.

This is an address you previously supplied Simplex in response to a `get-destination-crypto-address` message.

### user_id ###
#### (Id, **required**)

A unique identifier, created by Simplex, for the end-user performing the transaction.

Same `user_id` as a previous message means same end-user.

### user_aka_ids ###
#### (List<Id>, **required**)

A list of unique identifiers, on top of `user_id`, by which the user is also known.

## Response ##

Name         | Type        |   |
------------ | ----------- | - |
liquidation  | Liquidation | **required**

## Type Liquidation ###

Name                   | Type           |   |
---------------------- | -------------- | - |
id                     | Id             | **required**
status                 | String         | **required**
result                 | String         | **required** if `status == "completed"`
reasons                | List\<String\> | **required** if `result == "rejected"`
blockchain_txn_hash    | Id             | **required** if `result == "accepted"`

### id ###
#### (Id, **required**)

An opaque string generated by you and stored by Simplex.

Alternatively you can use `liquidation_order_id` for this value.

### status ###
#### (String, **required**)

One of { `"pending"`, `"completed"` }.

### result ###
#### (String, **required** if `status == "completed"`, missing otherwise)

One of { `"accepted"`, `"rejected"` }.

### reasons ###
#### (List\<String\>, **required** if `result == "rejected"`)

If you reply with a `"reject"` result, this is a list of all the reason codes why.

Each reason code is one of:

 * `"quote_invalid"` : the quote supplied is not valid.

 * `"quote_expired"` : the quote supplied is no longer valid.

 * `"quote_amount_mismatch"` : the actual crypto amount you received is "too different" from the one specified in the quote, and you cannot honor the quote rate for the actual amount.

 * `"crypto_amount_mismatch"` : to be used if a calculation error/mismatch has occurred.

 * `"crypto__address_bad_reputation"` : you checked the reputation of crypto address that is the source of the blockchain transaction, and concluded you do not wish to perform this transaction.

 * `"user_banned"` : the user is on your blacklist and you are cannot accept any transaction from this user.

 * `"aml_reject"` : you cannot accept the cryptocurrency due to an AML-related reason not specified above.

 * `"reject_other"` : you cannot accept the cryptocurrency due to a reason not specified above.

 * `"txn_needs_poid"` : according to your AML policy this transaction requires that the user have a valid proof-of-identity, but you do not have a valid proof-of-identity for the user.


### blockchain_txn_hash ###
#### (Id, **required** if `result == "accepted"`, may be present even otherwise)

Blockchain transaction hash.