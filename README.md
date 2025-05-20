# eucalypsih_rcrapss
ldr reg, [reg] // membaca nilai dari memori di alamat yang ada di register [reg]
str reg, [reg] // menulis nilai dari register [reg] ke alamat memori

mov x0,  1 // stdout (file descriptor 1)
mov x0,  0 // status keluar 0

mov x8, 64 // syscall number untuk write
mov x8, 93 // syscall number untuk exit

ldr x2, [x2] LDR r2, [r2]

