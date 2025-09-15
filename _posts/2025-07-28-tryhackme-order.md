---
title: "TryHackMe: Order"
author: Dustin Merritt
categories: [TryHackMe]
tags: [XOR cipher, Repeating-ey, Encrypt,]
render_with_liquid: false
media_subpath: /images/tryhackme_order/
image:
  path: room_image.webp
---

The goal is to decrypt a message that was XOR encrypted using a repeateing key, and we know the message starts with `ORDER:`.

Here is the intercepted message:

`1c1c01041963730f31352a3a386e24356b3d32392b6f6b0d323c22243f6373`

`1a0d0c302d3b2b1a292a3a38282c2f222d2a112d282c31202d2d2e24352e60`

---

## Key Concepts
- XOR Encrytion
- XOR is a simple encrytion technique where each byte of plaintext is XOR'd with a byte from a key.
- If the result was XOR with the same key again, you get the original message back.
---
Example:
```
A ^ B = C
C ^ B = A   ← this lets us reverse it
```
---
## Step-by-Step Breakdown
### Step 1: Get the Encrypted Messasge
```python
ciphertext_hex = (
    "1c1c01041963730f31352a3a386e24356b3d32392b6f6b0d323c22243f6373"
    "1a0d0c302d3b2b1a292a3a38282c2f222d2a112d282c31202d2d2e24352e60"
)
```
This is the message, written in hexadecimal (base 16).
---
### Step 2: Convert Hex to Bytes
```python
ciphertext = bytes.fromhex(ciphertext_hex)
```
This turns the hex string into raw bytes -- the format the encryption actually operates on.
---
### Step 3: Use the Known Heading
```python
known_plaintext = b"ORDER:"
```
The messasges start with `"ORDER:"` which will be used to guess part of the encryption key.
---
### Step 4: Recover the Key Using XOR
```python
key = b''
for i in range(len(known_start)):
  key_byte = ciphertext[i] ^ known_start[i]
  key += bytes([key_byte])

# Show the recovered repeating key
From XORing the ciphertext with "ORDER:", the key was recovered.
```
ORDER: Attack at dawn. Target: THM{the_hackfinity_highschool}.
```

<style>
.center img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
.wrap pre {
  white-space: pre-wrap;
}
</style>
