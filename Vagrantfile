# -*- mode: ruby -*-
# vi: set ft=ruby :

IP_NW = "192.168.33."

K8S_IP = 30
MINIO_IP = 20

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_check_update = false

  # microk8s
  config.vm.define "microk8s" do |node|
    node.vm.provider "virtualbox" do |vb|
      vb.name = "microk8s"
      vb.memory = 2048
      vb.cpus = 1
    end
    node.vm.hostname = "microk8s"
    node.vm.network "private_network", ip: IP_NW + "#{K8S_IP}"
    node.vm.network 'forwarded_port',  guest: 22,    host: 2201,  id: 'ssh',       host_ip: '127.0.0.1', auto_correct: true
    node.vm.network 'forwarded_port',  guest: 80,    host: 8000,  id: 'ingress',   host_ip: '127.0.0.1', auto_correct: true
    node.vm.network 'forwarded_port',  guest: 8080,  host: 8080,  id: 'apiserver', host_ip: '127.0.0.1', auto_correct: true
    node.vm.network 'forwarded_port',  guest: 32000, host: 32000, id: 'registry',  host_ip: '127.0.0.1', auto_correct: true

    node.vm.provision "update-dns", type: "shell", path: "./update-dns.sh"
    node.vm.provision "install-microk8s", type: "shell", path: "./install-microk8s.sh"
  end

  # minio
  config.vm.define "minio" do |node|
    node.vm.provider "virtualbox" do |vb|
      vb.name = "minio"
      vb.memory = 512
      vb.cpus = 1
    end
    node.vm.hostname = "minio"
    node.vm.network "private_network", ip: IP_NW + "#{MINIO_IP}"
    node.vm.network 'forwarded_port',  guest: 22,   host: 2202, id: 'ssh',   host_ip: '127.0.0.1', auto_correct: true
    node.vm.network 'forwarded_port',  guest: 9000, host: 9000, id: 'minio', host_ip: '127.0.0.1', auto_correct: true

    node.vm.provision "update-dns", type: "shell", path: "./update-dns.sh"
    node.vm.provision "install-docker", type: "shell", path: "./install-docker.sh"
  end

end
