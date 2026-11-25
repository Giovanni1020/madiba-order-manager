# CLAUDE.md — Madiba Order Manager

O contexto, as decisões e a governança deste projeto vivem em **`docs/`** (fonte de verdade).
Este arquivo é só o **ponto de entrada** — não duplica conteúdo, para nunca sair de sincronia.

## Ao iniciar qualquer sessão neste repo

1. Leia **`docs/README.md`** — índice de toda a documentação.
2. Atue conforme o doc de trabalho, importado aqui: @docs/como-trabalhamos.md
   Ele cobre papel, dinâmica "Grill Me", PT-BR em tudo (inclusive commit e PR), economia de
   contexto e regras de governança. O `@` importa o doc, então ele entra no contexto de toda
   sessão sem depender de alguém abrir o arquivo.
3. Escopo atual, fases e o que segue em aberto: **`docs/escopo-v1.md`**.
4. Contexto do negócio e o que este projeto herda do catálogo:
   **`docs/contexto-herdado.md`**. Modelo de pedido: **`docs/modelo-de-pedido.md`**.
5. Decisões de arquitetura: **`docs/adr/`** (0001–0010 aceitos; `docs/README.md` tem o
   resumo de cada uma).

Toda decisão de arquitetura nova vira um ADR numerado em `docs/adr/`, e o índice
`docs/README.md` é mantido atualizado.

## Projeto irmão

Este repo é o **gerenciador de pedidos** da mesma loja atendida pelo
**Madiba Gift Catalog** (`C:\Users\filip\OneDrive\Área de Trabalho\gift-catalog`,
<https://github.com/Giovanni1020/madiba-gift-catalog>). O catálogo é a **origem** dos
pedidos; este projeto é o **consumo** deles. Ver `docs/contexto-herdado.md`.
