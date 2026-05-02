# WORKSTATION AI 3D

> Documentação completa, passo a passo, da construção de uma estação de trabalho híbrida focada em **Inteligência Artificial**, **3D**, **criação de conteúdo** e **gaming**, em duas máquinas: um notebook profissional e um servidor.

**Estado:** 🚧 Em construção (Maio/2026)
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
| Armazenamento | NVMe1: UMIS 1 TB (Windows 11) • NVMe2: BIWIN NV7400 4 TB (Linux) |
| Wi-Fi | Intel Wi-Fi 6E AX211 |
| Ethernet | Intel I219-LM |
| Segurança | TPM 2.0 STM • Secure Boot Disabled |
| BIOS | Lenovo N44ET35W v1.18 |

### Servidor — Lenovo ThinkServer TD350

| Componente | Especificação |
|---|---|
| CPU | 2× Intel Xeon E5-2643 (v3 ou v4 — confirmar) |
| GPU | NVIDIA RTX 4090 (24 GB GDDR6X) |
| RAM | 512 GB DDR4 ECC |
| Storage | A confirmar quando chegarmos no servidor |

### Periféricos relevantes

- **Dock:** Lenovo Universal Thunderbolt 4 Dock (oficial)
- **Pendrive:** Ventoy com ISOs do Hiren's BootCD PE (com Macrium Reflect dentro) e Kubuntu 24.04 LTS
- **Disco externo:** SSK Portable SSD 1 TB (uso flexível)

---

## 🎯 Decisões principais

Resumo executivo. Veja [docs/decisoes.md](docs/decisoes.md) para o histórico completo.

| Decisão | Escolha |
|---|---|
| Distribuição Linux | **Kubuntu 24.04 LTS + HWE kernel** |
| Estratégia notebook | Dual boot Windows 11 (principal) + Kubuntu (secundário) |
| Estratégia servidor | Proxmox VE 8 + VM Kubuntu para IA |
| Sistema de arquivos | BTRFS com Snapper (snapshots) |
| Bootloader | GRUB (substituindo o systemd-boot atual do EndeavourOS) |
| Adobe + Autodesk | Windows nativo (não tentar em Linux) |
| Substance Painter | Linux nativo (build oficial Adobe Linux) |
| Blender / Unreal 5 | Linux nativo |
| Gaming | Steam + Proton no Linux |
| GPU passthrough notebook | **Adiado** — só após dominar Linux básico |

---

## 🗺️ Roadmap em 6 milestones

| # | Milestone | Status |
|---|---|---|
| **M0** | Setup base — dual boot Windows + Kubuntu funcional | 🚧 Em progresso |
| M1 | Linux fluente do dia a dia (uso real, dotfiles, scripts) | ⏸️ Aguardando M0 |
| M2 | VM Windows básica no Linux (sem GPU passthrough) | ⏸️ |
| M3 | Boot do Windows físico em VM (physical disk passthrough) | ⏸️ |
| M4 | Single GPU passthrough (Windows VM com a NVIDIA) | ⏸️ |
| M5 | Dual GPU passthrough na dock (final game) | ⏸️ |

Detalhes de cada milestone em [docs/milestones.md](docs/milestones.md).

---

## 📂 Estrutura do repositório

```
workstation-ai-3d/
├── README.md              ← este arquivo (visão geral)
├── LICENSE                ← MIT
├── .gitignore
├── docs/
│   ├── decisoes.md        ← histórico de TODAS as decisões com justificativas
│   ├── milestones.md      ← roadmap detalhado dos milestones
│   ├── glossario.md       ← termos técnicos explicados em português
│   └── m0-*.md            ← documentos passo a passo do milestone atual
└── cheatsheet/
    └── README.md          ← índice de comandos úteis (vai sendo preenchido)
```

---

## 🚀 Como usar este repositório

### Se você está acompanhando do zero

Leia nessa ordem:

1. Este `README.md` (você está aqui)
2. [`docs/decisoes.md`](docs/decisoes.md) — entenda o porquê de cada escolha
3. [`docs/milestones.md`](docs/milestones.md) — veja o plano completo
4. [`docs/glossario.md`](docs/glossario.md) — referência de termos
5. Documentos `m0-*.md` — passos executáveis do milestone atual

### Se você é o autor do projeto

- Cada decisão nova vai em `docs/decisoes.md` com data e justificativa
- Cada comando que você roda em qualquer máquina vira entrada em `cheatsheet/`
- Cada milestone executado tem seu próprio doc `m{N}-{nome}.md`

---

## 🤝 Contribuições

Este é um projeto pessoal documentado publicamente para servir de referência. Issues e sugestões são bem-vindas, mas o foco é registrar este caminho específico, não generalizar.

---

## 📜 Licença

MIT — veja [LICENSE](LICENSE).
