# eucalypsih_rcrapss
ldr reg, [reg] // membaca nilai dari memori di alamat yang ada di register [reg]
str reg, [reg] // menulis nilai dari register [reg] ke alamat memori

mov r0,  #1    mov x0,  1 // stdout (file descriptor 1)
mov r0,  #0    mov x0,  0 // status keluar 0

mov r7,  #4    mov x8, 64 // syscall number untuk write
mov r7,  #1    mov x8, 93 // syscall number untuk exit

adr x2,  msg // cocok dipakai saat string/data delay dengnan kode.
ldr x2,  =msg // digunakan kalau data jauh atau ingin alamat pasti melalui literal pool.

LDR r2, [r2] // arm
ldr x2, [x2] // arch64
LW $t0,  0($t0) // mips
LD $t0,  0($t0) // mips64
LOAD.W (r2), (r2) // esp32


```assembly
.global _start

_start:
    // Syscall: write (4)
    mov r7, #4          // Syscall number for 'write'
    mov r0, #1          // File descriptor 1 (stdout)
    ldr r1, =message    // Load adress of the message to write
    mov r2, #13         // Length of the message
   // or ldr r2, =length  // Load address of the length
    svc #0              // Make the system call

    // Syscall: exit (1)
    mov r7, #1          // Syscall number for 'exit'
    mov r0, #0          // Exit status 0
    svc #0              // Make the system call

message:
    .asciz "Hello, ARM!\n"

length:
    .word 12

```




