+ objdump反汇编

# objdump反汇编

./tools/toolchain/gcc-linaro-7.2.1-2017.11-x86_64_aarch64-linux-gnu/aarch64-linux-gnu/bin/objdump -S -l -z vmlinux > vmlinux.txt

反汇编vmlinux到vmlinux.txt，其中含有汇编和c源文件的混合代码，看起来很方便。而且能一步步看linux怎么一步步运行的。但是花时间比较长，需要耐心等待；如果你的模块是编译成ko加载，并不在内核的话，那么把vmlinux换成ko也是相同的效果。
