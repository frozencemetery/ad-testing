ENV["VAGRANT_DEFAULT_PROVIDER"] = "libvirt"

$ipa_setup = <<SCRIPT
echo Provisioning ipaserver...

PASS=secretes
HOSTNAME=ipaserver.ipa.test
REALM=$(echo $HOSTNAME | awk -F. -vOFS=. '{print toupper($2), toupper($3)}')
NETBIOS=$(echo $REALM | awk -F. '{print $1}')

if [ -f /etc/vagrant_provisioned_at ]; then
    echo Provisioning already done!
    exit 0
fi

printf "${PASS}\n${PASS}\n" | passwd root
printf "${PASS}\n${PASS}\n" | passwd vagrant
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' \
    /etc/ssh/sshd_config
systemctl reload-or-restart sshd

setenforce 0
sed -i 's/SELINUX=\(enforcing\|permissive\)/SELINUX=disabled/g' \
    /etc/selinux/config

dnf config-manager --set-disabled \*
dnf config-manager --set-enabled {fedora,updates{,-testing}}{,-debuginfo}

# mirror failures again...
dnf -y distro-sync --downloadonly
dnf -yC distro-sync || true

dnf -y install psmisc gdb valgrind git emacs-nox tmux gdb strace nmap \
    ltrace lsof ipa-server{,-dns,-trust-ad} krb5-{libs,workstation,server} \
    sssd-winbind-idmap

dnf -y debuginfo-install krb5-{libs,workstation,server} ipa-server

hostnamectl set-hostname $HOSTNAME

cat > /etc/hosts <<EOF
127.0.0.1 localhost
192.168.3.3 $HOSTNAME
192.168.3.2 adserver adserver.ad.test
EOF

ipa-server-install -NU -r $REALM -p $PASS -a $PASS \
    --setup-dns --auto-forwarders --auto-reverse

echo "${PASS}" | kinit admin
ipa-adtrust-install -U --netbios-name=$NETBIOS -a $PASS

cat > /etc/krb5.conf.d/ad_test <<EOF
[realms]
AD.TEST = {
    kdc = adserver.ad.test
}
EOF

ipa dnsforwardzone-add ad.test --forwarder=192.168.3.2 --forward-policy=only

cat >> install_trust.sh <<EOF
#!/bin/bash
echo "${PASS}" | kinit admin
echo "vagrant" | ipa trust-add --type=ad ad.test --admin vagrant --password
EOF
chmod +x install_trust.sh

date > /etc/vagrant_provisioned_at
SCRIPT

Vagrant.configure("2") do |config|
    config.vm.define("adserver") do |adserver|
        adserver.vm.network :private_network, ip: "192.168.3.2"
        adserver.vm.box = "peru/windows-server-2019-standard-x64-eval"
        adserver.vm.box_version = "20210401.01" # TODO fix this
        adserver.vm.provider("libvirt") do |libvirt|
            libvirt.memory = 4096
        end
        adserver.vm.provision :ansible do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "adserver.yaml"
        end
    end

    config.vm.define("ipaserver") do |ipaserver|
        ipaserver.vm.network :private_network, ip: "192.168.3.3"
        ipaserver.vm.box = "fedora/34-cloud-base"
        ipaserver.vm.provider("libvirt") do |libvirt|
            libvirt.random :model => "random"
            libvirt.memory = 4096
        end
        ipaserver.vm.provision "ipa", type: "shell", inline: $ipa_setup
    end
end
