# Capstone初始化环境

## Mac中安装Capstone

* 概述
  * Mac
    * `brew install capstone`
  * Ubuntu
    * `sudo apt-get install libcapstone3`

---

详解：

* 先安装Capstone的**core**
  * 命令
    ```bash
    brew install capstone
    ```
  * 注：查看已安装信息
    * `brew info capstone`
      * 已安装版本：`capstone: stable 4.0.2 (bottled), HEAD`
* 再安装对应的**binding**
  * Python
    * 命令
      ```bash
      sudo pip install capstone
      ```
    * 注：
      * 查看已安装信息
        * `pip show capstone`
      * 已安装的版本：`Capstone: 4.0.2`

## 测试Capstone运行正常

用Capstone的Python测试代码：

* X86

```py
# test1.py
from capstone import *

CODE = b"\x55\x48\x8b\x05\xb8\x13\x00\x00"

md = Cs(CS_ARCH_X86, CS_MODE_64)
for i in md.disasm(CODE, 0x1000):
  print("0x%x:\t%s\t%s" %(i.address, i.mnemonic, i.op_str))
```

预期输出：

```bash
0x1000: push    rbp
0x1001: mov     rax, qword ptr [rip + 0x13b8]
```

* arm64

```py
from capstone import *
from capstone.arm64 import *

CODE = b"\xe1\x0b\x40\xb9\x20\x04\x81\xda\x20\x08\x02\x8b"

md = Cs(CS_ARCH_ARM64, CS_MODE_ARM)
md.detail = True

for insn in md.disasm(CODE, 0x38):
    print("0x%x:\t%s\t%s" %(insn.address, insn.mnemonic, insn.op_str))

    if len(insn.operands) > 0:
        print("\tNumber of operands: %u" %len(insn.operands))
        c = -1
        for i in insn.operands:
            c += 1
            if i.type == ARM64_OP_REG:
                print("\t\toperands[%u].type: REG = %s" %(c, insn.reg_name(i.value.reg)))
            if i.type == ARM64_OP_IMM:
                print("\t\toperands[%u].type: IMM = 0x%x" %(c, i.value.imm))
            if i.type == ARM64_OP_CIMM:
                print("\t\toperands[%u].type: C-IMM = %u" %(c, i.value.imm))
            if i.type == ARM64_OP_FP:
                print("\t\toperands[%u].type: FP = %f" %(c, i.value.fp))
            if i.type == ARM64_OP_MEM:
                print("\t\toperands[%u].type: MEM" %c)
                if i.value.mem.base != 0:
                    print("\t\t\toperands[%u].mem.base: REG = %s" \
                        %(c, insn.reg_name(i.value.mem.base)))
                if i.value.mem.index != 0:
                    print("\t\t\toperands[%u].mem.index: REG = %s" \
                        %(c, insn.reg_name(i.value.mem.index)))
                if i.value.mem.disp != 0:
                    print("\t\t\toperands[%u].mem.disp: 0x%x" \
                        %(c, i.value.mem.disp))

            if i.shift.type != ARM64_SFT_INVALID and i.shift.value:
                print("\t\t\tShift: type = %u, value = %u" %(i.shift.type, i.shift.value))

            if i.ext != ARM64_EXT_INVALID:
                print("\t\t\tExt: %u" %i.ext)

    if insn.writeback:
        print("\tWrite-back: True")
    if not insn.cc in [ARM64_CC_AL, ARM64_CC_INVALID]:
        print("\tCode condition: %u" %insn.cc)
    if insn.update_flags:
        print("\tUpdate-flags: True")
```

预期输出：

```bash
0x38:   ldr     w1, [sp, #8]
        Number of operands: 2
                operands[0].type: REG = w1
                operands[1].type: MEM
                        operands[1].mem.base: REG = sp
                        operands[1].mem.disp: 0x8
0x3c:   cneg    x0, x1, ne
        Number of operands: 2
                operands[0].type: REG = x0
                operands[1].type: REG = x1
        Code condition: 2
0x40:   add     x0, x1, x2, lsl #2
        Number of operands: 3
                operands[0].type: REG = x0
                operands[1].type: REG = x1
                operands[2].type: REG = x2
                        Shift: type = 1, value = 2
```

-> 更多测试代码，详见：

* [Programming with Python language – Capstone – The Ultimate Disassembler (capstone-engine.org)](http://www.capstone-engine.org/lang_python.html)
* [capstone/bindings/python at master · capstone-engine/capstone · GitHub](https://github.com/capstone-engine/capstone/tree/master/bindings/python)
