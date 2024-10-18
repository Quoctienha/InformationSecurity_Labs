# Lab ctf
Hà Quốc Tiến - 22110075<br>
## Target
- Causing buffer-overflow in order to call `myfunc()` and print out `You got the flag`.
## Solution
- Stack frames when call `vuln()` <br>
![ctf_1](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_1.png)
- Viewing the stack frame, we have an idea that change the `Return Address` of `vuln()` with `myfunc()`'s address for it to run.

1. Get `myfunc()`'s address, using GDB or this.
    > objdump -d ctf.out|grep myfunc

    *Inspect with GDB

    > gdb ctf.out

    Disassemble `myfunc()`
    > disas myfunc
    In this case is `0x0804851b`
    ![ctf_4](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_4.png)

2. To get pass the first if statment, create a `flag1.txt` that isn't empty. Here we can't use the `fgets()` to conduct buffer-overflow, because `fgets()` take 64 bytes of data and `filebuf[64]` can store 64 bytes of data. So it's safe here.
    > echo "flag1 is here" > flag1.txt

    ![ctf_2](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_2.png)
3. In order to get pass, the two next if statment, we need `q = 0x44644262` and `p = 0x04081211`.

    We can she that the computer will base on the `EBP` to calculate the position of `q`([ebp+0xc] which is ebp + 12 bytes) and `p` ([ebp+0x8] which is ebp + 8 bytes)<br>
    ![ctf_3](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_3.png)
    
    So, if we want to change the data in `p` and `q` using buffer-overflow. We have to know the position of `EBP` when `vuln()` is done.<br>

    When `vuln()` is done, `ESP` will move to the `EBP` position to release memory `mov ebp, esp`. Then `pop ebp`, so the `ESP` will point to the Return Address of `vuln()`.

    `myfunc()` will be called. But it doesn't have a caller so it don't have a Return Address. So when `myfunc()` run, the stack frames look like and we will see where is `p` and `q`.<br>

     ![ctf_5](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_5.png)

- Conclusion, we have to insert 104 random bytes + `myfunc()`'s address + 4 random bytes(or address of `exit()` to cover your track => no segmentation fault) + `0x04081211` + `0x44644262`
## Execution
- We will cover the track, so we will get the address of `exit()` using GDB.<br>
    > info func

    And we can see the address of `exit()` from the first if statment `0x080483e0`.<br>
    ![ctf_6](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_6.png) 

- Buffer-overflow.
    > ./ctf.out $(python -c "print('a'*104 +'\x1b\x85\x04\x08' + '\xe0\x83\x04\x08' + '\x11\x12\x08\x04' +'\x62\x42\x64\x44')")
    
    ![ctf_7](https://github.com/Quoctienha/InformationSecurity_Labs-/blob/main/Buffer-overflow/img/ctf_7.png) 
