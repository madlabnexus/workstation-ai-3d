# STATE — Onde estamos agora

> 🧭 **Este é o GPS do projeto.** Ponto de entrada para qualquer chat novo (com IA ou colaborador humano). Sempre atualizado ao final de cada sessão de trabalho.
>
> **📅 Última atualização:** 02/05/2026 (v5: M0.1 CONCLUÍDO — Windows preparado para dual boot com BIOS atualizada, vBIOS NVIDIA atualizada com backup, mapa real de discos corrigido, MUXless confirmado, D-013 nova sobre D: NTFS, M0.7 novo)
> **🔢 Versão do estado:** 5
> **🎯 Milestone atual:** M0 — Setup base do dual boot
> **📍 Sub-passo atual:** M0.1 ✅ concluído. Pronto para M0.2 (coleta de estado no EndeavourOS).

---

## 📌 Resumo executivo (1 parágrafo)

M0.1 fechado em sessão única no Windows. **Windows preparado para conviver com Linux:** Fast Startup desligado, hibernação desabilitada (24-32 GB liberados), BitLocker confirmado off em todos os volumes, BIOS atualizada (1.18 → 1.20), vBIOS NVIDIA atualizada (.01 → .1c) com backup salvo. **Mapa real do hardware descoberto** (era invertido no STATE v4). **Hardware classificado MUXless** (sem opção Discrete Graphics no BIOS). **Próxima ação:** M0.2 — bootar EndeavourOS atual e coletar estado real (efibootmgr, lsblk, blkid, fstab) antes de qualquer mudança destrutiva.

---

## ✅ O que foi concluído (cronológico, mais recente embaixo)

### Fase de planejamento (concluída em 02/05/2026)

- [x] Análise completa do hardware via HWiNFO (`p16v.LOG`)
- [x] Pesquisa extensa de distros Linux 2026 (Nobara, Fedora, Ubuntu, Bazzite, Kubuntu)
- [x] Pesquisa de status atual de softwares profissionais em Linux (Adobe, Autodesk)
- [x] **8 decisões formais registradas** em `docs/decisoes.md`:
  - D-001: Distro = Kubuntu 24.04 LTS + HWE
  - D-002: Notebook = dual boot Windows (principal) + Kubuntu
  - D-003: Servidor = Proxmox + VM Kubuntu
  - D-004: Bootloader = GRUB (substituir systemd-boot atual)
  - D-005: Filesystem = BTRFS + Snapper
  - D-006: SEM backup do Windows (decisão consciente do usuário)
  - D-007: Backup do EndeavourOS via Macrium/Hiren no Ventoy
  - D-008: Adobe + Autodesk só no Windows
- [x] Roadmap de 6 milestones (M0-M5) + milestones do servidor (MS-A, MS-B)
- [x] Glossário com 11 categorias e ~150 termos técnicos

### Fase de setup do repositório (concluída em 02/05/2026)

- [x] Estrutura do projeto criada localmente (8 arquivos)
- [x] Git instalado no Windows: `git version 2.54.0.windows.1`
- [x] GitHub CLI instalado: `gh version 2.92.0 (2026-04-28)`
- [x] Configuração global do Git aplicada
- [x] Privacidade GitHub ativada
- [x] PAT criado e `gh auth login` concluído — `Logged in as madlabnexus`
- [x] Repo `workstation-ai-3d` criado e push inicial feito
- [x] Push para GitHub: https://github.com/madlabnexus/workstation-ai-3d (público)

### Fase de execução M0.1 (concluída em 02/05/2026 — esta sessão)

**Sequência real executada (com sub-passos M0.1.x adicionais que emergiram na execução):**

- [x] **M0.1.1** — PowerShell aberto como administrador, validado com `IsInRole(Administrator) = True`
- [x] **M0.1.2** — `powercfg /h off` executado (desabilita Fast Startup + Hibernação juntos, libera ~24-32 GB do `hiberfil.sys`)
- [x] **M0.1.3** — `powercfg /a` confirmou: Hibernate/Fast Startup/Hybrid Sleep todos em "not available" com razão `"Hibernation has not been enabled"`. Modern Standby (S0 Low Power Idle) ativo, S3 não suportado pelo firmware (esperado em Meteor Lake). **Achado:** Hyper-V/VBS detectado ativo (`The hypervisor does not support this standby state` no Hybrid Sleep)
- [x] **M0.1.4** — `manage-bde -status` confirmou BitLocker **OFF** em todos os volumes (C:, D:, E:, VTOYEFI, F:). Sem ação de suspender necessária
- [x] **M0.1.4.1 (sub-passo emergente)** — Mapa real dos discos via `Get-Partition` + `Get-Disk`: descoberto que STATE v4 estava com NVMe1/NVMe2 invertidos. Mapa correto: **Disk 0 (UMIS) = Linux**, **Disk 1 (BIWIN) = Windows + D:**, **Disk 2 (SSK USB) = Ventoy + F: (ignorar)**
- [x] **M0.1.5.1** — Versão BIOS atual confirmada via `Get-CimInstance Win32_BIOS`: `N44ET35W v1.18`, release 27/08/2025. Bate com `p16v.LOG`
- [x] **M0.1.5.2** — vBIOS NVIDIA atual obtida via `nvidia-smi`: `95.06.31.40.01`, driver `595.71`. Subsystem ID `17AA-232D` (Lenovo P16v Gen 2 com RTX 3000 Ada)
- [x] **M0.1.5.3.A** — Flash da BIOS principal: pacote `n44uj11w` (UEFI 1.20 + ECP 1.10) baixado e executado como admin. Memory training ~2 min após reboot. Validado pós-flash: **`N44ET37W v1.20`**, release 10/12/2025
- [x] **M0.1.5.3.B (sub-passo emergente)** — Backup da vBIOS NVIDIA antes do flash. Memory Integrity (HVCI) precisou ser desligado pra `nvflash64` funcionar (depois reativado). Comando: `nvflash64.exe --index=0 --protectoff` seguido de `nvflash64.exe --index=0 --save vbios-p16v_bkp-95.06.31.40.01.rom`. Arquivo de 2.048.000 bytes salvo e copiado pra `C:\Users\<user>\bkp-bios\`
- [x] **M0.1.5.3.C** — Flash da vBIOS NVIDIA: pacote `n44vw02w_v3` executado. Validado: **`95.06.31.40.1c`** (mesma família/major/sub-vendor/ROM, apenas patch level subiu `.01 → .1c` em hex)
- [x] **M0.1.6** — Config gráfica coletada: ambas GPUs em `Status: OK` (Intel Arc Pro `8086:7D55` + NVIDIA `10DE:2838`), confirma Hybrid Graphics ativo. **Achado:** P16v Gen 2 é **MUXless** — verificação visual no BIOS Setup confirmou que opção `Discrete Graphics` **não existe** (só Hybrid)
- [x] **M0.1.6.1 (sub-passo emergente)** — Anomalia detectada via WMI: `BootMode = Diagnostics` (provável side-effect do flash da BIOS). Corrigido manualmente no BIOS Setup pra `Auto`/`UEFI`
- [x] **M0.1.7** — Settings de virtualização e segurança coletados via WMI Lenovo (sem reboot adicional): `VirtualizationTechnology = Enable` (VT-x ✅), `VTdFeature = Enable` (VT-d ✅), `SecurityChip = Enable` (TPM ✅), `SecureBoot = Disable` ✅
- [x] **D-013 nova** — D: NTFS como staging RW compartilhado entre Windows ↔ Linux, mount automático no Kubuntu via fstab (executar em M0.7)
- [x] **M0.7 novo** adicionado ao roadmap — configurar mount automático do D: NTFS no Kubuntu

---

## 🎯 Próximo passo concreto

**M0.2 — Coleta do estado atual no EndeavourOS** (documento `m0-coleta-estado.md` a ser criado quando chegar a hora)

Sequência prevista:

1. Bootar EndeavourOS atual (atalho F12 no boot Lenovo, escolher entry do UMIS — Disk 0)
2. Abrir terminal e coletar estado real (cada comando vai pro cheatsheet):
   - `efibootmgr -v` — entries UEFI atuais
   - `lsblk -f` — layout completo de discos com filesystems
   - `blkid` — UUIDs e tipos de partição
   - `cat /etc/fstab` — montagem atual configurada no Endeavour
   - `lspci -v` — confirma classificação de PCI devices vs Windows
   - `lscpu` — confirma topologia hybrid (P-cores + E-cores + LP-cores)
   - `inxi -Fxxxz` — overview saneado
3. Documentar layout exato da EFI System Partition (provavelmente compartilhada no BIWIN ou separada no UMIS — cada NVMe parece ter sua própria ESP, baseado em `Get-Partition`)
4. Validar quanto está ocupado de fato no Endeavour (`df -h` + `du -sh /home`) pra dimensionar tamanho do clone em M0.3
5. Reportar tudo de volta para iniciar M0.3 (clone via Macrium)

---

## 🔖 TODOs explícitos do usuário (a slotar em milestones)

Requisitos declarados pelo usuário na abertura do projeto que precisam ser registrados explicitamente para não esquecer durante a execução. Já refletidos em `milestones.md`:

- [ ] **Microsoft Edge para Office 365** → entra em **M1.1** (apps essenciais). Build oficial Edge for Linux disponível como `.deb` direto da Microsoft.
- [ ] **Emuladores (retro, DOS, Windows antigo)** → **M1.9**: RetroArch, DOSBox-Staging, 86Box ou PCem, QEMU desktop para Win98/XP.
- [ ] **DAW para gravação de música** → **D-010 pendente** (Reaper proprietário vs Ardour open source). Decidir antes de M1.6.
- [ ] **Ferramentas de descompactação** → entra em **M0.5** via `p7zip-full`, `unrar`, `unzip`, `zstd`, `xz-utils`.
- [ ] **Git no Linux** → entra em **M0.5** (no Windows já está, falta replicar).
- [ ] **Dotfiles auto-reinstaláveis** → **D-009 pendente** (chezmoi vs GNU Stow vs custom). Decidir antes de M1.5.
- [ ] **Bootstrap script idempotente** → previsto em **M1.4**.
- [ ] **D: NTFS como staging compartilhado** → mount automático em **M0.7** (novo).

---

## 🧠 Contexto crítico para retomar

### Estado real da máquina (corrigido na v5 — ATENÇÃO: era invertido na v4!)

| Disco físico | Modelo | Tamanho | Bus | Conteúdo atual |
|---|---|---|---|---|
| **Disk 0** | UMIS RPETJ1T24MKP2QDQ | ~1.02 TB | NVMe interno | **🐧 EndeavourOS** (será substituído por Kubuntu em M0.4) |
| **Disk 1** | BIWIN NV7400 4TB | ~4.10 TB | NVMe interno | **🪟 Windows 11 (C:) + Data NTFS (D:)** — não tocar |
| **Disk 2** | SSK Portable SSD 1TB | ~1.02 TB | USB externo | **Ventoy (E:) + bkpunvsal (F:)** — pendrive de boot, ignorar |

**Particionamento detalhado:**

- **Disk 0 (UMIS, Linux):** EFI 2GB + root Linux 918GB + swap 33.9GB
- **Disk 1 (BIWIN, Windows):** MSR 16MB + C:Windows 1.81TB + EFI 4GB + Recovery 735MB + D:Data 2.00TB
- **Disk 2 (SSK USB):** E:Ventoy 454GB + VTOYEFI 32MB + F:bkpunvsal 500GB

**Implicação pro projeto:**
- Quando M0.6 (instalar Kubuntu), **alvo é o UMIS (Disk 0)**, não o BIWIN
- Cada NVMe interno tem sua própria ESP — robusto: se GRUB der pau, F12 boota Windows direto via ESP do BIWIN
- D: NTFS de 2TB é staging compartilhado (D-013 nova) — montar via fstab em M0.7
- SSD externo dedicado pro M0.3 (clone Endeavour) **ainda não plugado** — primeira pergunta no M0.3 vai ser tamanho/modelo dele

### Estado real do BIOS/firmware (após M0.1)

| Componente | Antes | Depois (v5) |
|---|---|---|
| BIOS principal | `N44ET35W v1.18` (27/08/2025) | **`N44ET37W v1.20`** (10/12/2025) |
| ECP (Embedded Controller) | desconhecido | v1.10 (junto com BIOS principal) |
| vBIOS NVIDIA | `95.06.31.40.01` (Build 30/08/2023, Mod 19/02/2024) | **`95.06.31.40.1c`** |
| Driver NVIDIA Windows | `595.71` | `595.71` (não tocado) |
| BootMode | `Diagnostics` (side-effect do flash) | **`Auto`/`UEFI`** (corrigido) |

**Configuração gráfica confirmada (M0.1.6):**
- Hybrid Graphics ativo (Intel Arc Pro Graphics + NVIDIA RTX 3000 Ada)
- **MUXless** — opção `Discrete Graphics` não existe no BIOS Setup
- Implicação Linux: usar PRIME render offload ou envycontrol pra alternar dGPU
- Implicação M4-M5: GPU passthrough sem display dedicado, vai requerer Looking Glass / Sunshine / rede

**Settings de segurança/virtualização (M0.1.7):**

| Setting | Valor | OK? |
|---|---|---|
| `SecureBoot` | Disable | ✅ pré-req Kubuntu sem dor de signed kernel |
| `VirtualizationTechnology` (VT-x) | Enable | ✅ pré-req M2 |
| `VTdFeature` (VT-d) | Enable | ✅ pré-req M4-M5 |
| `SecurityChip` (TPM 2.0) | Enable | ✅ |
| `KernelDMAProtection` | Disable | ✅ Linux gerencia IOMMU |
| `TotalMemoryEncryption` | Disable | ✅ TME ligado quebra hibernate Linux |
| `BootOrder` | NVMe0 (UMIS=Linux) primeiro | ✅ |

### Decisões que mudaram durante o planejamento (importante para não confundir)

- **Distro:** Nobara → Fedora → Kubuntu 24.04 LTS (decisão final, mantida)
- **Backup do Windows:** plano original tinha; decisão final é **NÃO fazer**
- **Bootloader:** systemd-boot atual será substituído por **GRUB** (decisão por simplicidade)
- **Hiren's BootCD:** já está no Ventoy junto com Kubuntu LTS — **pendrive pronto**
- **D: NTFS:** novo requisito (D-013) — staging compartilhado RW, mount automático em M0.7
- **MUXless confirmado:** anteriormente assumido "MUXed provavelmente" no glossário e milestones; agora é fato. Implica em retrabalho cosmético em M5.1 e referências do glossário (não bloqueia avanço)

### Premissas operacionais

- **Modo de trabalho:** um comando por vez, esperar output, validar, próximo passo
- **Documentação:** "passo a passo para criança" — sem assumir conhecimento técnico
- **Cada comando rodado real** vira entrada no `cheatsheet/README.md` (em batch ao final da sessão)
- **Cada decisão importante** vira ADR em `docs/decisoes.md` (D-XXX)
- **Identidade pública:** `madlabnexus` em qualquer arquivo público; **nome real do usuário NÃO entra em arquivos públicos**

### Coisas que ainda NÃO temos certeza (descobrir conforme avança)

- Layout exato da EFI System Partition: cada NVMe tem a sua, mas qual o GRUB vai usar no M0.4 — vamos descobrir em M0.2
- Versão atual do EndeavourOS instalado
- O que existe de relevante no EndeavourOS atual (vai ser apagado — clone é só backup)
- Tamanho/modelo do SSD externo destinado ao clone (M0.3)
- Anomalia cosmética do `nvidia-smi` reportando `Pwr:Usage 364W / 55W` em idle (provável bug de driver vs vBIOS nova; não impacta funcionalidade; pode resolver com driver mais novo no Linux)

---

## 📚 Arquivos do projeto (mapa)

| Arquivo | Função | Quando consultar |
|---|---|---|
| `STATE.md` | **Este arquivo.** GPS do projeto | Sempre, primeiro |
| `README.md` | Visão geral do projeto | Para ter mapa mental |
| `docs/decisoes.md` | 13 decisões formais com justificativas (D-013 nova) | Antes de questionar uma escolha |
| `docs/milestones.md` | Roadmap M0-M5 e sub-passos | Para entender direção |
| `docs/glossario.md` | ~160 termos técnicos | Quando aparecer termo desconhecido |
| `docs/m0-windows-tweaks.md` | Sub-passo M0.1 detalhado (concluído, agora histórico/referência) | Pra reinstalação futura |
| `cheatsheet/README.md` | Comandos validados (todos os do M0.1 + futuros) | Quando esquecer comando que já rodamos |
| `p16v.LOG` | Especificações exatas do hardware | Para discussões de hardware |

---

## 🤖 Para retomar conversa em chat novo

Se você está abrindo um chat NOVO comigo neste projeto, este é o approach mais eficiente:

```
"Vamos continuar o projeto workstation-ai-3d. 
Leia STATE.md, decisoes.md e milestones.md do project knowledge 
e me confirma onde estamos antes de prosseguir."
```

A IA vai automaticamente:
1. Buscar esses 3 arquivos no project knowledge
2. Fazer um resumo do estado atual
3. Confirmar contigo o próximo passo
4. Esperar sua ordem para prosseguir

---

## 🗂️ Convenção de nomes (project knowledge ↔ repositório GitHub)

O **project knowledge** do Claude não permite nomes de arquivo duplicados (não tem hierarquia de pastas). Como o repositório tem dois `README.md` (um na raiz e um em `cheatsheet/`), foi necessário renomear ao subir no PK. **No repositório GitHub os nomes seguem a árvore normal**; no PK eles ganham um sufixo descritivo.

| Nome no PK (Claude) | Caminho no repo (GitHub) | Função |
|---|---|---|
| `STATE.md` | `STATE.md` | GPS do projeto (mesmo nome) |
| `README-projeto.md` | `README.md` | README raiz do repo |
| `README-cheatsheet.md` | `cheatsheet/README.md` | Cheatsheet de comandos |
| `decisoes.md` | `docs/decisoes.md` | ADRs |
| `milestones.md` | `docs/milestones.md` | Roadmap |
| `glossario.md` | `docs/glossario.md` | Glossário |
| `m0-windows-tweaks.md` | `docs/m0-windows-tweaks.md` | Doc do M0.1 |
| `p16v.LOG` | `p16v.LOG` | Log HWiNFO |

**Regra prática ao editar:**
- **Para subir no PK do Claude:** usar o nome com sufixo (`README-projeto.md`, `README-cheatsheet.md`).
- **Para commitar no repo GitHub:** usar o caminho com pasta (`README.md`, `cheatsheet/README.md`, `docs/...`).
- **Os arquivos com sufixo são SOMENTE convenção do PK.** Em qualquer link, citação ou comando, sempre referenciar pelo nome no repositório.

---

## 🔄 Manutenção deste arquivo

**Quando atualizar:**

- ✅ Ao concluir um sub-passo de milestone (`M0.1` → `M0.2`)
- ✅ Ao tomar decisão nova (D-009, D-010, D-013...)
- ✅ Ao descobrir gotcha importante durante execução
- ✅ Ao mudar de plano (raro, mas acontece — vide histórico de mudanças de distro e mapa de discos)

**Como atualizar:**

1. Editar este arquivo (`STATE.md`) localmente
2. Atualizar campo "Última atualização" com a data atual
3. Incrementar "Versão do estado"
4. Adicionar entrada em "O que foi concluído"
5. Atualizar "Próximo passo concreto"
6. Revisar "Contexto crítico" (remover o que não é mais relevante, adicionar novidades)
7. `git add STATE.md && git commit -m "STATE: <resumo da mudança>" && git push`
8. **Re-upload** no Project Knowledge do Claude

**Princípio:** este arquivo deve **caber em uma tela** ao ler em terminal. Se ficou grande demais, mover detalhes históricos para `decisoes.md` ou `milestones.md`.
