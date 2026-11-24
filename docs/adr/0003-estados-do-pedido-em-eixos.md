# ADR-0003 — Estados do pedido em eixos ortogonais

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** o modelo de pedido herdado
  ([`modelo-de-pedido.md`](../modelo-de-pedido.md)) tem `Entrega` como **união discriminada**
  em `tipo` (`retirada` | `entrega`). Independente de stack.

## Contexto

A lista levantada na primeira passagem tinha oito valores num enum só: *pedido feito (sinal)*,
*pedido feito (pago)*, *em produção*, *pronto para entrega/retirada*, *em entrega*,
*entregue/retirado*, *cancelado*, *erro com o pedido*.

Três problemas aparecem quando isso é um enum único:

- **Pagamento não é fase do pedido.** Estar *em produção com apenas o sinal* é caso normal da
  loja. Com um enum só, esse pedido não é representável: ou se perde a informação de
  pagamento, ou a de produção.
- **Erro não é fase.** Um pedido com problema de comunicação **continua** em produção ou
  pronto. Se "erro" ocupa o estado, perde-se onde o pedido estava — e quando o problema se
  resolve, ninguém sabe para onde voltar.
- **O fluxo bifurca.** Retirada nunca passa por "em rota de entrega".

## Decisão

**D1 — Quatro eixos independentes**, não um enum:

| Eixo | Natureza | Valores |
|---|---|---|
| **Estado** | um por vez, transições controladas | ver D2 |
| **Pagamento** | ortogonal, manual | `sem sinal` · `sinal` · `pago` |
| **Atenção** | ortogonal, flag + motivo | `ok` · `precisa de intervenção` |
| **Tempo** | **derivado**, nunca gravado | `adiantado` · `no tempo` · `atrasando` · `atrasado` |

**D2 — A máquina de estados bifurca depois de "pronto"**, conforme `entrega.tipo`:

```
recebido ──▶ em produção ──▶ pronto ──┬─[retirada]─▶ retirado
                                      └─[entrega]──▶ em rota ──▶ entregue

cancelado: alcançável de qualquer estado anterior a entregue/retirado
```

"Em rota" significa **em rota de entrega** — o pedido saiu da loja.

**D3 — Pagamento é manual e arbitrário.** Não há integração com provedor nem recebimento
automático de dados de pagamento. É um campo de três valores que alguém marca. Barato de
implementar, por isso entra no v1. **É exibido na lista**: quem abre o app para ver a situação
do pedido precisa saber se está só com sinal.

**D4 — Atenção é flag com motivo, e é visualmente distinta na lista.** Preserva o estado real
por baixo, e a marcação precisa **destacar o pedido** entre os outros — é o que faz o caso de
intervenção humana ser notado (erro de comunicação com cliente, pedido duplicado). Resolver o
problema limpa a flag; o estado nunca foi perdido.

**D5 — Tempo é derivado da folga** ([ADR-0002](0002-fila-por-prazo-e-orcamento.md) D4) e
**não é coluna**.

**D6 — Transição é função pura.** `proximoEstado(pedido, evento)` devolve o pedido novo ou um
erro de domínio. Transição inválida falha **no domínio**, não por botão escondido na tela.

## Consequências

- `em produção` + `sinal` + `precisa de atenção` é um estado representável. Na lista original
  de oito valores, era impossível — e é caso real.
- Consultas de operação passam a cruzar eixos ("pronto e não pago", "atrasando e com
  atenção"). Isso é recurso, não custo.
- **`em rota` existe e é operado no v1**, marcado manualmente por quem atende. O que fica
  para a v1.1 é a **tela do entregador** (role própria, atualização pelo celular dele) e o
  modelo de capacidade de entrega
  ([ADR-0001](0001-peso-como-unidade-de-capacidade.md), Consequências).
- Cancelado é terminal e não volta. Reabrir é pedido novo — e o log liga os dois
  ([ADR-0004](0004-registro-e-auditoria.md)).
- Toda transição gera evento de auditoria ([ADR-0004](0004-registro-e-auditoria.md)).
