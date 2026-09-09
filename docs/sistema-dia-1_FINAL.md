# O Sistema do Dia 1 — como ficou
### Estado consolidado após a Fase A e a Fase B1

> **O que é este documento.** A foto técnica do **núcleo inicial fechado** do Manager de
> Futebol, pronta para orientar implementação. Ele descreve **como o sistema deve existir**
> depois que os marcos iniciais forem construídos; o `roadmap-jogo-futebol.md` define a
> ordem em que cada parte entra.
>
> **Importante sobre “Dia 1”.** Aqui “Dia 1” significa o **recorte inicial oficial do
> produto**, não “tudo precisa existir no primeiro commit” e nem “Marco 1”. Por exemplo:
> RabbitMQ faz parte deste núcleo consolidado, mas **não existe no Marco 1**; ele só entra
> quando surge a primeira fronteira real de integração, com `PartidaEncerrada`.
>
> **O que este documento NÃO é.** Não é histórico de decisões — esse papel pertence ao
> `decisoes-jogo-futebol.md`. Não é plano de execução — esse papel pertence ao roadmap.
> Também não antecipa Competition, autenticação, eventos ricos, tática avançada ou cloud.
>
> **Regra de leitura.** Quando este arquivo mencionar algo futuro, é apenas uma bandeira de
> continuidade. O que está descrito como parte do núcleo atual já foi fechado e não deve ser
> redesenhado durante a implementação sem uma contradição real.

---

## 1. O retrato do sistema num relance

O recorte inicial é um Manager de futebol com **dois clubes**:

- um controlado por um `Manager` humano;
- outro controlado por um `Manager` CPU.

O `Game` é o processo principal. Ele é dono do domínio necessário para preparar os times,
criar a partida, executar a simulação, manter o estado vivo, persistir o histórico e expor
as informações ao React.

O CPU **não é outro serviço**. Sua inteligência inicial vive dentro do Game, no componente
`Manager AI`.

A partida acontece **de verdade no servidor**, minuto a minuto. Uma partida normal possui:

- 90 minutos de jogo;
- 6 minutos reais de bola correndo;
- 1º tempo de 3 minutos reais;
- intervalo de 30 segundos reais;
- 2º tempo de 3 minutos reais;
- equivalência inicial de 1 minuto de jogo = 4 segundos reais.

O React envia comandos por HTTP. O Game envia atualizações ao vivo por SignalR.

RabbitMQ só aparece quando existe uma responsabilidade real fora do Game. O primeiro fluxo é:

```text
Game
  └─ finaliza Match
       └─ Outbox
            └─ Publish PartidaEncerrada
                 └─ RabbitMQ
                      └─ Manager Inbox
```

O `Manager Inbox` é o primeiro processo separado do núcleo. Ele recebe o fato de que a
partida terminou e cria um `MatchResult` para **cada Manager da partida**. A súmula, porém,
continua pertencendo ao Game.

Visão geral:

```text
                         ┌───────────────────────────┐
                         │           React           │
                         │ cadastros + partida + UI  │
                         └─────────────┬─────────────┘
                                       │
                           HTTP comandos / consultas
                                       │
                                       ▼
┌───────────────────────────────────────────────────────────────────┐
│                              GAME                                 │
│                                                                   │
│  Club / Player / Manager / vínculos / TeamSetup                  │
│                                                                   │
│  ┌──────────────────┐       ┌──────────────────────────────────┐  │
│  │    Manager AI    │       │           MatchEngine            │  │
│  │ escalação da CPU │       │ matemática + RNG + MatchEvents   │  │
│  └──────────────────┘       └──────────────────────────────────┘  │
│                                                                   │
│  Match + execução viva em memória + histórico/súmula             │
│                                                                   │
└───────────────┬────────────────────────────┬──────────────────────┘
                │                            │
              SignalR                  Outbox + Publish
                │                            │
                ▼                            ▼
              React                       RabbitMQ
                                               │
                                               ▼
                                    ┌──────────────────────┐
                                    │    Manager Inbox     │
                                    │ InboxItem/MatchResult│
                                    └──────────────────────┘
```

`Competition` não participa desta topologia inicial.

---

## 2. Fronteiras e responsabilidades

### 2.1 Game

O `Game` é dono de:

- `Club`;
- `Player`;
- `Manager`;
- vínculos atuais Player ↔ Club e Manager ↔ Club;
- `TeamSetup`;
- `Match`;
- criação e execução da partida;
- estado vivo da partida;
- persistência do histórico;
- súmula;
- `MatchEngine`;
- `Manager AI`;
- API usada pelo React;
- SignalR da partida;
- publicação de `PartidaEncerrada` através do Outbox.

O Game sabe **como uma partida acontece**.

### 2.2 MatchEngine

O `MatchEngine` é **componente interno do Game**, não processo, não microsserviço.

É a **única fonte da matemática da partida**.

Responsabilidades:

- calcular qualidades coletivas;
- aplicar pesos posicionais;
- tratar goleiro;
- calcular λ;
- aplicar mando;
- consumir `IRandomSource`;
- executar os sorteios;
- atualizar placar;
- produzir os eventos esportivos que pertencem ao motor.

Ele **não conhece**:

- React;
- SignalR;
- RabbitMQ;
- Manager Inbox;
- Competition;
- EF Core ou banco;
- DTOs HTTP.

O motor recebe entradas de domínio e devolve evolução/resultados de domínio.

### 2.3 Manager AI

O `Manager AI` também é **componente interno do Game**, separado do MatchEngine.

Responsabilidade geral:

> tomar decisões de técnico para Managers CPU.

No núcleo inicial, sua responsabilidade esportiva real é menor e objetiva:

> montar a escalação inicial da CPU.

Ele:

- recebe o elenco disponível;
- reutiliza as qualidades e pesos posicionais reconhecidos pelo domínio;
- usa `PreferredFormation` da CPU;
- respeita as posições reconhecidas pelo domínio;
- prioriza posição natural e secundária;
- improvisa jogador de linha apenas quando necessário;
- exige goleiro real no slot de GK;
- usa as mesmas qualidades reconhecidas pelo domínio para comparar encaixes.

Ele **não**:

- calcula λ em fórmula própria;
- simula a partida;
- possui uma “força paralela” diferente da usada pelo Game;
- conhece RabbitMQ;
- conhece Competition;
- persiste diretamente.

CPU fazendo substituições inteligentes durante o jogo é evolução futura.

### 2.4 Manager Inbox

O `Manager Inbox` é processo separado, com responsabilidade e persistência próprias. O Game e o Inbox não compartilham banco; cada processo é dono da sua persistência.

Seu primeiro caso de uso é:

> receber `PartidaEncerrada` e criar um `MatchResult` para **cada Manager da partida**.

Ele não é dono da Match e não replica a súmula.

### 2.5 Competition

`Competition` é futura e fica fora deste recorte.

Fronteira preservada:

> Game sabe como uma partida acontece.
>
> Competition saberá como uma competição acontece.

Não desenhar campeonato, calendário, rodadas ou classificação dentro do Game apenas para
antecipar essa etapa.

---

## 3. Club, Player, Manager e vínculos

### 3.1 Entidades independentes

`Club`, `Player` e `Manager` são entidades independentes.

Nenhuma delas existe apenas como subobjeto obrigatório de outra.

Um Player pode existir sem clube. Um Manager também pode existir sem clube.

Os vínculos atuais são conceitos próprios:

```text
PlayerClubAssignment
- PlayerId
- ClubId
- StartedAt
- EndedAt nullable

ManagerClubAssignment
- ManagerId
- ClubId
- StartedAt
- EndedAt nullable
```

`EndedAt = null` representa o vínculo atual.

Regras do recorte:

- Player possui no máximo um clube atual;
- Manager possui no máximo um clube atual;
- Club possui no máximo um Manager atual.

Contratos completos, salários, duração contratual, multas e rescisões são futuros.

### 3.2 Manager Human e CPU

Existe uma única entidade `Manager`.

O tipo de controle diferencia:

- `Human`;
- `Cpu`.

Não criar subclasses `HumanManager` e `CpuManager` apenas para representar esse tipo.

No primeiro mundo jogável:

- existe um único Manager Human;
- podem existir Managers CPU;
- há inicialmente dois clubes: um humano e um CPU.

Isso é recorte inicial, não proibição arquitetural de multiplayer futuro.

### 3.3 ManagerProfile

Preferências do técnico ficam separadas da escalação persistente do clube.

`ManagerProfile.PreferredFormation`:

- é obrigatória para CPU no recorte inicial;
- não é obrigatória para Human.

Não confundir `PreferredFormation` com `TeamSetup`.

---

## 4. Cadastros React do núcleo inicial

A interface inicial usa **telas reais do produto**, não wizard descartável e não apenas
seed de banco.

Áreas mínimas:

- Clubes;
- Jogadores;
- Técnicos;
- Escalação;
- Partida.

### 4.1 Jogadores

O cadastro de Player é global e independente do clube.

Permite:

- criar individualmente;
- criar em lote;
- editar;
- consultar;
- vincular a clube;
- desvincular de clube.

Cadastro em lote:

- funciona como grade com vários jogadores;
- pode receber um clube comum opcional para criar o vínculo;
- usa as mesmas validações do cadastro individual;
- é atômico: uma linha inválida cancela o lote inteiro.

Lista global:

- busca por nome;
- filtro por clube atual ou sem clube;
- filtro por posição natural;
- paginação server-side simples, aproximadamente 20 itens por página.

Ao abrir o elenco de um clube, mostrar todos os jogadores daquele clube sem paginação.

### 4.2 Técnicos

A área de Manager é global e equivalente ao cadastro de jogadores no que diz respeito a
existir independentemente do clube.

Deve permitir definir:

- identidade;
- `ControlType` Human/Cpu;
- `ManagerProfile.PreferredFormation` quando aplicável;
- vínculo/desvínculo com Club.

### 4.3 Exclusão

Exclusão física não é prioridade do primeiro recorte.

---

## 5. Elenco, posições e TeamSetup

### 5.1 Elenco inicial

Cada um dos dois clubes possui **16 jogadores**:

- 11 titulares;
- 5 reservas;
- idealmente 2 goleiros no elenco.

O domínio não deve possuir uma limitação estrutural eterna de “exatamente 16”. Esse é o
recorte do primeiro mundo jogável.

### 5.2 Formações suportadas

Formações iniciais:

- 4-4-2;
- 4-3-3;
- 3-5-2.

Posições reconhecidas pelo recorte:

`GK · CB · RB/LB · RWB/LWB · DM · CM · AM · RM/LM · RW/LW · ST`

### 5.3 Conhecimento posicional

O Player possui:

- uma `NaturalPosition`;
- zero ou mais `SecondaryPositions`.

A posição natural não pode ser repetida na coleção de secundárias.

**Improvisação não é persistida.**

Ela é derivada no momento da escalação:

> o jogador está improvisado quando ocupa um slot que não corresponde à sua posição natural
> nem a uma de suas posições secundárias.

No recorte inicial:

- não existe penalização matemática por improvisação;
- a penalização **não precisa ficar pré-construída** esperando ativação;
- posição ainda importa para montar os times;
- CPU prioriza Natural/Secundária;
- CPU improvisa linha somente quando necessário;
- Human pode improvisar jogador de linha, com aviso da UI;
- slot GK exige goleiro real.

Uma penalização numérica futura continua possível, mas será desenhada quando essa mecânica
for realmente implementada.

### 5.4 TeamSetup

`TeamSetup` é configuração persistente do trabalho atual do técnico no clube.

Conceito:

```text
TeamSetup
- ManagerId
- ClubId
- Formation
- Slots
```

Ele não é:

- campo solto dentro de Club;
- dado que nasce apenas dentro da Match;
- preferência do ManagerProfile.

O usuário monta e salva a equipe. Os 11 slots formam os titulares; os 5 restantes formam o
banco do recorte inicial.

A configuração permanece para os jogos seguintes até ser alterada.

Quando uma Match começa, ela tira uma **fotografia própria** da configuração válida naquele
momento. Alterar o TeamSetup depois não muda retroativamente a partida já iniciada.

Se um jogador escalado deixa o clube, o TeamSetup fica inválido e uma nova Match é bloqueada
até que a escalação seja corrigida. Não apagar silenciosamente a configuração.

### 5.5 SimpleManagerAI — preparação do lado CPU

Fluxo inicial:

1. recebe todo o elenco disponível;
2. tenta a `PreferredFormation`;
3. cria os slots da formação;
4. procura combinação válida de 11 jogadores únicos;
5. prioriza Natural/Secundária;
6. improvisa somente os slots que não tenham solução adequada;
7. compara candidatos possíveis com `ScoreNoSlot`;
8. GK exige goleiro real;
9. os 11 escolhidos viram titulares;
10. os 5 restantes viram banco.

Para jogador de linha:

```text
ScoreNoSlot =
  (QualidadeAtaque × PesoAtaque + QualidadeDefesa × PesoDefesa)
  / (PesoAtaque + PesoDefesa)
```

Para GK:

```text
ScoreNoSlot = QualidadeGoleiro
```

Esse score reutiliza as mesmas qualidades e pesos posicionais do domínio. Ele não calcula λ,
não cria uma força paralela e **não é penalização de improvisação**.

Se a formação preferida não puder ser montada, a CPU tenta as outras formações suportadas.
Se nenhuma produzir 11 jogadores únicos com goleiro real no slot GK, o caso de uso retorna
**falha de domínio**. Não existe escalação fixa/fake nem relaxamento silencioso das invariantes.

---

## 6. O jogador que o MatchEngine lê

### 6.1 Regra estrutural

O motor lê **números**, não rótulos narrativos.

Apelidos derivados como “motorzinho”, “firulento” ou “carregador de piano” podem existir na
UI no futuro, mas não substituem os atributos.

### 6.2 Jogador de linha — 6 atributos cadastrados diretamente

Escala 0–100:

- Finalização;
- Drible;
- Passe;
- Ritmo;
- Defesa;
- Físico.

Esses seis atributos são **cadastrados diretamente** no recorte inicial.

Os atributos finos preservados no `mapa-jogador.md` são mapa futuro. Não existe hoje uma
etapa “atributos finos → seis atributos grossos”.

Qualidades derivadas:

```text
QualidadeAtaque =
  (Finalização + Drible + Passe + Ritmo + Físico) / 5

QualidadeDefesa =
  (Defesa + Ritmo + Físico) / 3
```

Ritmo e Físico entram nos dois lados.

As qualidades derivadas não são digitadas nem persistidas como fonte independente da
verdade.

### 6.3 Goleiro — 6 atributos cadastrados diretamente

Escala 0–100:

- Mergulho;
- Manuseio;
- Chute;
- Reflexos;
- Velocidade;
- Posicionamento.

Qualidade derivada:

```text
QualidadeGoleiro = média dos 6 atributos
```

Chute ainda entra na qualidade geral do goleiro no motor inicial; modelar distribuição com
os pés é evolução futura.

### 6.4 Estados ricos do jogador

Lesão, suspensão, cansaço, moral e outros estados ricos **não pertencem ao núcleo inicial
implementado**.

Eles continuam mapeados como evolução futura e só devem ganhar regras quando a mecânica
correspondente existir.

---

## 7. Dos jogadores à força do time

A força coletiva é derivada dos jogadores em campo. Club não possui um “rating mágico” que
substitui o elenco.

### 7.1 Pesos posicionais

Para jogadores de linha, ataque e defesa usam pesos independentes por posição.

| Posição (tipo) | Peso ataque | Peso defesa |
|---|---:|---:|
| ST | 0,90 | 0,10 |
| RW/LW | 0,80 | 0,25 |
| AM | 0,75 | 0,30 |
| RWB/LWB | 0,70 | 0,60 |
| RM/LM | 0,65 | 0,45 |
| CM | 0,55 | 0,55 |
| RB/LB | 0,45 | 0,80 |
| DM | 0,45 | 0,75 |
| CB | 0,15 | 0,90 |
| GK | 0,00 | tratado separadamente |

Os valores exatos são parâmetros de game design. A direção geral tem respaldo futebolístico,
mas os números devem permanecer calibráveis.

### 7.2 Agregação dos jogadores de linha

A agregação usa média ponderada para permanecer na régua 0–100.

Ataque:

```text
AtaqueLinha =
  Σ(PesoAtaquePosição × QualidadeAtaqueJogador)
  ------------------------------------------------
                Σ(PesoAtaquePosição)
```

Defesa de linha:

```text
DefesaLinha =
  Σ(PesoDefesaPosição × QualidadeDefesaJogador)
  ------------------------------------------------
                Σ(PesoDefesaPosição)
```

Não existe multiplicador de aptidão/improvisação no cálculo atual.

### 7.3 Goleiro na defesa

O goleiro não entra como “mais um de 11” na média defensiva.

```text
DefesaTime = 0,70 × DefesaLinha + 0,30 × QualidadeGoleiro
```

O percentual 30/70 é parâmetro de game design e deve continuar calibrável.

No ataque, o goleiro contribui zero no motor inicial.

### 7.4 Formação

Não existe modificador adicional de “qualidade da formação”.

A formação influencia a força porque muda os slots e, portanto, os pesos posicionais
aplicados aos jogadores.

Função tática, contexto coletivo adicional da formação e outras camadas ficam para depois.

---

## 8. Da força ao λ e do λ ao gol

### 8.1 Força → gols esperados

Fórmula vigente:

```text
λ = BaseGoals × 2 ^ ((Ataque − DefesaAdversária) / DifferenceScale)
```

Valores iniciais:

```text
λ = 1,35 × 2^((Ataque − DefesaAdversária) / 30)
```

A diferença de força é usada porque ratings 0–100 representam uma régua de qualidade; a
razão entre ratings não possui interpretação útil equivalente.

### 8.2 Mando

O mando é aplicado em λ, não no rating do jogador ou do time.

Parâmetro inicial:

```text
HomeAdvantage = 0,15
```

Aplicação:

- mandante × 1,15;
- visitante × 0,85.

### 8.3 Travas e parâmetros

Centralizar em `MatchSimulationParameters`:

| Parâmetro | Inicial | Função |
|---|---:|---|
| BaseGoals | 1,35 | λ neutro quando Ataque = Defesa |
| DifferenceScale | 30 | 30 pontos de gap = dobro/metade de λ |
| HomeAdvantage | 0,15 | mando inicial |
| ReferenceStrength | 70 | metadado de referência; não entra na fórmula |
| MinExpectedGoals | 0,10 | trava inferior |
| MaxExpectedGoals | 5,00 | trava superior |

### 8.4 λ → chance por minuto

```text
p = 1 − e^(−λ/90)
```

A cada minuto do jogo, o MatchEngine compara um sorteio do `IRandomSource` com a
probabilidade correspondente.

O motor inicial modela gol/não-gol por minuto. Não modela ainda cada chute individual.

### 8.5 Função pura

O cálculo de λ deve permanecer puro:

```text
CalculateExpectedGoals(ataque, defesaAdversaria)
```

Ele não conhece:

- banco;
- HTTP;
- SignalR;
- RabbitMQ;
- escalação como entidade;
- relógio real;
- sorteio.

O mando é aplicado em passo separado.

### 8.6 λ(t)

λ só muda quando muda uma entrada efetiva da conta.

No primeiro caso de intervenção esportiva já fechado:

> uma substituição muda quem está em campo → forças podem mudar → λ dos minutos futuros
> pode mudar.

O passado não é recalculado.

Não alterar λ apenas porque:

- o minuto avançou;
- o usuário pausou;
- saiu uma atualização de UI;
- o placar mudou.

Outros modificadores dependem de mecânicas futuras.

---

## 9. Aleatoriedade — distribuição, não destino

A força dos times define uma **distribuição de resultados possíveis**. A Seed produz uma
realização dessa distribuição.

```text
força + mando + contexto efetivo
            ↓
            λ
            ↓
      probabilidades
            ↓
      IRandomSource
            ↓
       resultado
```

Regras:

- favoritos ainda podem empatar ou perder;
- cada execução nova recebe uma Seed nova;
- todo sorteio passa por `IRandomSource`;
- nada de `Random.Shared` ou geradores espalhados pelo domínio;
- Seed serve para diagnóstico/reprodução técnica da execução;
- `EngineVersion` acompanha Seed para permitir diagnóstico dentro da mesma versão do motor;
- Seed e EngineVersion são metadados internos do Game;
- não são enviados em `PartidaEncerrada`.

### Testes do motor

A rede de segurança principal é estatística.

Testar propriedades como:

- média de gols;
- taxa de empate;
- frequência de zebra;
- vitórias de favorito;
- goleadas;
- distribuição de placares;
- comportamento do mando.

Não transformar um placar específico em contrato de gameplay.

A calibração preservada no documento de decisões partiu de simulações em larga escala e
levou `BaseGoals` de 1,20 para 1,35, buscando aproximadamente 2,7 gols por jogo. Uma
calibração séria contra liga-alvo real continua futura e não bloqueia o núcleo.

---

## 10. Nascimento e lifecycle da Match

### 10.1 MatchId identifica execução concreta

`MatchId` nunca significa “o confronto abstrato entre dois clubes”.

Ele identifica **uma execução concreta**.

Não reutilizar MatchId ao reiniciar uma partida perdida ou abandonada.

### 10.2 Nascimento

A execução nasce de verdade nesta ordem:

1. validar os dois lados;
2. gerar `MatchId`;
3. gerar nova Seed;
4. registrar EngineVersion atual;
5. persistir Match como `InProgress`;
6. persistir os fatos iniciais necessários ao histórico;
7. iniciar o estado vivo em memória.

Persistir no nascimento:

- MatchId;
- status `InProgress`;
- StartedAt;
- ClubId + ClubName histórico dos dois lados;
- ManagerIds;
- os 16 relacionados de cada lado;
- PlayerId + PlayerName usado naquela partida;
- papel inicial `Starter` ou `Substitute`;
- Seed;
- EngineVersion.

### 10.3 O que NÃO é checkpoint

Não persistir a cada minuto:

- minuto corrente;
- placar parcial;
- cursor/estado atual do RNG;
- estado tático vivo;
- eventos provisórios;
- estado de pausa vivo.

Não existe checkpoint/resume.

### 10.4 Status persistidos

Status históricos:

- `InProgress`;
- `Finished`;
- `NotCompleted`.

Estados vivos da execução:

- `Running`;
- `Paused`;
- `Halftime`.

Não misturar status persistido com estado vivo.

---

## 11. Execução viva da partida

### 11.1 Autoridade do servidor

A partida pertence ao Game.

O React não é autoridade sobre:

- minuto;
- placar;
- RNG;
- estado da partida;
- pausas;
- resultado.

Fechar aba, dar F5 ou perder SignalR não encerra a execução.

### 11.2 Relógio de simulação × relógio real

O minuto esportivo é estado da simulação.

O relógio real serve para cadenciar a experiência.

Configuração inicial:

- minuto 1–45: 3 minutos reais;
- intervalo: 30 segundos reais;
- minuto 46–90: 3 minutos reais;
- cada minuto esportivo corresponde inicialmente a 4 segundos reais de bola correndo.

O intervalo termina automaticamente e inicia o segundo tempo.

### 11.3 SignalR

SignalR entra já na primeira partida interativa.

Fluxo:

```text
comandos do usuário:
React → HTTP → Game

atualizações da execução:
Game → SignalR → React
```

Atualizações iniciais incluem:

- minuto/estado;
- placar;
- novos eventos;
- transições `Running`;
- `Paused`;
- `Halftime`;
- `Finished`.

Ao retornar para uma partida que ainda está viva:

1. React recupera uma fotografia atual por HTTP;
2. depois acompanha as mudanças seguintes por SignalR.

Queda da conexão SignalR não pausa a Match e não altera MatchId ou Seed.

### 11.4 Pausas manuais

Cada partida possui **3 pausas manuais**.

Cada pausa:

- dura no máximo 60 segundos reais;
- congela a simulação;
- não processa minuto;
- não avança RNG;
- não muda λ apenas por existir;
- pode terminar antes por comando `Continue`;
- retoma automaticamente ao zerar o cronômetro.

Abrir uma pausa consome uma pausa manual.

Pausas obrigatórias por lesão, vermelho ou outros eventos ricos são futuras.

### 11.5 Intervalo

O intervalo é estado próprio (`Halftime`) e não consome pausa manual.

Dura 30 segundos reais.

Quando terminar, o segundo tempo recomeça automaticamente.

---

## 12. Sair, desconexão e crash

### 12.1 Navegador desconectado

Não encerra a partida.

A execução continua no Game.

O usuário pode voltar e recuperar a fotografia atual se a execução ainda estiver viva.

### 12.2 Sair explícito

O comando de sair encerra aquela execução como:

```text
NotCompleted
```

Consequências:

- não é W.O.;
- não existe resultado oficial;
- não publica `PartidaEncerrada`;
- a execução viva termina;
- uma eventual nova tentativa recebe outro MatchId e outra Seed.

W.O. e “reinício rápido” são features futuras.

### 12.3 Crash do Game

Se o processo do Game morrer:

- o estado vivo em memória se perde;
- não tentar reconstruir minuto;
- não reconstruir placar parcial;
- não reconstruir eventos provisórios;
- não reconstruir RNG;
- não reconstruir escalação viva alterada durante a execução.

Na subida do Game:

> Match persistida como `InProgress` sem execução viva correspondente → marcar
> `NotCompleted`.

A próxima execução é nova:

- novo MatchId;
- nova Seed.

Não existe recuperação técnica da partida interrompida.

---

## 13. Histórico e súmula

O Game é dono do histórico esportivo da Match.

### 13.1 Relacionados

Todos os 16 relacionados de cada lado pertencem à súmula, inclusive reservas que nunca
entraram.

Preservar identidade histórica suficiente:

- PlayerId;
- PlayerName usado na partida;
- ClubId;
- ClubName usado na partida;
- papel inicial `Starter/Substitute`.

Não congelar desnecessariamente:

- atributos completos;
- overall futuro;
- evolução completa do Player.

### 13.2 Persistência

Persistir os conceitos reais:

- Match;
- RelatedPlayers;
- MatchEvents.

Não criar um agregado persistido duplicado `MatchReport` apenas para materializar uma visão.

O Game compõe a súmula sob demanda.

Consulta fechada:

```text
GET /matches/{MatchId}/report
```

### 13.3 MatchEvents do recorte

No núcleo inicial consolidado:

- `Goal`;
- `Substitution` quando o Marco 6 estiver implementado.

Cartões, lesões, pênaltis e eventos esportivos ricos continuam futuros.

Não tratar uma única formação como “a formação da partida inteira”, porque a composição pode
mudar durante a execução quando substituições existirem.

---

## 14. Finalização normal e consistência transacional

### 14.1 Final normal

Quando o minuto 90 termina normalmente, persistir na mesma transação:

- placar final;
- eventos definitivos;
- `Match.Status = Finished`;
- `FinishedAt`;
- intenção de publicação através do MassTransit Transactional Outbox com EF Core.

### 14.2 Outbox

Usar **MassTransit Transactional Outbox com EF Core**.

Não construir Outbox caseira.

Objetivo:

> manter o resultado persistido e a intenção de publicar `PartidaEncerrada` consistentes.

Depois do commit, a infraestrutura entrega o evento.

Se RabbitMQ estiver indisponível:

- o resultado `Finished` continua válido;
- a intenção permanece no Outbox;
- a publicação pode acontecer depois.

Broker indisponível não transforma uma partida corretamente finalizada em falha esportiva.

### 14.3 Falha da persistência final

Se a persistência final falhar enquanto o processo ainda está vivo:

- pode repetir a gravação do mesmo resultado já calculado;
- não executar novamente o MatchEngine;
- não gerar outra Seed.

Se o processo morrer antes do commit:

- a execução viva se perde;
- o `InProgress` inicial será tratado como `NotCompleted` no startup.

---

## 15. `PartidaEncerrada` — primeiro evento de integração real

`PartidaEncerrada` é um **fato** e, portanto, usa `Publish`.

É o primeiro uso real de RabbitMQ no roadmap atual.

Contrato:

```text
PartidaEncerrada
- EventId
- MatchId
- Home
    - ClubId
    - ClubName
    - ManagerId
- Away
    - ClubId
    - ClubName
    - ManagerId
- GolsMandante
- GolsVisitante
- EncerradaEm

Header:
- CorrelationId
```

Não incluir:

- escalações;
- atributos;
- λ;
- Seed;
- EngineVersion;
- súmula;
- timeline completa;
- texto pronto de UI.

Semântica das identidades:

- `MatchId` = execução concreta da partida;
- `EventId` = ocorrência concreta do evento de integração.

O pacote `Contracts` deve continuar mínimo: recebe apenas contratos que realmente cruzam
processos. No núcleo atual, `PartidaEncerrada` é o primeiro contrato esportivo real dessa
fronteira.

---

## 16. Manager Inbox

### 16.1 Primeiro tipo

Primeiro tipo de mensagem:

```text
MatchResult
```

### 16.2 InboxItem mínimo

```text
InboxItem
- Id
- ManagerId
- Type
- Title
- Message
- MatchId
- IsRead
- CreatedAt
```

Não incluir:

- EventId;
- CorrelationId;
- ReadAt;
- Seed;
- EngineVersion;
- cópia da súmula.

Não criar hierarquia de classes por tipo de mensagem neste recorte.

### 16.3 Multiplicidade

Cada `PartidaEncerrada` gera **um `MatchResult` para cada Manager da partida**:

- um item com `ManagerId = Home.ManagerId`;
- um item com `ManagerId = Away.ManagerId`.

Os dois apontam para o mesmo `MatchId`. O Inbox continua organizado por Manager e não
transforma o evento de integração em uma única mensagem global da partida.

### 16.4 Leitura

Superfícies mínimas: lista e detalhe.

Regras:

- uma nova mensagem nasce com `IsRead = false`;
- listar mensagens não marca como lidas;
- abrir detalhe faz `IsRead: false → true`;
- depois de lida, não volta a `false`.

### 16.5 Idempotência

Chave de negócio:

```text
ManagerId + MatchId + Type
```

Criar constraint UNIQUE correspondente.

Se a mesma entrega chegar novamente:

- duplicata equivalente = no-op;
- não criar segunda mensagem.

### 16.6 Retry e fila de erro

MassTransit v8 permanece como biblioteca de mensageria do projeto.

Política inicial:

```text
Immediate(3)
```

Sem redelivery atrasado no primeiro fluxo.

Se o consumo continuar falhando após as tentativas, a mensagem vai para a fila `_error`
padrão do MassTransit.

### 16.7 Ver súmula

Inbox guarda `MatchId`.

Ao escolher “Ver súmula”:

> Inbox/React consulta o Game pela Match correspondente.

A súmula não é copiada para o banco do Inbox.

---

## 17. Primeira intervenção esportiva — substituição humana

A primeira intervenção esportiva real é a substituição feita pelo Manager Human.

Ela ocorre durante:

- uma pausa manual;
- ou o intervalo.

Fluxo conceitual:

1. usuário escolhe quem sai;
2. escolhe quem entra;
3. React envia comando por HTTP;
4. Game valida;
5. composição em campo muda;
6. registra `MatchEvent Substitution`;
7. recalcula forças afetadas;
8. recalcula λ dos minutos futuros quando necessário;
9. passado permanece intacto;
10. SignalR atualiza a tela.

### 17.1 Regras mínimas

- quem entra deve estar relacionado e no banco;
- quem sai deve estar em campo;
- jogador que saiu não volta;
- não pode haver jogador duplicado;
- continuam 11 jogadores em campo;
- máximo de 5 substituições na partida;
- máximo de 3 paradas de substituição com bola rolando;
- substituições no intervalo não consomem parada.

### 17.2 Três contadores diferentes

Não misturar:

1. **pausas manuais**: máximo 3;
2. **paradas de substituição com bola rolando**: máximo 3;
3. **substituições totais**: máximo 5.

Abrir pausa consome pausa manual.

Se o usuário apenas consulta o time e continua, não consome parada de substituição.

Se confirmar uma ou mais substituições naquela pausa, consome **uma** parada de
substituição.

No intervalo:

- não consome pausa manual;
- não consome parada de substituição;
- substituições ainda contam no limite total de 5.

CPU fazendo substituições durante a partida continua fora do núcleo inicial.

---

## 18. Convenções de engenharia

Esta seção é o chão do projeto. Não é um marco separado.

### 18.1 Domínio e dependências

- Clean Architecture: regra de futebol não depende de React, banco ou broker.
- DDD onde há regra real a proteger.
- Invariantes ficam junto do dono da regra, não apenas no Controller.
- Result Pattern para falhas esperadas de negócio.
- Casos de uso explícitos na Application.
- Repository através de interfaces consumidas pela Application; EF Core implementa na
  Infrastructure.
- Persistência se adapta ao domínio; não deixar detalhe de banco definir entidade.
- DTO HTTP, domínio e contrato de integração têm papéis diferentes.
- Evento de domínio não vira automaticamente evento de integração.

### 18.2 Determinismo e tempo

- todo acaso passa por `IRandomSource`;
- minuto esportivo é estado da simulação;
- wall-clock só entra onde há tempo real, como cadência da partida, intervalo e pausa;
- motor matemático permanece síncrono quando só realiza cálculo;
- async/await fica nas bordas de I/O.

### 18.3 API e infraestrutura

Base:

- ASP.NET Core Web API;
- Controllers;
- injeção de dependência;
- DTOs;
- FluentValidation;
- EF Core;
- banco relacional;
- migrations;
- health checks;
- Swagger;
- configuração por variável de ambiente;
- logging estruturado;
- CorrelationId.

`CorrelationId` nasce ou é recebido na borda e deve ser propagado quando o fluxo cruzar
HTTP e, depois, mensageria.

### 18.4 Mensageria

- RabbitMQ + MassTransit só quando existe fronteira real;
- primeiro Publish real: `PartidaEncerrada`;
- Outbox no Game para finalização + publicação;
- idempotência no Manager Inbox;
- retry imediato + `_error` para falhas de consumo.

Não usar Saga no núcleo atual: não existe conversa distribuída que justifique isso.

### 18.5 Testes

Pirâmide:

- MatchEngine/Domínio → muitos testes rápidos, sem infraestrutura;
- Application → testes focados em casos de uso;
- integração → banco/broker reais apenas onde necessário;
- Testcontainers para infraestrutura descartável nos testes de integração.

Ferramentas já alinhadas:

- xUnit;
- Moq quando apropriado;
- FluentAssertions;
- Coverlet.

Testes de motor devem combinar:

- propriedades determinísticas das funções;
- reprodução controlada quando útil para diagnóstico;
- testes estatísticos de distribuição.

### 18.6 Infra local e CI

Docker/docker-compose deve permitir subir localmente os componentes que já existirem em cada
marco.

A topologia cresce junto com o produto:

- no começo: Game + banco + frontend conforme a organização adotada;
- quando a integração entrar: RabbitMQ;
- quando o Inbox entrar: Manager Inbox + banco próprio.

Não manter um Opponent Service vazio apenas para “ter microsserviços”.

CI no GitHub Actions desde o começo:

- restaurar;
- compilar;
- rodar testes;
- falhar a mudança se o projeto não estiver verde.

### 18.7 ADRs

Registrar ADR/notinha somente para decisões arquiteturais significativas e caras de reverter.

Não transformar parâmetro de balanceamento, como peso ofensivo de uma posição, em ADR.

### 18.8 O que não entra por catálogo

Não adicionar automaticamente:

- MediatR/CQRS pleno;
- Redis;
- Kubernetes;
- OpenTelemetry;
- Event Sourcing;
- Saga;
- AutoMapper;
- Specification;
- Unit of Work custom;
- novos microsserviços sem responsabilidade real.

Problema primeiro; ferramenta depois.

---

## 19. Ciência, calibração e heurística

Manter separação explícita entre fundamento e game design.

| Mecânica | Base mais forte | Heurística / parâmetro calibrável |
|---|---|---|
| Gols por λ / Poisson | Poisson e forma log-linear são modelos usados no futebol | parâmetros específicos do jogo |
| Força → λ | diferença ataque/defesa em forma exponencial | `BaseGoals = 1,35`, `DifferenceScale = 30` |
| Mando | direção e existência do efeito | `HomeAdvantage = 0,15` |
| Pesos por posição | direção dos papéis | valores exatos 0,90, 0,45 etc. |
| Goleiro | goleiro merece tratamento separado | mistura exata 30/70 |
| Improvisação | conceito futebolístico | eventual penalização futura ainda não desenhada |

Mesmo uma fórmula baseada em modelo estatístico precisa de calibração para produzir gameplay
plausível.

---

## 20. O que fica fora do núcleo inicial

Não desenhar nesta fase além da bandeira:

- Competition;
- campeonato;
- vários clubes jogando entre si;
- calendário;
- rodadas;
- classificação;
- autenticação/autorização;
- multiplayer humano;
- lesões;
- cartões;
- expulsões;
- pênaltis;
- pausas obrigatórias;
- cansaço;
- CPU fazendo substituições inteligentes;
- mudança tática/formação dinâmica;
- funções táticas;
- instruções condicionais;
- contratos completos;
- diretoria;
- torcida;
- finanças profundas;
- evolução de jogador;
- dados reais externos;
- xG/PSxG e motor por chutes;
- duelos/fases/posicionamento avançado;
- calibração séria contra liga real;
- cloud/deploy.

Cloud terá etapa própria no futuro. Não escolher agora AWS, Azure, ECS, RDS, Amazon MQ ou
qualquer topologia de produção.

---

## 21. Decisões antigas que NÃO pertencem mais ao sistema atual

Não reintroduzir durante implementação:

- Opponent Service no núcleo inicial;
- robô em processo separado apenas para justificar RabbitMQ;
- `SolicitarEscalacao` / `EnviarEscalacao` como primeiro fluxo do sistema;
- Saga de cumprimento;
- RabbitMQ no Marco 1;
- `PartidaCriada` como primeiro Publish;
- escalação fixa/fake da CPU;
- apenas 11 jogadores por clube;
- escolher os 11 melhores gerais antes de encaixar posições;
- proibir toda improvisação;
- penalização de improvisação pré-construída e apenas “desligada”;
- polling como canal principal da partida ao vivo;
- partida em andamento sem qualquer identidade persistida;
- tentar recuperar Match após crash;
- Seed/EngineVersion em `PartidaEncerrada`;
- súmula inteira em `PartidaEncerrada`;
- liga/Competition dentro do núcleo inicial apenas para justificar Publish;
- reação da CPU a gol atravessando RabbitMQ;
- pausa obrigatória por lesão/vermelho no primeiro motor.

---

## 22. Critério de coerência do documento

Este sistema está coerente quando as seguintes frases podem ser verdade ao mesmo tempo:

1. O Game é a autoridade da partida.
2. O MatchEngine é interno e não conhece infraestrutura.
3. O Manager AI é interno e não existe para justificar mensageria.
4. React manda comandos por HTTP e recebe atualizações por SignalR.
5. A Match possui identidade persistida desde o início, mas estado vivo não é checkpointado.
6. Crash perde a execução viva e transforma a Match órfã em `NotCompleted`.
7. Final normal é persistido antes de ser considerado fato de integração.
8. RabbitMQ começa em uma fronteira real: `PartidaEncerrada` → Manager Inbox.
9. Inbox é dono da mensagem; Game continua dono da súmula.
10. A primeira intervenção esportiva é substituição humana e altera apenas o futuro da
    simulação.
11. Competition permanece fora até seu próprio desenho.
12. O roadmap pode implementar tudo isso em seis marcos sem precisar criar feature fake.

---

*Documento consolidado da Fase B2 — acompanha `briefing-jogo-futebol.md`,
`decisoes-jogo-futebol.md`, `mapa-jogador.md` e `roadmap-jogo-futebol.md`. Revisão cruzada
final dos cinco documentos oficiais concluída.*
