# Radeon E9173 — VBIOS original e com overclock

Duas ROMs da **AMD Radeon E9173** (Polaris 12 "Lexa XL", 2 GB GDDR5 de 64 bits):

| Arquivo | O que é |
| :--- | :--- |
| `Lexa.rom` | VBIOS original, sem modificação |
| `Lexa_OC.rom` | A mesma VBIOS, com clocks e limite de energia maiores |

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

## Como gravar

> ⚠️ **Por sua conta e risco.** Gravar uma VBIOS errada pode deixar a placa sem vídeo. Faça um backup da ROM atual antes de tudo e só use estas ROMs numa placa com o mesmo PCI ID, subsystem e memória.

Use o **amdvbflash** (ATI Flash), em Windows ou Linux, como administrador/root:

```bash
amdvbflash -i                       # lista as placas e o índice de cada uma (0, 1…)
amdvbflash -s 0 backup.rom          # salva a ROM atual da placa 0
amdvbflash -unlockrom 0             # libera a gravação, se ela estiver protegida
amdvbflash -p 0 Lexa_OC.rom         # grava a ROM com overclock
```

Reinicie o PC depois de gravar. Para voltar ao original, grave `Lexa.rom` (ou o seu `backup.rom`) do mesmo jeito.

Se a placa ficar sem vídeo, ligue o PC com outra saída de vídeo (a integrada ou outra placa) e grave o backup de volta.

### Sem gravar nada (Linux)

No Linux dá para chegar aos mesmos clocks sem tocar na VBIOS, pelo overdrive do driver `amdgpu`. Adicione `amdgpu.ppfeaturemask=0xffffffff` aos parâmetros do kernel e ajuste os estados em `/sys/class/drm/card0/device/pp_od_clk_voltage`. Ao reiniciar, a placa volta ao padrão.

## Monitorar

Para acompanhar uso, clocks, temperatura e consumo da placa no painel do GNOME, veja o [gpu-monitor-gnome](https://github.com/1155dosilicon/gpu-monitor-gnome), que foi testado nesta mesma placa.
