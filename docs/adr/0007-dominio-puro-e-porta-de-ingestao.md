# ADR-0007 — Domínio puro e porta única de ingestão

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** o requisito original pede "lógica re-utilizável para quando o site
  virar full e-commerce". Independente de stack — é justamente o ponto.

## Contexto

"Lógica re-utilizável" costuma ser cobrada cedo e entregue caro: monorepo, package
compartilhado e abstração genérica de pedido **antes de existir o segundo consumidor**. O
resultado é arquitetura para dois sistemas e nenhum dos dois funcionando.

Há um agravante de timing: o catálogo está **no meio da migração CRA → Next.js** (v1.2 lá).
Integrar com alvo em movimento é a pior janela possível.

## Decisão

**D1 — A camada de domínio é pura.** Mesma entrada, mesma saída, sempre: sem relógio, sem
banco, sem rede, sem sorteio, sem escrita. Tudo o que a função precisa chega por parâmetro.

Isso é o mecanismo de reuso mais barato que existe, porque uma função pura **não tem
dependência de onde roda** — navegador, servidor, teste e a futura integração executam a
mesma coisa. Reaproveitar não exige refatorar, porque não há nada emaranhado para
desemaranhar. É disciplina, não infraestrutura: nenhum monorepo, nenhum package, nenhum passo
de build.

**D2 — `agora` e fuso entram por parâmetro.** Quem sabe que horas são é a **borda** do
sistema; o núcleo nunca adivinha. Assinaturas iniciais:

```ts
calcularPeso(itens: Item[], tabela: TabelaPesos): number
verificarViabilidade(pedidos: Pedido[], capacidade: Capacidade): Viabilidade
classificarUrgencia(pedido: Pedido, agora: Date, tz: string): Urgencia
proximoEstado(pedido: Pedido, evento: Evento): Resultado<Pedido>
```

Esse único hábito elimina a maior fonte de bug de fuso horário que existe, e é a razão pela
qual o `checkoutMessage.ts` do catálogo formata data ISO quebrando a string em vez de criar
`Date`. Vale especialmente para `verificarViabilidade`
([ADR-0002](0002-fila-por-prazo-e-orcamento.md) D3), a única lógica do app em que o bug custa
dinheiro físico — pura, ela é testável em vinte casos sem banco, sem servidor e sem mock.

**D3 — Porta única de ingestão: `criarPedido(input)`.** A tela de cadastro manual chama ela.
O site, quando chegar a hora, chama a mesma. Havendo **um só caminho**, integrar é expor esse
caminho por HTTP.

**D4 — Chave de idempotência no input desde já.** Integração automática reenvia — e pedido
duplicado em floricultura é prejuízo físico, não linha a mais numa tabela.

**D5 — Formato compatível com o `Pedido` do catálogo**, não compartilhado: `Entrega` como
união discriminada, dinheiro em centavos inteiros, data ISO `yyyy-mm-dd`. Compatibilidade de
forma é o que torna a integração futura um mapeamento trivial; package compartilhado hoje
seria acoplar a um alvo em migração.

**D6 — Nada da integração é construído agora.** O que se constrói é a porta (D3) e a forma
(D5). Expor é [`escopo-v1.md`](../escopo-v1.md), seção Futuro.

## Consequências

- Efeitos (gravar, notificar, ler o relógio) ficam **nas bordas**. Pureza vale na camada de
  domínio; não é religião para o resto do app.
- Testar o coração do app não exige infraestrutura — o que é o que torna viável revisar o
  modelo de peso depois da primeira semana de uso real.
- `como-trabalhamos.md` já exige "funções puras na camada de domínio"; este ADR fixa o que
  isso significa em concreto e por quê.
- A escolha de stack ([`escopo-v1.md`](../escopo-v1.md), em aberto) não afeta o domínio. É de
  propósito: se a stack mudar, o núcleo não muda.
