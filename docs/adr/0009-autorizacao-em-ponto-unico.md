# ADR-0009 — Autorização em ponto único; matriz de permissões deferida

- **Status:** Aceito — 2026-11-22
- **Contexto do projeto:** poucas pessoas da empresa, contas **criadas manualmente**, sem
  opção de autocadastro. Provedor de autenticação **pendente** (vai com o ADR de stack).

## Contexto

O requisito é: login para poucas pessoas, adicionadas à mão, sem criar conta; papéis com
permissões diferentes (admin define limite de capacidade; funcionário cadastra e altera
pedidos).

Há uma restrição honesta de equipe: **não há ninguém de segurança por profissão**. Quem
implementar isso é quem está escrevendo o app, e segurança é o ponto declarado de menor
conhecimento. Isso **aumenta**, não diminui, o peso de não escrever primitivos próprios.

## Decisão

**D1 — Autenticação não é escrita à mão.** Nenhuma implementação própria de hash de senha,
sessão, token ou reset. Provedor/biblioteca estabelecida, com **autocadastro desabilitado** e
usuários criados por admin. "Poucas pessoas, adicionadas manualmente, sem criar conta" é
**configuração**, não feature.

**D2 — Ponto único de autorização: `pode(usuario, acao, recurso)`, no servidor.** Toda
verificação passa por aí. Esconder o botão **não é controle de acesso**: sem a checagem na
rota ou na mutation, qualquer pessoa autenticada com o devtools aberto é admin.

**D3 — O conteúdo da matriz é deliberadamente deferido.** Quais papéis existem e o que cada um
pode é informação que muda em um dia depois do primeiro feedback de uso, e é **barata de
mudar**. O que **não** é deferido é o mecanismo de D2: com ele, mudar permissão é mudar dado;
sem ele, são dez `if (role === ...)` em dez arquivos e um deles vai ser esquecido. O que custa
caro não é escrever o `if` — é descobrir onde todos eles estão.

**D4 — Papel vive no banco**, não apenas no token. Token pode estar velho; a fonte de verdade
é consultada no servidor.

**D5 — Duas regras já valem, porque não são condicional de tela:**

- **capacidade da hora é entidade auditada com dono** — quem alterou, quando, de quanto
  ([ADR-0001](0001-peso-como-unidade-de-capacidade.md) D5,
  [ADR-0004](0004-registro-e-auditoria.md) D5);
- **imutabilidade após estado terminal** — se pedido entregue/retirado não pode ser alterado,
  isso é regra de domínio, aplicada em `proximoEstado`
  ([ADR-0003](0003-estados-do-pedido-em-eixos.md) D6), não permissão de tela. *Quais* campos
  travam e a partir de qual estado fica em aberto.

**D6 — Toda escrita carrega ator identificado**, requisito da auditoria
([ADR-0004](0004-registro-e-auditoria.md)).

## Consequências

- O v1 precisa de pelo menos dois papéis para que `pode()` seja exercitado de verdade — um
  papel só esconde o erro de desenho até a hora errada.
- Visibilidade de dados de cliente é decisão de permissão, não de tela: funcionário vendo
  endereço de toda cliente do histórico é escolha a ser feita de propósito
  ([`escopo-v1.md`](../escopo-v1.md), item jurídico em aberto).
- A escolha do provedor de autenticação entra no ADR de stack e **não** altera D2–D6.
