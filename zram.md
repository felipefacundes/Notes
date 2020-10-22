## Como habilitar o módulo zRAM para uma troca mais rápida no Linux
### Economizando memória com o zRAM
#### How to enable the zRAM module for faster swapping on Linux

<br/>
<br/>

O zRAM (antes conhecido como compcache) é um recurso do kernel Linux para transformar uma região de memória RAM em um dispositivo de bloco com suporte à compressão de dados. Com este recurso, é possível criar uma espécie de disco virtual, onde os dados são armazenados em RAM de forma comprimida.

<br/>

Alguns casos de uso interessantes para esta funcionalidade:

- Ponto de montagem para o armazenamento de informações temporárias, em substituição ao tmpfs, porém economizando RAM, já que as informações são armazenadas de forma comprimida.
- Região de SWAP, economizando RAM devido à compressão e melhorando a performance, já que não serão necessários ciclos de I/O para fazer paginação de memória em disco.

<br/>

Este recurso ficou bastante tempo na área de staging do kernel, e foi promovido para o diretório drivers/block/zram na versão 3.14. Sua documentação está disponível no código-fonte do kernel em Documentation/blockdev/zram.txt. Para utilizá-lo, basta compilar o kernel com a opção CONFIG_ZRAM habilitada.

<br/>

O zRAM disponibiliza um ou mais arquivos de dispositivo de bloco em /dev/zramX (X = 0, 1, 2, …). A quantidade de dispositivos criada pode ser configurada através do parâmetro num_devices (por padrão, é criado apenas um dispositivo do tipo zRAM)

<br/>

##### Se você quiser usar o zram sem reiniciar o sistema, siga os seguintes comandos:

```bash
sudo modprobe zram

sudo su -c "echo 2G > /sys/block/zram0/disksize"   # Use até metade da memória RAM disponível.

sudo mkswap /dev/zram0

sudo swapon /dev/zram0 -p 32767
```

##### Para que o zram seja habilitado a cada reinicialização.

O módulo zRAM é controlado pelo systemd, portanto, não há necessidade de uma entrada fstab. E como tudo já vem instalado fora da caixa, precisamos apenas criar alguns arquivos e modificar um.

Abra uma janela de terminal e crie um novo arquivo com o comando: 

`sudo vim /etc/modules-load.d/zram.conf`

Nesse arquivo, adicione a palavra: 

`zram`

Salve e feche o arquivo. 
Em seguida, crie um segundo novo arquivo com o comando:

`sudo vim /etc/modprobe.d/zram.conf`

Nesse arquivo, cole a linha: 

`options zram num_devices=1`

Salve e feche o arquivo.
Em seguida, precisamos configurar o tamanho da partição zRAM. Crie um novo arquivo com o comando: 

`sudo vim /etc/udev/rules.d/99-zram.rules`

Nesse arquivo, cole o seguinte, modificando o atributo disksize para atender às suas necessidades, no exemplo a seguir está 2G, é aconselhável usar até a metade da memória RAM disponível, se você tem 4G de RAM, use até 2G ou menos:

`KERNEL=="zram0", ATTR{disksize}="2G",TAG+="systemd"`

Salve e feche o arquivo.

##### Como desativar a troca (swap) tradicional:

Para que o zRAM funcione como o esperado, para aumentar o desempenho do sistema, você precisará desabilitar a troca tradicional. Isso é tratado no arquivo `fstab`. Abra esse arquivo com o comando:

`sudo vim /etc/fstab`

Nesse arquivo, comente (adicione um caractere # à esquerda) a linha que começa com `/swapfile` ou `/swap` ou `/swap.img`

Caso você use o arquivo de paginação (`"swapfile"`) para hibernar o sistema desconsidere a dica acima e não a comente, no entanto, altere a prioridade do arquivo de paginação, com a opção `,pri=100` como no exemplo a seguir:

`/swap   none   swap   defaults,pri=100   0 0`

Salve e feche o arquivo.
 
##### Como criar um arquivo de unidade systemd 

Para que o zRAM seja executado, precisamos criar um arquivo de unidade systemd. Crie este arquivo com o comando:

`sudo vim /usr/lib/systemd/system/zram.service`

Nesse arquivo, cole o seguinte conteúdo:

```
[Unit]
Description=Swap with zram
After=multi-user.target

[Service]
Type=oneshot 
RemainAfterExit=true
ExecStartPre=/sbin/mkswap /dev/zram0
ExecStart=/sbin/swapon /dev/zram0 -p 32767
ExecStop=/sbin/swapoff /dev/zram0

[Install]
WantedBy=multi-user.target
```

Salve e feche o arquivo. 

Habilite a nova unidade com o comando:

`sudo systemctl enable zram`

Reinicie o sistema

##### Como descobrir se o zRAM está funcionando

Assim que o sistema for reiniciado, faça login novamente. Em uma janela de terminal, emita o comando:

`/proc/swaps`

Parabéns, o zRAM está funcionando agora. Você deve ver um aumento de desempenho quando os aplicativos e / ou serviços começarem a usar a swap no sistema.
