# Contexto herdado — do Madiba Gift Catalog

> Levantado em **2026-11-22** a partir do repo irmão e das sessões anteriores deste PC.
> Serve para esta janela e as próximas não precisarem redescobrir o negócio.
> **Instantâneo, não espelho:** se divergir do repo do catálogo, o catálogo é a verdade.

## O negócio

Loja de presentes **Madiba Garden**: buquês (naturais e de chocolate), doces, balões,
plaquinhas, polaroides, cartões. Site público: <https://madibagarden.com>
(deploy em <https://madiba-garden.vercel.app>).

## O projeto irmão (origem dos pedidos)

| | |
|---|---|
| Pasta | `C:\Users\filip\OneDrive\Área de Trabalho\gift-catalog` |
| Repo | <https://github.com/Giovanni1020/madiba-gift-catalog> |
| Stack | React + TypeScript, **CRA** (`react-scripts`), em **migração para Next.js App Router** (v1.2, ADR-0009) |
| Hospedagem | Vercel, Production Branch = `production` |
| Branches | `main` (CRA, base de hotfix) · `production` (no ar) · `next` (**prevista**: integração da migração, nasce no scaffold da v1.2 e só existe durante ela) |
| Board | [Madiba Gift Catalog](https://trello.com/b/BAJGQ2Xg/madiba-gift-catalog) |
| Docs | `docs/` é a fonte de verdade; 14 docs + 9 ADRs |

**Sem backend.** O catálogo é app estático: o pedido **não é persistido** — ele é montado no
checkout e enviado como **mensagem de texto pelo WhatsApp**
(`src/components/checkoutMessage.ts`). Rastreamento via Meta Pixel e Google Ads (gtag direto,
uma única conversão: o envio do carrinho pelo WhatsApp).

> Essa é a lacuna que este projeto existe para preencher: hoje o pedido só existe como texto
> no WhatsApp, sem registro, sem estado e sem histórico.

## Estado do catálogo (em 2026-11-22)

- v1.0 e v1.1 **encerradas** (v1.1 no ar).
- **v1.2 ativa**: migração CRA → Next.js + SEO.
- Roadmap declarado até v1.5.

## ADRs do catálogo que podem influenciar aqui

| ADR | Decisão | Por que importa |
|---|---|---|
| 0002 | Carrinho: Context + reducer; `sessionStorage`; `localStorage` opt-in para dados do cliente | Define o **modelo de pedido** que chega aqui |
| 0005 | Variantes de produto (preço/imagem/chocolates por opção); helper `basePrice` | Preço de um item **não** é campo fixo |
| 0006 | Promoções por tempo limitado (`promo` com preço absoluto + `expiresAt` ISO) | Preço histórico de um pedido ≠ preço atual do produto |
| 0008 | Google Ads: uma só conversão (envio pelo WhatsApp) | Marca o ponto exato em que o pedido nasce |

## O que este projeto herda, explicitamente

1. **Governança e comportamento** → [`como-trabalhamos.md`](como-trabalhamos.md).
2. **Fluxo de branches e a regra de produção** → [`branches-e-deploy.md`](branches-e-deploy.md).
3. **Formato de card do Trello** → [`trello-padrao.md`](trello-padrao.md).
4. **O modelo de pedido** → [`modelo-de-pedido.md`](modelo-de-pedido.md).
5. **A convenção de ADR numerado** em [`adr/`](adr/).

## O que este projeto NÃO herda

- A stack (CRA/Next) — **decisão aberta**, reservada para o
  [ADR-0011](adr/README.md) daqui.
- A janela da branch `next` — exceção da migração do catálogo.
- Variáveis de ambiente `REACT_APP_*` / `NEXT_PUBLIC_*` do catálogo.
- Os escopos v1/v1.1/v1.2 — são do catálogo; o escopo daqui é o
  [`escopo-v1.md`](escopo-v1.md).
