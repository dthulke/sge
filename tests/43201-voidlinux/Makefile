Port:=$(shell basename $$PWD |cut -d - -f 1)
DaikerOpts:=-D $(shell [ -n "$$DISPLAY" ] && echo gtk || echo vnc) -c 8 -r 2 -T 22-$(Port)
Ssh:=ssh -p $(Port) -i var/id_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ConnectTimeout=1
var/voidlinux-sge-container : var/voidlinux-sge-image
	fuser -k $@.qcow2 $(Port)/tcp && sleep 2 || :
	rm -f $@.qcow2
	var/daiker run $(DaikerOpts) -b $<.qcow2 $@.qcow2 &
	for ((i=1; ; i++)); do ! $(Ssh) root@localhost id || break; sleep 2; echo "MSG-002: Retrying ssh $$i"; done
	rsync -ave "$(Ssh)" --exclude tests --exclude .git ../../ root@localhost:sge
	$(Ssh) root@localhost < lib/voidlinux.sh 2>&1 | tee $@.log
	touch $@
	$(Ssh) root@localhost <<< poweroff || :
var/voidlinux-sge-image : var/voidlinux-sge.iso var/void-rootfs-latest-x86_64.tar.xz var/daiker
	fuser -k $@.qcow2 $(Port)/tcp && sleep 2 || :
	rm -f $@.qcow2
	var/daiker build $(DaikerOpts) -i $< -H 20 $@.qcow2 &
	for ((i=1; ; i++)); do ! $(Ssh) root@localhost id || break; sleep 2; echo "MSG-001: Retrying ssh $$i"; done
	scp -P $(Port) -i var/id_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ConnectTimeout=1 var/void-rootfs-latest-x86_64.tar.xz root@localhost:
	$(Ssh) root@localhost < lib/voidlinux-install.sh
	while fuser $@.qcow2 $(Port)/tcp; do sleep 2; done
	touch $@
var/voidlinux-sge.iso : var/void-live-latest-x86_64.iso var/id_ed25519
	[ ! -d $@.d ] || chmod -R u+wX $@.d && rm -rf $@.d
	mkdir -p $@.d $@.m
	bsdtar xfp $< -C$@.d
	chmod -R u+wX $@.d
	cd $@.d/LiveOS && unsquashfs -no-xattrs squashfs.img
	sudo mount $@.d/LiveOS/squashfs-root/LiveOS/ext3fs.img $@.m
	sudo mkdir $@.m/root/.ssh
	cat var/id_ed25519.pub | sudo tee $@.m/root/.ssh/authorized_keys
	sudo chmod 700 $@.m/root/.ssh
	sudo sed -ie "s#/root:/bin/sh#/root:/bin/bash#" $@.m/etc/passwd
	sudo rm $@.m/etc/passwde
	sudo umount $@.m
	cd $@.d/LiveOS && rm squashfs.img && mksquashfs squashfs-root squashfs.img
	rm -r $@.d/LiveOS/squashfs-root
	genisoimage -quiet -l -r -J -V "$$(isoinfo -d -i $< | grep 'Volume id: ' |cut -c 12-)" -no-emul-boot -boot-load-size 4 -boot-info-table -b boot/isolinux/isolinux.bin -c boot/isolinux/boot.cat -o $@.tmp $@.d
#	rm -rf $@.d
	mv $@.tmp $@
var/void-rootfs-latest-x86_64.tar.xz :
	mkdir -p $(@D)
	wget -cO $@.tmp https://alpha.de.repo.voidlinux.org/live/current/void-x86_64-ROOTFS-20210930.tar.xz
	mv $@.tmp $@
var/void-live-latest-x86_64.iso :
	mkdir -p $(@D)
	wget -cO $@.tmp https://alpha.de.repo.voidlinux.org/live/current/void-live-x86_64-20210930.iso
	mv $@.tmp $@
var/id_ed25519 :
	mkdir -p $(@D)
	ssh-keygen -t ed25519 -C SGE -f $@ -N ""
var/daiker :
	mkdir -p $(@D)
	wget -cO $@.tmp https://raw.githubusercontent.com/daimh/daiker/master/daiker
	chmod +x $@.tmp
	mv $@.tmp $@

clean :
	chmod -R u+w var/voidlinux-sge* var/id_ed25519* var/daiker || :
	rm -rf var/voidlinux-sge* var/id_ed25519* var/daiker || :
