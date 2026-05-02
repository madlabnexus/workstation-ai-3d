# Roadmap dos Milestones

> Estratégia de progressão **incremental**, do simples ao complexo. Cada milestone entrega valor próprio — você pode parar em qualquer um e ainda ter um sistema produtivo.

---

## Visão geral

| # | Milestone | Tempo estimado | Status |
|---|---|---|---|
| **M0** | Setup base — dual boot Windows + Kubuntu funcional | 1-3 dias | 🚧 Em progresso |
| M1 | Linux fluente do dia a dia | 2-6 semanas (uso real) | ⏸️ |
| M2 | VM Windows básica no Linux (sem GPU) | 1 dia | ⏸️ |
| M3 | Boot do Windows físico em VM (P2V live) | 2-3 dias | ⏸️ |
| M4 | Single GPU passthrough (Windows VM com NVIDIA) | 1-2 semanas | ⏸️ |
| M5 | Dual GPU passthrough na dock (final game) | 2-4 semanas | ⏸️ |

**Em paralelo (não bloqueia M0-M1):**

| # | Milestone | Quando |
|---|---|---|
| MS-A | Servidor TD350: Proxmox + VM Kubuntu para IA | Após M1 estabilizado |
| MS-B | VM Windows no servidor (para Adobe/CAD remoto) | Após MS-A |

---

## M0 — Setup base do dual boot

**Objetivo:** Sair do estado atual (Windows + EndeavourOS via systemd-boot) para o estado-alvo (Windows + Kubuntu 24.04 LTS via GRUB), com BTRFS+Snapper configurado e apto para uso diário básico.

**Sub-passos:**

### M0.1 — Tweaks no Windows (sem backup)
- Desabilitar Fast Startup
- Desabilitar Hibernação
- Verificar status do BitLocker (suspender se ativo)
- Atualizar BIOS Lenovo se houver versão mais nova que 1.18
- Verificar configurações gráficas no BIOS (Hybrid Graphics vs Discrete)
- **Documento:** `m0-windows-tweaks.md`

### M0.2 — Coleta do estado atual no EndeavourOS
- Bootar EndeavourOS atual
- Coletar: `efibootmgr -v`, `lsblk -f`, `blkid`, `cat /etc/fstab`
- Identificar onde está a EFI System Partition (provavelmente NVMe1)
- Documentar layout exato do disco
- **Documento:** `m0-coleta-estado.md` (a ser criado quando chegar a hora)

### M0.3 — Clone do EndeavourOS via Macrium
- Bootar Ventoy → escolher Hiren's BootCD PE
- Abrir Macrium Reflect
- Clonar NVMe2 inteiro para destino externo (SSK 1 TB)
- Verificar integridade do clone
- **Documento:** `m0-clone-endeavouros.md` (a ser criado quando chegar a hora)

### M0.4 — Instalação do Kubuntu 24.04 LTS
- Bootar Ventoy → escolher Kubuntu 24.04 LTS
- Particionamento manual no NVMe2:
  - Reusar EFI System Partition existente (não reformatar!)
  - Criar partição BTRFS principal com subvolumes (@, @home, @log, @cache, etc.)
  - Sem swap em disco (vamos usar zram depois)
- Bootloader: GRUB na ESP existente
- Não criptografar (BitLocker do Windows é separado, criptografia adicional complica snapshots)
- **Documento:** `m0-instalar-kubuntu.md` (a ser criado quando chegar a hora)

### M0.5 — Pós-instalação imediata
- Validar dual boot funcional (boot Windows e Kubuntu pelo GRUB)
- Atualizar sistema (`apt update && apt full-upgrade`)
- Instalar HWE kernel (`linux-generic-hwe-24.04`)
- Instalar driver NVIDIA proprietário (PPA `graphics-drivers`)
- Instalar codecs proprietários (`ubuntu-restricted-extras`)
- Configurar Snapper para o subvolume `@` (root)
- Instalar `grub-btrfs` para snapshots no menu de boot
- **Documento:** `m0-pos-instalacao.md` (a ser criado quando chegar a hora)

### M0.6 — Validação final
- Testar reboot múltiplas vezes (Windows ↔ Kubuntu)
- Tirar primeiro snapshot manual com Snapper
- Validar que NVIDIA + Wayland funcionam (sem freezes)
- Confirmar que BTRFS está saudável (`btrfs filesystem usage /`)
- **Documento:** `m0-validacao.md` (a ser criado quando chegar a hora)

**Critério de "M0 concluído":**
- ✅ Windows e Kubuntu bootam consistentemente pelo GRUB
- ✅ Kubuntu funciona com NVIDIA acelerada
- ✅ Snapshot Snapper funcional
- ✅ Sistema apto para uso diário básico (web, terminal, gerenciamento de arquivos)

---

## M1 — Linux fluente do dia a dia

**Objetivo:** Usar o Kubuntu de verdade por algumas semanas, construindo a base de configuração que todo o resto do projeto vai depender.

**Sub-passos previstos:**

- M1.1 — Apps essenciais (browser, terminal, editor, comunicação)
- M1.2 — Steam + Proton + jogos
- M1.3 — Stack de criação: Blender, Unreal Engine 5, Substance Painter
- M1.4 — Bootstrap script idempotente (instala tudo automaticamente em outra máquina)
- M1.5 — Dotfiles versionados em git (decisão pendente: chezmoi vs GNU Stow)
- M1.6 — DAW e gravação de música (decisão pendente: Reaper vs Ardour)
- M1.7 — Distrobox para experimentos isolados
- M1.8 — Configuração KDE Plasma personalizada

**Critério de "M1 concluído":** sistema reproduzível via script + dotfiles, uso confortável diariamente.

---

## M2 — VM Windows básica no Linux (sem GPU passthrough)

**Objetivo:** Subir uma VM Windows simples no Kubuntu, **sem GPU passthrough** ainda. Permite tarefas leves (Office, Edge, alguns programas) sem rebootar. Performance ruim para Adobe.

**Sub-passos previstos:**

- M2.1 — Instalar QEMU/KVM + libvirt + virt-manager
- M2.2 — Criar VM Windows 11 do zero (não passthrough do disco físico ainda)
- M2.3 — Configurar VirtIO drivers
- M2.4 — TPM emulado para Windows 11
- M2.5 — Compartilhamento de pastas via VirtIO-FS

**Critério:** VM Windows acessível, conexão rede, compartilhamento de arquivos.

---

## M3 — Boot do Windows físico em VM (P2V live / Dual Reality)

**Objetivo:** A mesma instalação Windows do NVMe1 boota em **bare metal** OU como VM dentro do Linux. Sem reinstalar Windows, sem duas licenças.

**Sub-passos previstos:**

- M3.1 — Preparação: extrair SMBIOS exato com `dmidecode`
- M3.2 — Configurar libvirt XML com physical disk passthrough
- M3.3 — Replicar SMBIOS na VM para evitar reativação Windows
- M3.4 — Instalar VirtIO drivers no Windows (cuidado: drivers funcionam em ambos os modos)
- M3.5 — Validar que Windows boota igual em bare metal e VM

**Critério:** consigo bootar Windows em bare metal OU em VM, mesma instalação, sem reativar.

---

## M4 — Single GPU passthrough sob demanda

**Objetivo:** Quando precisar Windows pesado (Adobe, jogos AAA), sair da sessão Plasma, passar a NVIDIA pra VM, usar com performance bare metal, e voltar depois.

**Sub-passos previstos:**

- M4.1 — Validar IOMMU e isolation groups
- M4.2 — Configurar VFIO + dynamic unbind/rebind da NVIDIA
- M4.3 — Lidar com NVIDIA Code 43 (kvm=off, hidden=1)
- M4.4 — Looking Glass para mostrar tela da VM em janela do Linux
- M4.5 — Scripts de "swap dance" (parar Plasma → subir VM → desligar VM → voltar Plasma)

**Critério:** consigo lançar Windows com 95% de performance bare metal sob demanda, e voltar pro Linux sem reboot.

---

## M5 — Dual GPU passthrough na dock (final game)

**Objetivo:** Conectado à dock Lenovo TB4, rodar **Linux na iGPU + Windows VM na NVIDIA simultaneamente**, em monitores diferentes. Sem precisar parar nada.

**Sub-passos previstos:**

- M5.1 — Validar que P16v Gen 2 é MUXed (provável)
- M5.2 — Validar IOMMU groups com tudo conectado na dock
- M5.3 — Configurar Linux para rodar na iGPU Intel Arc Pro
- M5.4 — Passthrough permanente da NVIDIA pra VM Windows
- M5.5 — Mapear monitores corretamente
- M5.6 — Sem dock: fallback para M4 (single GPU swap)

**Critério:** uso simultâneo, dois monitores, dois SOs, dois usos paralelos.

---

## Milestones do servidor (paralelo, depois do M1)

### MS-A — Servidor TD350: Proxmox + VM Kubuntu para IA
- Backup do estado atual do servidor (verificar o que tem nele hoje)
- Instalar Proxmox VE 8
- Configurar pool ZFS
- IOMMU + VFIO para passthrough da 4090
- Criar VM Kubuntu (mesmo bootstrap script do notebook)
- Stack de IA: Ollama, ComfyUI, vLLM, llama.cpp

### MS-B — VM Windows no servidor (Adobe/CAD remoto)
- Criar VM Windows 11 com 4090 alternada
- Instalar Adobe Suite, 3ds Max, SolidWorks
- Sunshine + Moonlight para acesso remoto do notebook
