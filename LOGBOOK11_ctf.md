# RSA

## CTF Week 11

### Understanding `challenge.py` 

```python
import os
import sys
from rsa_utils import getParams
from binascii import hexlify, unhexlify

FLAG_FILE = '/flags/flag.txt'

def enc(x, e, n):
    int_x = int.from_bytes(x, "little")
    y = pow(int_x,e,n)
    return hexlify(y.to_bytes(256, 'little'))

def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'little')

with open(FLAG_FILE, 'r') as fd:
	un_flag = fd.read()

(p, q, n, phi, e, d) = getParams()

print("Public parameters -- \ne: ", e, "\nn: ", n)
print("ciphertext:", hexlify(enc(un_flag.encode(), e, n)).decode())
sys.stdout.flush()
```

The `challenge.py` file is a simple RSA encryption/decryption program. It takes in a flag file, reads it, and encrypts it using the RSA algorithm. The encrypted flag is then printed out, along with the public parameters `e` and `n`.

### The exploit

We have the following information about the set of primes `p` and `q`:
- `p` is a prime number close to `2^512`
- `q` is a prime number close to `2^513`

```python
from binascii import hexlify, unhexlify
import miller_rabin as mr

def gcdExtended(a, b): 
    if a==0:
        return b,0,1   
    gcd,x1,y1=gcdExtended(b%a,a)
    x=y1-(b//a)*x1
    y=x1
    return gcd,x,y

def enc(x, e, n):
    int_x = int.from_bytes(x, "little")
    y = pow(int_x,e,n)
    return hexlify(y.to_bytes(256, 'little'))

def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'little')

possible_p = []
search_range = 200000

for i in range(2**512, 2**512 + search_range):
    if mr.miller_rabin(i):
        possible_p.append(i)

for i in range(2**512 - search_range, 2**512):
    if mr.miller_rabin(i):
        possible_p.append(i)

p = q = -1
print(f"Number of candidates: {len(possible_p)}\n")

for i in possible_p:
    if n % i == 0:
        p = i
        q = n // i
        break

assert p != -1 and q != -1
assert p * q == n

print("Found p and q")
print(f"p = {p}")
print(f"q = {q}\n")

phi = (p - 1)*(q - 1)
d = gcdExtended(e, phi)[1]
print(f"e * d % phi = {e * d % phi}\n")

print("Decoded cyphertext:")
flag = dec(unhexlify(cyphertext.encode()), d, n).decode()
print(flag)
```

The exploit is pretty simple. We first generate a list of possible primes `p` in the range `2^512` to `2^512 + 200000` and `2^512 - 200000` to `2^512`. The miller-rabin primality test is used to obtain the list of primes. We then check if `n` is divisible by any of the primes in the list. If it is, we have found `p` and subsequently `q`. We then calculate `phi`, and use the extended euclidean algorithm to find `d`. Finally, we decrypt the cyphertext using `d` and `n`.

Note that the chosen range of primes is arbitrary. We can choose a larger range if we want to, but it will take longer to run. We can also choose a smaller range, but there is a chance that we will not find `p`.