# Calibração — quantos litros vale 1 pulso?

O fator de calibração (litros por pulso) define se o sensor diz a verdade. No reed DAEFLEX a especificação de fábrica é **10 L/pulso**, mas vale **conferir contra o display** — é rápido e elimina qualquer dúvida.

---

## O fator do reed DAEFLEX

O fabricante especifica o reed switch DAEFLEX com **resolução de 10 litros por pulso** (1 pulso = 0,01 m³ = 100 pulsos/m³). Isso corresponde a uma volta completa do último rolo do contador (unidades de litro), que carrega o ímã.

| Medido | Valor |
|--------|-------|
| 1 pulso | **0,01 m³ = 10 L** |
| 100 pulsos | 1 m³ |

> 💡 Diferente do hidrômetro (onde a etiqueta "R500" enganava e o fator real só apareceu na calibração), aqui o **10 L/pulso é a especificação do próprio sensor** — mas confirmar contra o display custa nada e protege contra surpresas (ex.: ímã pulsando 2× por volta).

---

## Procedimento de calibração (recomendado)

1. Anote a leitura do **display do medidor** (último dígito vermelho = litros). → `display_inicio`
2. Anote o `Volume Gás` da ESP no mesmo instante. → `volume_inicio`
3. **Use um volume conhecido de gás** — deixe o aquecedor/fogão ligado até consumir algumas dezenas de litros (quanto maior, menor o erro relativo).
4. Anote display e volume no fim. → `display_fim`, `volume_fim`
5. Compare o incremento do display (em m³) com o incremento do HA. Devem ser iguais. Se quiser o fator em L/pulso:

   ```
   pulsos = (volume_fim − volume_inicio) ÷ multiply_total_atual
   fator (L/pulso) = (display_fim − display_inicio) × 1000 ÷ pulsos
   ```

6. Deve dar **≈ 10 L/pulso**. Se der o dobro (≈ 20), o ímã pode estar gerando 2 contagens por volta — veja [DIAGNOSTICO.md](DIAGNOSTICO.md) (contar o dobro).

---

## Como ajustar o YAML

O firmware usa dois multiplicadores em [`esphome/medidor-gas.yaml`](../esphome/medidor-gas.yaml):

```yaml
# Vazão (pulsos/min → m³/h):  multiply = (L/pulso ÷ 1000) × 60
filters:
  - multiply: 0.6     # para 10 L/pulso

# Total (pulsos → m³):  multiply = L/pulso ÷ 1000
total:
  filters:
    - multiply: 0.01  # para 10 L/pulso
```

Regra geral, conforme o fator:

| L/pulso | `multiply` da vazão | `multiply` do total |
|---------|---------------------|---------------------|
| 1 L | 0,06 | 0,001 |
| 5 L | 0,30 | 0,005 |
| **10 L** | **0,60** | **0,01** |
| 100 L | 6,0 | 0,1 |

Fórmulas: vazão = `(L/pulso ÷ 1000) × 60` · total = `L/pulso ÷ 1000`.

---

## Offset (leitura inicial do medidor)

Por padrão o total **começa em 0** e conta a partir da instalação (foi a opção escolhida neste projeto). Se você quiser que o `Volume Gás` reflita a leitura real do medidor, adicione um `offset` no filtro do total:

```yaml
total:
  filters:
    - multiply: 0.01
    - offset: 1634.27   # leitura do display no dia da instalação, em m³
```

---

## Tabela para você preencher

| Campo | Valor |
|-------|-------|
| `display_inicio` (m³) | __________ |
| `volume_inicio` (HA, m³) | __________ |
| `display_fim` (m³) | __________ |
| `volume_fim` (HA, m³) | __________ |
| Incremento display (L) = (display_fim − display_inicio) × 1000 | __________ |
| Incremento HA (L) = (volume_fim − volume_inicio) × 1000 | __________ |
| Batem? (sim/não) | __________ |

---

➡️ Sinal não conta ou não bate? Veja **[DIAGNOSTICO.md](DIAGNOSTICO.md)**.
