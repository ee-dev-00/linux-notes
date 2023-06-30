# Basic

### find
Search directories by name

    find ./external -type d -name '.git*'

Search with exec

    find ./external -type d -name '.git*' -exec rm -rf {} \;

# Diagnostic

### strace
Allows to print all system calls made by the specified process. Use `-f` to follow the child process, `-t` prints timestamps. You can use the `-k` switch to figure out where did the system call originate from

    strace ls

    strace -e trace=open,stat ./my_program

    strace -t -f -e trace=file -o strace_output.txt ./my_program