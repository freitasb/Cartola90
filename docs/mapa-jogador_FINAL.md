# Mapa do Jogador — atributos idealizados (completo)

> **O que é este documento.** O mapa COMPLETO e idealizado de tudo que um jogador
> pode ter no jogo — mesmo o que não entra no dia 1. Serve de continuidade: desenhar
> uma mecânica lá na frente olhando o mapa inteiro, sem retrabalho.
>
> **Idealizar ≠ metrificar.** Aqui a gente diz *quais* atributos podem existir e *o que
> cada um significa*. No recorte atual, os 6 atributos de linha e os 6 de goleiro
> são **cadastrados diretamente em 0–100**; eles não são calculados a partir dos
> atributos finos deste mapa. Os finos permanecem como direção futura e só serão
> metrificados quando uma mecânica realmente precisar deles.
>
> **Etiqueta "quando entra":**
> - `[D1]` = dia 1 (o motor inicial já usa)
> - `[G2]` = quando chegar o degrau 2 (duelos individuais)
> - `[G3]` = quando chegar o degrau 3 (fases/posicionamento estilo FM)
> - `[H]` = horizonte / algum dia (não amarrado a um degrau específico)
>
> As etiquetas `[G2]`/`[G3]`/`[H]` foram preservadas como **mapa histórico de
> profundidade/evolução**, não como roadmap atual. A Fase A não redesenhou esses
> atributos futuros; eles só devem ser refinados quando a mecânica correspondente
> chegar.

---

## Família 1 — Técnicos
O que o jogador sabe fazer com a bola.

| Atributo | O que mede | Quando |
|---|---|---|
| Finalização | Converter uma chance clara em gol | `[G2]` |
| Passe (curto) | Acerto do passe de curta distância | `[G3]` |
| Passe (longo) | Acerto do lançamento / bola longa | `[G3]` |
| Drible | Superar o marcador com a bola no pé | `[G3]` |
| Cabeceio | Disputa e finalização de cabeça | `[G3]` |
| Cobrança de bola parada | Faltas, escanteios, pênaltis | `[H]` |
| Cruzamento | Qualidade da bola alçada na área | `[G3]` |
| Desarme | Tirar a bola do adversário sem falta | `[G2]` |

## Família 2 — Físicos
O corpo do jogador.

| Atributo | O que mede | Quando |
|---|---|---|
| Velocidade | Quão rápido corre | `[G3]` |
| Resistência (fôlego) | Aguentar o jogo sem cair de rendimento | `[G2]` (liga com cansaço) |
| Força | Ganhar disputa física, proteger a bola | `[G3]` |
| Agilidade | Mudança de direção, explosão curta | `[H]` |
| Impulsão | Altura do salto (liga com cabeceio) | `[H]` |

## Família 3 — Mentais / Psicológicos
A cabeça do jogador. **ATENÇÃO: base científica rala** (ver ADR de heurística).
Aqui mora o que atrai o Brian (odeia banco, treme na decisão), mas quase nada vira
fórmula pronta — é estudo + regra inventada, e precisa ser marcado como heurística.

> **PRINCÍPIO IMPORTANTE (descoberto na sabatina):** um comportamento observável
> (ex: "tomar amarelo", "engasgar no pênalti") **quase nunca sai de um atributo só**
> — sai de uma COMBINAÇÃO, e às vezes a combinação muda de sinal dependendo de outro
> atributo. Exemplo do Brian: "mental fraco" gera comportamentos OPOSTOS conforme o
> jogador é introspectivo (engasga, some) ou extrovertido (reclama, leva cartão).
> Logo, ao chegar no degrau que gera cartão/pressão, NÃO derivar de um atributo
> único — usar Temperamento + Agressividade + Disciplina + o eixo de personalidade
> abaixo + a situação. (É como o FM faz.) Plantar a bandeira agora, construir no
> degrau certo — não modelar essa precisão hoje (o dia 1 nem gera cartão).

| Atributo | O que mede | Quando |
|---|---|---|
| Frieza / Compostura | Render sob pressão (pênalti, minuto final) | `[H]` heurística |
| Concentração | Não "desligar" e cometer erro bobo | `[H]` heurística |
| Decisão | Escolher a jogada certa na hora | `[G3]` heurística |
| Liderança | Influência no moral do grupo | `[H]` heurística |
| Determinação | Não desistir quando o jogo está difícil | `[H]` heurística |
| Temperamento | Propensão a cartão / descontrole | `[H]` heurística |
| **Eixo introspecção↔extroversão** | **Personalidade que MODULA como os outros mentais se expressam (mesmo "mental fraco" → engasga vs reclama). Candidato a atributo próprio, não derivado.** | `[H]` heurística |

## Família 4 — Táticos
Como o jogador se encaixa no coletivo.

| Atributo | O que mede | Quando |
|---|---|---|
| Posicionamento (defensivo) | Estar no lugar certo sem a bola, defendendo | `[G3]` |
| Posicionamento (ofensivo) | Movimentação para receber / criar espaço | `[G3]` |
| Marcação | Anular o adversário direto | `[G2]` |
| Visão de jogo | Enxergar o passe que ninguém vê | `[G3]` |
| Disciplina tática | Cumprir a função pedida pelo técnico | `[G3]` |

## Família 5 — Estado / Condição
Não é "quão bom o jogador é", é "como ele está AGORA". Muda a cada partida.
No recorte atual, lesão, suspensão e demais estados ricos permanecem como evolução futura.

| Atributo | O que mede | Quando |
|---|---|---|
| Forma física | Estar "ligado" ou "apagado" na fase atual | `[H]` |
| Moral | Ânimo (afeta rendimento) | `[H]` heurística |
| Cansaço (na partida) | Queda de rendimento ao longo do jogo | `[G2]` |
| Lesão (estado) | Apto / lesionado; poderá afetar disponibilidade e pausas obrigatórias quando a mecânica existir | `[H]` |
| Suspensão (estado) | Apto / suspenso (cartões acumulados) | `[H]` |
| Idade | Afeta pico/declínio (curva de idade) | `[H]` (base forte) |

## Família 6 — Goleiro
Atributos próprios; goleiro não usa os de linha da mesma forma.

| Atributo | O que mede | Quando |
|---|---|---|
| Reflexo | Defender o chute à queima-roupa | `[G2]` |
| Posicionamento (gol) | Estar no ângulo certo, diminuir o gol | `[G2]` |
| Saída do gol | Sair bem no cruzamento / mano a mano | `[G3]` |
| Jogo com os pés | Participar da construção | `[H]` |
| Reposição | Qualidade do lançamento/tiro de meta | `[H]` |

---

## Conhecimento posicional — dado à parte dos atributos (atualizado na Fase A)

Não é atributo (não é "quão bom ele é") nem estado. Representa **onde o jogador
sabe atuar**.

O Player possui:
- uma `NaturalPosition`;
- zero ou mais `SecondaryPositions`.

A posição natural não pode se repetir entre as secundárias.

**Improvisação não é persistida.** Ela é derivada da escalação: o jogador está
improvisado quando ocupa um slot que não corresponde nem à posição natural nem a
uma posição secundária.

No recorte inicial:
- não existe penalização numérica por aptidão;
- a posição continua importando para montagem da escalação;
- a CPU prioriza Natural/Secundária e improvisa somente quando necessário;
- o Manager Human pode improvisar jogador de linha, com aviso da UI;
- o slot GK exige goleiro real.

Uma penalização matemática futura continua possível, mas seus valores seriam
**heurística de game design** e só devem ser desenhados quando a mecânica entrar.

> **Vínculo com clube (sincronização Fase A).** Player é entidade independente e
> pode existir sem clube. O vínculo atual é conceitualmente representado por
> `PlayerClubAssignment` (`PlayerId`, `ClubId`, `StartedAt`, `EndedAt nullable`),
> com no máximo um clube atual por jogador. Contrato completo é evolução futura.

---

## O QUE O RECORTE INICIAL REALMENTE USA

Tudo acima continua sendo o mapa idealizado/futuro. O MatchEngine inicial trabalha
com atributos **grossos cadastrados diretamente** e deriva deles a qualidade do
jogador; não existe hoje uma etapa "fino → gordo".

### Jogador de linha — 6 atributos cadastrados diretamente `[D1]`
| Atributo `[D1]` | Entra em |
|---|---|
| Ritmo | Ataque **e** Defesa |
| Finalização | Ataque |
| Drible | Ataque |
| Passe | Ataque |
| Defesa | Defesa |
| Físico | Ataque **e** Defesa |

Qualidades derivadas (não cadastradas):

`QualidadeAtaque = (Finalização + Drible + Passe + Ritmo + Físico) / 5`

`QualidadeDefesa = (Defesa + Ritmo + Físico) / 3`

> Ritmo e Físico contam nos DOIS lados: time rápido/forte é melhor atacando E
> defendendo.

### Goleiro — 6 atributos cadastrados diretamente `[D1]`
| Atributo `[D1]` | Entra em |
|---|---|
| Mergulho | Defesa |
| Manuseio | Defesa |
| Chute | Defesa (distribuição fica pra depois) |
| Reflexos | Defesa |
| Velocidade | Defesa |
| Posicionamento (gol) | Defesa |

Qualidade derivada:

`QualidadeGoleiro = média dos 6 atributos`

A defesa coletiva usa a mistura já fechada no documento de decisões:

`DefesaTime = 0,70 × DefesaLinha + 0,30 × QualidadeGoleiro`

O **traço "cobrador de falta/pênalti"** (goleiro tipo Rogério Ceni) continua como
bandeira futura: campo próprio quando existir evento de bola parada; não faz parte
do recorte inicial.

> **Importante:** os atributos finos das Famílias 1–6 continuam preservados como
> mapa futuro, mas não são a origem matemática dos 12 atributos cadastrados hoje.

---

## Pendências deste mapa
- ✅ A fatia atual está fechada: 6 atributos de linha + 6 de goleiro cadastrados diretamente.
- ✅ `QualidadeAtaque`, `QualidadeDefesa` e `QualidadeGoleiro` são derivadas, não cadastradas.
- ✅ Força de ataque/defesa do time é derivada dos jogadores, não atributo direto do clube/time.
- ✅ Conhecimento posicional atual = `NaturalPosition` + `SecondaryPositions`; improvisação é derivada.
- ✅ Penalização numérica por improvisação está desligada e não precisa ficar pré-construída.
- ⬜ Etiquetas dos atributos finos seguem como mapa idealizado e podem ser refinadas apenas quando cada mecânica futura for desenhada.
- ⬜ ADR/notinha de heurística quando uma mecânica psicológica ou outro parâmetro de game design realmente for implementado.
