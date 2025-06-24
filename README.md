# eucalypsih_rcrapss
ldr reg, [reg] // membaca nilai dari memori di alamat yang ada di register [reg]
str reg, [reg] // menulis nilai dari register [reg] ke alamat memori

ldr r1, =msg        ldr x1, =msg     // load address of the message to write

mov r0,  #1    mov x0,  1 // stdout (file descriptor 1)
mov r0,  #0    mov x0,  0 // status keluar 0

mov r7,  #4    mov x8, #64 // syscall number untuk 'write'
mov r7,  #1    mov x8, #93 // syscall number untuk 'exit'

mov r2, #14   mov x2, #14 // length of the message

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
   // ldr r2, =length  // Load address of the length
    // mov r2,  [r2] // Load the length of the message
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

```assembly
.global _start

_start:
    /* ==================== 1. Buat Socket ==================== */
    mov r0, #2          /* AF_INET (IPv4) */
    mov r1, #1          /* SOCK_STREAM (TCP) */
    mov r2, #0          /* Protocol (0 = default) */
    mov r7, #281        /* syscall socketcall (socket) */
    svc #0              /* Panggil sistem */

    /* Error Handling: Cek apakah socket berhasil dibuat */
    cmp r0, #0
    ble error_exit      /* Jika return <= 0, berarti gagal */
    mov r4, r0          /* Simpan fd socket */

    /* ==================== 2. Connect ke Server ==================== */
    ldr r1, =server_addr
    mov r2, #16         /* Ukuran sockaddr_in */
    mov r7, #283        /* syscall connect */
    svc #0
    cmp r0, #0
    blt error_exit      /* Jika connect gagal */

    /* ==================== 3. Kirim HTTP Request ==================== */
    ldr r1, =http_request
    ldr r2, =http_request_len
    ldr r2, [r2]        /* Load panjang request */
    mov r7, #4          /* syscall write */
    svc #0
    cmp r0, #0
    ble error_exit      /* Jika kirim gagal */

    /* ==================== 4. Setup SELECT ==================== */
    /* Struktur timeval untuk timeout */
    ldr r8, =timeout
    ldr r9, [r8]        /* timeout.tv_sec */
    ldr r10, [r8, #4]   /* timeout.tv_usec */

    /* FD_SET untuk socket */
    ldr r5, =readfds
    mov r6, #0
    str r6, [r5]        /* Bersihkan readfds */
    mov r0, #1
    lsl r0, r4          /* Set bit ke-N (N = fd socket) */
    str r0, [r5]        /* readfds = { socket } */

    /* Panggil select */
    mov r0, r4          /* nfds = socket_fd + 1 */
    add r0, #1
    mov r1, r5          /* readfds */
    mov r2, #0          /* writefds = NULL */
    mov r3, #0          /* exceptfds = NULL */
    ldr r7, =select_syscall
    ldr r7, [r7]        /* Load nomor syscall select */
    svc #0

    cmp r0, #0          /* Cek return select */
    ble timeout_error   /* <= 0: timeout/error */

    /* ==================== 5. Baca Response ==================== */
    ldr r1, =buffer
    mov r2, #1024       /* Ukuran buffer */
    mov r7, #3          /* syscall read */
    svc #0
    cmp r0, #0
    ble error_exit

    /* ==================== 6. Tampilkan Response ==================== */
    mov r2, r0          /* Panjang yang dibaca */
    ldr r1, =buffer
    mov r0, #1          /* stdout */
    mov r7, #4          /* syscall write */
    svc #0

    /* ==================== 7. Exit ==================== */
exit:
    mov r0, #0          /* Exit code 0 */
    mov r7, #1          /* syscall exit */
    svc #0

error_exit:
    mov r0, #1          /* Exit code 1 */
    mov r7, #1          /* syscall exit */
    svc #0

timeout_error:
    ldr r1, =timeout_msg
    ldr r2, =timeout_msg_len
    ldr r2, [r2]
    mov r0, #1          /* stdout */
    mov r7, #4          /* syscall write */
    svc #0
    b exit

.data
/* Struktur sockaddr_in */
server_addr:
    .short 2            /* AF_INET */
    .short 0x5000       /* Port 80 (network byte order) */
/*    .word 0xD83A1D5E     216.58.29.94 (httpbin.org) */
/*    .word 0x23A9F8E8     35.169.248.232 (httpbin.org) */
    .word 0x2CD5A2BA
    .space 8            /* Padding */

/* HTTP GET request */
http_request:
    .ascii "GET /get HTTP/1.1\r\nHost: httpbin.org\r\nUser-Agent: ARM-Asm\r\nConnection: close\r\n\r\n"
http_request_len:
    .word . - http_request

/* Buffer untuk response */
buffer:
    .space 1024

/* Timeout 5 detik */
timeout:
    .word 5             /* tv_sec */
    .word 0             /* tv_usec */

/* fd_set untuk select */
readfds:
    .word 0

/* Nomor syscall select */
select_syscall:
    /* Nomor bisa beda tergantung arch */
    .word 142           /* Syscall select di ARM EABI */

/* Pesan timeout */
timeout_msg:
    .ascii "Error: Timeout menunggu response\n"
timeout_msg_len:
    .word . - timeout_msg

```


