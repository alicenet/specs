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

### UTXO

UTXOs are Unspent Transaction Outputs.
A UTXO may be either a ValueStore or a DataStore.

#### DataStore

The DataStore object stores data (bytes) for a specific period of time
(in terms of epochs; 1 epoch is 1024 AliceNet blocks).
The total cost of creating a DataStore is determined by
the amount of data (in terms of bytes)
and length of storage (in terms of epochs);
there is also a minimum size cost
(for associated storage related to the DataStore)
and a per-epoch fee.

The DataStore object contains the following fields:

 *  ChainID (uint32):
    specifies the ChainID the UTXO references; **must** be nonzero
 *  IssuedAt (uint32):
    specifies the epoch originally issued; **must** be nonzero
 *  Index (32-byte value):
 *  RawData (bytes):
    specifies the raw data of the DataStore;
    **must** be nonzero number of bytes (as well as
    **to be determined** upper bound on size)
 *  TxOutIdx (uint32):
    specifies the index of the UTXO within the transaction
 *  Deposit (uint256):
    total cost of DataStore
 *  Owner:
    specifies the account owner
 *  Fee: 
    specifies the per-epoch fee

#### ValueStore

The ValueStore object stores value (in terms of ALCB [BToken]).

The ValueStore object contains the following fields:

 *  ChainID (uint32):
    specifies the ChainID the UTXO references; **must** be nonzero
 *  TxOutIdx (uint32):
    specifies the index of the UTXO within the transaction
 *  Value (uint256):
    total value of ValueStore object; **must** be nonzero
 *  Owner:
    specifies the account owner
 *  Fee: 
    specifies the ValueStore fee

## Transaction Object

The transaction object will have the following subobjects:

 *  Version number (uint32): specify which transaction version is being used
 *  Metadata (bytes array): additional data for processing; may be empty
 *  Vin (list of txin objects): a list of consumed utxos
 *  Vout (list of utxo objects): a list of newly created utxos

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
