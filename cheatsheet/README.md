# Cheatsheet — Dicionário de Comandos Úteis

> Aqui ficam os comandos reais que rodamos durante o projeto, organizados por tema. **Cada comando que executamos juntos vira uma entrada aqui** — assim, quando você precisar de novo no futuro, encontra rápido.

**Como usar:** Ctrl+F com o que você quer fazer. Ex: "snapshot", "atualizar", "wifi".

**Convenção de validação:** comandos com ✅ foram **executados de verdade neste hardware específico** (P16v Gen 2, BIOS N44ET37W v1.20, Win11 Pro Workstation 25H2). Comandos sem marca ainda são planejados ou referenciais.

---

## Status atual

✅ **M0.1 concluído (02/05/2026)** — todos os comandos abaixo na seção "Windows (PowerShell)" foram validados nesta sessão.

⏸️ Demais seções (Linux, BTRFS, NVIDIA Linux) — aguardando M0.2 em diante.

---

## 1. Windows — PowerShell admin

### 1.1 Como abrir PowerShell como Administrador

**Win+X (mais rápido):**
1. `Windows + X`
2. Escolher **"Terminal (Admin)"**
3. UAC → Sim

**Validar que está como admin:**

```powershell
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

✅ Esperado: `True`. Se `False`, fechar e reabrir como admin.

**O que faz:** consulta `WindowsIdentity::GetCurrent()`, embrulha como WindowsPrincipal, testa membership no role `Administrator`.

---

### 1.2 Desabilitar Fast Startup + Hibernação (uma cajadada)

```powershell
powercfg /h off
```

✅ Output: silêncio (prompt volta direto). Silêncio = sucesso (padrão Unix-like).

**O que faz:**
- Desliga hibernação Windows (`/h` = `/hibernate`)
- Fast Startup cai junto (Fast Startup é hibernação parcial)
- Apaga `C:\hiberfil.sys` (libera ~tamanho-da-RAM em GB — no nosso caso, ~24-32 GB)

**Reverter:** `powercfg /h on`

---

### 1.3 Listar estados de suspensão suportados

```powershell
powercfg /a
```

✅ Output validado (Meteor Lake usa Modern Standby / S0ix, não S3):
```
The following sleep states are available on this system:
    Standby (S0 Low Power Idle) Network Connected
The following sleep states are not available on this system:
    Standby (S1)/(S2)/(S3) — firmware does not support
    Hibernate — Hibernation has not been enabled.
    Hybrid Sleep — multiple deps missing
    Fast Startup — Hibernation is not available.
```

**Validador chave após `/h off`:** `Hibernate` e `Fast Startup` em "not available" com razão `"Hibernation has not been enabled"`.

**Bandeira interessante:** se aparecer `"The hypervisor does not support this standby state"` no Hybrid Sleep → Hyper-V/VBS está ativo.

---

### 1.4 Verificar status do BitLocker

```powershell
manage-bde -status
```

(sem `C:` → lista todos os volumes)

✅ Output esperado para volume sem BitLocker:
```
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
```

**Procurar por:**
- `Conversion Status: Fully Decrypted` — não criptografado ✅
- `Protection Status: Protection Off` — BitLocker desativado ✅
- `Key Protectors: None Found` — sem chave ativa ✅

**Se BitLocker estiver ativo:** suspender antes de mudar boot loader: `manage-bde -protectors -disable C:`

---

### 1.5 Mapear discos físicos (descobrir qual NVMe é qual)

```powershell
Get-Partition | Sort-Object DiskNumber, PartitionNumber | Format-Table -AutoSize DiskNumber, PartitionNumber, DriveLetter, Size, Type
```

✅ Lista todas as partições com disco físico, número, letra, tamanho e tipo.

```powershell
Get-Disk | Sort-Object Number | Format-Table -AutoSize Number, FriendlyName, Size, BusType, OperationalStatus
```

✅ Lista discos físicos com modelo, tamanho, tipo de bus (NVMe / USB / SATA), status.

**Combinação:** `DiskNumber` do `Get-Partition` cruza com `Number` do `Get-Disk`. Permite descobrir qual disco físico (modelo, bus) está cada partição (letra, função).

---

### 1.6 Versão atual da BIOS Lenovo

```powershell
Get-CimInstance -ClassName Win32_BIOS | Format-List Manufacturer, Name, SMBIOSBIOSVersion, Version, ReleaseDate
```

✅ Output validado:
```
Manufacturer      : LENOVO
Name              : N44ET37W (1.20 )
SMBIOSBIOSVersion : N44ET37W (1.20 )
Version           : LENOVO - 1200
ReleaseDate       : 12/10/2025 9:00:00 PM
```

**Family code Lenovo P16v Gen 2:** `N44ETxxW` — cada minor release sobe o `xx`.

---

### 1.7 Identificar Machine Type Lenovo (para download de BIOS)

```powershell
Get-CimInstance -ClassName Win32_ComputerSystemProduct | Format-List Vendor, Name, Version
```

✅ Os 4 primeiros caracteres do `Name` são o Machine Type (ex: `21KX` ou `21KY` no P16v Gen 2). Crítico pra confirmar compatibilidade com pacote Lenovo antes de baixar.

> 🔒 **Privacidade:** o `Name` completo inclui SKU regional (semi-sensível). Compartilhar só os 4 primeiros caracteres. NÃO rodar com `IdentifyingNumber` — esse campo retorna serial number da máquina.

---

### 1.8 Status da bateria (validação pré-flash de BIOS)

```powershell
Get-CimInstance -ClassName Win32_Battery | Format-List Name, EstimatedChargeRemaining, BatteryStatus, EstimatedRunTime
```

✅ Output validado:
```
Name                     : 5B11M90092
EstimatedChargeRemaining : 100
BatteryStatus            : 2
EstimatedRunTime         : 71582788
```

**Códigos de `BatteryStatus`:**
- `2` = Carregando (na tomada) ✅ ideal pra flash de BIOS
- `3` = Carga completa ✅
- `1` = Descarregando ⚠️
- `4` = Baixa, `5` = Crítica ❌ não fazer flash

**`EstimatedRunTime` muito alto (~71M segundos):** está em AC power, não usando bateria.

---

### 1.9 Listar GPUs ativas (Hybrid Graphics check)

```powershell
Get-PnpDevice -Class Display | Format-List FriendlyName, Status, InstanceId
```

✅ Output validado:
```
FriendlyName : Intel(R) Arc(TM) Pro Graphics
Status       : OK
InstanceId   : PCI\VEN_8086&DEV_7D55&SUBSYS_232D17AA&REV_08\...

FriendlyName : NVIDIA RTX 3000 Ada Generation Laptop GPU
Status       : OK
InstanceId   : PCI\VEN_10DE&DEV_2838&SUBSYS_232D17AA&REV_A1\...
```

**Leitura:**
- Ambas com `Status: OK` = Hybrid Graphics ativo (Optimus)
- Vendor IDs: `8086` (Intel), `10DE` (NVIDIA)
- Subsystem `232D17AA` confirma Lenovo P16v Gen 2 com RTX 3000 Ada

---

### 1.10 Listar TODOS os settings do BIOS Lenovo via WMI

```powershell
Get-CimInstance -Namespace root\WMI -ClassName Lenovo_BiosSetting | Where-Object { $_.CurrentSetting -ne "" } | Select-Object CurrentSetting | Format-Table -AutoSize -Wrap | Out-String -Width 200
```

✅ Lista ~100 settings em formato `Nome,Valor`. Filtrar por interesse:

```powershell
# Settings de virtualização
Get-CimInstance -Namespace root\WMI -ClassName Lenovo_BiosSetting | Where-Object { $_.CurrentSetting -match "Virtualization|VTd|Hypervisor" } | Select-Object -ExpandProperty CurrentSetting

# Settings de segurança
Get-CimInstance -Namespace root\WMI -ClassName Lenovo_BiosSetting | Where-Object { $_.CurrentSetting -match "SecureBoot|SecurityChip|TPM|TXT" } | Select-Object -ExpandProperty CurrentSetting

# Settings de boot
Get-CimInstance -Namespace root\WMI -ClassName Lenovo_BiosSetting | Where-Object { $_.CurrentSetting -match "Boot" } | Select-Object -ExpandProperty CurrentSetting
```

**Settings críticos pro projeto e valores esperados:**

| Setting | Valor esperado pra dual boot Linux |
|---|---|
| `SecureBoot` | `Disable` |
| `VirtualizationTechnology` (VT-x) | `Enable` |
| `VTdFeature` (VT-d) | `Enable` |
| `SecurityChip` (TPM) | `Enable` |
| `KernelDMAProtection` | `Disable` |
| `TotalMemoryEncryption` | `Disable` |
| `BootMode` | `Auto` ou `UEFI` (não `Diagnostics`!) |

---

## 2. NVIDIA — vBIOS e diagnóstico (Windows)

### 2.1 Versão atual da vBIOS NVIDIA

```powershell
nvidia-smi --query-gpu=name,vbios_version,driver_version --format=csv
```

✅ Output validado:
```
name, vbios_version, driver_version
NVIDIA RTX 3000 Ada Generation Laptop GPU, 95.06.31.40.1c, 595.71
```

**Formato vBIOS NVIDIA:** `XX.XX.XX.XX.XX` (5 partes, hex). Não confundir com formato de pacote Lenovo (`20.26.01.2001` é ID interno do pacote, não a vBIOS dentro dele).

---

### 2.2 Status completo NVIDIA (single shot)

```powershell
nvidia-smi
```

✅ Mostra: nome GPU, driver, CUDA version, temp, perf state (P0-P12), power usage/cap, memória usada/total, processos ativos.

**Bug cosmético conhecido em P16v após flash de vBIOS:** `Pwr:Usage/Cap` reporta valores absurdos tipo `364W / 55W` em idle. Não impacta uso. Fix: update do driver NVIDIA.

---

### 2.3 Status NVIDIA com refresh contínuo (loop)

```powershell
nvidia-smi -l 1
```

Refresh a cada 1 segundo. Útil para **acordar dGPU em modo Optimus PowerD3** (que pode bloquear `nvflash`). Rodar em segundo terminal mantém a GPU acordada enquanto outra ferramenta tenta acessá-la.

Sair: `Ctrl+C`.

---

### 2.4 Listar GPUs NVIDIA detectadas pelo nvflash

```powershell
.\nvflash64.exe --list
```

✅ Output validado:
```
NVIDIA display adapters present in system:
<0> NVIDIA RTX 3000 Ada Generation Laptop GPU (10DE,2838,17AA,232D) S:00,B:01,D:00,F:00
```

**Importante:** `--list` consulta via DXGI/driver (userspace). `--save` precisa de acesso direto PCIe (ring-0). Se `--list` enxerga mas `--save` falha, problema é com PowerD3 / Memory Integrity bloqueando ring-0 — usar `--protectoff` primeiro.

---

### 2.5 Backup da vBIOS atual

```powershell
.\nvflash64.exe --index=0 --protectoff
.\nvflash64.exe --index=0 --save vbios-p16v_bkp-<versao>.rom
```

✅ Sequência validada. `--protectoff` desativa proteção do EEPROM E acorda a GPU pra acesso ring-0.

Output esperado do `--save`: dump de metadados do firmware (Build GUID, Subsystem ID, Version) + arquivo `.rom` de **2.048.000 bytes (2 MB exatos)** salvo no diretório atual.

**Validação do arquivo:**
```powershell
Get-Item .\vbios-p16v_bkp-*.rom | Format-List Name, Length, LastWriteTime
```

`Length: 2048000` = arquivo íntegro. Se vier 0 ou poucos KB, falhou.

**Mover pra local seguro (longe de pendrive volátil):**
```powershell
New-Item -Path "C:\Users\<user>\bkp-bios" -ItemType Directory -Force
Copy-Item .\vbios-p16v_bkp-*.rom -Destination C:\Users\<user>\bkp-bios\
```

---

### 2.6 Restaurar vBIOS antiga (recovery, raramente necessário)

```powershell
.\nvflash64.exe --override -6 C:\Users\<user>\bkp-bios\vbios-p16v_bkp-<versao>.rom
```

`--override`: bypass checks de versão (permite "downgrade").  
`-6`: aceita warnings sobre subsystem ID mismatch e força flash.

⚠️ Só usar se versão atual quebrou algo. Memory Integrity precisa estar OFF.

---

## 3. Pre-requisitos pra ferramentas que mexem em firmware

### 3.1 Desabilitar Memory Integrity (HVCI) temporariamente

Pra `nvflash` ou outras ferramentas de firmware GPU funcionarem, Memory Integrity precisa estar OFF.

**Caminho GUI:**
1. `Windows + I` (Settings)
2. Privacy & Security → Windows Security → Device security
3. Core isolation details
4. Memory integrity → **OFF**
5. Reboot obrigatório

**Reabilitar:** mesma navegação, toggle ON, reboot.

> ⚠️ **Lembrar de reabilitar após terminar.** Memory Integrity é proteção real contra ataques ao kernel. Não deixar OFF permanentemente.

---

## 4. Bootloader / dual boot (Windows side)

### 4.1 Listar entradas de boot UEFI no Windows

```powershell
bcdedit /enum firmware
```

Equivalente Windows do `efibootmgr` Linux. Mostra entries UEFI ativas.

(Não foi necessário rodar em M0.1, mas vai aparecer em M0.4 quando GRUB for instalado pra confirmar que boot order está correto.)

---

## 5. Linux — geral (placeholder, M0.2 em diante)

### 5.1 Listar discos e partições
- [ ] `lsblk -f` — listar discos com filesystem types
- [ ] `fdisk -l` — listar partições detalhado
- [ ] `parted -l` — alternativa moderna

### 5.2 Identificar UUID de partições
- [ ] `blkid` — UUIDs e tipos
- [ ] `findmnt --target /` — onde root está montado

### 5.3 Ver entradas de boot UEFI
- [ ] `sudo efibootmgr -v` — entries verboso

### 5.4 Atualizar sistema
- [ ] `sudo apt update && sudo apt full-upgrade`

### 5.5 Instalar/remover pacotes
- [ ] `sudo apt install <pacote>`
- [ ] `sudo apt remove <pacote>` (mantém configs)
- [ ] `sudo apt purge <pacote>` (remove configs)
- [ ] `sudo apt autoremove`

### 5.6 Gerenciar serviços
- [ ] `systemctl status <serviço>`
- [ ] `sudo systemctl start/stop/restart <serviço>`
- [ ] `sudo systemctl enable/disable <serviço>`

### 5.7 Ver logs
- [ ] `journalctl -xe` — últimos eventos
- [ ] `journalctl -u <serviço>` — logs de serviço específico
- [ ] `journalctl -b` — logs do boot atual
- [ ] `journalctl -b -1` — logs do boot anterior

### 5.8 Uso de disco
- [ ] `df -h` — espaço por mountpoint
- [ ] `du -sh <pasta>` — tamanho de pasta
- [ ] `ncdu` — interactive disk usage

### 5.9 Uso de memória
- [ ] `free -h` — memória rapidinho
- [ ] `htop` — top interativo

---

## 6. BTRFS e Snapper (placeholder, M0.5 em diante)

### 6.1 Subvolumes
- [ ] `sudo btrfs subvolume list /` — listar subvolumes
- [ ] `sudo btrfs subvolume create <nome>` — criar
- [ ] `sudo btrfs subvolume delete <nome>` — apagar

### 6.2 Snapshots via Snapper
- [ ] `sudo snapper -c root list` — listar snapshots
- [ ] `sudo snapper -c root create -d "descrição"` — criar manual
- [ ] `sudo snapper -c root undochange <num>..<num>` — comparar/reverter

### 6.3 Espaço BTRFS
- [ ] `sudo btrfs filesystem usage /` — uso real (não confiar em `df` puro com BTRFS)

---

## 7. NVIDIA Linux (placeholder, M0.5 em diante)

### 7.1 Verificar driver instalado
- [ ] `nvidia-smi` — mesmo comando do Windows, mas no Linux

### 7.2 Listar drivers disponíveis
- [ ] `ubuntu-drivers list` — recomendados pelo Ubuntu

### 7.3 Instalar driver recomendado
- [ ] `sudo ubuntu-drivers autoinstall`

### 7.4 Configurar GPU primária (em hardware MUXless)
- [ ] `sudo prime-select intel` — modo iGPU only (economia)
- [ ] `sudo prime-select nvidia` — modo dGPU only (alta performance)
- [ ] `sudo prime-select on-demand` — modo PRIME render offload (apps escolhem)

> 📝 P16v Gen 2 é MUXless (confirmado em M0.1.6) — `on-demand` é o modo natural pra esse hardware.

### 7.5 Run app específico na dGPU (PRIME render offload)
- [ ] `__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia <app>`

---

## 8. Bootloader Linux (placeholder, M0.4 em diante)

### 8.1 GRUB
- [ ] `sudo update-grub` — regerar `/boot/grub/grub.cfg`
- [ ] `sudo grub-install` — reinstalar (raro)

### 8.2 efibootmgr
- [ ] `sudo efibootmgr -v` — listar entries
- [ ] `sudo efibootmgr -o XXXX,YYYY,ZZZZ` — mudar ordem
- [ ] `sudo efibootmgr -B -b XXXX` — apagar entry

---

## 9. Diagnóstico hardware Linux (placeholder)

- [ ] `inxi -Fxxxz` — overview saneado
- [ ] `lshw -short` — hardware listado
- [ ] `lspci -v` — devices PCI verboso
- [ ] `lsusb` — devices USB
- [ ] `lscpu` — topologia CPU (P-cores + E-cores etc)
- [ ] `sensors` — temperaturas
- [ ] IOMMU groups script:
  ```bash
  for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%%/*}; printf 'IOMMU Group %s ' "$n"; lspci -nns "${d##*/}"; done
  ```

---

## 10. Pacotes alternativos Linux (placeholder)

- [ ] `flatpak list` — Flatpaks instalados
- [ ] `flatpak install flathub <app>` — instalar
- [ ] `flatpak update` — atualizar

- [ ] `snap list` — Snaps instalados
- [ ] `sudo snap install <pacote>` — instalar
- [ ] `sudo snap remove <pacote>` — remover

---

## 11. Wine / Gaming (placeholder, M1 em diante)

- [ ] Steam → Settings → Compatibility → habilitar Proton para todos os títulos
- [ ] Lutris setup
- [ ] Heroic Games Launcher (Epic, GOG)

---

## 12. Virtualização (placeholder, M2 em diante)

- [ ] `virsh list --all` — listar VMs
- [ ] `virsh start <nome>` / `virsh shutdown <nome>`
- [ ] `virsh edit <nome>` — editar XML

---

## Apêndice — Conversão de tamanhos (referência rápida)

Outputs do PowerShell costumam vir em **bytes**. Conversão:

| Bytes | Tamanho legível |
|---|---|
| 1.024 | 1 KB |
| 1.048.576 | 1 MB |
| 1.073.741.824 | 1 GB |
| 1.099.511.627.776 | 1 TB |

**Macetes:**
- `2147483648` ≈ 2 GB (potência exata: 2^31)
- `2.048.000` exatos = vBIOS NVIDIA (2 MiB padrão)
- `1.024.209.543.168` ≈ 1 TB (capacidade real de SSD "1 TB")
- `4.096.805.658.624` ≈ 4 TB (capacidade real de SSD "4 TB")

---

> **Princípio do cheatsheet:** Conforme cada item vira realidade neste projeto, ele sai do "placeholder" e ganha entrada completa com **comando completo**, **explicação**, **output validado** e **gotchas conhecidos** — como já está feito pra seção 1, 2 e 3 (executados em M0.1).
