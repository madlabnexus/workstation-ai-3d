# Roadmap dos Milestones

> Estratégia de progressão **incremental**, do simples ao complexo. Cada milestone entrega valor próprio — você pode parar em qualquer um e ainda ter um sistema produtivo.

---

## Visão geral

| # | Milestone | Tempo estimado | Status |
|---|---|---|---|
| **M0** | Setup base — dual boot Windows + Kubuntu funcional | 1-3 dias | 🚧 Em progresso (M0.1 ✅, M0.2 próximo) |
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

### M0.1 — Tweaks no Windows ✅ CONCLUÍDO em 02/05/2026

Sub-passos executados (ordem real, com sub-passos emergentes que apareceram durante execução):

- **M0.1.1** ✅ PowerShell aberto como administrador (validado com `IsInRole(Administrator)`)
- **M0.1.2** ✅ `powercfg /h off` — desabilitou Fast Startup + Hibernação juntos (~24-32 GB liberados)
- **M0.1.3** ✅ `powercfg /a` — confirmou Hibernate/Fast Startup/Hybrid Sleep todos em "not available". Modern Standby S0ix ativo (esperado em Meteor Lake)
- **M0.1.4** ✅ `manage-bde -status` — BitLocker OFF em todos os volumes (C:, D:, E:, F:, VTOYEFI). Sem ação de suspender necessária
- **M0.1.4.1** ✅ (sub-passo emergente) Mapa real dos discos via `Get-Partition` + `Get-Disk`. **Descoberto que o STATE v4 estava com NVMe1/NVMe2 invertidos**:
  - Disk 0 = UMIS = Linux (EndeavourOS)
  - Disk 1 = BIWIN = Windows + D:
  - Disk 2 = SSK USB = Ventoy + F: (ignorar)
- **M0.1.5.1** ✅ Versão BIOS atual via `Get-CimInstance Win32_BIOS`: `N44ET35W v1.18` (release 27/08/2025)
- **M0.1.5.2** ✅ vBIOS NVIDIA atual via `nvidia-smi`: `95.06.31.40.01`, driver `595.71`, subsystem `17AA-232D`
- **M0.1.5.3.A** ✅ Flash da BIOS principal (`n44uj11w`) — `1.18 → 1.20`. Memory training ~2 min após reboot. Pós-flash: `N44ET37W v1.20` (release 10/12/2025)
- **M0.1.5.3.B** ✅ (sub-passo emergente) Backup da vBIOS NVIDIA via `nvflash64.exe --save`. Memory Integrity (HVCI) precisou ser desabilitado pra `nvflash64` funcionar (depois reativado). Arquivo de 2 MB salvo em `C:\Users\<user>\bkp-bios\vbios-p16v_bkp-95.06.31.40.01.rom`
- **M0.1.5.3.C** ✅ Flash da vBIOS NVIDIA (`n44vw02w_v3`) — `.01 → .1c`. Pós-flash: `95.06.31.40.1c`
- **M0.1.6** ✅ Config gráfica via WMI: ambas GPUs `Status: OK` (Intel Arc Pro + NVIDIA RTX 3000 Ada). **MUXless confirmado** — opção `Discrete Graphics` não existe no BIOS Setup
- **M0.1.6.1** ✅ (sub-passo emergente) Anomalia `BootMode = Diagnostics` (provável side-effect do flash da BIOS) corrigida no BIOS Setup pra `Auto`/`UEFI`
- **M0.1.7** ✅ Settings de virtualização e segurança coletados (sem reboot adicional): VT-x, VT-d, TPM, Secure Boot — todos no estado certo

**Achados que viraram entrada permanente no STATE.md:**

- Mapa real dos discos (UMIS=Linux, BIWIN=Windows, SSK USB=ignorar)
- Hyper-V/VBS ativo (Windows 11 Pro Workstation 25H2)
- Memory Integrity bloqueia ferramentas de firmware GPU — desabilitar temporariamente, reabilitar depois
- P16v Gen 2 é MUXless (sem opção Discrete no BIOS)
- Cada NVMe interno tem sua própria ESP (UMIS 2GB, BIWIN 4GB) — robustez de dual boot
- D: NTFS de 2TB no BIWIN é staging compartilhado (D-013) — mount em M0.7

**Documento de referência:** `docs/m0-windows-tweaks.md` (atualizado pra refletir o que foi REALMENTE executado, não o plano original)

### M0.2 — Coleta do estado atual no EndeavourOS

- Bootar EndeavourOS atual (F12, escolher entry do UMIS — Disk 0)
- Coletar via terminal: `efibootmgr -v`, `lsblk -f`, `blkid`, `cat /etc/fstab`, `lspci -v`, `lscpu`, `inxi -Fxxxz`
- Identificar layout exato da EFI System Partition (cada NVMe tem a sua, baseado em `Get-Partition`)
- Validar quanto está ocupado de fato no Endeavour: `df -h` + `du -sh /home` (pra dimensionar clone em M0.3)
- Confirmar topologia hybrid Meteor Lake (P-cores + E-cores + LP-cores) via `lscpu`
- Documentar layout exato do disco
- **Documento:** `m0-coleta-estado.md` (a ser criado quando chegar a hora)

### M0.3 — Clone do EndeavourOS via Macrium

- **Pré-requisito:** plugar SSD externo dedicado (cenário (a) confirmado em D-007 atualizada). Primeira pergunta no M0.3 será tamanho/modelo do SSD
- Bootar Ventoy → escolher Hiren's BootCD PE
- Abrir Macrium Reflect
- Clonar **Disk 0 (UMIS, ~1 TB)** inteiro para destino externo
- Verificar integridade do clone
- **Documento:** `m0-clone-endeavouros.md` (a ser criado quando chegar a hora)

### M0.4 — Instalação do Kubuntu 24.04 LTS

- Bootar Ventoy → escolher Kubuntu 24.04 LTS
- Particionamento manual no **Disk 0 (UMIS, NÃO BIWIN)**:
  - Reusar EFI System Partition existente do UMIS (não reformatar!)
  - Criar partição BTRFS principal com subvolumes (@, @home, @log, @cache, etc.)
  - Sem swap em disco (vamos usar zram depois)
- Bootloader: GRUB na ESP do UMIS
- Não criptografar (BitLocker do Windows é separado, criptografia adicional complica snapshots)
- Confirmar pós-instalação: F12 mostra entries pra UMIS (Linux) E BIWIN (Windows Boot Manager) — duas rotas de boot independentes
- **Documento:** `m0-instalar-kubuntu.md` (a ser criado quando chegar a hora)

### M0.5 — Pós-instalação imediata

- Validar dual boot funcional (boot Windows e Kubuntu pelo GRUB)
- Atualizar sistema (`apt update && apt full-upgrade`)
- Instalar HWE kernel (`linux-generic-hwe-24.04`)
- Instalar driver NVIDIA proprietário (PPA `graphics-drivers`)
- Instalar codecs proprietários (`ubuntu-restricted-extras`)
- **Instalar Git e ferramentas de desenvolvimento básicas:** `git`, `curl`, `wget`, `build-essential`
- **Instalar ferramentas de descompactação completas:** `p7zip-full`, `p7zip-rar`, `unrar`, `unzip`, `zip`, `zstd`, `xz-utils`, `tar`, `gzip`, `bzip2`
- Configurar Snapper para o subvolume `@` (root)
- Instalar `grub-btrfs` para snapshots no menu de boot
- **Documento:** `m0-pos-instalacao.md` (a ser criado quando chegar a hora)

### M0.6 — Validação final

- Testar reboot múltiplas vezes (Windows ↔ Kubuntu)
- Tirar primeiro snapshot manual com Snapper
- Validar que NVIDIA + Wayland funcionam (sem freezes — ou X11 se Wayland NVIDIA der problema)
- Confirmar que BTRFS está saudável (`btrfs filesystem usage /`)
- **Documento:** `m0-validacao.md` (a ser criado quando chegar a hora)

### M0.7 — Mount automático do D: NTFS (NOVO em D-013)

> 📝 **Adicionado em 02/05/2026 (v5):** D-013 estabeleceu que D: NTFS no BIWIN será montado RW no Kubuntu como staging compartilhado. M0.7 é o sub-passo de execução.

- Identificar UUID do D: via `blkid` (vai aparecer como NTFS)
- Criar mount point: `sudo mkdir -p /mnt/data` (ou `/data` — decidir convenção)
- Adicionar em `/etc/fstab`:
  ```
  UUID=<uuid-do-D:>  /mnt/data  ntfs3  defaults,nofail,uid=1000,gid=1000,umask=022,windows_names  0  0
  ```
- Testar: `sudo mount -a` (não deve dar erro)
- Validar: `ls -la /mnt/data` (vai listar conteúdo do D: como o Windows vê)
- Reboot pra confirmar que monta automaticamente
- **Documento:** `m0-mount-d-ntfs.md` (a ser criado em M0.7)

**Critério de "M0 concluído":**
- ✅ Windows e Kubuntu bootam consistentemente pelo GRUB
- ✅ Kubuntu funciona com NVIDIA acelerada
- ✅ Snapshot Snapper funcional
- ✅ Sistema apto para uso diário básico (web, terminal, gerenciamento de arquivos)
- ✅ Git, curl/wget, e descompactadores instalados
- ✅ D: NTFS monta automaticamente em `/mnt/data` (M0.7)

---

## M1 — Linux fluente do dia a dia

**Objetivo:** Usar o Kubuntu de verdade por algumas semanas, construindo a base de configuração que todo o resto do projeto vai depender.

**Sub-passos previstos:**

### M1.1 — Apps essenciais
- **Microsoft Edge for Linux** (`.deb` oficial Microsoft) — para Office 365, Teams web, AutoCAD Web
- **Firefox** — backup browser
- Terminal: Konsole (já vem) + `tmux` ou `zellij`
- Editor: VSCode (`.deb` oficial Microsoft)
- Comunicação: Telegram, Discord, Slack (Flatpak ou `.deb`)
- Office: LibreOffice (já vem com Kubuntu) + OnlyOffice (alternativa de melhor compatibilidade `.docx`/`.xlsx`)
- PDF: Okular (já vem) + Master PDF Editor se precisar editar
- Cliente de email: Thunderbird

### M1.2 — Steam + Proton + jogos modernos
- Steam (`steam-installer` ou Flatpak)
- Proton-GE (via ProtonUp-Qt)
- Lutris (para jogos non-Steam, GOG, Battle.net)
- Heroic Games Launcher (Epic Games e GOG)
- MangoHud + GameMode

### M1.3 — Stack de criação 3D/2D nativa Linux
- Blender (Flatpak ou tarball oficial)
- Unreal Engine 5 (Epic Launcher via Heroic ou source build)
- Substance Painter (build oficial Adobe Linux via portal)
- Krita (já é nativo, alternativa a Photoshop para pintura)
- GIMP (alternativa básica a Photoshop)
- DaVinci Resolve (build oficial Linux — alternativa a Premiere)

### M1.4 — Bootstrap script idempotente
- Script bash em `bootstrap/install.sh` que instala TUDO automaticamente
- Lê listas modulares: `apt-packages.txt`, `flatpaks.txt`, `snap-packages.txt`, `manual-debs.txt`
- Idempotente: rodar de novo não estraga nada, só atualiza
- Roda em máquina nova após instalação Kubuntu limpa e reproduz todo o ambiente
- **Critério:** se eu reinstalar Kubuntu do zero, em <30 minutos rodando o script tenho tudo de volta

### M1.5 — Dotfiles versionados em git
- **D-009 pendente:** chezmoi vs GNU Stow vs solução custom
- Repo separado `madlabnexus/dotfiles` no GitHub (público ou privado, decidir)
- Inclui: shell config (bash/zsh), git config, KDE Plasma layouts, configs de apps específicos
- Integração com bootstrap script

### M1.6 — DAW e gravação de música
- **D-010 pendente:** Reaper (proprietário, leve, Linux nativo) vs Ardour (open source, robusto)
- Plugins: LSP Plugins, Calf, x42, Surge XT (sintetizador)
- Pipewire-jack (já vem por padrão no Kubuntu 24.04+)
- Drivers ASIO substitutos no Linux: pipewire low-latency
- Considerar: Carla (host de plugins), Hydrogen (drum machine)

### M1.7 — Distrobox para experimentos isolados
- Instalar Distrobox + Podman
- Containers: Arch para AUR, Fedora para testar pacotes RHEL, Alpine para builds minimal
- Permite testar coisas sem sujar o host

### M1.8 — Configuração KDE Plasma personalizada
- Tema, ícones, fontes
- KWin scripts (forçar regras de janela)
- Atalhos de teclado personalizados
- Activities (se quiser usar)
- Latte Dock (se quiser dock estilo macOS)

### M1.9 — Emuladores e jogos retro/legacy
- **RetroArch** (consoles retro: SNES, Mega Drive, PSX, PS2, GameCube, Switch via Ryujinx, etc.)
- **DOSBox-Staging** (jogos DOS — versão moderna do DOSBox clássico)
- **86Box** ou **PCem** (PC antigo até Pentium III com drivers de época)
- **QEMU** com VMs separadas para Win98/WinXP isolados (jogos antigos que não rodam em Win10/11 modernos)
- ScummVM (point-and-click clássicos: Monkey Island, Day of the Tentacle)
- PPSSPP (PSP), Dolphin (GameCube/Wii), PCSX2 (PS2)
- **Critério:** consigo rodar jogos das eras DOS, Win9x, PSX, PS2, retro consoles sem stress

### M1.10 — Tuning de performance (opcional, não-bloqueador)

> ⚠️ **Quando executar:** apenas **após 2+ semanas de uso real** do sistema com M1.1-M1.9 estabilizados. Tuning prematuro é otimização sem baseline — você não sabe o que precisa melhorar até usar de verdade.

**Objetivo:** Aproveitar características específicas do P16v Gen 2 (Meteor Lake hybrid cores, RTX 3000 Ada, 32GB DDR5, NVMe BIWIN 4TB) para workloads pesados (3D, IA local, gaming) **sem trocar estabilidade por benchmarks**.

**Princípio operacional:**
- Mudar **uma coisa por vez**, medir antes/depois (idealmente com benchmark real do workload, não synthetic puro)
- Cada tuning aplicado vira **ADR** (D-XXX) com motivação + benchmark mensurado
- Lista de coisas **testadas e descartadas** também vira ADR (evita reentrar no mesmo loop daqui a 6 meses)
- Snapshot Snapper ANTES de cada tuning não-trivial — rollback imediato se quebrar algo

**Sub-passos:**

#### M1.10.1 — Energia e térmico (laptop-specific)
- **TLP** vs **power-profiles-daemon** — decidir qual usar (cada um tem trade-offs em laptop ThinkPad). Vira ADR.
- **thermald** já vem ativo no Ubuntu — só validar
- **Profiles diferenciados:** bateria vs tomada vs dock Thunderbolt

#### M1.10.2 — CPU scheduler e governor
- **Intel P-State driver** — modo `passive` vs `active` (afeta como governor controla frequência)
- **Governor:** `powersave` (default) vs `performance` vs `schedutil`. Em laptop moderno geralmente `schedutil` é o melhor compromisso.
- **Hybrid scheduler do kernel:** já gerencia P-cores (6×) + E-cores (8×) + LPE-cores (2×) automaticamente. **Só intervir manualmente se houver problema mensurado** — não tunar profilaticamente.

#### M1.10.3 — Memória e swap
- **ZRAM** finalizado (configuração inicial vai em M0.4 — fechar otimização aqui)
- **`vm.swappiness`** — default 60 do Ubuntu costuma ser alto demais para SSD/NVMe; 10-20 geralmente melhor
- **`vm.vfs_cache_pressure`** — pode reduzir para manter mais cache em RAM
- **Limite tmpfs** se workload usar `/tmp` intensivamente

#### M1.10.4 — Storage NVMe
- **I/O scheduler `none`** para NVMe (vs `mq-deadline` default — em NVMe modernos, scheduler em software só atrapalha; o controlador do disco já faz queue management)
- **`fstrim.timer`** já vem ativo via systemd — validar
- **Mount options BTRFS:** `noatime,compress=zstd:3,space_cache=v2` (provavelmente já aplicado em M0.4 — revisar)

#### M1.10.5 — NVIDIA tuning
- **`nvidia-persistenced`** ativo — reduz latência de cold-start CUDA, útil em IA local
- **Power limit** via `nvidia-smi -pl` se houver thermal throttling em workload longo (ex: render Blender 1h+)
- **KMS modeset** (`nvidia-drm.modeset=1`) — geralmente já é default no PPA `graphics-drivers` atual; validar
- **PowerMizer mode:** `Adaptive` (default) vs `Maximum Performance` — só mudar se necessário, mata bateria
- **Bug cosmético conhecido:** após flash da vBIOS no M0.1.5, `nvidia-smi` no Windows reportou `Pwr:Usage 364W / 55W` em idle (lixo de leitura, não impacta funcionalidade). Revalidar no Linux com driver NVIDIA novo.

#### M1.10.6 — Gaming (overlap explícito com M1.2)
- **MangoHud** + **GameMode** já estarão instalados em M1.2 — aqui só validar funcionamento e ajustar profiles
- **Não duplicar instalação** — referenciar M1.2

#### M1.10.7 — Experimental / alto risco (OPCIONAL — documentado para registro)

> 🚨 **ATENÇÃO:** tudo abaixo é "executar **só com motivação clara e benchmark de baseline**, e snapshot Snapper recente". Nenhum item é recomendado por padrão. Lista existe para que seja considerada conscientemente, não como sugestão a aplicar.

- **Undervolting CPU** (`intel-undervolt`)
  - **Pode:** reduzir 5-10°C, ganhar margem antes de thermal throttling em sustained loads
  - **Risco:** voltagem errada → travamento aleatório, em casos extremos corrupção de dados
  - **Quando faz sentido:** se houver thermal throttling mensurado em workload real (não synthetic)
  - **Quando NÃO faz sentido:** só por status / "porque dá pra fazer"

- **`mitigations=off` no kernel cmdline**
  - **Pode:** ganhar 5-15% em workloads CPU-bound (especialmente compilação, IA com batch CPU)
  - **Risco:** desliga proteções contra Spectre/Meltdown e variantes — máquina fica vulnerável a side-channel attacks
  - **Quando faz sentido:** workstation isolada, sem código não-confiável rodando, sem browsing pesado/desconhecido
  - **Quando NÃO faz sentido:** máquina de uso geral com browser aberto o dia todo

- **Custom kernel** (Liquorix, XanMod, CachyOS-style patches)
  - **Pode:** scheduler mais agressivo, latência menor em desktop interativo
  - **Risco:** sai do suporte oficial do Ubuntu HWE — qualquer bug sério deixa de ter canal de fix
  - **Quando faz sentido:** depois de testar HWE padrão por meses e ter problema específico não resolvido
  - **Quando NÃO faz sentido:** "porque benchmarks na internet são melhores"

**Critério de "M1.10 concluído":**
- ✅ Cada tuning aplicado tem ADR documentando: motivação, comando exato, benchmark antes/depois (com workload real)
- ✅ Lista de tunings descartados também documentada (com motivo do descarte)
- ✅ Sistema mensurável melhor em **alguma** métrica relevante (responsividade, temperatura sustentada, autonomia em bateria, ou tempo de tarefa real específica)
- ✅ Nenhum tuning aplicado em produção sem snapshot Snapper anterior

**Critério de "M1 concluído":** sistema reproduzível via script + dotfiles, uso confortável diariamente, todos os softwares essenciais instalados. **M1.10 (tuning) é opcional e pode ser executado sob demanda após uso real — não bloqueia o avanço para M2.**

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

**Objetivo:** A mesma instalação Windows do Disk 1 (BIWIN) boota em **bare metal** OU como VM dentro do Linux. Sem reinstalar Windows, sem duas licenças.

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
- M4.4 — Looking Glass para mostrar tela da VM em janela do Linux (**necessário em hardware MUXless** — confirmado em M0.1.6)
- M4.5 — Scripts de "swap dance" (parar Plasma → subir VM → desligar VM → voltar Plasma)

**Critério:** consigo lançar Windows com 95% de performance bare metal sob demanda, e voltar pro Linux sem reboot.

> 📝 **Nota v5:** P16v Gen 2 confirmado **MUXless** em M0.1.6 — Looking Glass não é "alternativa nice-to-have", é **requisito** já que dGPU não tem saída de vídeo direta. Output da VM precisa passar pela iGPU pra chegar no display.

---

## M5 — Dual GPU passthrough na dock (final game)

**Objetivo:** Conectado à dock Lenovo TB4, rodar **Linux na iGPU + Windows VM na NVIDIA simultaneamente**, em monitores diferentes. Sem precisar parar nada.

**Sub-passos previstos:**

- M5.1 — ~~Validar que P16v Gen 2 é MUXed~~ → **Já confirmado em M0.1.6: P16v Gen 2 é MUXless.** Sub-passo se torna: **investigar limites do dual GPU em hardware MUXless** (dock TB4 pode dar saída separada à dGPU? saída via dock seria caminho)
- M5.2 — Validar IOMMU groups com tudo conectado na dock
- M5.3 — Configurar Linux para rodar na iGPU Intel Arc Pro
- M5.4 — Passthrough permanente da NVIDIA pra VM Windows
- M5.5 — Mapear monitores corretamente (com TB4 dock, possivelmente 1 saída direta da dGPU via dock, 1 da iGPU local)
- M5.6 — Sem dock: fallback para M4 (single GPU swap)

**Critério:** uso simultâneo, dois monitores, dois SOs, dois usos paralelos.

> 📝 **Nota v5:** Em hardware MUXless puro sem dock, **dual passthrough simultâneo não é viável** (dGPU não tem caminho de saída independente). Com dock TB4, **pode ser** que a dock dê saída direta da NVIDIA — investigar em M5.1 quando chegarmos lá. Realisticamente, M5 pode terminar como "M4 + dock" se hardware não permitir.

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
