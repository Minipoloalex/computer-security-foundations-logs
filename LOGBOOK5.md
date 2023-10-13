TODO: not complete
sudo sysctl -w kernel.randomize_va_space=0
to remove address space randomization - ASLR

dash prevents setuid programs... (change EUID to RUID)
use zsh

stackguard and nonexecutable stack avoided during compilation
compilation uses execstack option which allows code from stack to be executed
without this, it fails


Both versions (32 and 64 bits) worked, and executed the shellcode.

aaaaaaaa... no badfile levou a segfault


stack

buffer de baixo, cresce para cima

what stack looks like:
- caller func
- args
- ret
- fp
- buffer

shell code não pode estar no ret, basta estar algures
descobrir posição do ret
escrever no ret o endereço do nosso código


ret endereço absoluto


objetivo gdb: obter &return addr

no badfile pomos &buf + inicio shell code (saltamos para &buffer + inicio shell code), onde deveria estar o ret
analise no gdb e verificação de distancia de 108 entre fp e &buf, ou seja, 112 entre ret e &buffer
offset = 112
ret = &buffer
executing gdb worked and the shell was executed but not outside
the shell was executed only inside gdb

Decided to add a printf of the buffer address to the code to make it easier to see the difference between gdb and outside gdb.
We realised we had to do the first steps again because of the execution to execution differences, so we did.
Using the addr printed, in the ret (exploit.py), with offset of 112 as usual, it worked.



### Task 4
Noops vs ret_addr (&buffer)

Instead of using noops, we could put a lot of ret_addr's in the stack. Whatever the size of the buffer and the place for the fp/ret, there would be a ret_addr there, pointing to the shell code.
Changed the python code to this:
content = bytearray()
for i in range(0, 517, 4):
    content.extend((ret).to_bytes(L,byteorder='little'))
print(content)
Essentially making our desired ret_addr (to the shell code) be everywhere up in the stack.
This made it work after reading the correct buffer address, for L2.
