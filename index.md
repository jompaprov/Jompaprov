# Exam 2018-06-05

## Assignment 2

This program starts a process called `watcher` that should monitor the rest of the program.
There are three calls to `system ps(...)`.
The first one from `uppg2` and the two other ones from `watcher`.

### Watcher
```c
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char** argv)
{
    sleep(2);
    system("ps -e -o pid,ppid,comm | grep uppg2");

    sleep(2);
    system("ps -e -o pid,ppid,comm | grep uppg2");

    return 0;
}
```
> This file might not match the one used during examination exactly because it was already written.
The timing might therefore not be the same.

### Uppg2
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <time.h>

int main(int argc, char** argv)
{
    if (argc != 2) {
        printf("Usage: %s [number]\n", argv[0]);
        return 0;
    }

    // Start watcher process
    if (!fork()) {
        if (!fork()) {
            execlp("./watcher", "", NULL);
            printf("Cannot find watcher process\n"); // Debug print
            exit(0);
        }
        exit(0);
    }

    wait(0);
    sleep(1); // Timing
    system("ps -e -o pid,ppid,comm | grep watcher");

    // Start child processes
    srand(time(NULL)); // Set seed
    
    int children = atoi(argv[1]);
    int killer = rand() % children;
    int status;

    // Print to match exemple output
    printf("Nr of children: %d\nKiller: %d\n", children, killer + 1);

    int killerPipe[2];

    int i;
    for (i = 0; i < children; i++) {
        pipe(killerPipe); // Create pipe in loop, or the program might not work

        if (!fork()) {
            close(killerPipe[1]); // Not required

            int isKiller;
            read(killerPipe[0], &isKiller, sizeof(int));

            sleep(2); // Timing
            if (isKiller)
                kill(getppid(), 9);
            else
                sleep(2); // Do nothing for the rest of the program
            
            close(killerPipe[0]); // Not required

            exit(0);
        }

        status = (i == killer);
        write(killerPipe[1], &status, sizeof(int));

        close(killerPipe[0]);
        close(killerPipe[1]);
    }

    sleep(10); // Sleep untill killed

    printf("Child failed to kill parent\n"); // This should not print

    return 0; // Return something to make the compiler happy
}
```
