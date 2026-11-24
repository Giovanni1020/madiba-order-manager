# Documentação — Madiba Order Manager

> Pasta **canônica e sempre atualizada**. É a **fonte de verdade** do projeto.
> Em qualquer sessão/janela nova, **comece por aqui.**

> **Status:** Fase 0 (decisões) **concluída** em 2026-11-22 — ADRs 0001–0010 e
> [escopo-v1.md](escopo-v1.md) fechados. **Sem código e sem repositório git ainda.**
> Próximo passo: **Fase 1 — mockup das telas**. Stack e persistência seguem **em aberto**
> (futuro ADR-0011).

## Índice

| Doc | O que é |
|---|---|
| [como-trabalhamos.md](como-trabalhamos.md) | Papel do Claude + dinâmica de trabalho (Grill Me, PT-BR, economia de contexto, governança). **Leia antes de pedir código.** |
| [escopo-v1.md](escopo-v1.md) | **Escopo da v1**: objetivo, critério, o que está dentro/fora, as 6 fases, prazo declarado e o que segue em aberto. |
| [contexto-herdado.md](contexto-herdado.md) | O negócio (Madiba Garden), o projeto irmão, e o que este projeto herda ou não. |
| [modelo-de-pedido.md](modelo-de-pedido.md) | O formato com que o pedido nasce hoje no catálogo (tipos, centavos, adicionais) e as consequências disso aqui. |
| [branches-e-deploy.md](branches-e-deploy.md) | Estratégia de branches (feature → `main` → `production`) e a regra: **produção só com permissão humana.** |
| [trello-padrao.md](trello-padrao.md) | Padrão de card do Trello (tipos, estrutura da descrição, checklist de aceite). |

## ADRs

| ADR | Decisão |
|---|---|
| [0001](adr/0001-peso-como-unidade-de-capacidade.md) | **Peso** mede só tempo e esforço de produção; é atributo da **montagem**; o que a gestão ajusta é a **capacidade da hora**; limite **soft** (avisa, não bloqueia). |
| [0002](adr/0002-fila-por-prazo-e-orcamento.md) | Fila por **prazo** (EDF, janela de 1h); capacidade como **fluxo acumulado**; viabilidade por janela; urgência **derivada**; tela do dia **hora a hora**. |
| [0003](adr/0003-estados-do-pedido-em-eixos.md) | Estado, **pagamento**, **atenção** e **tempo** são **eixos ortogonais**, não um enum; a máquina bifurca em retirada × entrega. |
| [0004](adr/0004-registro-e-auditoria.md) | Estado atual + **auditoria append-only** com eventos **semânticos**; auditar desde o primeiro insert; tela de log. |
| [0005](adr/0005-identidade-do-pedido.md) | **UUID** interno + código humano **`DDMM-NN`** com reinício diário. |
| [0006](adr/0006-congelamento-de-preco-e-peso.md) | **Preço e peso congelados** no pedido: o pedido é registro histórico. |
| [0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) | **Domínio puro** (`agora` por parâmetro) + **porta única** `criarPedido` + idempotência + formato compatível com o catálogo. |
| [0008](adr/0008-pwa-online-only.md) | **PWA** mobile-first, **online-only**; offline-first e WebView wrapper rejeitados. |
| [0009](adr/0009-autorizacao-em-ponto-unico.md) | Autenticação **não escrita à mão**; `pode()` em **ponto único no servidor**; conteúdo da matriz deferido. |
| [0010](adr/0010-deteccao-de-conflito-por-versao.md) | **Detecção** de conflito por `version` no v1; **resolução** automática fora. |

## Decisões em aberto

Lista completa, com o que falta decidir em cada uma, em
[escopo-v1.md § Em aberto](escopo-v1.md#em-aberto): **stack e persistência** (ADR-0011),
conteúdo da matriz de permissões, imutabilidade após estado terminal, **retenção e
visibilidade (LGPD, pendente de advogado)**, pedido sem janela definida, valor base da
capacidade por hora.

## Convenções

- Toda decisão de arquitetura nova → um **ADR numerado** em `adr/`.
- Este índice é **mantido atualizado** sempre que um doc é criado ou muda de propósito.
- **Toda mudança nasce numa branch** (`feat/…`, `fix/…`, `docs/…`) → PR para `main`.
- **`production` só com permissão humana explícita** (ver [branches-e-deploy.md](branches-e-deploy.md)).
