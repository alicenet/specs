---
title: Transaction Object Specification
author: chgorman
about: Create a transaction object specification
status: Draft
created: 2022-11-22
---

# Transaction Object Specification

We present the formal specification of the transaction object.

## Introduction

The transaction is central to AliceNet:
it fully specifies a state transition.

## Objects

In order to talk about transactions, we need to talk about utxos:
unspent transaction outputs.
These utxos may be consumed in order to make additional utxos.

### DataStore

The DataStore object stores data (bytes) for a specific period of time
(in terms of epochs; 1 epoch is 1024 AliceNet blocks).
The total cost of creating a DataStore is determined by
the amount of data (in terms of bytes)
and length of storage (in terms of epochs);
there is also a minimum size cost
(for associated storage related to the DataStore)
and a per-epoch fee.

```
DataStore {
    DSLinker  (DSLinker)
    Signature (DataStoreSignature)
}
```

```
DSLinker {
    DSPreImage (DSPreImage)
    Txhash     (32 byte array)
}
```

```
DSPreImage {
    ChainID  (uint32)
    IssuedAt (uint32)
    Index    (32 byte array)
    RawData  (byte array)
    TxOutIdx (uint32)
    Deposit  (uint256)
    Owner    (DataStoreOwner)
    Fee      (uint256)
}
```

```
DataStoreOwner {
    SVA       (byte)
    CurveSpec (byte) 
    Account   (20 byte array)
}
```

```
DataStoreSignature {
    SVA       (byte)
    CurveSpec (byte) 
    Signature (byte array)
}
```

 *  DataStore
    - DSLinker:
      specifies the DSLinker
    - Signature:
      specifies the signature of the DataStore
 *  DSLinker
    - DSPreImage:
      specifies the DSPreImage
    - Txhash:
      specifies the transaction hash
 *  DSPreImage
    - ChainID:
      specifies the ChainID for this object;
      **must** be nonzero
    - IssuedAt:
      specifies the epoch in which the DataStore was issued;
      **must** be nonzero
    - Index:
      specifies the index where the data is store
    - RawData:
      specifies the raw data to be stored;
      **must** be nonzero length byte array (upper bound **TBD**)
    - TxOutIdx:
      specifies the index of the UTXO within the transaction
    - Deposit:
      specifies the total cost of the DataStore
    - Owner:
      specifies the owner of the DataStore
    - Fee:
      specifies the per-epoch fee for storing the DataStore
 *  DataStoreOwner
    - SVA:
      specifies the Signature Verification Algorithm
    - CurveSpec:
      specifies the elliptic curve used for the signature
    - Account:
      specifies the account corresponding to the Owner
 *  DataStoreSignature
    - SVA:
      specifies the Signature Verification Algorithm
    - CurveSpec:
      specifies the elliptic curve used for the signature
    - Signature:
      specifies the digital signature



### ValueStore

The ValueStore object stores value (in terms of ALCB, the utility token).

```
ValueStore {
    VSPreImage (VSPreImage)
    TxHash     (32 byte array)
}
```

```
VSPreImage {
    ChainID  (uint32)
    TxOutIdx (uint32)
    Value    (uint256)
    Owner    (ValueStoreOwner)
    Fee      (uint256)
}
```

```
ValueStoreOwner {
    SVA       (byte)
    CurveSpec (byte) 
    Account   (20 byte array)
}
```

```
ValueStoreSignature {
    SVA       (byte)
    CurveSpec (byte) 
    Signature (byte array)
}
```

 *  ValueStore
    - VSPreImage:
      specifies the VSPreImage
    - TxHash:
      specifies the transaction hash
 *  VSPreImage
    - ChainID:
      specifies the ChainID for this object;
      **must** be nonzero
    - TxOutIdx:
      specifies the index of the UTXO within the transaction
    - Value:
      specifies the total value of the ValueStore;
      **must** be nonzero
    - Owner:
      specifies the owner of the ValueStore
    - Fee:
      specifies the fee for storing the ValueStore
 *  ValueStoreOwner
    - SVA:
      specifies the Signature Verification Algorithm
    - CurveSpec:
      specifies the elliptic curve used for the signature
    - Account:
      specifies the account corresponding to the Owner
 *  ValueStoreSignature
    - SVA:
      specifies the Signature Verification Algorithm
    - CurveSpec:
      specifies the elliptic curve used for the signature
    - Signature:
      specifies the digital signature

### UTXO

UTXOs are Unspent Transaction Outputs.
A UTXO may be either a ValueStore or a DataStore
and is denoted as `TxOut`.

```
TxOut {
    union {
        DataStore  (DataStore)
        ValueStore (ValueStore)
    }
}
```

### TXIN

TXINs are Transaction Inputs;
UTXOs are converted into TXINs when they are set to be consumed
by a transaction;
they are denoted as `TxIn`.

```
TxIn {
    TxInLinker (TxInLinker)
    Signature  (byte array)
}
```

```
TxInLinker {
    TxInPreImage (TxInPreImage)
    TxHash       (32 byte array)
}
```

```
TxInPreImage {
    ChainID          (uint32)
    ConsumedTxOutIdx (uint32)
    ConsumedTxHash   (32 byte array)
}
```

 *  TxIn
    - TxInLinker:
      specifies the TxInLinker object
    - Signature:
      specifies the signature of the object
 *  TxInLinker
    - TxInPreImage:
      specifies the TxInPreImage object
    - TxHash:
      specifies the transaction hash of the object
 *  TxInPreImage
    - ChainID:
      specifies the ChainID for this object;
      **must** be nonzero
    - ConsumedTxIdx:
      specifies the TxOutIdx of the UTXO being consumed
    - ConsumedTxHash:
      specifies the TxHash of the UTXO being consumed

### Tx

The transaction object is central to AliceNet.

```
Tx {
    Version  (uint32)
    Metadata (byte array)
    Fee      (uint256)
    Vin      (list of TxIn)
    Vout     (list of TxOut)
}
```

 *  Tx
    - Version:
      specifies the version of the transaction object
    - Metadata:
      specifies additional data required for transaction processing
    - Fee:
      specifies the total transaction fee
    - Vin:
      specifies the list of consumed UTXOs;
      **must** be nonempty
    - Vout:
      specifies the list of new UTXOs;
      **must** be nonempty

## Transaction Validation

## Transaction Hashing

Every transaction is referred to by its transaction hash (txhash).
This value uniquely specified a transaction.

## Versions

We list all version information here.

### Version 0

 *  Version Number: 0
 *  Metadata: **must** be empty
 *  Vin:  array of txin objects; **must** have at least 1 txin
 *  Vout: array of utxo objects; **must** have at least 1 utxo
