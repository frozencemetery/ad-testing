ENV["VAGRANT_DEFAULT_PROVIDER"] = "libvirt"

Vagrant.configure("2") do |config|
    config.vm.define("adserver") do |adserver|
        adserver.vm.box = "peru/windows-server-2019-standard-x64-eval"
        adserver.vm.box_version = "20210401.01" # TODO fix this
        adserver.vm.provider("libvirt") do |libvirt|
            libvirt.memory = 2048
        end
        adserver.vm.provision :ansible do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "adserver.yaml"
        end
    end
end
