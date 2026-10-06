# Workshop Archon — TLC Floripa

Exercícios progressivos de workflows do Archon para o TLC Floripa. Cada arquivo em `.archon/workflows/exercises/` é um workflow independente. A sequência começa em nós determinísticos de shell e chega em revisão com modelo, decisão humana e composição de workflows.

O Archon orquestra nós. Um nó pode ser um comando, um modelo ou outro workflow. A saída de um nó vira entrada do próximo, com validação, condição e pausa para uma pessoa quando o arquivo declara isso.

## Pré-requisitos

- CLI `archon` instalada e configurada (`archon` no `PATH`).
- Este repositório clonado. Os comandos abaixo rodam na raiz: o diretório atual é o projeto que o Archon usa para descobrir os workflows.

## Como rodar

Liste os workflows deste repositório:

```bash
archon workflow list
```

Rode um exercício pelo `name` declarado no YAML:

```bash
archon workflow run aprendizado-01
```

Os exercícios 01 a 07 só executam shell. Dá para percorrer o grafo sem chamar um modelo:

```bash
archon workflow run conditionals-05 --dry-run --exec-code
```

A partir do exercício 08 os nós usam modelo. Esses workflows declaram `mutates_checkout: false` e leem a solicitação em `$ARGUMENTS`. Os comentários nos YAML usam `--no-worktree` para executar no checkout atual:

```bash
archon workflow run first-ai-08 --no-worktree \
  "Problema: relatórios internos são montados manualmente. Precisamos de uma proposta inicial de automação."
```

Inputs nomeados entram com `--input`. O exercício 06 exige `tipo`:

```bash
archon workflow run inputs-06 --input tipo=BUG --input prioridade=alta
```

O exercício 07 separa a mensagem livre (`$ARGUMENTS`) dos parâmetros nomeados:

```bash
archon workflow run arguments-07 --input formato=curto \
  "Resuma o objetivo deste workshop em uma frase."
```

Workflows com `interactive: true` (exercícios 12 e 13) pausam para uma decisão e recusam `--detach` no lançamento. Inspecione a pausa e responda com uma das decisões declaradas no YAML:

```bash
archon workflow get <RUN_ID> --json
archon workflow respond <RUN_ID> approve "Plano revisado e aprovado."
archon workflow respond <RUN_ID> needs-revision "Detalhe o plano de rollback."
```

O stdout mostrado na CLI é o do último nó. O valor que segue adiante é o de `returns:`. Para ver esse valor depois da execução:

```bash
archon workflow get <RUN_ID> --json
```

Cada YAML traz comentários sobre o conceito daquele passo e, quando faz sentido, um briefing de exemplo. Leia o arquivo antes de rodar.

## Sequência

| # | Workflow | Arquivo | O que mostra |
| --- | --- | --- | --- |
| 01 | `aprendizado-01` | `01-hello.yaml` | Dois nós de shell em sequência, com `depends_on` |
| 02 | `parallelism-02` | `02-parallelism.yaml` | Nós independentes em paralelo e um nó que espera os dois |
| 03 | `outputs-03` | `03-outputs.yaml` | `$node.output` entre nós e `returns` como resultado do workflow |
| 04 | `structured-output-04` | `04-structured-output.yaml` | JSON validado com `output_format` e acesso a campos |
| 05 | `conditionals-05` | `05-conditionals.yaml` | `when` para caminhos alternativos e `trigger_rule` quando uma rota é ignorada |
| 06 | `inputs-06` | `06-inputs.yaml` | Inputs obrigatórios, valor padrão e `$INPUTS` / `$INPUTS_TIPO` |
| 07 | `arguments-07` | `07-arguments.yaml` | Mensagem livre em `$ARGUMENTS` e parâmetros nomeados em `inputs` |
| 08 | `first-ai-08` | `08-first-ai.yaml` | Primeiro nó de modelo, `allowed_tools: []` e `mutates_checkout: false` |
| 09 | `producer-reviewer-09` | `09-producer-reviewer.yaml` | Proposta e revisão em sessões separadas (`context: fresh`) |
| 10 | `structured-review-10` | `10-structured-review.yaml` | Veredito estruturado do modelo escolhendo o caminho determinístico |
| 11 | `outcome-11` | `11-outcome.yaml` | `outcome_field`: a execução terminou e o critério de negócio foi atingido |
| 12 | `human-approval-12` | `12-human-approval.yaml` | Pausa com `approval` e decisões fixas (`approve`, `needs-revision`) |
| 13 | `interactive-loop-13` | `13-interactive-loop.yaml` | Loop no mesmo nó até `ready`, com feedback em `$LOOP_USER_INPUT` |
| 14 | `loop-group-14` | `14-loop-group.yaml` | Pipeline produzir → revisar repetida até `until_bash` |
| 15 | `review-block-15` | `15-review-block.yaml` | Bloco reutilizável de revisão, com inputs próprios |
| 15 | `composition-15` | `15-composition.yaml` | `include` do bloco no mesmo run, worktree e custo |
| 16 | `child-run-16` | `16-child-run.yaml` | O mesmo bloco como child run, via `workflow:` |

`review-block-15` não é um passo solto da trilha: `composition-15` o inclui no run atual e `child-run-16` o dispara como execução filha.
