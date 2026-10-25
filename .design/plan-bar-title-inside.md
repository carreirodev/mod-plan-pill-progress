# Barra do plano com o título dentro

> Plan from this document. Each slice below carries its own shape - copy it, do not re-derive it.
> Status: confirmed by carreirodev, 2026-10-25

## Situation

- Project: in steady use. O plan-progress 1.3.0 está publicado no marketplace wpc-mods-claude e desenha no app de desktop, na extensão do VS Code e no terminal.
- Decision: open. O autor confirmou nesta conversa: no desktop e no VS Code, o título vai para dentro da barra e o preenchimento passa a ter pontos espalhados.
- In flight: o WPC-246 (status-pills na faixa acima do prompt do VS Code, em revisão) é o precedente de desenhar essa faixa no VS Code. O WPC-243 (dividir o módulo de ~4.000 linhas, no backlog) mexe no mesmo módulo e fica de fora.
- At stake: só a aparência. Nada salvo muda de forma, e uma versão nova reverte tudo. O risco é o título ficar ilegível sobre o preenchimento em algum estado ou tema, ou a linha ainda quebrar no VS Code em alguma largura.

## Problem

Na extensão do VS Code, cada barra de plano ocupa três linhas: o título em cima, a barra no meio, a porcentagem e o ✕ embaixo. Como a barra fica curta, a pílula deixa de mostrar o nome da etapa e mostra só o número dela ("3"). No desktop, as linhas dos agentes já têm pontos espalhados e o nome escrito dentro, mas a barra do plano logo acima delas tem uma grade regular de pontos e o título do lado de fora. As duas parecem coisas diferentes, uma em cima da outra. Não há números de uso: o mod não coleta dados, e não se sabe quantas pessoas o usam no VS Code. A evidência são duas capturas de tela do autor, de 2026-10-25, uma do desktop e outra do VS Code.

## Success

- Worked if: no VS Code, na largura de painel que o autor usa, cada barra de plano ocupa uma linha só e a pílula mostra o nome da etapa. No desktop, a barra do plano e as linhas dos agentes parecem da mesma família: o nome dentro, pontos espalhados.
- Going wrong: o título some ou fica difícil de ler sobre o preenchimento em algum estado (aguardando, erro, concluído) ou no tema claro; ou a linha ainda quebra quando o painel do VS Code estreita.
- Review: na primeira sessão depois da versão nova, no desktop e no VS Code - carreirodev.

## Boundary

In: a linha da barra do plano no desktop e no VS Code. Isso cobre onde fica o título, o preenchimento, a regra do nome na pílula e a linha única no VS Code. O app de celular desenha pelo mesmo caminho do desktop, então, se mostrar a faixa, recebe o desenho novo; isso não foi conferido.

Out:
- O terminal: fica como está, com o título fora da barra e os pontos em braille. O autor pediu só desktop e VS Code, então daqui em diante as superfícies deixam de ser iguais nesse ponto.
- O cabeçalho "From the plan-progress plugin" e a moldura em volta da faixa no VS Code: quem os desenha é a extensão, não o mod.

Unchanged: a aparência das linhas dos agentes. Elas continuam alinhadas sob a barra, que agora começa logo depois da marca de estado, então começam mais à esquerda e ficam mais largas. Também não mudam a porcentagem, a seta de recolher e o ✕; o relógio que a pílula mostra quando o mouse passa sobre a barra e as dicas dos marcos, que só mudam de posição (Key decision 1); o texto alternativo da barra; o registro `Plan` e as barras salvas por sessão; a janela de configurações.

## Shape

A barra do plano passa a ser uma cápsula como a linha de um agente: o título do plano à esquerda, num trecho só dele, e o progresso no resto (o preenchimento, os marcos e a pílula), sobre um campo de pontos espalhados. A linha perde a coluna do título, então a barra fica com toda a sobra e cabe numa linha só no VS Code. A porta é o lugar do título: depois que as pessoas se acostumarem a ler a barra assim, mudar de novo é outra reaprendizagem. Mas nada salvo muda, e uma versão nova desfaz tudo.

A alternativa mais pesada é levar o mesmo desenho ao terminal. Ela só ganha se as três superfícies precisarem continuar idênticas, e o autor deixou o terminal de fora.

## Key decisions

1. **O título do plano fica dentro da barra, à esquerda, num trecho que só ele usa. O preenchimento começa na ponta esquerda, por baixo do título. A ponta do preenchimento, a pílula, os marcos de etapa e de passo e as dicas correm do fim desse trecho até a ponta direita.** A pílula nunca cobre o título, nem no começo do plano. É a variante B, que o autor escolheu entre três desenhos (Sources). Na A, o título ficava coberto no começo de todo plano; na C, ele mudava de lugar uma vez em cada plano.
2. **No desktop e no VS Code a linha não tem mais coluna de título. Ela tem marca de estado, barra, porcentagem, seta de recolher e ✕, e a barra fica com toda a sobra.** O terminal continua com a coluna de título dele: a partir daqui as superfícies divergem de propósito.
3. **No VS Code a linha da barra não quebra em nenhuma largura em que caibam a barra mínima (120 px, o mínimo de hoje) e o resto da linha: com a conta da vscode row, a partir de 22 colunas no Windows e 33 nos outros sistemas (25 e 38 quando a linha tem a seta de recolher).** Abaixo disso, nada cabe numa linha só. O VS Code conta as colunas pela largura do mais largo entre "0" e "─" na fonte do painel (a da interface, não a do editor) e quebra toda linha que não cabe. O mod só recebe o número de colunas, não quantos pixels cada uma tem. Corrigido em 2026-10-25, na 1.4.1, com o autor: a 1.4.0 supôs a fonte do editor e 6,5 px por coluna, e no VS Code do autor, no Windows, a coluna mede cerca de 12 px, então a barra ficou com metade da faixa. Na mesma correção: a extensão não posiciona caixas por cima de outras nem mostra hover, e punha a camada de hover da barra ao lado dela, com a largura dela; no VS Code a barra vai sozinha.
4. **O preenchimento é um campo de pontos espalhados, rarefeito e fraco no começo, mais denso e claro perto da pílula, como o do terminal. Essa rampa corre só sobre o trecho do progresso: sob o título os pontos ficam sempre no nível mais fraco.** Isso refina os desenhos, onde a 4% os pontos sob o título já eram densos. Cada barra tem os seus pontos, cada um num lugar fixo que nunca muda nem se embaralha, nem quando a faixa é redesenhada. O que muda com o avanço é quais pontos aparecem e com que brilho: os de trás podem rarear e enfraquecer conforme a pílula anda. A onda de luz, o cintilar e a versão parada (cintilar desligado, ou movimento reduzido no sistema) continuam. A figura fica abaixo do limite do motor para um `Svg` (131.072 caracteres) mesmo na barra mais larga (1.400 px). A grade de hoje é um padrão que se repete e não cresce com a largura; um desenho ponto a ponto cresce, e é aí que esse limite morde.
5. **O título se lê com contraste AA sobre o preenchimento, nos dois temas e nos quatro estados (em andamento, aguardando, erro, concluído), em qualquer ponto do progresso.** O nível fraco e fixo sob o título (Key decision 4) é o que garante isso. Adensar ou clarear os pontos ali quebra esta regra.

## Work

| Slice | Delivers | Status |
|---|---|---|
| [Title zone](#title-zone) | o título dentro da barra, o progresso depois dele e a linha sem coluna de título, no desktop e no VS Code | open — 4 defaults taken |
| [vscode row](#vscode-row) | a linha da barra numa só, em qualquer largura do painel do VS Code | open — 1 default taken |
| [Scattered fill](#scattered-fill) | o preenchimento de pontos espalhados no desktop e no VS Code | clear |

Order: Title zone → vscode row, e Title zone → Scattered fill. As duas últimas não dependem uma da outra. Scattered fill pode começar antes, mas a regra de sob o título (Key decision 4) só se confere depois que o trecho do título existir.

Already handled by existing code: as linhas dos agentes sob a barra (já têm pontos espalhados e o nome dentro); o texto alternativo da barra, que já começa pelo título; as barras salvas, que voltam ao retomar a sessão.

Derivable from the repository, left to the plan: a cor do título em cada tema, como a do nome nas linhas dos agentes; o corte com "…", como nos rótulos da pílula; a dica do título, como as dicas dos marcos; o tom e a velocidade da onda em cada estado, como hoje; os casos de `regress.mjs` que conferem a grade e a coluna do título, que mudam junto com o desenho.

### Title zone

**Delivers** o título do plano dentro da barra, à esquerda, e o progresso no resto dela, no desktop e no VS Code. **Status: open.** É a porta: as outras duas fatias assumem a Key decision 1.

```
hoje     ● WPC-245        ╭▒▒▒▒▒▒▒▒▒(Build │ API)·····│·····╮  40%  ✕
depois   ● ╭WPC-245 ⠂⠄⡀⠂⣀⣠⣤(Build │ API)·····│·····╮  40%  ✕
            └ título ┘└──────────── progresso ───────────┘
```

| State | O que a pessoa vê |
|---|---|
| Plano novo, nenhum passo concluído | o título inteiro à esquerda, o preenchimento só sob ele, no nível fraco, e a pílula logo depois do trecho do título. Hoje uma barra nova não tem preenchimento nenhum; isso muda |
| Plano no meio | o título à esquerda, sobre o nível fraco do preenchimento, e a pílula na ponta do preenchimento |
| Plano perto do fim (85–99%) | a pílula perto da ponta direita, e o título no mesmo lugar |
| Concluído | a barra verde, o título no mesmo lugar e o tempo total na pílula, na ponta direita |
| Aguardando resposta, ou erro | a barra âmbar ou vermelha, a pílula com ? ou ✕, e o título legível (Key decision 5) |
| Duas ou três barras com títulos de tamanhos diferentes | o trecho do título tem a mesma largura em todas, a do título mais longo, então o progresso começa no mesmo ponto e os marcos ficam alinhados, como a coluna de título faz hoje |
| Título maior que o trecho | cortado com "…", e inteiro ao passar o mouse sobre ele e no texto alternativo |
| Trecho do progresso com 160 px ou mais | a pílula mostra o nome da etapa, encurtado se precisar, e o passo depois dele se couber |
| Trecho do progresso com menos de 160 px | a bolinha redonda com o número da etapa, que hoje aparece sempre que a barra inteira tem menos de 360 px |
| Tema claro | o título na cor de texto do tema claro, e os marcos como hoje |
| Mouse sobre o trecho do progresso | o relógio na pílula e as dicas dos marcos, como hoje |

Alternatives considered: o título passando por baixo da pílula (variante A) - ganharia se o progresso precisasse da barra inteira, mas esconde o título no começo de todo plano. O título acompanhando a pílula (variante C) - ganharia pelo mesmo motivo, mas o título muda de lugar uma vez em cada plano.

1. Largura máxima do trecho do título - 30% da barra. Hoje a coluna do título vai até 30% da linha inteira.
2. Barra estreita demais para "Abc…" caber nesses 30% - a barra fica sem trecho de título, e o título só aparece no texto alternativo.
3. Dica com o título inteiro - aparece ao passar o mouse sobre o trecho do título, e só quando ele foi cortado.
4. Onde a pílula vira bolinha - 160 px de progresso. É o mínimo para uma pílula com um nome curto ainda andar pela barra: em andamento ela tem pelo menos uns 70 px, porque precisa caber o relógio.

### vscode row

**Delivers** a linha da barra numa só, em qualquer largura do painel do VS Code. **Status: open.**

| State | O que a pessoa vê |
|---|---|
| Painel estreito, como na captura de 2026-10-25 | uma linha: marca de estado, barra, porcentagem e ✕; a pílula com o nome da etapa quando o progresso tem 160 px (Title zone) |
| Painel redimensionado | a linha se refaz no próximo desenho, sem quebrar, enquanto o painel tiver 40 colunas ou mais (Key decision 3) |
| Painel largo | o mesmo que no desktop |
| Barra com agentes | a seta de recolher entra na mesma linha, e as linhas dos agentes ficam embaixo da barra, como no desktop |

Os tipos do motor (2.1.308) dizem que a faixa acima do prompt só existe no terminal e no desktop. A extensão 2.1.310 desenha a faixa mesmo assim, como mostra a captura do autor e o código da extensão. Vale o que a extensão faz.

1. Pixels por coluna no VS Code - 12 px no Windows e 8 px nos outros sistemas (era 6,5 px na 1.4.0; trocado em 2026-10-25, na 1.4.1). O "─" vem de uma fonte reserva, larga no Windows (cerca de 12 px, tirado da captura do autor) e mais estreita fora dele (8 px, estimado, não medido). Errar para menos só deixa a barra mais curta, com sobra depois do ✕; errar para mais quebra a linha.

### Scattered fill

**Delivers** o preenchimento de pontos espalhados na barra do plano, no desktop e no VS Code. **Status: clear.**

| State | O que a pessoa vê |
|---|---|
| Em andamento, cintilar ligado | pontos espalhados, rarefeitos no começo e densos perto da pílula, com a onda de luz correndo e o cintilar |
| Cintilar desligado (`twinkle` em `/config`) ou movimento reduzido no sistema | os mesmos pontos, parados |
| Aguardando, erro, concluído | os pontos na cor do estado, e a onda na metade da velocidade, como hoje |
| Um passo termina e o preenchimento cresce | nenhum ponto muda de lugar. O trecho novo ganha pontos, e os de trás podem rarear e enfraquecer, porque o mais denso fica sempre perto da pílula |
| Sob o título | sempre o nível mais fraco, em qualquer ponto do progresso (Key decision 4) |
| A faixa é redesenhada por outro motivo | os mesmos pontos, sem piscar nem recomeçar |
| Duas barras ao mesmo tempo | cada uma com pontos diferentes |
| Tema claro | como o preenchimento de hoje no tema claro |
| Barra mais larga (1.400 px) | a figura abaixo do limite do `Svg` (Key decision 4) |

## Migration

Nenhum dado salvo muda: as barras salvas por sessão voltam já com o desenho novo. A mudança sai numa versão menor nova do plan-progress, e quem não atualizar continua com o desenho de hoje. Na mesma versão, os dois READMEs passam a descrever a linha nova. Hoje eles listam o título como uma peça da linha, ao lado da barra, e dizem que o preenchimento do terminal é a grade de pontos do desktop. O desktop deixa de ter essa grade.

## Sources

- Duas capturas de tela do autor, 2026-10-25, enviadas na conversa e fora do repositório. Na do desktop, as linhas dos agentes têm pontos espalhados e o nome dentro. Na do VS Code, a barra está quebrada em três linhas e a pílula mostra "3".
- [Desenhos A, B e C](plan-bar-title-inside-mockups.jpg) - os três jeitos de pôr o título dentro da barra, em 4%, 40% e 85%, no desktop e no VS Code estreito. São imagens paradas, sem a onda nem o cintilar. O autor escolheu o B.
- Tipos do motor de mods, Claude Code 2.1.308 - o limite de 131.072 caracteres de um `Svg`, e a largura de um `Svg` em pixels CSS.
- Extensão do Claude Code para VS Code 2.1.310 - desenha a faixa acima do prompt, conta as colunas pelo avanço da fonte do editor e quebra as linhas que não cabem.
- WPC-246 - o precedente da faixa no VS Code.
