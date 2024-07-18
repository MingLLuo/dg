---
{"dg-publish":true,"permalink":"/cryptography/oblivious-pseudorandom-function-oprf/","noteIcon":"","created":"2024-07-16T19:27:53.045+08:00","updated":"2024-07-18T22:14:36.647+08:00"}
---

#Cryptography #OPRF #PRF 
*An Oblivious Pseudorandom Function (OPRF)* is a two-party protocol between client and server for computing a PRF, where the PRF key is held by the server and the input to the function is provided by the client. The client does not learn anything about the PRF other than the obtained output and the server learns nothing about the client's input or the function output.

The following OPRF client APIs are used:
- `Blind(element)`: Create and output (`blind`, `blinded_element`), consisting of a blinded representation of input `element`, denoted `blinded_element`, along with a value to revert the blinding process, denoted `blind`.
- `Finalize(element, blind, evaluated_element)`: Finalize the OPRF evaluation using input `element`, random inverter `blind`, and evaluation output `evaluated_element`, yielding output `oprf_output`.

Moreover, the following OPRF server APIs are used:
- `BlindEvaluate(k, blinded_element)`: Evaluate blinded input element `blinded_element` using input key `k`, yielding output element `evaluated_element`.
- `DeriveKeyPair(seed, info)`: Create and output (`sk`, `pk`), consisting of a private and public key derived deterministically from a `seed` and `info` parameter.

Finally, the following shared APIs and parameters:
- `SerializeElement(element)`: Map input `element` to a fixed-length byte array.
- `DeserializeElement(buf)`: Attempt to map input byte array `buf` to an OPRF group element. This function can raise a DeserializeError upon failure.
- Noe: The size of a serialized OPRF group element output from SerializeElement.
- Nok: The size of an OPRF private key as output from DeriveKeyPair.

### Simple Routine using OPRF Protocol
- G: a prime-order group implementing the API described [Section 2.1](https://www.rfc-editor.org/rfc/rfc9497#pog)
- contextString: a `PublicInput` domain separation tag constructed during context setup
- skS and pkS: a Scalar and Element representing the private and public keys configured for the client and server

1. The OPRF protocol begins with the **client** blinding its input, as described by the `Blind` function below. Note that this function can fail with an `InvalidInputError` error for certain inputs that map to the group identity element. Dealing with this failure is an application-specific decision.
```python
Input:
  PrivateInput input

Output:
  Scalar blind
  Element blindedElement

Parameters:
  Group G

Errors: InvalidInputError

def Blind(input):
  blind = G.RandomScalar()
  inputElement = G.HashToGroup(input)
  if inputElement == G.Identity():
    raise InvalidInputError
  blindedElement = blind * inputElement

  return blind, blindedElement
```

2. **Clients** store `blind` locally and send `blindedElement` to the **server** for evaluation. Upon receipt, **servers** process `blindedElement` using the `BlindEvaluate` function described below.
```python
Input:
  Scalar skS
  Element blindedElement

Output:
  Element evaluatedElement

def BlindEvaluate(skS, blindedElement):
  evaluatedElement = skS * blindedElement
  return evaluatedElement
```

3. **Servers** send the output `evaluatedElement` to **clients** for processing. Recall that **servers** may process multiple **client** inputs by applying the `BlindEvaluate` function to each `blindedElement` received and returning an array with the corresponding `evaluatedElement` values. Upon receipt of `evaluatedElement`, **clients** process it to complete the OPRF evaluation with the `Finalize` function described below.
```python
Input:
  PrivateInput input
  Scalar blind
  Element evaluatedElement

Output:
  opaque output[Nh]

Parameters:
  Group G

def Finalize(input, blind, evaluatedElement):
  N = G.ScalarInverse(blind) * evaluatedElement
  unblindedElement = G.SerializeElement(N)

  hashInput = I2OSP(len(input), 2) || input ||
              I2OSP(len(unblindedElement), 2) || unblindedElement ||
              "Finalize"
  return Hash(hashInput)
```

4. An entity that knows both the private key and the input can compute the PRF result using the following `Evaluate` function.
```python
Input:
  Scalar skS
  PrivateInput input

Output:
  opaque output[Nh]

Parameters:
  Group G

Errors: InvalidInputError

def Evaluate(skS, input):
  inputElement = G.HashToGroup(input)
  if inputElement == G.Identity():
    raise InvalidInputError
  evaluatedElement = skS * inputElement
  issuedElement = G.SerializeElement(evaluatedElement)

  hashInput = I2OSP(len(input), 2) || input ||
              I2OSP(len(issuedElement), 2) || issuedElement ||
              "Finalize"
  return Hash(hashInput)
```

#### DH-OPRF
![Oblivious Pseudorandom Function (OPRF)_DH_OPRF.png](/img/user/Cryptography/attachments/Oblivious%20Pseudorandom%20Function%20(OPRF)_DH_OPRF.png)
### Reference
- [RFC 9497 Oblivious Pseudorandom Functions (OPRFs) Using Prime-Order Groups](https://www.rfc-editor.org/rfc/rfc9497)