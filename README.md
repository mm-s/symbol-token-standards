# symbol-token-standards

[![npm version](https://badge.fury.io/js/symbol-token-standards.svg)](https://badge.fury.io/js/symbol-token-standards)
[![Build Status](https://travis-ci.com/nemgrouplimited/symbol-token-standards.svg?branch=main)](https://travis-ci.com/nemgrouplimited/symbol-token-standards)
[![Slack](https://img.shields.io/badge/chat-on%20slack-green.svg)](https://symbol.slack.com/messages/CB0UU89GS//)

*The author of this package cannot be held responsible for any loss of money or any malintentioned usage forms of this package. Please use this package with caution.*

Symbol Token Standards library to create security tokens / financial instruments for the Symbol platform.

This is a PoC to validate the proposed [NIP13 - Security Token Standard](https://github.com/nemtech/NIP/blob/main/NIPs/nip-0013.md). When stable, the repository will be moved to the [nemtech](https://github.com/nemtech) organization.

## Installation

`npm install symbol-token-standards`

## Example Library Usage

:warning: The following example usage for the `symbol-token-standards` library is subject to change.

```javascript
import { AggregateTransaction, PublicAccount, SignedTransaction } from 'symbol-sdk'
import { MnemonicPassPhrase } from 'symbol-hd-wallets'
import { NIP13, NetworkConfig, TransactionParameters } from 'symbol-token-standards'
import { TransactionURI } from 'symbol-uri-scheme'

// :warning: The following settings are network specific and may need changes
const transactionParams = new TransactionParameters(
  Deadline.create(),
  750000, // maxFee
)

// :warning: You should create separate backups of
// authorities and security token pass phrases.
const authKeys = MnemonicPassPhrase.createRandom() // backup the resulting 24-words safely!
const tokenKeys = MnemonicPassPhrase.createRandom() // backup the resulting 24-words safely!

// Funded operator accounts, we only need their public key and address.
// :warning: It is recommended to create operator keys offline using a separate device.
console.log('Token Operator 1:');
//secret key. Only known by op1:
const op1_sk = '901AF6A4FD879950601D0ADA406431E8074C88AA0A0CD2B73F996CE03557056C';
const op1 = Account.createFromPrivateKey(op1_sk, NetworkType.TEST_NET);
console.log('  secret key:', op1.privateKey);
console.log('  public key:', op1.publicKey);
console.log('  address:', op1.address.pretty());

console.log('Token Operator 2:');
//secret key. Only known by op2:
const op2_sk = 'DC163CBA61350C79AC55470FB426A7D0C6CB9523FD22A142495B0744FA825C0E';
const op2 = Account.createFromPrivateKey(op2_sk, NetworkType.TEST_NET);
console.log('  secret key:', op2.privateKey);
console.log('  public key:', op2.publicKey);
console.log('  address:', op2.address.pretty());

const operators = [
  new PublicAccount(op1.publicKey, op1.address),
  new PublicAccount(op2.publicKey, op2.address)
  // , new PublicAccount('PUBLIC_KEY_OPERATOR', 'ADDRESS_OPERATOR'),
]

// initialize NIP13 library
const network = new NetworkConfig(...)
const tokenAuthority = new NIP13.TokenAuthority(network, authKeys)
const securityToken = new NIP13.Token(network, tokenKeys)

// offline creation of the `CreateToken` security token contract
// Metadata, can be filled with any value.

const metadata = new SecuritiesMetadata(
      '', // ISO_10962 (MIC)
      '', // ISO_6166 (ISIN)
      '', // ISO_10383,
      '', // Website
      '', // Sector
      '', // Industry
      {}, // customMetadata
    )

const metadata_example_2 = new SecuritiesMetadata(
      'anything', // ISO_10962 (MIC)
      'anything', // ISO_6166 (ISIN)
      'anything', // ISO_10383,
      'anything', // Website
      'anything', // Sector
      'anything', // Industry
      {
        'customkey1': 'customdata1',
        // ...
      }, // customMetadata
    )

const tokenId = securityToken.create(
  'My Awesome Security Token', // security token name
  securityToken.getTarget().publicAccount, // actor
  tokenAuthority.getAuthority().publicAccount, // token authority
  operators,
  123456789, // total outstanding shares
  metadata,
  transactionParams,
)

// get the transaction URI for `CreateToken` execution
const resultURI: TransactionURI = securityToken.result

// :warning: It is recommended to sign the resulting transactions
// using a hardware wallet rather than any type of software generated
// wallets.
const transaction: AggregateTransaction = resultURI.toTransaction()
const signedTransaction: SignedTransaction = securityToken.getTarget().sign(transaction, 'networkGenerationHash')

// `signedTransaction` can now be broadcast to the Symbol network of choice.

// It is important to denote that given the **aggregate** nature of security
// token contracts, multiple parties MAY be involved in the transaction and
// it is therefor required to issue a HashLockTransaction before announcing
// the aggregate bonded transaction that represents the contract.
```

## License

Copyright 2020-present NEM

Licensed under the [Apache v2.0 License](LICENSE).
