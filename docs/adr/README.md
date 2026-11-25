# ADRs — Madiba Order Manager

Decisões de arquitetura, numeradas (`0001-….md`, `0002-….md`, …), uma por arquivo — para que
cada uma possa ser revisada ou substituída sozinha. Formato: **Status** · **Contexto do
projeto** · **Substitui** e **Revisões** (opcionais) · **Contexto** · **Decisão** (`D1`…`Dn`) ·
**Consequências**, igual ao do catálogo.

**ADR novo parte do modelo [`0000-template.md`](0000-template.md)**, que traz o gate de 3 SIM
(alternativas reais + muda a estrutura + motivo não óbvio) e pede as **alternativas consideradas
no Contexto**. Os ADRs 0001–0010 são anteriores ao modelo: alguns já trazem as alternativas no
Contexto (0004, 0005), outros as deixam espalhadas pela Decisão. Não precisam ser reescritos só por
isso; quando um deles for alterado por outro motivo, a alteração aproveita para seguir o modelo.

## Como um ADR muda

- **Proposto:** pode ser editado livremente. Passa a **Aceito** no merge do PR que o aprova.
- **Aceito, sem código que implemente a decisão alterada:** o ADR é **editado no lugar**, no PR que
  muda a decisão, com uma linha no campo **Revisões** do cabeçalho (data, o que mudou, PR). O
  gatilho é **por decisão** (`Dn`): se o item que muda ainda não tem código na `main`, edita no
  lugar, mesmo que outros itens do mesmo ADR já tenham. Hoje, sem código nenhum no repositório,
  isso vale para todos os ADRs.
- **Aceito, com código na `main` implementando a decisão alterada:** o ADR **não é editado**. A
  mudança vira um **ADR novo**, com o campo **Substitui** apontando o ADR (ou o item `Dn`) antigo,
  e o antigo passa a `Status: Substituído pelo ADR-MMMM`. O histórico fica legível sem `git log`.
- **O que conta como código:** código de aplicação na `main` que executa a decisão. Mockup e
  protótipo de tela (Fase 1) não contam.
- Correção que não muda a decisão (texto desatualizado, typo, link quebrado) pode ser feita no
  lugar em qualquer fase, sem linha em Revisões.

O índice com o resumo de cada decisão vive em [`../README.md`](../README.md#adrs).

- **0001–0010** — aceitos em 2026-11-22, na Fase 0.
- **0011** — reservado para **stack e persistência** (framework, banco, provedor de
  autenticação), ainda em aberto. Ver
  [`../escopo-v1.md`](../escopo-v1.md#em-aberto).

A numeração deste projeto é **independente** da do catálogo. Referência a "ADR-0009" sem
qualificação, dentro deste repo, é a de **autorização** — não a migração para Next.js, que é
o ADR-0009 **do catálogo**.
