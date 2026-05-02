# STATE — Onde estamos agora

> 🧭 **Este é o GPS do projeto.** Ponto de entrada para qualquer chat novo (com IA ou colaborador humano). Sempre atualizado ao final de cada sessão de trabalho.
>
> **📅 Última atualização:** 02/05/2026 (TODOs explícitos do usuário registrados)
> **🔢 Versão do estado:** 2
> **🎯 Milestone atual:** M0 — Setup base do dual boot
> **📍 Sub-passo atual:** Pronto para iniciar M0.1

---

## 📌 Resumo executivo (1 parágrafo)

Estamos no início da fase de execução do projeto. Toda a arquitetura foi decidida, todas as 8 decisões formais (D-001 a D-008) foram registradas, a estrutura do repositório foi criada, o repositório GitHub público está no ar (https://github.com/madlabnexus/workstation-ai-3d), Git e gh CLI foram instalados e autenticados no Windows. **Nada ainda foi tocado no hardware** — o dual boot Windows + EndeavourOS atual continua funcionando normalmente. **Próxima ação:** começar M0.1 — tweaks no Windows (desabilitar Fast Startup, verificar BitLocker, atualizar BIOS, anotar config gráficos).

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
- [x] Configuração global do Git aplicada:
  - `user.name=madlabnexus`
  - `user.email=182696892+madlabnexus@users.noreply.github.com`
  - `init.defaultBranch=main`
  - `pull.rebase=false`
- [x] Privacidade GitHub ativada (`Keep my email private` + `Block command line pushes`)
- [x] PAT criado (note: `gh-cli-thinkpad-p16v`, scopes: `repo`, `workflow`, `read:org`, `read:user`, `gist`)
- [x] `gh auth login` concluído via PAT — `Logged in as madlabnexus`
- [x] Repo `workstation-ai-3d` (raiz local: `C:\Users\gcarneiro\repos\workstation-ai-3d`)
- [x] Primeiro commit: hash `6544717`, 8 arquivos, 1.421 insertions
- [x] Push para GitHub: https://github.com/madlabnexus/workstation-ai-3d (público)

### Fase de início da execução (em andamento)

- [x] Sessão de retomada via chat novo: STATE/decisoes/milestones lidos e confirmados
- [x] Distro Kubuntu 24.04 LTS reafirmada (D-001 mantida — decisão Nobara não reaberta)
- [x] TODOs explícitos do usuário registrados nesta versão (Edge, emuladores, descompactação, Git Linux, DAW, dotfiles)
- [x] `milestones.md` atualizado para incluir Edge (M1.1), descompactação+Git (M0.5) e nova seção M1.9 (emuladores)

---

## 🎯 Próximo passo concreto

**M0.1 — Tweaks no Windows** (documento: `docs/m0-windows-tweaks.md`)

Sequência:
1. Abrir PowerShell como administrador
2. Rodar `powercfg /h off` (desabilita Fast Startup + Hibernação de uma vez)
3. Validar com `powercfg /a`
4. Rodar `manage-bde -status` (verificar BitLocker)
5. Se BitLocker ativo: salvar chave de recuperação + suspender com `manage-bde -protectors -disable C:`
6. Abrir Lenovo Vantage → verificar atualização de BIOS (atual: N44ET35W v1.18)
7. Reboot → entrar na BIOS (F1) → anotar (não mudar): config gráfica, Secure Boot, VT-x, VT-d, TPM
8. Reportar tudo de volta para iniciar M0.2 (coleta de estado do EndeavourOS)

---

## 🔖 TODOs explícitos do usuário (a slotar em milestones)

Requisitos declarados pelo usuário na abertura do projeto que precisavam ser registrados explicitamente para não esquecer durante a execução. Já foram refletidos em `milestones.md`:

- [ ] **Microsoft Edge para Office 365** → entra em **M1.1** (apps essenciais). Build oficial Edge for Linux disponível como `.deb` direto da Microsoft.
- [ ] **Emuladores (retro, DOS, Windows antigo)** → **M1.9** (nova seção): RetroArch (consoles retro), DOSBox-Staging (DOS), 86Box ou PCem (PC antigo até Pentium III), QEMU desktop para Win98/XP isolados.
- [ ] **DAW para gravação de música** → **D-010 pendente** (Reaper proprietário vs Ardour open source). Decidir antes de M1.6.
- [ ] **Ferramentas de descompactação** (zip, rar, 7z, tar, gz, zstd, xz) → entra em **M0.5** (pós-instalação) via `p7zip-full`, `unrar`, `unzip`, `zstd`, `xz-utils`.
- [ ] **Git no Linux** → entra em **M0.5** (no Windows já está, falta replicar).
- [ ] **Dotfiles auto-reinstaláveis** → **D-009 pendente** (chezmoi vs GNU Stow vs custom). Decidir antes de M1.5.
- [ ] **Bootstrap script idempotente** → já previsto em **M1.4**. Vai consumir os TODOs acima como "lista mestra de pacotes".

---

## 🧠 Contexto crítico para retomar

### Sobre o estado real da máquina

- **NVMe1** (UMIS 1 TB) → Windows 11 Pro Workstation 25H2, **funcionando**, **NÃO tocar**
- **NVMe2** (BIWIN 4 TB) → EndeavourOS, **funcionando**, **será substituído** por Kubuntu
- **Dual boot atual** → systemd-boot, **funcionando perfeitamente**, ambos os SOs bootam normal
- **EFI partition** → ainda **não investigada**, será feito no M0.2 via boot do EndeavourOS

### Decisões que mudaram durante o planejamento (importante para não confundir)

- **Distro:** mudei de Nobara → Fedora → Kubuntu 24.04 LTS (decisão final, reafirmada no chat de 02/05)
- **Backup do Windows:** plano original tinha; decisão final é **NÃO fazer**
- **Mexer no Windows:** plano final permite tweaks (Fast Startup off etc) **mas sem backup**
- **Bootloader:** systemd-boot atual será substituído por **GRUB** (decisão por simplicidade)
- **Hiren's BootCD:** já está no Ventoy junto com Kubuntu LTS — **pendrive pronto**

### Premissas operacionais

- **Modo de trabalho:** um comando por vez, esperar output, validar, próximo passo
- **Documentação:** "passo a passo para criança" — sem assumir conhecimento técnico
- **Cada comando rodado real** vira entrada no `cheatsheet/README.md`
- **Cada decisão importante** vira ADR em `docs/decisoes.md` (D-XXX)
- **Identidade pública:** `madlabnexus` em qualquer arquivo público; **nome real do usuário NÃO entra em arquivos públicos**

### Coisas que ainda NÃO temos certeza (descobrir conforme avança)

- Layout exato da EFI System Partition (NVMe1 ou NVMe2? compartilhada?)
- Versão atual do EndeavourOS instalado
- O que existe de relevante no EndeavourOS atual (vai ser apagado — clone é só backup)
- Se há atualização de BIOS Lenovo mais nova que v1.18
- Status do BitLocker no Windows (ativo ou não)
- Se o P16v Gen 2 é MUXed ou MUXless (importante para milestones M4-M5)

---

## 📚 Arquivos do projeto (mapa)

| Arquivo | Função | Quando consultar |
|---|---|---|
| `STATE.md` | **Este arquivo.** GPS do projeto | Sempre, primeiro |
| `README.md` | Visão geral do projeto | Para ter mapa mental |
| `docs/decisoes.md` | 8 decisões formais com justificativas | Antes de questionar uma escolha |
| `docs/milestones.md` | Roadmap M0-M5 e sub-passos | Para entender direção |
| `docs/glossario.md` | ~150 termos técnicos | Quando aparecer termo desconhecido |
| `docs/m0-windows-tweaks.md` | Sub-passo M0.1 detalhado | Durante execução do M0.1 |
| `cheatsheet/README.md` | Comandos validados | Quando esquecer comando que já rodamos |
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

Sem isso, a IA pode acabar refazendo decisões já tomadas ou pulando contexto importante.

---

## 🔄 Manutenção deste arquivo

**Quando atualizar:**

- ✅ Ao concluir um sub-passo de milestone (`M0.1` → `M0.2`)
- ✅ Ao tomar decisão nova (D-009, D-010...)
- ✅ Ao descobrir gotcha importante durante execução
- ✅ Ao mudar de plano (raro, mas acontece — vide histórico de mudanças de distro)

**Como atualizar:**

1. Editar este arquivo (`STATE.md`) localmente
2. Atualizar campo "Última atualização" com a data atual
3. Incrementar "Versão do estado"
4. Adicionar entrada em "O que foi concluído"
5. Atualizar "Próximo passo concreto"
6. Revisar "Contexto crítico" (remover o que não é mais relevante, adicionar novidades)
7. `git add STATE.md && git commit -m "STATE: <resumo da mudança>" && git push`
8. **Re-upload** no Project Knowledge do Claude (ou conectar GitHub MCP — ver abaixo)

**Princípio:** este arquivo deve **caber em uma tela** ao ler em terminal. Se ficou grande demais, mover detalhes históricos para `decisoes.md` ou `milestones.md`.
