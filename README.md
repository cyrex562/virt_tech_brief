# virt_tech_brief

A brief on emerging tech to build a modern infrastructure platform

## Info

See "Virtualization Tech Briefing.pptx" for the briefing slides and notes

## Demos

### qemu-img Operations

* `qemu-img convert`

* `qemu-img info`

### qemu-system Operations

### QEMU HMP/QMP Operations

* `screendump`
* `x`
* `xp`
* `gpa2hpa`
* `gpa2hva`
* `i`
* `sendkey`
* `sum`
* `mouse_move`
* `mouse_button`
* `wavcapture`
* `stopcapture`
* `memsave`
* `pmemsave`
* `ringbuf_write`
* `ringbuf_read`
* `dump-guest-memory`
* `info`

### Libguestfs Operations

* `guestfish`
* `virt-rescue`
* `guestmount`/`guestumount`
* `virt-cat`
* `virt-copy-in`
* `virt-copy-out`
* `virt-df`
* `virt-edit`
* `virt-inspector`
* `virt-log`
* `virt-ls`
* `virt-tail`
* `virt-win-reg`
* `hivexml`

### VNC Operations

* Launch Remmina / MobaXterm

### Spice Operations

* Launch Remmina / MobaXterm
* Launch Virt-viewer
* Launch Virt-manager

### Libvirt Operations

* Create VM
* Virsh
    * VM operations
    * Network operations
    * Storage pool operations
    * Volume operations
    * Snaphost operations
    * Interface operations

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

