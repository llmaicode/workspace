Good choice—QEMU + KVM is the right way to get real isolation, snapshots, and clean rollback.

Below is a **reproducible VM lab setup on Debian**, including:

* KVM/QEMU install
* libvirt setup
* bridge/network basics
* a VM definition approach (cloud-init based, reproducible)
* snapshot + rollback workflow
* Makefile automation
* optional audit inside VM

---

# 1. Install KVM/QEMU + tooling (host)

```bash
sudo apt update
sudo apt install -y \
  qemu-kvm \
  libvirt-daemon-system \
  libvirt-clients \
  virtinst \
  bridge-utils \
  cpu-checker \
  cloud-image-utils \
  genisoimage \
  make
```

Check virtualization:

```bash
kvm-ok
```

Enable and start libvirt:

```bash
sudo systemctl enable --now libvirtd
```

Add your user to groups:

```bash
sudo usermod -aG libvirt,kvm $USER
newgrp libvirt
```

---

# 2. Verify KVM works

```bash
virsh list --all
```

If no errors → good.

---

# 3. Create working directory (infra-as-code style)

```bash
mkdir -p ~/vm-lab/{images,cloud-init,vm,iso,snapshots,scripts}
cd ~/vm-lab
```

---

# 4. Download Debian cloud image

We use cloud images because they are reproducible.

```bash
cd images

wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
```

Rename:

```bash
mv debian-12-generic-amd64.qcow2 base.qcow2
```

---

# 5. Create VM disk (copy-on-write)

```bash
qemu-img create -f qcow2 -b base.qcow2 vm.qcow2 20G
```

This ensures:

* base image stays clean
* VM changes are isolated

---

# 6. Cloud-init config (reproducible VM user setup)

## meta-data

```bash
cat > cloud-init/meta-data <<EOF
instance-id: vm1
local-hostname: untrusted-vm
EOF
```

## user-data

```bash
cat > cloud-init/user-data <<EOF
#cloud-config
users:
  - name: untrusted
    sudo: ALL=(ALL) ALL
    groups: users
    shell: /bin/bash
    ssh_authorized_keys: []

ssh_pwauth: true
disable_root: true

packages:
  - sudo
  - vim
  - htop
  - curl
  - git

runcmd:
  - echo "VM initialized"
EOF
```

Build seed ISO:

```bash
cloud-localds iso/seed.iso cloud-init/user-data cloud-init/meta-data
```

---

# 7. First VM boot (manual test)

```bash
qemu-system-x86_64 \
  -enable-kvm \
  -m 4096 \
  -smp 2 \
  -cpu host \
  -drive file=images/vm.qcow2,format=qcow2 \
  -drive file=iso/seed.iso,format=raw \
  -net nic -net user \
  -nographic
```

---

# 8. Better: libvirt VM definition (recommended)

Create VM:

```bash
virt-install \
  --name untrusted-vm \
  --memory 4096 \
  --vcpus 2 \
  --cpu host \
  --disk path=$HOME/vm-lab/images/vm.qcow2,format=qcow2,bus=virtio \
  --disk path=$HOME/vm-lab/iso/seed.iso,device=cdrom \
  --os-variant debian12 \
  --network network=default \
  --graphics spice \
  --import
```

---

# 9. Snapshots (your main safety tool)

Create snapshot before giving access:

```bash
virsh snapshot-create-as untrusted-vm clean-state "before untrusted access"
```

List snapshots:

```bash
virsh snapshot-list untrusted-vm
```

Rollback:

```bash
virsh snapshot-revert untrusted-vm clean-state
```

Delete snapshot:

```bash
virsh snapshot-delete untrusted-vm clean-state
```

---

# 10. Makefile (your “infra control panel”)

Create:

```bash
cat > Makefile <<'EOF'
VM=untrusted-vm

init:
	@echo "Creating snapshot baseline..."
	virsh snapshot-create-as $(VM) baseline "clean state"

start:
	virsh start $(VM)

stop:
	virsh shutdown $(VM)

reboot:
	virsh reboot $(VM)

console:
	virsh console $(VM)

snap:
	@echo "Creating snapshot..."
	virsh snapshot-create-as $(VM) snap-$(shell date +%F-%H%M%S) "manual snapshot"

list-snap:
	virsh snapshot-list $(VM)

rollback:
	@echo "Reverting to baseline..."
	virsh snapshot-revert $(VM) baseline

destroy:
	virsh destroy $(VM)

remove:
	virsh undefine $(VM) --remove-all-storage || true
EOF
```

---

# 11. Usage workflow (important)

## Before giving access:

```bash
make init
```

## Start VM:

```bash
make start
```

## Give access inside VM

User works inside VM, not host.

## During session:

```bash
make snap
```

## If something suspicious:

```bash
make rollback
```

---

# 12. Optional: stronger isolation (recommended upgrades)

If you want real “untrusted user” safety:

### A) Disable host access completely

* no shared folders
* no clipboard
* no USB passthrough

### B) Add strict network isolation

Replace default NAT with no network:

```bash
--network none
```

or firewall VM traffic:

```bash
sudo apt install ufw
```

---

# 13. Optional inside-VM auditing (if needed)

Inside VM:

```bash
sudo apt install auditd
sudo systemctl enable --now auditd
```

Command logging:

```bash
sudo auditctl -a always,exit -F arch=b64 -S execve
```

---

# 14. What you now have

You now have:

* reproducible VM (cloud-init)
* full snapshot/rollback system
* infra controlled via Makefile
* clean separation between host and untrusted user

---

# If you want next upgrade

I can extend this into:

### “Paranoid mode lab”

* auto-destroy VM after session
* immutable base image pipeline
* Git-controlled VM definitions (like Terraform but for QEMU)
* network egress monitoring
* full session recording (tty + keystrokes inside VM)

Just tell me.
