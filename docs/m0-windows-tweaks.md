# M0.1 — Preparando o Windows para o dual boot ✅ CONCLUÍDO

> **Status:** ✅ Concluído em 02/05/2026
> **Milestone:** M0 (Setup base)
> **Sub-passo:** 1 de 7 (M0.1 a M0.7)
> **Tempo real gasto:** ~3 horas (planejado: 30-60min — descobertas emergentes expandiram o escopo)
> **Pré-requisitos:** Nenhum
> **Risco realizado:** Baixo — todas as ações foram bem-sucedidas, sem necessidade de recovery

---

## 🎯 O que foi feito aqui

Antes de instalar o Linux, o Windows foi **configurado** para conviver bem com Linux no dual boot, e o estado do hardware foi totalmente mapeado pra eliminar suposições do plano original.

**Sub-passos executados (todos validados com output real):**

1. ✅ **M0.1.1** — PowerShell aberto como administrador
2. ✅ **M0.1.2** — Fast Startup + Hibernação desabilitados (`powercfg /h off`)
3. ✅ **M0.1.3** — Validação dos estados de suspensão (`powercfg /a`)
4. ✅ **M0.1.4** — Status do BitLocker confirmado (`manage-bde -status`) — desativado em todos os volumes
5. ✅ **M0.1.4.1** (emergente) — Mapa real dos discos físicos descoberto (`Get-Partition` + `Get-Disk`) — corrigiu inversão NVMe1/NVMe2 do STATE v4
6. ✅ **M0.1.5.1** — Versão BIOS atual confirmada (`Get-CimInstance Win32_BIOS`)
7. ✅ **M0.1.5.2** — Versão vBIOS NVIDIA atual obtida (`nvidia-smi`)
8. ✅ **M0.1.5.3.A** — Flash da BIOS principal (`N44ET35W v1.18` → `N44ET37W v1.20`)
9. ✅ **M0.1.5.3.B** (emergente) — Backup da vBIOS NVIDIA via `nvflash64` antes do flash dela
10. ✅ **M0.1.5.3.C** — Flash da vBIOS NVIDIA (`.01 → .1c`)
11. ✅ **M0.1.6** — Configuração gráfica anotada — **MUXless confirmado**
12. ✅ **M0.1.6.1** (emergente) — `BootMode = Diagnostics` corrigido pra `Auto`/`UEFI`
13. ✅ **M0.1.7** — Settings de virtualização e segurança coletados via WMI

> ⚠️ **Importante:** todas essas mudanças são **reversíveis**. Esse documento serve de referência tanto pra reinstalação futura quanto pra entender o que cada passo fez.

---

## 🧠 Por que cada coisa foi feita

### Fast Startup e Hibernação

Quando você "desliga" o Windows com Fast Startup ativo, ele **não desliga de verdade** — hiberna parte do sistema (incluindo o estado dos drives). Se você bootar no Linux nesse estado e tentar **escrever** em uma partição Windows (ex: salvar arquivo na pasta NTFS compartilhada), você corrompe os dados.

Como D-013 estabelece D: NTFS como **staging RW compartilhado**, desabilitar Fast Startup é pré-requisito não-negociável.

Bônus: liberou 24-32 GB de espaço em disco no C: (era o tamanho do `hiberfil.sys`).

### BitLocker

Se ativo, Windows pode pedir **chave de recuperação** ao detectar mudança no boot loader (que vai acontecer quando GRUB substituir systemd-boot no M0.4). Sem a chave, Windows não abre. **Suspender** o BitLocker antes evita isso.

Resultado da verificação (M0.1.4): **BitLocker OFF em todos os volumes**, incluindo C:, D:, E:, F:, e VTOYEFI. Não foi necessário suspender nada.

### BIOS atualizada

Versão original no `p16v.LOG`: `N44ET35W (1.18)`, release 27/08/2025. 8 meses sem atualização.

Pacote Lenovo `n44uj11w` continha:
- UEFI BIOS v1.20
- ECP (Embedded Controller Program) v1.10

Após flash bem-sucedido (com memory training automático ~2 min), nova versão: **`N44ET37W v1.20`**, release 10/12/2025.

Razões pra atualizar BIOS antes de instalar Linux:
- Compatibilidade com kernels Linux novos (microcodes Intel pra Meteor Lake)
- Bugs de Optimus / NVIDIA hybrid
- IOMMU groupings (importa para milestones M4-M5)

### vBIOS NVIDIA

vBIOS original: `95.06.31.40.01`, Build 30/08/2023, Modificação 19/02/2024 (~2 anos de idade).

Pacote Lenovo `n44vw02w_v3` (publicado fev/2026) continha vBIOS atualizada. Após flash: **`95.06.31.40.1c`** (mesma família/major/sub-vendor/ROM, apenas patch level subiu `.01 → .1c` em hex).

**Cuidado especial tomado:** backup da vBIOS antes do flash, com `nvflash64.exe --save`. Backup salvo em `C:\Users\<user>\bkp-bios\vbios-p16v_bkp-95.06.31.40.01.rom` (2 MB exatos).

### Configuração de gráficos no BIOS

Implicação direta nos milestones M4-M5 (GPU passthrough).

**Achado importante:** P16v Gen 2 é **MUXless** — opção `Discrete Graphics` **não existe** no BIOS Setup. Anteriormente o glossário e milestones assumiam "MUXed provavelmente"; agora é fato. Significa:

- dGPU NVIDIA **sempre passa pelo iGPU** Intel Arc Pro pra mostrar imagem
- No Linux: usar PRIME render offload ou envycontrol pra alternar dGPU em apps específicos
- Em M4 (passthrough): saída de vídeo da VM **precisa** passar por Looking Glass / Sunshine / rede, não tem display direto da dGPU

---

## 📋 Comandos executados, com outputs reais

### Passo 1 — Validação de admin

#### Como abrir PowerShell admin

**Caminho recomendado (Win+X):**
1. Aperte `Windows + X`
2. Escolha **"Terminal (Admin)"**
3. UAC → Sim

**Validar que está como admin:**

```powershell
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

**Output validado:**
```
True
```

`True` = está como admin. Se `False`, fechar e reabrir como administrador.

---

### Passo 2 — Desabilitar Fast Startup + Hibernação

```powershell
powercfg /h off
```

**Output validado:** silêncio (prompt volta direto). Esse silêncio É sucesso — `powercfg` segue padrão Unix-like ("no news is good news").

**O que esse comando faz:**
- Desabilita hibernação do Windows (`/h` = `/hibernate`)
- Como Fast Startup é uma forma de hibernação parcial, ele cai junto
- Apaga `C:\hiberfil.sys` (~24-32 GB liberados — o tamanho do RAM do laptop)
- Remove "Hibernar" do menu de energia

**Reversível com:** `powercfg /h on`

---

### Passo 3 — Validar com `powercfg /a`

```powershell
powercfg /a
```

**Output validado (em inglês porque o Windows desse hardware está em EN):**

```
The following sleep states are available on this system:
    Standby (S0 Low Power Idle) Network Connected

The following sleep states are not available on this system:
    Standby (S1)
        The system firmware does not support this standby state.
        This standby state is disabled when S0 low power idle is supported.
    Standby (S2)
        ...
    Standby (S3)
        ...
    Hibernate
        Hibernation has not been enabled.
    Hybrid Sleep
        Standby (S3) is not available.
        Hibernation is not available.
        The hypervisor does not support this standby state.
    Fast Startup
        Hibernation is not available.
```

**Leitura:**
- ✅ `Hibernate` em "not available" com razão `"Hibernation has not been enabled"` — comando do Passo 2 funcionou
- ✅ `Fast Startup` em "not available" com razão `"Hibernation is not available"` — caiu junto, como esperado
- ✅ `Standby (S0 Low Power Idle) Network Connected` — Modern Standby ativo, esperado em Meteor Lake
- ❌ S1/S2/S3 não suportados pelo firmware — esperado em hardware moderno (substituídos pelo S0ix)
- 🔖 **Bandeira pra futuro:** `Hybrid Sleep` menciona `"The hypervisor does not support this standby state"` — confirma que **Hyper-V/VBS está ativo** no Windows. Vai importar nos milestones M2+ (servidor) e M4-M5 (passthrough)

---

### Passo 4 — Verificar status do BitLocker

```powershell
manage-bde -status
```

**Output validado (resumido — listou 5 volumes):**

```
BitLocker Drive Encryption: Configuration Tool version 10.0.26100

Volume C: [Windows 11]
[OS Volume]
    Size:                 1807.62 GB
    BitLocker Version:    None
    Conversion Status:    Fully Decrypted
    Percentage Encrypted: 0.0%
    Encryption Method:    None
    Protection Status:    Protection Off
    Lock Status:          Unlocked
    Key Protectors:       None Found

Volume D: [Data]
    Size:                 2003.09 GB
    [...] Protection Off, Key Protectors: None Found

Volume E: [Ventoy]      453.84 GB     Protection Off
Volume \\?\Volume{...}\ [VTOYEFI]  0.03 GB    Protection Off
Volume F: [bkpunvsal]   499.98 GB     Protection Off
```

**Leitura:** todos os volumes em `Protection Off`, `Conversion Status: Fully Decrypted`. **Nenhuma ação necessária** (não precisamos suspender BitLocker antes do GRUB no M0.4).

---

### Passo 5 — Mapa real dos discos (sub-passo emergente M0.1.4.1)

Antes do M0.1, o STATE v4 dizia "NVMe1 = UMIS = Windows, NVMe2 = BIWIN = Linux". O comando do BitLocker mostrou volumes que não estavam mapeados (D:, F:), então fomos a fundo:

```powershell
Get-Partition | Sort-Object DiskNumber, PartitionNumber | Format-Table -AutoSize DiskNumber, PartitionNumber, DriveLetter, Size, Type
```

**Output validado:**
```
DiskNumber PartitionNumber DriveLetter          Size Type
---------- --------------- -----------          ---- ----
         0               1               2147483648 System
         0               2             985703174144 Unknown
         0               3              36356754432 Unknown
         1               1                 16777216 Reserved
         1               2           C 1940914176000 Basic
         1               3               4293918720 System
         1               4                770703360 Recovery
         1               5           D 2150806585344 Basic
         2               1           E  487304007680 Basic
         2               2                 33554432 Basic
         2               3           F  536869863424 Basic
```

```powershell
Get-Disk | Sort-Object Number | Format-Table -AutoSize Number, FriendlyName, Size, BusType, OperationalStatus
```

**Output validado:**
```
Number FriendlyName                   Size BusType OperationalStatus
------ ------------                   ---- ------- -----------------
     0 UMIS RPETJ1T24MKP2QDQ 1024209543168 NVMe    Online
     1 BIWIN NV7400 4TB      4096805658624 NVMe    Online
     2 SSK Port able SSD 1TB 1024209543168 USB     Online
```

**Leitura — STATE v4 estava INVERTIDO:**

| Disco | Modelo | Tamanho | Bus | Conteúdo (correto v5) |
|---|---|---|---|---|
| **Disk 0** | UMIS RPETJ1T24MKP2QDQ | 1.02 TB | NVMe interno | **🐧 EndeavourOS** (será Kubuntu em M0.4) |
| **Disk 1** | BIWIN NV7400 4TB | 4.10 TB | NVMe interno | **🪟 Windows 11 (C:) + Data NTFS (D:)** |
| **Disk 2** | SSK Portable SSD 1TB | 1.02 TB | USB externo | **Ventoy (E:) + bkpunvsal (F:)** — ignorar, é o pendrive de boot |

Particionamento detalhado:
- **UMIS:** EFI 2GB + Linux root 918GB + swap 33.9GB
- **BIWIN:** MSR 16MB + Windows C: 1.81TB + EFI 4GB + Recovery 735MB + Data D: 2.00TB
- **SSK USB:** Ventoy E: 454GB + VTOYEFI 32MB + bkpunvsal F: 500GB

**Implicações pro projeto:**
- Cada NVMe interno tem **sua própria ESP** (UMIS 2GB, BIWIN 4GB) — robusto: GRUB não-funcional no UMIS = Windows ainda boota direto via ESP do BIWIN com F12
- M0.6 (instalar Kubuntu) tem como **alvo o Disk 0 (UMIS)**, não Disk 1
- D: NTFS de 2TB no BIWIN será staging RW compartilhado (D-013 nova) — montar em M0.7
- SSD externo **dedicado** para clone do M0.3 (cenário (a) confirmado pelo usuário) — diferente do SSK USB

---

### Passo 6 — Versão atual da BIOS

```powershell
Get-CimInstance -ClassName Win32_BIOS | Format-List Manufacturer, Name, SMBIOSBIOSVersion, Version, ReleaseDate
```

**Output ANTES do flash:**
```
Manufacturer      : LENOVO
Name              : N44ET35W (1.18 )
SMBIOSBIOSVersion : N44ET35W (1.18 )
Version           : LENOVO - 1180
ReleaseDate       : 8/27/2025 9:00:00 PM
```

**Output APÓS o flash:**
```
Manufacturer      : LENOVO
Name              : N44ET37W (1.20 )
SMBIOSBIOSVersion : N44ET37W (1.20 )
Version           : LENOVO - 1200
ReleaseDate       : 12/10/2025 9:00:00 PM
```

Note como o **family code muda** (`N44ET35W → N44ET37W`) — o "letra de release" da BIOS subiu, normal em update.

---

### Passo 7 — Versão atual da vBIOS NVIDIA

```powershell
nvidia-smi --query-gpu=name,vbios_version,driver_version --format=csv
```

**Output ANTES do flash:**
```
name, vbios_version, driver_version
NVIDIA RTX 3000 Ada Generation Laptop GPU, 95.06.31.40.01, 595.71
```

**Output APÓS o flash da vBIOS:**
```
name, vbios_version, driver_version
NVIDIA RTX 3000 Ada Generation Laptop GPU, 95.06.31.40.1c, 595.71
```

Apenas o **último campo** (build/revision) mudou: `01 → 1c` (hex). Mesma família NVIDIA, apenas patch level. Confirma que o pacote Lenovo `20.26.01.2001` carrega vBIOS NVIDIA `95.06.31.40.1c`. Update minor real, sem rewrite.

---

### Passo 8 — Validação pré-flash da BIOS principal

Antes de qualquer flash, conferir que bateria está OK:

```powershell
Get-CimInstance -ClassName Win32_Battery | Format-List Name, EstimatedChargeRemaining, BatteryStatus, EstimatedRunTime
```

**Output validado:**
```
Name                     : 5B11M90092
EstimatedChargeRemaining : 100
BatteryStatus            : 2
EstimatedRunTime         : 71582788
```

**Leitura:**
- `EstimatedChargeRemaining: 100` — bateria cheia ✅
- `BatteryStatus: 2` — em modo "Charging" (carregando, ou seja, na tomada) ✅
- `EstimatedRunTime: 71582788` — valor "infinito" indica que está em AC power, não usando bateria ✅

**Triplo OK:** carregador conectado, bateria 100%, máquina em AC power.

---

### Passo 9 — Flash da BIOS principal (executável Lenovo `n44uj11w`)

Página de download: https://support.lenovo.com/br/pt/downloads/ds568968

Pacote contém: UEFI BIOS v1.20 + ECP (Embedded Controller Program) v1.10

Execução:
1. Baixou o `.exe` direto (sem usar Lenovo Vantage — usuário não tem instalado)
2. Click direito → "Executar como administrador"
3. UAC → Sim
4. Disclaimer Lenovo → Aceitar
5. Reboot automático
6. Tela "Lenovo BIOS Update" com barra de progresso
7. **Memory training ~2 min** (mensagem `"Memory training will take approximately 2 minutes, after which the system will reboot. Do NOT turn off your computer!"`) — **comportamento esperado em DDR5 após flash de BIOS** (vBIOS apaga MRC cache, UEFI nova precisa redescobrir timings DDR5)
8. Reboot adicional
9. Boot normal Windows

**Tempo total:** ~10 minutos do "Start" até login Windows.

Pós-flash, validações:
- `Get-CimInstance Win32_BIOS` → versão nova confirmada
- `powercfg /a` → confirmou que `powercfg /h off` do Passo 2 não foi revertido pelo update da BIOS

---

### Passo 10 — Backup da vBIOS NVIDIA antes do flash dela

> ⚠️ **Decisão consciente:** vBIOS é one-way street. Lenovo não distribui versões antigas, instalador oficial bloqueia downgrade. Sem backup salvo em arquivo, reverter é caçar dump em fórum (TechPowerUp pode não ter da SKU exata Lenovo). Backup é proteção que vale 5 minutos pra eventual reversão futura.

#### Ferramenta usada: `nvflash64.exe` v5.867 (NVIDIA Firmware Update Utility)

Baixado de: https://www.techpowerup.com/download/nvidia-nvflash/

#### Issue 1 encontrado: Memory Integrity (HVCI) bloqueia nvflash

Ao tentar rodar pela primeira vez, output:
```
WARNING: NVFlash detected that Windows Memory Integrity is enabled.
         Windows Memory Integrity may interfere with some NVFlash operations.
         Windows Memory Integrity can be disabled via Windows Settings.
```

**Por quê:** Memory Integrity (HVCI / Hypervisor-Protected Code Integrity) usa virtualização (VBS) pra proteger o kernel. Bloqueia acesso direto ao chip da GPU que `nvflash` precisa pra ler/gravar firmware.

**Caminho:** desabilitar temporariamente, fazer flash + backup, reabilitar depois.

1. `Windows + I` → Privacy & Security → Windows Security → Device security → Core isolation details → toggle Memory integrity OFF
2. Reboot obrigatório
3. Após reboot, tentou rodar nvflash de novo

#### Issue 2 encontrado: GPU não detectada após desabilitar Memory Integrity

```
PS> .\nvflash64.exe --index=0 --save vbios-p16v_bkp-95.06.31.40.01.rom
ERROR: No NVIDIA display adapters found.
```

Mas:
```
PS> .\nvflash64.exe --list
NVIDIA display adapters present in system:
<0> NVIDIA RTX 3000 Ada Generation Laptop GPU (10DE,2838,17AA,232D) S:00,B:01,D:00,F:00
```

`--list` enxergava (consulta via DXGI/driver), `--save` não (precisa acesso direto ao dispositivo PCIe ring-0). Driver NVIDIA estava mantendo a GPU em PowerD3 (Optimus dynamic power management).

**Solução:** rodar `--protectoff` antes:

```powershell
.\nvflash64.exe --index=0 --protectoff
```

Output:
```
NVIDIA Firmware Update Utility (Version 5.867.0)
Copyright (C) 1993-2024, NVIDIA Corporation. All rights reserved.
Setting EEPROM protection complete.
```

`--protectoff` desativa proteção do EEPROM E acorda a GPU pra acesso ring-0. Depois disso:

```powershell
.\nvflash64.exe --index=0 --save vbios-p16v_bkp-95.06.31.40.01.rom
```

Output:
```
Reading EEPROM (this operation may take up to 30 seconds)
Build GUID            : 296302CF3C394CDAAED47DBA95ED1C3C
Build Number          : 33249836
IFR Subsystem ID      : 17AA-232D
Subsystem Vendor ID   : 0x17AA
Subsystem ID          : 0x232D
Version               : 95.06.31.40.01
Image Hash            : N/A
Hierarchy ID          : Normal Board
Build Date            : 08/30/23
Modification Date     : 02/19/24
UEFI Version          : No Version Found or Out-dated (  )
UEFI Variant ID       : No Variant ID Found ( No Variant ID Found )
UEFI Signer(s)        : Unknown signer
XUSB-FW Version ID    : N/A
XUSB-FW Build Time    : N/A
InfoROM Version       : G002.0000.00.03
InfoROM Backup        : Present
License Placeholder   : Present
GPU Mode              : N/A
CEC OTA-signed Blob   : Not Present
```

Validação:
```powershell
Get-Item .\vbios-p16v_bkp-95.06.31.40.01.rom | Format-List Name, Length, LastWriteTime
```

```
Name          : vbios-p16v_bkp-95.06.31.40.01.rom
Length        : 2048000
LastWriteTime : 5/2/2026 11:42:01 PM
```

**2.048.000 bytes (2 MB exatos)** — tamanho típico de vBIOS NVIDIA. Arquivo íntegro.

Cópia pra local seguro (longe do pendrive volátil):

```powershell
New-Item -Path "C:\Users\<user>\bkp-bios" -ItemType Directory -Force
Copy-Item .\vbios-p16v_bkp-95.06.31.40.01.rom -Destination C:\Users\<user>\bkp-bios\
```

---

### Passo 11 — Flash da vBIOS NVIDIA (executável Lenovo `n44vw02w_v3`)

Mesma página Lenovo Support. Pacote `n44vw02w_v3.html` correspondia a "NVIDIA Video BIOS" version `20.26.01.2001` (ID interno do pacote Lenovo, não a versão NVIDIA contida).

Execução: igual ao flash da BIOS principal — `.exe` como admin, reboot, validação com `nvidia-smi`.

Pós-flash, vBIOS subiu de `95.06.31.40.01` → `95.06.31.40.1c`.

#### Anomalia cosmética observada (não impacta funcionalidade)

```powershell
nvidia-smi
```

Output completo:
```
Sat May  2 23:49:14 2026
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 595.71                 Driver Version: 595.71         CUDA Version: 13.2     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                  Driver-Model | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|=========================================+========================+======================|
|   0  NVIDIA RTX 3000 Ada Gene...  WDDM  |   00000000:01:00.0 Off |                  Off |
| N/A   46C    P3            364W /   55W |       0MiB /   8188MiB |      0%      Default |
+-----------------------------------------+------------------------+----------------------+
```

**Nota o `Pwr:Usage/Cap | 364W / 55W`** — número fisicamente impossível em P3 idle (RTX 3000 Ada Laptop tem TGP máximo ~80W). É **bug de leitura do sensor de power**: nova vBIOS mudou tabelas de calibração, driver `595.71` está lendo escala antiga e vomitando lixo.

**Não impacta uso real.** GPU está em P3 (low power), temp 46°C OK, memória 0/8188 MB. Resolverá com:
- Update do driver NVIDIA Windows para versão >596.x, OU
- Será resolvido naturalmente no Linux quando instalarmos driver fresh em M0.5

Registrado como pendência cosmética, não bloqueador.

---

### Passo 12 — Reativar Memory Integrity

`Windows + I` → Privacy & Security → Windows Security → Device security → Core isolation details → toggle Memory integrity ON. Reboot.

Confirmado de volta ao normal.

---

### Passo 13 — Configuração gráfica via WMI

```powershell
Get-PnpDevice -Class Display | Format-List FriendlyName, Status, InstanceId
```

**Output validado:**
```
FriendlyName : Intel(R) Arc(TM) Pro Graphics
Status       : OK
InstanceId   : PCI\VEN_8086&DEV_7D55&SUBSYS_232D17AA&REV_08\3&11583659&0&10

FriendlyName : NVIDIA RTX 3000 Ada Generation Laptop GPU
Status       : OK
InstanceId   : PCI\VEN_10DE&DEV_2838&SUBSYS_232D17AA&REV_A1\4&1EBFE625&0&0008
```

**Leitura:**
- ✅ Ambas GPUs ativas e operacionais simultaneamente
- ✅ Hybrid Graphics confirmado — Optimus funcionando
- 📝 iGPU é "Intel Arc Pro Graphics" (nome novo do Meteor Lake) — não "Intel UHD" ou "Iris Xe"
- 📝 Subsystem `232D17AA` confirma identidade Lenovo P16v Gen 2 com RTX 3000 Ada

---

### Passo 14 — Confirmação visual MUXless no BIOS Setup

Reboot com **F1** durante logo Lenovo → entrou no BIOS Setup → Config → Display.

**Resultado:** **opção `Discrete Graphics` não existe no menu**. Apenas configurações de output display (Boot Display Device, etc.).

**Conclusão:** P16v Gen 2 é **MUXless** — confirmado empiricamente. Anteriormente o glossário e milestones assumiam "MUXed provavelmente"; agora é fato.

---

### Passo 15 — Settings de virtualização e segurança via WMI

```powershell
Get-CimInstance -Namespace root\WMI -ClassName Lenovo_BiosSetting | Where-Object { $_.CurrentSetting -ne "" } | Select-Object CurrentSetting | Format-Table -AutoSize -Wrap | Out-String -Width 200
```

Output completo retornou ~100 settings. Os relevantes pro projeto:

| Setting | Valor | OK pro projeto? |
|---|---|---|
| `SecureBoot` | `Disable` | ✅ pré-req Kubuntu sem signed kernel |
| `VirtualizationTechnology` (VT-x) | `Enable` | ✅ pré-req M2 |
| `VTdFeature` (VT-d) | `Enable` | ✅ pré-req M4-M5 (IOMMU passthrough) |
| `SecurityChip` (TPM 2.0) | `Enable` | ✅ disponível pra disk encryption futura |
| `KernelDMAProtection` | `Disable` | ✅ Linux gerencia IOMMU |
| `TotalMemoryEncryption` | `Disable` | ✅ TME ligado quebra hibernate Linux |
| `BootOrder` | `NVMe0:USBCD:USBFDD:NVMe1:USBHDD:PXEBOOT:LENOVOCLOUD:ON-PREMISE` | ✅ Disk 0 (UMIS=Linux) já tem prioridade — não muda no M0.4 |
| `BootMode` | `Diagnostics` ⚠️ | Anômalo — provável side-effect do flash |

#### M0.1.6.1 — Corrigir `BootMode = Diagnostics`

`BootMode = Diagnostics` é modo especial de debug Lenovo, não normal de produção. Provável side-effect do flash da BIOS.

**Caminho:** reboot, F1, Startup → Boot Mode → mudar de `Diagnostics` para **`Auto`** ou **`UEFI`** → F10 salvar.

Aproveitou-se o reboot pra confirmação visual do MUXless (Passo 14).

---

## ✅ Checklist final do M0.1 (com tudo concluído)

- [x] Fast Startup desabilitado (`powercfg /a` mostra "Hibernation has not been enabled")
- [x] BitLocker verificado em todos os volumes — todos `Protection Off`
- [x] BIOS atualizada de `N44ET35W v1.18` para `N44ET37W v1.20`
- [x] vBIOS NVIDIA atualizada de `95.06.31.40.01` para `95.06.31.40.1c`
- [x] Backup da vBIOS antiga salvo em `C:\Users\<user>\bkp-bios\vbios-p16v_bkp-95.06.31.40.01.rom`
- [x] Mapa real dos discos confirmado: Disk 0 = UMIS (Linux), Disk 1 = BIWIN (Windows + D:), Disk 2 = SSK USB (ignorar)
- [x] Hybrid Graphics confirmado, **MUXless empiricamente confirmado**
- [x] BootMode corrigido de `Diagnostics` para `Auto`/`UEFI`
- [x] Settings VT-x, VT-d, TPM, Secure Boot, KernelDMAProtection, TME, BootOrder anotados — todos OK pro projeto
- [x] Memory Integrity reabilitado após backup/flash da vBIOS
- [x] Documento `m0-windows-tweaks.md` atualizado pra refletir execução real
- [x] STATE.md atualizado para v5

---

## 📤 Para usar este documento como referência futura

Em caso de **reinstalação total do Windows**, replicar os passos críticos pra preservar dual boot estável:

1. **Sempre desabilitar Fast Startup** logo após instalação — `powercfg /h off`
2. **Verificar BitLocker** — `manage-bde -status`. Se ligar (Lenovo costuma ligar Device Encryption por default), suspender antes de qualquer mudança no boot loader
3. **Manter BIOS atualizada** quando puder — `Get-CimInstance Win32_BIOS` pra ver versão atual, comparar com Lenovo Support
4. **NÃO atualizar vBIOS sem backup** — sempre `nvflash64 --save` antes do flash
5. **Memory Integrity** afeta ferramentas de firmware GPU — desligar temporariamente quando precisar de `nvflash`
6. **Ao trocar HW (substituir SSD, troca de motherboard, etc.):** rerodar `Get-Partition` + `Get-Disk` pra confirmar que mapa de discos não inverteu

---

## 🚨 Se algo der errado — recovery procedures

### "powercfg não é reconhecido"
PowerShell sem admin, ou está em CMD. Reabrir PowerShell admin (Win+X → Terminal Admin).

### Acesso negado ao desabilitar Fast Startup
Sem admin. Confirmar com `IsInRole(Administrator)` retornando `True`.

### `manage-bde` retorna erro
- `Acesso negado`: sem admin
- `não reconhecido`: PATH corrompido (raro, contatar suporte)

### `nvflash --save` retorna `No NVIDIA display adapters found`
Sequência conhecida do P16v MUXless:
1. Verificar Memory Integrity está OFF
2. Rodar `nvflash --list` pra confirmar que GPU é detectada
3. Rodar `nvflash --index=0 --protectoff` pra acordar GPU
4. Tentar `--save` de novo

### Flash da BIOS travou na tela "Updating BIOS"
- Esperar mínimo 20 minutos (pode parecer travado mas está flashando)
- Memory training (DDR5) sozinho leva ~2 min
- Após 30 min sem progresso: aceitar risco, força shutdown segurando Power 10s
- Próximo boot: tentar entrar normal. Se não entrar, recovery via Crisis Recovery Lenovo (Boot Block) — guia separado

### vBIOS nova introduz bug que afeta uso
1. Localizar arquivo de backup `.rom` em `C:\Users\<user>\bkp-bios\`
2. Reabrir admin PowerShell + nvflash, desabilitar Memory Integrity novamente
3. Comando: `nvflash64 --override -6 vbios-p16v_bkp-95.06.31.40.01.rom` (override force flash, contornando proteção de versão)
4. Reboot, confirmar versão antiga restaurada

---

## 📚 Termos novos vistos aqui

- **Modern Standby (S0ix)** — ver glossário (substitui S3 em Meteor Lake)
- **Hyper-V / VBS** — Virtualization-Based Security do Windows 11 Pro
- **Memory Integrity (HVCI)** — Hypervisor-Protected Code Integrity, parte do VBS
- **MRC cache** — Memory Reference Code cache, recalibração DDR5 após flash BIOS
- **MUX / MUXed / MUXless** — ver glossário (P16v Gen 2 confirmado MUXless)
- **vBIOS / GOP** — firmware da GPU + Graphics Output Protocol UEFI
- **ECP** — Embedded Controller Program, firmware do controlador embutido
- **nvflash** — utilitário NVIDIA para read/write de vBIOS
- **PRIME render offload** — método Linux para alternar entre iGPU/dGPU em apps específicos

---

**➡️ Próximo passo:** [M0.2 — Coleta do estado atual no EndeavourOS](m0-coleta-estado.md) (será criado na próxima sessão de trabalho)
