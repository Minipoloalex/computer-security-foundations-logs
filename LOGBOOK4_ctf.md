## CTF Linux Environment

This is the logbook for the Linux Environment CTF.

### Start
Starting out, we tried to get as much information from the /home/flag_reader directory. We read all possible files using cat, such as main.c, my_script.sh, admin_note.txt. Also, after checking the permissions, we tested running ./my_script.sh and ./reader. We concluded reader was main.c compiled.

### Affecting program execution/output
By reading my_script.sh, [TODO] if we put some variables in "env", they will be read as environment variables for the my_script.sh program.
Furthermore, reading admin_note.txt led us to search for the /tmp folder and the permissions to write in it.
So, we simply built a library in that folder, compiling it, as seen here.


Afterwards, we made the environment variable "LD_PRELOAD" be inserted into the program by doing:
echo "LD_PRELOAD=..." > env, which would be read into the program.

After this, we ran my_script.sh, effectively changing the program, by changing the access function's code but could not read the file, since the permissions we were the running the program were still basic.

### Concluding and reading /flags/flag.txt
After reading the CTF's statement and the tips for the tasks, we finally deducted that the system would run the script with higher permissions, and we could not run the script ourselves.
After browsing the Internet for where to find scheduled processes in the system, we found /etc/cron.d/, which had the crucial command that runs every minute in the system, with flag_reader's privilege. This is the privilege needed to read the flag. We just had to redirect the output to a file we could read, make flag_reader have write permissions on it and wait for the script to run.

echo 'LD_PRELOAD=/tmp/lib PATH=/tmp' > env && cd /tmp && touch file.txt && chmod 777 file.txt && echo '
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int access (const char* path, int mode)
{
    system("/bin/cat /flags/flag.txt > /tmp/file.txt");
    return 0;
}' > lib.c && gcc -fPIC -g -c lib.c && gcc -shared -o lib lib.o -lc && chmod 777 lib

