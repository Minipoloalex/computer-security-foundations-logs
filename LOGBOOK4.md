# Environment variabls and SetUID (seed labs)
This is the logbook for the Environment_Variable_and_SetUID guide from seedlabs.

### Task1
We tested several commands to print environmental variables (printenv, printenv X, env), to set them using export (e.g. export MY_ENV_VARIABLE=HELLO_WORLD), and to unset them (unset X)
printenv
printenv PWD

unset USER
export HELLO=WORLD


### Task 2
In this task, we concluded that, using fork(), the parent's environment variables are inherited by the child. In fact, testing the program given and printing the child and then the parent's environment variables, and saving the output to files, we checked that the files had no difference.


### Task 3
The command execve() replaces our program with the new program. The first program we are asked to run has the third parameter of execve as NULL, meaning NULL environment variables. As a consequence, the command "env" executed has no output.
After changing the third parameter to be the environment variables of the original program, these will now also be present in the new program. Therefore, the output from "env" is a list of those environment variables, common to both programs.


### Task 4
As referenced in the guide, system does not directly execute the command like execve(). It asks the shell to execute the command. It uses execl() to execute the shell, which passes the environment variables to the shell. So, the environment variables in a system() call are maintained, which is shown by the output of the program given in the guide.


### Task 5
In this task, we learned the effects of a setuid program.
Changing the program to be owned by root and making it a setuid program ("chmod `4`755"), we make it run with the owner's privilege (root), independently of the user that is running the program.
Changing the environment variables, we are affecting the program output from the outside. Changing "PATH" and creating our own variable "MY_VARIABLE" worked as expected and it appeared in the program. However, "LD_LIBRARY_PATH" did not appear in the output.
In the guide, it is explained that a new child process is created, but as seen here, not all variables are passed to the child.


### Task 6
Using system() in setuid programs is dangerous, because of the way environment variables affect the program. Being a setuid program, it runs with its owner's privileges, so any user can change the environment variables to try to modify the program's behaviour.
In this example, we add a directory to the beginning of PATH. By running system("ls"), the first directory to search will be the one at the beginning of PATH. Therefore, by introducing our code in that directory, we'll be executing that code instead of the usual "ls" code.

- We tested running simple code printing "Hello World"
- Using different shells dash or zsh does not change the output for this simple program. However, zsh would not remove LD_LIBRARY_PATH as in dash (zsh does not have that countermeasure, that would protect avoid certain attacks)


### Task 7
LD_PRELOAD specifies user libraries to be loaded before all others. So, if we build some code in a directory and change LD_PRELOAD to point to it, we might be able to run our code in another program, possibly with higher privileges.

In this task, we built our program and compiled it to a library. If that code is runned by a priviliged program, we could do some priviliged actions, such as read a file we don't have access to. This is what we used to solve the CTF.

> - Describe behaviour of program in different scenarios

> - LD_PRELOAD - vê a nossa linked library que define sleep
> - ao correr o programa seguinte, já executa a definição nossa de sleep e não a do SO.

./my_program
I am not sleeping

sudo chown root myprogram
I am not sleeping

sudo chmod 4755 myprogram
sleep(1) esperou 1 segundo, fez o normal

changed program to be a user1 program and it still overrides the sleep function and prints "I am not sleeping"


1 - não dá sleep
2 - dá sleep
3 - não dá sleep



