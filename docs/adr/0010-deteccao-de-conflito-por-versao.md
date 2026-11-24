# ADR-0010 — Detecção de conflito por versão; fila de comandos fora do v1

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** poucas pessoas usando ao mesmo tempo, em celular, com rede de loja.

## Contexto

Duas pessoas podem abrir o mesmo pedido e salvar alterações diferentes. A proposta inicial era
uma **fila de comandos** (serializar operações, com retry e ordenação), com a dúvida de se
valeria no v1.

A distinção que resolve a questão: **detecção** de conflito e **resolução** de conflito têm
custos muito diferentes.

## Decisão

**D1 — Detecção entra no v1: coluna `version`.** O update leva `WHERE version = ?` e
incrementa. Zero linhas afetadas ⇒ alguém alterou no meio ⇒ a tela avisa e pede recarregar.
São poucas linhas de código.

**D2 — Resolução automática (fila de comandos, merge) fica fora do v1.** Com poucas pessoas
simultâneas, não se paga. Avisar e recarregar é resposta suficiente.

**D3 — Pular detecção é inaceitável, e o motivo é a auditoria.** Sem D1, a última gravação
apaga a anterior **em silêncio** — e o log
([ADR-0004](0004-registro-e-auditoria.md)) registraria as **duas** edições como se ambas
tivessem valido. Um log que mente com autoridade é pior que não ter log.

## Consequências

- Quem usa vê, ocasionalmente, um aviso de "este pedido foi alterado, recarregue". Aceito: é
  o preço de não perder dado.
- A tela precisa conseguir recarregar sem perder o que a pessoa digitou — mesma exigência de
  [ADR-0008](0008-pwa-online-only.md) D3.
- Quando a fila de comandos for revisada (v1.1+), `version` continua valendo: ela é a base de
  qualquer resolução, não um substituto temporário.
