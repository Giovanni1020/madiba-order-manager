# ADR-0002 — Fila por prazo e orçamento de produção ao longo do dia

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** depende de [ADR-0001](0001-peso-como-unidade-de-capacidade.md)
  (peso). Independente de stack.

## Contexto

O pedido já nasce com **data** (ISO `yyyy-mm-dd`) e **janela de horário** — o catálogo
trabalha com **períodos de 1 hora**, e o app adota o mesmo agrupamento. Duas perguntas
diferentes precisam ser respondidas pela mesma estrutura:

- *"ainda dá para encaixar?"* (atendente, no momento de aceitar)
- *"o que eu faço agora?"* (produção, durante o dia)

A palavra "fila" sugere ordem de chegada, mas buquê é **perecível com prazo marcado**: um
pedido registrado hoje para o dia 30 não vem antes de um registrado amanhã para o dia 24.

## Decisão

**D1 — Ordenação por prazo mais próximo primeiro (EDF).** O prazo é a janela de 1 hora da
entrega ou retirada. *Earliest Deadline First* tem a propriedade de que **se existe alguma
ordem capaz de cumprir todos os prazos, EDF cumpre**. Consequência prática: **não existe
prioridade manual** na fila, e não precisa existir.

**D2 — Capacidade é fluxo acumulado, não cota isolada por janela.** O trabalho de um buquê
para as 14h acontece **antes** das 14h, então orçamento preso dentro da janela de entrega não
modela a realidade. A capacidade disponível até um instante `T` é a soma dos limites das horas
até `T`:

```
capacidade(até T) = Σ limite(h), para toda hora h ≤ T
```

**D3 — Teste de viabilidade do dia.** Para **cada** janela `T`:

```
Σ peso dos pedidos com prazo ≤ T   ≤   capacidade(até T)
```

Válido para todas as janelas ⇒ dia viável. Se quebra, a conta diz **em qual janela** e **de
quanto**. É uma passada única sobre a lista já ordenada por D1. O resultado é **aviso**, nunca
bloqueio ([ADR-0001](0001-peso-como-unidade-de-capacidade.md) D6).

**D4 — Urgência é derivada da folga e nunca é armazenada.**

```
folga = capacidade restante até o prazo − peso que falta produzir
```

Folga confortável ⇒ *adiantado*; perto de zero ⇒ *atrasando*; negativa ⇒ *atrasado*. Guardar
essa classificação num campo garante que ela estará errada na próxima vez que a tela abrir,
porque **ela muda com o relógio sem ninguém tocar em nada**.

**D5 — A fila não é entidade.** É uma *view* ordenada sobre `pedidos`. Não existe tabela
`fila`, nem posição persistida. Posição persistida criaria dois lugares discordando sobre o
que vem primeiro, e o resto do projeto seria gasto sincronizando os dois.

**D6 — A tela do dia é segmentada hora a hora.** Cada hora exibe: os pedidos daquela janela,
o peso somado da hora, o limite da hora e a folga acumulada. Razão: um número agregado do dia
**não significa nada para uma pessoa** sem a fórmula na cabeça, enquanto a lista hora a hora é
legível direto — dá para ver o que sai às 10h e o que sai às 15h. A conta de D3 existe para
alimentar o aviso; a leitura do dia é a lista.

**D7 — O relógio e o fuso entram por parâmetro.** Fuso da loja: **`America/Sao_Paulo`**
(Alvorada/RS). "Hoje", "agora" e o recorte das horas são resolvidos na **borda** do sistema e
passados ao domínio ([ADR-0007](0007-dominio-puro-e-porta-de-ingestao.md)). O núcleo nunca
adivinha que horas são.

## Consequências

- Expandir o limite de uma hora **re-avalia o dia a partir dela**: a folga de todas as
  janelas seguintes muda, porque capacidade é acumulada (D2).
- A viabilidade é a única lógica do app em que um erro custa dinheiro físico (aceitar pedido
  que a loja não consegue montar). Por isso ela é **função pura e testada**
  ([ADR-0007](0007-dominio-puro-e-porta-de-ingestao.md)).
- Um pedido **sem janela definida** (cliente ainda não decidiu o horário) não tem prazo e
  portanto não entra na conta de D3. Onde ele aparece na tela do dia fica **em aberto**.
- Pedidos de dias futuros consomem capacidade dos **seus** dias, não do dia corrente. A tela
  do dia é por data.
