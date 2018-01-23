# fd

## Code
In this challenge, we are given this code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

## The Catch
The problem of the code lies with:

```c
int fd = atoi( argv[1] ) - 0x1234;
```

In Linux, if the file descriptor is `0`, the program will read the input from `stdin`.

## Solution
We will just have to input an integer such that `atoi( argv[1] )` returns `0x1234`.
Since `0x1234` is in hex representation, the decimal representation is `4660`.
```
./fd 4660
LETMEWIN
```
