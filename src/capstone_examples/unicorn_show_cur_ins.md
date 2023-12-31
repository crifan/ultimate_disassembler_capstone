# Unicorn中打印当前指令

此处介绍，在Unicorn模拟代码期间，用Capstone打印当前正在执行的指令：

自己的实际代码

[模拟akd函数symbol2575](https://book.crifan.org/books/cpu_emulator_unicorn/website/examples/example_akd_symbol2575.html)

中的 `hook_code` 中的这部分的代码：

```py
from capstone import *
from capstone.arm64 import *

BYTES_PER_LINE = 4

# Init Capstone instance
cs = Cs(CS_ARCH_ARM64, CS_MODE_ARM + CS_MODE_LITTLE_ENDIAN)
cs.detail = True

#-------------------- Code --------------------

# memory address where emulation starts
CODE_ADDRESS = 0x10000
# code size: 4MB
CODE_SIZE = 4 * 1024 * 1024
CODE_ADDRESS_END = (CODE_ADDRESS + CODE_SIZE) # 0x00410000


def bytesToOpcodeStr(curBytes):
    opcodeByteStr = ''.join('{:02X} '.format(eachByte) for eachByte in curBytes)
    return opcodeByteStr

# callback for tracing instructions
def hook_code(mu, address, size, user_data):

    # logging.info(">>> Tracing instruction at 0x%x, instruction size = 0x%x", address, size)
    lineCount = int(size / BYTES_PER_LINE)
    for curLineIdx in range(lineCount):
        startAddress = address + curLineIdx * BYTES_PER_LINE
        codeOffset = startAddress - CODE_ADDRESS
        opcodeBytes = mu.mem_read(startAddress, BYTES_PER_LINE)
        opcodeByteStr = bytesToOpcodeStr(opcodeBytes)
        decodedInsnGenerator = cs.disasm(opcodeBytes, address)
        # if gSingleLineCode:
        for eachDecodedInsn in decodedInsnGenerator:
            eachInstructionName = eachDecodedInsn.mnemonic
            offsetStr = "<+%d>" % codeOffset
            logging.info("--- 0x%08X %7s: %s -> %s\t%s", startAddress, offsetStr, opcodeByteStr, eachInstructionName, eachDecodedInsn.op_str)

...
def emulate_akd_arm64_symbol2575():
...
        mu.hook_add(UC_HOOK_CODE, hook_code, begin=CODE_ADDRESS, end=CODE_ADDRESS_END)
```

主要目的就是：

优化了log日志打印，希望打印输出的内容，尽量贴近之前Xcode调试（iOS的ObjC的）ARM汇编代码的（lldb反汇编的）显示效果：

```asm
libobjc.A.dylib`objc_alloc_init:
->  0x19cbd3c3c <+0>:  stp    x29, x30, [sp, #-0x10]!
    0x19cbd3c40 <+4>:  mov    x29, sp
    0x19cbd3c44 <+8>:  cbz    x0, 0x19cbd3c5c          ; <+32>
    0x19cbd3c48 <+12>: ldr    x8, [x0]
    0x19cbd3c4c <+16>: and    x8, x8, #0xffffffff8
    0x19cbd3c50 <+20>: ldrb   w8, [x8, #0x1d]
    ...
```

即，是类似于这种格式：

* 当前地址 <+偏移量>: 指令 操作数

且还希望，加上IDA中能显示opcode的信息：

* 当前地址 <+偏移量>: opcode -> 指令 操作数

所以最后经过优化，用上述代码，实现了类似Xcode中的输出效果：

```asm
--- 0x000113AC <+5036>: 28 01 08 0B  -> add    w8, w9, w8
--- 0x000113B0 <+5040>: 08 09 01 11  -> add    w8, w8, #0x42
--- 0x000113B4 <+5044>: 28 DB A8 B8  -> ldrsw  x8, [x25, w8, sxtw #2]
--- 0x000113B8 <+5048>: 1F 20 03 D5  -> nop
--- 0x000113BC <+5052>: 29 D4 2B 58  -> ldr    x9, #0x68e40
--- 0x000113C0 <+5056>: 08 01 09 8B  -> add    x8, x8, x9
--- 0x000113C4 <+5060>: 00 01 1F D6  -> br     x8
```

如此，可以方便的查看到：

* 当前代码执行到哪里了== 当前的地址 == PC的值
* 函数内的偏移量
* opcode=指令的二进制值
* （借助Capstone解析后的）当前正在执行什么指令
