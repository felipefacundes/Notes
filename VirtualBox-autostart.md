## Configurando VirtualBox para auto iniciar no Linux
#### Configuring VirtualBox autostart on Linux

Muitas vezes é útil configurar as máquinas virtuais do VirtualBox para iniciar e parar automaticamente na inicialização e desligamento. A documentação oficial não estava clara, então fiz um pequeno tutorial sobre ela.

1. Configurando arquivos
`/etc/default/virtualbox`

    `sudo nano /etc/default/virtualbox` então adicione:

```
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/autostart.cfg
```

`/etc/vbox/autostart.cfg`

`sudo nano /etc/vbox/autostart.cfg` adicione:

```bash
default_policy = deny
# Crie uma entrada para cada usuário com permissão para usar a inicialização automática
UserName = {
allow = true
}
```
```
sudo chgrp vboxusers /etc/vbox
sudo chmod 1775 /etc/vbox
```

Para cada nome de usuário permitido: `sudo usermod -aG vboxusers USERNAME`, em seguida será aplicado o registro de entrada e saída.

2. Escolha VMs para iniciar e parar automaticamente

- Na primeira vez que um usuário configura a inicialização automática, o comando: `VBoxManage setproperty autostartdbpath /etc/vbox` precisa ser executado.
- Observação: as opções de inicialização automática são armazenadas no arquivo /etc/vbox e na própria VM. Se mover o vm, as opções podem precisar ser definidas novamente.
- `VBoxManage modifyvm <uuid|vmname> --autostart-enabled <on|off>`
- Você também pode: `VBoxManage modifyvm <uuid|vmname> --autostop-type <disabled|savestate|poweroff|acpishutdown>`

3. Se preferir fazer todo processo acima de forma automática:

Crie um script vboxautostart com o seguinte conteúdo:

```bash
#!/bin/bash
sudo mkdir /etc/vobx
# set environment variables
sudo tee -a /etc/default/virtualbox <<'EOF'
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/vboxauto.conf
EOF

# set vboxautostart policy
sudo tee -a /etc/vbox/vboxauto.conf << "EOF"
default_policy = deny
$USER = {
    allow = true
}
EOF

sudo chgrp vboxusers /etc/vbox
sudo chmod 1775 /etc/vbox
sudo usermod -aG vboxusers $USER
VBoxManage setproperty autostartdbpath /etc/vbox

sudo service vboxautostart-service restart
sudo reboot
```

4. Habilite autostart para sua VM

```
VBoxManage -nologo list vms
VM="Name_Of_Your_VM"
VBoxManage modifyvm "${VM}" --autostart-enabled on --autostop-type acpishutdown
sudo systemctl stop vboxautostart-service
sudo systemctl start vboxautostart-service
```

