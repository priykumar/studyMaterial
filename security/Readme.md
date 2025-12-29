# 🔐 HTTP vs HTTPS: Security Explained
Imagine you want to send a message to your friend:  
**HTTP** ─→ Sending a Postcard  
  - Anyone handling the postcard can read your message  
  - Someone could even alter your message along the way  

**HTTPS** ─→ Sending a Sealed Letter in a Locked Box  
  - Message is inside an envelope, inside a locked box and only your friend has the key to open it  
  - Even if someone intercepts it, they can't read the contents, If someone tries to tamper with it, you'll know  


## 🛡️ Why HTTPS
Data flow over HTTP looks
```
┌─────────────┐                     ┌─────────────┐
│   Browser   │ ──── Plain Text ──→ │   Server    │
└─────────────┘ ←── Plain Text ──── └─────────────┘
       ↓                                  ↓
Anyone could read this            Anyone could read this
```
  
and this could lead to     
- **Eavesdropping**: The attacker silently reads everything!
```
    You                    Attacker                  Website
      │                        │                        │
      │────HTTP Request───────→│──────Forward──────────→│
      │   (username: john)     │  (can read it!)        │
      │   (password: secret)   │                        │
      │                        │                        │
      │←───HTTP Response───────│←─────Forward───────────│
      │   (account balance)    │  (can read it!)        │
```
       
- **Man-in-the-Middle**: The attacker can modify, inject, or steal data!
```
    You                    Attacker                  Website
      │                        │                         │
      │────Login Request──────→│                         │
      │                        │                         │
      │                        │───Fake Response────────→│
      │←───"Error, retry"──────│   (pretends to be you)  │
      │                        │                         │
      │────Retry (same pw)────→│                         │
```

With **HTTPS**, _Encryption_, _Authentication_ and _Integrity_ are guaranteed
- **Encryption**: Data is encrypted during transit and only intended recipient can decrypt it. Even if intercepted, data is unreadable
- **Authentication**: It ensures you're talking to the real website. It uses digital certificates
- **Integrity**: It ensures data arrives unchanged that means no tampering happened in the middle and if happens it gets detected. It uses cryptographic hashes


## ⚖️ HTTP vs HTTPS 
| Feature              | HTTP                | HTTPS               |
|----------------------|---------------------|---------------------|
| URL Scheme           | http://             | https://            |
| Default Port         | 80                  | 443                 |
| Browser Indicator    | ⚠️ Not Secure        | 🔒 Padlock          |
| Data Encryption      | ❌ None              | ✅ Encrypted        |
| Certificate Required | ❌ No                | ✅ Yes              |

 This is how a HTTP site looks like - shows a warning saying site is insecure
 <img width="2580" height="802" alt="image" src="https://github.com/user-attachments/assets/b1b63572-d28c-4d2e-8a60-da222c6d5e3c" />


This is how a HTTPs site looks like - shows certificate information
<img width="1470" height="880" alt="image" src="https://github.com/user-attachments/assets/07f90014-b325-4ca9-b826-94ddc12baa78" />

## 🔑 Understanding Keys and Certificates
### Private Key 🔐
- Like a secret key that only YOU have
- Used to **decrypt** messages encrypted by public key (Confidentiality)
- Used to sign documents to prove they're from you (Authenticity)
- If compromised, security is broken!

### Public Key 🔓
- Safe to share publicly
- Used by others to encrypt messages so that only private key holder can decrypt that
- Used to verify signatures created by the private key (Authenticity)
- The certificate contains the public key along with identity information.

### 🏛️ Certificate Authorities (CA)
>It is like a trusted government office that issues passports. Just as you trust a passport because it's issued by a government, you trust a certificate because it's issued by a CA.

- Verifies you own the domain
- Signs your certificate with their private key
- Makes their public key available to everyone(browser)
- Maintains records of all issued certificates
- Famous CAs: Digicert, Let's Encrypt, GlobalSign, IdenTrust

### 🌳 Root Certificates
>The top-level certificate that browsers inherently trust

```
          🌳 Root CA Certificate (Self-Signed)
         "I am DigiCert, trust me!"
                   │
                   │ Signs
                   ↓
         🌿 Intermediate CA Certificate
         "DigiCert says I'm trustworthy"
                   │
                   │ Signs
                   ↓
         🍃 End-Entity Certificate (Your Website)
         "Intermediate CA says I'm github.com"
```

#### Chain of Trust
```
Your Browser Has:
├─ Root CA Certificates (pre-installed)
│  ├─ DigiCert Root
│  └─ ~100+ other trusted roots
│
Website Sends:
├─ github.com certificate (signed by Intermediate CA)
├─ Intermediate CA certificate (signed by Root CA)
│
Browser Verifies:
├─ Is Intermediate CA signed by a Root CA I trust? ✓
├─ Is github.com certificate signed by Intermediate CA? ✓
├─ Chain complete! Trust established! ✓
```

Why Intermediate CAs?
- Root CA private key stays offline (extremely secure)
- If intermediate is compromised, only revoke that, not root
- Allows better organization and management

#### 📝 Certificate Signing Request (CSR)
>CSR is like a job application - you fill in your details and send it to the CA for approval

CSR Process:
```
Step 1: Generate Key Pair
┌──────────────────────────┐
│  You Generate:           │
│  🔐 Private Key (keep!)  │ ←─────┐
│  🔓 Public Key           │       │
└──────────────────────────┘       │
           ↓                       │ SAME Private Key
Step 2: Create CSR                 │ Throughout!
┌──────────────────────────┐       │ (Never sent to CA)
│  CSR Contains:           │       │ 
│  • Domain name           │       │ Private key is never
│  • Organization info     │       │ included in CSR
│  • 🔓 Public Key         │       │
│  (Private key NOT sent!) │       │
└──────────────────────────┘       │
           ↓                       │
Step 3: Send to CA                 │
┌──────────────────────────┐       │
│  CA Verifies:            │       │
│  • You own the domain    │       │
│  • Info is correct       │       │
│  (CA NEVER sees your     │       │
│   private key!)          │       │
└──────────────────────────┘       │
           ↓                       │
Step 4: CA Signs                   │
┌──────────────────────────┐       │
│  CA Creates Certificate: │       │
│  • Your public key       │       │
│  • Your info             │       │
│  • CA's signature 🔏     │       │
└──────────────────────────┘       │
           ↓                       │
Step 5: You Install on Server      │
┌──────────────────────────┐       │
│  Install on Server:      │       │
│  • 📜 Certificate        │       │
│  • 🔐 Private Key        │ ←─────┘
│    (The original one!)   │
└──────────────────────────┘
```

## ✍️ Digital Signatures
>Digital signature is like signing a document, but cryptographically - it proves who signed it and that the document hasn't been altered

## 📜 Digital Certificates    
>Think of a digital certificate as a digital passport or ID card for websites. Just like a passport proves your identity when traveling, a certificate proves a website's identity when you connect to it.

The Solution With Certificates
```
You: "Hey, are you really github.com?"
Server: "Yes, here's my certificate signed by DigiCert!"
You: "Let me verify this certificate..."
     ├─ Check: Is it signed by a trusted CA? ✓
     ├─ Check: Is the domain name correct? ✓
     ├─ Check: Has it expired? ✓
     └─ Check: Has it been revoked? ✓
You: "Certificate verified! I trust you're github.com"
```

### 📋 Certificate Fields Explained
| Field              | Value                | Significance               |
|----------------------|---------------------|---------------------|
|Version  |X.509 v3 |Certificate format version|
|Serial Number |0f:a0:7d:82:e5:4c:8f:9a:2b:3d:1e:4f:6a:8c |Unique identifier for this certificate, assigned by certificate authority(CA) |
|Signature Algorithm | sha256WithRSAEncryption |Algorithm used to sign the certificate|
|Issuer|↓ |CA details that issued/signed the certificate |
| |```CN (Common Name): DigiCert TLS RSA SHA256 2020 CA1```|Name of the CA|
| |```O  (Organization): DigiCert Inc```|Organization name |
| |```C  (Country): US```|Country code |
|Validity Period |↓ |Certificate validity time period|
| |Not Before |Mar 15 00:00:00 2024 GMT |Certificate is not valid before this date |
| |Not After |Mar 15 23:59:59 2025 GMT |Certificate is not valid after this date |
|Subject |↓ |Certificate Owner |
| |CN (Common Name): github.com|Domain name (most important!) |
| |O  (Organization): GitHub, Inc.|Company/organization name |
| |L  (Locality): San Francisco|City |
| |ST (State/Province): California|State/Province |
| |C  (Country): US|Country |
| |CN (Common Name): github.com| |
|Public Key |↓ |Public key used for encryption |
| |Algorithm: RSA Encryption | |
| |Public Key: 30:82:01:0a:02:82:01:01:00:b4...| |
|Certificate Signature |98:7a:3d:5f:2e:1b:8c:4a:9d:6f:3e:2a:7b:5c:1d:9e.... |CA's digital signature on this certificate |

### 🔑 **How CA digital certificate work?**  
CA takes all certificate data → Creates a hash (SHA-256) → Encrypts hash with CA's private key → This encrypted hash = digital signature

_Verification:_  
Browser decrypts signature with CA's public key → Gets the original hash → Browser calculates hash of certificate data → If hashes match → Certificate is authentic! ✓

## 🤝 How HTTPS Communication Works
When you visit https://github.com, here's what happens behind the scenes:
1. TCP Handshake          (Establish connection)     
2. TLS Handshake          (Secure the connection)     
3. Certificate Exchange   (Verify identity)          
4. Key Exchange           (Agree on encryption keys)  
5. Encrypted Communication (Send actual data)

#### 🔌 **TCP Handshake (Establish Connection)**
>Like picking up a phone and hearing the dial tone - line is open but conversation hasn't started yet.
```
    Client                                        Server
     │                                             │
     │────────── SYN ─────────────────────────────→│
     │←────────── SYN-ACK ──────────────────────── │
     │────────── ACK ─────────────────────────────→│
     │        TCP Connection Established           │
     │═══════════════════════════════════════════  │
```
#### 🔐 **TLS Handshake (Secure the Connection)**
>Negotiate how to communicate securely, which encryption algorithm to use and which TLS version

#### 📜 **Certificate Exchange (Verify Identity)**
```
You: "Prove you're really github.com"
Server: "Here's my digital certificate (ID card)"
You: "Let me check..."
     ✓ Certificate is for github.com
     ✓ Signed by DigiCert (trusted authority)
     ✓ Not expired
     ✓ Not revoked
You: "OK, I trust you!"
```

#### 🔑 **Key Exchange (Agree on Encryption Keys)**
Both create the same encryption keys WITHOUT sending them over the network!
```
You have:                   Server has:
├─ Your random number       ├─ Your random number
├─ Server's random number   ├─ Server's random number  
└─ Server's public key      └─ Server's private key

Both calculate → Same encryption key!
```

#### 💬 **Encrypted Communication (Send Actual Data)**
>Now you can safely exchange information.

## 🔒 Encryption
>Converting readable data (plaintext) into unreadable data (ciphertext)

Types of encryption
- Symmetric Encryption
- Asymmetric Encryption

### 🔐 Symmetric Encryption
>Same key for encryption and decryption

Disadvantage:
- How do you share the key securely?
- Key management: Many people = Many keys needed
- Anyone with the key can decrypt

Eg: AES, DES, 3DES

### 🔑 Asymmetric Encryption
>Two keys work together (key pair) Public key (encrypt) + 🔐 Private key (decrypt)
Eg: RSA, ECC, DSA
```
Bob generates a key pair:
├─ Public Key:  "ABC123..." (can share)
└─ Private Key: "XYZ789..." (keeps secret)

Bob publishes public key online
Anyone can see it, but it's useless without private key!

Alice wants to send Bob a secret:
┌─────────────────────────────────────────────┐
│  1. Alice gets Bob's PUBLIC key (ABC123)    │
│  2. Alice encrypts: "Hello" with ABC123     │
│  3. Result: "8f3c7a9e2b1d..."               │
│  4. Alice sends encrypted message           │
│                                             │
│  5. Bob receives: "8f3c7a9e2b1d..."         │
│  6. Bob decrypts with his PRIVATE key       │
│  7. Bob reads: "Hello"                      │
│                                             │
│  ✓ Only Bob can decrypt (has private key)   │
│  ✓ Even Alice can't decrypt her own msg!    │
└─────────────────────────────────────────────┘
```

```
Alice                                    Bob
  │                                        │
  │                     Bob's Public Key: "PUB_BOB"
  │←─────── Publish ────────────────────── │
  │                                        │
  │                     Bob's Private Key: "PRIV_BOB"
  │                     (Keeps secret!)    │
  │                                        │
  │ Plaintext: "Hello"                     │
  │      ↓                                 │
  │ Encrypt("Hello", "PUB_BOB")            │
  │      ↓                                 │
  │ Ciphertext: "8f3c7a9e"                 │
  │                                        │
  │────── "8f3c7a9e" ─────────────────────→│
  │                                        │
  │ (Alice can't decrypt                   │
  │  even her own message!)                │
  │                                        │
  │                  Decrypt("8f3c7a9e", "PRIV_BOB")
  │                           ↓            │
  │                  Plaintext: "Hello"    │
```

### 🧩 **How This Fits Into HTTPS**
>HTTPS uses both asymmetric for handshake, symmetric for data

- Certificates contain PUBLIC keys
- Servers keep PRIVATE keys secret
- TLS handshake uses asymmetric encryption
- Actual data transfer uses symmetric encryption
- Digital signatures verify certificate authenticity

## 🔄 Mutual TLS (mTLS) - Two-Way Authentication
>Client and server both proves their identities, unlike regular TLS where only server proves its identity

In TLS, client knows that he is talking to real server but there is no proof client is who he claims to be, because server has to be dependent on either of these
- Username/password
- API keys
- Session tokens
- OAuth tokens
to know who the client is. But all of these can be stolen, leaked, hijacked

In mTLS, 
- Client is who they claim to be
- Server is who they claim to be
- Strong authentication (certificates)
  * Both sides have certificates:  
    ├─ Client certificate (proves client identity)  
    └─ Server certificate (proves server identity)  

```
Regular TLS (HTTPS):
┌─────────────────────────────────────────────────┐
│  Client: "Prove you're the real server"         │
│  Server: "Here's my certificate" ✓              │
│  Client: "OK, I trust you"                      │
│  → Only server is authenticated                 │
└─────────────────────────────────────────────────┘

Mutual TLS (mTLS):
┌────────────────────────────────────────────────┐
│  Client: "Prove you're the real server"        │
│  Server: "Here's my certificate" ✓             │
│  Server: "Now YOU prove who you are"           │
│  Client: "Here's MY certificate" ✓             │
│  → Both client AND server authenticated        │
└────────────────────────────────────────────────┘
```

### 🔐 **How mTLS Works**
```
Client                                          Server
  │                                               │
  │─────── TCP Connection ───────────────────────→│
  │                                               │
  │─────── ClientHello ──────────────────────────→│
  │  • Supported ciphers                          │
  │                                               │
  │←────── ServerHello, Certificate ───────────── │
  │  • Server's certificate                       │
  │  • "Send me YOUR certificate too!" ←───────── │
  │                                               │
  ├─ Client verifies server cert                  │
  │                                               │
  │─────── Client Certificate ───────────────────→│
  │  • Client's certificate                       │
  │                                               │
  │                  Server verifies client cert ─┤
  │                                               │
  │←────── Server accepts/rejects ─────────────── │
  │                                               │
  │═══════ Mutual authentication complete ════════│
  │                                               │
  │   ←─────→ Encrypted communication ←─────→     │
```

### 💼 **mTLS Use Cases**
#### **Microservices Communication**
Services automatically authenticate each other as there is no api key to manage. Certificates are rotation automatically
```
┌────────────────────────────────────────────────┐
│                  Service Mesh                  │
├────────────────────────────────────────────────┤
│                                                │
│  Service A ←─mTLS─→ Service B                  │
│      │                  │                      │
│      └─────mTLS─────────┴───→ Service C        │
│                              │                 │
│                              └─mTLS→ Database  │
│                                                │
│  Every connection is mutually authenticated!   │
└────────────────────────────────────────────────┘
```

#### **API Authentication**
TLS API uses API key as `curl -H "X-API-Key: abc123..." ...`  
mTLS uses cert key as `curl --cert client.pem --key ... `
