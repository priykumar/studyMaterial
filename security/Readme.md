Imagine you want to send a message to your friend:  
**HTTP** ─→ Sending a Postcard  
  - Anyone handling the postcard can read your message  
  - Someone could even alter your message along the way  

**HTTPS** ─→ Sending a Sealed Letter in a Locked Box  
  - Message is inside an envelope, inside a locked box and only your friend has the key to open it  
  - Even if someone intercepts it, they can't read the contents, If someone tries to tamper with it, you'll know  


## Why HTTPS
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


 ## HTTP vs HTTPS 
 <img width="2580" height="802" alt="image" src="https://github.com/user-attachments/assets/b1b63572-d28c-4d2e-8a60-da222c6d5e3c" />

 <img width="1470" height="873" alt="image" src="https://github.com/user-attachments/assets/00ec2032-cb08-4867-9ea6-680a2cc411cc" />

