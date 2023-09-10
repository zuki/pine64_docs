# tftp経由のプログラムロード

## tftpdの起動と停止

```bash
# 起動
$ sudo launchctl load -w /System/Library/LaunchDaemons/tftp.plist
# 起動確認
$ sudo lsof -i:69
# 停止
$ sudo launchctl unload -w /System/Library/LaunchDaemons/tftp.plist
```

## テストプログラムの用意

```bash
$ vi main.c
$ aarch64-elf-gcc -Wl,-Ttext=0x42000000 -nostdlib main.c -o serial_test
$ aarch64-elf-objdump -ld serial.test
0000000042000000 <_start>:
_start():
    42000000:	d2900000 	mov	x0, #0x8000                	// #32768
    42000004:	f2a03840 	movk	x0, #0x1c2, lsl #16
    42000008:	d2800aa1 	mov	x1, #0x55                  	// #85
    4200000c:	f9000001 	str	x1, [x0]
    42000010:	17fffffc 	b	42000000 <_start>
$ aarch64-elf-objcopy -Obinary serial_test serial.test.img
$ sudo cp serial_test.img /private/tftpboot
```

## U-Bootで実行

```bash
=> bootp 192.168.10.103:serial_test.img
BOOTP broadcast 1
DHCP client bound to address 192.168.10.121 (3 ms)
Using ethernet@1c30000 device
TFTP from server 192.168.10.103; our IP address is 192.168.10.121
Filename 'serial_test.img'.
Load address: 0x40080000                # 0x40080000にロードされる (kernel_addr)
Loading: #                              # Linuxではこれを0x42000000に移動している
         2 KiB/s
done
Bytes transferred = 20 (14 hex)
=> go 0x40080000                        # プログラムは正常に動いている
## Starting application at 0x40080000 ...
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU?
```
