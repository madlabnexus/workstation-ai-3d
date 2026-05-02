# Cheatsheet — Dicionário de Comandos Úteis

> Aqui ficam os comandos reais que rodamos durante o projeto, organizados por tema. **Cada comando que executarmos juntos vira uma entrada aqui** — assim, quando você precisar de novo no futuro, encontra rápido.

**Como usar:** Ctrl+F com o que você quer fazer. Ex: "snapshot", "atualizar", "wifi".

---

## Status atual

Este cheatsheet vai sendo construído **conforme avançamos nos milestones**. Inicialmente está vazio. Conforme rodamos comandos reais e validamos que funcionam no nosso hardware específico, eles vêm pra cá com explicação.

---

## Índice planejado (vai ser preenchido)

### Linux geral
- [ ] Listar discos e partições (`lsblk`, `fdisk -l`, `parted`)
- [ ] Identificar UUID de partições (`blkid`)
- [ ] Ver entradas de boot UEFI (`efibootmgr -v`)
- [ ] Atualizar sistema (`apt update`, `apt full-upgrade`)
- [ ] Instalar/remover pacotes (`apt install`, `apt remove`, `apt autoremove`)
- [ ] Gerenciar serviços (`systemctl status/start/stop/enable/disable`)
- [ ] Ver logs (`journalctl -xe`, `journalctl -u serviço`)
- [ ] Ver uso de disco (`df -h`, `du -sh`)
- [ ] Ver uso de memória (`free -h`, `htop`)

### BTRFS e Snapper
- [ ] Listar subvolumes (`btrfs subvolume list /`)
- [ ] Criar snapshot manual (`snapper create --description "..."`)
- [ ] Listar snapshots (`snapper list`)
- [ ] Reverter para snapshot (`snapper rollback`)
- [ ] Ver espaço usado (`btrfs filesystem usage /`)
- [ ] Compactar arquivos existentes (`btrfs filesystem defragment`)

### NVIDIA
- [ ] Verificar driver instalado (`nvidia-smi`)
- [ ] Listar drivers disponíveis (`ubuntu-drivers list`)
- [ ] Instalar driver recomendado (`ubuntu-drivers autoinstall`)
- [ ] Configurar GPU primária (`prime-select`)

### Bootloader / dual boot
- [ ] Atualizar GRUB (`update-grub`)
- [ ] Reinstalar GRUB (`grub-install`)
- [ ] Listar entradas EFI (`efibootmgr`)
- [ ] Mudar ordem de boot (`efibootmgr -o`)

### Pacotes alternativos
- [ ] Listar Flatpaks instalados (`flatpak list`)
- [ ] Instalar Flatpak (`flatpak install flathub <app>`)
- [ ] Remover Snap (`snap remove <pacote>`)

### Wine / Gaming
- [ ] Ver versões de Proton (Steam → Settings)
- [ ] Listar jogos (`steam` ou Lutris/Heroic)
- [ ] Instalar Proton-GE via ProtonUp-Qt

### Diagnóstico
- [ ] Ver hardware completo (`inxi -Fxxxz`, `lshw`)
- [ ] Ver dispositivos PCI (`lspci -v`)
- [ ] Ver dispositivos USB (`lsusb`)
- [ ] Verificar IOMMU groups (`for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%%/*}; printf 'IOMMU Group %s ' "$n"; lspci -nns "${d##*/}"; done`)
- [ ] Temperatura CPU/GPU (`sensors`, `nvidia-smi`)

### Virtualização
- [ ] Listar VMs (`virsh list --all`)
- [ ] Iniciar/parar VM (`virsh start <nome>`, `virsh shutdown <nome>`)
- [ ] Editar config VM (`virsh edit <nome>`)

### Windows (PowerShell)
- [ ] Desabilitar Fast Startup (`powercfg /h off` + GUI)
- [ ] Status BitLocker (`manage-bde -status`)
- [ ] Versão Windows (`winver`)
- [ ] Atualizar BIOS via Lenovo Vantage

---

> Conforme cada item vira realidade neste projeto, ele vai do "índice planejado" para uma seção dedicada com o **comando completo**, **explicação linha por linha**, **saída esperada** e **gotchas conhecidos**.
