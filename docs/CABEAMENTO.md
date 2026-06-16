# Cabeamento — do medidor de gás ao ESP32

Este é o documento mais importante do projeto. Acertar o cabeamento é 90% do trabalho; o resto é copiar e colar YAML.

---

## O reed switch DAEFLEX

O reed switch é um **contato seco** (sem eletrônica, sem alimentação): dois fios que se fecham quando o ímã do contador passa perto. Como é contato puro, **a polaridade não importa** eletricamente — um fio vai ao GPIO, o outro ao GND.

| Fio do reed | Função | Liga a |
|-------------|--------|--------|
| Fio de sinal A | um lado do contato | ✅ GPIO27 |
| Fio de sinal B | outro lado do contato | ✅ GND |
| *(se houver) saída anti-fraude* | sinaliza remoção/fraude magnética | ❌ não usado neste projeto (isolar) |

> 💡 **Como funciona a contagem:** em repouso o contato está aberto e o **pull-up interno do ESP32** segura a linha em 3,3 V (HIGH). Quando o ímã passa, o contato fecha e puxa a linha para 0 V (LOW). O firmware conta **a borda de descida** — 1 pulso = 1 passagem do ímã = **10 L**.

![Reed switch encaixado no medidor de gás](imgs/medidor_reed.jpg)

---

## Por que cabo de rede CAT6 blindado (FTP)?

O cabo do reed é curto (75 cm). Para estendê-lo até a caixa do ESP32, o **cabo CAT6 FTP externo** é uma escolha excelente:

- **Pares trançados** reduzem ruído capacitivo/indutivo — protege o pulso de "contagem fantasma".
- A **blindagem (malha FTP)** dá imunidade extra, importante porque aqui o cabo pode correr perto de 220 V dentro da caixa.
- É de **dupla capa / externo**, próprio para ficar exposto ao tempo.

> ⚠️ **Aterre a malha (shield) só de um lado** — no GND do ESP32 — para evitar loop de terra.

---

## Mapeamento dos fios nos pares do cabo

O segredo da imunidade a ruído: **sinal e GND andam SEMPRE no mesmo par trançado.**

| Par do CAT6 | Liga a |
|-------------|--------|
| **Par laranja** (laranja + branco-laranja) | sinal do reed + GND — **essencial, não separe** |
| Malha (FTP) | GND do ESP (um lado só) |
| Demais pares | livres / reserva |

> 💡 As cores do cabo de rede não têm relação com as do reed. O que importa: **os dois fios do contato no mesmo par trançado**.

---

## Caminho do sinal

```
Reed switch (no medidor, 75 cm)
   │  2 fios do contato
   ▼
Emenda IP68 (conector alavanca 2 vias)
   │  par laranja do CAT6 FTP
   ▼
Cabo CAT6 FTP blindado (externo)
   │  passa pelo prensa-cabo da caixa IP65
   ▼
WAGO 221 (distribuição) → borne da placa adaptador
   │  sinal→GPIO27, retorno→GND, malha→GND
   ▼
ESP32
```

![Emenda IP68 entre o reed e o cabo de extensão](imgs/emenda_ip68.jpg)

---

## Cuidados

- 💧 **Vede bem a emenda externa** (lado do medidor). Ela fica exposta — a emenda IP68 já resolve, mas confira o aperto. Água na emenda = leitura maluca ou sensor mudo.
- ⚡ **Separe do 220 V dentro da caixa.** A blindagem ajuda, mas mantenha o cabo do reed o mais longe possível da fiação da tomada/fonte.
- 📏 **Comprimento:** o reed é contato seco (sinal robusto, não é sinal fraco como hall), então estender até ~10 m com o CAT6 funciona bem. Mantenha o par trançado sinal+GND.
- 🧲 **Posição do reed:** alinhe ao **último dígito** do contador (rolo com o ímã). Se não pulsar, reposicione poucos milímetros até pegar o ímã (ver [DIAGNOSTICO.md](DIAGNOSTICO.md)).

---

➡️ Próximo passo: a montagem física, em **[INSTALACAO.md](INSTALACAO.md)**.
