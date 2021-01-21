# virt_tech_brief

A brief on emerging tech to build a modern infrastructure platform

## Info

See "Virtualization Tech Briefing.pptx" for the briefing slides and notes

## Demos

### qemu-img Operations

convert from a vmdk file to a qcow2 file:
`qemu-img convert -p -f vmdk -O qcow2 al-redis-1-disk0.vmdk al-redis-1-disk0.qcow2`

get info on a disk image file:

```
qemu-img info al-redis-1-disk0.qcow2 
image: al-redis-1-disk0.qcow2
file format: qcow2
virtual size: 64 GiB (68719476736 bytes)
disk size: 7.48 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

```

### qemu-system Operations


[https://qemu.readthedocs.io/en/latest/system/invocation.html]

Typical invocation for a Linux guest:

```sh
/usr/bin/qemu-system-x86_64 \
-machine pc-q35-4.2 \
-smp cpus=2 \
-m 4G \
-enable-kvm \
-cpu host \
-device ich9-usb-ehci1,id=usb,bus=pcie.0 \
-netdev user,id=usernet0 \
-device virtio-net-pci,netdev=usernet0,id=net0 \
-device usb-tablet,id=input0,bus=usb.0,port=1 \
-spice port=5900,addr=127.0.0.1 \
-device qxl-vga,id=video0 \
-blockdev driver=file,filename=/tank/virtual_machines/al-redis-1-disk0.qcow2,node-name=libvirt-1-storage \
-blockdev node-name=libvirt-1-format,driver=qcow2,file=libvirt-1-storage \
-device virtio-blk-pci,scsi=off,drive=libvirt-1-format,id=virtio-disk0,bootindex=1
```


What Libvirt uses for the same guest:

```sh
/usr/bin/qemu-system-x86_64 \
-name guest=redis1,debug-threads=on \
-S -object secret,id=masterKey0,format=raw,file=/var/lib/libvirt/qemu/domain-1-redis1/master-key.aes \
-machine pc-q35-4.2,accel=kvm,usb=off,vmport=off,dump-guest-core=off \
-cpu Haswell-noTSX-IBRS,vme=on,ss=on,vmx=on,f16c=on,rdrand=on,hypervisor=on,arat=on,tsc-adjust=on,umip=on,md-clear=on,stibp=on,arch-capabilities=on,ssbd=on,xsaveopt=on,pdpe1gb=on,abm=on,ibpb=on,amd-stibp=on,amd-ssbd=on,skip-l1dfl-vmentry=on,pschange-mc-no=on \
-m 16384 \
-overcommit mem-lock=off -smp 2,sockets=2,cores=1,threads=1 -uuid eb85a40a-785a-4f25-a853-bd3142eb2da8 \
-no-user-config \
-nodefaults \
-chardev socket,id=charmonitor,fd=31,server,nowait \
-mon chardev=charmonitor,id=monitor,mode=control \
-rtc base=utc,driftfix=slew \
-global kvm-pit.lost_tick_policy=delay \
-no-hpet \
-no-shutdown \
-global ICH9-LPC.disable_s3=1 \
-global ICH9-LPC.disable_s4=1 \
-boot strict=on \
-device pcie-root-port,port=0x10,chassis=1,id=pci.1,bus=pcie.0,multifunction=on,addr=0x2 \
-device pcie-root-port,port=0x11,chassis=2,id=pci.2,bus=pcie.0,addr=0x2.0x1 \
-device pcie-root-port,port=0x12,chassis=3,id=pci.3,bus=pcie.0,addr=0x2.0x2 \
-device pcie-root-port,port=0x13,chassis=4,id=pci.4,bus=pcie.0,addr=0x2.0x3 \
-device pcie-root-port,port=0x14,chassis=5,id=pci.5,bus=pcie.0,addr=0x2.0x4 \
-device pcie-root-port,port=0x15,chassis=6,id=pci.6,bus=pcie.0,addr=0x2.0x5 \
-device ich9-usb-ehci1,id=usb,bus=pcie.0,addr=0x1d.0x7 \
-device ich9-usb-uhci1,masterbus=usb.0,firstport=0,bus=pcie.0,multifunction=on,addr=0x1d \
-device ich9-usb-uhci2,masterbus=usb.0,firstport=2,bus=pcie.0,addr=0x1d.0x1 \
-device ich9-usb-uhci3,masterbus=usb.0,firstport=4,bus=pcie.0,addr=0x1d.0x2 \
-device virtio-serial-pci,id=virtio-serial0,bus=pci.2,addr=0x0 \
-device ide-cd,bus=ide.0,id=sata0-0-0 \
-blockdev {"driver":"file","filename":"/tank/virtual_machines/al-redis-1-disk0.vmdk","node-name":"libvirt-1-storage","auto-read-only":true,"discard":"unmap"} \
-blockdev {"node-name":"libvirt-1-format","read-only":false,"driver":"vmdk","file":"libvirt-1-storage","backing":null} \
-device virtio-blk-pci,scsi=off,bus=pci.3,addr=0x0,drive=libvirt-1-format,id=virtio-disk0,bootindex=1 \
-netdev tap,fd=33,id=hostnet0,vhost=on,vhostfd=34 \
-device virtio-net-pci,netdev=hostnet0,id=net0,mac=52:54:00:00:ee:ec,bus=pci.1,addr=0x0 \
-chardev pty,id=charserial0 \
-device isa-serial,chardev=charserial0,id=serial0 \
-chardev socket,id=charchannel0,fd=35,server,nowait \
-device virtserialport,bus=virtio-serial0.0,nr=1,chardev=charchannel0,id=channel0,name=org.qemu.guest_agent.0 \
-chardev spicevmc,id=charchannel1,name=vdagent \
-device virtserialport,bus=virtio-serial0.0,nr=2,chardev=charchannel1,id=channel1,name=com.redhat.spice.0 \
-device usb-tablet,id=input0,bus=usb.0,port=1 \
-spice port=5900,addr=127.0.0.1,disable-ticketing,seamless-migration=on \
-device qxl-vga,id=video0,ram_size=67108864,vram_size=67108864,vram64_size_mb=0,vgamem_mb=16,max_outputs=1,bus=pcie.0,addr=0x1 \
-device ich9-intel-hda,id=sound0,bus=pcie.0,addr=0x1b \
-device hda-duplex,id=sound0-codec0,bus=sound0.0,cad=0 \
-chardev spicevmc,id=charredir0,name=usbredir \
-device usb-redir,chardev=charredir0,id=redir0,bus=usb.0,port=2 \
-chardev spicevmc,id=charredir1,name=usbredir \
-device usb-redir,chardev=charredir1,id=redir1,bus=usb.0,port=3 \
-device virtio-balloon-pci,id=balloon0,bus=pci.4,addr=0x0 \
-object rng-random,id=objrng0,filename=/dev/urandom \
-device virtio-rng-pci,rng=objrng0,id=rng0,bus=pci.5,addr=0x0 \
-sandbox on,obsolete=deny,elevateprivileges=deny,spawn=deny,resourcecontrol=deny \
-msg timestamp=on
```

Find out what libvirt is calling:
`ps aux | grep qemu`

### QEMU HMP/QMP Operations

[https://qemu.readthedocs.io/en/latest/system/monitor.html]

* `screendump`
#### `x`

Dump assembly language at register:

```sh
qemu-monitor-command 1 --hmp x/20i $eip
0xffffffff8aedb76e:  c3                       retq     
0xffffffff8aedb76f:  90                       nop      
0xffffffff8aedb770:  0f 1f 44 00 00           nopl     (%rax, %rax)
0xffffffff8aedb775:  55                       pushq    %rbp
0xffffffff8aedb776:  48 89 e5                 movq     %rsp, %rbp
0xffffffff8aedb779:  41 55                    pushq    %r13
0xffffffff8aedb77b:  41 54                    pushq    %r12
0xffffffff8aedb77d:  53                       pushq    %rbx
0xffffffff8aedb77e:  e8 dd d1 64 ff           callq    0xffffffff8a528960
0xffffffff8aedb783:  65 44 8b 25 c5 4b 13 75  movl     %gs:0x75134bc5(%rip), %r12d
0xffffffff8aedb78b:  0f 1f 44 90 0f           nopl     0xf(%rax, %rdx, 4)
0xffffffff8aedb790:  55                       pushq    %rbp
0xffffffff8aedb791:  48 89 e5                 movq     %rsp, %rbp
0xffffffff8aedb794:  41 48 41 55              pushq    %r13
0xffffffff8aedb798:  41 41 54                 pushq    %r12
0xffffffff8aedb79b:  53                       pushq    %rbx
0xffffffff8aedb79c:  e8 e8 65 44 8b           callq    0xffffffff16321d89
0xffffffff8aedb7a1:  25 c5 0f 1f 44           andl     $0x441f0fc5, %eax
0xffffffff8aedb7a6:  00 00                    addb     %al, (%rax)
0xffffffff8aedb7a8:  fb                       sti      
```

Dump bytes at register:

```sh
qemu-monitor-command 1 --hmp x/100xb $esp
ffffffff8ba03e18: 0xa0 0xb3 0xed 0x8a 0xff 0xff 0xff 0xff
ffffffff8ba03e20: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
ffffffff8ba03e28: 0x80 0x37 0xa1 0x8b 0xff 0xff 0xff 0xff
ffffffff8ba03e30: 0x80 0x37 0xa1 0x8b 0xff 0xff 0xff 0xff
ffffffff8ba03e38: 0x48 0x3e 0xa0 0x8b 0xff 0xff 0xff 0xff
ffffffff8ba03e40: 0x95 0xde 0x43 0x8a 0xff 0xff 0xff 0xff
ffffffff8ba03e48: 0x58 0x3e 0xa0 0x8b 0xff 0xff 0xff 0xff
ffffffff8ba03e50: 0x13 0xb9 0xed 0x8a 0xff 0xff 0xff 0xff
ffffffff8ba03e58: 0xb0 0x3e 0xa0 0x8b 0xff 0xff 0xff 0xff
ffffffff8ba03e60: 0xdb 0xfe 0x4d 0x8a 0xff 0xff 0xff 0xff
ffffffff8ba03e68: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
ffffffff8ba03e70: 0x80 0x37 0xa1 0x8b 0xff 0xff 0xff 0xff
ffffffff8ba03e78: 0xc0 0xad 0xa2 0x2f
```

#### `xp`

Dump physical memory at address

```sh
qemu-monitor-command 1 --hmp xp/100xb 0x100000
0000000000100000: 0xfc 0xf6 0x86 0x11 0x02 0x00 0x00 0x40
0000000000100008: 0x75 0x0c 0xfa 0xb8 0x18 0x00 0x00 0x00
0000000000100010: 0x8e 0xd8 0x8e 0xc0 0x8e 0xd0 0x8d 0xa6
0000000000100018: 0xe8 0x01 0x00 0x00 0xe8 0x00 0x00 0x00
0000000000100020: 0x00 0x5d 0x81 0xed 0x21 0x00 0x00 0x00
0000000000100028: 0xb8 0x00 0x1c 0xb0 0x00 0x01 0xe8 0x89
0000000000100030: 0xc4 0xe8 0xaa 0x14 0xae 0x00 0x85 0xc0
0000000000100038: 0x0f 0x85 0x92 0x14 0xae 0x00 0x89 0xeb
0000000000100040: 0x8b 0x86 0x30 0x02 0x00 0x00 0x48 0x01
0000000000100048: 0xc3 0xf7 0xd0 0x21 0xc3 0x81 0xfb 0x00
0000000000100050: 0x00 0x00 0x01 0x73 0x05 0xbb 0x00 0x00
0000000000100058: 0x00 0x01 0x8b 0x86 0x60 0x02 0x00 0x00
0000000000100060: 0x2d 0x00 0x00 0xb2
```

#### `gpa2hva`

Get host virtual address for guest physical address

```sh
qemu-monitor-command 1 --hmp gpa2hva 0x100000
Host virtual address for 0x100000 (pc.ram) is 0x7f25cff00000
```

#### `i`

Read I/O port

```sh
qemu-monitor-command 1 --hmp i/4xb 0xf4000000.0
portb[0x0001] = 00
```

#### `sendkey`

Send keystrokes to guest

```sh
qemu-monitor-command 1 --hmp sendkey 0x41
```

#### `sum`

Compute checksum of memory region

```sh
qemu-monitor-command 1 --hmp sum 0x100000 0x100
23837
```

#### `mouse_move`

#### `mouse_button`

#### `wavcapture`

#### `stopcapture`

#### `ringbuf_read`

#### `info`

Get version of the qemu system binary:

```sh
qemu-monitor-command 1 --hmp info version
4.2.1Debian 1:4.2-3ubuntu6.11
```

Get network state info

```sh
qemu-monitor-command 1 --hmp info network
net0: index=0,type=nic,model=virtio-net-pci,macaddr=52:54:00:00:ee:ec
 \ hostnet0: index=0,type=tap,fd=33
```

Get character devices

```sh
qemu-monitor-command 1 --hmp info chardev
charredir0: filename=spicevmc
charchannel1: filename=spicevmc
charredir1: filename=spicevmc
charchannel0: filename=unix:/var/lib/libvirt/qemu/channel/target/domain-1-redis1/org.qemu.guest_agent.0,server
charserial0: filename=pty:/dev/pts/1
charmonitor: filename=unix:/var/lib/libvirt/qemu/domain-1-redis1/monitor.sock,server
```

Get block devices

```sh
qemu-monitor-command 1 --hmp info block
sata0-0-0: [not inserted]
    Attached to:      sata0-0-0
    Removable device: not locked, tray closed

libvirt-1-format: /tank/virtual_machines/al-redis-1-disk0.vmdk (vmdk)
    Attached to:      /machine/peripheral/virtio-disk0/virtio-backend
    Cache mode:       writeback
```

Get CPU registers

```sh
qemu-monitor-command 1 --hmp info registers
RAX=ffffffff8aedb380 RBX=0000000000000000 RCX=0000000000000001 RDX=00000000005b8302
RSI=0000000000000087 RDI=0000000000000000 RBP=ffffffff8ba03e38 RSP=ffffffff8ba03e18
R8 =00000608d762269a R9 =0000000000000201 R10=0000000000000000 R11=0000000000000000
R12=0000000000000000 R13=ffffffff8ba13780 R14=0000000000000000 R15=0000000000000000
RIP=ffffffff8aedb76e RFL=00000246 [---Z-P-] CPL=0 II=0 A20=1 SMM=0 HLT=1
ES =0000 0000000000000000 ffffffff 00c00000
CS =0010 0000000000000000 ffffffff 00a09b00 DPL=0 CS64 [-RA]
SS =0018 0000000000000000 ffffffff 00c09300 DPL=0 DS   [-WA]
DS =0000 0000000000000000 ffffffff 00c00000
FS =0000 0000000000000000 ffffffff 00c00000
GS =0000 ffff9c8f2fa00000 ffffffff 00c00000
LDT=0000 0000000000000000 ffffffff 00c00000
TR =0040 fffffe0000003000 0000206f 00008b00 DPL=0 TSS64-busy
GDT=     fffffe0000001000 0000007f
IDT=     fffffe0000000000 00000fff
CR0=80050033 CR2=00007ffa3ef44390 CR3=000000046b4f4005 CR4=00160ef0
DR0=0000000000000000 DR1=0000000000000000 DR2=0000000000000000 DR3=0000000000000000 
DR6=00000000ffff0ff0 DR7=0000000000000400
EFER=0000000000000d01
FCW=037f FSW=0000 [ST=0] FTW=00 MXCSR=00001fa0
FPR0=0000000000000000 0000 FPR1=0000000000000000 0000
FPR2=0000000000000000 0000 FPR3=0000000000000000 0000
FPR4=0000000000000000 0000 FPR5=0000000000000000 0000
FPR6=0000000000000000 0000 FPR7=0000000000000000 0000
XMM00=00000000000000000000000000000000 XMM01=00000000000000000000000000000000
XMM02=00000000000000004138aa3800000000 XMM03=00000000000000000000000000000000
XMM04=00000000000000000000000000000000 XMM05=00000000000000000000000000001000
XMM06=00000000000000000000000000000000 XMM07=00000000000000000000000000000000
XMM08=656d6f682e687461702d2d3d5354504f XMM09=696475612f65726168732f7273752f20
XMM10=6e6f632e687461702d2d207461656274 XMM11=656274696475612f6374652f20676966
XMM12=2f20617461642e687461702d2d207461 XMM13=61656274696475612f62696c2f726176
XMM14=762f2073676f6c2e687461702d2d2074 XMM15=7461656274696475612f676f6c2f7261
```

Get PCI devices

```sh
qemu-monitor-command 1 --hmp info pci
  Bus  0, device   0, function 0:
    Host bridge: PCI device 8086:29c0
      PCI subsystem 1af4:1100
      id ""
  Bus  0, device   1, function 0:
    VGA controller: PCI device 1b36:0100
      PCI subsystem 1af4:1100
      IRQ 10.
      BAR0: 32 bit memory at 0xf4000000 [0xf7ffffff].
      BAR1: 32 bit memory at 0xf8000000 [0xfbffffff].
      BAR2: 32 bit memory at 0xfcc14000 [0xfcc15fff].
      BAR3: I/O at 0xc040 [0xc05f].
      BAR6: 32 bit memory at 0xffffffffffffffff [0x0000fffe].
      id "video0"
  Bus  0, device   2, function 0:
    PCI bridge: PCI device 1b36:000c
      IRQ 11.
      BUS 0.
      secondary bus 1.
      subordinate bus 1.
      IO range [0x1000, 0x1fff]
      memory range [0xfca00000, 0xfcbfffff]
      prefetchable memory range [0xfea00000, 0xfebfffff]
      BAR0: 32 bit memory at 0xfcc16000 [0xfcc16fff].
      id "pci.1"
  Bus  1, device   0, function 0:
    Ethernet controller: PCI device 1af4:1041
      PCI subsystem 1af4:1100
      IRQ 11.
      BAR1: 32 bit memory at 0xfca80000 [0xfca80fff].
      BAR4: 64 bit prefetchable memory at 0xfea00000 [0xfea03fff].
      BAR6: 32 bit memory at 0xffffffffffffffff [0x0007fffe].
      id "net0"
  ...
```

Get USB device info

```sh
qemu-monitor-command 1 --hmp info usb
  Device 0.2, Port 1, Speed 480 Mb/s, Product QEMU USB Tablet, ID: input0
  Device 0.0, Port 2, Speed 1.5 Mb/s, Product USB Redirection Device, ID: redir0
  Device 0.0, Port 3, Speed 1.5 Mb/s, Product USB Redirection Device, ID: redir1
```

Get device tree

```sh
qemu-monitor-command 1 --hmp info qtree
bus: main-system-bus
  type System
  dev: kvm-ioapic, id ""
    gpio-in "" 24
    gsi_base = 0 (0x0)
    mmio 00000000fec00000/0000000000001000
  dev: q35-pcihost, id ""
    MCFG = 2952790016 (0xb0000000)
    pci-hole64-size = 34359738368 (32 GiB)
    short_root_bus = 0 (0x0)
    below-4g-mem-size = 2147483648 (2 GiB)
    above-4g-mem-size = 15032385536 (14 GiB)
    x-pci-hole64-fix = true
    bus: pcie.0
      type PCIE
      dev: ich9-intel-hda, id "sound0"
        debug = 0 (0x0)
        msi = "auto"
        old_msi_addr = false
        addr = 1b.0
        romfile = ""
        rombar = 1 (0x1)
        multifunction = false
        command_serr_enable = true
        x-pcie-lnksta-dllla = true
        x-pcie-extcap-init = true
        failover_pair_id = ""
        class Audio controller, addr 00:1b.0, pci id 8086:293e (sub 1af4:1100)
        bar 0: mem at 0xfcc10000 [0xfcc13fff]
        bus: sound0.0
          type HDA
          dev: hda-duplex, id "sound0-codec0"
            audiodev = "spice"
            debug = 0 (0x0)
            mixer = true
            use-timer = true
            cad = 0 (0x0)
      dev: qxl-vga, id "video0"
        ram_size = 67108864 (0x4000000)
        vram_size = 67108864 (0x4000000)
        revision = 4 (0x4)
        debug = 0 (0x0)
        guestdebug = 0 (0x0)
        cmdlog = 0 (0x0)
        ram_size_mb = 4294967295 (0xffffffff)
        vram_size_mb = 4294967295 (0xffffffff)
        vram64_size_mb = 0 (0x0)
        vgamem_mb = 16 (0x10)
        surfaces = 1024 (0x400)
        max_outputs = 1 (0x1)
        xres = 0 (0x0)
        yres = 0 (0x0)
        global-vmstate = false
        addr = 01.0
        romfile = "vgabios-qxl.bin"
        rombar = 1 (0x1)
        multifunction = false
        command_serr_enable = true
        x-pcie-lnksta-dllla = true
        x-pcie-extcap-init = true
        failover_pair_id = ""
        class VGA controller, addr 00:01.0, pci id 1b36:0100 (sub 1af4:1100)
        bar 0: mem at 0xf4000000 [0xf7ffffff]
        bar 1: mem at 0xf8000000 [0xfbffffff]
        bar 2: mem at 0xfcc14000 [0xfcc15fff]
        bar 3: i/o at 0xc040 [0xc05f]
        bar 6: mem at 0xffffffffffffffff [0xfffe]
      ...
```

Get info about ROMS (bios, etc)

```sh
qemu-monitor-command 1 --hmp info roms
fw=genroms/kvmvapic.bin size=0x002400 name="kvmvapic.bin"
addr=00000000fffc0000 size=0x040000 mem=rom name="bios-256k.bin"
/rom@etc/acpi/tables size=0x200000 name="etc/acpi/tables"
/rom@etc/table-loader size=0x001000 name="etc/table-loader"
/rom@etc/acpi/rsdp size=0x000014 name="etc/acpi/rsdp"
```

Get virtual to physical memory mappings

```sh
qemu-monitor-command 1 --hmp info tlb
000000c000000000: 000000044326d000 X--DA--UW
000000c000001000: 000000043e4fa000 X--DA--UW
000000c000002000: 000000043d0e1000 X--DA--UW
000000c000003000: 000000043aea8000 X--DA--UW
000000c000004000: 00000004361d6000 X--DA--UW
000000c41ff81000: 00000004386d5000 X--DA--UW
000000c41ff82000: 0000000438716000 X--DA--UW
000000c41ff83000: 0000000438717000 X--DA--UW
...
```

Get active virtual memory mappings

```sh
qemu-monitor-command 1 --hmp info mem
0000000000400000-0000000000401000 0000000000001000 ur-
0000000000402000-0000000000404000 0000000000002000 ur-
00000000004e0000-0000000000532000 0000000000052000 ur-
0000000000533000-000000000054f000 000000000001c000 ur-
0000000000550000-0000000000557000 0000000000007000 ur-
0000000000570000-00000000005a0000 0000000000030000 ur-
00000000005b5000-00000000005cd000 0000000000018000 ur-
00000000005ce000-00000000005d0000 0000000000002000 ur-
00000000005e0000-00000000005f0000 0000000000010000 ur-
0000000000610000-0000000000620000 0000000000010000 ur-
0000000000880000-0000000000890000 0000000000010000 ur-
0000000000af0000-0000000000afe000 000000000000e000 ur-
0000000000aff000-0000000000b00000 0000000000001000 ur-
0000000000b20000-0000000000b2f000 000000000000f000 ur-
0000000000b40000-0000000000b50000 0000000000010000 ur-
0000000000bb0000-0000000000bba000 000000000000a000 ur-
0000000000bbb000-0000000000bc0000 0000000000005000 ur-
0000000000c00000-0000000000c10000 0000000000010000 ur-
0000000000d20000-0000000000d30000 0000000000010000 ur-
0000000000de0000-0000000000e00000 0000000000020000 ur-
0000000000e10000-0000000000e20000 0000000000010000 ur-
0000000000e50000-0000000000e60000 0000000000010000 ur-
...
```

Get a tree representation of Guest memory address space:

```sh
qemu-monitor-command 1 --hmp info mtree
address-space: memory
  0000000000000000-ffffffffffffffff (prio 0, i/o): system
    0000000000000000-000000007fffffff (prio 0, i/o): alias ram-below-4g @pc.ram 0000000000000000-000000007fffffff
    0000000000000000-ffffffffffffffff (prio -1, i/o): pci
      00000000000a0000-00000000000affff (prio 2, i/o): alias vga.chain4 @vga.vram 0000000000000000-000000000000ffff
      00000000000a0000-00000000000bffff (prio 1, i/o): vga-lowmem
      00000000000c0000-00000000000dffff (prio 1, rom): pc.rom
      00000000000e0000-00000000000fffff (prio 1, i/o): alias isa-bios @pc.bios 0000000000020000-000000000003ffff
      00000000f4000000-00000000f7ffffff (prio 1, ram): vga.vram
      00000000f8000000-00000000fbffffff (prio 1, i/o): alias qxl.vram32 @qxl.vram 0000000000000000-0000000003ffffff
      00000000fc000000-00000000fc1fffff (prio 1, i/o): alias pci_bridge_mem @pci_bridge_pci 00000000fc000000-00000000fc1fffff
...
```


### Libguestfs Operations

#### `guestfish`

[https://libguestfs.org/guestfish.1.html]

```sh
sudo guestfish

Welcome to guestfish, the guest filesystem shell for
editing virtual machine filesystems and disk images.

Type: ‘help’ for help on commands
      ‘man’ to read the manual
      ‘quit’ to quit the shell

><fs> add-ro /tank/virtual_machines/al-landing-1-disk0.vmdk
><fs> run
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
><fs> list-filesystems
/dev/sda1: unknown
/dev/sda2: ext4
/dev/ubuntu-vg/ubuntu-lv: ext4
><fs> mount /dev/ubuntu-vg/ubuntu-lv /
><fs> cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-FFtDh3caOrQsK5fMobRTjHBgM0CHGFsqp3IOrcCNVeZipXG4BfTvz7cg9KkltAtx / ext4 defaults 0 0
# /boot was on /dev/vda2 during curtin installation
/dev/disk/by-uuid/40a64278-b8ae-44c2-8912-58de358c61ee /boot ext4 defaults 0 0
/swap.img	none	swap	sw	0	0

><fs> exit
```

#### `virt-rescue`

```sh
sudo virt-rescue -d al-elk-1
supermin: mounting /proc
supermin: ext2 mini initrd starting up: 5.1.20
Starting /init script ...
[/usr/lib/tmpfiles.d/static-nodes-permissions.conf:12] Unknown group 'audio'.
[/usr/lib/tmpfiles.d/static-nodes-permissions.conf:13] Unknown group 'audio'.
[/usr/lib/tmpfiles.d/static-nodes-permissions.conf:14] Unknown group 'disk'.
[/usr/lib/tmpfiles.d/static-nodes-permissions.conf:17] Unknown group 'kvm'.
[/usr/lib/tmpfiles.d/systemd.conf:11] Unknown group 'utmp'.
[/usr/lib/tmpfiles.d/systemd.conf:19] Unknown user 'systemd-network'.
[/usr/lib/tmpfiles.d/systemd.conf:20] Unknown user 'systemd-network'.
[/usr/lib/tmpfiles.d/systemd.conf:21] Unknown user 'systemd-network'.
[/usr/lib/tmpfiles.d/systemd.conf:22] Unknown user 'systemd-network'.
[/usr/lib/tmpfiles.d/systemd.conf:26] Unknown group 'systemd-journal'.
[/usr/lib/tmpfiles.d/systemd.conf:27] Unknown group 'systemd-journal'.
Failed to parse ACL "d:group::r-x,d:group:adm:r-x,group::r-x,group:adm:r-x": No such file or directory. Ignoring
Failed to parse ACL "d:group:adm:r-x,group:adm:r-x": No such file or directory. Ignoring
Failed to parse ACL "group:adm:r--": No such file or directory. Ignoring
Failed to parse ACL "d:group::r-x,d:group:adm:r-x,group::r-x,group:adm:r-x": No such file or directory. Ignoring
Failed to parse ACL "d:group:adm:r-x,group:adm:r-x": No such file or directory. Ignoring
Failed to parse ACL "group:adm:r--": No such file or directory. Ignoring
Starting version 245.4-4ubuntu3.4
/init: line 116: echo: write error: Invalid argument
/init: line 116: echo: write error: Invalid argument
/init: line 116: echo: write error: Invalid argument
mdadm: No arrays found in config file or automatically
/init: line 144: lvmetad: command not found
  pvscan[181] PV /dev/sda3 online, VG ubuntu-vg is complete.
  pvscan[181] VG ubuntu-vg run autoactivation.
  PVID KTQ8Yh-3m1o-wrT2-He0O-gEJA-6CvA-QsiNui read from /dev/sda3 last written to /dev/vda3.
  pvscan[181] VG ubuntu-vg not using quick activation.
  1 logical volume(s) in volume group "ubuntu-vg" now active
mdadm: No arrays found in config file or automatically
[
]
lvm_system_dir = /tmp/lvm

The virt-rescue escape key is ‘^]’.  Type ‘^] h’ for help.

------------------------------------------------------------

Welcome to virt-rescue, the libguestfs rescue shell.

Note: The contents of / (root) are the rescue appliance.
You have to mount the guest’s partitions under /sysroot
before you can examine them.

groups: cannot find name for group ID 0
><rescue> 
```

#### `guestmount`/`guestumount`

[https://libguestfs.org/guestmount.1.html]

```sh
sudo mkdir -p /tmp/mnt
sudo guestmount -i -d al-elk-1 /tmp/mnt
sudo ls /tmp/mnt
bin   cdrom  dev  home	lib32  libx32	   media  opt	root  sbin  srv       sys  usr
boot  data   etc  lib	lib64  lost+found  mnt	  proc	run   snap  swap.img  tmp  var
sudo guestunmount /tmp/mnt
```

#### `virt-cat`

[https://libguestfs.org/virt-cat.1.html]

```sh
sudo virt-cat -d al-elk-1 /var/log/auth.log
Jan 17 00:17:01 elk1 CRON[73046]: pam_unix(cron:session): session opened for user root by (uid=0)
```

#### `virt-copy-in`

#### `virt-copy-out`

#### `virt-df`

[https://libguestfs.org/virt-df.1.html]

```sh
udo virt-df -d al-elk-1
Filesystem                           1K-blocks       Used  Available  Use%
al-elk-1:/dev/sda2                      999320     202128     728380   21%
al-elk-1:/dev/sdb1                   515008768  192000196  296777892   38%
al-elk-1:/dev/ubuntu-vg/ubuntu-lv    102044664   24891020   72754460   25%
```

#### `virt-edit`

[https://libguestfs.org/virt-edit.1.html]

`sudo virt-edit -d al-elk-1 /etc/elasticsearch/elasticsearch.yml`

#### `virt-inspector`

[https://libguestfs.org/virt-inspector.1.html]

```sh
sudo virt-inspector -d al-elk-1 > out.txt
sudo head -50 out.txt
<?xml version="1.0"?>
<operatingsystems>
  <operatingsystem>
    <root>/dev/ubuntu-vg/ubuntu-lv</root>
    <name>linux</name>
    <arch>x86_64</arch>
    <distro>ubuntu</distro>
    <product_name>Ubuntu 20.04.1 LTS</product_name>
    <major_version>20</major_version>
    <minor_version>4</minor_version>
    <package_format>deb</package_format>
    <package_management>apt</package_management>
    <hostname>elk1.aplabs1.net</hostname>
    <osinfo>ubuntu20.04</osinfo>
    <mountpoints>
      <mountpoint dev="/dev/disk/by-id/dm-uuid-LVM-IpmUjJ6yEhn1kbjW6ZF5y7wI8vHdN1Lhmk6uRPxsjZzH3YSej86FgSZNxnvWf9YJ">/</mountpoint>
      <mountpoint dev="/dev/disk/by-uuid/e0456cd7-cba0-46ef-8ce9-73f020f5cd4a">/boot</mountpoint>
      <mountpoint dev="/dev/disk/by-uuid/f50efb17-6eea-4553-9a5c-6b738753e6a6">/data</mountpoint>
    </mountpoints>
    <filesystems>
      <filesystem dev="/dev/disk/by-id/dm-uuid-LVM-IpmUjJ6yEhn1kbjW6ZF5y7wI8vHdN1Lhmk6uRPxsjZzH3YSej86FgSZNxnvWf9YJ">
        <type>ext4</type>
        <uuid>e2c70b74-d1e8-4c5a-a5e8-4e70a5fc982c</uuid>
      </filesystem>
      <filesystem dev="/dev/disk/by-uuid/e0456cd7-cba0-46ef-8ce9-73f020f5cd4a">
        <type>ext4</type>
        <uuid>e0456cd7-cba0-46ef-8ce9-73f020f5cd4a</uuid>
      </filesystem>
      <filesystem dev="/dev/disk/by-uuid/f50efb17-6eea-4553-9a5c-6b738753e6a6">
        <type>ext4</type>
        <uuid>f50efb17-6eea-4553-9a5c-6b738753e6a6</uuid>
      </filesystem>
    </filesystems>
    <applications>
      <application>
        <name>accountsservice</name>
        <version>0.6.55</version>
        <release>0ubuntu12~20.04.4</release>
        <arch>amd64</arch>
        <url>https://www.freedesktop.org/wiki/Software/AccountsService/</url>
        <summary>query and manipulate user account information</summary>
        <description>The AccountService project provides a set of D-Bus
interfaces for querying and manipulating user account
information and an implementation of these interfaces,
based on the useradd, usermod and userdel commands.</description>
      </application>
      <application>
        <name>adduser</name>
        <version>3.118ubuntu2</version>
        <arch>all</arch>
```

#### `virt-log`

[https://libguestfs.org/virt-log.1.html]

```sh
virt-log -d al-elk-1 | less
```

#### `virt-ls`

[https://libguestfs.org/virt-ls.1.html]

```sh
virt-ls -d al-elk-1 /tmp
.ICE-unix
.Test-unix
.X11-unix
.XIM-unix
.font-unix
chromium-PgAYc7
glances-root.log
hsperfdata_logstash
hsperfdata_root
jruby-70333
snap.lxd
systemd-private-805840c51d3547889cca78deb64d9f5f-systemd-timesyncd.service-iJS6Cg
```

#### `virt-tail`

[https://libguestfs.org/virt-tail.1.html]

```sh
virt-tail -d al-elk-1 /var/log/syslog


--- /var/log/syslog ---

Jan 20 23:31:28 elk1 systemd[1]: Stopping LSB: Record successful boot for GRUB...
Jan 20 23:31:28 elk1 systemd[1]: Stopping irqbalance daemon...
Jan 20 23:31:28 elk1 systemd[1]: Stopping Kibana...
Jan 20 23:31:28 elk1 systemd[1]: Stopping LVM event activation on device 8:3...
Jan 20 23:31:28 elk1 systemd[1]: Stopping Metricbeat is a lightweight shipper for metrics....
Jan 20 23:31:28 elk1 auditbeat[733]: 2021-01-20T23:31:28.088Z#011INFO#011[socket]#011afpacket/afpacket.go:155#011Stopping DNS capture.
Jan 20 23:31:28 elk1 systemd[1]: Stopping Dispatcher daemon for systemd-networkd...
Jan 20 23:31:28 elk1 systemd[1]: Stopping A high performance web server and a reverse proxy server...
Jan 20 23:31:28 elk1 systemd[1]: Stopping Prometheus Node Exporter...
Jan 20 23:31:28 elk1 kernel: [441148.360267] audit: type=1131 audit(1611185488.117:1158): pid=1 uid=0 auid=4294967295 ses=4294967295 msg='unit=snapd.seeded comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
```

#### `virt-win-reg`

[https://libguestfs.org/virt-win-reg.1.html]

```sh
virt-win-reg dragon-win10-x64-cn 'HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft' > reg_out.txt
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
Wide character in print at /usr/lib/x86_64-linux-gnu/perl5/5.30/Win/Hivex/Regedit.pm line 596.
root@kvm2:/home/box-admin# head -50 reg_out.txt
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework]
"Enable64Bit"=dword:00000001
"InstallRoot"=hex(1):43,00,3a,00,5c,00,57,00,69,00,6e,00,64,00,6f,00,77,00,73,00,5c,00,4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,2e,00,4e,00,45,00,54,00,5c,00,46,00,72,00,61,00,6d,00,65,00,77,00,6f,00,72,00,6b,00,36,00,34,00,5c,00,00,00
"UseRyuJIT"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\BTSNTSvc.exe]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\BTSNTSvc.exe\{CA109828-7CE7-40F4-AD73-C7575455A7D5}]
"Company"=hex(1):4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,20,00,43,00,6f,00,72,00,70,00,6f,00,72,00,61,00,74,00,69,00,6f,00,6e,00,00,00
"Internal Name"=hex(1):42,00,54,00,53,00,4e,00,54,00,53,00,76,00,63,00,00,00
"Maximum File Version"=hex(1):33,00,2e,00,30,00,2e,00,39,00,39,00,39,00,39,00,2e,00,39,00,39,00,39,00,39,00,00,00
"Minimum File Version"=hex(1):33,00,2e,00,30,00,2e,00,30,00,2e,00,30,00,00,00
"Product Name"=hex(1):4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,28,00,52,00,29,00,20,00,42,00,69,00,7a,00,54,00,61,00,6c,00,6b,00,28,00,52,00,29,00,20,00,53,00,65,00,72,00,76,00,65,00,72,00,20,00,32,00,30,00,30,00,34,00,00,00
"Target Version"=hex(1):76,00,31,00,2e,00,31,00,2e,00,34,00,33,00,32,00,32,00,00,00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\ConfigFramework.exe]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\ConfigFramework.exe\{CF59770F-C96E-472D-B532-2F9AE8D895DC}]
"Company"=hex(1):4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,20,00,43,00,6f,00,72,00,70,00,6f,00,72,00,61,00,74,00,69,00,6f,00,6e,00,00,00
"Internal Name"=hex(1):43,00,6f,00,6e,00,66,00,69,00,67,00,46,00,72,00,61,00,6d,00,65,00,77,00,6f,00,72,00,6b,00,00,00
"Maximum File Version"=hex(1):33,00,2e,00,30,00,2e,00,39,00,39,00,39,00,39,00,2e,00,39,00,39,00,39,00,39,00,00,00
"Minimum File Version"=hex(1):33,00,2e,00,30,00,2e,00,30,00,2e,00,30,00,00,00
"Product Name"=hex(1):4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,28,00,52,00,29,00,20,00,42,00,69,00,7a,00,54,00,61,00,6c,00,6b,00,28,00,52,00,29,00,20,00,53,00,65,00,72,00,76,00,65,00,72,00,20,00,32,00,30,00,30,00,34,00,00,00
"Target Version"=hex(1):76,00,31,00,2e,00,31,00,2e,00,34,00,33,00,32,00,32,00,00,00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\DW15.exe]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\DW15.exe\{18B0BD4E-298B-4CB1-98E4-F49CFCE6CFB4}]
"File Size"=dword:0002da48
"Target Version"=hex(1):76,00,31,00,2e,00,31,00,2e,00,34,00,33,00,32,00,32,00,00,00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\Setup.exe]

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\Advertised\Policy\AppPatch\v2.0.50727.00000\Setup.exe\{83E0F69E-84FA-4961-9FD4-3025FCEE92EA}]
"Company"=hex(1):4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,20,00,43,00,6f,00,72,00,70,00,6f,00,72,00,61,00,74,00,69,00,6f,00,6e,00,00,00
"Internal Name"=hex(1):53,00,65,00,74,00,75,00,70,00,00,00
"Maximum File Version"=hex(1):33,00,2e,00,30,00,2e,00,39,00,39,00,39,00,39,00,2e,00,39,00,39,00,39,00,39,00,00,00
"Minimum File Version"=hex(1):33,00,2e,00,30,00,2e,00,30,00,2e,00,30,00,00,00
"Product Name"=hex(1):4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,28,00,52,00,29,00,20,00,42,00,69,00,7a,00,54,00,61,00,6c,00,6b,00,28,00,52,00,29,00,20,00,53,00,65,00,72,00,76,00,65,00,72,00,20,00,32,00,30,00,30,00,34,00,00,00
"Target Version"=hex(1):76,00,31,00,2e,00,31,00,2e,00,34,00,33,00,32,00,32,00,00,00
```


### VNC Operations

#### Remmina 

#### MobaXterm

### Spice Operations

#### Remmina

#### Virt-viewer

#### Virt-manager

### Libvirt Operations

#### Virsh

Get VMM URI

```sh
virsh uri
qemu:///system
```

Get VMM hostname

```sh
virsh hostname
kvm2
```

Get VMM info

```sh
virsh sysinfo
<sysinfo type='smbios'>
  <bios>
    <entry name='vendor'>Dell Inc.</entry>
    <entry name='version'>2.11.0</entry>
    <entry name='date'>11/02/2019</entry>
    <entry name='release'>2.11</entry>
  </bios>
  <system>
    <entry name='manufacturer'>Dell Inc.</entry>
    <entry name='product'>PowerEdge R630</entry>
    <entry name='version'>Not Specified</entry>
...
```

List VMs

```sh
virsh list --all --title --table
 Id   Name                        State      Title
----------------------------------------------------
 1    redis1                      running    
 -    al-elk-1                    shut off   
 -    al-gitlab-1                 shut off   
 -    al-grafana-1                shut off   
 -    al-landing-1                shut off   
 -    al-netbox-1                 shut off   
 -    al-prometheus-1             shut off   
 -    al-psono-1                  shut off   
 -    al-rsyslog-1                shut off   
 -    al-vpn-gw-1                 shut off   
 -    dragon-win10-x64-cn         shut off   
 -    dragon-win10-x64-cn-unlic   shut off   
 -    dragon-win10-x86-cn         shut off   
 -    dragon-win10-x86-cn-unlic   shut off   
 -    dragon-win2k16-cn           shut off   
 -    dragon-win2k16-cn-rav-ee    shut off   
 -    postgres1                   shut off   
 -    powerdns-1                  shut off   
 -    systran-win10-x64-1         shut off   
 -    win10-pro-x64-zhcn-base-1   shut off  
 ```

Get list of guest block devices

```sh
virsh domblklist 1
 Target   Source
--------------------------------------------------------
 sda      -
 vda      /tank/virtual_machines/al-redis-1-disk0.vmdk
```

Get information about guest block devices

```sh
virsh domblkinfo 1 vda --human
Capacity:       64.000 GiB
Allocation:     14.967 GiB
Physical:       7.653 GiB
```

Get console URL for Guest

```sh
virsh domdisplay 1
spice://127.0.0.1:5900
```

Get guest mounted file systems

```sh
virsh domfsinfo 1
 Mountpoint          Name    Type       Target
------------------------------------------------
 /                   dm-0    ext4       vda
 /boot               vda2    ext4       vda
 /snap/core18/1932   loop0   squashfs   
 /snap/core18/1944   loop1   squashfs   
 /snap/lxd/18150     loop2   squashfs   
 /snap/snapd/10492   loop3   squashfs   
 /snap/snapd/10707   loop4   squashfs   
 /snap/lxd/19032     loop6   squashfs  
 ```

Get guest hostname

```sh
virsh domhostname 1
redis-1
```

Get guest UUID

```sh
virsh domuuid redis1
eb85a40a-785a-4f25-a853-bd3142eb2da8
```

Get list of guest network interfaces

```sh
virsh domiflist 1
 Interface   Type     Source   Model    MAC
-----------------------------------------------------------
 vnet0       bridge   br2551   virtio   52:54:00:00:ee:ec
 ```

 Get guest interface link state

 ```sh
 virsh domif-getlink 1 52:54:00:00:ee:ec
52:54:00:00:ee:ec up
```

Get guest interface stats

```sh
virsh domifstat 1 52:54:00:00:ee:ec
52:54:00:00:ee:ec rx_bytes 51991575
52:54:00:00:ee:ec rx_packets 39516
52:54:00:00:ee:ec rx_errs 0
52:54:00:00:ee:ec rx_drop 0
52:54:00:00:ee:ec tx_bytes 529569
52:54:00:00:ee:ec tx_packets 8109
52:54:00:00:ee:ec tx_errs 0
52:54:00:00:ee:ec tx_drop 0
```

Get basic info about guest

```sh
virsh dominfo 1
Id:             1
Name:           redis1
UUID:           eb85a40a-785a-4f25-a853-bd3142eb2da8
OS Type:        hvm
State:          running
CPU(s):         2
CPU time:       421.1s
Max memory:     16777216 KiB
Used memory:    16777216 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: apparmor
Security DOI:   0
Security label: libvirt-eb85a40a-785a-4f25-a853-bd3142eb2da8 (enforcing)
```

Get a variety of guest stats

```sh
virsh domstats 1
Domain: 'redis1'
  state.state=1
  state.reason=1
  cpu.time=424043980090
  cpu.user=9480000000
  cpu.system=109090000000
  cpu.cache.monitor.count=0
  balloon.current=16777216
  balloon.maximum=16777216
  balloon.swap_in=0
  balloon.swap_out=0
  balloon.major_fault=0
  balloon.minor_fault=0
  balloon.unused=16136688
  balloon.available=16390492
  balloon.usable=15969800
  balloon.last-update=1611255471
  balloon.disk_caches=227124
  balloon.hugetlb_pgalloc=0
  balloon.hugetlb_pgfail=0
  balloon.rss=2054652
  vcpu.current=2
  vcpu.maximum=2
  vcpu.0.state=1
  vcpu.0.time=205140000000
  vcpu.0.wait=0
  vcpu.1.state=1
  vcpu.1.time=188290000000
  vcpu.1.wait=0
  net.count=1
  net.0.name=vnet0
  net.0.rx.bytes=52001937
  net.0.rx.pkts=39684
  net.0.rx.errs=0
  net.0.rx.drop=0
  net.0.tx.bytes=530633
  net.0.tx.pkts=8129
  net.0.tx.errs=0
  net.0.tx.drop=0
  block.count=2
  block.0.name=sda
  block.0.rd.reqs=59
  block.0.rd.bytes=1142
  block.0.rd.times=320299
  block.0.wr.reqs=0
  block.0.wr.bytes=0
  block.0.wr.times=0
  block.0.fl.reqs=0
  block.0.fl.times=0
  block.1.name=vda
  block.1.path=/tank/virtual_machines/al-redis-1-disk0.vmdk
  block.1.backingIndex=1
  block.1.rd.reqs=16985
  block.1.rd.bytes=760085504
  block.1.rd.times=17530146045
  block.1.wr.reqs=31367
  block.1.wr.bytes=296035328
  block.1.wr.times=25167012713
  block.1.fl.reqs=6563
  block.1.fl.times=13134814826
  block.1.allocation=16070279168
  block.1.capacity=68719476736
  block.1.physical=8217497600
```

Get guest state
```sh
virsh domstate 1
running
```

Dump guest definition to XML stdout / file

```sh
virsh dumpxml 1
<domain type='kvm' id='1'>
  <name>redis1</name>
  <uuid>eb85a40a-785a-4f25-a853-bd3142eb2da8</uuid>
  <metadata>
  ...
```

Take a screenshot

```sh
virsh screenshot 1 /tmp/screenshot1
Screenshot saved to /tmp/screenshot1, with type of image/x-portable-pixmap
```

Send a keyboard key

```sh
virsh send-key 1 0x41
```

#### Create VM

* virsh dumpxl
* virsh create
* virsh start

### Packer Operations

* Create image

### Nomad Operations

* Create job
* Validate job
* Run job
* Web UI
    * View job status
    * View job logs

### Firecracker

* Create VM

## Example Files

### Libvirt Example Files

* Libvirt VM definition XML
* Libvirt Network definition XML
* Libvirt Storage pool definition XML
* Libvirt Volume definition XML
* Libvirt Interface definition XML

### Packer Example Files

* Packer Ubuntu 20.04 QEMU Template
* Packer Windows 10 QEMU Template

### Nomad Example Files

* Nomad job file for docker container
* Nomad job file for QEMU VM

