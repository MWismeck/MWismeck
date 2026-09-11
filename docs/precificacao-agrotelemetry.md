# Modelo de Precificação — AgroTelemetry

> **Objetivo:** entender o custo total de operar a ferramenta (infraestrutura própria na fase
> inicial, assinaturas de IA, tempo de desenvolvimento e migração para AWS) para chegar a um
> **preço por máquina/mês** que cubra os custos e gere margem.
>
> **Escala considerada:** começa com **~70 máquinas** monitoradas, cresce até **~300** em cerca de
> **6 meses** (fase de infra própria). Depois migra para **AWS**, que fica **estável por 1–2 anos**
> sem precisar reescalar.

---

## ⚠️ Como ler este documento

Não consegui abrir o repositório `agrotelemetry` nesta sessão (fora do escopo de acesso), então
**todos os números abaixo são premissas editáveis**, não valores confirmados. A estrutura do modelo
está pronta: é só substituir os valores da tabela de premissas pelos reais e os totais se ajustam.

**Premissa importante que você precisa confirmar:** neste modelo, *"máquinas"* = **equipamentos
agrícolas monitorados** (tratores/colheitadeiras/sensores enviando telemetria), que são a **carga**
do sistema e a **base de cobrança** — não são servidores. Se "máquinas" no seu contexto forem
servidores, me avise que o modelo muda de forma.

**Câmbio usado:** `US$ 1 = R$ 5,50` (ajuste conforme o dia).

---

## 1. Resumo executivo (a resposta)

Considerando o **cenário base** (detalhado adiante), ao longo dos ~30 meses de horizonte
(6 meses fase própria + 24 meses AWS):

| Métrica | Sem contar mão de obra | Contando mão de obra |
|---|---:|---:|
| Custo total do período | ~R$ 123.000 | ~R$ 411.000 |
| **Custo por máquina/mês** | **~R$ 15** | **~R$ 50** |
| **Preço sugerido (margem ~40%)** | **~R$ 25/máquina/mês** | **~R$ 70/máquina/mês** |
| Break-even mensal (fase AWS) | ~45 máquinas | ~150 máquinas |

**Recomendação:** precifique entre **R$ 60 e R$ 90 por máquina/mês** (contando mão de obra e margem).
A R$ 70/máquina, com 300 máquinas você fatura ~R$ 21.000/mês e cobre todo o custo operacional já a
partir de ~150 máquinas. A faixa de R$ 25–35 só se sustenta se o tempo de desenvolvimento for tratado
como investimento próprio (equity) e não precisar ser recuperado no preço.

> Estes valores saem 100% das premissas da Seção 3. Troque as premissas e recalcule.

---

## 2. As categorias de custo

Todo o custo da ferramenta se divide em 5 baldes:

1. **Infraestrutura própria** — fase 1 (meses 0–6, servidores durante o crescimento 70→300).
2. **Infraestrutura AWS** — fase 2 (meses 6+, operação estável).
3. **Assinaturas de IA / ferramentas de dev** — Claude e Cursor (custo de *construir*, não de rodar).
4. **Mão de obra / tempo** — 2–3 devs (o maior custo; pode ser tratado como custo real ou como equity).
5. **Custos acessórios** — domínio, SSL, e-mail, monitoramento, backup, gateway de pagamento.

---

## 3. Premissas (edite esta tabela)

### 3.1 Escala e cronograma

| Premissa | Valor | Observação |
|---|---:|---|
| Máquinas no início | 70 | ponto de partida |
| Máquinas ao fim da fase 1 | 300 | teto da fase própria |
| Duração da fase 1 (infra própria) | 6 meses | crescimento 70→300 |
| Duração da fase 2 (AWS estável) | 24 meses | "1–2 anos sem reescalar" |
| Média de máquinas na fase 1 | ~185 | média entre 70 e 300 |
| **Máquina-meses fase 1** | **~1.110** | 185 × 6 |
| **Máquina-meses fase 2** | **~7.200** | 300 × 24 |
| **Total máquina-meses (30 meses)** | **~8.310** | base do rateio |

### 3.2 Assinaturas de IA / ferramentas (mensal, 2 devs)

| Ferramenta | Plano | Preço unit. | Qtd | US$/mês | R$/mês |
|---|---|---:|---:|---:|---:|
| Claude | Max (5×) | US$ 100 | 2 | 200 | 1.100 |
| Cursor | Pro | US$ 20 | 2 | 40 | 220 |
| **Subtotal IA** | | | | **240** | **~1.320** |

> Planos de referência (jul/2026): **Claude** — Pro US$ 20, Max 5× US$ 100, Max 20× US$ 200,
> Team ~US$ 30/usuário. **Cursor** — Pro US$ 20, Business US$ 40/usuário, Ultra US$ 200.
> Com 3 devs, multiplique proporcionalmente (~R$ 1.980/mês).
>
> **Atenção à distinção:** Claude/Cursor aqui são o custo de **desenvolver** a ferramenta.
> Se o *produto em si* chamar a API da Claude em runtime (ex.: gerar insights sobre a telemetria),
> isso é um **custo variável separado por uso** — precifica por tokens e escala com o nº de máquinas.
> Não está incluído acima; me diga se o produto usa IA em runtime que eu modelo à parte.

### 3.3 Infraestrutura própria — fase 1 (mensal)

| Item | R$/mês | Observação |
|---|---:|---|
| Servidor de aplicação (backend Go + API) | 600 | VPS/dedicado médio |
| Banco de dados / série temporal | 300 | TimescaleDB/Influx para telemetria |
| Rede / energia / margem | 100 | se for hardware próprio, some energia+colocation |
| **Subtotal infra fase 1** | **~1.000** | escala modesta: 300 máquinas é carga leve-média |

### 3.4 Infraestrutura AWS — fase 2 (mensal, 300 máquinas estáveis)

| Serviço | US$/mês | R$/mês | Observação |
|---|---:|---:|---|
| EC2 (2× instâncias app) | 180 | 990 | m5.large ou equivalente |
| RDS / banco gerenciado | 150 | 825 | ou Timescale autogerido em EC2 |
| Ingestão de telemetria (IoT Core / fila) | 120 | 660 | ~78M msg/mês a ~10s/leitura |
| S3 + transferência + backup | 60 | 330 | armazenamento histórico |
| CloudWatch / diversos | 40 | 220 | monitoramento, logs |
| **Subtotal AWS fase 2** | **~550** | **~3.025** | ~R$ 3.000/mês |

> A ingestão é o item que mais varia com a **frequência de envio**. A 1 leitura/10s por máquina,
> 300 máquinas geram ~78M mensagens/mês. Se a frequência cair para 1/min, o custo de ingestão
> despenca (~13M msg). **Confirme a frequência real** — é a alavanca nº 1 do custo AWS.

### 3.5 Mão de obra / tempo (o maior balde)

| Fase | Alocação | R$/mês | Meses | Total |
|---|---|---:|---:|---:|
| Fase 1 (construção intensiva) | 2 devs full | 24.000 | 6 | 144.000 |
| Fase 2 (manutenção/evolução) | ~0,5 dev | 6.000 | 24 | 144.000 |
| **Subtotal mão de obra** | | | | **~288.000** |

> Assume custo/oportunidade de **R$ 12.000/dev/mês** (pleno-sênior Go no Brasil). Se vocês são
> fundadores e não vão pagar salário agora, esse balde vira **investimento (equity)** — daí a coluna
> "sem mão de obra" da Seção 1. A decisão de incluí-lo ou não é o que mais muda o preço final.

### 3.6 Acessórios (mensal)

| Item | R$/mês |
|---|---:|
| Domínio + SSL + e-mail | 50 |
| Monitoramento/erro (ex.: Sentry free/pago) | 100 |
| Gateway de pagamento (fixo; % fica no preço) | 50 |
| **Subtotal acessórios** | **~200** |

---

## 4. Custo total do horizonte (30 meses)

| Categoria | Cálculo | Total |
|---|---|---:|
| Infra própria (fase 1) | R$ 1.000 × 6 | 6.000 |
| Infra AWS (fase 2) | R$ 3.000 × 24 | 72.000 |
| Assinaturas IA | R$ 1.320 × 30 | ~39.000 |
| Acessórios | R$ 200 × 30 | 6.000 |
| **Subtotal operacional (sem mão de obra)** | | **~123.000** |
| Mão de obra | Seção 3.5 | 288.000 |
| **TOTAL (com mão de obra)** | | **~411.000** |

### Custo por máquina/mês (rateio pelos 8.310 máquina-meses)

| Base | Total | ÷ máquina-meses | **Custo/máquina/mês** |
|---|---:|---:|---:|
| Operacional | 123.000 | 8.310 | **~R$ 15** |
| Com mão de obra | 411.000 | 8.310 | **~R$ 50** |

---

## 5. Cenários

Três olhares para a mesma estrutura, variando frequência de telemetria, câmbio e alocação de devs.

| Item | Conservador | **Base** | Otimista |
|---|---:|---:|---:|
| Infra AWS/mês | R$ 4.500 | R$ 3.000 | R$ 1.800 |
| IA/mês | R$ 1.980 (3 devs) | R$ 1.320 | R$ 660 (1 dev) |
| Mão de obra total | R$ 360.000 | R$ 288.000 | R$ 180.000 |
| Custo/máquina/mês (c/ MO) | ~R$ 68 | ~R$ 50 | ~R$ 30 |
| **Preço sugerido (margem 40%)** | **~R$ 95** | **~R$ 70** | **~R$ 42** |

---

## 6. Preço, margem e break-even

### 6.1 Preço sugerido
Partindo do custo com mão de obra (~R$ 50/máquina/mês) e aplicando **margem de 40%**:

> **Preço ≈ R$ 70 por máquina/mês.**

Faixa saudável para negociar por volume: **R$ 60 (planos com muitas máquinas) a R$ 90 (poucas máquinas)**.

### 6.2 Break-even na operação estável (fase AWS)
Custo mensal recorrente na fase 2 (AWS + IA + acessórios + 0,5 dev de manutenção):

`3.000 + 1.320 + 200 + 6.000 = ~R$ 10.520/mês`

| Preço/máquina | Máquinas p/ break-even |
|---:|---:|
| R$ 50 | ~210 |
| **R$ 70** | **~150** |
| R$ 90 | ~117 |

A **R$ 70/máquina com 300 máquinas** → receita ~**R$ 21.000/mês**, cobrindo o recorrente com folga
(~2×) e começando a amortizar o investimento de construção.

### 6.3 Recuperação do investimento inicial
Investimento de construção (fase 1: MO + infra) ≈ **R$ 150.000**. Com margem de contribuição de
~R$ 60/máquina/mês (preço R$ 70 − recorrente ~R$ 10/máquina a 300 máquinas), a 300 máquinas você gera
~R$ 18.000/mês de contribuição → **payback em ~8–9 meses** de operação estável.

---

## 7. O que mais mexe no preço (sensibilidade)

Em ordem de impacto:

1. **Mão de obra incluída ou não** — muda o custo/máquina de ~R$ 15 para ~R$ 50. É a maior alavanca.
2. **Frequência de envio da telemetria** — dirige o custo de ingestão AWS (item 3.4). 1/10s vs 1/min
   pode dividir o custo de ingestão por ~6.
3. **Nº de máquinas efetivo** — o rateio é super sensível ao volume; quanto mais máquinas, menor o
   custo unitário (economia de escala forte, pois a infra é quase fixa até certo ponto).
4. **Câmbio** — infra AWS e IA são em dólar; uma alta de 10% no câmbio sobe ~R$ 4.000 no total.
5. **Se o produto usa IA em runtime** — vira custo variável por token que escala com o uso.

---

## 8. Checklist — trocar premissas pelos números reais

- [ ] "Máquinas" = equipamentos monitorados? (confirmar a base de cobrança)
- [ ] Frequência de envio de telemetria por máquina (1/10s? 1/min? evento?)
- [ ] Volume de dados por leitura (nº de campos/bytes) → dimensiona storage
- [ ] Custo real dos servidores da fase 1 (VPS alugado? hardware próprio? colocation?)
- [ ] Planos reais de Claude e Cursor e nº de devs assinantes
- [ ] Vão pagar salário aos devs agora, ou tratar como investimento/equity?
- [ ] O produto chama IA (Claude API) em runtime? Se sim, qual volume por máquina?
- [ ] Câmbio de referência para o orçamento
- [ ] Margem-alvo (usei 40%)

Preenchendo esses itens, os totais das Seções 4–6 se ajustam e o **preço por máquina/mês** fica firme.

---

## 9. Próximo passo sugerido

Se quiser, eu transformo este modelo em uma **planilha Excel com fórmulas** (abas de premissas +
cenários que recalculam sozinhos), ou em uma **calculadora HTML interativa** onde você arrasta os
valores e vê o preço mudar ao vivo. É só pedir.
