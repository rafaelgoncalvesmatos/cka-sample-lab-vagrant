# Vagrant + Virtualbox

````
configuracoes = {
  "control1" => {
    "images"   => "ubuntu/jammy64",
    "memory"  => "4048",
    "cpu"     => "4",
    "ip"      => "101"
  },
  "control2" => {
    "images"   => "ubuntu/jammy64",
    "memory"  => "4048",
    "cpu"     => "4",
    "ip"      => "101"
  },
  "control3" => {
    "images"   => "ubuntu/jammy64",
    "memory"  => "4048",
    "cpu"     => "4",
    "ip"      => "101"
  },
  "worker1" => {
    "images"   => "ubuntu/jammy64",
    "memory"  => "6048",
    "cpu"     => "4",
    "ip"      => "102"
  },
  "worker2" => {
    "images"   => "ubuntu/jammy64",
    "memory"  => "6048",
    "cpu"     => "4",
    "ip"      => "103"
  },
}

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false

  configuracoes.each do |nome, capacity|
    config.vm.define "#{nome}" do |maquina|
      maquina.vm.box = "#{capacity["images"]}"
      maquina.vm.network "public_network", bridge: "Killer(R) Wi-fi 6 AX 1650i 1600MHz Wireless Network Adapter (20 1NGW)"
      maquina.vm.hostname = "#{nome}.myckacertificate.com"

      maquina.vm.provider "virtualbox" do |customizando|
        customizando.cpus = capacity["cpu"]
        customizando.memory = capacity["memory"]
        customizando.name = "#{nome}.myckacertificate.com"
        # customizando.customize [ "modifyvm", :id, "--groups", "/CKACertificate" ]
      end

      # Provisionamento (script inline)
      maquina.vm.provision "shell", run: "once", inline: <<-SHELL
        echo "===> Clone do repositorio"
        git clone https://github.com/sandervanvugt/cka
        echo "===> Instalacao do setup-container"
        bash ~/cka/setup-container.sh
        echo "===> Instalacao dos pacotes Baseline"
        sudo apt install net-tools
        echo "===> Desativando checagem da partição raiz no /etc/fstab"
        sudo sed -i 's/errors=remount-ro/defaults,noatime/' /etc/fstab
        
        echo "===> Configurando /etc/netplan/50-cloud-init.yaml"
        sudo tee /etc/netplan/50-cloud-init.yaml > /dev/null <<EOF
network:
  ethernets:
    enp0s3:
      dhcp4: true
      dhcp4-overrides:
        use-routes: false
EOF

        echo "===> Criando /etc/netplan/50-vagrant.yaml"
        sudo tee /etc/netplan/50-vagrant.yaml > /dev/null <<EOF
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
        - 192.168.15.#{capacity["ip"]}/24
      routes:
        - to: default
          via: 192.168.15.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
EOF

        echo "===> Aplicando configurações do Netplan"
        sudo netplan apply
      SHELL
    end
  end
end

````
