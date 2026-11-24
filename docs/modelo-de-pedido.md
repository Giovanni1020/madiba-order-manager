# Modelo de pedido — herdado do catálogo

> Transcrito de `src/types/order.ts` e `src/components/checkoutMessage.ts` do
> **Madiba Gift Catalog** em **2026-11-22**. É o formato com que o pedido **nasce hoje**.
> Decisões próprias deste projeto (persistência, status, identidade do pedido) virão em ADRs.

## Tipos do domínio (catálogo)

```ts
export interface Endereco {
  cidade: string;
  rua: string;
  numero: string;      // string de propósito: aceita "123A", "s/n"
  bairro: string;
  complemento: string; // opcional (apto, bloco, referência) — até 50 chars; "" quando vazio
}

// UNIÃO DISCRIMINADA em `tipo`: o bloco de endereço só existe na entrega.
export type Entrega =
  | { tipo: "retirada"; data: string; horario: string }
  | { tipo: "entrega"; endereco: Endereco; recebe: string; data: string; horario: string };

export interface Cliente {
  nome: string;
  telefone: string;    // só dígitos (DDD + número); o DDI entra no helper do WhatsApp
}

export type Pagamento = "pix" | "link";

export interface Pedido {
  itens: CartItem[];
  total: number;       // CENTAVOS
  cliente: Cliente;
  entrega: Entrega;
  pagamento: Pagamento;
}
```

`data` é **ISO `yyyy-mm-dd`** e é formatada sem criar `Date` (evita problema de fuso/UTC).
`horario` é uma **janela** de entrega/retirada, não um instante.

## Um item do pedido

Um `CartItem` carrega `product`, `quantity`, opcionalmente `variant` e `extras`. Os adicionais
observados no formatador do WhatsApp:

- `balao` (texto), `plaquinha` (texto), `polaroides` (bool)
- `cartao`: `"pre_escrito" | "branco"` — quando `"branco"`, `cartaoMensagem` leva o texto
- `chocolates`: mapa `id → quantidade` (`CHOCOLATE_OPTIONS`)

O preço de linha é
`quantity * (basePrice(product, variant) + extrasTotal(extras, product))` — ou seja,
**preço não é campo do produto**: depende de variante (ADR-0005) e de promoção vigente
(ADR-0006).

## Consequências para este projeto

1. **Dinheiro em centavos, inteiro.** Nada de float.
2. **Preço precisa ser congelado no pedido.** Variante + promoção com `expiresAt` significam
   que recalcular pelo catálogo depois dá valor diferente. Um pedido registrado guarda o preço
   praticado.
3. **`Entrega` é união discriminada** — preservar isso evita endereço vazio em retirada.
4. **O pedido chega como texto de WhatsApp**, não como JSON. Qualquer importação precisa
   decidir entre *parsear a mensagem*, *mudar o catálogo para emitir dados estruturados*, ou
   *entrada manual*. **Decisão aberta → ADR.**
5. **Não existe identificador de pedido hoje.** Criar um é decisão deste projeto →
   resolvido em [ADR-0005](adr/0005-identidade-do-pedido.md).
6. **Não existe status de pedido hoje.** A máquina de estados é o coração deste projeto e não
   existe em lugar nenhum no catálogo → resolvida em
   [ADR-0003](adr/0003-estados-do-pedido-em-eixos.md), que a fatora em eixos ortogonais
   (estado, pagamento, atenção, tempo derivado).

> **Decisões deste projeto que partem daqui:** congelamento de preço e peso
> ([ADR-0006](adr/0006-congelamento-de-preco-e-peso.md)), porta única de ingestão com formato
> compatível ([ADR-0007](adr/0007-dominio-puro-e-porta-de-ingestao.md)) e **peso** como
> unidade de capacidade de produção, que não existe no modelo do catálogo
> ([ADR-0001](adr/0001-peso-como-unidade-de-capacidade.md)).
