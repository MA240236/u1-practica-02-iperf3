# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Práctica 1.4 — Medición de Ancho de Banda con iPerf3
# Asignatura: Administración de Redes (SCA-1002)
# Grupo: SCA1002_8SB | Periodo: AGO-DIC 2026
#
# Uso:
#   vagrant up              → levantar ambas VMs
#   vagrant ssh servidor    → conectarse al servidor
#   vagrant ssh cliente     → conectarse al cliente
#   vagrant halt            → apagar las VMs
#   vagrant destroy -f      → eliminar las VMs

Vagrant.configure("2") do |config|

  # ── Box base: Ubuntu 22.04 LTS (Jammy Jellyfish) ──────────────────────────
  config.vm.box = "bento/ubuntu-22.04"

  # Deshabilitar la carpeta sincronizada por defecto
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # ── VM 1: Servidor iPerf3 ──────────────────────────────────────────────────
  config.vm.define "servidor" do |srv|
    srv.vm.hostname = "iperf-servidor"

    # Red privada con IP estatica (rango permitido por VirtualBox: 192.168.56.x)
    srv.vm.network "private_network", ip: "192.168.56.10"

    # Recursos de la VM
    srv.vm.provider "virtualbox" do |vb|
      vb.name   = "iperf-servidor"
      vb.memory = 1024
      vb.cpus   = 1
    end

    # Provisioning: instalar iPerf3 al levantar la VM
    srv.vm.provision "shell", inline: <<-SHELL
      apt-get update -q
      apt-get install -y iperf3
      echo "============================================"
      echo " iPerf3 instalado en SERVIDOR"
      echo " IP: 192.168.56.10"
      echo " Para iniciar el servidor: iperf3 -s"
      echo "============================================"
    SHELL
  end

  # ── VM 2: Cliente iPerf3 ───────────────────────────────────────────────────
  config.vm.define "cliente" do |cli|
    cli.vm.hostname = "iperf-cliente"

    # Red privada con IP estatica
    cli.vm.network "private_network", ip: "192.168.56.11"

    # Recursos de la VM
    cli.vm.provider "virtualbox" do |vb|
      vb.name   = "iperf-cliente"
      vb.memory = 1024
      vb.cpus   = 1
    end

    # Provisioning: instalar iPerf3 al levantar la VM
    cli.vm.provision "shell", inline: <<-SHELL
      apt-get update -q
      apt-get install -y iperf3
      echo "============================================"
      echo " iPerf3 instalado en CLIENTE"
      echo " IP: 192.168.56.11"
      echo " Para conectar al servidor: iperf3 -c 192.168.56.10"
      echo "============================================"
    SHELL
  end

end
