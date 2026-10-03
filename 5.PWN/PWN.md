PWN
===
🔙 [MENU README](../README.md)

# Foreword
```
C is the language that created modern operating systems (e.g. Unix); it remains irreplaceable because it forms the foundation of modern computing.
```

# Execuable
![](./Executable.png)

![](./MemoryMappping.png)

Windows
```
- For a program in Windows to execute successfully, the files and configuration must both match
- MS Visual C Runtime (msvcrt.dll) : Packages Microsoft's C standard library function implementations
- When msvcrt.dll is missing, we need to download the Visual C++ Redistributable; which exact version is needed is often known only by the developer
- cygwin : Runs a POSIX/Linux emulation runtime environment directly under Windows
```

Linux
```
- GNU C Library (glibc.so)
- Wine (Wine Is Not an Emulator) : Brings the Windows runtime/API to Linux, implementing Windows libraries natively on Linux
```
# Memory
```
2**32 = 4G
```
![](../5.PWN/Memory.png)

# Assembly
```
AT&T  => source, destination
Intel => destination, source

Assemble => Assembly to Machine code
Compile  => Source code to Machine code / Assembly
```

# StackFrame
Stack
```bash 
# Above Saved EBP is where the new stack frame begins


Local Var  
#<---New Function Frame
Save EBP
ret addr
Arg1
Arg2 
# ----main----
```

# Function
Function Prologue
```asm
PUSH %EBP
MOV %ESP, %EBP
SUB  n, %ESP
```
Function Epilogue
```asm
MOV %EBP, %ESP
POP %EBP
RET
```

# PEDA
```bash
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/gdbinit
echo "DONE! debug your program with gdb and enjoy"
```

# Challenge
Lab Enviroment
```
Module 13 : Binanry Analysis and Exploitation
Software-Test-Linux-32bits

Kevin-32bit
studentpassword
```
1. [crackme0x00a](./Challlenge/1.crackme0x00a.md)
2. [shellcode](./Challlenge/2.shellcode.md)
3. [BOF : ret2sc](./Challlenge/3.ret2sc.md)
4. [NX : ret2libc](./Challlenge/4.ret2libc.md)
5. [Static-ROP : challenge-one](./Challlenge/5.Static-ROP.md)
6. [ROP : level-two](./Challlenge/6.ROP.md)