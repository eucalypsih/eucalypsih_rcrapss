# eucalypsih_rcrapss
ldr reg, [reg] // membaca nilai dari memori di alamat yang ada di register [reg]
str reg, [reg] // menulis nilai dari register [reg] ke alamat memori

mov x0,  1 // stdout (file descriptor 1)
mov x0,  0 // status keluar 0

mov x8, 64 // syscall number untuk write
mov x8, 93 // syscall number untuk exit

adr x2,  msg // cocok dipakai saat string/data delay dengnan kode.
ldr x2,  =msg // pointer ke string, bahkan kalau jauh

LDR r2, [r2] // arm
ldr x2, [x2] // arch64
LW $t0,  0($t0) // mips
LD $t0,  0($t0) // mips64
LOAD.W (r2), (r2) // esp32






