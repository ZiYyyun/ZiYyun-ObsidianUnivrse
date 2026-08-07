


### 编译流程
```
预处理：xxx.c -> xxx.i
编译：  xxx.i -> xxx.s
汇编：  xxx.s -> xxx.o
链接：  xxx.o -> 可执行文件
```



### makefile
```makefile
objs := start.o main.o
ledc.bin:$(objs)
    arm-linux-gnueabihf-ld -Ttext 0X87800000 -o ledc.elf $^
    arm-linux-gnueabihf-objcopy -O binary -S ledc.elf $@
    arm-linux-gnueabihf-objdump -D -m arm ledc.elf > ledc.dis
%.o:%.s
    arm-linux-gnueabihf-gcc -Wall -nostdlib -c -o $@ $<
%.o:%.S
    arm-linux-gnueabihf-gcc -Wall -nostdlib -c -o $@ $<
%.o:%.c
    arm-linux-gnueabihf-gcc -Wall -nostdlib -c -o $@ $<
clean:
    rm -rf *.o ledc.bin ledc.elf ledc.dis
```

以这份`Makefile`为例，

`:=` 可以看做是赋值，那么objs就是变量
