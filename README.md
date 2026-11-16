# Tanque do Claude (claude-tank)

Monitor do contexto e dos limites de uso do Claude Code no Windows. Funciona com o Claude Code em qualquer lugar: terminal (CLI), app desktop e extensões de IDE (VS Code, Cursor, Windsurf), porque todos gravam na mesma pasta `~/.claude`. O Clawd boia num tanque que enche conforme o contexto do chat cresce: feliz com pouco contexto, de cara fechada no meio e sufocado perto do limite.

![Clawd no tanque](monitor-app/clawd.png)

## O que faz

- **Origem do chat:** uma pill mostra se o chat roda no Terminal, no App ou no VS Code.
- **Contexto do chat atual** em %, com tokens no contexto e o total gasto no chat.
- **Limites da conta:** sessão de 5 horas e semana, com o horário em que cada um renova.
- **Ritmo da sessão:** leve, moderado ou avançado. No avançado o limite acaba antes de renovar, e o Clawd sua.
- **Pills do chat:** se ele usou o Obsidian, se o projeto tem Graphify e quais skills foram usadas.
- **Dicas** de economia de token e de uso do Claude Code, escolhidas pelo momento do chat.
- **Avisos do Windows,** mesmo com o app fechado:
  - contexto em 65% e 80%;
  - sessão ou semana em 75%;
  - limite renovado.
- **Aviso pro Claude aos 65%:** o hook manda ele parar, escrever um handoff das pendências e pedir um chat novo.
- **Retomar o chat atual quando o limite renovar** (opcional, desligado por padrão): 1 minuto depois do reset, roda `claude --resume <chat> -p ...` em modo `auto`. Acha o `claude` no PATH, no npm ou na extensão da IDE. Só retoma com algum Claude Code aberto (IDE, app ou terminal) e o chat parado há 3 minutos, e grava log em `~/.claude/hooks/logs`.
- **Ícone animado** na barra de tarefas, com o mesmo humor do Clawd.

## Instalar

Precisa do **Node.js** no PATH, porque os hooks do Claude Code rodam com `node`.

1. Baixe o `TanqueDoClaude-x.y.z.exe` em [Releases](https://github.com/luizxvk/claude-tank/releases) e abra.
2. No primeiro uso, o app copia os hooks pra `~/.claude/hooks` e registra `Stop`, `PostToolUse` e `UserPromptSubmit` no `~/.claude/settings.json`, sem duplicar se já existirem.

### Pelo código

```sh
git clone https://github.com/luizxvk/claude-tank.git %USERPROFILE%\.claude\hooks
cd %USERPROFILE%\.claude\hooks
npm install
npm start
```

Nesse caso, registre os hooks no `~/.claude/settings.json` com o comando `node "C:/Users/<você>/.claude/hooks/context-alert.js"` nos três eventos.

## Configuração

O arquivo `~/.claude/hooks/tanque-config.json` é criado pelo app e não vai pro git.

| Chave | O que faz |
| - | - |
| `avisarClaude` | Liga o aviso pro Claude aos 65% (padrão `true`). |
| `vault` | Pasta do vault do Obsidian pros handoffs (`<vault>/projetos/<projeto>/sessoes/`). Sem ela, o handoff vai pra `<projeto>/docs/handoff/`. |
| `retomar` | Liga a retomada automática do chat atual quando o limite renova. |

Variáveis de ambiente opcionais:

| Variável | Padrão | O que muda |
| - | - | - |
| `CLAUDE_CTX_ALERTS` | `65,80` | Limites de contexto que disparam aviso. |
| `CLAUDE_USAGE_ALERT` | `75` | % da sessão ou da semana que dispara aviso. |
| `CLAUDE_CONFIG_DIR` | `~/.claude` | Pasta de dados do Claude Code, se você usa outra. |
| `CLAUDE_CTX_WINDOW` | automático | Força o tamanho da janela de contexto. Opus 5.x e Fable usam 1M; os outros, 200k. |

## Como funciona

- O **contexto** vem do `usage` da última resposta no transcript da sessão (`~/.claude/projects/**.jsonl`).
- Os **limites** vêm do mesmo endpoint que o `/usage` usa, lido com o token que o Claude Code guarda no seu PC. Nada sai da sua máquina além dessa consulta.
- O **aviso de renovação** é uma tarefa agendada do Windows chamada `Claude limite renovou - <sessao|semana>`.

## Como o release é feito

O `.exe` da página de [Releases](https://github.com/luizxvk/claude-tank/releases) é um app portátil (não precisa instalar) gerado assim:

1. Suba a `version` no `package.json` (ex.: `1.0.0` → `1.0.1`).
2. `npm install` (só na primeira vez) e `npm run dist`. O `electron-builder` empacota o app Electron e os hooks e grava `dist/TanqueDoClaude-<versão>.exe`.
3. Commit e push das mudanças:
   ```sh
   git add -A && git commit -m "Tanque do Claude <versão>" && git push
   ```
4. Publique o release com o `.exe` anexado (precisa do [GitHub CLI](https://cli.github.com)):
   ```sh
   gh release create v<versão> dist/TanqueDoClaude-<versão>.exe --title "Tanque do Claude <versão>" --notes "O que mudou"
   ```

A pasta `dist/` não vai pro git; o `.exe` só existe como anexo do release.

**O que o `.exe` faz no primeiro uso:** copia `context-alert.js`, `clawd.js`, `renovou.js`, `renovou.ps1`, `toast.ps1` e `context-monitor.html` pra `~/.claude/hooks` e registra os três hooks no `~/.claude/settings.json`. Depois disso, os avisos funcionam mesmo com o app fechado.

## Desenvolvimento

```sh
npm start                  # abre o app
npm run icone              # regera clawd.ico / clawd.png a partir do desenho
npm run dist               # gera dist/TanqueDoClaude-<versão>.exe
node context-alert.js --test           # dispara um aviso de teste
node renovou.js sessao --seco          # mostra o que a retomada faria, sem rodar o Claude
```

Para conferir a tela sem abrir janela: `electron monitor-app/shot.js saida.png [82]`. O número opcional força um % de contexto.

O Clawd é o mascote do Claude, da Anthropic. Este é um projeto pessoal, sem vínculo com a Anthropic.
