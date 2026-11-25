# Conversa sobre o app · 20/11/2026 (decisões complementadas em 25/11)

> **Registro de discussão.** O que a família decidiu sobre o app numa conversa presencial em
> 20/11, mais as respostas do Filipe em 25/11. Fica aqui só o que vale como decisão ou insumo; a
> gravação e a transcrição **não** são versionadas. Decisão de arquitetura continua nos ADRs: este
> doc é a fonte do pedido, não o lugar da decisão técnica.
>
> Participantes: Filipe, Bruna, Edson e Giovanni.

## O objetivo, numa frase

O app é uma **fila de produção digital**: mostra a quem atende e a quem produz o que precisa sair e
em que horário, para que o atendente saiba **se pode aceitar mais um pedido e para qual horário**,
sem interromper quem está montando buquê.

O que ele **não** é: não resolve todos os problemas da loja, não controla estoque, não ensina
ninguém a trabalhar.

## O problema que ele resolve

- Para saber se cabe mais um pedido, o atendente pergunta para quem produz, que para e olha os
  papéis. Isso acontece várias vezes no mesmo dia.
- Aceitar um pedido urgente sem enxergar a fila atrasa todos os seguintes (efeito cascata).
- Numa data de pico, organizar os papéis da parede **por horário** funcionou: a produção passou a
  responder sozinha "esse só à tarde". **O app é essa parede, digital e sempre ordenada.**
- Marcação manual se perde (ex.: o marcador de "entrega" colado ao contrário).
- Picos são o caso crítico: no Dia dos Namorados foram cerca de 400 pedidos. No dia a dia a demanda
  não é alta, então o app precisa ser leve.

## Decisões

| Tema | Decisão |
|---|---|
| Ferramenta simples | O app tem que ser **prático e fácil de usar**. Próximo passo: **mockup** com telas simuladas para a equipe testar e aprovar ([`escopo-v1.md`](../escopo-v1.md), Fase 1). |
| Quem usa e quem alimenta | **Consulta:** todo mundo que precisa ver a fila, no próprio celular. **Alimenta** (cadastra, edita, define pesos e capacidade, move pedido de horário): Bruna, Edson e Giovanni. **Avança o pedido de etapa:** quem produz. |
| Papéis | **Proposta do Filipe:** três papéis, **Administrador**, **Produtor** e **Visualizador**. Os nomes e o que cada um pode ficam com o Giovanni. A matriz de permissões pode ficar fora do mockup, mas **é obrigatória na v1.0**: deixa de ser deferida, como diz hoje o [ADR-0009](../adr/0009-autorizacao-em-ponto-unico.md) D3. |
| Papel x app | A **comanda de papel continua nos primeiros meses**, em paralelo com o app, até a equipe estar treinada. |
| Capacidade | Medida em **pontos (peso)**, não em número de funcionários. Quando falta alguém, a capacidade cai (ex.: de 100 para 80) e **um admin escolhe quais pedidos mover**. O app não decide sozinho. |
| Edição de pedido | O pedido **pode ser editado** depois de registrado (cartão, bombons pedidos depois). Depois de entregue/retirado, não. |
| Desktop | É PWA, mobile-first, mas **as telas de cadastro e edição têm que funcionar no computador da loja**. |
| Produção antecipada | Só em picos (ex.: semana do Dia dos Namorados). **O modelo de capacidade não muda por isso**: pedido pronto antes sai da conta do dia; estoque pré-produzido é tratado com o admin aumentando a capacidade do dia do pico. |
| Entrada do pedido | Três canais: **site** (que não fecha pedido nem cobra: monta o carrinho e manda uma mensagem padronizada pelo WhatsApp), **WhatsApp direto** e **presencial**. Todo pedido passa pela **comanda de papel**. A inserção no app é **manual**, mas deve ser facilitada (ver Futuro). |
| Comanda | O formato atual **não muda agora**: há um lote impresso para alguns meses. |

## Insumo recebido: tempo de preparo (25/11)

| Montagem | Tempo |
|---|---|
| Buquê até 5 rosas | 25 min |
| Buquê acima de 6 rosas | 35 min |
| Buquê de lírios | 35 min |
| Solitário de rosa | 5 min |
| Box sem flor | 15 min |
| Box com flor | 20 min |

Ainda incompleta: faltam o buquê de exatamente 6 rosas, girassóis, alstroemérias, buquês de cetim
com chocolate, temáticos, o "buquê grande" e o tempo dos adicionais. As perguntas estão em
[`perguntas-de-negocio.md`](../perguntas-de-negocio.md).

## Futuro (registrado, fora do app agora)

- **Pagamento online no site:** pedido já chega pago. Ideia para o próximo ano, e é do **site**.
- **Leitura da comanda por foto (IA)** e **leitura da conversa do WhatsApp:** depois do v1, com
  conferência humana antes de salvar e decisão de LGPD sobre enviar dado de cliente a terceiro.
- **Redesenho da comanda** com campos de data de entrega, horário, entrega/retirada, pagamento e
  taxa de entrega: na próxima impressão.
- **Push notifications** para eventos da fila (pedido movido, fila reduzida).
- **Pagamento dos entregadores:** hoje anotado em caderno, com erros recorrentes. Encaixa quando
  vier a lista de entregas da v1.1.
- **Estoque e insumos:** fora do app.
- **Rotas de entrega:** fora.

## Pendente

- Completar a tabela de tempos com a Bruna.
- Histórico de demanda (média de buquês por dia, ex.: sábados dos últimos 6 meses) com o Edson,
  para a capacidade inicial.
