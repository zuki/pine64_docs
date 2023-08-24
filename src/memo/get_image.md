# pine64 linux imageの取得

```bash
$ wget https://www.stdin.xyz/downloads/people/longsleep/pine64-images/simpleimage-pine64-latest.img.xz
$ unxz simpleimage-pine64-latest.img.xz
$ Etcherでsdカードに書き込み
$ cd /Volumes/BOOT
$ ls -l
total 1074
-rwxrwxrwx 1 dspace staff 1094464  3 11  2017 initrd.img
drwxrwxrwx 1 dspace staff    2048  3 11  2017 pine64
-rwxrwxrwx 1 dspace staff     111  3 11  2017 uEnv.txt
$ ls -l pine64/
total 11764
-rwxrwxrwx 1 dspace staff 11835840  3 11  2017 Image
-rwxrwxrwx 1 dspace staff    69414  3 11  2017 sun50i-a64-pine64-plus.dtb
-rwxrwxrwx 1 dspace staff    69434  3 11  2017 sun50i-a64-pine64-so.dtb
-rwxrwxrwx 1 dspace staff    69322  3 11  2017 sun50i-a64-pine64.dtb
```
