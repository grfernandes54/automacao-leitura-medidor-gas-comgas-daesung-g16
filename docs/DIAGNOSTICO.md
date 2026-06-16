# Diagnóstico e troubleshooting

Como confirmar que o sinal está chegando e o que fazer quando algo não bate — incluindo os erros reais enfrentados nesta instalação.

---

## Características do sinal (referência)

| Característica | Valor |
|---------------|-------|
| Tipo | contato seco (reed switch) |
| Repouso (idle) | ~3,3 V (puxado pelo pull-up interno do ESP32) |
| Ativo (ímã presente) | cai para ~0 V (contato fechado) |
| Cadência | lenta — no fluxo máximo (Qmáx 2,5 m³/h) ≈ 4 pulsos/min (1 a cada ~14 s) |
| *Bounce* | milissegundos (contato mecânico) → exige `internal_filter` em ms |
| Risco para o GPIO | zero — contato seco, sem tensão própria |

---

## Verificação sem instrumentos

1. Multímetro em **continuidade/resistência** direto nos 2 fios do reed.
2. Aproxime um **ímã** do reed (ou gire o último dígito consumindo gás): o multímetro deve indicar **contato fechado** (bipe/continuidade) quando o ímã está presente e **aberto** quando se afasta.
3. Com o ESP ligado e o fio no GPIO27, medindo tensão DC (preto no GND, vermelho no sinal): **~3,3 V em repouso**, caindo para perto de 0 V a cada passagem do ímã.

---

## Sintomas → causa → solução

| Sintoma | Causa provável | Solução |
|---------|----------------|---------|
| Sensor zerado, mas há gás passando | reed mal posicionado (não pega o ímã), fiação solta, `count_mode` errado | reposicione o reed sobre o último dígito; confira os bornes; confirme `pullup: true` e `falling_edge: INCREMENT` |
| `Maximum internal filter value when using ESP32 hardware PCNT is 13us` (no validate) | reed precisa de debounce em ms, mas o PCNT limita a 13 µs | **`use_pcnt: false`** no `pulse_counter` (conta por interrupção, aceita filtro grande) |
| Conta o dobro do esperado | contando 2 bordas, ou *bounce* não filtrado | garanta `rising_edge: DISABLE`; aumente `internal_filter` (100 ms → 200 ms) |
| Contagem fantasma / pulsos a mais | ruído acoplado no cabo (perto do 220 V) | sinal+GND no mesmo par trançado; aterre a malha do FTP no GND; afaste o cabo da fiação AC |
| Total zerou após reboot | falta o `utility_meter` (total do pulse_counter vive na RAM) | configure o `utility_meter` — ver [INSTALACAO.md](INSTALACAO.md#10-configuração-do-utility_meter) |
| Vazão oscila/zera em fluxo baixo | normal: pulsos muito espaçados | é esperado; o que importa é o **total**. Pode aumentar o `update_interval` |
| Valores plausíveis mas não batem com o display | fator de calibração | calibre contra o display — ver [CALIBRACAO.md](CALIBRACAO.md) |

---

## Erros de gravação / adoção (vividos nesta instalação)

| Erro | Causa | Correção |
|------|-------|----------|
| `Secret 'xxx' not defined` no validate | nome do secret não existe no `secrets.yaml` | alinhe os nomes: `wifi_ssid`, `wifi_password`, `api_key`, `ota_password`, `ap_password` |
| OTA: `Authentication invalid. Is the password correct?` | a senha OTA do firmware atual no ESP ≠ a do config | regrave **por USB** uma vez (serial ignora a auth OTA); depois OTA volta a funcionar |
| HA pede "Chave de encriptação" e falha | falta/erro da `api_key`, ou chave repetida com outro device | cole o valor exato de `api_key`; use uma **chave exclusiva** por ESP (gere com `openssl rand -base64 32`) |
| Warnings de chip rev / SRAM1 no log | tuning de binário | no `esp32: framework: advanced:` use `minimum_chip_revision: "3.1"` e `sram1_as_iram: true` (se o seu ESP for rev ≥ 3.0) |

---

## pulse_counter vs pulse_meter (importante para reed)

- **`pulse_counter` com `use_pcnt: false`** (usado aqui): conta por **interrupção de software**, com `internal_filter` em **milissegundos** — ideal para o *bounce* do reed mecânico.
- **`pulse_counter` com `use_pcnt: true`**: usa o periférico PCNT (filtro só em µs). Ótimo para sensor eletrônico limpo (é o caso do hidrômetro ultrassônico), **mas não serve para reed** — gera o erro do filtro de 13 µs.
- **`pulse_meter`**: alternativa também por interrupção; serve para reed, mas o `pulse_counter` com `use_pcnt: false` já cobre bem este caso.

> 🧲 Resumo: **reed = mecânico = `use_pcnt: false` + filtro em ms.** Essa é a diferença-chave em relação ao projeto do hidrômetro.
