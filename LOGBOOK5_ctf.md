## CTF buffer overflow

### Challenge 1
In the first challenge, the program reads 40 characters from stdin and then opens a file with the name given in a buffer.
It opens the file "mem.txt" and reads it.
At first glance, there is no way to control the file read, since it does not come from our input. However, since it reads a string of up to 40 characters and saves the information inside a buffer of only size 32, we can overflow the buffer and overwrite the file name.
In fact, 8 characters (40 - 32) actually overflow, so we can only overwrite the file name with "flag.txt".

So, we just need to put arbitrary first 32 chars and 8 chars corresponding to the file we want to open.
- e.g.: 01234567890123456789012345678901flag.txt

Notes:
Existe algum ficheiro que é aberto e lido pelo programa?
- opens mem.txt and reads from it
Existe alguma forma de controlar o ficheiro que é aberto?
- At first glance, there is no way to control it, since it does not come from our input.
Existe algum buffer-overflow? Se sim, o que é que podes fazer?
- However, as seen in the seed-labs guide, we can overflow the buffer and it will overflow and overwrite that file name. We can only overflow the file name, because only 40 characters are read from stdin. The buffer to overflow is 32 chars and the file name buffer is 8 chars.


### Challenge 2
In the second challenge, the program now reads up to 45 characters from stdin and then opens the same file. However, a new buffer was inserted before the file name, with 4 characters. Another change was the buffer for the file name having 9 characters instead of 8.
There is also a new check using the new buffer of 4 characters, which checks if the first 4 bytes of the input are equal to a certain value. If they are, the file is opened and read.

This does not completely mitigate the problem. Overflow is still possible and we can write over the file name buffer and this new buffer (45 characters and buffers have size 32 + 9 + 4 = 45)

We expected to do the same as before and just appending the \x24\x23\xfc\xfe to the end of the input to make the if clause pass.
However, testing the program, we noticed that the file name was starting with .txt and it was not flag.txt as expected. Therefore, we concluded that the first 4 bytes (flag) overflowed to the val buffer instead of the file name buffer.


So, we sent b"01234567890123456789012345678901\x24\x23\xfc\xfeflag.txt" through the python script and the if clause will pass (the int value of "val" will be 0xfefc2324 because of little-endianess). "flag.txt" will be written in the file name buffer in the same way as before, and the val buffer will have the correct value to pass the if clause. Therefore, the contents of flag.txt will be printed to stdout.



NOTES:
Que alterações foram feitas?
- Now max 45 chars are read, and a new buffer was inserted before the file name, with 4 chars. 2 '\0's were included in the initial file name buffer. Another buffer was included (value "deadbeef" in binary). Looking at the value in main.c, we concluded little endianess was used.
Mitigam na totalidade o problema?
- They do not completely mitigate the problem, since overflow is still possible.
É possível ultrapassar a mitigação usando uma técnica similar à que foi utilizada anteriormente?
- We expected to do the same as before and just appending the \x24\x23\xfc\xfe to the end of the input to make the if clause pass. However, when given that, the file name was starting with .txt and it was not flag.txt as expected. Therefore, we concluded that the first 4 bytes overflowed to the val buffer instead of the file name buffer.
So, we send b"01234567890123456789012345678901\x24\x23\xfc\xfeflag.txt" through the python script and the if clause will pass (the int value of "val" will be 0xfefc2324 because of little-endian). flag.txt will be written in the file name buffer in the same way as before, and the val buffer will have the correct value to pass the if clause.

