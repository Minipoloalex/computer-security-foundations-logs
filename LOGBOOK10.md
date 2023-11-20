<!--

Setup
dcbuild
dcup


Task 1: Frequency Analysis

ytn -> the

tr 'ytn' 'THE' < out.txt > out2.txt


THvT -> THAT
'vup' -> AND

tr 'vup' 'AND' < ciphertext.txt > out.txt


mN -> IN
tr 'm' 'I' < out2.txt > out.txt

iANDINr
a lot of words that end with INr -> ING

tr 'r' 'G' < out.txt > out2.txt

DIfIDED gETlEEN
gETlEEN -> between
g -> B
l -> W
f -> V

tr 'glf' 'BWV' < out2.txt > out.txt

THIq -> THIS
Iq -> IS

tr 'q' 'S' < out.txt > out2.txt

THhEE -> THREE
bAh BETWEEN -> FAR BETWEEN

tr 'hb' 'RF' < out2.txt > out.txt

tr 'izx' 'LUO' < out.txt > out2.txt

FILcS
HISTORIaAL

tr 'aec' 'CPM' < out2.txt > out.txt


-->

```
tr 'ytn' 'THE' < out.txt > out2.txt
tr 'vup' 'AND' < ciphertext.txt > out.txt
tr 'm' 'I' < out2.txt > out.txt
tr 'r' 'G' < out.txt > out2.txt
tr 'glf' 'BWV' < out2.txt > out.txt
tr 'q' 'S' < out.txt > out2.txt
tr 'hb' 'RF' < out2.txt > out.txt
tr 'izx' 'LUO' < out.txt > out2.txt
tr 'aec' 'CPM' < out2.txt > out.txt
tr 'duv' 'YNA' < out2.txt > out.txt
tr 'p' 'D' < out.txt > out2.txt
tr 'ks' 'XK' < out.txt > out2.txt
tr 'jo' 'QJ' < out2.txt > out.txt
tr 'w' 'Z' < out.txt > out2.txt
```

### Task 2
<!--



openssl enc -ciphertype -e -in plain.txt -out cipher.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708

openssl enc -aes-256-ecb -e -in plain.txt -out cipher.bin -K 00112233445566778889aabbccddeeff
openssl enc -aes-192-cbc -e -in p1.bmp -out p2.bmp -K 00112233445566778889aabbccddeeff -iv 0102030405060708


ECB -> input = -> output predictable
CBC -> input = -> output not predictable (can't see the pattern in the image)

openssl enc -aes-128-ecb -e -in p1.bmp -out p2.bmp -K 00112233445566778889aabbccddeeff
head -c 54 p1.bmp > header
tail -c +55 p2.bmp > body
cat header body > new.bmp
eog new.bmp

openssl enc -aes-128-cbc -e -in p1.bmp -out p2.bmp -K 00112233445566778889aabbccddeeff -iv 0102030405060708
head -c 54 p1.bmp > header
tail -c +55 p2.bmp > body
cat header body > new.bmp
eog new.bmp
-->
