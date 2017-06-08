# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.box = "centos/atomic-host"

    config.vm.provider "virtualbox" do |vb|
        vb.gui = true
        vb.cpus = 2
        vb.memory = "2048"
        vb.name = "node01"
    end

    config.vm.define "node01" do |c|
        c.vm.provision "shell", inline: <<-SHELL
            # Upgrade the Atomic Host
            sudo atomic host upgrade --reboot
        SHELL

        c.vm.provision "shell", inline: <<-SHELL
            # Local Docker Registry
            sudo docker create -p 5000:5000 \
                -v /var/lib/local-registry:/var/lib/registry \
                -e REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/var/lib/registry \
                -e REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io \
                --name=local-registry registry:2
            sudo mkdir -p /var/lib/local-registry
            sudo chcon -Rvt svirt_sandbox_file_t /var/lib/local-registry
            sudo sh -c 'echo "\
[Unit]
Description=Local Docker Mirror registry cache
Requires=docker.service
After=docker.service

[Service]
Restart=on-failure
RestartSec=10
ExecStart=/usr/bin/docker start -a %p
ExecStop=-/usr/bin/docker stop -t 2 %p

[Install]
WantedBy=multi-user.target
            " > /etc/systemd/system/local-registry.service'

            sudo systemctl daemon-reload
            sudo systemctl enable local-registry
            sudo systemctl start local-registry
        SHELL

        # Configuring Kubernetes Master
        c.vm.provision "shell", inline: <<-SHELL
            # Configure etcd
            sudo sed -i 's ETCD_LISTEN_CLIENT_URLS="http://localhost:2379" ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001" ' /etc/etcd/etcd.conf
            sudo sed -i 's ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379" ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001" ' /etc/etcd/etcd.conf
            
            # Generating certificates
            curl -L -O https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz
            tar xzf easy-rsa.tar.gz
            cd easy-rsa-master/easyrsa3
            ./easyrsa init-pki


        SHELL
    end
end
