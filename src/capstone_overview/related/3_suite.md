# 三件套：Capstone+Unicorn+Keystone

* 三件套：Capstone+Unicorn+Keystone
  * Logo
    * ![capstone_logo_3_group](../../assets/img/capstone_logo_3_group.png)
  * 关系
    * 说明
      * **Binary二进制**==**Opcode操作码**
    * 文字
      * `Capstone`是`disassembler`=**反汇编器**：从 **Binary二进制**到 **Assembly汇编代码**
      * `Unicorn`是`emulator`=**模拟器**：**模拟**（CPU）**运行** **Binary二进制**
      * `Keystone`是`assembler`=**汇编器**：从 **Assembly汇编代码** 到 **Binary二进制**
    * 图
      * ![binary_emulator_assembly](../../assets/img/binary_emulator_assembly.png)
      * ![binary_emulator_assembly_hand_draw](../../assets/img/binary_emulator_assembly_hand_draw.jpg)
  * 3个项目
    * 概述：逆向工程的基础框架
      * Fundamental frameworks for Reverse Engineering
    * `Capstone disassembler`
      * 官网
        * Next Generation Disassembler Engine
          * http://capstone-engine.org
    * `Unicorn emulator`
      * 官网
        * Next Generation CPU Emulator
          * http://unicorn-engine.org
    * `Keystone assembler`
      * 官网
        * http://keystone-engine.org
      * 流程
        * ![keystone_assembler_flow](../../assets/img/keystone_assembler_flow.png)
      * IDA插件
        * Keypatch – Keystone – The Ultimate Assembler
          * https://www.keystone-engine.org/keypatch/
            * ![ida_plugin_keypatch](../../assets/img/ida_plugin_keypatch.png)
      * 竞品
        * `Radare2`
          * Unix-like reverse engineering framework and commandline tools
        * `Pwnypack`
          * CTF toolkit with Shellcode generator Ropper: Rop gadget and binary information tool
        * `GEF`
          * GDB plugin with enhanced features
        * `Usercorn`
          * Versatile kernel+system+userspace emulator
        * `X64dbg`
          * An open-source x64/x32 debugger for windows
        * `Liberation`
          * code injection library for iOS
        * `Demovfuscator`
          * Deobfuscator for movfuscated binaries
