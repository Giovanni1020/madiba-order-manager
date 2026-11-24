# ADR-0001 — Peso como unidade de capacidade de produção

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** app interno de acompanhamento de pedidos de buquê, **sem código
  ainda**. Stack e persistência pendentes — **nada neste ADR depende delas.**

## Contexto

A loja **já mede** o "peso" de um buquê: uma grandeza que representa **tempo e esforço de
montagem**. Um solitário de rosa é uma pessoa por ~2 minutos; um buquê de 10 rosas precisa de
mais de uma pessoa e leva bem mais tempo. A medida existe para a maioria dos buquês e é usada
hoje no **cálculo de preço**, como componente de mão de obra — um buquê vendido só com lucro
sobre a flor dá prejuízo quando se contam armazenagem e funcionários.

Duas necessidades operacionais dependem dessa grandeza:

- o **atendente** precisa saber se ainda dá para encaixar mais um pedido;
- **quem produz** precisa saber o que fazer e em que ordem.

O risco a evitar é pegar pedido demais.

## Decisão

**D1 — Peso mede exclusivamente tempo e esforço de produção.** Ele **não define preço**: é
*insumo* do preço, representando mão de obra, ao lado dos outros componentes. A unidade é a
que a loja já pratica — o app **não a redefine** — e é armazenada como **inteiro** (nunca
ponto flutuante, pela mesma razão que dinheiro vive em centavos).

**D2 — Peso nunca é alavanca comercial.** Desconto é **campo próprio no preço final**.
Abaixar peso para fechar venda é proibido por construção: se peso pudesse ser negociado, o
planejamento de produção passaria a mentir — o sistema acharia que o buquê leva menos tempo
porque alguém quis cobrar menos.

**D3 — Peso é atributo da montagem, não do produto nem da variante.** Variantes que diferem
apenas na aparência da flor (5 rosas importadas × 5 rosas nacionais) **compartilham o mesmo
peso**: o trabalho é idêntico. A tabela de pesos é indexada pela **forma da montagem**, não
por `product_id`. Indexar por produto duplicaria o mesmo peso em N variantes, e elas
divergiriam sozinhas na primeira revisão.

**D4 — Exceções entram por override manual no pedido.** Casos atípicos que a tabela não
cobre são ajustados no pedido, não na tabela. O override é auditado
([ADR-0004](0004-registro-e-auditoria.md)).

**D5 — O que a gestão ajusta é a capacidade da hora, não o peso.** São coisas de naturezas
diferentes: **peso é medida da realidade** (quanto custa montar aquilo) e **capacidade é
decisão de gestão** (quanto a equipe se compromete a produzir naquela hora). Quando a equipe
adianta o serviço e sobra tempo, o gerente **expande a capacidade** daquela hora para caber
mais alguns buquês. A tabela de pesos não é tocada nessa operação.

**D6 — O limite é "soft": avisa, nunca bloqueia.** Aproximando do limite, aviso; passando do
limite, **aviso forte** — e o pedido entra. Razão: a loja legitimamente aceita pedido acima do
limite quando decide aceitar. Ferramenta que bloqueia é ferramenta que as pessoas contornam
por fora, e aí o sistema perde justamente o dado que existia para capturar.

## Consequências

- A **capacidade da hora é entidade de primeira classe**, com histórico: quem expandiu,
  quando, de quanto e para qual hora ([ADR-0004](0004-registro-e-auditoria.md),
  [ADR-0009](0009-autorizacao-em-ponto-unico.md)).
- O peso praticado é **congelado no pedido** ([ADR-0006](0006-congelamento-de-preco-e-peso.md)):
  revisar a tabela de pesos não pode reescrever o histórico de produção.
- A aritmética de fila e orçamento sai daqui para o
  [ADR-0002](0002-fila-por-prazo-e-orcamento.md).
- **Capacidade de entrega não usa peso.** Peso mede montagem, não rota: dois buquês de peso
  igual podem ser uma entrega de 10 e outra de 50 minutos, porque o que limita entrega é
  distância, trânsito e número de paradas. Medir isso exige dados que a loja não tem hoje —
  nem com entrada manual, já que duas entregas próximas custam menos que duas em sentidos
  opostos. Fica como **objetivo futuro**, com unidade própria, fora do v1 e da v1.1.
- Buquês sem peso medido precisam de um caminho: por ora, override manual (D4). Se virarem
  rotina, revisar.
