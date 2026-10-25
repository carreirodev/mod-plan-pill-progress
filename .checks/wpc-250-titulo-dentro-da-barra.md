# Título dentro da barra do plano e pontos espalhados (WPC-250)

Perfil: `light` (sem `AGENTS.md`, vale o padrão), handoff ligado, orçamento de 150k. O design é **binding** para a interface, que é o caso para o qual o perfil `ui` existe; quem quiser a comparação tela a tela do `ui` pode subir o perfil.

Sources:

- `.tasks/wpc-250-titulo-dentro-da-barra.md` - o registro da decisão: os critérios 1 a 29, que viram C1 a C29 aqui, com o mesmo número; o que fica de fora; Swept; Impact; e os 8 pontos em aberto, cada um com o padrão que esta construção usa
- `.design/plan-bar-title-inside.md` - **binding para a interface**: Key decisions 1 a 6 (a 6, desde a 1.4.2) e as tabelas de estado das fatias Title zone, vscode row e Scattered fill
- `.design/plan-bar-title-inside-mockups.jpg` - **binding para a interface**: a variante B (a marca fora e à esquerda; o título claro dentro da barra, à esquerda; a pílula depois do trecho do título; a porcentagem e o ✕ fora e à direita; no VS Code estreito, a mesma ordem, com o nome na pílula). Desde a 1.4.2 a porcentagem vai dentro da pílula (pedido do autor, 2026-10-25, depois de ver o "70%" e o ✕ caírem para a linha de baixo no VS Code)
- [WPC-250](https://linear.app/wpcsolutions/issue/WPC-250/plan-progress-title-inside-the-plan-bar-and-scattered-dots-on-desktop) - o pedido e a estimativa (M)
- Extensão do VS Code 2.1.310, lida em 2026-10-25 (memória do projeto): a faixa existe na superfície `vscode`, linhas que não cabem quebram, e `bodyColumns` vem do avanço da fonte do editor

## Como rodar as provas

Da raiz do repositório, uma vez a cada build:

```bash
cd plugins/plan-progress/tests && TYPESCRIPT="C:/Users/Eduardo/AppData/Local/Programs/Microsoft VS Code/e646868227/resources/app/extensions/node_modules/typescript/lib/typescript.js" node compile.cjs ../hooks/register.tsx register.mjs
```

Depois, de `plugins/plan-progress/tests`, cada prova é `ONLY=<caso> node regress.mjs`. Várias de uma vez: `ONLY=a,b,c node regress.mjs`. A suíte inteira é `node regress.mjs` e termina em `all passed`.

## Out of scope

- O título dentro da barra no terminal - o autor pediu só desktop e VS Code
- O cabeçalho "From the plan-progress plugin" e a moldura da faixa no VS Code - quem desenha é a extensão
- Conferir o app de celular - usa o caminho de desenho do desktop, mas nada aqui é verificado lá
- O tema do Omarchy no desktop - o critério 10 fala dos dois temas do app; uma cor de destaque do Omarchy pode ser qualquer uma, e o contraste do título sobre ela não é provado

## Landing

Mexe em `hooks/register.tsx` (o desenho da barra no desktop, `drawTrack`/`trackSvg`, e o `AbovePrompt` fora do terminal), em `tests/engine.mjs` (o motor simulado passa a desenhar a superfície `vscode`) e em `tests/regress.mjs`. Reaproveita `labelWidth`/`fitLabel` para medir e cortar o título, `SCHEMES` para a cor do título e da página, `streamHash`/`seedOf` para sortear os pontos, as constantes da onda e do cintilar (só lidas, nunca mudadas, porque o terminal também as usa) e o padrão de dicas da camada de hover.

| One-way door | Literal shape | Alternative rejected |
| --- | --- | --- |
| None - nada salvo nem consumido fora do mod muda; o registro `Plan` e as barras salvas ficam como estão, e uma versão nova desfaz tudo | | |

Decisões tomadas na construção (reversíveis, registradas antes do código que as fecha):

| Decisão | Forma | Alternativa rejeitada |
| --- | --- | --- |
| A barra deita a página do tema por baixo do trilho, como as linhas dos agentes | `<rect class="pg">` dentro da cápsula; `.pg` com `SCHEMES.dark.page`, e `SCHEMES.light.page` no tema claro | o trilho translúcido sobre o fundo do app, como hoje: o contraste do título (C10) dependeria de uma cor que o mod não conhece |
| A dica do título cabe na barra, como as outras dicas | o texto da dica é `fitLabel(título, W − 16, 11)`; um título mais largo que a barra inteira aparece cortado na dica, e inteiro no texto alternativo | uma dica que passa da barra: a camada de hover é do tamanho da barra e cortaria a dica de qualquer jeito |
| O corte do título pode deixar menos de 3 letras | `fitLabel` ganha um mínimo opcional (padrão 3, como hoje); o título usa 1 | o mínimo de 3 de hoje: num trecho estreito, "WWW…" passa da margem de 12 px e entra no progresso |
| O cintilar vai nos próprios pontos | os pontos de cada uma das 24 classes do cintilar formam um `path`, desenhado uma vez em `<defs>` e usado duas vezes: pelo brilho de base e pela classe que cintila | os 24 padrões de ladrilho de hoje sob uma máscara de pontos: o ladrilho repete a grade inteira, e os pontos teriam de vir duas vezes |

## Checks

### S1 - Título dentro da barra · 2 files · 580 KB · ~145k

**C1** - (1.4.2: a porcentagem saiu da linha e foi para a pílula, C31.) No desktop e no VS Code, a linha de cada barra traz, nesta ordem, a marca de estado, a figura da barra, a seta de recolher (só quando há agentes para recolher, como hoje) e o ✕, e nenhum `Text` da linha traz o título nem a porcentagem
Proof: `ONLY=row_is_mark_bar_close node regress.mjs`

**C2** - O título aparece dentro da figura, num texto em `x = 12`; o trecho do título tem a mesma largura em todas as barras mostradas, a que o título mais longo pede (largura medida no Chromium + 24 px, até 15% acima disso pela folga da medida), e no máximo `floor(0,3 × W)`
Proof: `ONLY=title_zone_fits_the_longest_title node regress.mjs`

**C3** - Um título que não cabe no trecho aparece cortado com "…", e a camada de hover tem uma área sobre o trecho do título que mostra uma dica com o título inteiro (numa barra que comporta a dica); um título que cabe não tem essa área nem essa dica
Proof: `ONLY=cut_title_shows_whole_on_hover node regress.mjs`

**C4** - Quando `0,3 × W` não comporta "Abc…" com as margens (no VS Code fora do Windows, 30 colunas, barra de 184 px), a barra não tem trecho de título: nenhum texto do título na figura, e o preenchimento e os marcos usam a barra inteira; com 31 colunas (192 px), o trecho existe; nos dois casos o texto alternativo começa pelo título. (1.4.2: a barra mais estreita do desktop passou a 220 px e sempre tem o trecho; até a 1.4.1 o caso era o desktop com 41 e 42 colunas)
Proof: `ONLY=narrow_bar_has_no_title_zone node regress.mjs`

**C5** - Com uma fração f do plano feita, a ponta do preenchimento fica em `zona + f × (W − zona)` (±0,1 px), nenhuma parte da pílula começa antes do fim do trecho, e os marcos e as áreas dos marcos que respondem ao mouse ficam entre o fim do trecho e a ponta direita
Proof: `ONLY=progress_runs_from_the_title_zone_to_the_end node regress.mjs`

**C6** - Um plano sem nenhum passo concluído tem o preenchimento cobrindo exatamente o trecho do título, no nível mais fraco (a primeira parada do gradiente do preenchimento), e a borda esquerda da pílula no fim do trecho
Proof: `ONLY=fresh_plan_fills_the_title_zone node regress.mjs`

**C7** - Quando um passo termina, a pílula desliza do lugar antigo para o novo sem que sua borda esquerda passe antes do fim do trecho, e o preenchimento desliza a partir de um ponto dentro do progresso
Proof: `ONLY=pill_glides_inside_the_progress node regress.mjs`

**C8** - Quando entra ou sai uma barra que muda a largura do trecho, as outras barras são redesenhadas com o trecho novo e sem nenhum deslize (`<animate>`/`<animateTransform>`)
Proof: `ONLY=title_zone_change_redraws_without_a_glide node regress.mjs`

**C9** - Um plano concluído tem a barra verde, o título no trecho dele e a pílula na ponta direita com o tempo total
Proof: `ONLY=finished_bar_keeps_its_title node regress.mjs`

**C10** - Nos quatro estados e nos temas claro e escuro, o título tem contraste de pelo menos 4,5:1 contra o trilho sobre a página do tema, contra o preenchimento sob ele e contra os pontos no brilho mais forte que alcançam sob ele (onda no pico, cintilar incluído)
Proof: `ONLY=title_reads_in_every_state_and_theme node regress.mjs`

**C11** - Com 160 px ou mais de trecho do progresso, a pílula é a cápsula com o nome da etapa; com menos, é uma pílula de 46 px só com a porcentagem (até a 1.4.1, a bolinha redonda com o número da etapa), no desktop (41 e 40 colunas) e no VS Code fora do Windows (70 e 32 colunas)
Proof: `ONLY=pill_name_needs_160_px_of_progress node regress.mjs`

**C12** - Uma linha de agente esconde a palavra da ferramenta numa barra de 356 px e a mostra numa de 364 px: o limite de 360 px continua o mesmo
Proof: `ONLY=strip_tool_word_keeps_its_360_px node regress.mjs`

**C13** - As linhas dos agentes ficam na mesma coluna da barra, logo abaixo dela, com a mesma largura
Proof: `ONLY=agent_strips_line_up_under_the_bar node regress.mjs`

**C14** - As áreas de hover dos marcos ficam nas posições novas, `zona + k/total × (W − zona)`, cada uma com a sua dica, e o relógio da pílula fica sobre a pílula no lugar novo
Proof: `ONLY=hover_marks_and_clock_follow_the_progress node regress.mjs`

### S2 - Linha única no VS Code · 3 files · 604 KB · ~151k

**C15** - (1.4.1, renegociado com o autor em 2026-10-25: a 1.4.0 usava 6,5 px por coluna, e no VS Code do autor a barra ficou com metade da faixa. 1.4.2, de novo com o autor no mesmo dia: os 12 px da 1.4.1 eram uma estimativa; medida na captura dele, a coluna tem cerca de 10 px, e a linha quebrava, jogando a porcentagem e o ✕ para baixo.) Na superfície `vscode` com `bodyColumns` = N, uma coluna vale 10 px no Windows (instalação numa letra de unidade) e 8 px nos outros sistemas. A barra tem `floor(N × coluna)` menos o resto da linha em colunas (7: a marca, dois espaços e o ✕; 13 com a seta e o espaço dela), até 1.400 px, sempre que isso dá 120 px ou mais; senão, fica em 120 px. A divisória entre as barras nunca passa de N colunas; com `bodyColumns` = 0 vale N = 100. Conferido nos dois sistemas, com N = 20, 30, 40, 60, 100, 200 e 0, e com a seta N = 25, 40, 60 e 100
Proof: `ONLY=vscode_row_fits_its_columns node regress.mjs`

**C32** - (1.4.2) A linha da barra não pode quebrar (`flexWrap="nowrap"`), e no VS Code a coluna da barra pode encolher (`flexShrink` 1, `minWidth` 0): onde as colunas forem mais estreitas que a conta, a figura encolhe em vez de o ✕ cair para a linha de baixo. No desktop a coluna continua do tamanho da figura (`flexShrink` 0)
Proof: `ONLY=vscode_row_cannot_break node regress.mjs`

**C30** - (1.4.1) No VS Code a barra vai sozinha na sua caixa: sem a camada de hover, sem o relógio, sem nenhuma caixa posicionada por cima de outra na faixa, e a seta de recolher é um botão de texto; o desktop continua com os três. A extensão ignora a posição absoluta e o hover, e desenhava a camada ao lado da barra, com a largura dela
Proof: `ONLY=vscode_bar_goes_alone node regress.mjs`

**C16** - No desktop a barra tem `max(120, min(1400, max(320, 8 × N) − 100 − 40 se há seta))` px, para N = 30, 100, 120 com a seta e 300 (1.4.2: os 40 px da porcentagem fora da barra passaram para a barra)
Proof: `ONLY=desktop_row_counts_8_px_a_column node regress.mjs`

**C31** - (1.4.2) A porcentagem fica na ponta direita da pílula, a mesma do texto alternativo, em andamento, aguardando e com erro, e também na cópia da pílula que mostra o relógio; com o progresso curto, a pílula traz só a porcentagem. A pílula de um plano concluído mostra o tempo total e nenhuma porcentagem, curta ou não. O espaço da porcentagem comporta dois algarismos, então a pílula tem a mesma largura a 5% e a 50%
Proof: `ONLY=pill_carries_the_percent_until_done node regress.mjs`

**C17** - No VS Code do autor, na largura de painel que ele usa, cada barra de plano aparece numa linha só, com o ✕ ao lado da barra, e a pílula mostra o nome da etapa e a porcentagem quando o progresso tem 160 px ou mais
Proof: manual - o autor confere na extensão depois de `/reload-plugins` (fica em Blocked on me; sem isso, o WPC-250 para em In Review)

### S3 - Pontos espalhados · 2 files · 580 KB · ~145k

**C18** - O preenchimento é feito de pontos de 2 × 2 px nas posições da grade de 3 px (`x = 3c`, `y = 1 + 3r`, 7 linhas). Num preenchimento de progresso com 90 colunas ou mais, em janelas de 30 colunas inteiras dentro do preenchimento: a primeira tem entre 10% e 35% das posições com ponto; as que começam a partir de dois terços do caminho até a ponta têm entre 55% e 80%; nenhuma tem 100%
Proof: `ONLY=fill_dots_scatter_on_a_ramp node regress.mjs`

**C19** - Sob o trecho do título, no máximo 30% das posições têm ponto, com o brilho de base do começo do preenchimento e sem cintilar, a 0%, 50% e 100% do plano
Proof: `ONLY=title_zone_dots_stay_faint node regress.mjs`

**C20** - O mesmo plano, na mesma largura e no mesmo estado, desenhado de novo, tem os pontos nas mesmas posições, inclusive depois de o mod recarregar
Proof: `ONLY=fill_dots_stay_put node regress.mjs`

**C21** - Quando o preenchimento cresce de f1 para f2 > f1, todo ponto do desenho em f2 dentro do preenchimento de f1 também estava no desenho em f1, na mesma posição
Proof: `ONLY=growing_fill_adds_no_dot_behind node regress.mjs`

**C22** - Duas barras com ids diferentes, na mesma largura e no mesmo progresso, não têm os pontos todos nas mesmas posições
Proof: `ONLY=two_bars_scatter_their_own_dots node regress.mjs`

**C23** - Com o cintilar ligado, a figura tem a onda (um padrão que desliza uma coluna por passo) e o cintilar sobre os pontos (cada ponto em exatamente uma das 24 classes, cada classe com a sua linha do tempo no ritmo de 50 ms); a onda corre na velocidade de hoje quando a barra está em andamento e na metade dela aguardando, com erro ou concluída
Proof: `ONLY=fill_stream_still_dots_wave_and_flicker node regress.mjs`
Proof: `ONLY=done_bar_streams_at_half_speed node regress.mjs`

**C24** - Com o cintilar desligado, a figura não tem animação nos pontos (nem onda que desliza, nem classe de cintilar); com ele ligado, a folha de estilo para a onda e o cintilar sob `prefers-reduced-motion: reduce`
Proof: `ONLY=twinkle_off_draws_the_fill_still node regress.mjs`
Proof: `ONLY=fill_stream_still_dots_wave_and_flicker node regress.mjs`

**C25** - A figura da barra (sem a camada de dicas) tem no máximo 131.072 caracteres na barra de 1.400 px, nos quatro estados, com o cintilar ligado, num plano com marcos o bastante para encher a cota dos marcos
Proof: `ONLY=widest_bar_stays_under_the_svg_limit node regress.mjs`

**C26** - No tema claro, os pontos têm as cores do preenchimento da 1.3.0 em cada estado (congeladas no teste a partir do build da 1.3.0), e nenhuma regra do tema claro muda a cor deles
Proof: `ONLY=light_theme_dots_keep_their_colours node regress.mjs`

### S4 - O que não muda e a versão · 5 files · 593 KB · ~148k

**C27** - No terminal, a linha e a barra são desenhadas como na 1.3.0: a árvore e cada célula das figuras do terminal, nos quatro estados, nas larguras 40, 120 e 200, em tela cheia ou não, com o cintilar ligado e desligado e com uma barra com agentes, batem com o hash tirado do build da 1.3.0; e os casos do terminal de `regress.mjs` passam sem mudança
Proof: `ONLY=terminal_band_is_drawn_as_in_1_3_0 node regress.mjs`
Proof: `ONLY=terminal_bar_is_a_raster_that_fits,terminal_pill_has_round_ends_where_they_draw,terminal_track_and_strips_are_capsules_where_they_draw,terminal_animates_by_blits,narrow_terminal_row_fits_the_band,folded_bar_in_the_terminal,wide_title_keeps_its_columns,terminal_resize_draws_the_head_in_place,twinkle_off_terminal_fill_holds node regress.mjs` e `git diff b0f064f -- plugins/plan-progress/tests/regress.mjs` sem mudança nesses casos

**C28** - `plugin.json` diz `1.4.2` ou posterior (a 1.4.2 pôs a porcentagem na pílula), e os dois READMEs descrevem a linha do desktop e do VS Code como marca de estado, barra com o título dentro e a porcentagem na pílula ("etiqueta" no README em português), e ✕; nenhum dos dois diz mais que o preenchimento do terminal é a grade de pontos do desktop
Proof: `ONLY=version_and_readmes_describe_the_new_row node regress.mjs`

**C29** - Barras salvas exatamente como a 1.3.0 salva (o texto guardado, congelado no teste) voltam numa sessão retomada já com o desenho novo, e o plano restaurado é o mesmo que estava salvo, sem conversão
Proof: `ONLY=saved_bars_from_1_3_0_draw_the_new_way node regress.mjs`

## Casos de hoje que mudam com o desenho

O Impact da tarefa autoriza: "os casos de `regress.mjs` que conferem a grade de pontos (o padrão regular e a sua máscara) e a coluna do título no desktop mudam junto com o desenho".

- `fill_stream_still_grid_wave_and_flicker` vira `fill_stream_still_dots_wave_and_flicker`: os pontos parados, a onda que desliza uma coluna por passo sobre um padrão que a cobre uma vez, cada ponto numa classe só, as linhas do tempo no ritmo de 50 ms, tudo atrasado por `--f`/`--w`, e o movimento reduzido parando tudo continuam conferidos; sai só a grade regular
- `fill_stream_same_size_at_any_width` sai: a Key decision 4 diz que o desenho ponto a ponto cresce com a largura; quem guarda o limite agora é C25
- `fill_stream_reaches_the_fill_during_a_glide_back`: continua conferindo que o preenchimento que encolhe nunca fica sem pontos; muda só onde ele lê os pontos
- `twinkle_off_draws_the_fill_still`: lia `url(#dg)`; passa a ler os pontos. O teto de 8.000 caracteres fica, contado sem o caminho dos pontos: ele guardava o cintilar fora da figura parada, e o caminho dos pontos cresce com a largura (Key decision 4; a figura parada de 780 px tem 8.845 caracteres, 5.322 deles de pontos)
- `done_bar_streams_at_half_speed`: lia os 24 retângulos de ladrilho; passa a ler os usos das classes, um por classe que tem pontos. O plano ganha um passo feito, porque um plano novo só tem os pontos ralos do trecho do título
- `marks_show_in_the_light_theme`: passou sem mudança
- `fill_stream_still_dots_wave_and_flicker`, além de trocar a grade pelos pontos: o brilho de base agora começa fraco, acima de zero (Key decision 4: os pontos sob o título ficam no nível mais fraco, visíveis), em vez de começar em zero; e as classes de cintilar contadas são as que têm pontos (20 ou mais), porque um trecho curto pode deixar alguma classe sem ponto
- `fill_stream_reaches_the_fill_during_a_glide_back`: o ponto mais à direita pode ficar até 6 px antes da borda antiga, porque os pontos são espalhados e a última coluna pode não ter nenhum
- `done_bar_streams_at_half_speed`, depois do Verifier: passou a fixar a volta da onda em andamento em 11,65 s, a da 1.3.0, e não só a razão de 2 entre parada e em andamento
- `more_agents_line_fits_a_narrow_strip`: usava um título longo ao lado da barra para estreitar a linha de agentes a 120 px; sem a coluna do título, a linha mais estreita é a do VS Code com 30 colunas (120 px, a mesma largura), e o desktop com 40 colunas fica no lugar das 60
- Na 1.4.2 (a porcentagem na pílula e a linha do VS Code que não quebra): `row_is_mark_bar_percent_close` virou `row_is_mark_bar_close`; as larguras fixadas em `narrow_bar_has_no_title_zone`, `title_zone_change_redraws_without_a_glide` (820 → 860), `finished_bar_keeps_its_title` (820 → 860), `pill_name_needs_160_px_of_progress`, `strip_tool_word_keeps_its_360_px` (67/68 → 62/63 colunas), `vscode_row_fits_its_columns` e `desktop_row_counts_8_px_a_column` seguiram a reserva nova; `long_pill_labels_keep_the_pill_to_320_px` passou sem mudança no teste, depois de o teto de 320 px passar a incluir a porcentagem

## Swept

- validation: C3, C4. Títulos de qualquer tamanho, com letras largas e emoji, são medidos com `labelWidth` e cortados com `fitLabel`, como os rótulos da pílula
- failure modes: C25. Uma figura acima do limite faz o motor pular a faixa inteira
- idempotency and retry: C20
- authorization: n/a - desenho local, sem quem chame
- concurrency and ordering: C8. Duas atualizações ao mesmo tempo continuam existing: cada uma parte da lista mais recente guardada
- data lifecycle: C29
- external-dependency failure: existing - com `bodyColumns` 0 o mod usa 100 colunas (`e.props.bodyColumns || 100`), e C15 confere isso também na superfície `vscode`
- state transitions: C6, C7, C9, C10, C23
- observability: existing - o motor registra quando pula a faixa; nada novo

## Handoff

Pela conta do orçamento, cada fatia sozinha já chega perto de 150k, porque todas leem os mesmos dois arquivos grandes (`register.tsx` com 260 KB e `regress.mjs` com 319 KB; o tamanho do módulo já está no WPC-243). A conta não distingue os agrupamentos, então o corte fica onde o código lido muda: o lote 1 é S1 + S2 (a geometria de `drawTrack` e o `AbovePrompt` fora do terminal, mais a superfície `vscode` do motor simulado); o lote 2 é S3 + S4 (o desenho dos pontos, a versão e os READMEs). O lote 1 é construído nesta sessão; a passagem para um agente novo antes do lote 2 só acontece se o contexto daqui não comportar o lote 2.

- Onde o corte caiu: o lote 1 (C1-C9, C11-C16; C10 ficou para o lote 2, porque depende do brilho dos pontos sob o título) fechou em `82d5844`. O lote 2 seguiu nesta mesma sessão, sem passagem
- O que o usuário decidiu durante a construção: nada novo
- O que foi abandonado: os pontos como máscara sobre os ladrilhos de hoje (o ladrilho repete a grade inteira e os pontos viriam duas vezes); o caminho único por classe ficou no lugar
