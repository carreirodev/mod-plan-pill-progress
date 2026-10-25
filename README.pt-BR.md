# Plan Pill Progress

[English](README.md) · **Português (Brasil)**

Barras de progresso ao vivo acima do campo de entrada do Claude Code. O Claude divide a tarefa em etapas e passos, e a barra avança conforme ele trabalha. Os subagentes de cada tarefa aparecem abaixo da barra.

Baseado no plan-progress de Kirill Serditov, sob a licença MIT.

- Uma linha por tarefa: no desktop e no VS Code, estado, a barra com o título dentro e a porcentagem na etiqueta, botão de fechar; no terminal, estado, título, barra, porcentagem, botão de fechar
- A etiqueta na barra mostra a etapa e o passo atuais e, no desktop e no VS Code, a porcentagem até a tarefa terminar; ao passar o mouse, o tempo decorrido
- Etapas são cápsulas e passos são pontos; ao passar o mouse sobre um já alcançado, mostra quanto tempo levou, e sobre um à frente, o que começa ali
- Uma barra concluída fica verde, mostra o tempo total e fica até ser fechada; com `/plan-progress-autoclose` ativado, some após `doneBarSeconds` (30 s por padrão, em `/config`)
- Os pontos do preenchimento e das linhas de subagentes cintilam; desative `twinkle` em `/config` para desenhá-los parados
- Quatro estados: em andamento, aguardando resposta, erro, concluído
- Cada subagente tem uma linha sob sua tarefa: seu número, nome, modelo e effort, ferramenta atual, tempo; no terminal a marca dele gira enquanto roda e vira ✓ ou ✕ quando termina
- No terminal, ligue `hideAgentRows` em `/config` e a lista de agentes do próprio Claude Code, embaixo do campo de entrada, deixa de mostrar os agentes que uma barra já mostra; os outros continuam lá, e a seta ↓ ainda chega a todos. Para isso o mod acrescenta uma linha `subagentStatusLine` ao `settings.json` do Claude Code e a tira de novo quando a opção é desligada. Se você mesmo já definiu `subagentStatusLine`, o mod deixa como está e não esconde nada
- O plano pode mudar durante o trabalho; os passos concluídos são mantidos pelo título
- As barras são salvas por sessão e voltam quando a sessão é retomada
- Sons curtos para pergunta, erro e conclusão
- O rodapé mostra a velocidade com que a conversa principal escreve, `⚡ N tok/s`; subagentes não contam
- O ⚙ ao lado (no app de desktop e no terminal em tela cheia) abre uma janela de configurações: efeitos, fechamento automático e seus segundos, sons, o próprio número, a versão do mod e a busca por atualizações; `/plan-progress-settings` a abre em qualquer lugar. A janela usa o idioma do `/config`; se não houver, o do app ou o do sistema
- Atualizações: clicar na versão na janela de configurações procura uma versão nova, e «Atualizar» a instala e recarrega o mod na hora (se a recarga não funcionar, pede para reiniciar). A cada 6 horas o mod também verifica sozinho e marca o ⚙ com 🔹 (● no terminal) quando sai uma versão nova. Cada verificação é uma única requisição à API do GitHub (`api.github.com/repos/carreirodev/mod-plan-pill-progress/releases`), sem dados da conta ou da sessão; desligue em «Procurar atualizações» na janela ou com `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`. Uma cópia instalada de uma pasta não é verificada
- Funciona no app de desktop, na extensão do VS Code e no terminal
- No terminal a barra segue o modo claro ou escuro do GNOME com o tema `auto` e usa as cores do tema ativo do Omarchy
- No terminal o preenchimento é feito de pontos em braille, esparsos no começo e densos perto da etiqueta, com a mesma onda de luz do desktop correndo sobre eles
- No terminal a barra, a etiqueta e as linhas dos subagentes têm pontas redondas, e cada linha um robô antes do número, no WezTerm, no Ghostty (e nos terminais feitos sobre ele, como o noctty) e no kitty, e com um tema do Omarchy; nos outros terminais ficam as pontas retas e só o número, porque a fonte pode não ter esses símbolos

## Instalação

Requer o Claude Code 2.1.295 ou mais recente (`claude --version`; atualize com `claude update`). Em versões anteriores parte do mod não funciona: a janela de configurações fica vazia e, antes da 2.1.286, o módulo de hooks nem carrega, nenhuma barra aparece e na inicialização aparece `plan-progress: hooks module did not load`.

Ele se instala pelo seu identificador, `plan-progress`, a partir do [WPC Mods for Claude](https://github.com/carreirodev/wpc-mods-claude/blob/main/README.pt-BR.md), o marketplace com todos os meus mods. Há dois jeitos, e os dois fazem a mesma coisa: escolha um.

**Dentro do Claude Code**, numa sessão:

```
/plugin marketplace add carreirodev/wpc-mods-claude
```

```
/plugin install plan-progress@wpc-mods-claude
```

**Pelo terminal**, fora do Claude Code:

```
claude plugin marketplace add carreirodev/wpc-mods-claude
```

```
claude plugin install plan-progress@wpc-mods-claude
```

De um jeito ou do outro, depois abra uma sessão nova, ou rode `/reload-plugins` numa sessão já aberta.

Atualização, pelo terminal:

```
claude plugin marketplace update wpc-mods-claude
```

```
claude plugin update plan-progress@wpc-mods-claude
```

Instalou antes pelo marketplace deste repositório (`plan-progress@plan-pill-progress`)? Essa cópia continua atualizando como antes. Para trocar, siga [estes passos](https://github.com/carreirodev/wpc-mods-claude/blob/main/README.pt-BR.md#se-você-já-instalou-um-mod-pelo-repositório-dele); não fique com as duas, porque só uma roda.

## Comandos

- `/progress` mostra ou oculta as barras
- `/progress-clear` remove todas as barras
- `/progress-agents` recolhe as faixas de agentes sob as barras ou as mostra de novo (o botão de seta ao lado do ✕ de uma barra faz isso só para ela)
- `/plan-progress-autoclose` ativa o fechamento automático das barras concluídas (desativado por padrão) ou o desativa de novo; a escolha é mantida entre sessões
- `/plan-progress-settings` abre a janela de configurações, como o ⚙ do rodapé (no terminal o ⚙ só aparece em tela cheia)

## Como funciona

O mod registra a ferramenta `plan_progress`. O Claude envia o plano uma vez e depois atualizações curtas, como `{id, next: true}` ou `{id, done: ["Routes"]}`. Um nome de passo desconhecido é recusado com a lista de passos da barra. Um plano aprovado no plan mode vira a barra `plan`. As linhas dos agentes vêm dos eventos do motor e não gastam tokens.

No app de desktop a barra é uma imagem SVG com uma camada de hover por cima. No terminal é uma grade de caracteres que só anima enquanto o Claude trabalha.

## Testes

`plugins/plan-progress/tests` executa o módulo real sobre um motor simulado: `node compile.cjs ../hooks/register.tsx register.mjs`, depois `node regress.mjs` e `node scenarios.mjs`.

## Licença

MIT
