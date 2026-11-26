# Radeon E9173 — VBIOS original, overclock e curva de ventoinha

Três ROMs da **AMD Radeon E9173** (Polaris 12 "Lexa XL", 2 GB GDDR5 de 64 bits):

| Arquivo | O que é |
| :--- | :--- |
| `Lexa.rom` | VBIOS original, sem modificação |
| `Lexa_OC.rom` | A mesma VBIOS, com clocks e limite de energia maiores |
| `Lexa_OC_Fan_v2.rom` | Baseada na `Lexa_OC.rom`, com curva de ventoinha mais responsiva |

## Identificação

Confira se a sua placa é igual a esta antes de usar qualquer ROM.

| Campo | Valor |
| :--- | :--- |
| PCI ID | `1002:699F` (Lexa) |
| Subsystem | `1002:EA73` |
| Part number | `113-D0912600-100` |
| Descrição na ROM | `D09124 Lexa XL A0 GDDR5 64bit 2GB 214e/300m` |
| Versão ATOMBIOS | `015.050.002.001.009139` (13/02/2018) |
| Tamanho | 256 KiB (imagem de 58,5 KiB) |

No Linux, `lspci -nn -d 1002:` mostra o PCI ID e `lspci -vnn -d 1002:699f` mostra o subsystem.

## O que muda na `Lexa_OC.rom`

São 14 bytes diferentes, todos na tabela PowerPlay, mais a soma de verificação, que foi corrigida. As voltagens, a curva da ventoinha e os limites de temperatura continuam iguais aos da original.

| Parâmetro | `Lexa.rom` | `Lexa_OC.rom` |
| :--- | ---: | ---: |
| Clock máximo da GPU | 1219 MHz | **1315 MHz** |
| Clock máximo da memória | 1500 MHz | **1805 MHz** |
| Limite de energia (TDP) | 17 W | **30 W** |
| Limite de entrega de energia | 17 W | **30 W** |
| Limite de overdrive da GPU | — | 1585 MHz |
| Limite de overdrive da memória | — | 1950 MHz |
| Voltagem no estado máximo | 1050 mV | 1050 mV |
| Temperatura máxima (TjMax) | 93 °C | 93 °C |

Os demais estados de clock da GPU (214 a 1183 MHz) e da memória (300 e 625 MHz) não mudaram.

> O overclock usa a mesma voltagem da original. A estabilidade depende do chip, e cada placa reage de um jeito.

## O que muda na `Lexa_OC_Fan_v2.rom`

Mantém todos os clocks, voltagens e limites de 30 W da `Lexa_OC.rom` e altera 15 bytes na tabela da ventoinha (`FanTable`, rev 9, offset `0x9e07`) mais o checksum (`0x21`: `0x76` → `0x0c`), totalizando 16 bytes diferentes. Na VBIOS original, a curva parava em 60% a 85 °C com temperatura alvo de 80 °C e teto de 4300 RPM, deixando a placa passar de 80 °C em carga total.

| Parâmetro | `Lexa_OC.rom` | `Lexa_OC_Fan_v2.rom` |
| :--- | ---: | ---: |
| Ponto mínimo (`TMin` → `PWMMin`) | 40 °C → 20% | **35 °C → 30%** |
| Ponto médio (`TMed` → `PWMMed`) | 65 °C → 40% | **50 °C → 60%** |
| Ponto alto (`THigh` → `PWMHigh`) | 85 °C → 60% | **80 °C → 85%** |
| Temperatura alvo (`ucTargetTemperature`) | 80 °C | **65 °C** |
| RPM máximo (`usFanRPMMax`) | 4300 RPM | **5500 RPM** |
| Modo de controle (`ucFanControlMode`) | `1` (Fuzzy) | `1` (mantido) |
| Temperatura de 100% (`usTMax`) | 109 °C | 109 °C (mantido) |
| PWM mínimo (`ucMinimumPWMLimit`) | 20% | 20% (mantido) |

Os campos `ucFanControlMode = 1`, `usTMax = 109 °C` e `ucMinimumPWMLimit = 20%` foram mantidos iguais aos da original para evitar falha de inicialização (Erro 43) no driver do Windows. Em teste de estresse 3D em carga total, a placa caiu de 80+ °C para **70 °C no máximo**, mantendo folga na ventoinha e sustentando o boost do overclock.

## Como gravar

> ⚠️ **Por sua conta e risco.** Gravar uma VBIOS errada pode deixar a placa sem vídeo. Faça um backup da ROM atual antes de tudo e só use estas ROMs numa placa com o mesmo PCI ID, subsystem e memória.

Use o **amdvbflash** (ATI Flash), em Windows ou Linux, como administrador/root. No Windows, use a versão **2.93** (a série 5.x removeu o suporte a Polaris `0x699f`):

```bash
amdvbflash -i                       # lista as placas e o índice de cada uma (0, 1…)
amdvbflash -s 0 backup.rom          # salva a ROM atual da placa 0
amdvbflash -unlockrom 0             # libera a gravação, se ela estiver protegida
amdvbflash -f -p 0 Lexa_OC_Fan_v2.rom  # grava a ROM com overclock + curva de ventoinha
```

Reinicie o PC depois de gravar. No Windows, se o driver oficial bloquear a VBIOS modificada (Erro 43), aplique o *AMD/ATI Pixel Clock Patcher* (`atikmdag-patcher`) e reinicie. Para voltar ao original, grave `Lexa.rom` (ou o seu `backup.rom`) do mesmo jeito.

Se a placa ficar sem vídeo, ligue o PC com outra saída de vídeo (a integrada ou outra placa) e grave o backup de volta.

### Sem gravar nada (Linux)

No Linux dá para chegar aos mesmos clocks sem tocar na VBIOS, pelo overdrive do driver `amdgpu`. Adicione `amdgpu.ppfeaturemask=0xffffffff` aos parâmetros do kernel e ajuste os estados em `/sys/class/drm/card0/device/pp_od_clk_voltage`. Ao reiniciar, a placa volta ao padrão.

## Monitorar

Para acompanhar uso, clocks, temperatura e consumo da placa no painel do GNOME, veja o [gpu-monitor-gnome](https://github.com/1155dosilicon/gpu-monitor-gnome), que foi testado nesta mesma placa.
