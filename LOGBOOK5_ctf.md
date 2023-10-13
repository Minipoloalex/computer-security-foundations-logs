## CTF buffer overflow

### Challenge 1
Existe algum ficheiro que é aberto e lido pelo programa?
- opens mem.txt and reads from it
Existe alguma forma de controlar o ficheiro que é aberto?
- At first glance, there is no way to control it, since it does not come from our input.
Existe algum buffer-overflow? Se sim, o que é que podes fazer?
- However, as seen in the seed-labs guide, we can overflow the buffer and it will overflow and overwrite that file name. We can only overflow the file name, because only 40 characters are read from stdin. The buffer to overflow is 32 chars and the file name buffer is 8 chars.


So, we just need to put arbitrary first 32 chars and 8 chars corresponding to the file we want to open.
- e.g.: 01234567890123456789012345678901flag.txt


### Challenge 2
Que alterações foram feitas?
- Now max 45 chars are read, and a new buffer was inserted before the file name, with 4 chars. 2 '\0's were included in the initial file name buffer. Another buffer was included (value "deadbeef" in binary). Looking at the value in main.c, we concluded little endianess was used.
Mitigam na totalidade o problema?
- They do not completely mitigate the problem, since overflow is still possible.
É possível ultrapassar a mitigação usando uma técnica similar à que foi utilizada anteriormente?
- We expected to do the same as before and just appending the \x24\x23\xfc\xfe to the end of the input to make the if clause pass. However, when given that, the file name was starting with .txt and it was not flag.txt as expected. Therefore, we concluded that the first 4 bytes overflowed to the val buffer instead of the file name buffer.
So, we send b"01234567890123456789012345678901\x24\x23\xfc\xfeflag.txt" through the python script and the if clause will pass (the int value of "val" will be 0xfefc2324 because of little-endian). flag.txt will be written in the file name buffer in the same way as before, and the val buffer will have the correct value to pass the if clause.

