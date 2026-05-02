# M0.1 — Preparando o Windows para o dual boot

> **Milestone:** M0 (Setup base)
> **Sub-passo:** 1 de 6
> **Tempo estimado:** 30-60 minutos
> **Pré-requisitos:** Nenhum
> **Risco:** Baixo (mexemos só em configurações reversíveis do Windows)

---

## 🎯 O que vamos fazer aqui

Antes de instalar o Linux, precisamos **configurar o Windows** para que ele se comporte bem ao dividir o computador com o Linux. Sem essas configurações, problemas como **corrupção de arquivos compartilhados** ou **menu de boot que não aparece** podem acontecer.

São 5 ajustes principais:

1. **Desabilitar Fast Startup** — para o Windows fechar de verdade ao desligar
2. **Desabilitar Hibernação** — pelo mesmo motivo
3. **Verificar BitLocker** — se a criptografia estiver ativa, pode pedir senha de recuperação ao mudar particionamento
4. **Atualizar BIOS** (se houver versão nova) — drivers e correções de hardware
5. **Verificar configuração de gráficos no BIOS** — entender se o notebook é "MUXed" (importante pro futuro)

> ⚠️ **Importante:** todas essas mudanças são **reversíveis**. Se algo der errado, você consegue desfazer.

---

## 🧠 Por que precisamos fazer isso?

### Fast Startup e Hibernação

Quando você "desliga" o Windows com Fast Startup ativo, ele **não desliga de verdade** — ele hiberna parte do sistema (incluindo o estado dos drives). Se você bootar no Linux nesse estado e tentar **escrever** em uma partição Windows (ex: salvar um arquivo na pasta NTFS compartilhada), você corrompe os dados do Windows.

Mesmo se nunca pretendermos escrever do Linux no Windows, é mais seguro desabilitar — porque um descuido pode acontecer.

### BitLocker

Se está ativo, o Windows pode pedir a **chave de recuperação** ao detectar mudança no boot loader (que vai acontecer quando instalarmos GRUB). Sem a chave, o Windows não abre. **Suspender** o BitLocker antes evita isso.

### BIOS atualizado

Versão atual no log: `N44ET35W (1.18)`. Se houver versão mais nova, geralmente trazem correções de:
- Compatibilidade com kernels Linux novos
- Bugs de Optimus / NVIDIA
- IOMMU groupings (importa para milestones futuros)

### Configuração de gráficos no BIOS

Tem implicação direta nos milestones M4-M5 (GPU passthrough). Vamos só **olhar** agora, não mudar.

---

## 📋 Passo a passo

### Passo 1 — Desabilitar Fast Startup

#### O que é Fast Startup?
Recurso do Windows que acelera o boot **hibernando parcialmente** ao desligar. Causa problemas em dual boot.

#### Como desabilitar (caminho 1: interface gráfica)

1. **Abra o Painel de Controle**
   - Aperte `Windows + R` (abre a caixa "Executar")
   - Digite: `control panel` → Enter

2. **Vá para "Opções de Energia"**
   - Se a visualização estiver em "Categoria", clique em "Hardware e Sons" → "Opções de Energia"
   - Se estiver em "Ícones grandes/pequenos", clique direto em "Opções de Energia"

3. **Clique em "Escolher a função dos botões de energia"** (na lateral esquerda)

4. **Clique em "Alterar configurações não disponíveis no momento"** (no topo)
   - Pode pedir senha de administrador

5. **Role para baixo até "Configurações de desligamento"**
   - **Desmarque** a opção "Ligar inicialização rápida (recomendado)"

6. **Clique em "Salvar alterações"**

#### Caminho 2: PowerShell (mais rápido, requer admin)

Abra o **PowerShell como administrador**:
- Tecla Windows → digite `powershell`
- Clique direito em "Windows PowerShell" → "Executar como administrador"

Cole e execute:

```powershell
powercfg /h off
```

> 💡 **O que esse comando faz:** desabilita a hibernação do Windows. Como o Fast Startup **depende** da hibernação, desabilitar a hibernação automaticamente desabilita o Fast Startup também. Mata dois coelhos com uma cajadada.

#### Como verificar se funcionou

No PowerShell, rode:

```powershell
powercfg /a
```

**Saída esperada (parcial):**
```
Os seguintes estados de suspensão estão disponíveis neste sistema:
    Espera (S3)

Os seguintes estados de suspensão não estão disponíveis neste sistema:
    Hibernar
        Hibernação não foi habilitada.
    Inicialização Rápida
        Hibernação não foi habilitada.
```

✅ Se aparecer "Hibernação não foi habilitada" e "Inicialização Rápida" listados como **não disponíveis**, deu certo.

---

### Passo 2 — Verificar status do BitLocker

#### O que estamos verificando

Se a criptografia BitLocker está **ativa** em alguma partição. Se estiver, precisamos **suspender** antes de mexer no particionamento.

#### Como verificar

No **PowerShell como administrador**, rode:

```powershell
manage-bde -status
```

**Saídas possíveis:**

**Caso A — BitLocker desativado (cenário ideal):**
```
Volume C: []
[OS Volume]

    Tamanho:                 XXX,XX GB
    Versão do BitLocker:     2.0
    Status da Conversão:     Totalmente Descriptografado
    Porcentagem Criptografada: 0,0%
    Método de Criptografia:  Nenhum
    Status da Proteção:      Proteção Desativada
    Status do Bloqueio:      Desbloqueado
```
✅ Não precisa fazer nada. BitLocker está off.

**Caso B — BitLocker ativo:**
```
    Status da Conversão:     Totalmente Criptografado
    Porcentagem Criptografada: 100,0%
    Status da Proteção:      Proteção Ativada
```
⚠️ **Precisamos suspender** antes de instalar Linux. Veja o sub-passo abaixo.

#### Sub-passo 2.1 — Suspender BitLocker (se estiver ativo)

> **Importante:** "Suspender" é diferente de "Descriptografar". Suspender é instantâneo e não bagunça os dados — apenas desliga a verificação até o próximo reboot. Você reativa quando quiser.

Comando:
```powershell
manage-bde -protectors -disable C:
```

Verificar:
```powershell
manage-bde -status
```

Procure por:
```
Status da Proteção:      Proteção Suspensa
```

**Antes de tudo isso, GUARDE A CHAVE DE RECUPERAÇÃO:**
```powershell
manage-bde -protectors -get C:
```

Anote a "Chave de Recuperação Numérica" em local seguro. Salvá-la em conta Microsoft também é recomendado.

---

### Passo 3 — Atualizar a BIOS (se houver versão nova)

#### Estado atual
- BIOS: `N44ET35W (1.18)` — registrado no log do HWiNFO
- Modelo: ThinkPad P16v Gen 2

#### Como verificar/atualizar (recomendado: Lenovo Vantage)

1. **Instalar o Lenovo Vantage** (se ainda não tem)
   - Microsoft Store → buscar "Lenovo Vantage" → Instalar

2. **Abrir o Lenovo Vantage**
   - Menu Iniciar → Lenovo Vantage

3. **Ir para "Atualizações de sistema"**
   - Geralmente em "Suporte" ou "Hardware"

4. **Verificar atualizações disponíveis**
   - Se houver atualização de **BIOS / UEFI**, instale ela primeiro
   - Pode pedir reiniciar — **conecte o carregador** antes (NUNCA atualizar BIOS na bateria)

#### Caminho alternativo: site Lenovo
Se Lenovo Vantage der problema:
- Acesse https://pcsupport.lenovo.com → busque "ThinkPad P16v Gen 2"
- Vá em Drivers & Software → BIOS/UEFI
- Baixe o instalador e execute

#### O que NÃO fazer
- ❌ Não desligar o computador durante atualização de BIOS
- ❌ Não atualizar BIOS apenas com bateria — sempre conectado à tomada
- ❌ Não pular outras atualizações importantes (firmware Thunderbolt, ME)

---

### Passo 4 — Olhar (não mudar) configuração de gráficos no BIOS

#### Por que estamos olhando agora
Para confirmar se o notebook é **MUXed** (tem caminho dedicado da NVIDIA para alguma saída) ou **MUXless**. Isso afeta milestones futuros (M4-M5 de GPU passthrough). Por enquanto **só vamos olhar e anotar**, sem mudar nada.

#### Como entrar na BIOS

1. **Reiniciar o notebook**
2. Quando aparecer o **logo Lenovo**, aperte `F1` repetidamente (várias vezes seguidas)
   - Se passar do logo, deixe entrar no Windows e tente de novo
3. Você entra na **BIOS / UEFI Setup**

#### O que procurar

Navegue (use setas e Enter, mouse não funciona) até:

**Config → Display**

Procure por opções como:
- `Graphics Device` ou `GPU Mode`
- `Hybrid Graphics` (Intel + NVIDIA juntos)
- `Discrete Graphics` (só NVIDIA)
- `Integrated Graphics` (só Intel — provavelmente não disponível em P-series workstation)

#### O que anotar (não mudar)

Crie uma anotação no celular ou em um caderno:

```
Modelo: ThinkPad P16v Gen 2
BIOS atual: <versão que está agora>
Configuração gráfica atual: <Hybrid Graphics ou Discrete Graphics>
Opções disponíveis: <listar todas que aparecem no menu>
Outras opções relevantes vistas:
- Secure Boot: <Disabled/Enabled>
- Intel VT-x: <Enabled/Disabled>
- Intel VT-d: <Enabled/Disabled>
- TPM: <Enabled/Disabled>
```

#### Sair da BIOS sem salvar

Aperte `F10` → ele pergunta se quer salvar mudanças. Como **não mudamos nada**, escolha "No" ou "Discard Changes and Exit".

Ou: aperte `Esc` várias vezes até voltar ao menu principal → "Exit" → "Discard Changes and Exit".

---

## ✅ Checklist de conclusão do M0.1

Depois de completar tudo:

- [ ] Fast Startup desabilitado (`powercfg /a` mostra "Hibernação não habilitada")
- [ ] BitLocker verificado (e suspendido se estava ativo)
- [ ] BIOS atualizada (se havia versão mais nova)
- [ ] Configuração gráfica anotada (Hybrid ou Discrete)
- [ ] Secure Boot, VT-x, VT-d e TPM anotados

---

## 📤 O que reportar para o próximo passo (M0.2)

Quando terminar, me retorne com:

1. **Resultado do `powercfg /a`** após desabilitar Fast Startup (pode ser captura de tela ou texto)
2. **Resultado do `manage-bde -status`** (estado do BitLocker)
3. **Versão da BIOS** após atualização
4. **Anotação da BIOS** com:
   - Configuração gráfica atual e opções
   - Estado de Secure Boot, VT-x, VT-d, TPM

Com esses dados, **eu construo o M0.2** (coleta de estado do EndeavourOS) já adaptado ao seu hardware exato.

---

## 🚨 Se algo deu errado

### "powercfg não é reconhecido"
Você não está no PowerShell ou não está como admin. Reabra PowerShell **clicando direito → Executar como administrador**.

### "Acesso negado" ao desabilitar Fast Startup
Você não tem permissão de admin. Confirme que está rodando como administrador.

### Lenovo Vantage não detecta atualizações
Pode estar com cache. Feche, reabra. Se persistir, use o caminho do site Lenovo.

### BitLocker pede chave que não tenho
- Verifique sua conta Microsoft em https://account.microsoft.com/devices/recoverykey
- Se não estiver lá, pode estar em backup que você fez antes (pendrive, e-mail)
- Se realmente perdeu, **não suspenda o BitLocker** sem ter a chave — você pode perder acesso aos dados

### BIOS não abre com F1
Tente:
- F2 (alternativo Lenovo)
- Manter F1 pressionado desde antes do logo
- Restart segurando Shift no Windows → Solução de Problemas → Configurações de Firmware UEFI

---

## 📚 Termos novos vistos aqui

- **Fast Startup** — ver [glossário](glossario.md#fast-startup-windows)
- **BitLocker** — ver [glossário](glossario.md#bitlocker)
- **BIOS / UEFI** — ver [glossário](glossario.md#efi--uefi)
- **TPM** — ver [glossário](glossario.md#tpm-trusted-platform-module)
- **Secure Boot** — segurança de boot UEFI; vamos manter Disabled (nosso log mostra que já está)
- **VT-x / VT-d** — virtualização Intel; vamos precisar habilitados nos milestones M2+

---

**➡️ Próximo passo:** [M0.2 — Coleta do estado atual no EndeavourOS](m0-coleta-estado.md) (será criado quando você concluir M0.1)
