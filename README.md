# Cascata de modelos no Claude Code

Seu Opus não precisa fazer trabalho de Haiku. Este protocolo faz o Claude Code mandar cada tarefa para o **modelo mais barato que dá conta**. Quando esse nível não tem certeza do resultado, a tarefa **sobe um nível**.

```
tarefa → Opus (roteador) ─┬─ simples → rapido (Haiku)  ─┐
                          ├─ média   → medio  (Sonnet) ─┼─ CONFIANCA: baixa? sobe 1 nível
                          └─ difícil → pesado (Opus)  ──┘
```

## Como funciona

| Nível | Agente | Modelo | Para quê |
| --- | --- | --- | --- |
| 1 | `rapido` | Haiku | buscar, listar, ler e resumir, renomear, formatar, typos |
| 2 | `medio` | Sonnet | editar código, scripts, refatoração pequena, bug comum |
| 3 | `pesado` | Opus | arquitetura, bug complexo, trade-offs |

- Todo agente termina com `VALIDACAO: ok | falhou | nao_validado` e `CONFIANCA: alta | baixa`.
- Se voltar `falhou` ou `baixa`, a mesma tarefa sobe para o próximo nível.
- Se a tarefa depende de uma decisão sua (requisitos que se contradizem, pedido ambíguo, falta de permissão), o agente para e responde `BLOQUEIO: decisao_usuario`. Nesse caso a tarefa **não** sobe de nível: a pergunta vem para você. E nenhum agente pode mudar um teste só para fazê-lo passar.
- **Delegar tem custo fixo**, porque o subagente começa do zero. Por isso, tarefa minúscula (1–2 passos) fica com quem já está nela, e só tarefa com 3+ passos ou com leitura grande é delegada.

## Resultados do teste (06/11/2026)

Os 3 níveis rodaram as mesmas 4 tarefas, 2 vezes cada (24 execuções): somar uma planilha, achar as chamadas de uma função cheia de pegadinhas, corrigir um bug e uma armadilha (um teste errado de propósito).

| Nível | Acertos | Tokens por tarefa | Custo das 8 execuções (preço de API) |
| --- | --- | --- | --- |
| `rapido` (Haiku) | 8/8 | ~36 mil | US$ 0,017 |
| `medio` (Sonnet) | 8/8 | ~32 mil | US$ 0,26 |
| `pesado` (Opus) | 8/8 | ~32 mil | US$ 0,52 |

- **Mesmo resultado, ~30x mais barato:** os tokens empatam e o que muda é o preço de cada token.
- **Na armadilha, 6/6 execuções pararam e pediram uma decisão**, sem editar o teste. Isso inclui o Haiku.
- São tarefas pequenas e bem definidas. Em tarefas difíceis, o nível de cima continua sendo a escolha certa.

## Instalação (2 minutos)

1. Copie os 3 agentes para a sua pasta de agentes:

   **macOS / Linux / Git Bash**
   ```bash
   mkdir -p ~/.claude/agents && cp agents/*.md ~/.claude/agents/
   ```

   **Windows (PowerShell)**
   ```powershell
   New-Item -ItemType Directory -Force "$HOME\.claude\agents" | Out-Null; Copy-Item agents\*.md "$HOME\.claude\agents\"
   ```

2. Abra (ou crie) o arquivo `~/.claude/CLAUDE.md` e cole no fim o conteúdo de [`CLAUDE-snippet.md`](CLAUDE-snippet.md).

3. Reinicie o Claude Code. Para conferir, peça algo simples, como "liste os arquivos desta pasta e resuma o README". Deve aparecer uma linha assim:
   ```
   [cascata] listar e resumir → rapido → ok/alta
   ```

## Ajustes

- Quer um nível mais rígido? Edite o `description` de cada agente, que é o texto que o roteador lê para decidir.
- Todo agente tem a linha `tools:`. Sem ela, o agente herda todas as ferramentas da sessão (incluindo conectores MCP) e cada chamada carrega muito mais contexto. No teste, o `pesado` caiu de ~62 mil para ~11 mil tokens por chamada só com essa linha.
- O `rapido` só tem ferramentas de leitura (sem `Edit`/`Write`) de propósito: tarefa que edita arquivo já é nível 2.
- O registro `[cascata]` mostra o que foi delegado. Se algo sempre escala, mude a regra para mandar direto ao nível de cima.

## Licença

MIT. Use, adapte e compartilhe.

Feito por [@querol.ai](https://instagram.com/querol.ai).
