---
title: Transaction Object Specification
author: chgorman
about: Create a transaction object specification
status: Draft
created: 2022-11-22
---

# Transaction Object Specification

We present the formal specification of the transaction object.

Note: all references to the hash function `Hash`
will refer to `Keccak256`;
it has a 32 byte (256 bit) digest.

**TODO: determine upper bound on size of DataStore**

## Introduction

The transaction is central to AliceNet:
it specifies a state transition.
A transaction consumes unspent transaction outputs (UTXOs)
and makes new UTXOs;
these will be defined more precisely below.
It is expected that multiple independent transactions
will be combined to specify the state transition
during each block on AliceNet.

We begin by discussing the [transaction object and associated subobjects](#objects).
From there, we look at a particular [Merkle Tree](#merkle-trees)
which will be used as we develop our [transaction hash](#transaction-hashes)
(`TxHash`; a way to compactly identify a transaction).

## Objects

Before discussing transactions, we will first talk about UTXOs:
unspent transaction outputs.
These UTXOs may be consumed in order to make additional UTXOs.

### DataStore

The DataStore object stores data (bytes) for a specific period of time
(in terms of epochs; 1 epoch is 1024 AliceNet blocks).
The total cost of creating a DataStore is determined by
the amount of data (in terms of bytes)
and length of storage (in terms of epochs);
there is also a minimum size cost
(for associated storage related to the DataStore)
and a per-epoch fee which we do not discuss here.

```
DataStore {
    DSLinker
    Signature (DataStoreSignature)
}
```

```
DSLinker {
    DSPreImage
    TxHash     (32 byte array)
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
    - TxHash:
      specifies the transaction hash
 *  DSPreImage
    - ChainID:
      specifies the ChainID for this object;
      **must** be nonzero
    - IssuedAt:
      specifies the epoch in which the DataStore was issued;
      **must** be nonzero
    - Index:
      specifies the index where the data is stored
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
    VSPreImage
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
        DataStore
        ValueStore
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
    TxInLinker
    Signature  (byte array)
}
```

```
TxInLinker {
    TxInPreImage
    TxHash       (32 byte array)
}
```

```
TxInPreImage {
    ChainID        (uint32)
    ConsumedTxIdx  (uint32)
    ConsumedTxHash (32 byte array)
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

### Deposit

A Deposit is a special type of TxIn:
it specifies value which has been transferred to AliceNet.
Deposits may only be consumed.
It is important to note that a
**deposit's validity must come from outside of AliceNet**;
this is a necessary part of the system.

Each deposit type will be contained within their own separate
subtrees of the `StateTrie`;
the location is specified by `key` (32 byte value)
and `number` (`uint256` value starting at 1).

### Tx

The transaction object is central to AliceNet.
The primary components of a transaction are
the list of consumed UTXOs (specified as an array of TxIn objects)
and the list of new UTXOs (specified as an array of TxOut objects).
It also contains the version number and metadata information
necessary for transaction validation.
A Fee object is included for completeness and is required
for validation;
it is used implicitly in the TxHash computation.

```
Tx {
    Version  (uint32)
    Metadata (byte array)
    Fee      (uint256)
    Vin      (TxIn array)
    Vout     (TxOut array)
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

For a transaction to be valid, it must
 *  have valid version number
 *  have valid metadata corresponding to specified version number
 *  have at least one TXIN
 *  have at least one UTXO
 *  have valid transaction indices on all new UTXOs
 *  have valid DataStore indices
 *  have valid inputs (unique)
 *  have matching input and output values
 *  have valid fees
 *  have valid TxHash

Before a transaction is processed,
all of its TxIn values must have valid signatures.

## Merkle Trees

We now discuss
[Merkle trees](https://en.wikipedia.org/wiki/Merkle_tree)
before we talk about transaction hashing.
Our discussion is based on the
[OpenZeppelin Merkle tree library](https://github.com/OpenZeppelin/merkle-tree).

We can compute the Merkle Tree root hash using
the following functions:

```
def LeafHash(data):
    return Hash(Hash(data))

def HairPair(a, b):
    if int(a) < int(b):
        return Hash(a || b)
    else:
        return Hash(b || a)

def ProcessData(Data):
    n = len(Data)
    Leaves = make(array, n)
    for (k = 0; k < n; k++):
        Leaves[k] = LeafHash(Data[k])
    return Leaves

def ProcessLeaves(Leaves):
    n = len(Leaves)
    Tree = make(array, 2*n - 1)
    for (k = 0; k < n; k++):
        Tree[2*n - 2 - k] = Leaves[k]
    for (k = n-1; k >= 0; k--):
        childL = Tree[2k+1]
        childR = Tree[2k+2]
        Tree[k] = HashPair(childL, childR)
    return Tree[0]

def ComputeRootHash(Data):
    Leaves = ProcessData(Data)
    root = ProcessLeaves(Leaves)
    return root

def VerifyMerkleProof(data, proof, root):
    tmp = LeafHash(data)
    for (k = 0; k < len(proof); k++):
        tmp = HashPair(tmp, proof[k])
    return tmp == root
```

This works for data of any length;
in particular, the length of the data is not required
to be a power-of-two.
Double-hashing the leaf nodes provides additional protection
by making a distinction between leaf nodes and interior nodes
within the Merkle Tree.

## Transaction Hashes

Every transaction is referred to by its transaction hash (TxHash).
This value is meant to uniquely specify a transaction
and should be determined deterministically from
the consumed inputs and created outputs.
Before discussing the precise method of computing transaction hashes,
we first discuss how to identify TXINs and UTXOs.

Broadly speaking, the transaction version and metadata
information will specify how the remaining information
(TXINs and UTXOs) will be hashed.

### `utxoID`s

We now focus on uniquely identifying transaction inputs and outputs;
that is, we must identify the TXINs and UTXOs.
To do this, we specify a `utxoID` for each object.

#### `utxoID`s for TxIns

We specify the `utxoID` for TXINs as

```
utxoID = Hash(ConsumedTxHash, ConsumedTxIdx)
```

In this case, `ConsumedTxIdx` is serialized as a big endian integer (`uint32`).

#### `utxoID`s for UTXOs

We specify the `utxoID` for UTXOs as

```
prehash = Hash(encode(utxo))
utxoID  = Hash(prehash, utxo.TxOutIdx)
```
In this case, `TxOutIdx` is serialized as a big endian integer (`uint32`).
Also, the `utxo` must first be serialized in a deterministic manner
(not specified here).

#### `utxoID`s for Deposits

Deposits are a special type of TxIn which must be defined separately.
We specify the `utxoID` for Deposits as

```
utxoID = Hash(key, number)
```

Here, `key` is the location of Deposit subtree within `StateTrie`
(32 byte value)
and `number` is a `uint256` value denoting which deposit
for that subtree;
`number` starts counting at 1.


### `TxHash`es

The transaction hash `TxHash` will uniquely identify a transaction.
It does this using its Version number, Metadata, TXINs,
and UTXOs.
The Version and Metadata information is processed first
and form one subtree;
after this, the Version and Metadata are then used
to process the TXINs and UTXOs.
The Version specifies valid Metadata, TXIN, and UTXO values.

We use the following algorithm:

```
def ComputeTxHash(version, metadata, txins, utxos)
    v  = LeafHash(00000000||version)
    md = LeafHash(00000001||metadata)
    tmp1 = HashPair(v, md)
    tmp2 = ComputeVinVoutHash(version, metadata, txins, utxos)
    txhash = HashPair(tmp1, tmp2)
    return txhash

def ComputeVinVoutHash(version, metadata, txins, utxos)
    if version == 0:
        if len(metadata) > 0:
            return "Error: invalid metadata"
        if len(txins) == 0:
            return "Error: invalid txins"
        if len(utxos) == 0:
            return "Error: invalid utxos"
        vinHash  = ComputeVinHash(txins)
        voutHash = ComputeVoutHash(utxos)
        tmp = HashPair(vinHash, voutHash)
        return tmp
    else:
        return "Error: invalid version"

def ComputeVinHash(txins)
    data = []
    for (k = 0; k < len(txins); k++)
        txin = txins[k]
        utxoID = makeUTXOID(txin)
        d = 00000002||utxoID
        data = append(data, d)
    vin = ComputeRootHash(data)
    return vin

def ComputeVoutHash(utxos)
    data = []
    for (k = 0; k < len(utxos); k++)
        utxo = utxos[k]
        utxoID = makeUTXOID(utxo)
        d = 00000002||utxoID
        data = append(data, d)
    vout = ComputeRootHash(data)
    return vout

def makeUTXOID(value)
    if IsValidTXIN(value):
        consumedTxHash = value.ConsumedTxHash
        consumedTxIdx  = value.ConsumedTxIdx
        return Hash(consumedTxHash||consumedTxIdx)
    else if IsValidUTXO(value):
        prehash  = Hash(encode(value))
        txOutIdx = value.TxOutIdx
        return Hash(prehash||txOutIdx)
    else if IsValidDeposit(value):
        key    = value.DepositKey
        number = value.DepositNumber
        return Hash(key||number)
    else:
        return "Error: Invalid object"
```

### Discussion about `utxoID`s and `TxHash`es

It is recognized that there is an extremely small (albeit nonzero)
probability that two different values may produce
the same hash output;
this is called a
[hash collision](https://en.wikipedia.org/wiki/Hash_collision).
We note that the work required to find a generic hash collision
for a 256 bit hash function is approximately $2^{128}$;
this is believed to be impractical.
That is, while it is **theoretically possible** for two different
consumed UTXOs to produce the same `utxoID`,
it is thought to be **impractical**.
The same comment applies to two different transactions
producing the same `TxHash`.
In any case, the `StateTrie` makes it impossible for there
to be two different UTXOs with the same `utxoID`,
as this would lead to an invalid state transition.

## Transaction Versions

We list all valid transaction version information here.

### Version 0
 *  Version Number: 0
 *  Metadata: **must** be empty
