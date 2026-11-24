# ADR-0008 — PWA mobile-first, online-only

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** app interno, menos de 10 pessoas, uso praticamente só em celular.

## Contexto

O foco é mobile (declarado como "99.9%"), e app nativo foi descartado desde o início como
complicação sem retorno. A pergunta real era outra: **o app precisa funcionar sem rede?**

## Decisão

**D1 — PWA instalável na tela inicial.** Sem loja de aplicativos. Reaproveita integralmente o
conhecimento de web do catálogo.

**D2 — Online-only.** Offline-first foi **rejeitado** com argumento operacional: nem pagamento
nem comunicação com a cliente funcionam sem conexão. Um app de fila que insiste em funcionar
offline, quando a parte essencial da venda não funciona, é trabalho jogado fora — e custa de 3
a 5 vezes mais (fila local, sincronização, resolução de conflito).

**D3 — O que fica, e não é offline-first: não perder o que já foi digitado.** Formulário que
sobrevive a alguns segundos de queda, com retry. Isso é não maltratar quem usa, não
sincronização.

**D4 — WebView wrapper (Capacitor e similares) rejeitado por ora.** Só se passar a ser
necessária presença em loja de apps ou API nativa que o PWA não alcance. Para app interno de
menos de 10 pessoas, não se paga.

**D5 — Mobile de verdade.** Usado de pé, uma mão, segurando flores: alvos de toque grandes,
zero dependência de `hover`, confirmação em ação destrutiva. A lista de viewports é a do
catálogo — `docs/aparelhos-suportados.md` lá — reaproveitada, não reescrita.

## Consequências

- Sem rede o app não serve. **Aceito**, por D2.
- Instalação é por navegador, e cada pessoa precisa fazer uma vez. Vira item de implantação,
  não de código.
- Desktop não é proibido, só não é prioridade de layout. A tela do dia
  ([ADR-0002](0002-fila-por-prazo-e-orcamento.md) D6) é a candidata natural a ganhar uma
  versão larga depois, para quem produz olhar numa tela maior.
