# macでu-bootをSDカードに書き込む

1. デバイス名を確認する。この場合、目的のSDカードは`/dev/disk6`

```bash
$ diskutil list
...
/dev/disk5 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +1000.0 GB  disk5
                                 Physical Store disk4s2
   1:                APFS Volume wdb                     132.5 GB   disk5s1

/dev/disk6 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.5 GB    disk6
   1:             Windows_FAT_32 NO NAME                 67.1 MB    disk6s1
   2:                      Linux                         104.9 MB   disk6s2
                    (free space)                         15.3 GB    -
```

2. SDカードをアンマウントする

```bash
$ sudo diskutil unmountDisk /dev/disk6
```

3. u-bootを書き込む。pine64の場合、先頭からオフセット`8 KiB`の位置に書き込む必要がある。

```bash
$ sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/disk6 bs=1024 seek=8
```

4. SDカードをアンマウントする

```bash
$ sudo diskutil unmountDisk /dev/disk6
```
