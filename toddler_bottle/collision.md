# collision

## Code
In this challenge, we are given:
```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

Our aim is to make `check_password(argv[1])` to be equal to `0x21DD09EC` or `568134124` in decimal representation.

## Solution
From the code, we derive the following truths:

```
Sum of ip - 2^32 = 568134124
Sum of ip = 4863101423
Each ip = 972620284
```

This means that each 4-byte value of the input arguments must be equal to `972620284` when converted from char* to int.
Hence, convert `972620284` to hex and it will be `39F901FC`.

We have to echo it in the little-endian manner, i.e. `\xfc\x01\xf9\x39 * 5`:
```
./col `echo -n -e "\xfc\x01\xf9\x39\xfc\x01\xf9\x39\xfc\x01\xf9\x39\xfc\x01\xf9\x39\xfc\x01\xf9\x39"`
```
