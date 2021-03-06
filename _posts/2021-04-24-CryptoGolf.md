---
title: 'CryptoGolf'
---

### Challenge Description: 

```
Description: 
nc challs.m0lecon.it 11000
```


### **Foreword**
CryptoGolf was the first challenge of a series of very fun and *interesting* cryptography challenges that my teammates and I solved in [m0leconCTF World Quals](https://ctftime.org/event/1025) where [we](https://rgbsec.org) placed 5th globally in the Open Division, qualifying for the Grand Finals in Turin, Italy. 

<!--more-->
### **Introduction**
We are given `server.py`:

```python
import random
import hashlib
import sys

p = ''.join(random.choice('0123456789abcdef') for i in range(6))
print("Give me a string such that its sha256sum ends in {}.".format(p))
l = input().strip()
if hashlib.sha256(l.encode('ascii')).hexdigest()[-6:] != p:
    print("Wrong PoW")
    sys.exit(1)


from secrets import flag1, flag2, lim1, lim2

```

Note from writeup writers: limits were actually imported from a file that was not given. Their values were released as a hint, so we added their known specified values here: 

```python
lim1 = 128
lim2 = 45
import binascii
import string
import numpy

letters = string.ascii_lowercase + string.ascii_uppercase
secret = numpy.random.permutation(128)
chall = ''.join(random.choice(letters) for _ in range(96))

def pad(s):
    m = 192 - len(s)
    return s + hex(m % 16)[2:] * m

def apply_secret(c):
    r = bin(c)[2:].rjust(128,'0')
    return int(''.join([str(r[i]) for i in secret]), 2)

def encrypt(s):
    s = pad(s)
    to_encrypt = int(s, 16)
    for _ in range(9):
        x = apply_secret(to_encrypt >> 640)
        to_encrypt = (((x^to_encrypt) & 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF) << 640) | (to_encrypt >> 128)
    return hex(to_encrypt)[2:]

print("Encrypted challenge (hex):")
print(encrypt(binascii.hexlify(chall.encode()).decode()))
req = 0

for _ in range(1024):
    print("What do you want to do?")
    print("1. Encrypt something")
    print("2. Give me the decrypted challenge")
    ans = int(input())
    if ans == 1:
        print("Give me something to encrypt (hex):")
        s = input()
        if len(s) > 192 or not all(c in string.hexdigits for c in s):
            print("Nope1.")
            break
        print(encrypt(s))
        req = req + 1
    elif ans == 2:
        print("Give me the decrypted challenge:")
        c = input().strip()
        if c == chall:
            print("Good job! You did it in {} requests".format(req))
            if req <= lim1:
                print("Level 1 completed: {}".format(flag1))
                if req <= lim2:
                    print("Level 2 completed: {}".format(flag2))
                else:
                    print("Unfortunately, that's not enough for the second flag")
            else:
                print("Unfortunately, that's not enough for the first flag")

        else:
            print("Nope.")
        break
```

Our task is essentially the following: perform at most `lim1 – 1 `encryptions and send the decrypted challenge, so we need to uncover the secret in 128 – 1 == 127 queries. ```apply_secret(c)``` essentially does the following: the $i^{th}$ bit of the output (where the MSB’s index is 0, etc) is ```r[secret[i]]```, where `r` is the binary representation of the input block. The first key observation is the following: `secret` is a **permutation** of the numbers in range 0 .. 127, so every number in that range appears on it exactly *once and only once in a unique* position. The concept is to check what happens to a single 1 bit where all the rest of the bits are 0. 

One shall claim that t is a [bijection](https://en.wikipedia.org/wiki/Bijection#:~:text=In%20mathematics%2C%20a%20bijection%2C%20bijective,element%20of%20the%20first%20set) from the set `{2^i | 0 <= i <= 127}` to itself. Why? Well, based on our analysis of the function apply_secret, if we input to the function $2^i$, the $j^{th}$ bit (again, where MSB is 0) will be zero except when `secret[j] = 127 - i` (again, not `i` due to endianess ($2^{127}$ is every bit 0 except the MSB)). Secret is a PERMUTATION that spans 0 .. 127, therefore, there exists only a single unique `j` such that `secret[j] = 127 - i`. 


Based on the above analysis, t($2^{127-i}$) == $2^{127-secret[i]}$ (since MSB is $2^{127}$, etc); note again that secret is a permutation of the index range, therefore this is a bijection, and in fact, a very useful one.


### **Analysis**
Now, one shall continue by performing an analysis of the encrypt function. Based on the shift arithmetic one can assume that we are working with 6 blocks, each containing 128 bits. Let us denote the blocks with `A`, `B`, `C`, `D`, `E`, `F`, where `A` is the most significant block.



Note that to_encrypt is the integer form of the hex message
The encryption loop:

```python
for _ in range(9):
    x = apply_secret(to_encrypt >> 640)
    to_encrypt = (((x^to_encrypt) & 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF) << 640) | (to_encrypt >> 128)
```

the following process occurs:

1. set `x` to `t(A)` (the most significant block)
2. move the last block, `F` (least significant), to be the first block and shift everything one block to the right, and `xor` the new last block with `x`

essentially, this transforms :

```
A   B   C   D   E   F --> t(A)^F  A  B  C  D  E
```

One good technique when solving crypto challenges is to fix some controllable variables with “nice” values. one might intuitively try to set all the blocks except `A` to be 0, to avoid `xor`ing a lot of blocks to simply the process a single bit. After 6 iterations, the process applied on `A`, `0`, `0`, `0`, `0`, `0` would result in the following block sequence:

```
t(t(t(t(t(t(A)))))), t(t(t(t(t(A))))), t(t(t(t(A)))), 
1                        2                  3    

t(t(t(A))), t(t(A)), t(A)
 4          5         6
```

We know that after 3 more iterations, the last 3 blocks (least significant) will be:

```
t(t(t(t(t(t(A)))))), t(t(t(t(t(A))))), t(t(t(t(A))))
```

Substitute `v = t(t(t(t(A))))`, then the last 3 blocks are essentially `t(t(v)), t(v), v`. Now, since `t` is a bijection from the set `{2^i| 0 <= i <= 127}` to itself, then `t(t(t(...))` iterated for any amount of times is a bijection from the set `{2^i | 0 <= i <= 127}` to itself as well. (because it receives values that span the index range and outputs them, the output set is the same as the input set).



So we know that if we set `A` to $2^i$, we get a power of 2 in `v`, and a UNIQUE one, since based on the bijection t(t(t...( $2^i$ ))) `!=` t(t(t...( $2^{j}$ ))) unless `j=i`. The beautiful observation: We know `t(v)` and `v`, and since v can span the entire bit range we can recover the indexes in the secret array. That would take precisely 128 queries, for every power of 2, but with the decryption challenge that would result in a total of 129 queries, where the limit is 128. However, after the 127th encryption, only one `t` value will be missing, so we can simply check what is missing and since t is a bijection what value of t we haven't seen yet, and set that to the value of the missing `v`. Therefore, the entire process would take 127 + 1 == 128 queries, which is exactly `lim1`!

### **Solving**
The algorithm:

```python
for i in 0 .. 126
	out_blocks = encrypt(2^i)
	find j such that 2^j == out_blocks[5] # last block
	find k such that 2^k == out_blocks[4] # one before last block
	set secret[127-k] = 127-j
```

Fill the `t` value you haven’t seen yet in the missing spot in secret array. We have uncovered the secret! The algorithm of decryption using the secret is documented in the solution code we programmed.

Solution :

```python
import binascii
from hashlib import sha256
from itertools import product
from math import log2
from pwn import remote


# solve the Proof-Of-Work
def find_tail(tail: str, tail_len=6):
    for L in range(1, 10):
        for s in product(b"0123456789abcdefghijklmnopqrstuvwxyz", repeat=L):
            if sha256(bytearray(s)).hexdigest()[-tail_len:] == tail:
                return bytearray(s).decode()
    raise ValueError("unable to find solution to POW")


# split a hex string into blocks
def blocks(s):
    ret = []
    for block in range(0, len(s), 32):
        ret.append(bin(int(s[block:block + 32], 16))[2:].rjust(128, '0'))
    return ret


# This apply_secret has the same functionality as the source on the server side
# We don't really need to know how this works - we can abstract it away
# If it matters, it shuffles the bits of a number with a permutation
def apply_secret(c, secret):
    r = bin(c)[2:].rjust(128, '0')
    return int(''.join([str(r[i]) for i in secret]), 2)


# Given the secret, reverse the encryption
# Encrypt splits the number into 6 blocks of 128 bits each
# It runs 9 times
# Each time, it takes the top block, applies the secret, and xors it with the bottom block
# Then each block is shifted down
#   A     B     C     D     E     F
# f(A)^F  A     B     C     D     E
# We known A, B, C, D, and E. Since we have the secret, we also know f(A)
# Then to get F, we can xor the top block with f(A)
# Then shift the blocks back
def decrypt(n: int, secret: list):
    to_decrypt = n
    for _ in range(9):
        top_block = to_decrypt >> 640  # f(A) ^ F
        second_block = (to_decrypt >> 512) & 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF  # A
        # 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF is 2^128 - 1
        # this is the same as mod 2^128

        to_decrypt = (to_decrypt << 128) % (1 << 768)  # shift the blocks up, and take out the top one
        x = apply_secret(second_block, secret)  # get f(A)
        to_decrypt |= x ^ top_block  # fill in the bottom block with F

    dec = hex(to_decrypt)
    print(dec)
    dec = dec[2:]  # remove "0x"

    # unhexlify only works if its input has an even length
    if len(dec) & 1:
        dec = '0' + dec

    return binascii.unhexlify(dec)


# test decrypt with locally generated examples
def test():
    enc = 0xcb97861c616e58a69126260cf4c64e74f0fa21838edf2f3a047e38a95328823b60b636e2c9502d88623080299f85d4e572464edabb8863389937df66ac2262b4b8fc0bb93c51684e8c0e8fd82ac04f06c15a31fc1d9fc71cac0f2ad15dcf39b9
    secret = [48, 14, 40, 108, 94, 0, 79, 25, 59, 58, 104, 88, 1, 3, 73, 7, 90, 53, 106, 24, 92, 50, 103, 17, 64, 111,
              18, 101, 12, 65, 83, 96, 13, 43, 4, 127, 119, 67, 23, 77, 2, 87, 123, 38, 118, 97, 113, 70, 74, 121, 98,
              16, 71, 60, 41, 22, 33, 80, 102, 100, 39, 116, 10, 115, 49, 27, 125, 124, 76, 109, 8, 37, 112, 84, 63, 31,
              117, 56, 114, 69, 36, 46, 11, 45, 52, 86, 6, 82, 122, 32, 66, 93, 91, 55, 72, 51, 62, 105, 68, 5, 75, 29,
              20, 95, 54, 126, 47, 28, 19, 57, 44, 21, 107, 30, 110, 61, 120, 85, 35, 78, 34, 9, 81, 26, 42, 15, 89, 99]

    print(decrypt(enc, secret))


def main():
    # test()
    r = remote("challs.m0lecon.it", 11000)

    resp = r.recvline().decode().strip()
    tail = resp.split()[-1].strip('.')
    print(resp)

    pow_ans = find_tail(tail)

    r.sendline(pow_ans)
    print(">", pow_ans)

    print(r.recvline().decode().strip())

    chall = r.recvline().decode().strip()
    print(chall)

    secret = [-1 for _ in range(128)]
    for i in range(127):
        print(r.recvuntil("Give me the decrypted challenge\n").decode().strip())
        query = str(hex(1 << i))[2:].rjust(32 * 6, '0')
        r.sendline('1')
        print('> 1')
        print(r.recvuntil("Give me something to encrypt (hex):\n").decode().strip())
        r.sendline(query)
        print('>', query)

        result = r.recvline().decode().strip()
        print(result)
        value = round(log2(int(result[-32:], 16)))
        index = round(log2(int(result[-64:-32], 16)))
        secret[127 - index] = 127 - value

    print("secret:", secret)

    missing = 0
    for i in range(128):
        if i not in secret:
            missing = i
            break
    for i in range(128):
        if secret[i] == -1:
            secret[i] = missing
            break

    dec = decrypt(int(chall, 16), secret)

    r.sendline('2')
    print('> 2')

    print(r.recvuntil("Give me the decrypted challenge:\n").decode().strip())
    r.sendline(dec)
    print('>', dec)

    print(r.recvall(4).decode())


if __name__ == '__main__':
    main()
```

> Flag: **ptm{sometimes_encryption_can_be_as_bad_as_decryption_ecdb556edcffd}**

Thank you to team *pwnthem0le* from Politecnico di Torino for hosting such a fun event. Also special shoutout to my teammates Lior and Stanley for their work on this problem. We look forward to competing in future CTFs and conferences :)



