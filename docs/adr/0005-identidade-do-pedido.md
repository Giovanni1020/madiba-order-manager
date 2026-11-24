# ADR-0005 — Identidade do pedido: UUID interno + código curto com data

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** o modelo herdado do catálogo **não tem identificador de pedido**
  ([`modelo-de-pedido.md`](../modelo-de-pedido.md)). Independente de stack.

## Contexto

A equipe fala de pedido **em voz alta e no WhatsApp**. Um identificador técnico resolve o
banco e não resolve a conversa: UUID é impronunciável.

Duas tentativas óbvias falham por motivos opostos:

- **Sequencial global** (`1`, `2`, … `1378`) — em três meses chega a quatro dígitos e fica
  difícil de falar.
- **Reinício diário sem data** (`1`..`N`, zerando todo dia) — mantém o número curto, mas
  *"pedido 3"* lido no dia seguinte aponta para outro pedido.

## Decisão

**D1 — UUID interno como chave primária.** É o que as relações e a auditoria usam. Nunca é
mostrado nem falado.

**D2 — Código humano `DDMM-NN`**, com o contador **reiniciando a cada dia**: `2211-03` é o
terceiro pedido de 22 de novembro. O reinício diário é o que mantém `NN` em um ou dois
dígitos; o prefixo de data é o que remove a ambiguidade entre dias. Os dois juntos resolvem os
dois problemas do Contexto.

**D3 — `unique (data, numero)` no banco, e o número vem de sequência por dia.** Nunca de
`count(*) + 1`: duas inserções simultâneas leriam a mesma contagem. A restrição de unicidade
é a rede de segurança — em dia de pico ela é o que impede dois pedidos com o mesmo código.

**D4 — O código é para falar, não para ordenar nem para identificar.** Nenhuma lógica de
domínio resolve pedido por ele.

## Consequências

- `DDMM-NN` **não ordena cronologicamente** como texto (`0101` vem antes de `3112`, e o ano
  não está lá). Aceito: ordenação usa a data real do pedido, e o código existe para a
  conversa. Incluir o ano custaria dois dígitos no que precisa ser pronunciável.
- O código **não é único no histórico de vários anos** — `2211-03` repete em 2027. Para a
  conversa do dia a dia isso não atrapalha; para busca no histórico, a chave é data + código.
- A sequência por dia é estado do lado do servidor. Precisa existir antes do primeiro
  cadastro, junto com a tabela de pedidos.
