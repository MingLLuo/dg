---
{"dg-publish":true,"permalink":"/cryptography/opaque/","noteIcon":"","created":"2024-07-13T12:15:18.574+08:00","updated":"2024-07-17T14:28:01.538+08:00"}
---

> Following content all based in OPAQUE Draft, and explained in chinese.

The name OPAQUE is a homonym of O-PAKE where O is for Oblivious.

- OPAQUE是一种增强的/非对称的PAKE协议，支持client和sercer的相互验证，而不依赖于PKI（客户端注册期间除外），且拥有抵御预计算攻击的安全性。此外，还提供前向安全与隐藏明文口令的能力，包括口令注册的阶段
- 其中有三个主要的功能构件，分别是 an oblivious pseudorandom function (OPRF), a key recovery mechanism, and an authenticated key exchange (AKE) protocol
- OPAQUE被分为两个阶段，注册和AKE
	- 客户端向服务器注册其密码，并在服务器上存储用于恢复身份验证凭据(authentication credentials)的信息
	- 客户端使用其密码来恢复这些凭据，然后将它们用作 AKE 协议的输入。此阶段有额外的机制来防止主动攻击者与服务器交互以猜测或确认通过第一阶段注册的客户端。
- OPAQUE依赖如下的加密协议与原语
	- [[Cryptography/Oblivious Pseudorandom Function (OPRF)\|Oblivious Pseudorandom Function (OPRF)]]
	- [[Cryptography/Key Derivation Function (KDF)\|Key Derivation Function (KDF)]]
	- [[Cryptography/Message Authentication Code (MAC)\|Message Authentication Code (MAC)]]
	- Cryptographic Hash Function
	- [[Cryptography/Key Stretching Function (KSF)\|Key Stretching Function (KSF)]]

### 协议初览
#### 初始化
- Server为AKE选择密钥对(`server_private_key` and `server_public_key`)，设置OPRF的种子(oprf_seed, Nh bytes)
- Server可以为不同的Client分配不同的seed

#### 注册
- OPAQUE的注册环节是唯一一个需要有confidentiality和integrity保证的环节，可以是基于物理通道、Out-of-band management、基于PKI等
- Client需要输入其凭据（包含密码和uid），Server需要输入其参数（如私钥和其他信息）
- 在此阶段，Client会导出`export_key`
	- The client output of this stage is a single value `export_key` that the client may use for application-specific purposes, e.g., as a symmetric key used to encrypt additional information for storage on the server. The server does not have access to this `export_key`.
- Server会输出Client注册所产生的记录，根据需要与其他客户端的注册信息中存储在凭证文件内
	- 在此交互结束时，服务器将 `record` 对象以及关联的 `credential_identifier` 和 `client_identity` （如果不同）存储为每个客户端的凭证文件。请注意，服务器设置阶段的值 `oprf_seed` 和 `server_private_key` 也必须保留。 `oprf_seed` 值应该用于所有客户端；其中应用程序可以选择避免在客户端之间使用全局 `oprf_seed` 值，而是为每个客户端唯一地采样 OPRF 密钥。 `server_private_key` 对于每个客户端来说可能是唯一的。
```cpp
    creds                                   parameters
      |                                         |
      v                                         v
    Client                                    Server
    ------------------------------------------------
                registration request
             ------------------------->
                registration response
             <-------------------------
                      record
             ------------------------->
   ------------------------------------------------
      |                                         |
      v                                         v
  export_key                                 record

struct { 
 uint8 blinded_message[Noe];
} RegistrationRequest;
// blinded_message: A serialized OPRF group element

struct { 
 uint8 evaluated_message[Noe];
 uint8 server_public_key[Npk];
} RegistrationResponse;
// evaluated_message: A serialized OPRF group element.
// server_public_key: The server's encoded public key that will be used for the online AKE stage.

struct { 
 uint8 client_public_key[Npk]; 
 uint8 masking_key[Nh]; 
 Envelope envelope; 
} RegistrationRecord;
// client_public_key: The client's encoded public key, corresponding to the private key `client_private_key`.
// masking_key: An encryption key used by the server with the sole purpose of defending against client enumeration attacks.
// envelope: The client's `Envelope` structure.

  Client                                         Server
 ------------------------------------------------------
 (request, blind) = CreateRegistrationRequest(password)

                        request
              ------------------------->

 response = CreateRegistrationResponse(request,
                                       server_public_key,
                                       credential_identifier,
                                       oprf_seed)

                        response
              <-------------------------

 (record, export_key) = FinalizeRegistrationRequest(response,
                                                    server_identity,
                                                    client_identity)

                        record
              ------------------------->



FinalizeRegistrationRequest

Input:
- password, an opaque byte string containing the client's password.
- blind, an OPRF scalar value.
- response, a RegistrationResponse structure.
- server_identity, the optional encoded server identity.
- client_identity, the optional encoded client identity.

Output:
- record, a RegistrationRecord structure.
- export_key, an additional client key.

Exceptions:
- DeserializeError, when OPRF element deserialization fails.

def FinalizeRegistrationRequest(password, blind, response, server_identity, client_identity):
  evaluated_element = DeserializeElement(response.evaluated_message)
  oprf_output = Finalize(password, blind, evaluated_element)

  stretched_oprf_output = Stretch(oprf_output)
  randomized_password = Extract("", concat(oprf_output, stretched_oprf_output))

  (envelope, client_public_key, masking_key, export_key) =
    Store(randomized_password, response.server_public_key,
          server_identity, client_identity)
  Create RegistrationRecord record with (client_public_key, masking_key, envelope)
  return (record, export_key)
```

##### Envelope
```c
struct { 
uint8 nonce[Nn];
uint8 auth_tag[Nm]; 
} Envelope;
// nonce: A randomly-sampled nonce of length `Nn`, used to protect this `Envelope`.
// auth_tag: An authentication tag protecting the contents of the envelope, covering the envelope nonce and `CleartextCredentials`.
```
- 客户端在注册时使用下面定义的函数 `Store` 创建一个 `Envelope` 。请注意，此函数中的 `DeriveDiffieHellmanKeyPair` 失败的概率可以忽略不计。如果发生这种情况，服务器应重新运行该函数，对新的 `envelope_nonce` 进行采样，直至完成
```python
Store

Input:
- randomized_password, a randomized password.
- server_public_key, the encoded server public key for
  the AKE protocol.
- server_identity, the optional encoded server identity.
- client_identity, the optional encoded client identity.

Output:
- envelope, the client's Envelope structure.
- client_public_key, the client's AKE public key.
- masking_key, an encryption key used by the server with the sole purpose
  of defending against client enumeration attacks.
- export_key, an additional client key.

def Store(randomized_password, server_public_key, server_identity, client_identity):
  envelope_nonce = random(Nn)
  masking_key = Expand(randomized_password, "MaskingKey", Nh)
  auth_key = Expand(randomized_password, concat(envelope_nonce, "AuthKey"), Nh)
  export_key = Expand(randomized_password, concat(envelope_nonce, "ExportKey"), Nh)
  seed = Expand(randomized_password, concat(envelope_nonce, "PrivateKey"), Nseed)
  (_, client_public_key) = DeriveDiffieHellmanKeyPair(seed)

  cleartext_credentials =
    CreateCleartextCredentials(server_public_key, client_public_key,
                               server_identity, client_identity)
  auth_tag = MAC(auth_key, concat(envelope_nonce, cleartext_credentials))

  Create Envelope envelope with (envelope_nonce, auth_tag)
  return (envelope, client_public_key, masking_key, export_key)
```
- 客户端在登录期间使用下面定义的 `Recover` 函数恢复其 `Envelope`
	- 在引发 `EnvelopeRecoveryError` 的情况下，必须删除该函数中所有先前计算的中间值。
```python
Recover

Input:
- randomized_password, a randomized password.
- server_public_key, the encoded server public key for the AKE protocol.
- envelope, the client's Envelope structure.
- server_identity, the optional encoded server identity.
- client_identity, the optional encoded client identity.

Output:
- client_private_key, the encoded client private key for the AKE protocol.
- cleartext_credentials, a CleartextCredentials structure.
- export_key, an additional client key.

Exceptions:
- EnvelopeRecoveryError, the envelope fails to be recovered.

def Recover(randomized_password, server_public_key, envelope,
            server_identity, client_identity):
  auth_key = Expand(randomized_password, concat(envelope.nonce, "AuthKey"), Nh)
  export_key = Expand(randomized_password, concat(envelope.nonce, "ExportKey"), Nh)
  seed = Expand(randomized_password, concat(envelope.nonce, "PrivateKey"), Nseed)
  (client_private_key, client_public_key) = DeriveDiffieHellmanKeyPair(seed)

  cleartext_credentials = CreateCleartextCredentials(server_public_key,
                      client_public_key, server_identity, client_identity)
  expected_tag = MAC(auth_key, concat(envelope.nonce, cleartext_credentials))
  If !ct_equal(envelope.auth_tag, expected_tag)
    raise EnvelopeRecoveryError
  return (client_private_key, cleartext_credentials, export_key)

```

#### 在线认证密钥交换
- 客户端获取先前在服务器中注册的凭据，使用密码恢复私钥材料，然后将它们用作 AKE 协议的输入
- 与注册阶段一样，客户端输入其凭证，包括密码和用户标识符，服务器输入其参数和与客户端对应的凭证文件记录。客户端输出两个值，一个 `export_key` （与注册中的值匹配）和一个 `session_key` ，后者是主要 AKE 输出。服务器输出与客户端的值匹配的单个值 `session_key` 
- 我们会将 OPRF 和 AKE 的结果并行发送给server

```
    creds                             (parameters, record)
      |                                         |
      v                                         v
    Client                                    Server
    ------------------------------------------------
                   AKE message 1
             ------------------------->
                   AKE message 2
             <-------------------------
                   AKE message 3
             ------------------------->
   ------------------------------------------------
      |                                         |
      v                                         v
(export_key, session_key)                  session_key


  Client                                         Server
 ------------------------------------------------------
  ke1 = GenerateKE1(password)

                         ke1
              ------------------------->

  ke2 = GenerateKE2(server_identity, server_private_key,
                    server_public_key, record,
                    credential_identifier, oprf_seed, ke1)

                         ke2
              <-------------------------

    (ke3,
    session_key,
    export_key) = GenerateKE3(client_identity,
                               server_identity, ke2)

                         ke3
              ------------------------->

                       session_key = ServerFinish(ke3)
```
- 在协议执行的过程中，双方可能有需要临时存储的字段，在协议完成后应该擦除
##### AKE消息定义
```c
struct {
  uint8 client_nonce[Nn];
  uint8 client_public_keyshare[Npk];
} AuthRequest;
// client_nonce: A fresh randomly generated nonce of length `Nn`.
// client_public_keyshare: A serialized client ephemeral public key of fixed size `Npk`.

struct {
  CredentialRequest credential_request;
  AuthRequest auth_request;
} KE1;
// credential_request: A `CredentialRequest` structure.
// auth_request: An `AuthRequest` structure.

struct {
  uint8 server_nonce[Nn];
  uint8 server_public_keyshare[Npk];
  uint8 server_mac[Nm];
} AuthResponse;
// server_nonce: A fresh randomly generated nonce of length `Nn`.
// server_public_keyshare: A server ephemeral public key of fixed size `Npk`, where `Npk` depends on the corresponding prime order group.
// server_mac: An authentication tag computed over the handshake transcript computed using `Km2`, defined below.
struct {
  CredentialResponse credential_response;
  AuthResponse auth_response;
} KE2;
// credential_response: A `CredentialResponse` structure.
// auth_response: An `AuthResponse` structure.

struct {
  uint8 client_mac[Nm];
} KE3;
// client_mac: An authentication tag computed over the handshake transcript of fixed size `Nm`, computed using `Km2`, defined below.
```

##### AKE功能
```python
GenerateKE1

State:
- state, a ClientState structure.

Input:
- password, an opaque byte string containing the client's password.

Output:
- ke1, a KE1 message structure.

def GenerateKE1(password):
  request, blind = CreateCredentialRequest(password)
  state.password = password
  state.blind = blind
  ke1 = AuthClientStart(request)
  return ke1

---
GenerateKE2

State:
- state, a ServerState structure.

Input:
- server_identity, the optional encoded server identity, which is set to
  server_public_key if not specified.
- server_private_key, the server's private key.
- server_public_key, the server's public key.
- record, the client's RegistrationRecord structure.
- credential_identifier, an identifier that uniquely represents the credential.
- oprf_seed, the server-side seed of Nh bytes used to generate an oprf_key.
- ke1, a KE1 message structure.
- client_identity, the optional encoded client identity, which is set to
  client_public_key if not specified.

Output:
- ke2, a KE2 structure.

def GenerateKE2(server_identity, server_private_key, server_public_key,
               record, credential_identifier, oprf_seed, ke1, client_identity):
  credential_response = CreateCredentialResponse(ke1.credential_request, server_public_key, record,
    credential_identifier, oprf_seed)
  cleartext_credentials = CreateCleartextCredentials(server_public_key,
                      record.client_public_key, server_identity, client_identity)
  auth_response = AuthServerRespond(cleartext_credentials, server_private_key,
                      record.client_public_key, ke1, credential_response)
  Create KE2 ke2 with (credential_response, auth_response)
  return ke2
  
---
GenerateKE3

State:
- state, a ClientState structure.

Input:
- client_identity, the optional encoded client identity, which is set
  to client_public_key if not specified.
- server_identity, the optional encoded server identity, which is set
  to server_public_key if not specified.
- ke2, a KE2 message structure.

Output:
- ke3, a KE3 message structure.
- session_key, the session's shared secret.
- export_key, an additional client key.

def GenerateKE3(client_identity, server_identity, ke2):
  (client_private_key, cleartext_credentials, export_key) =
    RecoverCredentials(state.password, state.blind, ke2.credential_response,
                       server_identity, client_identity)
  (ke3, session_key) =
    AuthClientFinalize(cleartext_credentials, client_private_key, ke2)
  return (ke3, session_key, export_key)
---
ServerFinish

State:
- state, a ServerState structure.

Input:
- ke3, a KE3 structure.

Output:
- session_key, the shared session secret if and only if ke3 is valid.

def ServerFinish(ke3):
  return AuthServerFinalize(ke3)

```

##### 凭据检索
```c
struct {
  uint8 blinded_message[Noe];
} CredentialRequest;

// blinded_message: A serialized OPRF group element.

struct {
  uint8 evaluated_message[Noe];
  uint8 masking_nonce[Nn];
  uint8 masked_response[Npk + Nn + Nm];
} CredentialResponse;

// evaluated_message: A serialized OPRF group element.
// masking_nonce: A nonce used for the confidentiality of the `masked_response` field.
// masked_response: An encrypted form of the server's public key and the client's `Envelope` structure.
```
- 客户端使用 `CreateCredentialRequest` 来启动凭证检索过程，并生成 `CredentialRequest` 消息和相应的 OPRF 状态。与 `CreateRegistrationRequest` 一样，此函数可能会失败并出现 `InvalidInputError` 错误，概率可以忽略不计。不应该让这种情况发生，因为在提供相同的密码输入时注册（通过 `CreateRegistrationRequest` ）将会失败
```python
CreateCredentialRequest

Input:
- password, an opaque byte string containing the client's password.

Output:
- request, a CredentialRequest structure.
- blind, an OPRF scalar value.

Exceptions:
- InvalidInputError, when Blind fails

def CreateCredentialRequest(password):
  (blind, blinded_element) = Blind(password)
  blinded_message = SerializeElement(blinded_element)
  Create CredentialRequest request with blinded_message
  return (request, blind)
```
- 服务器使用 `CreateCredentialResponse` 函数处理客户端的 `CredentialRequest` 消息并完成凭证检索过程，生成 `CredentialResponse`。构建 `CredentialResponse` 对象有两种情况需要处理：要么客户端的记录存在（对应于正确注册的客户端），要么从未创建过（对应于未注册的客户端身份，可能是枚举攻击尝试的结果）。
	- 对于具有相应标识符 `credential_identifier` 的现有记录，服务器调用以下函数来生成 `CredentialResponse`
```python
CreateCredentialResponse

Input:
- request, a CredentialRequest structure.
- server_public_key, the public key of the server.
- record, an instance of RegistrationRecord which is the server's
  output from registration.
- credential_identifier, an identifier that uniquely represents the credential.
- oprf_seed, the server-side seed of Nh bytes used to generate an oprf_key.

Output:
- response, a CredentialResponse structure.

Exceptions:
- DeserializeError, when OPRF element deserialization fails.

def CreateCredentialResponse(request, server_public_key, record,
                             credential_identifier, oprf_seed):
  seed = Expand(oprf_seed, concat(credential_identifier, "OprfKey"), Nok)
  (oprf_key, _) = DeriveKeyPair(seed, "OPAQUE-DeriveKeyPair")

  blinded_element = DeserializeElement(request.blinded_message)
  evaluated_element = BlindEvaluate(oprf_key, blinded_element)
  evaluated_message = SerializeElement(evaluated_element)

  masking_nonce = random(Nn)
  credential_response_pad = Expand(record.masking_key,
                                   concat(masking_nonce, "CredentialResponsePad"),
                                   Npk + Nn + Nm)
  masked_response = xor(credential_response_pad,
                        concat(server_public_key, record.envelope))
  Create CredentialResponse response with (evaluated_message, masking_nonce, masked_response)
  return response
```
- 如果record不存在并且需要防止客户端枚举，则服务器必须响应凭证请求，伪造记录的存在。服务器应该使用伪造的客户端记录参数调用 `CreateCredentialResponse` 函数，该参数配置为：
	- `record.client_public_key` is set to a randomly generated public key of length `Npk
	- `record.masking_key` is set to a random byte string of length `Nh`
	- `record.envelope` is set to the byte string consisting only of zeros of length `Nn + Nm
- 建议创建的 fake record（例如作为应用程序的第一个用户记录）与合法客户记录一起存储。服务器能与合法record能有相近的检索时间
- 对于无法猜测与 `credential_identifier` 对应的客户端的注册密码的对手来说，这两种情况输出的响应都是无法区分的
 - Client使用 `RecoverCredentials` 函数处理服务器的 `CredentialResponse` 消息并生成客户端的私钥、服务器公钥和 `export_key`
```python
RecoverCredentials

Input:
- password, an opaque byte string containing the client's password.
- blind, an OPRF scalar value.
- response, a CredentialResponse structure.
- server_identity, The optional encoded server identity.
- client_identity, The encoded client identity.

Output:
- client_private_key, the encoded client private key for the AKE protocol.
- cleartext_credentials, a CleartextCredentials structure.
- export_key, an additional client key.

Exceptions:
- DeserializeError, when OPRF element deserialization fails.

def RecoverCredentials(password, blind, response,
                       server_identity, client_identity):
  evaluated_element = DeserializeElement(response.evaluated_message)

  oprf_output = Finalize(password, blind, evaluated_element)
  stretched_oprf_output = Stretch(oprf_output)
  randomized_password = Extract("", concat(oprf_output, stretched_oprf_output))

  masking_key = Expand(randomized_password, "MaskingKey", Nh)
  credential_response_pad = Expand(masking_key,
                                   concat(response.masking_nonce, 
                                   "CredentialResponsePad"),
                                   Npk + Nn + Nm)
  concat(server_public_key, envelope) = xor(credential_response_pad,
                                              response.masked_response)
  (client_private_key, cleartext_credentials, export_key) =
    Recover(randomized_password, server_public_key, envelope,
            server_identity, client_identity)

  return (client_private_key, cleartext_credentials, export_key)

```
#### 客户端凭证存储和密钥恢复
- OPAQUE 使用名为 `Envelope` 的结构来管理客户端凭据。客户端在注册时创建其 `Envelope` 并将其发送到服务器进行存储。每次登录时，服务器都会将此 `Envelope` 发送到客户端，以便客户端可以恢复其密钥材料以在 AKE 中使用。
	- 密钥材料可能会被关联为id，如果一方没有提供identifier，将默认设置为其公钥
		- client_private_key: The encoded client private key for the AKE protocol.
		- client_public_key: The encoded client public key for the AKE protocol.
		- server_public_key: The encoded server public key for the AKE protocol.
		- client_identity: The client identity. This is an application-specific value, e.g., an e-mail address or an account name. If not specified, it defaults to the client's public key.
		- server_identity: The server identity. This is typically a domain name, e.g., example.com. If not specified, it defaults to the server's public key. See [Section 10.3](https://cfrg.github.io/draft-irtf-cfrg-opaque/draft-irtf-cfrg-opaque.html#identities) for information about this identity.
```python
struct { 
 uint8 server_public_key[Npk]; 
 uint8 server_identity<1..2^16-1>; 
 uint8 client_identity<1..2^16-1>; 
} CleartextCredentials;


CreateCleartextCredentials
Input:
- server_public_key, the encoded server public key for the AKE protocol.
- client_public_key, the encoded client public key for the AKE protocol.
- server_identity, the optional encoded server identity.
- client_identity, the optional encoded client identity.

Output:
- cleartext_credentials, a CleartextCredentials structure.

def CreateCleartextCredentials(server_public_key, client_public_key,
                               server_identity, client_identity):
  # Set identities as public keys if no application-layer identity is provided
  if server_identity == nil
    server_identity = server_public_key
  if client_identity == nil
    client_identity = client_public_key

  Create CleartextCredentials cleartext_credentials with
    (server_public_key, server_identity, client_identity)
  return cleartext_credentials
```
##### 密钥恢复
[[Cryptography/OPAQUE#Envelope\|OPAQUE#Envelope]]

### 实例化 3DH
- 客户端 AKE 状态 `ClientAkeState` 具有以下字段：
	- client_secret: An opaque byte string of length `Nsk`.
	- ke1: A value of type `KE1`.
- 服务器 AKE 状态 `ServerAkeState` 具有以下字段：
	- expected_client_mac: An opaque byte string of length `Nm`.
	- session_key: An opaque byte string of length `Nx`.

#### 3DH Key Exchange Function
- DeriveDiffieHellmanKeyPair(seed)：从输入 `seed` 中确定性地派生私有和公共 Diffie-Hellman 密钥对。私钥的类型取决于实现，而公钥的类型是 `Npk` 字节的字节串
- DiffieHellman(k, B)：在私有输入 `k` 和公共输入 `B` 之间执行 Diffie-Hellman 操作的函数。该函数的输出是一个唯一的、固定长度的字节字符串

#### AKE 密钥调度
- The OPAQUE-3DH key derivation procedures make use of the functions below, re-purposed from TLS 1.3 [[RFC8446](https://cfrg.github.io/draft-irtf-cfrg-opaque/draft-irtf-cfrg-opaque.html#RFC8446)].
```c
Expand-Label(Secret, Label, Context, Length) = 
  Expand(Secret, CustomLabel, Length)

struct {
  uint16 length = Length;
  opaque label<8..255> = "OPAQUE-" + Label;
  uint8 context<0..255> = Context;
} CustomLabel;

Derive-Secret(Secret, Label, Transcript-Hash) =
    Expand-Label(Secret, Label, Transcript-Hash, Nx)
```
- OPAQUE-3DH 可以选择在转录中包含特定于应用程序的共享 `context` 信息，例如配置参数或特定于应用程序的信息，例如“appXYZ-v1.2.3”
- The OPAQUE-3DH key schedule requires a preamble, which is computed as follows
```python
Preamble

Parameters:
- context, optional shared context information.

Input:
- client_identity, the optional encoded client identity, which is set
  to client_public_key if not specified.
- ke1, a KE1 message structure.
- server_identity, the optional encoded server identity, which is set
  to server_public_key if not specified.
- credential_response, the corresponding field on the KE2 structure.
- server_nonce, the corresponding field on the AuthResponse structure.
- server_public_keyshare, the corresponding field on the AuthResponse structure.

Output:
- preamble, the protocol transcript with identities and messages.

def Preamble(client_identity, ke1, server_identity, credential_response,
             server_nonce, server_public_keyshare):
  preamble = concat("OPAQUEv1-",
                     I2OSP(len(context), 2), context,
                     I2OSP(len(client_identity), 2), client_identity,
                     ke1,
                     I2OSP(len(server_identity), 2), server_identity,
                     credential_response,
                     server_nonce,
                     server_public_keyshare)
  return preamble
```
- 在密钥交换协议期间导出的 OPAQUE-3DH 共享秘密是使用以下辅助函数计算的
```python
DeriveKeys

Input:
- ikm, input key material.
- preamble, the protocol transcript with identities and messages.

Output:
- Km2, a MAC authentication key.
- Km3, a MAC authentication key.
- session_key, the shared session secret.

def DeriveKeys(ikm, preamble):
  prk = Extract("", ikm)
  handshake_secret = Derive-Secret(prk, "HandshakeSecret", Hash(preamble))
  session_key = Derive-Secret(prk, "SessionKey", Hash(preamble))
  Km2 = Derive-Secret(handshake_secret, "ServerMAC", "")
  Km3 = Derive-Secret(handshake_secret, "ClientMAC", "")
  return (Km2, Km3, session_key)
```
- Client使用 `AuthClientStart` 函数创建 `KE1` 
```python
AuthClientStart

Parameters:
- Nn, the nonce length.

State:
- state, a ClientAkeState structure.

Input:
- credential_request, a CredentialRequest structure.

Output:
- ke1, a KE1 structure.

def AuthClientStart(credential_request):
  client_nonce = random(Nn)
  client_keyshare_seed = random(Nseed)
  (client_secret, client_public_keyshare) = DeriveDiffieHellmanKeyPair(client_keyshare_seed)
  Create AuthRequest auth_request with (client_nonce, client_public_keyshare)
  Create KE1 ke1 with (credential_request, auth_request)
  state.client_secret = client_secret
  state.ke1 = ke1
  return ke1

```
-  Client使用 `AuthClientFinalize` 函数创建 `KE3` 消息，并使用Server的 `KE2` 消息输出 `session_key` 以及恢复凭证
```python
AuthClientFinalize

State:
- state, a ClientAkeState structure.

Input:
- cleartext_credentials, a CleartextCredentials structure.
- client_private_key, the client's private key.
- ke2, a KE2 message structure.

Output:
- ke3, a KE3 structure.
- session_key, the shared session secret.

Exceptions:
- ServerAuthenticationError, the handshake fails.

def AuthClientFinalize(cleartext_credentials, client_private_key, ke2):
  dh1 = DiffieHellman(state.client_secret, ke2.auth_response.server_public_keyshare)
  dh2 = DiffieHellman(state.client_secret, cleartext_credentials.server_public_key)
  dh3 = DiffieHellman(client_private_key, ke2.auth_response.server_public_keyshare)
  ikm = concat(dh1, dh2, dh3)

  preamble = Preamble(cleartext_credentials.client_identity,
                      state.ke1,
                      cleartext_credentials.server_identity,
                      ke2.credential_response,
                      ke2.auth_response.server_nonce,
                      ke2.auth_response.server_public_keyshare)
  Km2, Km3, session_key = DeriveKeys(ikm, preamble)
  expected_server_mac = MAC(Km2, Hash(preamble))
  if !ct_equal(ke2.auth_response.server_mac, expected_server_mac),
    raise ServerAuthenticationError
  client_mac = MAC(Km3, Hash(concat(preamble, expected_server_mac)))
  Create KE3 ke3 with client_mac
  return (ke3, session_key)
```

- `AuthServerRespond` 函数用于服务器处理客户端的 `KE1` 消息和公共凭证信息以创建 `KE2` 消息
```python
AuthServerRespond

Parameters:
- Nn, the nonce length.

State:
- state, a ServerAkeState structure.

Input:
- cleartext_credentials, a CleartextCredentials structure.
- server_private_key, the server's private key.
- client_public_key, the client's public key.
- ke1, a KE1 message structure.

Output:
- auth_response, an AuthResponse structure.

def AuthServerRespond(cleartext_credentials, server_private_key, client_public_key, ke1, credential_response):
  server_nonce = random(Nn)
  server_keyshare_seed = random(Nseed)
  (server_private_keyshare, server_public_keyshare) = DeriveDiffieHellmanKeyPair(server_keyshare_seed)
  preamble = Preamble(cleartext_credentials.client_identity,
                      ke1,
                      cleartext_credentials.server_identity,
                      credential_response,
                      server_nonce,
                      server_public_keyshare)

  dh1 = DiffieHellman(server_private_keyshare, ke1.auth_request.client_public_keyshare)
  dh2 = DiffieHellman(server_private_key, ke1.auth_request.client_public_keyshare)
  dh3 = DiffieHellman(server_private_keyshare, client_public_key)
  ikm = concat(dh1, dh2, dh3)

  Km2, Km3, session_key = DeriveKeys(ikm, preamble)
  server_mac = MAC(Km2, Hash(preamble))

  state.expected_client_mac = MAC(Km3, Hash(concat(preamble, server_mac)))
  state.session_key = session_key
  Create AuthResponse auth_response with (server_nonce, server_public_keyshare, server_mac)
  return auth_response
```
- `AuthServerFinalize` 函数用于服务器处理客户端的 `KE3` 消息并输出最终的 `session_key`
```python
AuthServerFinalize

State:
- state, a ServerAkeState structure.

Input:
- ke3, a KE3 structure.

Output:
- session_key, the shared session secret if and only if ke3 is valid.

Exceptions:
- ClientAuthenticationError, the handshake fails.

def AuthServerFinalize(ke3):
  if !ct_equal(ke3.client_mac, state.expected_client_mac):
    raise ClientAuthenticationError
  return state.session_key
```
### 参考

- [The OPAQUE Augmented PAKE Protocol](https://github.com/cfrg/draft-irtf-cfrg-opaque)

