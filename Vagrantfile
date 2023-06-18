MACHINES ={
    :IR => {
        :box_name => "centos/7",
        :vm_name => "IR",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "IR"},
            {adapter: 3, auto_config: false, virtualbox__intnet: "IR"},
        ]
    },
    :CR => {
        :box_name => "centos/7",
        :vm_name => "CR",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "IR"},
            {adapter: 3, auto_config: false, virtualbox__intnet: "IR"},
            {adapter: 4, auto_config: false, virtualbox__intnet: "R1"},
            {adapter: 5, auto_config: false, virtualbox__intnet: "R2"},
        ]
    },
    :R1 => {
        :box_name => "centos/7",
        :vm_name => "R1",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "R1"},
            {adapter: 3, auto_config: false, virtualbox__intnet: "V101"},
        ]
    },
    :R2 => {
        :box_name => "centos/7",
        :vm_name => "R2",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "R2"},
            {adapter: 3, auto_config: false, virtualbox__intnet: "V102"},
        ]
    },
    :S1 => {
        :box_name => "centos/7",
        :vm_name => "S1",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "V101"},
        ]
    },
    :C1 => {
        :box_name => "centos/7",
        :vm_name => "C1",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "V101"},
        ]
    },
    :S2 => {
        :box_name => "centos/7",
        :vm_name => "S2",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "V102"},
        ]
    },
    :C2 => {
        :box_name => "centos/7",
        :vm_name => "C2",
        :net => [
            {adapter: 2, auto_config: false, virtualbox__intnet: "V102"},
        ]
    },
}

Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
        config.vm.define boxname do |box|
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxconfig[:vm_name]

            boxconfig[:net].each do |ipconf|
                box.vm.network "private_network", ipconf
            end

            box.vm.provision :ansible do |ansible|
                ansible.playbook = "provisioning/playbook-#{box.vm.hostname}.yml" # Main playbook for host
            end

        end
    end
end
        

