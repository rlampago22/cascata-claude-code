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
- **Delegar tem custo fixo**, porque o subagente começa do zero. Por isso, tarefa minúscula (1–2 passos) fica com quem já está nela, e só tarefa com 3+ passos ou com leitura grande é delegada.

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
- O `rapido` só tem ferramentas de leitura (sem `Edit`/`Write`) de propósito: tarefa que edita arquivo já é nível 2.
- O registro `[cascata]` mostra o que foi delegado. Se algo sempre escala, mude a regra para mandar direto ao nível de cima.

## Licença

MIT. Use, adapte e compartilhe.

Feito por [@querol.ai](https://instagram.com/querol.ai).
