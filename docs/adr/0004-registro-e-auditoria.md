# ADR-0004 — Estado atual + auditoria append-only, com tela

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** requisito original do projeto ("log de alterações/adições").
  Independente de stack.

## Contexto

O log de alterações foi pedido como quinto item da lista de requisitos, mas **determina o
schema inteiro**: a escolha de como registrar histórico define se existe uma tabela de pedidos
ou nenhuma.

Duas saídas estavam na mesa:

- **Event sourcing** — o pedido é a soma dos seus eventos. Atende "log" por construção.
- **Estado + auditoria** — tabela de pedidos com o estado atual, mais tabela de eventos.

## Decisão

**D1 — Estado atual em `pedidos`; histórico em tabela de eventos append-only.** Sem `UPDATE`,
sem `DELETE`, nunca. Event sourcing foi **rejeitado**: reconstruir estado consumiria a fase
inteira de cadastro, e o requisito real é "ver quem mudou o quê", que estado + auditoria
resolve direto.

**D2 — Eventos são semânticos, não diff de coluna.** Grava-se `tipo` (enum de evento do
domínio) + `payload` + ator + timestamp. Razão: o log vai ser **lido por uma pessoa**.
`peso: 8 → 12` exige que o leitor conheça o schema e **envelhece junto com ele** — renomear
uma coluna torna o histórico antigo ilegível. O texto é renderizado na leitura, a partir do
tipo.

**D3 — Rótulos vão em snapshot no payload.** Se o log diz *"removeu o adicional Balão
Coração"* e esse adicional for descontinuado depois, a entrada continua fazendo sentido.
Referência pura pelo id não sobrevive à evolução do catálogo.

**D4 — Auditar desde o primeiro insert.** Retrofit de auditoria deixa todos os registros
anteriores **sem proveniência para sempre**. Isso significa que a tabela de eventos nasce na
mesma fase que a de pedidos, não depois.

**D5 — O que é auditado:**

- **pedido** — criação, cada alteração de campo, cada transição de estado
  ([ADR-0003](0003-estados-do-pedido-em-eixos.md)), override manual de peso
  ([ADR-0001](0001-peso-como-unidade-de-capacidade.md) D4);
- **capacidade da hora** — quem expandiu, quando, de quanto, para qual hora
  ([ADR-0001](0001-peso-como-unidade-de-capacidade.md) D5);
- **usuários e papéis** — criação, mudança de papel, desativação.

**D6 — O log tem tela dentro do app.** Log sem leitor é custo de disco. **Admin vê o
histórico completo**; funcionário vê uma **janela recente** (da ordem de últimos dias /
semana). O recorte exato e a retenção estão **em aberto, pendentes de orientação jurídica** —
ver [`escopo-v1.md`](../escopo-v1.md).

## Consequências

- Corrigir um erro no histórico é **gravar um evento novo**, nunca editar o log. Entrada
  errada permanece, com a correção ao lado.
- O log cresce sem limite. Aceito: o volume de uma loja com menos de 10 funcionários é
  pequeno, e retenção vem da decisão jurídica (D6), não de restrição técnica.
- O log é o que dá sentido às permissões — a pergunta *"quem expandiu a capacidade daquela
  hora?"* só tem resposta por causa de D5.
- Auditoria pressupõe **ator identificado em toda escrita**, o que acopla este ADR ao
  [ADR-0009](0009-autorizacao-em-ponto-unico.md).
- Conflito de escrita concorrente não pode passar como duas alterações válidas no log — ver
  [ADR-0010](0010-deteccao-de-conflito-por-versao.md).
