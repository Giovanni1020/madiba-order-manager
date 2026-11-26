# Escopo da v1 — Madiba Order Manager

> **Status:** escopo fechado nas decisões listadas em [`README.md`](README.md); itens em aberto
> na seção final. Nenhuma linha de código escrita — **Fase 1 (mockup) é o próximo passo.**
> Fechado em 2026-11-22.

## Objetivo do produto

Facilitar o manuseio dos pedidos de buquê: registrar o pedido, enxergar a fila de produção e
saber se ainda cabe mais trabalho no dia. App **interno**, mobile, para a equipe da loja.

Integração com o site é **potencial futuro**, não objetivo do v1 — mas a porta por onde o
pedido entra já é construída pensando nela
([ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md)).

## Critério do v1

> O v1 é a **menor coisa** que permite um pedido **real** ir do cadastro até entregue sem
> ninguém sair do app.

O v1 não existe para entregar funcionalidade: existe para descobrir se o **modelo de peso por
hora funciona na rotina da loja** ([ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md),
[ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md)). Toda feature somada antes da primeira
semana de uso real adia a única informação capaz de invalidar o desenho. Se peso por janela
não refletir a prática, é melhor saber em duas semanas que em três meses.

## Dentro do v1

| Item | Decisão que governa |
|---|---|
| Login invite-only, contas criadas por admin | [ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) |
| Dois papéis, com `pode()` verificado no servidor | [ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) |
| Cadastro manual de pedido de buquê | [ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) |
| Preço e peso congelados no pedido | [ADR-0006](adr/0006-congelamento-de-preco-e-peso.md) |
| Código humano `DDMM-NN` + UUID interno | [ADR-0005](adr/0005-identidade-do-pedido.md) |
| 5 estados + eixo de pagamento manual + flag de atenção | [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) |
| Tela do dia, hora a hora, ordenada por prazo, com folga visível | [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) |
| Verificação de viabilidade ao aceitar pedido (aviso, não bloqueio) | [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) · [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md) |
| Limite de capacidade por hora, ajustável pelo gerente, auditado | [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md) |
| Log de eventos com tela (completo para admin) | [ADR-0004](adr/0004-registro-e-auditoria.md) |
| Detecção de conflito por `version` | [ADR-0010](adr/0010-deteccao-de-conflito-por-versao.md) |
| PWA instalável, online-only, mobile-first | [ADR-0008](adr/0008-pwa-online-only.md) |

## Fora do v1 — vai para a v1.1

- **Lista paralela de entregas**, com estados e tempos próprios, sincronizada com os pedidos.
- **Tela e papel do entregador** (atualizar status do celular dele). O estado `em rota` **é
  operado no v1**, marcado manualmente por quem atende — o que falta é a superfície do
  entregador.
- **Fila de comandos** / resolução automática de conflito
  ([ADR-0010](adr/0010-deteccao-de-conflito-por-versao.md) D2).
- **Relatórios e métricas.**

## Fora de escopo — objetivo distante

- **Capacidade de entrega com unidade própria** (paradas, rota, distância). Hoje a loja não
  tem como medir: nem com entrada manual, porque duas entregas próximas custam muito menos que
  duas em sentidos opostos. Peso **não serve** para isso
  ([ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md), Consequências).
- **Ingestão automática de pedidos do site.** A porta existe
  ([ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) D3), mas só é exposta depois do
  fechamento da v1.2 do catálogo — integrar com alvo em migração é a pior janela possível.

## Nunca, enquanto for app interno

- **Link de acompanhamento para a cliente.** Se algum dia existir, nasce no **site**, depois
  de ele virar e-commerce completo — não aqui.
- **Integração com provedor de pagamento.** O status de pagamento é **manual e arbitrário**
  ([ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D3).

## Fases

| Fase | O que é | Estado |
|---|---|---|
| **0 — Decisões** | ADRs 0001–0010 + este escopo. Nenhum código. | ✅ **Concluída** (itens em aberto abaixo) |
| **1 — Mockup** | Ilustração das telas para demonstração, feedback e aprovação **antes** de desenvolvimento real. | ⏭️ **Próxima** |
| **2 — Acesso** | Fatia vertical fina: login + rota protegida + papel verificado no servidor + `pode()`. Nenhum pedido ainda. | Pendente |
| **3 — Pedido** | Cadastro manual, estados, auditoria **desde o primeiro insert**, código humano, `version`. | Pendente |
| **4 — Capacidade** | Peso, limite por hora, viabilidade, tela do dia hora a hora, folga. | Pendente |
| **5 — Polimento** | PWA, estados vazios e de erro, mobile real, `main → production` com permissão humana. | Pendente |

### Fase 1 — condições do mockup

- **Não precisa ser bonito.** Precisa ser **honesto com os números**.
- Mostra **dois dias**: um dia calmo e um **Dia das Mães** — propositalmente irrealista de
  ruim, para expor como a tela se comporta quando a conta de viabilidade quebra. Mockup com
  dados confortáveis esconde exatamente o caso que importa.
- Não congela escopo: se a conta de viabilidade contradisser o desenho, **quem muda é o
  desenho**.

### Prazo declarado

O alvo é **menos de 5 dias** para a Fase 1 (mockup). O v1 completo nesse mesmo prazo é
**tensão aberta** — a lista de "Dentro do v1" tem doze itens, dois deles (capacidade e
auditoria) com lógica própria. A aferição honesta do prazo do v1 só é possível **depois** do
mockup, quando as telas estiverem definidas.

## Em aberto

| Tema | O que falta decidir |
|---|---|
| **Stack e persistência** | Framework, banco e provedor de autenticação. Precisa de backend — diferente do catálogo, que é estático. Vira **ADR-0011**. Não afeta o domínio ([ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md)). |
| **Matriz de permissões** | Quais papéis e o que cada um pode. Deferido **de propósito** ([ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) D3); o mecanismo já está decidido. |
| **Imutabilidade após estado terminal** | Quais campos travam, a partir de qual estado ([ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) D5). |
| **LGPD: retenção e visibilidade** | Admin vê histórico completo; funcionário vê janela recente (ordem de últimos dias/semana). Recorte exato e prazo de retenção **pendentes de conversa com advogado** — a decidir antes da Fase 3. |
| **Valor base da capacidade por hora** | O número de partida, por hora e por dia da semana. |
| **Estado `em fila`** | Aberto pela Fase 1: um estado entre `recebido` e `em produção`. Altera o [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D2, então **vira ADR próprio** (não o 0011). Falta decidir a transição, quem pode fazê-la e se `recebido` consome capacidade. Ver [feedback-mockup-fase1.md](feedback-mockup-fase1.md#em-fila). |
| **Faixa de `atrasando`** | Aberto pela Fase 1: o número que separa `no tempo` de `atrasando` no eixo de tempo. O [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D4 só diz "perto de zero". |

### Fechado pela Fase 1

- **Pedido sem janela definida** — aparece numa **bandeja fixa no topo** da tela do dia, com o
  peso somado à parte e marcado como fora da conta de viabilidade
  ([ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md), Consequências). Decidido no mockup e
  não contestado na primeira rodada de feedback
  ([feedback-mockup-fase1.md](feedback-mockup-fase1.md)).

### Nota sobre LGPD

O banner de consentimento do catálogo é de **rastreamento** (Meta Pixel, Google Ads):
finalidade e base legal diferentes. Guardar nome, telefone e endereço **para cumprir o
pedido** normalmente é execução de contrato, não consentimento — ou seja, o remédio
provavelmente **não** é estender o banner, e sim ter **aviso de privacidade** (o que se guarda,
por quanto tempo, quem acessa) mais política de retenção.

O argumento de que os dados já ficam em papel na loja até a entrega joga a favor da
**retenção**, não da guarda indefinida: se o papel é destruído após a entrega, um banco que
guarda para sempre é risco **novo**. Nada disso é opinião jurídica — a decisão fica com o
advogado.
