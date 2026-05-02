# Registro de Decisões

> Este documento registra **todas as decisões importantes** tomadas neste projeto. Cada uma traz o contexto que levou à escolha, as alternativas avaliadas e a justificativa final.
>
> O objetivo é duplo:
> 1. Permitir que **conversas futuras com IA** retomem o contexto sem perder o raciocínio
> 2. Servir como referência para qualquer pessoa que queira entender o "porquê" de cada parte

**Formato:** ADR (Architecture Decision Record) simplificado — cronológico, mais recente embaixo.

---

## D-001 — Escolha da distribuição Linux: Kubuntu 24.04 LTS + HWE

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
O projeto exige uma distribuição Linux que sirva tanto para o notebook Lenovo P16v Gen 2 (Meteor Lake, hardware muito novo) quanto para a futura VM no servidor TD350 (Xeon E5, hardware maduro). Precisa funcionar bem com NVIDIA, ter snapshots robustos, e dar suporte longo (não vai ser reformatada a cada 6 meses).

### Opções consideradas

1. **Nobara 43 Official-NVIDIA** — Fedora customizado pelo criador do Proton-GE, gaming/creator pré-pronto
2. **Fedora KDE 42** — base Fedora pura, "industrial"
3. **Ubuntu 26.04 LTS** — recém-lançada (23/abr/2026), kernel 7.0
4. **Ubuntu 24.04 LTS + HWE** — LTS estável, HWE traz kernel novo
5. **Bazzite** — gaming-first, imutável (rpm-ostree)

### Decisão
**Kubuntu 24.04 LTS + HWE kernel.**

### Justificativa
- **Notebook + servidor + VM exige uma distro:** Ubuntu LTS é o "padrão de fato" tanto em IA quanto em servidores Proxmox
- **Suporte 5 anos** (até 2029) elimina pressão de upgrade traumático
- **HWE kernel** entrega kernel 6.17+ (e em agosto/2026 chega o 7.0) sem perder o LTS — resolve o suporte a Meteor Lake
- **CUDA Toolkit oficial NVIDIA** distribuído como `.deb` — instalação limpa
- **Ecossistema Debian** alinhado com Proxmox (que é Debian por baixo)
- **Ubuntu 26.04 foi descartada** porque acabou de sair (23/abr) e tem bug conhecido de NVIDIA suspend/resume — exatamente o caso de notebook
- **Nobara foi descartada** porque é "hobby distro, sem garantias" segundo os próprios mantenedores; aceitável em desktop pessoal isolado, problemático em servidor de IA
- **Bazzite foi descartada** porque o sistema imutável (rpm-ostree) limita demais a instalação de software profissional variado

### Consequências
- Vamos precisar instalar manualmente: NVIDIA drivers (PPA oficial), Wine, Proton-GE, codecs proprietários, Steam — todos via script de bootstrap
- Vamos precisar configurar BTRFS + Snapper manualmente (não é o default do instalador Kubuntu)

---

## D-002 — Estratégia notebook: dual boot Windows + Kubuntu, Windows como principal

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
O usuário precisa rodar Photoshop, After Effects, 3ds Max, SolidWorks e AutoCAD profissionalmente. Após análise detalhada (ver [milestones.md](milestones.md) e abaixo), descobriu-se que:

- **Não rodam em Linux:** After Effects, 3ds Max, SolidWorks (projeto comunitário arquivado em fev/2026)
- **Rodam parcialmente em Linux:** Photoshop (Wine patched de jan/2026 funciona com PS 2021/2025, mas exige build manual)
- **Rodam nativamente:** Blender, Unreal Engine 5, Substance Painter (build oficial Adobe Linux)
- **AutoCAD:** versão Web no Edge funciona; desktop full só em Windows

### Opções consideradas

1. **Linux primário + Windows VM com GPU passthrough** — elegante mas extremamente complexo em laptop
2. **Dual boot tradicional Windows + Linux** — simples, performance bare metal de ambos
3. **Windows primário + WSL2** — descarta a experiência Linux completa
4. **Linux only com alternativas open** — exige mudar workflow profissional

### Decisão
**Dual boot tradicional**, com **Windows como primário** inicialmente. Linux secundário no NVMe2 (que hoje tem EndeavourOS).

### Justificativa
- **Adobe + Autodesk são profissionais e diários** — performance bare metal é não-negociável
- **Linux é experimental/aprendizado/IA** no momento — pode evoluir para primário no futuro
- **GPU passthrough no notebook é projeto de 1-3 meses** para iniciantes — não cabe agora
- **Caminho de migração futura existe:** plano é, eventualmente, mover Windows para uma VM no servidor TD350 com GPU passthrough da 4090

### Consequências
- Precisa de pasta NTFS compartilhada para arquivos entre os dois sistemas
- Reboot necessário para alternar — aceitável dado uso ocasional do Linux inicialmente
- Windows não vai ser tocado durante a instalação (só configurações leves: Fast Startup off etc)

---

## D-003 — Estratégia servidor TD350: Proxmox + VM Kubuntu

**Data:** 02/05/2026
**Status:** ✅ Decidido (a executar depois do M0)

### Contexto
Servidor Lenovo TD350 com 2× Xeon E5, 512 GB RAM e RTX 4090 (24 GB) será usado primariamente para workloads de IA local (LLMs, Stable Diffusion, ComfyUI, etc).

### Opções consideradas

1. **Kubuntu bare metal** — simples, tudo dedicado a um SO
2. **Proxmox VE + VM Kubuntu** — flexibilidade, snapshots de VM, possíveis VMs adicionais no futuro
3. **Fedora bare metal** — alternativa sem virtualização

### Decisão
**Proxmox VE 8 como host + VM Kubuntu 24.04 LTS** com 4090 em passthrough.

### Justificativa
- **Snapshots de VM via ZFS do Proxmox** — camada extra de proteção além do BTRFS+Snapper interno
- **Possibilidade futura de VM Windows** com 4090 alternada (para Adobe/Autodesk pesado acessível remotamente)
- **GPU passthrough em desktop com 1 GPU é caso simples** — diferente do laptop com Optimus
- **Mesma distro do notebook** = scripts/dotfiles portáveis

### Consequências
- Quando a 4090 está passada para uma VM, o host Proxmox não tem aceleração gráfica (mas isso é aceitável — Proxmox é gerenciado via web)
- Configuração inicial mais complexa que bare metal — vale pelos benefícios

---

## D-004 — Bootloader: GRUB substitui systemd-boot atual

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
O notebook atualmente tem dual boot Windows + EndeavourOS gerenciado por **systemd-boot**, funcionando normalmente. Quando instalarmos o Kubuntu (que vai substituir o EndeavourOS no NVMe2), o instalador padrão usa GRUB.

### Opções consideradas

1. **Manter systemd-boot, adicionar Kubuntu manualmente** — preserva o existente, mas requer trabalho manual e conhecimento que ainda não temos
2. **Trocar para GRUB durante a instalação Kubuntu** — padrão, instalador faz automaticamente, GRUB detecta Windows via os-prober
3. **Manter ambos coexistindo** — complexo, propenso a erros

### Decisão
**GRUB**, deixando o instalador do Kubuntu configurar normalmente.

### Justificativa
- **GRUB é o padrão e tem mais documentação** — útil dado que o usuário está aprendendo Linux
- **os-prober detecta Windows automaticamente** — dual boot sai funcionando sem trabalho manual
- **systemd-boot é mais "puro" e elegante**, mas exige edição manual de arquivos `.conf` para cada kernel/SO — não é o foco agora
- O EndeavourOS atual será **descartado** (clone de segurança feito antes), então as entries dele do systemd-boot são irrelevantes

### Consequências
- Instalador Kubuntu vai escrever GRUB na EFI System Partition existente (a do Windows)
- Instalador adicionará pasta `/EFI/ubuntu/` na ESP, mantendo `/EFI/Microsoft/` intacta
- Windows continua bootável via menu GRUB

---

## D-005 — Sistema de arquivos: BTRFS com Snapper

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
Um dos requisitos do projeto é "se eu precisar formatar o ROOT não perco nenhum aplicativo nem projeto" e "snapshots". Snapshots são fundamentais para reverter atualizações que quebrem o sistema.

### Opções consideradas

1. **ext4 + Timeshift** — tradicional, mas Timeshift no ext4 usa rsync (lento, espaço duplicado)
2. **BTRFS + Snapper** — snapshots nativos instantâneos, integração com `apt`
3. **BTRFS + Timeshift** — menos integrado que Snapper
4. **ZFS root** — poderoso mas complexo, especialmente em laptop

### Decisão
**BTRFS** com layout de subvolumes + **Snapper** para snapshots automáticos.

### Justificativa
- **Snapshots instantâneos com Copy-on-Write** — sem custo de espaço imediato
- **Subvolumes** permitem segregar `@` (root), `@home` (dados), `@var-log` (logs), etc.
- **Snapper integra com apt** via hooks — snapshot automático antes de cada `apt upgrade`
- **`grub-btrfs`** permite bootar diretamente em snapshots antigos no menu GRUB — recuperação trivial

### Consequências
- Particionamento manual no instalador (Kubuntu não cria subvolumes BTRFS por padrão)
- Configuração de Snapper pós-instalação (ferramenta não vem ativa)
- Espaço gerenciado: snapshots crescem com mudanças, precisa política de retenção

---

## D-006 — Não fazer backup do Windows antes da instalação

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
Plano original incluía backup completo da instalação Windows no NVMe1 antes de mexer no NVMe2 (onde está o EndeavourOS, que será substituído por Kubuntu).

### Decisão
**Não fazer backup do Windows.** O usuário considera o risco aceitável.

### Justificativa
- A instalação do Kubuntu vai mexer **apenas no NVMe2**
- A EFI partition compartilhada terá uma pasta nova adicionada (`/EFI/ubuntu/`), sem mexer na pasta `/EFI/Microsoft/`
- Se algo der errado e o GRUB sobrescrever o boot Windows, é recuperável via Windows recovery
- Decisão consciente do usuário

### Consequências
- Aumenta risco se algo realmente catastrófico acontecer
- Mitigação: vamos validar EFI antes de qualquer mudança destrutiva e usar particionamento manual (não automático)

---

## D-007 — Backup do EndeavourOS atual via Macrium Reflect

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
O NVMe2 tem EndeavourOS funcional. Será substituído pelo Kubuntu, mas é boa prática preservar a instalação caso queira reverter.

### Decisão
**Clonar o NVMe2 (EndeavourOS) com Macrium Reflect**, executado a partir do **Hiren's BootCD PE**, lido via **Ventoy**.

### Justificativa
- Macrium Reflect Free foi descontinuado em jan/2024, **mas continua funcional em quem já tem** ou em ambientes Hiren's PE que vêm com ele incluso
- Hiren's BootCD PE é um Windows PE com ferramentas de manutenção — ambiente adequado para clonagem offline
- Ventoy permite ter múltiplas ISOs no mesmo pendrive (Hiren + Kubuntu) e escolher na hora do boot

### Consequências
- Precisa destino para o clone (SSK 1TB externo é candidato natural)
- Tempo de clone depende do tamanho real ocupado no NVMe2 (não dos 4 TB totais)

---

## D-008 — Adobe e Autodesk ficam só no Windows

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
A pesquisa atual (maio/2026) sobre compatibilidade de softwares profissionais em Linux mostrou:

| Software | Linux nativo | Wine viável | Veredito |
|---|---|---|---|
| Photoshop | ❌ | 🟡 (Wine patched jan/2026 com PS 2021) | Linux experimental |
| After Effects | ❌ | ❌ (Wine não suporta) | **Só Windows** |
| 3ds Max | ❌ | ❌ (Wine quebra no instalador) | **Só Windows** |
| SolidWorks | ❌ | ❌ (projeto comunitário Wine arquivado em fev/2026) | **Só Windows** |
| AutoCAD | ❌ | 🟡 (versões antigas) | Web no Edge ou Windows |
| Substance Painter | ✅ (via portal Adobe Linux) | — | Linux nativo |
| Blender | ✅ | — | Linux nativo |
| Unreal Engine 5 | ✅ | — | Linux nativo |

### Decisão
- **After Effects, 3ds Max, SolidWorks:** usar Windows do dual boot, sem tentativas em Linux
- **Photoshop:** experimentar Wine patched em milestone futuro (não-crítico)
- **AutoCAD:** Web no Edge para uso casual, Windows para uso pesado
- **Substance Painter, Blender, Unreal 5:** Linux nativo (decisão fácil)

### Justificativa
Tempo gasto tentando rodar After Effects ou SolidWorks via Wine seria perdido. Ambos têm mecanismos profundos do Windows que Wine não cobre. A pesquisa ATUAL confirma que o status não mudou em anos.

---

## Próximas decisões pendentes

- **D-009:** Estratégia de dotfiles (chezmoi vs GNU Stow vs solução custom) — decidir antes de M1
- **D-010:** DAW para gravação de música (Reaper proprietário vs Ardour open source) — decidir em M1
- **D-011:** Estratégia de backup pessoal (rsync, restic, BorgBackup) — decidir em M1
- **D-012:** Modelo de Adobe via Wine (PhialsBasement patched vs Bottles vanilla) — decidir em M-experimental futuro
