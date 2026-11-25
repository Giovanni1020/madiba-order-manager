---
name: revisar-doc
description: Passe frio e independente sobre um ADR, SPEC ou doc de processo recém-escrito ou alterado. Monta o briefing disciplinado, invoca o agente revisor-artefato (Opus, contexto limpo, read-only) e mostra o laudo. On-demand, passe único, sem loop. Use quando o usuário pedir uma revisão fria de um documento ("revisa esse ADR", "/revisar-doc docs/adr/0002-...").
argument-hint: "Caminho do doc (ou arquivo + branch). Vazio = o doc alterado nesta sessão"
disable-model-invocation: true
---

# /revisar-doc $ARGUMENTS: o maestro do passe frio

Você orquestra a revisão fria de um documento. O **julgamento** é do agente
[`revisor-artefato`](../../agents/revisor-artefato.md) (Opus, contexto limpo, read-only); você
**não** decide o que é bloqueante. Seu trabalho: montar o briefing disciplinado, invocar o agente
**uma vez** e apresentar o laudo. Sem loop, sem correção automática.

`$ARGUMENTS` = o caminho do doc a revisar (ou o arquivo + a branch). Vazio = o doc criado ou
alterado nesta sessão.

## Onde roda

- **Avulso:** invocado direto sobre um doc, quando alguém quer um passe frio.
- **Sugerido, nunca automático:** quando uma sessão cria ou altera um ADR, uma SPEC ou um doc de
  processo em `docs/`, ela pode **oferecer** o `/revisar-doc` antes do PR. Quem decide é a pessoa.
  O custo de Opus só existe quando ela aceita.

Não dispara sozinho. O valor é o passe raro no doc que sustenta decisão, não um pedágio em todo
doc.

## 0. Fixar a raiz: o doc e o código têm que ser da mesma versão

O revisor confere o doc contra o código **da mesma versão**. Nunca contra outra branch.

- **O doc está no checkout atual** (sessão que acabou de escrever, ou branch já em check-out): a
  raiz é a pasta do repositório. Siga para o passo 1.
- **O doc está em outra branch:** não troque o checkout de ninguém. Crie um worktree só de leitura
  para a revisão e use-o como raiz:

  ```sh
  git fetch origin
  git worktree add .claude/worktrees/revisao-<branch> origin/<branch>
  ```

  `.claude/worktrees/` não é versionado. **Remova o worktree no fim** (passo 3), com
  `git worktree remove .claude/worktrees/revisao-<branch>`.
- **Working tree sujo com mudança que não é do doc:** diga isso no briefing (campo "fatos de
  apoio"), para o revisor saber que o código local pode não ser o da branch.

## 1. Montar o briefing disciplinado

Passe ao agente **só estes campos, crus**, nada composto ou resumido por você:

- **A raiz:** declare **uma vez** o diretório-raiz de onde ler doc e código, e passe todo o resto
  **relativo a ela**. Se houver mais de um clone ou worktree na máquina, diga explicitamente qual
  **não** usar. Path absoluto repetido campo a campo faz o agente abrir o clone errado.
- **O doc:** o **caminho** relativo à raiz. O ponteiro, não o conteúdo colado.
- **O tipo:** ADR / SPEC / processo (governança) / glossário.
- **A natureza:** _descritivo_ (documenta comportamento que já existe no código) ou _prescritivo_
  (decide ou especifica comportamento ainda não codado). **Enquanto o repositório não tiver o
  código que o doc descreve, todo ADR e toda SPEC são prescritivos.** Isso diz ao agente o que ele
  pode checar contra o código.
- **O contrato:** o link ou texto cru da issue do GitHub que pediu o doc, e a spec relacionada, se
  houver. Sem issue, diga de onde veio o pedido (ex.: "decisão registrada em
  `docs/escopo-v1.md`").
- **O escopo:** quais afirmações são novas ou alteradas.
  - Arquivo novo (untracked) → "doc inteiro".
  - Arquivo modificado, ou reinvocação após correção → os **hunks** do `git diff`, crus, não
    resumidos.
  - Hunks acima de **~100 linhas**: salve a saída crua do `git diff` num arquivo temporário fora do
    repositório e passe o **path**. São os mesmos bytes, então continua passando no teste da linha.
  - O agente lê o doc inteiro de todo jeito; o escopo só limita a verificação contra código, que é
    o que custa.
- **Fatos de apoio:** saída crua de `ls docs/adr/`, o path do template do tipo (se existir) e
  `git diff --name-only` da branch. Adianta round-trip que o agente pagaria sozinho.

**Branch com vários docs:** o padrão é revisar o **doc que sustenta a decisão** (o ADR ou a SPEC) e
1 ou 2 docs que ele governa, não o PR inteiro. O resto fica fora do escopo do briefing, e o laudo o
declara como não auditado. Se a pessoa pedir o PR inteiro, avise o custo (cada doc é uma rodada de
Opus) e siga. A alavanca de custo é escopo menor, não prompt mais magro.

⚠️ **NÃO passe o raciocínio da sessão que escreveu o doc** ("decidi X porque Y"), **nem um resumo
do doc composto por você.** Se você roda na mesma sessão que escreveu, qualquer coisa que você
redija carrega o viés do autor e transforma a independência do subagente em teatro. Só ponteiros e
saída crua de comando.

**O teste da linha:** o agente obteria **exatamente esses bytes** rodando o comando ele mesmo?
Sim → passe (você só poupou o round-trip). Não → não passe. Por isso a lista inteira de
`ls docs/adr/` é livre, mas **escolher quais ADRs são relevantes** já é julgamento seu e volta a
contaminar. Lista crua sim, curadoria não.

## 2. Invocar o revisor-artefato

Invoque o agente `revisor-artefato` com o briefing da seção 1. Ele devolve o laudo
(`VEREDITO: OK | BLOQUEADO` + afirmações conferidas + bloqueantes + sugestões + ponto cego). Você
**não** reinterpreta a classificação dele.

## 3. Apresentar o laudo

- **Triagem pós-laudo (anotar, nunca reclassificar):** antes de apresentar, confira cada bloqueante
  contra o que o autor **já publicou**: issue, corpo do PR, dependência declarada ali. Achado que o
  autor já declarou como intencional (ex.: link que só resolve depois do merge de outro PR
  anunciado como pré-requisito) sai com a anotação "já declarado pelo autor em <lugar>", **ao lado**
  da classificação do juiz, que fica intacta. Isso existe porque o briefing magro (correto) esconde
  do juiz o que o autor publicou; a correção é na saída, não engordando o briefing.
- Mostre o laudo: bloqueantes (com a evidência e a anotação da triagem, quando houver), sugestões
  (com severidade) e o ponto cego, se houver.
- **Ofereça persistir o laudo**, porque ele morre no transcript se ninguém salvar:
  - comentário no PR, quando o alvo é um PR, **com confirmação antes de postar**;
  - e/ou um arquivo local `.claude/laudo-YYYY-MM-DD-<alvo>-<sha>.local.md`. O SHA vai no nome
    porque o laudo vale para **aquela** versão do doc.
  - **Nunca** em `docs/`: laudo é veredito datado, não verdade do projeto.
- Se criou worktree no passo 0, **remova** agora.
- **Você não corrige nada sozinho e não reroda.** A pessoa decide:
  - `VEREDITO: OK` → segue.
  - `VEREDITO: BLOQUEADO` → a pessoa decide corrigir ou não. Se corrigir e quiser reconferir,
    **reinvoca o `/revisar-doc`**: passe novo e independente. Não há loop automático.

## Princípios (por que é assim)

- **Juiz ≠ maestro.** O `revisor-artefato` classifica; você só monta o briefing, invoca e
  apresenta. A independência do juiz mora no contexto limpo do subagente, e um briefing que vaza o
  racional do autor destrói isso. A triagem pós-laudo não fura a regra: acontece **depois** do
  julgamento, anota fato publicado e não toca no veredito.
- **Passe único, não loop.** Correção de doc costuma sair numa tentativa só. Se precisar
  reconferir, é uma reinvocação manual.
- **Priorizar não é afrouxar, e compra latência, não token.** A meta de ~8 afirmações e o orçamento
  de leitura do agente existem porque varredura exaustiva é round-trip serial, e afirmação que não
  pode virar bloqueante não paga o grep que a confirmaria. O que fica de fora é **declarado** no
  laudo (contagem de não auditadas), não escondido.

  > Medição herdada do projeto de origem desta skill (A/B no mesmo ADR, 2026-08): com o orçamento
  > de leitura, a rodada caiu de 30 para 16 tool calls e de 4m39s para 3m05s, com token empatado,
  > e o laudo achou uma sugestão de severidade alta que a versão sem orçamento perdeu. A mesma rodada
  > num modelo menor saiu pior nos três eixos e perdeu o achado: modelo mais fraco compensa com
  > round-trip, e round-trip é o que custa. **Não troque o modelo do agente e não persiga token
  > afinando o prompt.** Se o custo incomoda, rode menos vezes e com escopo menor.

- **On-demand, não gate automático.** Rodar Opus em todo doc vira carimbo. O passe frio se paga no
  doc raro que sustenta decisão, e quem sabe qual é esse doc é a pessoa.
