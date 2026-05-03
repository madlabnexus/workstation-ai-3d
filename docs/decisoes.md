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
**Dual boot tradicional**, com **Windows como primário** inicialmente. Linux secundário no Disk 0 (UMIS, que hoje tem EndeavourOS).

> 📝 **Nota v5 (02/05/2026):** O texto anterior dizia "NVMe2"; corrigido para "Disk 0 (UMIS)" após descoberta do mapa real dos discos no M0.1.4.1. UMIS é o NVMe interno menor (1 TB) onde está o EndeavourOS. BIWIN (4 TB) é onde está o Windows + D:.

### Justificativa
- **Adobe + Autodesk são profissionais e diários** — performance bare metal é não-negociável
- **Linux é experimental/aprendizado/IA** no momento — pode evoluir para primário no futuro
- **GPU passthrough no notebook é projeto de 1-3 meses** para iniciantes — não cabe agora
- **Caminho de migração futura existe:** plano é, eventualmente, mover Windows para uma VM no servidor TD350 com GPU passthrough da 4090

### Consequências
- Precisa de pasta NTFS compartilhada para arquivos entre os dois sistemas → **endereçado em D-013 (D: como staging RW)**
- Reboot necessário para alternar — aceitável dado uso ocasional do Linux inicialmente
- Windows não vai ser tocado durante a instalação (só configurações leves: Fast Startup off etc, executadas em M0.1)

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
O notebook atualmente tem dual boot Windows + EndeavourOS gerenciado por **systemd-boot**, funcionando normalmente. Quando instalarmos o Kubuntu (que vai substituir o EndeavourOS no Disk 0/UMIS), o instalador padrão usa GRUB.

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
- Instalador Kubuntu vai escrever GRUB na ESP do Disk 0 (UMIS — onde está o Linux atual)
- Cada disco interno tem sua própria ESP (descoberto no M0.1.4.1): UMIS tem ESP 2GB, BIWIN tem ESP 4GB. Isso é robusto: se GRUB der pau, F12 boota Windows direto via ESP do BIWIN
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
Plano original incluía backup completo da instalação Windows antes de mexer no disco do Linux.

### Decisão
**Não fazer backup do Windows.** O usuário considera o risco aceitável.

### Justificativa
- A instalação do Kubuntu vai mexer **apenas no Disk 0 (UMIS)** — disco do Linux atual
- O Disk 1 (BIWIN — Windows + D:) não será tocado em nenhum momento
- Cada disco tem sua própria ESP (descoberto em M0.1.4.1): GRUB vai escrever na ESP do UMIS, ESP do BIWIN fica intacta. Plano B grátis: se GRUB der pau, F12 boota Windows direto via ESP do BIWIN
- Decisão consciente do usuário

### Consequências
- Aumenta risco se algo realmente catastrófico acontecer
- Mitigação: vamos validar EFI antes de qualquer mudança destrutiva e usar particionamento manual (não automático)

---

## D-007 — Backup do EndeavourOS atual via Macrium Reflect

**Data:** 02/05/2026
**Status:** ✅ Decidido

### Contexto
O Disk 0 (UMIS, 1 TB) tem EndeavourOS funcional. Será substituído pelo Kubuntu, mas é boa prática preservar a instalação caso queira reverter.

### Decisão
**Clonar o Disk 0 (EndeavourOS) com Macrium Reflect**, executado a partir do **Hiren's BootCD PE**, lido via **Ventoy**. Destino do clone = SSD externo dedicado **ainda não plugado** (cenário (a) confirmado pelo usuário).

> 📝 **Nota v5 (02/05/2026):** anteriormente texto dizia "NVMe2" e mencionava "SSK 1 TB externo" como candidato a destino. Corrigido após M0.1.4.1: a fonte é **Disk 0 (UMIS)**, e o destino é um **SSD externo separado, ainda não plugado** (não o SSK que está no Ventoy).

### Justificativa
- Macrium Reflect Free foi descontinuado em jan/2024, **mas continua funcional em quem já tem** ou em ambientes Hiren's PE que vêm com ele incluso
- Hiren's BootCD PE é um Windows PE com ferramentas de manutenção — ambiente adequado para clonagem offline
- Ventoy permite ter múltiplas ISOs no mesmo pendrive (Hiren + Kubuntu) e escolher na hora do boot

### Consequências
- Precisa destino para o clone — SSD externo dedicado a ser plugado quando chegar M0.3
- Tempo de clone depende do tamanho real ocupado no UMIS (não dos 1 TB totais) — coletar `df -h` em M0.2 antes
- SSK USB (Disk 2) é apenas o pendrive Ventoy de boot, **não** o destino do clone

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
- **After Effects, 3ds Max, SolidWorks, Photoshop:** usar Windows do dual boot, sem tentativas em Linux
- **AutoCAD:** Web no Edge para uso casual, Windows para uso pesado
- **Substance Painter, Blender, Unreal 5:** Linux nativo (decisão fácil)
- **Edição de imagem em Linux (substituto parcial de Photoshop):** Krita (pintura/ilustração) e GIMP (edição básica) — cobrem ~80% dos casos não-profissionais

### Justificativa
Tempo gasto tentando rodar After Effects, SolidWorks, 3ds Max ou Photoshop via Wine seria perdido — todos têm mecanismos profundos do Windows que Wine não cobre de forma estável. O Wine patched de janeiro/2026 chega a abrir versões antigas de Photoshop (PS 2021/2025), mas exige build manual e não tem garantia de estabilidade para uso profissional. Para edição pesada de imagem em Linux, Krita e GIMP cobrem o suficiente; o restante vai para o Windows do dual boot. A pesquisa atual confirma que o status não mudou em anos.

**Atualização 02/05/2026 (v2 do projeto):** Photoshop via Wine **cancelado como objetivo** por escolha do usuário. Adobe e Autodesk = Windows. Krita/GIMP cobrem o casual em Linux.

---

## D-013 — D: NTFS como staging RW compartilhado entre Windows e Linux

**Data:** 02/05/2026
**Status:** ✅ Decidido (executar em M0.7)

### Contexto
Durante o M0.1.4.1 foi descoberto que o Disk 1 (BIWIN, 4 TB onde está o Windows) tem **2 partições**: C: (Windows, 1.81 TB) e D: (Data, 2.00 TB). O `D:` foi declarado pelo usuário como **partição de staging compartilhada** entre Windows e o futuro Kubuntu, com requisitos:

- Acessível **read+write** dos dois sistemas
- Sobrevive a reinstalação do Windows (formatar C: não toca D:)
- Hospeda: backup, jogos, dados, documentos, projetos compartilhados

Necessário endereçar formalmente porque nenhum dos 8 ADRs anteriores cobria esse requisito.

### Opções consideradas

1. **Manter D: NTFS, montar no Linux via fstab** — preserva acesso Windows nativo, Linux lê via driver `ntfs3` (kernel 5.15+)
2. **Reformatar D: para ext4** — Linux nativo perfeito, mas Windows perde acesso (ou requer driver third-party `ext2fsd` instável)
3. **Reformatar D: para exFAT** — ambos lêem nativamente, mas sem permissões/journaling, risco maior de corrupção
4. **Criar duas partições novas (uma NTFS, uma ext4) divididas 50/50** — mais complexo, perde flexibilidade, exige reparticionar (intrusivo)

### Decisão
**Manter D: como NTFS, montagem automática RW no Kubuntu via fstab.** Sem restrições de conteúdo (decisão consciente do usuário, cenário previsto não envolve as armadilhas típicas).

### Justificativa
- **Único formato que preserva o requisito** "se reinstalar Windows, D: continua acessível pelo novo Windows sem reformatar"
- **Linux moderno tem driver maduro** (`ntfs3` no kernel oficial desde 5.15)
- **Fast Startup já desligado** no M0.1.2 — neutraliza a armadilha clássica de corrupção (montar NTFS hibernado)
- **Limitações de NTFS via Linux** (sem UID/GID nativo, performance ~70-85% de ext4) **são aceitáveis pro caso de uso declarado**: assets, mídia, jogos — workload de leitura sequencial, onde NTFS via Linux funciona muito bem
- **Sem reformatação** = zero risco de perda de dados durante setup do Kubuntu

### Política de uso (informativa, não bloqueante)

A tabela abaixo é referencial — usuário escolheu sem restrição formal de conteúdo, mas registramos as armadilhas conhecidas:

| Tipo de conteúdo | Recomendado em D:? | Motivo |
|---|---|---|
| Assets de projeto (.blend, .psd, .uasset, .max, .skp) | ✅ | Leitura sequencial, sem dependência de UID/GID |
| Mídia (vídeos, fotos, áudio) | ✅ | Mesmo motivo |
| Bibliotecas Steam, Epic Games | ✅ | Esperado funcionar bem |
| ROMs de emulador, ISOs | ✅ | Sem dependência de permissões UNIX |
| Repos Git em desenvolvimento ativo | ⚠️ | NTFS confunde execute bit, hooks Git, simlinks. Manter em `~/repos` ou `C:\Users\<user>\repos` |
| Ambientes virtuais Python / `node_modules` | ⚠️ | Performance ruim com I/O pequeno em massa |
| Arquivos com permissões UNIX importantes | ⚠️ | NTFS não preserva |

### Consequências

- **M0.7 novo no roadmap** — sub-passo dedicado ao mount automático em fstab, executado **após** M0.6 (instalar Kubuntu) e antes de uso intensivo
- **Configuração fstab planejada:** `defaults,nofail,uid=1000,gid=1000,umask=022,windows_names`
  - `defaults` — set de flags padrão (rw,suid,dev,exec,auto,nouser,async)
  - `nofail` — não trava boot se D: não estiver disponível
  - `uid=1000,gid=1000` — todos arquivos pertencem ao primeiro usuário Linux (UID 1000 = primeiro usuário criado na instalação Kubuntu)
  - `umask=022` — permissões 755 dirs / 644 files
  - `windows_names` — bloqueia nomes Linux que Windows não conseguiria ler depois (caracteres `<>:"|?*`, terminar com ponto)
- **Mount point sugerido:** `/mnt/data` ou `/data` (definir em M0.7)
- **Sem backup formal de D:** D-011 (backup pessoal) está cancelada; backup será ad-hoc se necessário

---

## Próximas decisões pendentes

- **D-009:** Estratégia de dotfiles (chezmoi vs GNU Stow vs solução custom) — decidir antes de M1.5
- **D-010:** DAW para gravação de música (Reaper proprietário vs Ardour open source) — decidir antes de M1.6

---

## Decisões canceladas / removidas

> Para histórico — itens que **estavam** na lista de pendências mas foram retirados para não poluir o roadmap.

- **D-011 — Estratégia de backup pessoal (rsync, restic, BorgBackup)**
  Cancelada em 02/05/2026 a pedido do usuário. Será endereçada *ad-hoc* quando surgir necessidade real (provavelmente via `rsync` para o SSD externo ou pasta NTFS compartilhada). Não é objetivo formal do projeto.

- **D-012 — Modelo de Adobe via Wine (PhialsBasement vs Bottles)**
  Cancelada em 02/05/2026 a pedido do usuário. Coerente com a atualização da D-008: Adobe = Windows. Sem tentativa de rodar Photoshop via Wine.
