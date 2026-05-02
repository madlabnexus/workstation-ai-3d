# Glossário de Termos Técnicos

> Dicionário rápido de todos os termos que aparecem neste projeto. Cada um tem uma definição **simples** primeiro, e depois detalhe técnico se relevante. Em ordem alfabética dentro de cada categoria.

**Convenção:** termos em código estão `assim` (são comandos ou nomes literais).

---

## 1. Conceitos gerais de Linux

### Bootloader
Programa que roda **antes** do sistema operacional carregar e decide o que vai bootar. No nosso caso, o GRUB vai mostrar um menu com "Windows" e "Kubuntu" para você escolher.

### Distribuição (distro)
Uma "versão" do Linux empacotada por uma comunidade ou empresa específica. Exemplos: Ubuntu, Fedora, Arch. Todas usam o mesmo kernel Linux, mas com diferentes seleções de programas, gerenciadores de pacotes, e filosofias.

### EFI / UEFI
**UEFI** (Unified Extensible Firmware Interface) é o sucessor da antiga BIOS — é o software que roda no chip da placa-mãe quando você liga o computador, antes de qualquer sistema operacional. **EFI** é o nome do padrão, **UEFI** é o nome moderno do mesmo padrão.

### EFI System Partition (ESP)
Uma partição especial pequena (~100-500 MB) formatada como **FAT32**, onde ficam os arquivos de boot dos sistemas UEFI. Cada SO instalado coloca seus arquivos numa subpasta:
- `/EFI/Microsoft/` → Windows
- `/EFI/ubuntu/` → Ubuntu/Kubuntu
- `/EFI/systemd/` → systemd-boot

Múltiplos SOs **podem** compartilhar a mesma ESP, e isso é o normal e seguro.

### Kernel
O coração do Linux. É o programa que gerencia hardware (CPU, RAM, disco, dispositivos USB) e expõe interfaces para os programas usarem. Quando alguém fala "kernel 6.17", está falando da versão dessa peça central.

### LTS (Long Term Support)
Versão "estável de longa duração". No Ubuntu, sai uma LTS a cada 2 anos (em abril) e tem 5 anos de suporte de segurança. **Ubuntu 24.04 LTS** = lançada em abril/2024, suportada até abril/2029.

### HWE (Hardware Enablement)
Pacote oficial Ubuntu que **traz kernel mais novo** para versões LTS antigas, sem perder o suporte LTS. Exemplo: Ubuntu 24.04 LTS saiu com kernel 6.8, e via HWE você instala kernel 6.17 mantendo o suporte de 5 anos.

### Kubuntu vs Ubuntu vs Ubuntu Studio
**Flavors** (sabores) do Ubuntu — todos usam a mesma base Ubuntu, diferem no ambiente desktop e nos pacotes pré-instalados.
- **Ubuntu** = GNOME
- **Kubuntu** = KDE Plasma (o nosso)
- **Ubuntu Studio** = KDE Plasma + suite multimedia + low-latency kernel

### Wayland / X11 / XWayland
**Display servers** — o software que desenha as janelas na tela.
- **X11** = legado, 30+ anos, sendo aposentado
- **Wayland** = moderno, mais seguro, melhor para múltiplos monitores
- **XWayland** = ponte que permite apps antigos X11 rodarem em sessão Wayland

---

## 2. Pacotes e gerenciamento de software

### apt
Gerenciador de pacotes do Ubuntu/Debian. Você usa `apt` no terminal para instalar, remover ou atualizar programas. Os arquivos têm extensão `.deb`.

### .deb
Formato de pacote do Debian/Ubuntu. Análogo ao `.exe` ou `.msi` do Windows.

### PPA (Personal Package Archive)
**Repositório de software adicional** mantido por terceiros (não a Canonical). Quando você precisa de um software mais novo do que o disponível no Ubuntu oficial (ex: drivers NVIDIA, Wine), adiciona uma PPA. Exemplo:
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt install nvidia-driver-570
```

### Snap
Formato de pacote da Canonical (empresa do Ubuntu), **sandboxed** (isolado). Polêmico: alguns usuários não gostam (lentidão de cold start, telemetria, mount loops). Exemplo: Firefox no Ubuntu vem como Snap por padrão.

### Flatpak
Formato de pacote **sandboxed** distribuído via [Flathub](https://flathub.org). Mais leve que Snap, padrão da indústria, comunidade grande. Vamos preferir Flatpak no projeto.

### AppImage
Pacote único e portável. **Não instala** — você baixa um arquivo `.AppImage`, dá permissão de execução, e roda. Equivalente a um `.exe` portátil.

### ubuntu-restricted-extras
Meta-pacote (um pacote que instala outros) com **codecs proprietários** (MP3, MP4, H.264) e fontes Microsoft (Arial, Times New Roman). Necessário para reproduzir mídia comum.

---

## 3. Sistemas de arquivos e armazenamento

### BTRFS
Sistema de arquivos moderno do Linux com snapshots, subvolumes e compressão nativos. **É o nosso filesystem principal.** Pronuncia-se "butter-FS" ou "B-tree-FS".

### Subvolume (BTRFS)
Como uma "pasta especial" dentro do BTRFS que pode ser tratada independentemente — montada, snapshotada, ter quotas separadas. Layout típico:
- `@` → montado em `/` (sistema raiz)
- `@home` → montado em `/home` (dados do usuário)
- `@var-log` → `/var/log` (logs, fora dos snapshots para não inchar)
- `@cache` → `/var/cache` (cache, idem)

### Snapshot
Foto instantânea do estado de um subvolume BTRFS. Ocupa **zero espaço** no momento da criação (Copy-on-Write) — só cresce conforme arquivos mudam.

### Snapper
Programa que automatiza snapshots BTRFS. Configura snapshots automáticos antes/depois de `apt`, agendados (a cada hora, dia, semana), com política de retenção.

### grub-btrfs
Programa que adiciona snapshots BTRFS como entradas no menu GRUB. Resultado: se uma atualização quebrar tudo, você reinicia, escolhe um snapshot do menu, e está de volta ao estado funcional anterior.

### ext4
Sistema de arquivos tradicional do Linux. Estável, robusto, **sem snapshots nativos**. Não vamos usar como root, mas pode aparecer em partições secundárias.

### NTFS
Sistema de arquivos do Windows. Linux pode ler e escrever via driver `ntfs-3g`. Mas atenção: se o Windows usar **Fast Startup** ou estiver hibernado, escrever NTFS do Linux pode corromper dados.

### ZFS
Sistema de arquivos avançado, padrão do **Proxmox VE** no servidor. Tem snapshots, replicação, deduplicação. Não vamos usar no notebook (é overkill).

---

## 4. Bootloader e partições

### GRUB (GRand Unified Bootloader)
Bootloader mais comum no Linux. Mostra um menu permitindo escolher SO, kernel, snapshot. Tem `os-prober` que detecta Windows automaticamente.

### systemd-boot
Bootloader minimalista alternativo ao GRUB. Usa arquivos `.conf` em `/boot/efi/loader/entries/`. É o que está atualmente no notebook (configurado pelo EndeavourOS).

### efibootmgr
Comando Linux que mostra/edita as **entries de boot UEFI** armazenadas no firmware da placa-mãe. Útil para diagnosticar dual boot:
```
sudo efibootmgr -v
```

### MBR vs GPT
Esquemas de tabela de partição.
- **MBR** (Master Boot Record) = legado, limite 4 partições primárias e 2 TB
- **GPT** (GUID Partition Table) = moderno, sem limites práticos
Sistemas UEFI requerem GPT. Estaremos sempre em GPT.

### Particionamento
Ato de dividir um disco em "fatias" (partições) lógicas. Cada partição pode ter um sistema de arquivos diferente. Layout típico nosso vai ser:
```
NVMe2 (4 TB):
├── (não tocar) EFI System Partition do Windows compartilhada
├── /boot (~2 GB, ext4) [opcional, BTRFS pode bootar direto]
└── BTRFS pool (~3.5 TB)
    ├── @ → /
    ├── @home → /home
    ├── @var-log → /var/log
    └── @snapshots → /.snapshots
```

---

## 5. Drivers e gráficos

### NVIDIA driver proprietário
Driver oficial da NVIDIA, **não é open source**. Necessário para CUDA e gaming. No Ubuntu, vem da PPA `graphics-drivers/ppa` ou do repositório padrão.

### Nouveau
Driver open source da NVIDIA, mantido pela comunidade. Funciona para 2D básico, mas péssima performance 3D em GPUs modernas. **Não vamos usar** — sempre o proprietário.

### NVIDIA Optimus
Tecnologia de laptops com **iGPU + dGPU**. A iGPU economiza bateria; a dGPU entra em ação para tarefas pesadas. No Linux moderno, usamos `nvidia-prime` ou `supergfxctl` para alternar.

### MUX switch / MUXed / MUXless
- **MUXed** (com MUX switch): a dGPU tem caminho próprio para uma saída de vídeo (ex: HDMI externo). Permite GPU passthrough mais facilmente.
- **MUXless**: a dGPU **não tem saída direta** — ela renderiza e copia o resultado pra iGPU mostrar. Comum em laptops consumer.
- Workstation laptops Lenovo P-series (incluindo P16v) **provavelmente são MUXed**.

### Mesa
Conjunto de drivers gráficos open source no Linux, incluindo OpenGL, Vulkan e drivers para Intel, AMD e (parcialmente) NVIDIA via Nouveau. Para nossa NVIDIA proprietária, Mesa não importa.

### Vulkan
API gráfica moderna, sucessora do OpenGL. Usada por jogos modernos, Proton, Blender, Unreal 5.

### CUDA
SDK da NVIDIA para programação de GPU (computação geral, não só gráficos). PyTorch, TensorFlow, Ollama, LLMs, Stable Diffusion — todos usam CUDA por baixo.

---

## 6. Virtualização

### KVM (Kernel-based Virtual Machine)
Tecnologia de virtualização nativa do kernel Linux. Permite rodar VMs com performance próxima do bare metal.

### QEMU
Emulador/virtualizador. Combinado com KVM, é o que efetivamente roda VMs no Linux. Usado por baixo do `virt-manager`, `libvirt`, Proxmox.

### libvirt
Biblioteca/daemon que abstrai múltiplos hipervisores (QEMU, KVM, Xen). Quando você usa `virsh` ou `virt-manager`, está falando com `libvirt`.

### virt-manager
Interface gráfica para gerenciar VMs via libvirt. É como o "gerente de VMs" do Linux desktop.

### IOMMU (Input-Output Memory Management Unit)
Recurso do CPU/chipset que permite isolar dispositivos PCI para passthrough seguro. Habilitado via BIOS (`Intel VT-d` ou `AMD-Vi`) e kernel parameter (`intel_iommu=on`).

### VFIO (Virtual Function I/O)
Framework do kernel Linux que permite passar dispositivos PCI (como GPU) diretamente pra uma VM. Driver `vfio-pci` "rouba" o hardware do host pra dar pra VM.

### GPU passthrough
Técnica de **dar uma GPU física inteira para uma VM**, com performance bare metal. Requer IOMMU + VFIO.

### Looking Glass
Programa que mostra a tela de uma VM com GPU passthrough numa janela do Linux host. Sem ele, você precisaria de um monitor físico separado pra ver a VM.

### Proxmox VE
Sistema operacional (Debian-based) especializado em virtualização. Combina KVM + LXC + ZFS + interface web. **É o que vai no servidor TD350.**

### Distrobox
Ferramenta que cria **containers** que parecem distros completas (Arch, Fedora, Ubuntu, Alpine) dentro do seu Linux. Útil para testar coisas, ter ambientes isolados sem rebootar.

### Docker / Podman
Sistemas de containers (mais leves que VMs). Usados para isolar aplicações, especialmente em stacks de IA. Podman é alternativa rootless ao Docker.

---

## 7. Inteligência Artificial / Machine Learning

### LLM (Large Language Model)
Modelo de linguagem grande, tipo ChatGPT/Claude. Quando rodamos "localmente" estamos rodando versões open weight desses modelos no nosso hardware.

### Ollama
Programa que simplifica rodar LLMs localmente. Comando `ollama run llama3.1:70b` baixa e executa um modelo.

### vLLM
Engine de inferência otimizada para servir LLMs em produção (mais rápido que Ollama em alta carga).

### llama.cpp
Implementação em C++ do Llama (e outros modelos), ultra-otimizada para CPU + GPU. Base de muitos outros projetos.

### ComfyUI
Interface node-based para Stable Diffusion (geração de imagens com IA). Roda localmente com a 4090.

### Quantização (Q4, Q5, Q8)
Compressão de modelos. Q4 = 4 bits por parâmetro (menor, mais rápido, qualidade ligeiramente menor). Q8 = quase qualidade original.

### Token / tokens/s
Token = pedaço de texto (palavra ou parte). Tokens/s = velocidade de geração. RTX 4090 faz ~50 tokens/s no Llama 70B Q4.

### VRAM
Memória da GPU. **Crítica** para LLMs: o modelo precisa caber na VRAM. Sua RTX 3000 Ada Laptop tem 8 GB, a 4090 tem 24 GB.

---

## 8. Gaming

### Steam
Loja/launcher de jogos da Valve. Tem versão Linux nativa.

### Proton
Camada de compatibilidade da Valve que permite rodar jogos Windows no Linux via Steam. Baseado em Wine + várias melhorias.

### Proton-GE (GloriousEggroll)
Fork comunitário do Proton com patches extras. Frequentemente faz jogos rodarem que Proton oficial não roda.

### Wine
Camada de compatibilidade que **traduz chamadas Windows** para Linux. Permite rodar `.exe` em Linux.

### Bottles
Frontend gráfico para Wine. Cria "bottles" (prefixos isolados) para cada programa.

### Lutris
Outro frontend para Wine, focado em jogos. Suporta Steam, Epic, GOG, Battle.net via Wine.

### DXVK / VKD3D
Tradutores **DirectX → Vulkan**. Razão pela qual jogos DirectX rodam tão bem em Linux hoje.

### Sunshine + Moonlight
- **Sunshine** = servidor de game streaming (instalado na máquina que tem GPU)
- **Moonlight** = cliente (instalado em outro PC, tablet, etc)
Permite jogar usando GPU de máquina remota.

---

## 9. Configuração e dotfiles

### Dotfiles
Arquivos de configuração que começam com ponto (e por isso ficam ocultos): `.bashrc`, `.zshrc`, `.config/kitty/kitty.conf`, etc. Versionar em git permite reaplicar em outra máquina.

### chezmoi
Ferramenta moderna para gerenciar dotfiles em git, com suporte a templates e dados sensíveis criptografados.

### GNU Stow
Ferramenta clássica para gerenciar dotfiles via symlinks.

### Bootstrap script
Script que pega uma máquina recém-instalada e configura **tudo automaticamente** — instala pacotes, aplica dotfiles, configura serviços. **Idempotente** = pode rodar várias vezes sem quebrar.

### Idempotência
Propriedade de uma operação que **pode ser executada múltiplas vezes** com o mesmo resultado. Importante para scripts: se o Wi-Fi caiu no meio, você roda de novo e ele continua de onde parou.

---

## 10. Backup e segurança

### Macrium Reflect
Software Windows de imagem/clone de disco. Versão Free descontinuada em jan/2024 mas ainda funcional para quem tem.

### Hiren's BootCD PE
Distribuição Windows PE (Windows portátil em pendrive) com ferramentas de manutenção e recuperação. Tradicionalmente vem com Macrium incluso.

### Ventoy
Programa que transforma um pendrive em **multiboot**. Você copia múltiplas ISOs para o pendrive e ele mostra um menu para escolher qual bootar. **É o que vamos usar.**

### Clonezilla / Rescuezilla
Alternativas open source ao Macrium. Bootam de live USB e clonam discos.

### TPM (Trusted Platform Module)
Chip dedicado a operações criptográficas. Versão 2.0 é requisito do Windows 11. Linux pode usar via `tpm2-tools` para várias coisas.

### BitLocker
Criptografia de disco do Windows. Se ativo, **suspender** antes de mexer em particionamento (senão pede chave de recuperação).

### Fast Startup (Windows)
Configuração do Windows que "hiberna" parte do sistema ao desligar para boot mais rápido. **Causa corrupção em NTFS** se o Linux escrever na partição. Sempre desabilitar em dual boot.

---

## 11. Termos específicos do hardware

### NVMe / M.2
- **M.2** = formato físico do "pente" (parecido com um chiclete)
- **NVMe** = protocolo de comunicação (muito mais rápido que SATA)
- A maioria dos SSDs M.2 modernos são NVMe.

### Thunderbolt 4 (TB4)
Padrão de conexão de alta velocidade (40 Gbps), suporta dados, vídeo (até 8K) e energia (100W). Sua dock Lenovo é TB4.

### DDR5 / DDR4 / LPDDR5
Gerações de memória RAM. Notebook tem DDR5 SODIMM, servidor tem DDR4 ECC.

### ECC (Error Correcting Code)
RAM que detecta e corrige erros automaticamente. Padrão em servidores. Seu TD350 tem.

### Optimus
Veja seção "Drivers e gráficos" acima.

---

## Termos que vão aparecer no futuro (placeholder)

À medida que avançamos, vou adicionar termos novos aqui. Lista de "ainda virá":

- ZFS, ZRAM, sanoid, syncoid (snapshots e replicação avançada)
- swtpm (TPM emulado para VM Windows)
- OVMF (firmware UEFI para VMs)
- SPICE / VNC / RDP (protocolos de tela remota)
- systemd, systemctl, journalctl (gerenciamento de serviços)
- cron, systemd timers (agendamento de tarefas)
- rsync, restic, borgbackup (backup automatizado)
- PipeWire, JACK, PulseAudio (audio stacks)
- Reaper, Ardour, Bitwig (DAWs)
- Steam Linux Runtime, Pressure Vessel (sandbox do Steam)
