# Branches e Deploy — Madiba Order Manager

> Como o código flui de uma mudança até a produção.
> **Regra de ouro:** mudanças só vão para `production` **com permissão humana explícita.**
>
> **Herdado do Madiba Gift Catalog.** A janela excepcional da branch de integração `next`
> (migração CRA → Next.js) é **específica daquele repo** e **não** se aplica aqui.

## Branches

| Branch | Papel |
|---|---|
| `feat/…`, `fix/…`, `chore/…`, `docs/…` | **Toda** mudança nasce aqui. Nunca se commita direto em `main` nem em `production`. |
| `main` | Integração / desenvolvimento. Recebe as branches via PR quando prontas. Ambiente de **teste** (preview). |
| `production` | O que está **no ar**. Só recebe da `main`, depois de testada, e **com permissão humana**. |

## Fluxo

```
feature branch ──(PR, quando pronta)──▶ main ──(testada + permissão humana)──▶ production
```

1. **Mudança** → branch separada (`feat/…`, `fix/…`, `chore/…`, `docs/…`).
2. Pronta → **PR para `main`** → merge.
3. `main` é **testada** (preview).
4. Aprovada **por uma pessoa** → **promove `main` → `production` por fast-forward** → push → deploy.

```sh
git checkout production
git merge --ff-only main      # falha se não for fast-forward (protege contra commits soltos)
git push origin production    # dispara o deploy de produção
git checkout main             # working tree volta sempre para a main
```

> **Working tree após uma feature branch:** depois do commit + push da branch, o working tree
> **permanece na própria branch**. O retorno para a `main` acontece só **após o humano
> confirmar o merge**.

## Regra de produção (inegociável)

**Nenhuma mudança vai para `production` sem permissão humana explícita.**

- O agente (Claude) **nunca** faz merge/push em `production` por conta própria — nem "para
  adiantar", nem porque os testes passaram.
- A promoção `main → production` só acontece quando uma pessoa **pede explicitamente**.
- Na dúvida, o padrão é **não promover**.

## Deploy

**A definir** quando a stack for decidida (vira ADR). O que já está fixado é a *regra* acima,
independente de plataforma. No catálogo a hospedagem é Vercel, com
**Production Branch = `production`**; se a escolha aqui for a mesma, repetir essa
configuração e **voltar o working tree para `main`** após publicar.
