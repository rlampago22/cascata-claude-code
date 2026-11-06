# Roteamento de modelos (cascata)
Antes de executar uma tarefa, classifique-a e delegue ao nível MAIS BARATO que dê conta:
- Simples/mecânica (buscar, listar, ler e resumir, renomear, formatar, typos, comandos simples) → agente `rapido` (Haiku).
- Média (editar código/texto com contexto, scripts, refatoração pequena, bug comum) → agente `medio` (Sonnet).
- Difícil (arquitetura, bug complexo, raciocínio longo, trade-offs) → faça você mesmo ou use o agente `pesado` (Opus).

Escalonamento: se um agente retornar `VALIDACAO: falhou` ou `CONFIANCA: baixa`, repita a tarefa no próximo nível (rapido → medio → pesado/você). Exceção: se o agente retornar `BLOQUEIO: decisao_usuario` (requisitos que se contradizem, ambiguidade, permissão), NÃO escale — um modelo maior não resolve isso; leve a decisão ao usuário. Prefira validação objetiva (compila, teste passa, arquivo existe) à confiança declarada.

Ponto de equilíbrio (economia de tokens): cada delegação tem custo fixo (o subagente começa do zero). Por isso:
- DELEGUE quando a tarefa exigir 3+ chamadas de ferramenta OU leitura de conteúdo grande (vários arquivos, logs, PDFs, saídas longas) — o subagente trabalha em contexto pequeno e devolve só o resumo, mantendo esta conversa leve.
- Escrever/editar script ou código com 3+ passos (criar, rodar, conferir) → `medio`.
- FAÇA DIRETO: conversa, perguntas sem ferramentas e tarefas de 1–2 chamadas com conteúdo pequeno (listar uma pasta, ler um arquivo curto, um comando).
Ao delegar, passe no prompt todo o contexto necessário (o subagente não vê esta conversa) e peça uma resposta curta.

Registro: sempre que delegar, mostre ao usuário uma linha no formato
`[cascata] <tarefa resumida> → <nivel> → <VALIDACAO>/<CONFIANCA>` (e uma linha extra se escalonou).
