# WORKSTATION AI 3D

> Documentação completa, passo a passo, da construção de uma estação de trabalho híbrida focada em **Inteligência Artificial**, **3D**, **criação de conteúdo** e **gaming**, em duas máquinas: um notebook profissional e um servidor.

**Estado:** 🚧 Em construção (Maio/2026 — M0.1 concluído)
**Licença:** MIT
**Idioma da documentação:** Português (Brasil)

---

## 📋 O que é este projeto

Este repositório documenta — em detalhe e passo a passo — todo o processo de configurar duas máquinas Linux para uso profissional misto:

- **Notebook Lenovo ThinkPad P16v Gen 2** (estação móvel, dual boot Windows + Kubuntu)
- **Servidor Lenovo ThinkServer TD350** (Proxmox VE, hospedando VMs de IA)

A documentação é escrita para que **qualquer pessoa**, mesmo sem experiência prévia com Linux, consiga reproduzir cada passo. Cada comando é explicado, cada termo técnico tem definição no [glossário](docs/glossario.md), e cada decisão importante está registrada em [decisões](docs/decisoes.md).

---

## 🖥️ Hardware

### Notebook — Lenovo ThinkPad P16v Gen 2 (2024)

| Componente | Especificação |
|---|---|
| CPU | Intel Core Ultra 9 185H (Meteor Lake, 16C/22T, turbo 5.1 GHz) |
| GPU dedicada | NVIDIA RTX 3000 Ada Generation Laptop (8 GB GDDR6, AD106GLM) |
| GPU integrada | Intel Arc Pro Graphics (Xe-LPG) |
| RAM | 32 GB DDR5 (2× 16 GB SK Hynix SODIMM) |
| Armazenamento | **Disk 0:** UMIS RPETJ 1 TB (Linux) • **Disk 1:** BIWIN NV7400 4 TB (Windows + D: NTFS staging) |
| Wi-Fi | Intel Wi-Fi 6E AX211 |
| Ethernet | Intel I219-LM |
| Segurança | TPM 2.0 STM • Secure Boot Disabled |
| BIOS atual | Lenovo **N44ET37W v1.20** (atualizada de v1.18 em 02/05/2026 — M0.1.5) |
| vBIOS NVIDIA atual | **95.06.31.40.1c** (atualizada de .01 em 02/05/2026 — M0.1.5) |
| Configuração gráfica | Hybrid Graphics ativo, **MUXless** (confirmado em M0.1.6) |

### Servidor — Lenovo ThinkServer TD350

| Componente | Especificação |
|---|---|
| CPU | 2× Intel Xeon E5-2643 (v3 ou v4 — confirmar) |
| GPU | NVIDIA RTX 4090 (24 GB GDDR6X) |
| RAM | 512 GB DDR4 ECC |
| Storage | A confirmar quando chegarmos no servidor |

### Periféricos relevantes

- **Dock:** Lenovo Universal Thunderbolt 4 Dock (oficial)
- **Pendrive:** Ventoy com ISOs do Hiren's BootCD PE (com Macrium Reflect dentro) e Kubuntu 24.04 LTS — está em **Disk 2 (SSK Portable SSD 1 TB) via USB** (E: + F: bkpunvsal)
- **SSD externo dedicado** (a plugar quando chegar M0.3) — destino do clone do EndeavourOS antes de wipar

---

## 🎯 Decisões principais

Resumo executivo. Veja [docs/decisoes.md](docs/decisoes.md) para o histórico completo.

| # | Decisão | Escolha |
|---|---|---|
| D-001 | Distribuição Linux | **Kubuntu 24.04 LTS + HWE kernel** |
| D-002 | Estratégia notebook | Dual boot Windows 11 (principal) + Kubuntu (secundário) |
| D-003 | Estratégia servidor | Proxmox VE 8 + VM Kubuntu para IA |
| D-004 | Bootloader | GRUB (substituindo o systemd-boot atual do EndeavourOS) |
| D-005 | Sistema de arquivos | BTRFS com Snapper (snapshots) |
| D-006 | Backup do Windows | **Sem backup** (decisão consciente) |
| D-007 | Backup do EndeavourOS | Macrium Reflect via Hiren's PE no Ventoy → SSD externo dedicado |
| D-008 | Adobe + Autodesk | Windows nativo (não tentar em Linux) |
| D-013 | D: NTFS staging | Mount automático RW no Kubuntu via fstab (M0.7) |

Decisões pendentes: **D-009** (dotfiles), **D-010** (DAW). Decisões canceladas: D-011 (backup pessoal), D-012 (Adobe via Wine).

---

## 🗺️ Roadmap em 6 milestones

| # | Milestone | Status |
|---|---|---|
| **M0** | Setup base — dual boot Windows + Kubuntu funcional | 🚧 M0.1 ✅, M0.2 próximo |
| M1 | Linux fluente do dia a dia (uso real, dotfiles, scripts) | ⏸️ Aguardando M0 |
| M2 | VM Windows básica no Linux (sem GPU passthrough) | ⏸️ |
| M3 | Boot do Windows físico em VM (physical disk passthrough) | ⏸️ |
| M4 | Single GPU passthrough (Windows VM com a NVIDIA) | ⏸️ |
| M5 | Dual GPU passthrough na dock (final game) | ⏸️ |

Detalhes de cada milestone em [docs/milestones.md](docs/milestones.md).

### M0 sub-passos

| # | Sub-passo | Status |
|---|---|---|
| M0.1 | Tweaks Windows + atualizações firmware | ✅ Concluído 02/05/2026 |
| M0.2 | Coleta de estado no EndeavourOS | ⏸️ Próximo |
| M0.3 | Clone EndeavourOS (Macrium → SSD externo) | ⏸️ |
| M0.4 | Instalação Kubuntu (alvo: Disk 0 / UMIS) | ⏸️ |
| M0.5 | Pós-instalação (drivers, codecs, descompactadores, Git) | ⏸️ |
| M0.6 | Validação dual boot + Snapper | ⏸️ |
| **M0.7** | **Mount automático D: NTFS (D-013 nova)** | ⏸️ |

---

## 📂 Estrutura do repositório

```
workstation-ai-3d/
├── STATE.md               ← 🧭 GPS do projeto (LER PRIMEIRO em qualquer chat novo)
├── README.md              ← este arquivo (visão geral)
├── LICENSE                ← MIT
├── .gitignore
├── docs/
│   ├── decisoes.md        ← histórico de TODAS as decisões com justificativas (D-001 a D-013)
│   ├── milestones.md      ← roadmap detalhado dos milestones
│   ├── glossario.md       ← termos técnicos explicados em português
│   └── m0-*.md            ← documentos passo a passo do milestone atual
└── cheatsheet/
    └── README.md          ← índice de comandos úteis (preenchido conforme execução real)
```

---

## 🚀 Como usar este repositório

### Se você está acompanhando do zero

Leia nessa ordem:

1. [`STATE.md`](STATE.md) — **GPS do projeto** (onde estamos agora; sempre o ponto de entrada)
2. Este `README.md` (visão geral e contexto)
3. [`docs/decisoes.md`](docs/decisoes.md) — entenda o porquê de cada escolha
4. [`docs/milestones.md`](docs/milestones.md) — veja o plano completo
5. [`docs/glossario.md`](docs/glossario.md) — referência de termos
6. Documentos `docs/m0-*.md` — passos executáveis do milestone atual (`m0-windows-tweaks.md` já tem o histórico completo do M0.1)

### Se você é o autor do projeto

- Cada decisão nova vai em `docs/decisoes.md` com data e justificativa
- Cada comando que você roda em qualquer máquina vira entrada em `cheatsheet/`
- Cada milestone executado tem seu próprio doc `m{N}-{nome}.md`
- Após cada sessão: STATE.md atualizado (versão bumpada se mudança real de plano), commit + push, re-upload no Project Knowledge

---

## 🤝 Contribuições

Este é um projeto pessoal documentado publicamente para servir de referência. Issues e sugestões são bem-vindas, mas o foco é registrar este caminho específico, não generalizar.

---

## 📜 Licença

MIT — veja [LICENSE](LICENSE).
