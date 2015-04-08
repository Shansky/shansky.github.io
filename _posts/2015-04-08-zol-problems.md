---
layout: post
title: Problemy z ZOL (ZFS On Linux)
---

## BUG: unable to handle kernel NULL pointer dereference at 0000000000000018
*pod ubuntu 14.04*

Takim to komunikatem przywitał mnie ostatnio bo rebootcie mój ubunciak 14.04 na ZOLu.
A dokładniej mniej więcej takim:

```
[57652.690951] BUG: unable to handle kernel NULL pointer dereference at 0000000000000018
[57652.691040] IP: [<ffffffffa1a7aa2c>] zap_count_write+0x16c/0x3f0 [zfs]
[57652.691120] PGD 0
[57652.691144] Oops: 0000 [#1] PREEMPT SMP                       
[57652.691190] Modules linked in: zfs(PO) nvidia(PO) zunicode(PO) zavl(PO) zcommon(PO) znvpair(PO) spl(O) microcode button                
[57652.691316] CPU: 1 PID: 1748 Comm: txg_sync Tainted: P           O  3.16.5-gentoo #        
[57652.691399] Hardware name: System manufacturer P5Q DELUXE/P5Q DELUXE, BIOS 0605    06/03/2008                                          
[57652.691489] task: ffff88013a83a3c0 ti: ffff8800b27d0000 task.ti: ffff8800b27d0000
[57652.691570] RIP: 0010:[<ffffffffa1a7aa2c>]  [<ffffffffa1a7aa2c>] zap_count_write+0x16c/0x3f0 [zfs]                                     
[57652.691673] RSP: 0018:ffff8800b27d3c28  EFLAGS: 00010286
[57652.691730] RAX: 000000000000001d RBX: ffff8801382a4000 RCX: 0000000000000008
[57652.691804] RDX: 000000000000001d RSI: 000000000000001c RDI: ffff8801382a4000
[57652.691880] RBP: ffff8800b27d3c80 R08: 0000000000000000 R09: 0000000000000002
[57652.691941] R10: 0000000000000000 R11: ffff880138200000 R12: 000000000000001c
[57652.691941] R13: 000000000000001d R14: 0000000000000002 R15: 0000000000000000
[57652.691941] FS:  0000000000000000(0000) GS:ffff88013fc80000(0000) knlGS:0000000000000000
[57652.691941] CS:  0010 DS: 0000 ES: 0000 CR0: 000000008005003b
[57652.691941] CR2: 0000000000000018 CR3: 0000000001c10000 CR4: 00000000000007e0
[57652.691941] Stack:
[57652.691941]  ffff88012c3c5300 0000000000000000 0000000000000202 000000000000001e
[57652.691941]  ffffffffa1a22aaf ffffffffa1a19c09 ffff8800bb006000 0000000000000000
[57652.691941]  ffff880066956540 ffff8800bb006000 0000000000000000 ffff8800b27d3cb0
[57652.691941] Call Trace:
[57652.691941]  [<ffffffffa1a22aaf>] ? dmu_bonus_hold+0xcf/0x2d0 [zfs]
[57652.691941]  [<ffffffffa1a19c09>] ? dbuf_rele_and_unlock+0x179/0x350 [zfs]
[57652.691941]  [<ffffffffa1a7b094>] spa_feature_decr+0x44/0xb0 [zfs]
[57652.691941]  [<ffffffffa1a46d4a>] dsl_scan_sync+0x7ba/0x9d0 [zfs] 
[57652.691941]  [<ffffffffa001fd34>] ? spl_kmem_cache_free+0xa4/0x190 [spl]
[57652.691941]  [<ffffffffa1aa9f1d>] ? zio_wait+0x12d/0x1c0 [zfs]
[57652.691941]  [<ffffffffa1a56b52>] spa_sync+0x4a2/0xae0 [zfs]          
[57652.691941]  [<ffffffff810831cd>] ? autoremove_wake_function+0xd/0x30   
[57652.691941]  [<ffffffff8109f0f3>] ? ktime_get_ts+0x43/0xe0     
[57652.691941]  [<ffffffffa1a65b72>] txg_delay+0x3d2/0x5c0 [zfs]             
[57652.691941]  [<ffffffffa1a65890>] ? txg_delay+0xf0/0x5c0 [zfs]         
[57652.691941]  [<ffffffffa0021ddc>] __thread_exit+0x8c/0xa0 [spl]       
[57652.691941]  [<ffffffffa0021d70>] ? __thread_exit+0x20/0xa0 [spl]    
[57652.691941]  [<ffffffff81065cf4>] kthread+0xc4/0xe0                
[57652.691941]  [<ffffffff81065c30>] ? kthread_create_on_node+0x170/0x170           
[57652.691941]  [<ffffffff817ddbec>] ret_from_fork+0x7c/0xb0     
[57652.691941]  [<ffffffff81065c30>] ? kthread_create_on_node+0x170/0x170     
[57652.691941] Code: 48 89 d0 48 89 e5 41 57 4d 89 c7 41 56 45 89 ce 41 55 49 89 d5 41 54 49 89 f4 53 48 89 fb 48 83 ec 30 48 89 4d c0 b9 
08 00 00 00 <41> 8b 70 18 44 89 4d c8 4c 8d 4d d0 49 8b 50 08 41 b8 01 00 00
[57652.691941] RIP  [<ffffffffa1a7aa2c>] zap_count_write+0x16c/0x3f0 [zfs]
[57652.691941]  RSP <ffff8800b27d3c28>
[57652.691941] CR2: 0000000000000018     
[57652.691941] ------------[ cut here ]------------
[57652.691941] WARNING: CPU: 1 PID: 1748 at kernel/softirq.c:146 __local_bh_enable_ip+0x6a/0xa0()                                         
[57652.691941] Modules linked in: zfs(PO) nvidia(PO) zunicode(PO) zavl(PO) zcommon(PO) znvpair(PO) spl(O) microcode button                
[57652.691941] CPU: 1 PID: 1748 Comm: txg_sync Tainted: P           O  3.16.5-gentoo #1
[57652.691941] Hardware name: System manufacturer P5Q DELUXE/P5Q DELUXE, BIOS 0605    06/03/2008                                          
[57652.691941]  0000000000000009 ffff8800b27d36e0 ffffffff817d5f76 0000000000000000
[57652.691941]  ffff8800b27d3718 ffffffff81045ba8 0000000000000201 ffff88008f42ccc0
[57652.691941]  0000000000000000 0000000000000000 000000000000003c ffff8800b27d3728
[57652.691941] Call Trace:
[57652.691941]  [<ffffffff817d5f76>] dump_stack+0x4e/0x7a
[57652.691941]  [<ffffffff81045ba8>] warn_slowpath_common+0x78/0xa0              
[57652.691941]  [<ffffffff81045c85>] warn_slowpath_null+0x15/0x20      
[57652.691941]  [<ffffffff8104adda>] __local_bh_enable_ip+0x6a/0xa0      
[57652.691941]  [<ffffffff817dd4d5>] _raw_spin_unlock_bh+0x15/0x20     
[57652.691941]  [<ffffffff814b09ce>] cn_netlink_send_mult+0x15e/0x1f0   
[57652.691941]  [<ffffffff814b0a76>] cn_netlink_send+0x16/0x20       
[57652.691941]  [<ffffffff81479a85>] uvesafb_exec+0x155/0x2e0               
[57652.691941]  [<ffffffff81479d45>] uvesafb_blank+0x135/0x190               
[57652.691941]  [<ffffffff8146fbe2>] fb_blank+0x52/0xc0                     
[57652.691941]  [<ffffffff8146a7cb>] fbcon_blank+0x21b/0x320                       
[57652.691941]  [<ffffffff8107236d>] ? get_parent_ip+0xd/0x50                 
[57652.691941]  [<ffffffff81408207>] ? debug_smp_processor_id+0x17/0x20      
[57652.691941]  [<ffffffff81070eb4>] ? get_nohz_timer_target+0x14/0x110
[57652.691941]  [<ffffffff8105083a>] ? internal_add_timer+0x2a/0x70
[57652.691941]  [<ffffffff817dd263>] ? _raw_spin_unlock_irqrestore+0x13/0x30
[57652.691941]  [<ffffffff81052dc1>] ? mod_timer+0x101/0x210
[57652.691941]  [<ffffffff8144d0a3>] do_unblank_screen+0xa3/0x1c0
[57652.691941]  [<ffffffff8144d1cb>] unblank_screen+0xb/0x10
[57652.691941]  [<ffffffff813ff9d9>] bust_spinlocks+0x19/0x40
[57652.691941]  [<ffffffff810063bf>] oops_end+0x2f/0xc0
[57652.691941]  [<ffffffff817d0e0f>] no_context+0x297/0x2a4
[57652.691941]  [<ffffffff817d0e84>] __bad_area_nosemaphore+0x68/0x1bf
[57652.691941]  [<ffffffff817d0fe9>] bad_area_nosemaphore+0xe/0x10
[57652.691941]  [<ffffffff8103a306>] __do_page_fault+0x86/0x490
[57652.691941]  [<ffffffff817dd263>] ? _raw_spin_unlock_irqrestore+0x13/0x30
[57652.691941]  [<ffffffff8107236d>] ? get_parent_ip+0xd/0x50
[57652.691941]  [<ffffffff810723fd>] ? preempt_count_add+0x4d/0xa0
[57652.691941]  [<ffffffff817dd7a8>] ? _raw_spin_lock_irqsave+0x18/0x50
[57652.691941]  [<ffffffff817dd263>] ? _raw_spin_unlock_irqrestore+0x13/0x30
[57652.691941]  [<ffffffff8103a73c>] do_page_fault+0xc/0x10
[57652.691941]  [<ffffffff817df6e2>] page_fault+0x22/0x30
[57652.691941]  [<ffffffffa1a7aa2c>] ? zap_count_write+0x16c/0x3f0 [zfs]
[57652.691941]  [<ffffffffa1a22aaf>] ? dmu_bonus_hold+0xcf/0x2d0 [zfs]
[57652.691941]  [<ffffffffa1a19c09>] ? dbuf_rele_and_unlock+0x179/0x350 [zfs]
[57652.691941]  [<ffffffffa1a7b094>] spa_feature_decr+0x44/0xb0 [zfs]
[57652.691941]  [<ffffffffa1a46d4a>] dsl_scan_sync+0x7ba/0x9d0 [zfs]
[57652.691941]  [<ffffffffa001fd34>] ? spl_kmem_cache_free+0xa4/0x190 [spl]
[57652.691941]  [<ffffffffa1aa9f1d>] ? zio_wait+0x12d/0x1c0 [zfs]
[57652.691941]  [<ffffffffa1a56b52>] spa_sync+0x4a2/0xae0 [zfs]
[57652.691941]  [<ffffffff810831cd>] ? autoremove_wake_function+0xd/0x30
[57652.691941]  [<ffffffff8109f0f3>] ? ktime_get_ts+0x43/0xe0
[57652.691941]  [<ffffffffa1a65b72>] txg_delay+0x3d2/0x5c0 [zfs]
[57652.691941]  [<ffffffffa1a65890>] ? txg_delay+0xf0/0x5c0 [zfs]
[57652.691941]  [<ffffffffa0021ddc>] __thread_exit+0x8c/0xa0 [spl]
[57652.691941]  [<ffffffffa0021d70>] ? __thread_exit+0x20/0xa0 [spl]
[57652.691941]  [<ffffffff81065cf4>] kthread+0xc4/0xe0
[57652.691941]  [<ffffffff81065c30>] ? kthread_create_on_node+0x170/0x170
[57652.691941]  [<ffffffff817ddbec>] ret_from_fork+0x7c/0xb0
[57652.691941]  [<ffffffff81065c30>] ? kthread_create_on_node+0x170/0x170
[57652.691941] ---[ end trace 80ac2dc931b9aee5 ]---
[57652.836655] ---[ end trace 80ac2dc931b9aee6 ]---
```
Tylko że w moim przypadku to nie gentoo i inna wersja jajka. Niestety nie udało mi się określić co spowodowało problem. Oczywiście zgłoszono już podobne problemy ale zazwyczaj pojawiały się po jakiś operacjach na poolach, w moim przypadku nie robiłem żadnych zmian, a sama historia zfs'a też nic nie pokazuje... dziwne.

Operacja naprawcza będzie opisana poniżej. Pomaga ona również na problem:
```
Failed to load ZFS module stack.
```
który pojawił się przy próbach naprawy wcześniejszego BUGa. Równocześnie jest to również sposób na rozwiązanie, równie popularnego problemu: **`Error! Could not locate dkms.conf file`**

#### Powiązane problemy:
 - [#1155: Failed to load ZFS module stack.](https://github.com/zfsonlinux/zfs/issues/1155)
 - [#3019: NULL pointer while trying to destroy a cloned filesystem.](https://github.com/zfsonlinux/zfs/issues/3019)
 - [#1518: zfs receive -> unable to handle kernel NULL pointer dereference.](https://github.com/zfsonlinux/zfs/issues/1518)
 - [Failed to load ZFS module stack](http://askubuntu.com/questions/552201/failed-to-load-zfs-module-stack)

### Environment Info

Kernel:

```
Linux mtiPC 3.13.0-48-generic #80-Ubuntu SMP Thu Mar 12 11:16:15 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```

SPL i ZFS:

```
spl, 0.6.3, 3.13.0-48-generic, x86_64: installed
zfs, 0.6.3, 3.13.0-48-generic, x86_64: installed
```

Poole:
Poola złożona z dwóch dysków SSD:

```
$ zfs list
NAME                  USED  AVAIL  REFER  MOUNTPOINT
rpool                 109G   109G    31K  /rpool
rpool/ROOT            109G   109G    31K  /rpool/ROOT
rpool/ROOT/ubuntu-1   109G   109G   106G  /
```

```
$ zpool list
NAME    SIZE  ALLOC   FREE    CAP  DEDUP  HEALTH  ALTROOT
rpool   222G   109G   113G    49%  1.00x  ONLINE  -
```

```
$ zpool status rpool 
  pool: rpool
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Sun Aug 24 15:07:19 2014
config:

  NAME                                                STATE     READ WRITE CKSUM
  rpool                                               ONLINE       0     0     0
    ata-INTEL_SSDSC2CT120A3_CVMP231503KM120BGN-part2  ONLINE       0     0     0
    ata-INTEL_SSDSC2BW120A4_CVDA442001XQ1207GN-part1  ONLINE       0     0     0

errors: No known data errors
```

### Reapair HOW-TO

#### Live-USB
Potrzebne będzie jakieś live distro, najlepiej na USB - polecam je wykonać przy pomocy *unetbooin* - w moim przypadku ubuntu gnome 14.04.2 - najbliższe mojemu systemowi. I z niego startujemy system.

#### Przygotowanie live ubuntu do pracy

```
$ sudo -i
# add-apt-repository --yes ppa:zfs-native/stable
# apt-get update
# apt-get install -y debootstrap spl-dkms zfs-dkms ubuntu-zfs
```

Włączenie modułu ZFS:

```
# modprobe zfs
# dmesg | grep ZFS
[    1.925403] ZFS: Loaded module v0.6.3-5~trusty, ZFS pool version 5000, ZFS filesystem version 5
```

Output może być trochę inny :)
#### Podmontowanie systemu bazowego i chroot na niego
Teraz można zaimportować poole:

```
# zpool import -d /dev/disk/by-id -R /mnt rpool
```
Powinno automatycznie zaciągnąć rpool, jeśli nie działa można spróbować:

```
# zpool import -R /mnt rpool
```

U mnie osobno jest partycja z GRUBem, więc mount boot/grub może się przydać (np. do sprawdzenia jakie jajka mamy) do utworzenia initramfs.

```
# mkdir -p /mnt/boot/grub
# mount /dev/disk/by-id/ata-INTEL_SSDSC2CT120A3_CVMP231503KM120BGN-part1 /mnt/boot/grub
```

[INFO] W tym momencie można wydać polecenie

```
# debootstrap trusty /mnt
```

by zainstalować świeży podstawowy ubuntu.
Kilka mountów i chroot na system bazowy.

```
# mount --bind /dev /mnt/dev
# mount --bind /proc /mnt/proc
# mount --bind /sys /mnt/sys
# chroot /mnt /bin/bash --login
```

W tym momencie jesteśmy praktycznie na naszym systemie bazowym, można bez problemu instalować paczki, podnosić kernel, ładować moduły, edytować pliczki, itp. itd.

Jednak żeby rozwiązać problem należy pobawić się z kernelami i załadowanymi dla nich modułami.

```
# dkms status
spl, 0.6.3, 3.13.0-48-generic, x86_64: installed
zfs, 0.6.3, 3.13.0-48-generic, x86_64: installed
```

**dkms** wyświetla moduły zainstalowane dla konkretnych kerneli, w moim przypadku dla 3.13.0-48-generic ale taki output to "zdrowy" output. Normalnie **dkms** wyświetla jakieś problemy w nawiasie przy module. Warto przebudować moduły **spl** i **zfs**. Tutaj warto zwrócić uwagę że domyślnie chroot dalej myśli że ma kernel w nietypowej wersji, której nie ma w systemie bazowym - jest to wersja kernela z dystrybucji live. Więc przy instalacji modułów warto podawać dokładną wersję dla której moduł budujemy i instalujemy - w przeciwnym przypadku pobrana zostanie wartość dla zmiennej $KERNELRELEASE z ```uname -r``` czyli wersja kernela dla dystrybucji live.

```
# dkms remove -m zfs -v 0.6.3 --all
# dkms remove -m spl -v 0.6.3 --all
# dkms add -m spl -v 0.6.3 -k 3.13.0-48-generic
# dkms add -m zfs -v 0.6.3 -k 3.13.0-48-generic
# dkms install -m spl -v 0.6.3 -k 3.13.0-48-generic
# dkms install -m zfs -v 0.6.3 -k 3.13.0-48-generic
```

Teoretycznie można przed ```dkms install``` zrobić jeszcze ```dkms build``` dla każdego z modułów ale nie jest to wymagane, **install** i tak zbuduje moduł.

Teraz należy powiadomić wszystkich o zmianie w naszym jajku.

```
# update-initramfs -c -k 3.13.0-48-generic
```

Warto również zweryfikować czy moduł znajduje się w bootowanym obrazie, żeby nie zaskoczył nas problem z **Failed to load ZFS module stack.**:

```
# gzip -dc /boot/initrd.img-3.13.0-48-generic | cpio -it | grep zfs.ko
```

Jeśli go nie ma to znaczy że coś nie tak poszło przy instalacji modułu. (Czytamy logi?)
[INFO] Może się zdarzyć że to nie pomoże, warto wtedy przeinstalować nagłówki jajka:

```
# apt-get install --reinstall linux-headers-3.13.0-48 linux-headers-3.13.0-48-generic
```

Warto odświeżyć gruba

```
# update-grub
```

#### Kroki końcowe:
Wychodzimy z chroota, odmontowanie i export pooli - jeśli export się nie powiedzie, system nie wystartuje.

```
# exit
# umount /mnt/boot/grub
# umount /mnt/dev
# umount /mnt/proc
# umount /mnt/sys
# zfs umount -a
# zpool export rpool
```

Koniec - reboot.
*Dziwne, u mnie działa*

-------
##### Source:
 1. [#1155: Failed to load ZFS module stack.](https://github.com/zfsonlinux/zfs/issues/1155)
 2. [#3019: NULL pointer while trying to destroy a cloned filesystem.](https://github.com/zfsonlinux/zfs/issues/3019)
 3. [#1518: zfs receive -> unable to handle kernel NULL pointer dereference.](https://github.com/zfsonlinux/zfs/issues/1518)
 4. [Failed to load ZFS module stack](http://askubuntu.com/questions/552201/failed-to-load-zfs-module-stack)
 5. [Problems installing ubuntu 11.04 with native ZFS root -msg#00148](https://osdir.com/ml/zfs-discuss/2011-05/msg00148.html)
 6. [HOWTO install Ubuntu to a Native ZFS Root Filesystem](https://github.com/zfsonlinux/pkg-zfs/wiki/HOWTO-install-Ubuntu-to-a-Native-ZFS-Root-Filesystem)