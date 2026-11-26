# Feedback do mockup da Fase 1 — rodada 1

> **Origem:** mockup navegável publicado em 2026-11-24
> ([artifact](https://claude.ai/artifact/HwJb3CLfazgyfd68r3KNhw)), com os dois dias previstos
> em [`escopo-v1.md` § Fase 1](escopo-v1.md#fase-1--condições-do-mockup): dia calmo (24/11) e
> Dia das Mães (09/05), este último com a conta de viabilidade quebrando de propósito.
> **Feedback de:** Giovanni, 2026-11-25. **Rodada interna**, antes de levar à equipe da loja.
>
> Este doc registra **o que o feedback decidiu** e **o que ele abriu**. Nenhum ADR foi
> alterado por causa dele — os itens que mexem em decisão aceita estão listados como
> pendentes de ADR próprio.

## Veredito dos oito itens

| # | Item levantado | Veredito | Consequência |
|---|---|---|---|
| 1 | Opção de white mode | **Aceito** | Alternador de tema explícito na própria tela. |
| 2 | Nomeação de botões confusa | **Aceito, e revelou um campo que falta** | Auditoria de copy + **nota por item** (ver § Nota ≠ atenção). |
| 3 | Mais campos no pedido e no cadastro | **Aceito com fronteira** | Pagamento e telefone no cadastro; "quem entregou" como texto livre (ver § Fronteira de entrega). |
| 4 | Tags mais visíveis para produção | **Aceito, com hierarquia em vez de volume** | Forma por papel, não intensidade em todas. |
| 5 | Em produção no topo da lista | **Aceito** (dentro da hora) | Ordenação por estado **dentro** da janela; ver § Por que isso não quebra o EDF. |
| 6 | Sub-lista de em produção acima das horas | **Aceito** | Bloco próprio no topo; blocos vazios não aparecem. |
| 7 | `atrasando` com amarelo mais visual | **Aceito, mas o pedido expôs um bug** | Ver § O eixo de tempo que o mockup não mostrou. |
| 8 | Pedido não entrar automaticamente como `recebido` | **Pendente de ADR** | Estado novo `em fila`; ver § Em fila. |

## O eixo de tempo que o mockup não mostrou

O item 7 pedia a tag amarela de `atrasando` mais visível. Rodando o eixo de tempo
([ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D1, derivado da folga do
[ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D4) sobre os dois dias da semente:

| Valor do eixo | Ocorrências nos dois dias |
|---|---|
| `atrasado` | 23 |
| `adiantado` | 5 |
| `no tempo` | 1 |
| **`atrasando`** | **0** |
| (não produz mais) | 8 |

**O mockup define um eixo de quatro valores e demonstra três.** A tag amarela nunca esteve na
tela. O pedido do item 7 não era de cor: era de um estado ausente.

Causa: a faixa de `atrasando` que eu adotei (`folga < limite da hora ÷ 2`) é estreita, e
nenhuma das duas sementes cai nela — o dia calmo tem folga larga em toda janela e o Dia das
Mães já está negativo. **A faixa é invenção do mockup**: o ADR-0002 D4 diz apenas "perto de
zero", sem número. Duas coisas a resolver:

1. **A semente precisa de uma janela em `atrasando`**, senão o estado continua sem revisão.
2. **A faixa precisa de um número decidido**, não herdado do mockup. Fica em aberto.

## Nota ≠ atenção

O item 2 começou como copy de botão e terminou num campo que não existe no modelo.

Hoje o único lugar da tela onde se escreve texto livre é a flag de atenção. Então o botão
"Marcar que precisa de atenção" virou, na prática, o botão de anotar qualquer coisa — e é por
isso que o nome soa vago: ele não descreve o que as pessoas fazem com ele.

São duas coisas de naturezas diferentes:

| | **Atenção** | **Nota** |
|---|---|---|
| Natureza | problema a resolver | instrução de montagem |
| ADR-0003 | D4, flag + motivo, destaca o pedido | **não existe** |
| Exemplo | "telefone não atende, confirmar endereço" | "cliente escolheu papel kraft" |
| Ciclo de vida | é resolvida e a flag limpa | acompanha o pedido até o fim |
| Na lista | tem de destacar | não deve destacar |

Papel kraft não é um problema; é como montar. Tratar as duas como a mesma coisa faz o
destaque visual do D4 perder valor, porque pedido com nota de rotina passa a parecer pedido
com problema.

**Decisão:** nota é campo próprio, **por item** do pedido, e não destaca a linha. Com ela
existindo, o botão de atenção pode ser nomeado pelo que faz.

Isso **não** é mudança de ADR: o ADR-0003 descreve os eixos de *estado*, e nota não é estado.
É um campo do item, no mesmo nível de `extras` em
[`modelo-de-pedido.md`](modelo-de-pedido.md).

## Em fila

O item 8 pede um estado que indique "o pedido não saiu nem está em produção ainda", explicitamente
**somado** a `recebido` e `em produção`, não no lugar de um deles. Leitura adotada:

```
recebido ──▶ em fila ──▶ em produção ──▶ pronto ──┬─[retirada]─▶ retirado
                                                  └─[entrega]──▶ em rota ──▶ entregue
```

- `recebido` — entrou no sistema, ninguém conferiu.
- `em fila` — conferido e aprovado para montar, aguardando a vez.
- `em produção` — sendo montado agora.

**Por que isso tem fundamento além da preferência:** no Dia das Mães do mockup, **23 dos 30
pedidos estão em `recebido`**. Um estado que contém 77% dos pedidos não informa nada a quem
abre a tela. O corte proposto separa "ninguém olhou ainda" de "está pronto para a bancada",
que são situações operacionalmente diferentes num dia de pico.

**Isto altera o [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D2, que está aceito.**
Portanto:

- **Vira ADR próprio**, com `recebido` → `em fila` como transição explícita e com a pergunta
  de quem pode fazê-la ([ADR-0009](adr/0009-autorizacao-em-ponto-unico.md)).
- **O número não é 0011**: o escopo reserva o 0011 para stack e persistência.
- O mockup vai mostrar a proposta **rotulada como proposta**, que é o jeito de a loja dizer
  se o corte faz sentido na rotina antes de a decisão ser gravada.

### Pergunta aberta deste item

Um pedido em `recebido` (não conferido) **já consome capacidade** na conta de viabilidade do
ADR-0002 D3? Hoje consome, porque tem prazo. Se `em fila` é que significa "aprovado para
montar", existe um argumento para `recebido` ficar fora da conta — e um argumento contrário,
de que o trabalho existe independente de alguém ter conferido. **A decidir no ADR**, não aqui.

## Por que o item 5 não quebra o EDF

O [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D1 ordena por prazo e diz que
prioridade manual não existe nem precisa existir. Flutuar pedidos por **estado** ao longo do
dia inteiro reintroduziria prioridade manual por outra porta.

Ordenar **dentro de uma janela de 1 hora** não tem esse efeito: todos os pedidos daquela
janela têm **o mesmo prazo**, então qualquer ordem entre eles satisfaz o EDF igualmente. O D1
não diz nada sobre desempate dentro do mesmo prazo.

Nota: a ordenação que o mockup usa hoje dentro da hora é **por peso decrescente**, que foi
invenção do mockup e não está em ADR nenhum. Passa a ser por estado.

## Fronteira de entrega

O campo "quem entregou" é aceito como **texto livre, registro retrospectivo** — para não ter
de procurar no WhatsApp quem combinou de levar o quê. Sem ele, o estado `em rota`, que
[é operado no v1](adr/0003-estados-do-pedido-em-eixos.md), não diz nada sobre quem está na rua.

**O que esse campo não pode se tornar, dentro do v1:**

- atribuição de entrega (fila do entregador, designação);
- papel de entregador com superfície própria;
- qualquer medida de capacidade de entrega.

Os três estão explicitamente [fora do v1](escopo-v1.md#fora-do-v1--vai-para-a-v11) ou
[fora de escopo](escopo-v1.md#fora-de-escopo--objetivo-distante). O campo é um registro, não
um plano. Consultar "quais entregas cada pessoa fez" é **relatório**, também fora do v1.

**Alerta de LGPD:** o item 3 aumenta os dados pessoais na tela (telefone do cliente em toda
linha, quem entregou). Isso amplia a superfície do item de retenção e visibilidade que segue
[em aberto e pendente de advogado](escopo-v1.md#em-aberto). Não bloqueia o mockup; entra na
conversa jurídica.

## Hierarquia das tags, não volume

O item 4 pede tags mais visíveis para o que está em produção. Aumentar o destaque de todas as
tags zera o ganho: se tudo grita, nada grita — e o destaque da flag de atenção
([ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D4) é o que mais perde, porque ele existe
justamente para ser notado entre os outros.

Adotado: distinção por **forma e papel**, numa escala só.

| Estado | Tratamento | Razão |
|---|---|---|
| `em produção` | preenchida, cor de destaque | é o que está acontecendo agora |
| `em fila` | preenchida, neutra | próximo da bancada |
| `recebido` | só contorno | ainda não é trabalho |
| `pronto` | contorno | esperando sair |
| `em rota` | contorno | fora da loja |
| `entregue` · `retirado` | quase invisível | pedido concluído não é informação para quem produz |

## Custo de desenho que o item 6 criou

Com a sub-lista de produção, passam a existir **três blocos acima da lista hora a hora**:
veredito de viabilidade, em produção agora, e sem horário definido. Em 375px, a primeira
janela do dia nasce abaixo da dobra — e a lista hora a hora é
[a leitura do dia](adr/0002-fila-por-prazo-e-orcamento.md) (D6), não um apêndice.

Mitigação acordada: **bloco vazio não aparece.** Num dia calmo sem pedidos sem horário e sem
nada em produção, sobra só o veredito. Fica para a próxima rodada confirmar se isso basta num
dia de pico, onde os três blocos existem ao mesmo tempo — que é exatamente quando a tela mais
precisa estar legível.

## Em aberto depois desta rodada

| Tema | O que falta decidir |
|---|---|
| **Estado `em fila`** | ADR próprio (não 0011). Transição, quem pode fazê-la, e se `recebido` entra na conta de viabilidade. |
| **Faixa de `atrasando`** | O número que separa `no tempo` de `atrasando`. O ADR-0002 D4 só diz "perto de zero". |
| **Semente do mockup** | Precisa de uma janela em `atrasando`, senão o estado segue sem revisão da loja. |
| **Três blocos no topo em dia de pico** | Se colapsar vazios basta, ou se o veredito precisa encolher. |
| **LGPD** | Telefone e "quem entregou" ampliam os dados em tela; entra na conversa com o advogado. |

Itens que já estavam em aberto no [escopo](escopo-v1.md#em-aberto) e que esta rodada **não**
fechou: valor base da capacidade por hora, matriz de permissões, imutabilidade após estado
terminal, stack e persistência.

Itens que esta rodada **fechou**: onde aparece o pedido sem janela definida (bandeja fixa no
topo, peso somado à parte, fora da conta de viabilidade) — aprovado no mockup e não contestado
neste feedback.
