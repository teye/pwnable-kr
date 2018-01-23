# random

## Code
In this challenge, we are given:

```c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

Suppose we insert some print statements and execute this a few times:
```
1234
random: 1804289383
key: 1234
key ^ random: 1804288437
0xdeadbeef: -559038737
Wrong, maybe you should try 2^32 cases.

999999999999
random: 1804289383
key: 2147483647
key ^ random: 343194264
0xdeadbeef: -559038737
Wrong, maybe you should try 2^32 cases.
```

We quickly notice that the `random` value is always `1804289383`. This is because `rand()` is executed without any inputs. If no input is supplied to `rand()`, it defaults to `rand(1)`, i.e. the seed valie is always the same.

Hence, we just have to solve the equation:
```
key ^ 1804289383 = -559038737
key = 1804289383 ^ -559038737
key = -1255736440
```
Therefore:
```
./random
-1255736440
```
