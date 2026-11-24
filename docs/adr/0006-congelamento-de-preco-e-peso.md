# ADR-0006 — Preço e peso congelados no pedido

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** o catálogo calcula preço por `basePrice(product, variant)`, com
  variantes de preço absoluto (ADR-0005 do catálogo) e promoções com `expiresAt` (ADR-0006 do
  catálogo). Peso vem de tabela por montagem
  ([ADR-0001](0001-peso-como-unidade-de-capacidade.md) D3).

## Contexto

Preço e peso são, os dois, **consultados em tabela** no momento da venda. A pergunta é se o
pedido guarda o **valor** ou a **referência**.

Vale separar duas preocupações que parecem a mesma:

- **Desync** — tabela e app discordando no mesmo instante. **Não é problema desta loja:**
  atualização é rara, feita de madrugada, com reset manual de servidor, e nunca houve
  incidente.
- **Reescrita do passado** — o pedido é **registro histórico**. Se guardar referência, uma
  atualização de preço perfeitamente legítima em janeiro muda **o que o pedido de dezembro diz
  que custou**. Não é bug de sincronização: é uma operação correta alterando o passado, sem
  ninguém ter motivo para desconfiar.

É a segunda que motiva este ADR.

## Decisão

**D1 — O pedido guarda valores, não referências.** No momento da inserção são copiados:

- **preço** de cada item (já com variante e promoção resolvidas) e o **total**, em centavos
  inteiros;
- **peso** de cada item e o **peso total**, em inteiro.

**D2 — Referência ao produto continua existindo**, para saber *o que* foi vendido. Ela não é
usada para **recalcular** preço nem peso de pedido já registrado.

**D3 — Vale também para o override manual de peso**
([ADR-0001](0001-peso-como-unidade-de-capacidade.md) D4): o valor praticado fica no pedido.

## Consequências

- Revisar a tabela de pesos (receitas mudam, fornecedor muda) **não reescreve** o histórico de
  produção. Sem isso, a pergunta *"quanto a gente conseguiu produzir em novembro?"* passaria a
  ter resposta diferente a cada revisão de tabela.
- Divergência entre um pedido antigo e o catálogo atual é **correta, não bug**. Qualquer tela
  que compare os dois precisa dizer qual é qual.
- Custo: duas colunas copiadas na inserção. Zero manutenção depois.
- Relatório de faturamento e de capacidade passam a ser deriváveis **só dos pedidos**, sem
  depender do estado atual das tabelas.
