---
title: Ethereum Signed Transactions
author: Troy Salem, Dan Horbatt, Bret Gregersen
discussion: alicenet/issues/465
team: Backend
category: Development
status: Review
related: alicenet/issues/463
created: 2022-11-07
---

## Introduction

#### Summary   
This allows transactions to be signed by the Ethereum personal_sign method.

#### Context
MetaMask and other popular Ethereum wallets allows signatures to be generated for arbitrary data using the personal_sign method. Using this method we can sign transactions used by AliceNet.

#### Goals
Allow for transactions to be signed by MetaMask and submitted to AliceNet.

#### Non Goals
Do not enable personal_sign signatures within our application.

#### Assumptions
Most users are comfortable with MetaMask and other Ethereum wallets and as such onboarding them with MetaMask is a good idea.

## Specification

#### Overview
The personal_sign method prefixes `"\x19Ethereum Signed Message:\n" + len(message)` to the message and then signs it. This type of signed SECP256k1 message is not supported by AliceNet, so we need to implement a check for this type of signature in the secp256k1 verification logic.

#### Data
N/A

#### Logic
This should be implemented in the `crypto` package in the file `secp.go` line 65 as try/catch logic incase the traditional signature verification fails, we can check for this type of signature.

```go
	pubk, err := eth.SigToPub(hsh, sig)
	if err != nil {
		pubk, err = tryEthPersonalSign(msg, sig)
		if err != nil {
			return nil, err
		}
	}
```

The logic to check for this type of signature is as follows in psuedocode:

```go
func tryEthPersonalSign(msg []byte, sig []byte) (*ecdsa.PublicKey, error) {
	ethMsg := []byte("\x19Ethereum Signed Message:\n" + fmt.Sprint(len(msg)))
	ethMsg = append(ethMsg, msg...)
	hsh := keccak256(ethMsg)
	pubk, err := eth.SigToPub(hsh, sig)
	if err != nil {
		return nil, err
	}
	return pubk, nil
}
```

#### Testing
Test the logic by signing a message in the same manner as personal_sign and then verifying it with the new logic.
```go
func TestEthSigValidate(t *testing.T) {
    s := new(Secp256k1Signer)
	privk := make([]byte, 32)
	s.SetPrivk(privk)
	msg := []byte("A message to sign")
    ethMsg := []byte("\x19Ethereum Signed Message:\n" + fmt.Sprint(len(msg)))
    ethMsg = append(ethMsg, msg...)
    digestHash := Hasher(ethMsg)
	signature := s.Sign(digestHash)
    v := new(Secp256k1Validator)
    pubk, err := v.Validate(ethMsg, signature)
	if err != nil {
		t.Fatal(err)
	}
}

```

#### Presentation
N/A

#### Security / Risks
When using the personal_sign method, the user is signing arbitrary data. This means that the user is signing a transaction that they have not seen. This is a security risk, but it is mitigated by the fact that the user is signing the transaction with their private key. This means that the user is aware of the transaction they are signing and is aware of the consequences of signing it.

## Further Considerations

#### Timeline
Should be done before bridge UI is implemented. 

#### Prioritization
Low / Medium priority.

#### Alternative Solutions
Creating our own browser plugin wallet is a possibility, but it would be a lot of work and would require a lot of testing.

#### Dependencies
N/A

#### Open Questions
Should this feature be supported by AliceNet in the long term? 

Should we support other types of signatures (ETH Typed Data)?




