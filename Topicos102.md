# Tópico 102: Instalação do Linux e administração de Pacotes

### Onde ficam salvas as informações a respeito das partições?
- As informações a respeito delas são armazenadas em uma tabela de partições
-  Essa tabela inclui informações sobre o primeiro e o último setores da partição e seu tipo e detalhes


### É correto afirmar que no sistema de arquivos descreve como as informações estarão armazenadas? Como é a estrutura?
- Sim.
- Disco >> Partição >> Sistema de arquivos


### Qual o principal objetivo do LVM?
- O LVM é útil em situações em que é necessário adicionar mais espaço a uma partição sem precisar migrar os dados para um dispositivo maior


## Pontos de montagem

### O que é um ponto de montagem?
- É quando eu vinculo um sistema de arquivos a um ponto especídico da árvore de diretórios do sistema.

### O que aconeceria se tentássemos montar um sistema de arquivos em um ponto de montagem inexistente?
- Não seria possível montar um sistema de arquivos em um ponto de montagem inexistente.

### O que aconteceria se tentássemos montar um sistema de arquivos em um ponto de montagem cujo o diretório já possui arquivos?
- Estes arquivos ficariam indisponíveis até você montar o sistema de arquivos.

### Onde são montados de forma automática os dispositivos externos conectados à maquina? e onde devo montar manual?
- Antigamente todos eram montados em **/mnt** (CD/DVD, Disquete...), mas atualmente o ponto de montagem padrão para qualquer mídia é o **/media** (como discos externos, drives flash USB, leitores de cartão de memória, discos óticos etc.) conectada ao sistema. 
- Caso a montagem seja manual, recomenda-se montar no /mnt


### Nos padrões novos de montagem, onde seriam montados, por exemplo, um drive flash USB com o nome FlashDrive conectado pelo usuário john?
- Em /media/jhon/FlashDrive, mas a maneira que isso é feito varia de acordo com o ambiente desktop.

## Mantendo as coisas separadas

### Quais os principais motivos para manter alguns os diretórios em partições separadas?
- /boot separado: garante que o sistema ainda vai inicializar em caso de flha no sistema de arquivos raiz
- /home ou /var separados: Mais fácil de reinstalar o sistema, visto que os dados dos usuários ou servidor web(por exemplo) estarão em outra partição.
- Para questões de desempenho, pode querer manter uma partição "/" em um SSD e um "/home" ou "/var" em um disco mais lento.

### O que há na partição /boot?
- Contém arquivos usados pelo Gerenciador de Inicialização (não são bootloaders (SysV, Upstart Sysd,)) 


### Quais são os principais gerenciadores de inicialização (bootloader)?
- No linux, costuma ser o GRUB2, nos mais antigos, o GRUB Legacy
- Gerenciadores de inicialização são diferentes de sistema de inicialização (init sistem: SysVinit, Upstart, Systemd)

### Uma partição para o /boot é necessária? Mas seria recomendado instalar separado?
- Tecnicamente não é necessária, pois na maioria dos casos o GRUB pode montar o /boot na partição raiz (/boot).
- Pode ser interessante, em caso de falha no sistema de arquivos raiz, ou se usar um método não suportado de criptografia ou compactação


## A partição do sistema EFI (ESP)

### O que você sabe sobre a partição do sistema EFI (Partição ESP)? Qual o sistem de arquivos utilizado? é obrigatório ser no início?
- A partição do sistema EFI (ESP) é utilizada por máquinas baseadas na UEFI.
- Armazena gerenciadores de inicialização e imagens do kernel para SO.
- Utiliza sistema de arquivos baseado em FAT.
- Em disco particionado com tabela GUID, ela possui o identificador global único  C12A7328-F81F-11D2-BA4B-00A0C93EC93B, se for formatado no esquema de particionamento do MBR, o Id seria 0xEF.
- Em máquinas windows a costuma sera primeira partição do disco, embora isso não seja obrigatório

### Após a instalação do sistema operacional, onde os arquivos da partição ESP são criados ou preenchidos em um sistema Linux?
- /boot/efi

## /home

### O que se sabe sobre a partição /home?
- Principal local de criação de usuários padrões
- O diretório do usuário root não é esse, é o /root.
- Alguns serviços do sistema por der usuários associados com diretórios pessoais em outros lugares.

## /var

### O que se tem no diretório /var?
Contém "dados variáveis", arquivos e diretórios que o sitema deve poder escrever durante a opraçao: Ex:
- /var/log
- /var/temp
- /var/cache
- /var/www/html (padrão apache web server) e 
- /var/lib/mysql (padrão banco de dados)

### Qual seria uma boa razão para colocar o /var em uma partição separada?
- Estabilidade, um processo defeituoso poderia encher os logs /var/log.
- Se /var estiver em /, poderia disparar um kernel panic e corromper o sistema de arquivos, o que seria muito ruim de resolver.
- Em um sistema crítico é interessante colocar o /var em um disco diferente adicionando uma camada extra de proteção.

## SWAP

### O que sabe sobre a partição swap ou partição de troca? Qual o utilitário utilizado?
- É usada para passar as páginas da memória RAM para o disco confome o necessário.
- Pode ser usado tanto partição SWAP, quanto ARQUIVOS SWAPs (pode ser 1 ou vários)
- No caso de partição, é de um tipo específico, configurada com o utilitário "mkswap"
- No caso de arquivos, serve para aumentar rapidamento o espaço  útil quando necessário.

### Qual a "Regra da memória SWAP"?
- Controversa. A regra antiga nem sempre se aplica (R. Antiga: 2x a quantidade de RAM)
- A quantidade, depende da carga de trabaho, O ideal, é que se o serviço for crítico, verificar na documentação do serviço.
- Mas A Red Hat tem um modelo:
(anexei ao anki)

## LVM

### O que seria o LVM? e qual sua estrutura base?
É uma forma de virtualizar o armazenamento, mais flexível que o particionamento tradicional.

- LV = Logical Volumes: Similar às partições, mas com flexibilidade.
    - LE: Logical Extensions
- VG = Volumes Groups: Vistos como um único dispositivo lógico.    
- PV = Phisical Volumes: Dispositivo de bloco.
    - PE = Phisical Extensions

Cada extensão lógica (LE) de forma geral é mapeada para uma extensão física (PE), mas pode ser alterado quando usado espelhamento de disco.


### Como podemos aumentar o LV (volume lógico) em uma estrutura de LVM?
- Basta adicionar mais extensões do pool disponível no Grupo de Volumes. Da mesmaforma, as extensões podem ser removidas para reduzir o LV.
- Volume lógico = PE (4MB por padrão) * LE

### Após a criação do LV (volume lógico), é correto afirmar que será visto pelo sistema operacional como um dispositivo de bloco normal. Um dispositivo será criado em /dev, com o nome /dev/VGNAME/LVNAME, em que VGNAME é o nome do Grupo de Volumes e LVNAME o nome do Volume Lógico?
- Verdadeiro

### Como os discos LV podem ser montados? (Resposta teórica)
- Podem ser formatados com o sistema de arquivos desejado usando utilitários padrão (como o mkfs.ext4, por exemplo) e montados com os métodos usuais, seja manualmente, com o comando mount, ou automaticamente, adicionando-os ao arquivo /etc/fstab.

### Onde deve estar a partição de inicialização para garantir que um PC seja capaz de carregar o kernel?
- Antes do cilindro 1024.

### Onde a partição EFI costuma ser montada?
- Em /boot/efi

### Qual a menor unidade dentro de um Grupo de Volumes?
- Os Grupos de Volumes são subdivididos em extensões

### Como se define o tamanho de um Volume Lógico?
- Pelo tamanho das extensões físicas multiplicado pelo número de extensões no volume.


### Em um disco formatado com o esquema de particionamento MBR, qual a ID da Partição do Sistema EFI?
- A ID é 0xEF

# 102.2 Instalar o gerenciador de inicialização

## Qual o gerenciador de inicialização na maioria das distribuições linux?
- GRUB, que carrega um kernel de um sistema operacional.

## O é o grub 2? e quais as principais vantagens relacionadas ao seu antecessor?
- Uma remodelagem completa do grub legacy (antigo), se tornou mais robusto e poderoso. Parece uma linguagem Script;
- Pode iniciar imagens ISOs LiveCDs direto do disco rígido;
- Suporte à temas;
- Melhor suoporte à arquitetura não-x86;
- suporte universal para UUIDs (ajuda a identificar discos e partições);

## Quando o GRUB (legacy) parou o seu desenvolvimento?
- Em 2005, desde então a maioria das principais distros linux vem com o GRUB 2. Mas ainda podemos encontrar sistemas usando o GRUB legacy

## Onde fica o gerenciador de inicialização?
Em um disco particionado em MBR o grub fica na partição MBR.

# Quais as limitações do esquema de particionamento MBR?
- Até 2TB
- Originalmente máximo de 4 partições primárias, mais tarde, 3 partições primárias e 1 estendida subdividida em várias partições lógicas.


## Os discos particionados em GPT podem ser usados em computadores com BIOS tradicional ou também com UEFI? [V/F]
- Verdadeiro. Em máquinas com BIOS, a segunda parte do GRUB é armazenada em uma partição especial de inicialização da BIOS.


## Em sistemas com firmware UEFI, o grub é carregado pelo firmware a partir dos arquivos grubia32.efi (para sistemas de 32 bits) ou grubx64.efi (para sistemas de 64 bits) em qual partiçã?
- ESP (EFI System Partition)


# A Partição /boot
### Por que, apesar de não ser imprescindível, é aconselhável utilizar uma partição /boot separada em sistemas Linux modernos?
- Porque a partição /boot separa os arquivos necessários para o processo de inicialização do restante do sistema de arquivos, o que pode aumentar a confiabilidade e facilitar o gerenciamento do sistema, além de garantir acesso mais fácil ao kernel e ao carregador de boot.


### Quais são as situações específicas em que uma partição /boot separada se torna necessária para garantir o funcionamento do GRUB 2?
- Quando a partição raiz do sistema está criptografada, compactada, ou utiliza um sistema de arquivos que o GRUB 2 não suporta, é necessário ter uma partição /boot separada, pois o GRUB precisa acessar diretamente os arquivos de inicialização.

### O que sabe sobre o conteúdo da partição /boot?

-  Arquivo de configuração (config-VERSION): Arquivo de configuração do kernel, este arquivo é gerado automaticamente quando um novo kernel é compilado, não deve ser editado manualmente. Ex: config-4.15.0-65-generic

- Mapa do sistema (System.map-VERSION): Tabela de consulta de nomes e símbolos com sua posição correspondente na memória. Pode ser últil para resolver problemas de kernel panic. EX: System.map-4.15.0-65-generic

- Kernel do Linux (vmlinux-VERSION): É o kernel propriamente dito. EX: vmlinux-4.15.0-65-generic. Também pode vir com um "z" no final (vmlinuz), que indica que o arquivo foi compactado.

- Arquivos relacionados ao Gerenciador de inicialização ficam em /boot/grub/:
 - /boot/grub/grub.cfg para o GRUB2  **ou** /boot/grub/menu.lst para o GRUB Legacy
 - /boot/grub/i386-pc: Módulos
 - /boot/grub/locale: arquivos de tradução
 - /boot/grub/fonts: fontes 

 # GRUB 2
 ### Qual a diferença entre "sudo su" e "sudo su -"?

 ### O que fazer caso seu sistema se recuse a iniciar? (checar melhor)
 - Será necessário iniciar a partir de um Live-CD ou um disco de recuperação, descobrir qual a partição de inicialização, monta-la, executar o utilitário grub-install para reinstalar o grub2.

 ```
fdisk -l /dev/sda     # Liste os diretórios
mkdir /mnt/tmp     # Crie o diretório temporário
mount /dev/sda1 /mnt/tmp    # Monte o diretório temporário
grub-install --boot-directory=/mnt/tmp /dev/sda    # Aponte o utilitário para o dispositivo de inicialização. 
```
Caso não tenha uma partição de inicialização (Coluna boot com asterisco) mas possua o diretório /boot, substitua o último comando por:
```
grub-install --boot-directory=/boot /dev/sda
```

### Como podemos checar se o disco possui uma partição ou um diretório /boot?
Execute o comando: "fdisk -l /dev/sda". 
A partição de inicialização é identificada com o * na coluna boot:
```
Device    Boot   Start    End      Sectors    Size    Id   Type
/dev/sda1  *     2048   2000895    1998848    976M    83   Linux
/dev/sda2       2002942 234440703 232437762   110,9G   5   Extended
/dev/sda5       2002944 18008063   16005120   7,6G     82  Linux swap / Solaris
/dev/sda6       18010112 234440703 216430592 103,2G 83 Linux
```
Caso esteja instalando em um sistema que não possui uma partição de inicialização, possivelmente não terá um "*", mas sim uma partição chamada "BIOS Boot":
```
Device       Start       End   Sectors Size Type
/dev/sda1     2048      4095      2048   1M BIOS boot
/dev/sda2     4096   2101247   2097152   1G Linux filesystem
/dev/sda3  2101248 157284351 155183104  74G Linux filesystem
```
