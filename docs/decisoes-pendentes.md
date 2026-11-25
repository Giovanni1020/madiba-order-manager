# Decisões pendentes · Giovanni

> **Doc vivo e lista única** do que falta o Giovanni decidir. Reúne os temas que estavam no "Em
> aberto" do [escopo](escopo-v1.md#em-aberto) e o que a revisão fria de 25/11/2026 levantou (o
> `/revisar-doc` nos 10 ADRs e nos docs, seguido de uma avaliação geral do projeto). Cada item diz o
> que está ambíguo, por que importa, as opções e uma recomendação. Responder é escolher uma opção
> (ou escrever outra) no campo _Resposta_, com a data. Quando a resposta mudar um ADR ou o escopo, o
> PR que aplica a mudança cita o número do item.
>
> Os itens estão agrupados pela **fase do [escopo](escopo-v1.md#fases) que eles travam**: o
> primeiro bloco precisa de resposta antes do mockup; o último pode esperar.
>
> Perguntas para a loja (Bruna e Edson) ficam em [`perguntas-de-negocio.md`](perguntas-de-negocio.md);
> aqui, quando um item depende de uma delas, ele cita "pergunta N".
>
> Quando o board do GitHub Projects existir, cada item aberto vira uma issue e este doc fica como
> histórico.
>
> Status: ⏳ aguardando decisão · ✅ decidido

## Já decidido (não precisa de resposta)

- **O repositório fica privado** (Filipe, 25/11). Consequência no plano Free do GitHub: não há
  proteção de branch, e as regras de branch dependem de disciplina (item 28).
- **A matriz de permissões é obrigatória na v1.0** (Filipe, 25/11). Pode ficar fora do mockup. O
  conteúdo está no item 14.

## Para não travar o mockup

A avaliação geral apontou que há muita decisão aberta para um app de menos de 10 usuários e
nenhuma linha de código. Só os itens 1 a 12 travam o mockup. **Proposta:** o que não tiver resposta
quando o mockup começar entra nele com a recomendação, como hipótese; o mockup existe justamente
para testar essas hipóteses com a equipe. Os outros blocos esperam a fase deles.

## Antes da Fase 1 (mockup)

O mockup precisa ser "honesto com os números" ([escopo](escopo-v1.md#fase-1--condições-do-mockup)).

**1. ⏳ Em que unidade o peso é medido?**

_Onde:_ [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md) Contexto, D1 e D3.

_O problema:_ o D1 manda guardar o peso como inteiro e diz que a unidade "é a que a loja já
pratica", e que o app não a redefine. Três coisas não fecham com isso: (1) a loja mandou uma tabela
de tempos **em minutos**
([`discussoes/2026-11-20`](discussoes/2026-11-20-conversa-sobre-o-app.md#insumo-recebido-tempo-de-preparo-2511));
(2) na mesma conversa a capacidade apareceu **em pontos** ("de 100 para 80"); (3) o ADR diz que o
solitário leva ~2 minutos, e a tabela da loja diz 5. O ADR também não diz quanto pesam os
adicionais (balão, cartão, chocolates).

_Depende de:_ perguntas 5 e 8.

_Opções:_ (a) peso = minutos de montagem; (b) a unidade que a loja já usa no preço, se for outra.

_Recomendação:_ se a loja já mede em minutos, (a), e o D1 continua verdadeiro. Se usa outra
unidade, o Giovanni escolhe entre seguir o D1 (usar a da loja) ou revisar o D1 para minutos, que a
equipe entende sem tradução. Nos dois casos: o peso do pedido é a montagem mais a soma dos
adicionais, a tabela da loja vira o peso de partida, e o ADR passa a usar o tempo do solitário que
a loja informou.

_Resposta:_

**2. ⏳ A situação de prazo de cada pedido tem 3 ou 4 valores, e como se chama?**

_Onde:_ [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D4 e
[ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D1/D5 (era bloqueante na revisão).

_O problema:_ é a cor do cartão do pedido na tela. O 0002 define **3 faixas** pela folga
(confortável: *adiantado*; perto de zero: *atrasando*; negativa: *atrasado*). O 0003 lista
**4 valores** para o mesmo eixo, com um *no tempo* que nenhuma faixa produz. O nome também diverge:
"Urgência" no 0002 e no 0007, "Tempo" no 0003. E "confortável" e "perto de zero" não têm número.

_Opções:_ (a) 3 valores; (b) 4 valores, definindo a faixa do *no tempo*.

_Recomendação:_ (a), com o eixo chamado **Tempo** e os valores *no tempo* · *atrasando* ·
*atrasado* (verde, amarelo, vermelho). Separar "adiantado" de "no tempo" não leva a nenhuma ação
diferente na loja. "Urgência" sugere prioridade manual, que o ADR-0002 rejeita. Cortes para
começar, com peso em minutos (item 1): *atrasando* quando a folga fica abaixo de 15 min; *atrasado*
quando fica negativa. O número é chute, para calibrar nas duas primeiras semanas.

_Resposta:_

**3. ⏳ O prazo de um pedido "das 14h às 15h" é 14h ou 15h?**

_Onde:_ [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D1 a D3.

_O problema:_ a conta de viabilidade soma a capacidade das horas até o prazo
(`Σ limite(h), h ≤ T`), mas não diz se a hora T entra, nem se o prazo é o início ou o fim da
janela. A diferença é uma hora inteira de capacidade, na conta que mais importa. O catálogo rotula
a janela pelo início ("14h às 15h" vira 14).

_Opções:_ (a) prazo = início da janela (o buquê fica pronto quando a janela abre); (b) prazo = fim
da janela.

_Recomendação:_ (a). É o lado seguro, é como o catálogo já rotula, e na retirada a cliente pode
chegar às 14h. Conta: a capacidade até o prazo T é a soma das horas que **terminam** até T (para
prazo 14h, conta até a hora das 13h às 14h).

_Consequência a aceitar:_ um pedido da primeira janela do dia (9h) não tem nenhuma hora antes do
prazo e sai sempre inviável. Isso é verdade na loja: ele precisa ser montado no dia anterior ou
antes de abrir (pergunta 11). O app trata isso deixando o admin abrir capacidade numa hora antes da
abertura (por exemplo, das 8h às 9h) nos dias em que a loja faz isso, com a mesma operação do item 5.
Se a entrega precisar sair antes do início da janela (tempo de deslocamento), é ajuste para depois
do v1.

_Resposta:_

**4. ⏳ A folga é calculada por janela ou por pedido? E as horas que já passaram contam?**

_Onde:_ [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D3, D4 e D6.

_O problema:_ o D6 mostra uma "folga acumulada" **por hora** na tela do dia, e o eixo Tempo do
ADR-0003 é **por pedido**. Os dois não dizem como um vira o outro. E às 13h, perguntando "cabe um
pedido para as 16h?", a conta de hoje soma também a capacidade das 9h às 13h, que já passou.

_Opções:_ (a) folga por janela, e cada pedido herda a folga da janela dele; (b) folga calculada
pedido a pedido, na ordem da fila.

_Recomendação:_ (a), que é a mesma conta da tela do dia e não precisa de uma segunda. A capacidade
disponível começa **agora**: horas passadas não contam, e a hora corrente conta só o que resta
dela.

_Resposta:_

**5. ⏳ Reduzir a capacidade de uma hora é a mesma operação que aumentar?**

_Onde:_ [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md) D5 e Consequências.

_O problema:_ o D5 diz que a gestão "ajusta" a capacidade, mas o texto só fala em expandir. Na
conversa com a loja ficou decidido que, quando falta alguém, a capacidade cai e **um admin escolhe
quais pedidos mover**
([`discussoes/2026-11-20`](discussoes/2026-11-20-conversa-sobre-o-app.md#decisões)).

_Recomendação:_ uma operação só, "ajustar capacidade da hora", auditada. Quando o novo valor fica
abaixo do que já está comprometido, a tela mostra quais pedidos deixaram de caber, e o admin move.
O app não move sozinho.

_Resposta:_

**6. ⏳ Quando o app avisa que uma hora está "chegando no limite"?**

_Onde:_ [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md) D6.

_O problema:_ o primeiro aviso não tem número.

_Recomendação:_ ao passar de 80% da capacidade da hora. Também é chute para calibrar.

_Resposta:_

**7. ⏳ O desenho aguenta um dia de pico de ~400 pedidos?**

_Onde:_ [ADR-0005](adr/0005-identidade-do-pedido.md) D2,
[ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D6,
[`discussoes/2026-11-20`](discussoes/2026-11-20-conversa-sobre-o-app.md#o-problema-que-ele-resolve).

_O problema:_ a loja contou cerca de 400 pedidos no último Dia dos Namorados. Três coisas não foram
desenhadas para isso: (1) o ADR-0005 promete que o `NN` do código fica em um ou dois dígitos, e
400 pedidos num dia passam de 99; (2) a tela do dia teria ~40 cartões por hora num celular;
(3) o cadastro manual é feito só por três pessoas (Bruna, Edson e Giovanni).

_Depende de:_ perguntas 10 e 12.

_Recomendação:_ (1) o `NN` cresce para três dígitos quando passa de 99, sem zero à esquerda fixo;
(2) a tela do dia, no pico, mostra cartões compactos e recolhe as horas já concluídas; (3) no pico,
quem atende também cadastra (o que pesa no item 14). O dia ruim do mockup (Dia das Mães) usa esse
número.

_Resposta:_

**8. ⏳ O ADR-0008 precisa dizer que cadastro e edição funcionam no computador**

_Onde:_ [ADR-0008](adr/0008-pwa-online-only.md) Consequências,
[`discussoes/2026-11-20`](discussoes/2026-11-20-conversa-sobre-o-app.md#decisões).

_O problema:_ a decisão de 20/11 é que as telas de cadastro e edição **têm que funcionar no
computador da loja**. O ADR-0008 diz que desktop "não é proibido, só não é prioridade de layout".

_Recomendação:_ atualizar as Consequências do ADR-0008: continua mobile-first, mas cadastro,
edição e tela do dia têm layout de computador. O mockup desenha as duas larguras para o cadastro.

_Resposta:_

**9. ⏳ Em que o mockup é feito?**

_Onde:_ [escopo](escopo-v1.md#fase-1--condições-do-mockup).

_O problema:_ a Fase 1 diz o que o mockup mostra, mas não em que ele é feito, e a equipe vai
testá-lo no próprio celular.

_Opções:_ (a) HTML estático, com dados de exemplo fixos e sem backend; (b) Figma ou ferramenta
parecida; (c) papel.

_Recomendação:_ (a). Abre no celular de qualquer um por um link, mostra os números de verdade
(item 7) e reaproveita o que a equipe já sabe de web. O Giovanni escolhe pelo que for mais rápido
para ele.

_Resposta:_

**10. ⏳ O v1 cadastra só buquê, ou qualquer produto do site?**

_Onde:_ [escopo](escopo-v1.md#dentro-do-v1) ("cadastro manual de pedido de buquê").

_O problema:_ a tabela de tempos da loja tem box e cesta, e o site vende também pelúcia, chocolates
e caixas. Um pedido comum mistura buquê com esses itens.

_Recomendação:_ qualquer item do catálogo. Os que não têm montagem pesam zero (pergunta 2).

_Resposta:_

**11. ⏳ Qual é a capacidade de partida de cada hora?**

_Onde:_ [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md) D5; estava no "Em aberto" do
escopo ("valor base da capacidade por hora").

_O problema:_ a conta de viabilidade e a tela do dia precisam de um número por hora e por dia da
semana, e nenhum foi definido. Sem ele o mockup não consegue ser "honesto com os números".

_Depende de:_ perguntas 9, 11 e 12.

_Recomendação:_ com peso em minutos (item 1), capacidade da hora = pessoas montando naquela hora ×
60 min × 80% (o resto fica para atendimento e imprevisto), por dia da semana. Conferir contra o
histórico de demanda (pergunta 9). O 80% é chute, para calibrar.

_Resposta:_

**12. ⏳ Onde aparece um pedido que ainda não tem horário?**

_Onde:_ [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) Consequências; estava no "Em aberto" do
escopo ("pedido sem janela definida").

_O problema:_ sem prazo, o pedido não entra na conta de viabilidade, e a tela do dia é organizada
hora a hora. Não está decidido onde ele fica nem se ocupa capacidade.

_Depende de:_ pergunta 21.

_Opções:_ (a) uma faixa "sem horário" no topo do dia, fora da conta, com flag de atenção; (b) o
cadastro exige horário; (c) entra no fim do dia como prazo provisório.

_Recomendação:_ (a). Fica visível sem distorcer a conta; quando o horário for combinado, o admin
edita a janela e o pedido entra na fila.

_Resposta:_

## Antes da Fase 2 (acesso)

**13. ⏳ Qual é a stack e onde os dados ficam (ADR-0011)?**

_Onde:_ [ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md),
[ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) D1; estava no "Em aberto" do escopo ("stack e
persistência"). O número ADR-0011 já está reservado.

_O problema:_ framework, banco e provedor de autenticação. O app precisa de backend, diferente do
catálogo, que é estático. A Fase 2 (login e `pode()` no servidor) não começa sem isso. O domínio
não depende dessa escolha (ADR-0007), e o mockup em HTML estático (item 9) também não.

_Recomendação:_ decidir no ADR-0011 depois do mockup, com `/revisar-doc` antes do PR. Critérios:
plano gratuito que caiba no tamanho da loja; autenticação pronta, não escrita à mão (ADR-0009 D1);
transação no banco, que o código `DDMM-NN` (ADR-0005) e a gravação da alteração junto com o log
(item 24) exigem. Candidato óbvio para avaliar: Next.js, que o catálogo já está adotando, com
Postgres e autenticação gerenciados. A escolha é do ADR.

_Resposta:_

**14. ⏳ Quais são os papéis e o que cada um pode fazer?**

_Onde:_ [ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) D3 e D5,
[ADR-0004](adr/0004-registro-e-auditoria.md) D6, [ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md)
D5, [escopo](escopo-v1.md#dentro-do-v1) (era bloqueante na revisão).

_O problema:_ o ADR-0009 adia a matriz de permissões, mas o ADR-0004 D6 já decide quem vê o log
(admin vê tudo, "funcionário" vê os últimos dias), o ADR-0001 fala em "gerente", e o escopo diz
"dois papéis". São três nomes e duas contagens.

_Posição do Filipe:_ a matriz é obrigatória na v1.0 (ver "Já decidido"). Proposta de nomes:
**Administrador**, **Produtor** e **Visualizador**. Os nomes e o conteúdo ficam com o Giovanni.

_Proposta de conteúdo, para o Giovanni ajustar:_

| Ação | Administrador | Produtor | Visualizador |
|---|---|---|---|
| Ver a fila e a tela do dia | sim | sim | sim |
| Simular se um pedido cabe, sem cadastrar | sim | sim | sim |
| Avançar o pedido de etapa | sim | sim | não |
| Marcar atenção | sim | sim | não |
| Cadastrar e editar pedido | sim | ? | não |
| Marcar pagamento | sim | ? | não |
| Cancelar pedido | sim | não | não |
| Mover pedido de horário | sim | não | não |
| Ajustar capacidade e tabela de pesos | sim | não | não |
| Ver telefone e endereço da cliente | sim | ? | não |
| Ver o log | completo | últimos dias | não |
| Gerenciar usuários | sim | não | não |

_As dúvidas principais:_
- **Simular encaixe** é o objetivo do app: o atendente saber se pode aceitar mais um pedido e para
  qual horário. Quem atende e não é admin cairia em Visualizador, então o Visualizador também
  precisa simular. A simulação só calcula; não grava nada.
- **O Produtor cadastra e edita pedido**, ou só avança etapa? Na conversa de 20/11, quem cadastra e
  edita são a Bruna, o Edson e o Giovanni, e quem produz só avança. No pico, porém, pode faltar
  gente para cadastrar (item 7).
- **Telefone e endereço da cliente:** o Produtor precisa ver? Quem entrega precisa do endereço; quem
  só monta, não.

_Resposta:_

## Antes da Fase 3 (pedido)

**15. ⏳ Retenção de dados (LGPD) contra log que nunca apaga**

_Já decidido (ADR-0004 D6):_ o admin vê o histórico completo; os outros papéis veem uma janela
recente, da ordem de dias ou semana.

_Pendente com o advogado, antes da Fase 3:_ o recorte exato dessa janela e o prazo de retenção. A
análise inicial está na [nota sobre LGPD do escopo](escopo-v1.md#nota-sobre-lgpd). A revisão
acrescentou o conflito abaixo.

_Onde:_ [ADR-0004](adr/0004-registro-e-auditoria.md) D1, D3, D5 e D6 (era bloqueante na revisão).

_O problema:_ o D1 diz que a tabela de eventos nunca recebe UPDATE nem DELETE. Mas qualquer prazo
de retenção menor que "para sempre" obriga a apagar ou anonimizar um dia. E como o D5 audita cada
alteração de campo, **nome, telefone e endereço da cliente vão parar nos eventos**: uma segunda
cópia do dado pessoal, que não some quando o pedido for apagado.

_Opções:_ (a) evento não guarda dado pessoal: registra "alterou o endereço", sem o endereço; o dado
pessoal mora só no pedido, que pode ser apagado ou anonimizado no prazo, e o log continua sem
DELETE; (b) o log guarda tudo, e uma exceção explícita ao D1 anonimiza os eventos vencidos.

_Recomendação:_ (a), porque mantém o "nunca" do D1 verdadeiro. O prazo continua com o advogado.

_Resposta:_

**16. ⏳ O preço congelado é o combinado na venda ou o do catálogo no momento do cadastro?**

_Onde:_ [ADR-0006](adr/0006-congelamento-de-preco-e-peso.md) D1,
[`modelo-de-pedido.md`](modelo-de-pedido.md).

_O problema:_ o Contexto do ADR-0006 diz "momento da venda", e o D1 diz "momento da inserção". O
pedido fechado no último dia de uma promoção e cadastrado na manhã seguinte sai sem a promoção que
a cliente pagou. O D1 também não diz se os adicionais entram no preço congelado, nem se o total é
antes ou depois do desconto.

_Depende de:_ perguntas 17 e 18.

_Recomendação:_ vale o **preço combinado com a cliente**: o app sugere o preço do catálogo e quem
cadastra confirma ou corrige (com a mensagem do site colada, o preço já vem dela). Congela o preço
**por linha, com os adicionais**. Desconto é campo próprio, também congelado, e o total é depois do
desconto.

_Resposta:_

**17. ⏳ Onde entra a taxa de entrega?**

_Onde:_ [`modelo-de-pedido.md`](modelo-de-pedido.md), [ADR-0006](adr/0006-congelamento-de-preco-e-peso.md).

_O problema:_ a loja cobra taxa de entrega, mas ela não existe no modelo de pedido do catálogo nem
em nenhum ADR. O item 16 também não diz se ela entra no total.

_Depende de:_ pergunta 20.

_Recomendação:_ campo próprio no pedido, em centavos, só quando é entrega; fora do peso; somado ao
total depois do desconto; congelado como o preço.

_Resposta:_

**18. ⏳ Como funciona editar um pedido já cadastrado?**

_Onde:_ [ADR-0006](adr/0006-congelamento-de-preco-e-peso.md) D1,
[ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) D5; estava no "Em aberto" do escopo
("imutabilidade após estado terminal").

_Já decidido em 20/11:_ o pedido pode ser editado depois de cadastrado, e depois de entregue ou
retirado, não.

_O que falta:_ (1) item adicionado na edição pega o preço e o peso de quando entra, ou o pedido
inteiro é recalculado (o que reescreveria o passado que o ADR-0006 congela)? (2) Cancelado também
trava? O ADR-0003 diz que cancelado é terminal, e o ADR-0009 só cita entregue e retirado. (3) A
edição passa por `proximoEstado`, como o ADR-0009 sugere, ou por outra função? `proximoEstado` é a
função de **transição de estado**, e editar o cartão não é transição.

_Recomendação:_ (1) itens existentes mantêm o valor congelado e o item novo congela quando entra;
(2) entregue, retirado e cancelado travam todos os campos; (3) uma função de domínio própria para
edição, com a mesma checagem de estado terminal.

_Resposta:_

**19. ⏳ O `DDMM` do código do pedido é a data do cadastro ou da entrega?**

_Onde:_ [ADR-0005](adr/0005-identidade-do-pedido.md) D2 e D3.

_O problema:_ se for a data da entrega, reagendar muda um código que já circulou no WhatsApp. Se for
a do cadastro, "2211-03" pode ser lido como "o da entrega do dia 22". O ADR também não diz qual
fuso marca a virada do contador. A largura do `NN` está no item 7.

_Depende de:_ pergunta 19.

_Recomendação:_ data do **cadastro**, porque o código nunca muda. Fuso `America/Sao_Paulo`, virada
à meia-noite.

_Resposta:_

**20. ⏳ O que acontece quando uma entrega falha?**

_Onde:_ [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D2 e D6.

_O problema:_ na máquina de estados, *em rota* só vai para *entregue* ou *cancelado*. Entrega em
que ninguém recebe, ou com endereço errado, não tem caminho de volta sem cancelar o pedido.

_Depende de:_ pergunta 16.

_Opções:_ (a) *em rota* pode voltar para *pronto*, para nova tentativa; (b) reagendar muda a
janela e o pedido volta para a fila; (c) só cancelar e abrir outro.

_Recomendação:_ (a), com flag de atenção e evento no log. Reagendar, se a loja fizer, é uma edição
de janela.

_Resposta:_

**21. ⏳ Como o log liga um pedido cancelado ao pedido que o "reabriu"?**

_Onde:_ [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) Consequências,
[ADR-0004](adr/0004-registro-e-auditoria.md) D5.

_O problema:_ o ADR-0003 diz "reabrir é pedido novo, e o log liga os dois", mas o ADR-0004 não tem
esse evento.

_Recomendação:_ um evento nos dois pedidos: "reaberto como `<código novo>`" no antigo e "reabertura
de `<código antigo>`" no novo.

_Resposta:_

**22. ⏳ O eixo "Pagamento" colide com o campo `pagamento` herdado do catálogo**

_Onde:_ [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D1,
[`modelo-de-pedido.md`](modelo-de-pedido.md).

_O problema:_ no catálogo, `pagamento` é o **meio** (`"pix" | "link"`). O ADR-0003 cria um segundo
"pagamento", que é a **situação** (pago, sinal, a pagar).

_Recomendação:_ dois nomes: `formaPagamento` e `situacaoPagamento`.

_Resposta:_

**23. ⏳ A flag de atenção tem motivo em texto livre ou lista fechada?**

_Onde:_ [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md) D1 e D4.

_O problema:_ o D4 diz "flag com motivo" sem dizer o formato. O mesmo valor aparece com dois nomes:
"precisa de intervenção" (D1) e "precisa de atenção" (Consequências).

_Recomendação:_ lista curta de motivos mais "outro" com texto; o motivo entra no evento do log; um
nome só, "precisa de atenção".

_Resposta:_

**24. ⏳ A alteração e o evento do log são gravados juntos? Em quais entidades vale o `version`?**

_Onde:_ [ADR-0010](adr/0010-deteccao-de-conflito-por-versao.md) D1 e D3,
[ADR-0004](adr/0004-registro-e-auditoria.md) D5.

_O problema:_ o D3 do ADR-0010 justifica a detecção dizendo que, **sem ela**, o log registraria as
duas edições como se ambas tivessem valido. Mas nenhum ADR diz que o evento só é gravado quando a
alteração realmente aconteceu, nem que alteração e evento vão na mesma transação. Sem isso, nem com
a detecção o log fica confiável. O ADR também não diz se o `version` vale para a capacidade da hora
(dois admins ajustando a mesma hora).

_Recomendação:_ alteração e evento na mesma transação, e evento só quando a alteração pegou. O
`version` vale para pedido, capacidade da hora e usuários.

_Resposta:_

**25. ⏳ Reenvio depois de queda: idempotência, `version` e rascunho**

_Onde:_ [ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) D4,
[ADR-0008](adr/0008-pwa-online-only.md) D3, [ADR-0010](adr/0010-deteccao-de-conflito-por-versao.md).

_O problema:_ três decisões boas que, juntas, têm buracos:
- o ADR-0008 manda o formulário tentar de novo após queda curta, mas não amarra o reenvio à chave de
  idempotência do ADR-0007; um reenvio pode duplicar o pedido;
- se a primeira edição chegou e só a resposta se perdeu, o reenvio vai com o `version` antigo e a
  pessoa recebe "alguém alterou" por causa da própria edição;
- chave repetida com conteúdo diferente: o ADR-0007 não diz se devolve o original, rejeita ou marca
  atenção. No cadastro manual, também não diz quem gera a chave;
- entre a queda e o reenvio, o rascunho fica onde? O cadastro real é copiar a mensagem do WhatsApp
  e voltar ao app, e o celular costuma descartar o app que fica em segundo plano.

_Recomendação:_ a chave nasce quando o formulário abre e é reusada em todo reenvio. Chave repetida
com o mesmo conteúdo devolve o resultado original (pedido ou edição já aplicada), sem acusar
conflito. Chave repetida com conteúdo diferente é rejeitada. O rascunho fica guardado no navegador
do aparelho até o envio dar certo, e é apagado depois. Não é o offline-first que o ADR-0008
rejeita: o app continua sem funcionar sem conexão, só não perde o que já foi digitado. O catálogo
já guarda o carrinho assim ([`contexto-herdado.md`](contexto-herdado.md), ADR-0002 do catálogo).

_Resposta:_

**26. ⏳ `criarPedido` é função pura ou borda?**

_Onde:_ [ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) D1 e D3.

_O problema:_ `criarPedido` grava, consulta a chave de idempotência e gera UUID e código. Nada disso
é puro, e o ADR não diz em que camada ela fica.

_Recomendação:_ borda, que chama as funções puras do domínio.

_Resposta:_

## Antes da Fase 4 (capacidade)

**27. ⏳ A função que calcula a cor do pedido não recebe o que precisa**

_Onde:_ [ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) D2 (era bloqueante na revisão).

_O problema:_ a assinatura escrita é `classificarUrgencia(pedido, agora, tz)`: um pedido e a hora.
Mas a folga depende da capacidade que sobra até o prazo, que é dividida com **todos os pedidos que
vêm antes na fila**. Olhando só o próprio pedido, a função não tem como saber se ele vai atrasar. É
como perguntar "esse buquê sai a tempo?" olhando só a comanda dele, sem ver a parede.

_Opções:_ (a) a função recebe também a capacidade e a lista de pedidos; (b) recebe a folga da
janela, já calculada pela verificação de viabilidade.

_Recomendação:_ (b), que sai direto da resposta do item 4. O nome segue o item 2.

_Resposta:_

## Antes da Fase 5 (produção e uso na loja)

**28. ⏳ Como se promove para `production`, e como é um hotfix?**

_Onde:_ [`branches-e-deploy.md`](branches-e-deploy.md).

_O problema:_ (1) o passo a passo promove a `main` local sem antes atualizá-la, e uma `main` local
velha publica código antigo; (2) não diz o que conta como "permissão explícita", nem quem executa;
(3) correção urgente com trabalho não testado na `main`: espera ou entra junto? (4) Com o repo
privado (já decidido) no plano Free do GitHub, não há proteção de branch: a regra depende só de
disciplina.

_Recomendação:_ (1) `git fetch` e `git pull --ff-only` da `main` antes de promover; (2) permissão é
mensagem escrita do Giovanni, e quem executa é ele; (3) hotfix sai de `production` numa branch
`hotfix/…`, volta para `production` e para a `main`; (4) registrar no doc que é disciplina.

_Resposta:_

**29. ⏳ Que sinal vai mostrar, nas duas primeiras semanas, que o modelo de peso falhou?**

_Onde:_ [escopo](escopo-v1.md#critério-do-v1).

_O problema:_ o critério do v1 é descobrir em cerca de duas semanas se o modelo de peso funciona,
mas relatórios e métricas ficaram para a v1.1.

_Recomendação:_ usar o que o log já grava: comparar, por pedido, a hora em que ele devia ficar
pronto com a hora real em que foi marcado *pronto*. Uma consulta simples basta; não precisa tela.

_Resposta:_

## Pode esperar

**30. ⏳ A ordem por prazo (EDF) continua certa com várias pessoas montando ao mesmo tempo?**

_Onde:_ [ADR-0002](adr/0002-fila-por-prazo-e-orcamento.md) D1.
A garantia matemática do EDF vale para uma fila única. Com várias pessoas em paralelo ela não vale
em geral, e o "não precisa de prioridade manual" se apoia nela.

_Recomendação:_ declarar no ADR que é uma hipótese, e ver na prática.

_Resposta:_

**31. ⏳ Quando o site chamar `criarPedido`, quem é o usuário?**

_Onde:_ [ADR-0009](adr/0009-autorizacao-em-ponto-unico.md) D2 e D6,
[ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md) D3.
`pode(usuario, acao, recurso)` e o "ator identificado" do log só falam de pessoa.

_Recomendação:_ um usuário de serviço próprio do site, com uma única permissão (criar pedido).
Decide quando a integração entrar.

_Resposta:_

**32. ⏳ Quando o catálogo passar a guardar pedidos, quem é a fonte da verdade?**

_Onde:_ [`contexto-herdado.md`](contexto-herdado.md). O roadmap do catálogo (v1.5) prevê pedido
guardado e aviso ao lojista.

_Recomendação:_ este app. O catálogo envia pelo `criarPedido` e não guarda uma segunda cópia.

_Resposta:_

**33. ⏳ Ajustes de texto que a revisão sugeriu e não foram aplicados**

Nenhum muda decisão; ficaram de fora para não mexer nos ADRs sem o Giovanni ver.
- Os ADRs 0001, 0002, 0007, 0008, 0009 e 0010 não listam as alternativas no Contexto (o modelo novo
  pede isso). Preencher agora ou só nos próximos?
- ADR-0007 cita uma "seção Futuro" do escopo que não existe (é "Fora de escopo").
- ADR-0006 fala em "duas colunas copiadas", mas o D1 copia quatro valores.
- ADR-0005: o exemplo de ordenação só mostra a virada do ano; o problema maior é `0212` ordenar
  antes de `1501`.
- ADR-0009: o link para o item jurídico do escopo não tem âncora.
- `branches-e-deploy.md` diz "vira ADR" sem o número (ADR-0011), e os prefixos de branch divergem do
  README (falta `chore/`).
- `CLAUDE.md` e `contexto-herdado.md` apontam o catálogo por um caminho do Windows de uma máquina
  específica, que não vale para outra pessoa.

_Resposta:_
