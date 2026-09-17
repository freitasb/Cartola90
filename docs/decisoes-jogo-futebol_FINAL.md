# Decisões da Sabatina — Jogo Manager de Futebol

> **Estado documental após a Fase A.** Este arquivo continua sendo o registro vivo
> das decisões do projeto, mas agora possui duas camadas explícitas:
>
> 1. **DECISÕES VIGENTES DA FASE A** — fonte operacional atual;
> 2. **HISTÓRICO DA SABATINA** — preserva decisões, pesquisas, parâmetros e caminhos
>    antigos. Quando um trecho histórico estiver marcado como **SUPERADO**, ele não
>    deve orientar implementação. Quando não houver conflito, o detalhamento técnico
>    histórico continua válido.
>
> Se houver conflito, a camada vigente desta seção vence.

**Última consolidação:** Fase B1, após encerramento da revisão arquitetural da Fase A.

**Emenda posterior:** 17/09/2026 — seção 25 acrescentada (plataforma de execução, nomes
oficiais e formato da solution). Ela vence sobre qualquer menção anterior a .NET 8,
`UUIDNext`, `manager-football` ou `ManagerFootball.sln` neste e nos demais documentos.

---

# PARTE I — DECISÕES VIGENTES APÓS A FASE A

## 1. Arquitetura inicial ✅ VIGENTE

### Game

O `Game` é o processo dono de:
- Club;
- Player;
- Manager;
- vínculos atuais necessários ao jogo;
- `TeamSetup`;
- execução e persistência da partida;
- histórico/súmula;
- `MatchEngine` interno;
- `Manager AI` interno.

### Manager Inbox

É processo separado e o primeiro consumidor real de mensageria.
Possui modelo e persistência próprios.

### Competition

**FUTURA/PARQUEADA.**

Fronteira preservada:
- Game sabe **como uma partida acontece**;
- Competition sabe **como uma competição acontece**.

Não redesenhar campeonato nesta fase.

---

## 2. MatchEngine ✅ VIGENTE

É componente interno do Game, não serviço.

É a única fonte da matemática da partida e responde por:
- qualidades coletivas;
- pesos posicionais;
- goleiro;
- λ;
- mando;
- RNG;
- sorteios;
- placar;
- eventos produzidos pela simulação.

Não conhece React, SignalR, RabbitMQ, Manager Inbox, Competition ou banco.

A partida progride de verdade no servidor, minuto a minuto. Não é pré-calculada
inteira para depois ser reproduzida visualmente.

> **Detalhamento matemático vigente:** permanece preservado mais abaixo na seção
> 4A/4B histórica deste mesmo arquivo, exceto onde houver nota explícita de correção
> da Fase A.

---

## 3. Manager AI ✅ VIGENTE

Também é componente interno do Game e separado do MatchEngine.

Responsabilidade:
> tomar decisões de técnico para gestores CPU.

No recorte inicial sua função real é montar a escalação inicial da CPU.

`SimpleManagerAI` usa as mesmas qualidades e pesos posicionais reconhecidos pelo
domínio. Não possui fórmula paralela inventada.

Não calcula λ, não simula partida, não conhece RabbitMQ/Competition e não persiste
diretamente.

---

## 4. Club, Player e Manager ✅ VIGENTE

São entidades independentes.

Conceitualmente existem:
- `PlayerClubAssignment`;
- `ManagerClubAssignment`.

Campos mínimos conceituais:
- identidade da entidade;
- `ClubId`;
- `StartedAt`;
- `EndedAt nullable`.

`EndedAt = null` representa vínculo atual.

Regras atuais:
- jogador: no máximo um clube atual;
- técnico: no máximo um clube atual;
- clube: no máximo um técnico atual.

Contrato completo (duração, salário, rescisão etc.) é **FUTURO/PARQUEADO**.

---

## 5. Manager Human e CPU ✅ VIGENTE

`Manager.ControlType`:
- `Human`;
- `Cpu`.

Não criar subclasses HumanManager/CpuManager.

No recorte inicial:
- existe um único Manager Human;
- podem existir vários Managers CPU.

Essa unicidade é regra inicial, não barreira eterna contra multiplayer.

`ManagerProfile.PreferredFormation`:
- obrigatória para CPU no recorte inicial;
- não obrigatória para Human.

`ManagerProfile` guarda preferências/configuração do técnico e não deve ser confundido
com `TeamSetup`, que representa a escalação atual persistida do trabalho no clube.

Formações suportadas:
- 4-4-2;
- 4-3-3;
- 3-5-2.

---

## 6. Mundo e elenco inicial ✅ VIGENTE

O primeiro recorte possui dois clubes:
- um do Manager Human;
- um de Manager CPU.

Sem seleção de adversário inicialmente.

Cada clube possui **16 jogadores**:
- 11 titulares;
- 5 reservas;
- idealmente 2 goleiros.

O domínio não fica estruturalmente limitado a 16.

---

## 7. Player — atributos atuais ✅ VIGENTE

### Linha

Cadastrados diretamente de 0 a 100:
- Finalização;
- Drible;
- Passe;
- Ritmo;
- Defesa;
- Físico.

Derivadas:

`QualidadeAtaque = (Finalização + Drible + Passe + Ritmo + Físico) / 5`

`QualidadeDefesa = (Defesa + Ritmo + Físico) / 3`

### Goleiro

Cadastrados diretamente de 0 a 100:
- Mergulho;
- Manuseio;
- Chute;
- Reflexos;
- Velocidade;
- Posicionamento.

`QualidadeGoleiro = média dos 6 atributos`

As qualidades derivadas não são digitadas.

### Conhecimento de posição

Player possui:
- `NaturalPosition`;
- zero ou mais `SecondaryPositions`.

Natural não se repete nas secundárias.

Não existe `Improvised` persistido. Improvisação é derivada quando o slot não é
natural nem secundário.

No recorte inicial:
- penalização numérica está desligada;
- posição continua importante;
- CPU prioriza natural/secundária e improvisa só quando necessário;
- Human pode improvisar linha com aviso de UI;
- GK exige goleiro real.

---

## 7A. Matemática do MatchEngine ✅ VIGENTE

A matemática fechada na sabatina **permanece válida**. A Fase A mudou topologia,
Player/posição e lifecycle, mas não substituiu a conta do motor.

### Pesos por posição

Dois pesos independentes por tipo de posição:

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

A agregação dos jogadores de linha usa média ponderada (divide pela soma dos pesos).
Os números exatos são **heurística de game design**, mesmo quando a direção tem
respaldo futebolístico.

### Goleiro

`DefesaTime = 0,70 × DefesaLinha + 0,30 × QualidadeGoleiro`

O 30% é parâmetro de game design, calibrável.

### Força → λ

`λ = 1,35 × 2^((Ataque − DefesaAdversária) / 30)`

Depois aplica mando:
- mandante × 1,15;
- visitante × 0,85.

Parâmetros preservados:

| Parâmetro | Inicial | Função |
|---|---:|---|
| BaseGoals | 1,35 | λ neutro quando ataque = defesa |
| DifferenceScale | 30 | 30 pontos de gap = dobro/metade do λ |
| ReferenceStrength | 70 | metadado para geração/interpretação; não entra na fórmula |
| MinExpectedGoals | 0,10 | trava inferior |
| MaxExpectedGoals | 5,00 | trava superior |
| HomeAdvantage | 0,15 | redistribuição inicial de mando |

A função de λ deve permanecer pura: recebe ataque/defesa efetivos e não conhece
banco, escalação, formação, sorteio ou infraestrutura.

### λ → minuto

`p = 1 − e^(−λ/90)`

O motor roda minuto a minuto. O placar não é pré-sorteado.

### Aleatoriedade

- força define a distribuição, não o destino;
- nova Seed por execução;
- todo sorteio passa por `IRandomSource`;
- Seed + EngineVersion são metadados técnicos internos;
- a mesma execução pode ser investigada tecnicamente, mas gameplay não promete
  "mesmas equipes = mesmo placar";
- testes principais são estatísticos, não snapshots de placar.

### λ(t)

λ só é recalculado quando muda uma entrada efetiva da conta. No primeiro caso já
planejado, uma substituição muda quem está em campo e pode mudar força/λ dos minutos
seguintes. Pausa por si só não muda λ.

### Registro de calibração preservado

A sabatina rodou simulações de 500 mil partidas. O ajuste de `BaseGoals` de 1,20
para 1,35 elevou a média de gols de aproximadamente 2,4 para aproximadamente 2,7 por
jogo, mantendo comportamento plausível de empate, mando e zebra. Os números completos
e ressalvas (0-0, eventual ajuste de mando/DifferenceScale e possível Dixon-Coles)
continuam preservados no detalhamento histórico desta mesma documentação.


---

## 8. SimpleManagerAI — montagem da escalação ✅ VIGENTE

Fluxo:
1. recebe todo o elenco disponível;
2. tenta `PreferredFormation`;
3. cria os slots;
4. procura combinação válida de 11 jogadores únicos;
5. prioriza Natural/Secundária;
6. se necessário, improvisa apenas nos slots sem solução adequada;
7. entre candidatos possíveis, compara o encaixe no slot com `ScoreNoSlot`;
8. GK exige goleiro;
9. 11 viram titulares;
10. 5 restantes viram banco.

`ScoreNoSlot` **não é fórmula paralela do motor** e não cria penalização de improvisação.
Ele reutiliza as qualidades e os pesos posicionais já vigentes:

`ScoreNoSlot = (QualidadeAtaque × PesoAtaque + QualidadeDefesa × PesoDefesa) / (PesoAtaque + PesoDefesa)`

Para GK:

`ScoreNoSlot = QualidadeGoleiro`

Se a formação preferida não puder ser montada, a AI tenta as demais formações
suportadas. Se nenhuma delas produzir 11 jogadores únicos com goleiro real no slot GK,
retorna **falha de domínio**; não inventa escalação fake nem relaxa silenciosamente as
invariantes.

---

## 9. TeamSetup ✅ VIGENTE

A escalação do usuário é uma configuração persistente do trabalho atual do técnico,
não um campo solto dentro de Club e não apenas dado de uma Match.

Conceito:
`TeamSetup`
- ManagerId;
- ClubId;
- Formation;
- Slots.

O usuário monta e salva a equipe; os 11 slots formam os titulares e os 5 jogadores
restantes ficam no banco. A configuração continua para jogos seguintes até ser alterada.

Quando uma Match nasce, tira fotografia própria da configuração válida. Alterar o
TeamSetup depois não altera a Match já iniciada.

Se jogador escalado deixa o clube, a configuração fica inválida e nova partida é
bloqueada até correção; não apagar silenciosamente o TeamSetup.

---

## 10. Cadastros React do núcleo inicial ✅ VIGENTE

Usar telas normais do produto, não wizard descartável e não apenas seed.

Áreas:
- Clubes;
- Jogadores;
- Técnicos;
- Escalação;
- Partida.

Jogadores são cadastro global independente, podem existir sem clube e permitem
cadastro individual/lote, edição, consulta e vínculo/desvínculo.

Cadastro em lote usa uma grade de vários jogadores, pode receber um clube comum
opcional para vínculo, aplica as mesmas validações do cadastro individual e é
atômico: linha inválida cancela o lote inteiro.

Lista de jogadores possui busca por nome, filtro por clube atual/sem clube, filtro
por posição natural e paginação server-side simples (aproximadamente 20 por página);
elenco de um clube mostra todos os jogadores sem paginação.

Técnicos possuem área global equivalente, incluindo Human/Cpu e PreferredFormation
para CPU.

Exclusão física não é prioridade do primeiro marco.

---

## 11. MatchId, nascimento e histórico inicial ✅ VIGENTE

`MatchId` identifica **uma execução concreta** e nunca é reutilizado.

Quando a execução nasce de verdade:
1. validar os dois lados;
2. gerar MatchId;
3. persistir Match como `InProgress`;
4. persistir fatos iniciais necessários ao histórico;
5. iniciar execução viva em memória.

Persistir no início:
- MatchId;
- status `InProgress`;
- StartedAt;
- ClubId + ClubName histórico dos dois lados;
- ManagerIds;
- todos os 16 relacionados de cada lado;
- PlayerId + PlayerName usado naquela partida;
- papel inicial Starter/Substitute;
- Seed;
- EngineVersion como metadados técnicos internos.

Não persistir a cada minuto:
- minuto atual;
- placar parcial;
- RNG corrente;
- estado tático vivo;
- eventos provisórios.

Não existe checkpoint/resume.

---

## 12. Súmula e histórico ✅ VIGENTE

Todos os 16 relacionados pertencem à súmula, inclusive reservas que não entraram.

Preservar identidade histórica suficiente:
- PlayerId;
- PlayerName usado na partida;
- ClubId;
- ClubName usado na partida;
- papel inicial Starter/Substitute.

Não congelar atributos, overall ou evolução futura completa.

Não criar agregado persistido duplicado `MatchReport`.

Persistir:
- Match;
- RelatedPlayers;
- MatchEvents.

O Game compõe a visão da súmula por `GET /matches/{MatchId}/report`.

Não tratar uma única formação como "a formação da partida inteira".

---

## 13. Experiência da partida e SignalR ✅ VIGENTE

Objetivo de UX:
- 90 minutos de jogo;
- 6 minutos reais de bola correndo;
- 1º tempo = 3 minutos reais;
- intervalo = 30 segundos reais;
- 2º tempo = 3 minutos reais;
- 1 minuto de jogo = 4 segundos reais.

Ao zerar o intervalo, o segundo tempo começa automaticamente.

SignalR entra na primeira partida interativa.

Atualizações iniciais incluem minuto/estado, placar, novos eventos e transições
`Running`, `Paused`, `Halftime` e `Finished`.

Fluxo:
- comandos: React → HTTP → Game;
- atualizações: Game → SignalR → React.

Ao reabrir a tela, HTTP recupera fotografia atual e SignalR acompanha as mudanças
seguintes.

F5, aba fechada ou queda de SignalR não mudam MatchId/Seed, não matam execução e não
pausam automaticamente. A partida pertence ao servidor.

---

## 14. Pausa manual e intervalo ✅ VIGENTE

Existem **3 pausas manuais por partida**.

Cada pausa:
- dura no máximo 60 segundos reais;
- congela a simulação;
- não processa minuto;
- não avança RNG;
- não muda λ apenas por existir;
- pode ser encerrada antes pelo comando `Continue`;
- retoma automaticamente ao chegar a zero se o usuário não continuar antes.

Estado vivo em memória:
- Running;
- Paused;
- Halftime.

Status persistido continua:
- InProgress;
- Finished;
- NotCompleted.

Pausas obrigatórias por lesão/vermelho permanecem futuras.

---

## 15. Pausas × paradas × substituições ✅ VIGENTE

Contadores separados:
- 3 pausas manuais;
- 3 paradas de substituição com bola rolando;
- 5 substituições totais.

Abrir pausa consome pausa manual.

Se apenas consultar/mudar algo sem substituição, não consome parada de substituição.
Se confirmar uma ou mais substituições naquela pausa, consome uma parada.

Trocas no intervalo não gastam pausa nem parada, mas consomem o limite total de 5.

---

## 16. Sair, desconexão e crash ✅ VIGENTE

### Navegador desconectado

Não interrompe partida. Usuário pode voltar e recuperar fotografia atual via HTTP.

### Sair explícito

Encerra execução como `NotCompleted`.
Não é W.O. ainda, não gera resultado oficial e não publica `PartidaEncerrada`.

W.O. e um possível `Reinício rápido` ficam para o futuro; a identidade de uma
eventual execução reiniciada só será decidida quando essa feature existir.

### Crash do Game

Estado vivo em memória se perde. `InProgress` órfã é marcada `NotCompleted` na
subida do Game.

Não reconstruir minuto, placar parcial, eventos provisórios, RNG ou escalação viva.

---

## 17. Finalização transacional e Outbox ✅ VIGENTE

Final normal persiste na mesma transação:
- placar final;
- eventos definitivos;
- Match → Finished;
- FinishedAt;
- MassTransit Transactional Outbox com EF Core.

Depois do commit, infraestrutura publica `PartidaEncerrada`. Broker indisponível não
invalida o resultado já persistido; a intenção registrada no Outbox pode ser entregue depois.

Se persistência final falhar enquanto o processo ainda vive, pode repetir a gravação
do mesmo resultado já calculado. Não rodar MatchEngine novamente e não gerar nova
Seed.

Se o processo morrer antes do commit, a execução se perde e o `InProgress` inicial
eventualmente vira `NotCompleted`.

Não criar Outbox caseira.

---

## 18. `PartidaEncerrada` ✅ VIGENTE

Primeiro evento de integração real.

Semântica: fato → `Publish`.

Corpo:
- EventId;
- MatchId;
- Home: ClubId, ClubName, ManagerId;
- Away: ClubId, ClubName, ManagerId;
- GolsMandante;
- GolsVisitante;
- EncerradaEm.

Header:
- CorrelationId.

Não incluir escalações, atributos, λ, Seed, EngineVersion, súmula, timeline completa
ou texto de UI.

`MatchId` = execução concreta da partida.
`EventId` = ocorrência do evento de integração.

---

## 19. Manager Inbox ✅ VIGENTE

Primeiro tipo: `MatchResult`.

Modelo mínimo `InboxItem`:
- Id;
- ManagerId;
- Type;
- Title;
- Message;
- MatchId;
- IsRead;
- CreatedAt.

Não incluir EventId, CorrelationId, ReadAt, Seed/EngineVersion ou cópia da súmula.
Sem herança para tipos.

### Multiplicidade

Ao consumir `PartidaEncerrada`, o Inbox cria **um `MatchResult` para cada Manager da partida**:
- um para `Home.ManagerId`;
- um para `Away.ManagerId`.

Os dois itens referenciam o mesmo `MatchId`; cada um pertence ao seu próprio `ManagerId`.
Essa multiplicidade é coerente com a chave de idempotência por Manager.

### Leitura

- lista não marca lido;
- abrir detalhe faz `IsRead: false → true`;
- nunca volta a false.

### Idempotência

Chave de negócio:
`ManagerId + MatchId + Type`.

Constraint UNIQUE. Duplicata equivalente = no-op. Falha real = retry básico e depois
`_error` conforme MassTransit.

### Súmula

Inbox guarda MatchId. "Ver súmula" chama o Game; Inbox não copia a súmula.

---

## 20. Primeira intervenção esportiva ✅ VIGENTE

Primeira intervenção real:
> substituição manual do Manager Human.

Durante pausa ou intervalo:
- escolhe quem sai;
- escolhe quem entra;
- Game valida;
- composição em campo muda;
- substituição é registrada;
- força é recalculada;
- λ futuro pode mudar.

O passado não é recalculado.

Regras mínimas:
- entra: relacionado e no banco;
- sai: em campo;
- quem saiu não volta;
- sem jogador duplicado;
- continuam 11 em campo;
- máximo 5 trocas;
- máximo 3 paradas;
- intervalo não consome parada.

MatchEvents do recorte: Goal e Substitution.

Ainda fora: tática/formação livre, lesão, cartão, cansaço e substituição inteligente
da CPU.

---

## 21. Walking skeleton / marcos atuais ✅ VIGENTE EM NÍVEL DE DECISÃO

### Marco 1 — Cadastros e preparação
Club/Player/Manager/vínculos, React real, 16 jogadores por clube, TeamSetup humano e
SimpleManagerAI. Sem partida e sem RabbitMQ.

### Marco 2 — Primeira partida interativa
Match InProgress + execução em memória + MatchEngine minuto a minuto + duração real +
intervalo + SignalR + pausas manuais + final normal + histórico/súmula. Sem substituição.

### Marco 3 — NotCompleted
Sair/crash/orfandade de InProgress, novo MatchId em nova execução, sem resume.

### Marco 4 — Outbox + RabbitMQ + PartidaEncerrada
Finalização transacional e Publish real.

### Marco 5 — Manager Inbox E2E
Consumo, persistência, idempotência, leitura e consulta de súmula no Game.

### Marco 6 — Primeira intervenção esportiva
Substituição humana, limites, recálculo apenas futuro e MatchEvent de substituição.

> O detalhamento oficial destes marcos está no `roadmap-jogo-futebol.md`, consolidado
> na Fase B2. Este resumo registra a decisão arquitetural vigente e substitui o roadmap
> antigo da sabatina.

---

## 21A. MassTransit / retry / fila de erro ✅ VIGENTE

As decisões de ferramenta da sabatina que não dependiam do Opponent Service
continuam preservadas:

- MassTransit **v8** como biblioteca escolhida para este projeto de aprendizado;
- retry imediato básico: `Immediate(3)`;
- sem redelivery atrasado no primeiro fluxo de integração;
- falha após as tentativas segue para a fila `_error` padrão;
- `MassTransit Transactional Outbox` com EF Core no Game;
- idempotência de efeito no Manager Inbox através da chave de negócio + constraint
  UNIQUE (`ManagerId + MatchId + Type`).

> **Correção de contexto:** no documento antigo essas decisões estreavam no
> "marco 1" da topologia Game ↔ Opponent. Esse timing foi superado. Hoje elas entram
> quando existe o fluxo real `PartidaEncerrada` → Manager Inbox. A política técnica,
> porém, não foi substituída pela Fase A.


---

## 22. Chão de engenharia ✅ VIGENTE

Preservar:
- Clean Architecture;
- DDD onde protege regra real;
- invariantes no domínio;
- Result Pattern para falhas esperadas;
- casos de uso explícitos;
- EF Core;
- testes e pirâmide de testes;
- Testcontainers quando infra real for necessária;
- logging estruturado;
- CorrelationId;
- docker-compose;
- CI desde o começo;
- ADR/notinha apenas para decisão significativa.

Evitar catálogo de buzzwords sem necessidade: Redis, Kubernetes, OpenTelemetry,
Event Sourcing, Saga, AutoMapper, Specification, Unit of Work custom,
microsserviços como objetivo etc.

---

## 23. Cloud ✅ ADIADA

Haverá etapa posterior de deploy real em nuvem.

Não fixar AWS/Azure/ECS/RDS/Amazon MQ ou topologia cloud agora.

AWS continua apenas candidata de aprendizado/deploy. A aplicação existente deve
dizer o que precisa ser hospedado antes da escolha.

---

## 24. Evoluções nomeadas, mas não desenhadas

Manter apenas como bandeiras:
- Competition/campeonato;
- vários clubes;
- calendário/rodadas/classificação;
- eventos ricos: cartão, lesão, pênalti etc.;
- pausa obrigatória;
- cansaço;
- CPU mais inteligente em jogo;
- mudança tática/formação dinâmica;
- instruções condicionais;
- autenticação/autorização;
- admin para edição de atributos;
- multiplayer/humano futuro;
- contratos;
- diretoria;
- torcida;
- evolução do jogador;
- dados reais externos;
- motor mais detalhado;
- calibração séria;
- cloud/deploy.

---

## 25. Plataforma de execução, nomes e formato da solution ✅ VIGENTE

**Data:** 17/09/2026. **Substitui:** a decisão de .NET 8 registrada no handoff §6 e no
Mapa Mestre §4, e os nomes `manager-football` / `ManagerFootball.sln`.

### 25.1 Contexto

A implementação da Parte A começou entre 09/09 e 11/09/2026 e produziu um estado que
divergia da documentação em três pontos: os projetos nasceram em `net10.0`, o `global.json`
foi fixado em `10.0.401`, e a solution nasceu como `Cartola90.slnx`.

Pela regra de precedência do projeto, divergência entre documento e código se resolve
corrigindo o derivado — nunca por inércia do que já existe. As três divergências foram
portanto reabertas como decisão explícita, e não homologadas por comodidade.

### 25.2 Decisão — .NET 10

O projeto adota **.NET 10**.

Razões, na ordem em que pesaram:

1. **Último momento responsável.** A decisão precisava ser tomada antes de A7, quando a
   versão do EF Core passa a estar escrita em pacotes e migrations. Depois disso, o custo
   sai de "editar cinco linhas" para "refazer infraestrutura".
2. **Elimina uma dependência que só existia por causa da versão.** O `UUIDNext` estava no
   desenho unicamente porque o .NET 8 não gera UUIDv7. O `Guid.CreateVersion7()` é nativo a
   partir do .NET 9. Manter o .NET 8 significaria adicionar biblioteca de terceiros para
   gerar identificador — contra o princípio "problema antes de ferramenta".
3. **Janela de suporte.** O suporte do .NET 8 termina em novembro de 2026, dentro da vida
   prevista deste projeto; o .NET 10 é LTS até novembro de 2028.

**Divergência registrada.** A decisão original pelo .NET 8 foi tomada conscientemente,
com o fim do suporte já conhecido, e tinha uma razão legítima: treinar na geração que
muitas empresas de fato operam. Esse argumento não foi invalidado — foi considerado menos
determinante para *este sistema* do que o acúmulo de dívida técnica. Fica registrado para
que a troca não pareça, no futuro, um esquecimento.

**Consequências imediatas:**

- `UUIDNext` **não entra** no projeto;
- EF Core passa a ser a linha 10;
- o `.slnx` torna-se viável (ver 25.4).

### 25.3 Decisão — nomes oficiais

```text
repositório .... Cartola90
solution ....... Cartola90.slnx
```

Substituem `manager-football` e `ManagerFootball.sln`. Escolha do autor, mantida por
preferência pessoal declarada. Não é decisão arquitetural e não altera nenhuma fronteira.

**Pendência derivada:** o `README.md` versionado ainda se intitula "Manager Football" e
precisa ser alinhado.

### 25.4 Decisão — formato `.slnx`

O projeto adota o formato **`.slnx`**.

A exigência anterior de `.sln` tinha causa técnica concreta, não estética: **o SDK do
.NET 8 não abre arquivos `.slnx`**. Com .NET 10, a restrição desapareceu.

Ganho real: a `.sln` clássica guarda GUIDs por projeto e blocos de aninhamento que
produzem conflitos de merge difíceis de resolver. O `.slnx` é XML legível.

**Requisito derivado para A12:** o runner do GitHub Actions precisará de SDK 10, senão
`dotnet build` não reconhece o arquivo.

### 25.5 O que esta seção NÃO decide

Nada sobre arquitetura, fronteiras, domínio, matemática do motor ou mensageria. As seções
1 a 24 permanecem integralmente vigentes. Esta seção trata apenas da plataforma de
execução e de nomes de arquivos.

---

# PARTE II — DETALHAMENTO E HISTÓRICO DA SABATINA

> **Como ler a parte abaixo.** Ela preserva a pesquisa, matemática, calibração e
> raciocínio acumulados. Trechos marcados como **SUPERADO** são história e não regra
> atual. Trechos matemáticos sem conflito permanecem válidos e complementam a Parte I.

## Seção 4D — Quem roda a simulação ⚠️ SUPERADA PARCIALMENTE PELA FASE A

> **O que permanece válido:** o `Game` é a autoridade da partida e o `MatchEngine`
> roda dentro dele, isolado atrás de sua fronteira interna.
>
> **O que foi superado:** a premissa de um `Opponent Service` separado, o papel
> mandante/visitante como fronteira de processos e a necessidade de preparar um
> futuro juiz neutro como evolução já direcionada. O CPU agora é `Manager AI`
> interno ao Game.


**Decisão principal:** Modelo "tudo no servidor" (estilo site web), com o serviço
do jogo rodando a simulação no começo, e o motor destacável para o futuro.

### O que foi decidido, item a item:

1. **Modelo de execução: tudo no SEU servidor (Modelo 1 / "site web").** ✅
   - Os dois serviços (jogo e adversário) rodam na sua infraestrutura.
   - O jogador humano NÃO instala nada — ele só abre uma **tela (React) no
     navegador**, que conversa com os seus serviços pela API.
   - Descartado o "Modelo 2" (cada jogador roda um pedaço na própria máquina),
     porque: (a) criaria o problema real de trapaça ("nunca confie no cliente"),
     (b) daria trabalho de distribuir/atualizar sem ensinar nada útil, e (c)
     brigaria com o assíncrono (se o jogador desligasse o PC, a partida sumiria).

2. **Confiança/trapaça: não é problema neste projeto.** ✅
   - Como os dois serviços são seus e rodam no seu servidor, ninguém hostil
     controla a simulação. O fantasma de trapaça só existiria no Modelo 2.

3. **Quem simula no começo: o serviço do jogo (Porta A).** ✅
   - O serviço do jogo é quem "segura o relógio" e roda a partida.
   - Justificativa que torna isso legítimo (e não gambiarra): o **mandante** é uma
     autoridade natural do domínio (jogo em casa), então a assimetria tem respaldo
     no futebol real — não é um "por que A e não B?" arbitrário.

4. **Motor plugável atrás de contrato — permanece válido por cima da Porta A.** ✅
   - O *como simular* fica ISOLADO atrás de um contrato:
     - **Entra:** Time A + Time B + táticas + instruções de cada lado.
     - **Sai:** placar + lista de eventos da partida.
   - Assim, no futuro dá para ARRANCAR o motor e botar num serviço juiz neutro
     (Porta B) **sem reescrever o resto** — só religando o cano.
   - Regra de ouro registrada: Porta A *pura* (motor grudado no serviço do jogo)
     seria gambiarra; Porta A *com motor destacável* é desenho bom.

### O que fica para depois (não decidido aqui):
- Migrar a simulação para um serviço juiz neutro (Porta B) — possível no futuro,
  sem reprojetar, graças ao motor destacável.

---

## Seção — Quantos serviços existem no dia 1 ❌ SUPERADA PELA FASE A

> **Histórico.** O desenho abaixo descreve Game + Opponent Service e não deve ser
> usado para implementação nova. No estado atual, o núcleo começa com `Game`; o
> primeiro processo separado de domínio é `Manager Inbox`, introduzido quando
> nasce a integração por `PartidaEncerrada`.


**Decisão: 2 serviços no dia 1.**

### O desenho do dia 1:
- **Serviço do jogo** — o cérebro. Guarda os clubes, roda a simulação (motor
  dentro dele, atrás do contrato), segura o relógio da partida. Faz o papel de
  **mandante**. → 1 processo.
- **Serviço do adversário** — quem toma as decisões do lado visitante. No dia 1 é o
  **robô simplório**. Depois, o humano entra pela mesma porta (item 6). → 1 processo.
- **RabbitMQ** no meio — infra de mensageria, não conta como "serviço de domínio".
- **Tela React** — uma por jogador, roda no navegador, conversa por HTTP. **NÃO é
  serviço** (não recebe mensagem de broker; é só a cara do sistema).
### Banco — a definir se é um ou um-por-serviço (próximo fio).

## Seção 4H (parcial) — Banco de dados ⚠️ SUPERADA PARCIALMENTE PELA FASE A

> **Princípio preservado:** processos separados são donos de sua persistência e
> não compartilham tabelas como atalho. **Topologia superada:** não existe mais um
> banco do Opponent Service. `Game` e `Manager Inbox` possuem persistências próprias
> quando ambos existirem.


**Decisão: dois bancos, um por serviço, desde o dia 1.**

### Motivo (importante — NÃO é "medo de protelar"):
- Separar banco depois é genuinamente **caro** (dois serviços que dividem tabela
  ficam grudados por baixo do pano — a "separação mentirosa"). Regra conhecida:
  *database-per-service* (cada serviço é dono do seu banco; ninguém enxerga o banco
  do outro; só conversam por mensagem).
- Fazer agora custa **pouco**: é ~uma linha a mais no docker-compose, sem lógica
  nova. Assimetria: **barato agora + caro depois → adianta.**

### Distinção-chave registrada (a cicatriz do ERP360):
- **Veneno 1 (o que mordeu o Brian no ERP360):** código FAKE fingindo ser real
  (mock/stub/tampão deixado no lugar de implementação). → **NUNCA fazer.** Tudo que
  existe no código existe de verdade.
- **Veneno 2 (o oposto, igualmente ruim):** construir tudo no talo no dia 1 "pra
  não protelar" → incha o projeto, mata o esqueleto que anda.
- **Caminho certo:** cada peça é REAL e HONESTA, mas o projeto cresce em fatias.
  "Menos features" é saudável; "features fake" é o câncer. Começar com menos ≠
  começar com mentira.

## Juiz (Porta B) — HISTÓRICO / NÃO É DIREÇÃO ATUAL

O Brian perguntou se valia já fazer o juiz de uma vez (aplicando a régua do banco).
Resposta: **não** — a régua dá resposta OPOSTA aqui, porque os custos são opostos.

- **Custo de adiantar o juiz agora: ALTO.** Não é "um serviço a mais" — arrasta uma
  **fronteira de mensageria inteira** (serviço do jogo ↔ juiz ↔ adversário), com
  todos os padrões da 4E (Outbox, idempotência, DLQ, "e se o juiz cair?"). E
  **mata o esqueleto que anda**: o primeiro esqueleto deixaria de ser fino, com 3
  serviços e 2 fronteiras pra debugar ao mesmo tempo (3 suspeitos em vez de 1).
- **Custo de adiar: BAIXO.** O motor já é destacável por contrato, então migrar
  Porta A → Porta B depois é "religar o cano", sem reescrever o resto.
- **Assimetria final:** Banco = barato agora + caro depois → adianta. Juiz = caro
  agora + barato depois → espera. Mesma régua, respostas opostas.
- Adiar o juiz **não** gera tampão fake: a Porta A é real e honesta (o serviço do
  jogo simula de verdade). É menos peças, não peças falsas.

### FOTO DA INFRA DO DIA 1 (fechada):
- Serviço do jogo (motor dentro, atrás do contrato) + seu banco
- Serviço do adversário (robô) + seu banco
- RabbitMQ no meio
- Tela React por jogador (navegador, não é serviço)
- Sem juiz, sem login

### Confusões desfeitas nesta seção (importantes):
1. **Tela ≠ serviço.** A tela é a cara do sistema, roda no navegador, fala HTTP.
   Não entra na contagem de serviços.
2. **Mandante/visitante ≠ dois serviços.** São **papéis dentro de uma partida**,
   não dois programas. Os dois serviços são "quem coordena a partida" vs "quem
   decide as jogadas do adversário" — não "um time, um serviço".
3. **Não existe "meu jogo" e "jogo dele".** Existe UM jogo no servidor central
   (arquitetura **cliente-servidor com servidor autoritativo**), e os dois
   jogadores se conectam como telas — igual xadrez online (Chess.com/Lichess). O
   servidor é a autoridade final sobre a verdade da partida.

### Modelo de jogo escolhido (decisão de design, fechada aqui):
- **Caminho 1 — jogo compartilhado no servidor central**, os dois lados conversando
  durante a partida inteira (mensageria de verdade cruzando o tempo todo). ✅
- Descartado o **Caminho 2** ("cada um dono do seu mundo, trocando cartões de
  time"): é um jogo legítimo, mas seria quase single-player e **esvaziaria** a
  mensageria (o coração de aprendizado, itens 5 e 7). Sem conversa durante a
  partida, RabbitMQ/fronteira/pausar-e-reagir virariam enfeite.

### Terceiro serviço (juiz/Porta B) — considerado e ADIADO de propósito:
- Nascer com 3 serviços violaria o esqueleto que anda (item 11): mais fronteira,
  mais docker-compose, mais mensagem — antes de provar que o básico anda.
- Como o motor já é destacável por contrato, separar depois é **barato** (religar o
  cano, não reescrever). Logo, não se perde nada esperando.

### Login fora do dia 1 ✅
- Começa com o robô como adversário → não precisa de autenticação no primeiro
  esqueleto. (confirmado pelo Brian)

---

## Seção 4A (parcial) — Jogador: força do time e modelagem ✅ FECHADO (vários pontos)

### Força de ataque/defesa: DERIVADA dos jogadores (Jeito B) ✅
- A força do time é calculada a partir dos atributos dos 11 escalados (não é um
  número escrito na mão). Mais fiel (trocar jogador muda a força) e já conecta o
  domínio rico ao motor desde o dia 1 — o "clube rico" (item 2) não fica decorativo.
- Custo aceito: precisa metrificar "como atributos viram força" já no dia 1.

### Jogos simples vs avançados (referência de design):
- **Simples (Brasfoot):** poucos atributos, cada um "gordo" (um atributo faz várias
  coisas). Conta enxuta e direta.
- **Avançado (FM):** ~40 atributos, especialização fina, trabalham em conjunto;
  mentais AMPLIFICAM técnicos/físicos (finalização 17 + compostura 9 = desperdiça
  chance). 
- **Regra de subida:** degrau 1 = "Brasfoot" (poucos/gordos); degrau 3 = "FM"
  (muitos/combinando). Sobe de um pro outro pelos degraus — o mapa já suporta.

### OS 6 ATRIBUTOS GORDOS DO DIA 1 (a paleta `[D1]`) ✅ [VALORES VIGENTES; ORIGEM CORRIGIDA NA FASE A]

> **Correção Fase A:** os seis atributos continuam exatamente estes, mas agora são
> **cadastrados diretamente em 0–100**. A descrição histórica abaixo de
> "descompactação fino→gordo" não é a origem matemática atual dos valores. Os
> atributos finos permanecem apenas como mapa futuro. As qualidades derivadas atuais
> são `QualidadeAtaque = (Finalização + Drible + Passe + Ritmo + Físico)/5` e
> `QualidadeDefesa = (Defesa + Ritmo + Físico)/3`.

- **Escolha do Brian:** **Ritmo, Finalização, Drible, Passe, Defesa, Físico.**
- **O que são:** praticamente os SEIS atributos de card do FIFA / EA FC (Pace,
  Shooting, Passing, Dribbling, Defending, Physical). Modelo comprovado de "poucos e
  gordos" — Brian chegou nele sozinho.
- **Cada gordo engole vários finos do `mapa-jogador`** (é a "descompactação por
  degrau" na prática, a confirmar fino a fino quando subir de degrau):
  - Ritmo ← Velocidade (+ Agilidade parte)
  - Finalização ← Finalização (+ Cabeceio na finalização)
  - Drible ← Drible
  - Passe ← Passe curto + Passe longo (+ Visão de jogo parte)
  - Defesa ← Desarme + Marcação + Posicionamento defensivo
  - Físico ← Força + Resistência + Agilidade + Impulsão
- **Amarra com a regra de subida:** os 6 são a CARA do dia 1; cada degrau futuro só
  descompacta um gordo nos seus finos, sem retrabalho.
- ⬜ **Falta fechar (próximos micro-fios da metrificação do dia 1):**
  1. ✅ Split ataque×defesa (jogador de linha): **ATAQUE ← Finalização, Drible,
     Passe, Ritmo, Físico; DEFESA ← Defesa, Ritmo, Físico.** Ritmo e Físico contam
     nos DOIS lados (decisão do Brian) — um time rápido/forte é melhor atacando E
     defendendo. (+ o goleiro entra na defesa, ver abaixo.)
  2. ✅ **Goleiro — FECHADO.** Os SEIS do goleiro (Mergulho, Manuseio, Chute,
     Reflexos, Velocidade, Posicionamento) entram TODOS na força de DEFESA no dia 1,
     Chute incluído (distribuição do goleiro fica pra depois). A ideia do Brian de
     "goleiro cobrador de falta/pênalti" é um **TRAÇO** (campo próprio que o motor lê
     direto, tipo Flair — ver decisão #2), NÃO um atributo derivado — e só liga no
     `[G2]`, quando existir o evento bola parada pra cobrar (no dia 1 a moedinha só
     cospe gol/não-gol, não há pênalti). Guardado como traço `[G2]`, não perdido.
  3. ✅ **Agregação — CIENTE DA POSIÇÃO (decisão do Brian), não média cega.** A
     contribuição de cada jogador é pesada pela função (atacante puxa o ataque,
     zagueiro puxa a defesa). É também o gancho natural pra formação/tática morderem
     o resultado (bloco C) já no dia 1. Granularidade FECHADA: **posições
     específicas** (estilo FIFA: LB, CB, ST…). FORMA DO PESO FECHADA na sessão 3 —
     ver "MODELO DE TRÊS CONCEITOS" abaixo (Opção 2: dois pesos soltos por posição +
     aptidão posicional). Ainda aberto: os NÚMEROS da tabela posição→peso; o peso do
     goleiro na defesa (não é 1/11). A aptidão FECHOU: **desligada no dia 1** (todo
     mundo tratado como natural; a penalização fica pronta no código e liga numa
     etapa seguinte).
  4. Fórmula força → λ (gols esperados) → chance da moedinha por minuto.

### MODELO DE TRÊS CONCEITOS na agregação ⚠️ SUPERADO PARCIALMENTE NA FASE A

> **Permanece:** qualidade do jogador + peso da posição são conceitos separados.
> **Mudou:** aptidão não é mais um terceiro multiplicador persistido/pré-construído
> no recorte inicial. O Player guarda `NaturalPosition` + `SecondaryPositions`;
> improvisação é derivada quando o slot não aparece em nenhuma delas. Penalização
> numérica continua desligada e só será desenhada se a feature realmente entrar.

O Brian separou três coisas que quase todo mundo mistura — e separou certo. O FM faz
a MESMA separação (pesquisado, não de memória: FM26 distingue posição × papel ×
familiaridade posicional). São três coisas independentes:

1. **Qualidade do jogador** — o que ele sabe fazer. São os dois números que cada
   jogador já produz pelo split: um de ataque (Fin+Drible+Passe+Ritmo+Físico) e um
   de defesa (Defesa+Ritmo+Físico). Ex: 72 de ataque / 78 de defesa. NÃO muda com a
   posição.
2. **Peso da posição** — quanto AQUELA POSIÇÃO influencia o ataque e a defesa do
   time. **Opção 2 (escolha do Brian): dois pesos SOLTOS por posição**, um de ataque
   e um de defesa, sem amarração de gangorra (rejeitada a Opção 1 do "um número só").
   Justificativa: coerente com a decisão já batida de Ritmo/Físico contarem nos DOIS
   lados — uma posição pode pesar nos dois. Ex do Brian: mesmo jogador rende
   diferente conforme o SLOT — LB ≈ 0,45 ataque / 0,80 defesa; LWB ≈ 0,70 / 0,60.
   Trocar o slot troca os PESOS, não os atributos do jogador. (Vale igual pra DM×CM.)
3. **Aptidão posicional** — quanto daquele jogador a gente CONSEGUE APROVEITAR naquele
   slot específico. É a **penalização por improvisação**. Três níveis (escolha do
   Brian): natural = aproveitamento total; secundária (adaptada) = pouca ou nenhuma
   penalização; improvisada = penalização maior. Um lateral-direito jogado de ponta-esquerda que
   nunca atuou ali não deve render igual. FM confirma o conceito (lá são ~6 níveis:
   Natural→Accomplished→Competent→Unconvincing→Awkward→Ineffectual; três é a versão
   enxuta pro dia 1).

**Como os três se combinam (a forma da conta, por lado — ataque mostrado, defesa é
igual):** para cada um dos 11 no slot p →
`contribuição = peso_da_posição(p) × aptidão(jogador,p) × qualidade(jogador)`.
A força do ataque do time = soma das contribuições ÷ soma dos pesos de ataque.
Detalhe que faz a improvisação DOER de verdade: a aptidão entra só em CIMA da fração
(no de baixo fica só o peso). Assim um improvisado contribui menos, mas o slot dele
ainda conta no divisor → puxa a força do time pra baixo (era o efeito desejado).

**Honestidade (marca pro 4J):** a DIREÇÃO dos pesos tem respaldo real (atacante puxa
ataque, zagueiro puxa defesa). Os NÚMEROS exatos (pesos e penalizações) são
HEURÍSTICA — chute de design informado, não fórmula. Nota de refinamento `[H]`: o FM
"cirúrgico" derruba só Posicionamento e Decisão do improvisado, não o jogador todo;
no dia 1 esses atributos nem existem separados (são `[H]` no mapa), então o
multiplicador chapado (aproveitamento único) é a simplificação honesta agora, com a
versão cirúrgica reservada pra quando esses atributos existirem.

### FORMAÇÕES E LISTA DE POSIÇÕES DO DIA 1 ✅ [FECHADO na sessão 3 — Caminho B]
- O dia 1 nasce com **um punhado de formações comuns: 4-4-2, 4-3-3, 3-5-2** (Caminho
  B). Escolher só uma formação (Caminho A) foi rejeitado porque matava o caso
  LB×LWB — justo o motivo de o Brian ter escolhido posições específicas.
- A lista de posições é a SOMA das que essas formações usam (~14 reais; direita e
  esquerda são espelho, mesmo peso, então lista-se por TIPO): GK, CB, RB/LB, RWB/LWB,
  DM, CM, AM, RM/LM, RW/LW, ST.

### TABELA POSIÇÃO→PESO ✅ LINHA FECHADA / 🔶 goleiro pendente (sessão 3)
Dois pesos soltos por posição (Opção 2). Números RELATIVOS (o que vale é a proporção
entre posições do mesmo lado; a conta divide pela soma dos pesos). Dois valores
ancorados no exemplo do Brian de propósito (lateral e ala).

**LINHA ✅ FECHADA (sessão 3, boa o suficiente pro dia 1).** Goleiro segue 🔶 (próximo
fio). Ajustes do Brian ao rascunho: RM/LM subiu pra 0,65/0,45 (faixa/lado assume mais
responsabilidade ofensiva que o CM — respaldo em estudos de função posicional); DM
subiu pra 0,45/0,75 (o peso BASE precisa ser a média de holding / ball-winning /
deep-lying playmaker, já que função tática só entra depois — não é só "cão de guarda").

| Posição (tipo) | Peso ataque | Peso defesa |
|---|---|---|
| ST — atacante | 0,90 | 0,10 |
| RW/LW — ponta | 0,80 | 0,25 |
| AM — meia ofensivo | 0,75 | 0,30 |
| RWB/LWB — ala | 0,70 | 0,60 |
| RM/LM — meia de beirada | 0,65 | 0,45 |
| CM — meia central | 0,55 | 0,55 |
| RB/LB — lateral | 0,45 | 0,80 |
| DM — volante | 0,45 | 0,75 |
| CB — zagueiro | 0,15 | 0,90 |
| GK — goleiro | 0,00 (não entra no ataque no dia 1) | mistura à parte — ver abaixo |

> Ressalva registrada (Brian): a pesquisa dá respaldo à DIREÇÃO e à separação de
> papéis, não a números específicos como 0,65 ou 0,45. Continuam sendo parâmetros de
> game design, a calibrar quando o motor rodar.

### GOLEIRO na defesa — Jeito 2 (mistura fixa) ✅ [FECHADO sessão 3]
O goleiro NÃO entra como "1 de 11" na média. Ele sai da média e entra por uma mistura
fixa, porque balança o gols-sofridos mais que qualquer jogador de linha sozinho
(pesquisa: impacto dominado pela defesa de chute; um goleiro que evita ~8 gols/temporada
acima do esperado vale ~5,5-6,5 pontos na tabela). Fórmula:

`defesa do time = 0,70 × (defesa média dos 10 de linha) + 0,30 × (qualidade do goleiro)`

- A "qualidade do goleiro" é o número único que já sai dos 6 atributos gordos dele
  (todos os 6 já alimentavam a defesa — fechado antes).
- Por que o Jeito 2 (e não peso alto na mesma média): a fatia do goleiro fica um número
  legível e ESTÁVEL ("30% da defesa, sempre"), não oscila com a formação — justo a
  posição que menos deveria oscilar.
- O **30%** é botão de game design, a calibrar. Não é verdade científica (não existe
  "goleiro = X% da defesa" pronto).

### PARQUEADO — motor de gols por xG/PSxG/xGOT 🅿️ [melhoria de fase futura, pedido do Brian]
O padrão-ouro pra gerar gols não é força-de-ataque/defesa abstrata: é modelar CHUTES e
a qualidade de cada um. **xG** = chance de um chute virar gol pela situação (ângulo,
distância, etc.), ANTES do chute. **PSxG/xGOT** = chance de virar gol pela forma como o
chute FOI batido (só nos que vão no gol) — serve pra separar finalização do atacante e
defesa do goleiro. Um motor assim geraria chutes com xG e resolveria com PSxG contra o
goleiro, em vez da "moedinha de gol" abstrata por minuto. Casa com os degraus (duelos,
fases) e com o importador de dados reais (football-data.org). NÃO é do dia 1 — é uma
proposta de melhoria pra depois do motor básico rodar. Bandeira plantada.

### FÓRMULA FORÇA → λ — exponencial da diferença ✅ [FECHADA sessão 3]

`λ = 1,35 × 2^((Ataque − DefesaAdversária) / 30)`   (depois: × fator_mando; ver mando)

⚠️ **CORREÇÃO — a "Forma A" (multiplicativa por razão) que eu havia gravado estava
degenerada. O Brian pegou.** Com `AtaqueMédio = DefesaMédia = 50`, os dois 50 se
cancelam e sobra `λ = G_médio × A/D` — a referência do "time médio" não faz NADA na
conta. O fio das âncoras perseguia um número decorativo. Fórmula certa é a
exponencial-da-diferença acima.

Por que a exponencial-da-diferença é a melhor (e não é sair do padrão):
- **Escala de intervalo, não de proporção.** Rating 0-100 é qualidade: a DIFERENÇA de
  pontos tem sentido; a razão (80 é "14,3% melhor" que 70) não tem. Diferença (A−D) é
  o tratamento certo.
- **É a forma log-linear do próprio Dixon-Coles.** O padrão é `log λ = intercepto +
  ataque − defesa + casa` → `λ = base × e^(ataque−defesa)…`, que é exatamente
  `2^((A−D)/escala)`. A razão de ratings crus era a versão pior.

Parâmetros (centralizados em `MatchSimulationParameters`, todos calibráveis):
| Parâmetro | Inicial | Função |
|---|---:|---|
| BaseGoals (neutro) | 1,35 | gols esperados quando A = D (gap 0) — **calibrado por simulação (sessão 4)** |
| DifferenceScale | 30 | 30 pts de gap = dobro/metade do λ |
| ReferenceStrength | 70 | **metadado** (gerar jogador / interpretar escala) — NÃO entra na conta |
| MinExpectedGoals | 0,10 | trava de segurança inferior |
| MaxExpectedGoals | 5,00 | trava de segurança superior |

Notas de calibração e propriedades (para não tropeçar depois):
- **Nível absoluto não importa, só o gap:** 90×90 = 50×50 = 1,35. Escolha deliberada e
  consciente (equilibrado → gols-base, independe do tier). Hipótese futura (não dia 1):
  verificar se o nível absoluto da partida precisa gerar um modificador próprio.
- **Neutro ≠ média realizada:** 1,35 é o λ NEUTRO (A=D), não promessa de que a média da
  liga por time será 1,35. Como 2^x é convexa, a média realizada fica ACIMA de 1,35.
  Calibrar contra a média realizada. Instinto solto: 30 pode comprimir confrontos muito
  desiguais, talvez ~25 após simular — mas é botão de calibração, começa em 30.
- **✅ NEUTRO CALIBRADO 1,20 → 1,35 (sessão 4, por SIMULAÇÃO — 500 mil partidas):** a 1ª
  passada com 1,20 saía sã em quase tudo (mando ~45% mandante, empate ~25-27% — no alvo
  real; o medo do "Poisson empata pouco" NÃO apareceu no agregado), MAS gols/jogo ficavam
  ~2,4, abaixo do real (~2,6-2,9). 2ª passada com 1,35 cravou os gols (~2,68 médios iguais
  / ~2,77 com espectro), zebra intacta (~26%), custo pequeno e dentro da faixa (empate cai
  pra ~23,5-25,4%, mandante sobe pra ~46,5-47%). Detalhe: 1,35 é o G_médio que o Brian
  havia proposto lá atrás — a simulação deu razão a ele. Único ponto a afinar DEPOIS (não
  dia 1): 0-0 caiu pra ~6,7% (real ~7-9%) — é exatamente a célula do Dixon-Coles (ajuste
  fino OPCIONAL), e mando ~0,13 traria mandante de 47% de volta pra ~45% se quiser. Ambos
  são fine-tuning de calibração com liga-alvo real; o 1,35 já deixa o motor são.
- **Componente é função PURA** `CalculateExpectedGoals(ataque, defesaAdv)`: sem banco,
  escalação, formação, sorteio, cartão, lesão ou mando. Isoladamente testável. Trabalha
  com decimal. (Proposta do Brian, adotada.)
- **λ → chance da moedinha por minuto ✅:** `p = 1 − e^(−λ/90)` (probabilidade exata de
  sair ≥1 gol no minuto num processo de Poisson). `λ/90` é só a aproximação pra λ pequeno;
  ficamos com a forma exata — custa igual e nunca passa de 1. (Se um dia quiser capturar o
  minuto de 2+ gols, aí seria sorteio Poisson por minuto em vez de moedinha; fora do dia 1.)

**Fork da normalização FECHADO por consequência:** como a fórmula usa (A−D), as forças
PRECISAM ficar em 0-100 → agregação usa **÷ soma dos pesos** (média ponderada). O
"formato da formação pesar sozinho" segue parqueado.

- **MANDO DE CAMPO ✅ [FECHADO sessão 3]:** `λ_final = λ × fator_mando`, multiplicativo
  e simétrico, um só parâmetro `HomeAdvantage = 0,15` → **mandante ×1,15 / visitante
  ×0,85**. Não toca no rating do time; entra só sobre o λ. Base (dado do Brasileirão):
  mandante ~1,51 gol/jogo × visitante ~1,11 (razão ~1,36 → h≈0,15); média geométrica dos
  fatores ~1, então o mando REDISTRIBUI os gols entre os dois sem inflar o total. Ressalva:
  retrato de temporada; mando no Brasil oscila (sem torcida derruba, viagem infla) — 0,15
  é ponto de partida, a simulação calibra (alguns recortes puxam 0,20-0,25).
- ✅ **CONTA DO DIA 1 COMPLETA:** elenco → atributos → peso por posição (÷ soma) →
  ataque/defesa do time (+ goleiro por mistura 30/70) → `λ = 1,35 × 2^((A−D)/30)` → ×
  fator_mando → `p = 1 − e^(−λ/90)` por minuto → Poisson por minuto → placar.

- ⚠️ **CORREÇÃO (sessão 3) — erro anterior desfeito.** Eu havia dito que "a formação
  morde o resultado de graça pela normalização". ERRADO. Dividindo pela SOMA dos pesos,
  se os 11 forem igualmente bons, TODA formação dá o mesmo número (a divisão cancela o
  formato). O que morde de verdade é o **encaixe jogador↔posição**: remontar a formação
  põe o MESMO jogador num slot de peso diferente (CM→DM etc.) e aí o peso muda. Com
  jogadores idênticos, o formato sozinho não muda nada.
- ✅ **Fork da normalização RESOLVIDO (sessão 3):** a agregação usa **÷ soma dos pesos**
  (média ponderada, mantém 0-100 — exigido pela fórmula do λ, que usa A−D). Fazer o
  FORMATO da formação pesar sozinho (÷ número fixo OU "Camada 4") segue PARQUEADO — não
  é da primeira fatia, e cuidado pra não ligar as duas coisas juntas (dobra a conta).
- ✅ Goleiro: resolvido por mistura fixa (Jeito 2, 30/70) — ver bloco "GOLEIRO na
  defesa" acima. Não entra como "1 de 11".

### ALEATORIEDADE — distribuição, não destino ✅ [FECHADA sessão 4]

Reenquadramento do Brian (adotado, e melhor que o enquadramento anterior "seed → placar
fixo"): a seed **não define** o resultado do confronto. A força dos times define a
**distribuição** de resultados possíveis; o sorteio tira UMA realização dela.

```
força + mando + contexto + eventos → λ (probabilidades) → aleatoriedade → resultado realizado
```

- **Distribuição, não destino.** Mesmas equipes, mesmas escalações e condições parecidas
  NÃO precisam dar o mesmo placar. Favorito (λ maior) tem a distribuição empurrada a seu
  favor, mas a cauda deixa a zebra acontecer — 3×0 hoje, 0×1 no próximo. É feature, não
  bug: é a variância própria do futebol.
- **A variedade vem do λ+Poisson, NÃO da estrutura da aleatoriedade.** Qualquer gerador
  uniforme e sem viés entrega a mesma variedade; quem decide quantas zebras/empates/
  goleadas saem é a FORMA da distribuição (os λ + o processo de Poisson por minuto). O
  gerador é só um dado honesto. Consequência: o fio "qual gerador" é pequeno; a carne
  (realismo da variedade) é CALIBRAÇÃO, e vira o fio da validação por simulação.
- **Seed NOVA por execução.** Cada partida rodada recebe uma seed nova → uma realização
  nova. NÃO existe "elenco A + elenco B + seed 42 = 2×1 pra sempre" como característica de
  jogo. A seed existe só tecnicamente.
- **`IRandomSource` centraliza todo sorteio.** Abstração própria; nada de `new Random()`
  espalhado pelo motor. Todo acaso passa por ela (deixa trocar o gerador depois sem mexer
  no motor — barato adiar).
- **Seed gravada só pra DIAGNÓSTICO.** Registrar `MatchId + EngineVersion + Seed` pra
  reproduzir uma execução esquisita e investigar. O `EngineVersion` é o que torna seguro
  usar o `Random` de fábrica: a reprodução só é prometida DENTRO da mesma versão do motor
  (o algoritmo do `System.Random` pode mudar entre versões do .NET — fonte: docs da MS).
  Não é "a seed do confronto".
- **Rede de segurança = testes ESTATÍSTICOS, não placar congelado.** NADA de teste de foto
  ("esse elenco + essa seed = esse placar exato"). Os testes afirmam propriedades da
  DISTRIBUIÇÃO: taxa de empate, frequência de zebra, vitórias de favorito, goleadas,
  espalho de placares, média de gols. (O teste de foto foi rejeitado de propósito —
  brigaria com "distribuição, não destino".)
- **Substituição/decisão muda a PROBABILIDADE, não a sorte (sacada do Brian).** O
  `IRandomSource` só responde "qual número saiu [0,1)"; ele não sabe de futebol, λ ou
  jogador. Ao trocar um atacante 65 por um 80, NÃO se troca o gerador — muda a força
  efetiva → o λ → o `p` contra o qual o MESMO número é comparado. O mesmo sorteio (ex:
  0,014) vira "não" com p=1,2% e "gol" com p=1,6%. A decisão do treinador muda o
  SIGNIFICADO do número, não o número. É o comportamento de manager desejado.
- **λ(t) — λ muda pela ENTRADA EFETIVA, não pelo evento/pausa ✅.** λ não é constante 90
  min, mas o recálculo dispara SÓ quando muda uma entrada que compõe formalmente o cálculo
  (Ataque efetivo, Defesa efetiva, ou outro parâmetro que venha a entrar na fórmula). O
  evento, a pausa ou qualquer ocorrência intermediária NÃO mudam λ por si — só se mudarem
  uma dessas entradas. A `CalculateLambda(A, D)` é a MESMA fórmula; só é CHAMADA DE NOVO
  quando a entrada efetiva muda. No dia 1, a única coisa que mexe em Ataque/Defesa é QUEM
  ESTÁ EM CAMPO → recalcula na troca (muda o 11) e quando a lesão tira/reduz um jogador.
  Pausar "só pra olhar" ou um evento informativo (lance de perigo, tiro de meta) NÃO
  recalcula. Mando é fixo por partida. λ mudar por EXPULSÃO, PLACAR (recuar vencendo) ou
  CARTÃO é `[G2]`+ — a arquitetura aceita, mas o dia 1 não gera esses gatilhos nem tem
  esses parâmetros na fórmula. (Base p/ λ dinâmico existe na literatura — ex.: expulsão
  derruba a intensidade de gols do time; pesquisado.)
- **Cartão NÃO deve mexer direto no λ (refinamento `[G2]`+).** Nada de "amarelo = −3
  defesa" (grosseiro). O caminho é: amarelo → comportamento defensivo muda (menos
  entrada/falta) → isso afeta a defesa efetiva → aí sim o λ do adversário muda. O número
  exato calibra quando o sistema chegar; a arquitetura já permite sem tocar no núcleo.
- **Persistência/lifecycle — ATUALIZADO NA FASE A ✅.** Continua definitivo que o
  **estado vivo** da simulação fica em memória e não existe checkpoint/resume: minuto,
  placar parcial, RNG corrente e estado tático vivo não são reconstruídos após crash.
  Porém a execução agora **nasce persistida** como `Match InProgress`, com `MatchId`,
  fatos históricos iniciais, relacionados, Seed e EngineVersion internos. Se o Game
  cair, a execução viva se perde; na subida, `InProgress` órfã vira `NotCompleted`. Uma
  nova tentativa recebe novo `MatchId` e nova Seed. F5/aba fechada/queda de SignalR não
  matam a execução porque a partida pertence ao servidor.

> **Correção registrada (sessão 4):** eu (Claude) havia trazido "moinho próprio de
> algoritmo congelado" e "streams separados" como fundação. Sob o reenquadramento do
> Brian, os dois só serviam ao mundo do placar congelado, que foi rejeitado — viram luxo
> OPCIONAL, fora do caminho crítico. Um gerador uniforme comum atrás do `IRandomSource`
> basta.

> **Correção registrada 2 (sessão 4):** (a) eu quase apontei que "pausa curta + fechou =
> descarta" brigava com o "assíncrono / xadrez por carta" do briefing — mas ao reler as
> decisões, "xadrez por carta" já tinha sido MORTO (async = conversa por mensagem, pode
> ser rápido, com prazo curto; ver "TENSÃO DO RELÓGIO" na 4B). Sem briga; flag retirado.
> (b) Eu havia falado em "persistir o estado alto da partida" como camada durável FUTURA
> (retomar após restart). O Brian DESCARTOU essa direção por inteiro: não há recuperação
> de partida em andamento, nem agora nem planejada — partida interrompida reinicia do zero.
> Removi a ideia de "saga durável de match-state"; ver bloco Persistência acima.

### CAMADAS FUTURAS DA AGREGAÇÃO — parqueadas (sessão 3) 🅿️
Vindas da proposta de melhoria do Brian. Registradas pra não perder a direção; NÃO
entram no dia 1 (bandeira plantada, construção no degrau certo).

- 🅿️ **Função tática (papel) — `[G2/G3]`.** Separar "onde joga" (posição) de "como
  joga" (função: lateral defensivo × de apoio × ofensivo, volante construtor,
  box-to-box, falso nove…). Desenho já acordado: a função é um MULTIPLICADOR em cima
  dos pesos da posição (ex.: RB ofensivo = ataque ×1,20 / defesa ×0,90), NÃO posições
  artificiais tipo RB_DEF/RB_ATT. É o papel/duty do FM. Entra depois do dia 1.
- 🅿️ **Contexto da formação — `[H]` (horizonte, o mais avançado).** Dividir a
  responsabilidade dentro da mesma linha: 1 atacante carrega mais que cada um de 2;
  três zagueiros × dois; alas numa defesa de três. É o efeito PRÓPRIO dela — o efeito
  "formação muda a força" NÃO é isto (ver fork da normalização acima). É a camada mais
  cara e mais fácil de desbalancear. Só depois do básico calibrado. **Cuidado ao
  ligar: não dobrar a conta** com o fork da normalização.
- **Aptidão em % contínua — refino `[H]`.** Começa em três baldes (abaixo); vira
  porcentagem por posição só se houver necessidade.
- **Transparência na tela — requisito de UI.** Mostrar a CAUSA da força (ex.: "João —
  CM · base 74 · aptidão 100% · peso ofensivo alto · função box-to-box → contribuição
  76"), sem despejar a matemática toda. Vale mais quanto mais camadas existirem.

### FORMAÇÃO no dia 1 — SEM modificador próprio ✅ [FECHADO sessão 3]
Nenhum modificador de formação no dia 1. A força coletiva já muda bastante só porque
remontar a formação joga o mesmo jogador num slot de peso diferente (RB→RWB, DM→CM, um
2º ST). Isso basta pra começar. Fazer o FORMATO pesar sozinho fica pro fork da
normalização / Camada 4 — depois de ver o modelo básico rodando, pra não duplicar o
efeito.

- ✅ **`mapa-jogador` atualizado nesta sessão:** os 6 de linha + 6 de goleiro
  marcados como paleta `[D1]`, com o split e a composição fino-a-gordo, na seção
  "O QUE O DIA 1 REALMENTE USA". O traço "cobrador" anotado como `[G2]`.

### #1 — Cartão/suspensão andam junto com lesão ❌ SUPERADA PARA O RECORTE INICIAL

> Lesão, cartão e pausas obrigatórias continuam bandeiras futuras, mas **não** fazem
> parte do núcleo inicial. A primeira pausa vigente é manual; a primeira intervenção
> esportiva real é substituição humana em marco posterior.

- Motivo real: os três disparam a MESMA mecânica (a pausa, 4C). Lesão grave e
  vermelho ambos pausam. São gêmeos funcionais → modelar na mesma Família 5
  (estado do jogador).
- Dia 1: o CAMPO "estado" (apto/lesionado/suspenso) se desenha junto agora `[D1]`;
  GERAR o evento na partida é `[G2]` (o Poisson não simula lances, não há de onde
  nascer cartão). Idealizar junto, construir separado.

### #2 — Atributo vs "habilidade"/apelido (DECISÃO ESTRUTURAL) ✅
- **O motor SEMPRE lê atributos (números). NUNCA lê rótulos.**
- "Firulento", "motorzinho", "carregador de piano" = **rótulos DERIVADOS** dos
  atributos, calculados na hora de mostrar na tela. Bonitos pro jogador, invisíveis
  pro motor.
- **Por que não deixar o motor ler "firulento":** duplicaria a informação (estaria
  nos atributos E no rótulo); um dia discordam (drible 40 mas marcado "firulento")
  e vira bug — o tipo de meia-boca do ERP360.
- **Exceção — traços (traits/PPMs):** comportamento que os atributos não capturam
  (ex: "cavadinha no pênalti", "Flair" do FM) = campo PRÓPRIO que o motor lê. Coexiste
  com os rótulos derivados, mas é natureza diferente (número definido vs derivado).
  Fica em `[H]`, horizonte.

### #4 — Dados reais / API (ex: copiar Neymar) — NÃO no começo ✅
- Existe (football-data.org = melhor gratuita, 12 ligas, 10 req/min; StatsBomb/
  Understat/FBref = datasets pra aprendizado). MAS:
- **Não é boa ideia no começo.** Nenhuma coisa que o Brian quer treinar (mensageria,
  serviços, motor, pausar-e-reagir) precisa de dados reais. "Fulano" com atributos
  inventados exercita o motor igual a um Neymar real.
- **Risco extra (casa com o ERP360):** API externa = dependência que ele não
  controla (rate limit, cair, mudar formato). No dia 1, mínimo de coisas que quebram
  por fora.
## HISTÓRICO — Pendências conhecidas naquele momento ❌ ESTADO SUBSTITUÍDO

**Já fechado:** 4D (quem simula) · contagem de serviços (2) · banco (dois) ·
4A parcial (Jeito B, atributo-vs-apelido, cartão/lesão, dados reais).

**Ainda cru:**
- ⬜ **[EM CURSO] Seção 4B — o motor da partida.** Já fechado dentro dele:
  - **Motor "quadro a quadro" (Jeito 1), pausável de verdade** — NÃO calcula a
    partida toda de uma vez. Motivo: o pausar-e-reagir (item 4) tem "efeito
    borboleta" — decisão no minuto 40 muda o minuto 41. Fita pré-calculada (Jeito 2,
    o FM antigo) não serve porque a decisão chegaria tarde. Lição registrada: a
    "melhor" arquitetura depende do que o JOGO precisa; Jeito 2 é ótimo pra FM
    single-player, péssimo pro reativo do Brian.
  - **Passo = de minuto em minuto, visível ao usuário (estilo Brasfoot).** O usuário
    vê "minuto 1, 2, 3..." correndo.
  - **DOIS TIPOS DE EVENTO (sacada do Brian, molda o motor todo):**
    - **Eventos que PEDEM DECISÃO** (lesão, vermelho, pênalti/quem bate) → PARAM o
      jogo = pausar-e-reagir.
    - **Eventos que só INFORMAM** (lance de perigo, finalização, bola na trave, tiro
      de meta) → NÃO param; são o "termômetro" da tática. Sem assistir em 3D, esses
      eventos são os OLHOS do jogador pra ler se o plano tático funciona. Não são
      enfeite — são informação pra decisão.
  - **TENSÃO DO RELÓGIO — RESOLVIDA (com correção de um mal-entendido meu):**
    - Erro anterior: eu tratei "assíncrono" como "lento / xadrez por carta / pode
      levar horas". ERRADO. **Assíncrono = conversa por MENSAGEM, os lados não presos
      ao mesmo instante travado. Pode ser RÁPIDO.** A essência é a mensagem, não a
      lentidão. A analogia "xadrez por carta" foi infeliz.
    - **O que o Brian quer (cláusula IMUTÁVEL = ter mensageria):** partida corre
      minuto a minuto mostrando os lances-termômetro; quando cai evento-chave, PAUSA
      e NOTIFICA o jogador com **prazo curto** ("decide em X min"); tudo por mensagem;
      prazo estoura → regra decide (robô/última ordem — a fechar). Ninguém espera a
      noite toda, mas ninguém fica preso ao mesmo segundo que o outro.
    - **Confusão desfeita:** "assíncrono" (como os serviços conversam) e "correr
      minuto a minuto mostrando lances" (como a partida é exibida) são
      INDEPENDENTES. O motor minuto-a-minuto (estilo Brasfoot) já estava fechado; o
      "pular de evento em evento" foi DESCARTADO. Async não traz o "pular" de volta.
    - **Decisão: SAÍDA A (assíncrona por mensagem), com partida dinâmica e prazo
      curto.** Custo vs Saída B (ao vivo, os dois juntos obrigatoriamente): a A é
      mais BARATA **e** protege a mensageria (o Brian já ia pagar esse custo de
      qualquer jeito). A B é mais CARA (sincronizar os dois no mesmo segundo, "e se
      um cair") **e** ENFRAQUECE a mensageria (no ao vivo o broker vira menos
      necessário). A ganha nos dois lados pro objetivo do Brian.
    - Modo "ao vivo puro" (Saída B) continua como camada futura opcional (item 12),
      não é o alvo.
  - **DEGRAU INICIAL DO MOTOR: Poisson rodado POR MINUTO (não moeda viciada de uma
    vez).** ✅ [FECHADO na sessão 2]
    - **A "briga" era falsa.** Parecia que "minuto a minuto pausável" brigava com
      "começar no Poisson", porque Poisson normal cospe o placar FINAL de uma vez
      (tipo "sai 2 a 1"), não um jogo se desenrolando. Não brigam: dá pra espalhar
      os "gols esperados no jogo todo" pelos 90 minutos e, a cada minuto, jogar uma
      **moedinha viciada** ("saiu gol neste minuto? sim/não", com chance baixa ~
      gols-esperados ÷ 90). No fim, a soma dos gols CONTINUA obedecendo o Poisson —
      é o MESMO modelo, só olhado por minuto em vez de por jogo. (Confirmado em
      modelos reais de futebol — pesquisado, não de memória.)
    - **Por que casa com o pausar-e-reagir:** lesão no minuto 40 → é só baixar a
      chance da moedinha nos minutos 41–90. O efeito borboleta sai DE GRAÇA, porque
      o placar nunca esteve pré-decidido; é recalculado minuto a minuto.
    - **Escolha do Brian — Opção B: PULAR a moeda viciada de uma vez.** O motor já
      NASCE minuto a minuto; não existe degrau descartável de "sorteio único do
      placar". Custo aceito e conhecido: o 1º marco de motor estreia DUAS coisas
      juntas (o laço minuto-a-minuto + a matemática do Poisson) — mas as duas são
      coladas (o laço É como o Poisson roda por minuto), então é um par honesto, não
      duas features soltas.
    - **O que a Opção B NÃO significa:** não é amontoar tudo num marco só. O 1º
      motor roda o laço e devolve placar + lista de eventos; SURGIR os eventos na
      linha do tempo e PAUSAR continuam sendo camadas DEPOIS, por cima do mesmo laço.
    - **Efeito no roadmap (4I):** o antigo marco "Nível 0 = moeda viciada / partida
      inteira de uma vez" SAI. Marco 1 (só encanamento, sem motor) fica; o 1º marco
      de motor passa a ser o Poisson-por-minuto. Renumerar quando a 4I for fechada.
    - ⬜ Refinamento de horizonte `[H]`: gols reais não caem uniformes (saem mais no
      fim de cada tempo) → dá pra pesar a moedinha por faixa de minuto. Não muda
      nada hoje; fica anotado.
  - ⬜ Ainda no motor: ATÉ ONDE o motor mira (o teto — degrau 1/2/3) é o PRÓXIMO fio
    junto de: como os atributos viram o λ (metrificação do dia 1, item 2 da
    retomada); como a aleatoriedade entra sem quebrar o determinismo por semente
    (item 10) — a moedinha-por-minuto é onde a semente pluga; estudar o open-football
    por dentro.

## Seção 4C (parcial) — Pausar-e-reagir ❌ SUPERADA PELA FASE A

> A seção abaixo é histórico da mecânica baseada em lesão/vermelho e prazo remoto.
> Foi substituída por: **3 pausas manuais**, até **60 segundos reais**, estados vivos
> `Running/Paused/Halftime`, e contadores separados de pausa, parada de substituição
> e substituições. Pausas obrigatórias voltam apenas com eventos ricos futuros.


### Dois tipos de pausa (nomear pra nunca confundir):
1. **Pausa que o SISTEMA obriga** — lesão grave, vermelho. O jogo TEM que parar.
   **É de graça** (não gasta do saldo do jogador — ele não escolheu o azar).
2. **Pausa que o JOGADOR escolhe** — saldo limitado, com cronômetro. Ele gasta
   quando QUER mexer no time.

### Regra do prazo estourado (quando notifica e o jogador não responde):
- O jogo **segue com o prejuízo natural** (ex: lesão sem resposta → joga com 10).
- Na **próxima parada natural do futebol** (escanteio, lateral, tiro de meta, falta)
  o sistema para de novo e dá **outra chance** — mas essa **gasta uma pausa** do
  jogador (a 1ª, do sistema, foi de graça; a 2ª é paga).
- **Sem saldo de pausa?** Fica com o prejuízo (ex: com 10) até a próxima parada
  natural OU até o fim do tempo (intervalo/fim de jogo). Só volta a poder mexer se
  tiver saldo.
- Amarração elegante: a "segunda chance" cai nas paradas naturais do futebol (que
  já param na vida real) → a pausa do sistema não parece artificial. E custar saldo
  impede muleta infinita — o sistema se auto-limita.

### Regras de substituição (regras reais da bola) ✅
- **Até 5 trocas** por equipe por partida.
- **Máximo 3 paradas com a bola rolando** para fazer essas trocas.
- **Intervalo:** trocas no intervalo NÃO gastam e NÃO contam como nenhuma das 3
  paradas com bola rolando.
- **Cada parada dura 1 minuto** (de jogo).

### ⚠️ A CONFIRMAR no próximo chat (relação pausa × parada de substituição):
- Ainda NÃO está fechado se "as 3 paradas de substituição" são a MESMA coisa que "o
  saldo de pausas do jogador" (4C) ou se são dois saldos separados. Ex: a pausa que
  o jogador gasta pra mexer na tática é a mesma parada que ele usa pra substituir, ou
  são contas diferentes? Provável que se relacionem, mas precisa ser desenhado com
  cuidado — é o primeiro fio a puxar quando a 4C for retomada.
- Também falta: **duração do cronômetro de cada pausa** (o "prazo curto" pro jogador
  decidir) — decisão numérica pequena, adiada pro próximo chat.

### Pausa é INDIVIDUAL (Caminho 1) ✅
- Quando um jogador pausa, **só ele mexe** (no próprio time). O adversário espera,
  não pode aproveitar.
- Descartado: pausa compartilhada ("salvo pelo adversário" / carona na pausa alheia).
- **Por que Caminho 1 é bom:** mata de uma vez os casos chatos que a pausa
  compartilhada criava — "quem espera quem?", "quando o jogo volta?", "o carona que
  mexe de graça na pausa que o outro pagou?". Com pausa individual: volta quando o
  cronômetro DELE acaba; saldo limpo (cada um gasta só o que usa). Nenhuma dessas
  perguntas precisa de resposta porque só existiam na pausa compartilhada.
- (NOTA: ninguém nunca mexe no time do outro — pausa individual reforça isso.)
- ⬜ O clube e as peças além do jogador — comissão técnica, finanças, situações
  reais de clube (Seção 4A). O Brian QUER isso rico. Fazer DEPOIS do motor, guiado
  pela mecânica que usa cada peça (senão vira "cômodo sem porta").
- ⬜ Pausar-e-reagir — o que dispara pausa, o que fazer, timeout (Seção 4C)
- ⬜ Contratos de mensagem do primeiro marco (Seção 4E)
- ⬜ Outbox / idempotência / DLQ — quando entram (Seção 4E)
- ⬜ A tela React — polling vs WebSocket (Seção 4F)
- ⬜ Autenticação — quando e como (Seção 4G)
- ⬜ Os marcos em ordem (Seção 4I)
- ⬜ Ciência vs heurística por mecânica (Seção 4J)

---

## Seção 4E — Fluxo Game ↔ Opponent pelo RabbitMQ ❌ SUPERADO PELA FASE A

> **Toda a topologia abaixo é histórica:** `SolicitarEscalacao`, `EnviarEscalacao`,
> saga do cumprimento e reação do visitante pelo broker não existem mais no núcleo.
> Os conceitos gerais estudados (Send/Publish, retry, Outbox, idempotência, `_error`)
> continuam úteis, mas a integração vigente nasce em `PartidaEncerrada` → Manager
> Inbox.


O coração dos microsserviços: o que atravessa a fronteira entre o **serviço do jogo**
(mandante, segura o relógio) e o **serviço do adversário** (robô no dia 1). Fio puxado
por sub-fios; um por vez.

### Sub-fio (a) — Quais mensagens cruzam a fronteira no esqueleto que anda ✅ FECHADO (sessão 6)

**Escopo do esqueleto que anda (marco 1) = APERTO DE MÃO PURO.** ✅
- O marco 1 é só a **troca de escalação**: prova que a mensagem atravessa a fronteira de
  verdade (com RabbitMQ no meio), o mínimo de peças pra debugar. **NÃO roda a partida** no
  marco 1 — rodar os 90 min + cuspir placar é marco SEGUINTE (o antigo "Nível 0" do briefing,
  hoje = Poisson-por-minuto).
- **Por quê (a régua do esqueleto que anda):** rodar o motor junto empilharia *motor novo*
  **+** *mensageria nova* no mesmo marco = dois problemas novos pra debugar de uma vez. O
  esqueleto que anda existe justamente pra evitar isso. O motor já está fechado no papel, mas
  isso não obriga a metê-lo no marco 1.
- Descartada a versão "gorda" (marco 1 já roda uma partida e reporta o resultado de volta),
  que traria uma 3ª mensagem (jogo→adversário "a partida acabou, foi assim" = placar+eventos).
  Essa 3ª mensagem **volta à mesa** quando o marco de rodar-a-partida chegar — não sumiu, só
  não é do marco 1.

**As mensagens que cruzam a fronteira no marco 1 (2 mensagens):** ✅
| # | Sentido | Conteúdo |
|---|---|---|
| 1 | **serviço do jogo → serviço do adversário** | "vai ter partida, me manda tua escalação" (o mandante começa a conversa — ele segura o relógio) |
| 2 | **serviço do adversário → serviço do jogo** | "aqui está minha escalação" |

**Cravado junto (pra não confundir depois):**
- O 3º passo do briefing 4I ("aparece na sua tela") **NÃO é mensagem de broker** — a tela
  fala **HTTP** com o serviço do jogo, nunca com o RabbitMQ. Das 3 setas que o briefing
  desenha, só 2 cruzam a fronteira **entre serviços**; a 3ª é a borda HTTP. O sub-fio (a) só
  conta as 2 de broker.
- A 1ª mensagem hoje está "gorda" de propósito (anuncia a partida **e** pede a escalação num
  aviso só). Se vale **partir** isso em dois (ex.: "partida criada" ⟶ depois "preciso da
  escalação") é granularidade/estilo — fica pro sub-fio (b)/(c), não se decide aqui.
- O **estilo** de cada mensagem (um ANUNCIA e o outro reage — publish/subscribe — vs. um
  PEDE e o outro responde — request/response) está **parqueado pro sub-fio (c)**. Aqui só se
  fechou *o quê* cruza e *pra que lado*, não *em que estilo*.

### Sub-fio (c) — Estilo da conversa (evento × comando) ✅ FECHADO (sessão 6)

**Dois eixos que NÃO são a mesma coisa** (a fonte de toda a confusão — separar mata a dúvida):
- **O que a mensagem SIGNIFICA:** *comando* (uma ordem: "faça isto") ou *evento* (um fato: "isto aconteceu").
- **COMO é entregue:** **`Send`** (vai pra caixa de UM destino que você conhece) ou **`Publish`** (é pregada no "mural" pra QUEM QUISER ler; o emissor não sabe quem consome).
- Costumam andar juntos (ordem→`Send`, fato→`Publish`), mas dá pra ter um *fato* entregue por `Send` quando só há um interessado conhecido.
- Fonte (MassTransit, verificado por busca sessão 6): `Send` entrega a um endereço de destino específico; `Publish` transmite a todos os assinantes do tipo; comando = verbo-substantivo estilo "faça isto", evento = substantivo-verbo no passado.

**Decisão: a conversa é COMANDO, entregue por `Send`.** ✅
- As 2 mensagens do aperto de mão são **ordens endereçadas** a um destino **único e conhecido** (seu serviço sabe exatamente de quem quer a escalação; o adversário sabe pra quem devolver). Ninguém "grita pro mural".
- Usar `Publish` aqui seria **cosplay de arquitetura** (o veneno que o briefing manda evitar): o mural só ganha o pão quando você NÃO sabe quem escuta, ou quando são VÁRIOS ouvintes. Com destinatário único, é enfeite — e pior, formato errado (mural pra um ouvinte só) que daria retrabalho desfazer.

**Decisão: Opção 3 — dois comandos de mão única coordenados pela SAGA.** ✅
- Seu serviço manda o comando "me manda tua escalação" e **não fica preso**; a escalação volta depois como um 2º comando ("aqui está"), e a **saga** (o gerente do processo da partida, 4C) junta as pontas.
- Descartada a **Opção 2 (pedido-resposta / `Request`)**: ela sabe "esperei → chegou / esperei → estourou o tempo = erro", mas NÃO sabe "estourou o tempo → a partida segue com 10, com 2ª chance na próxima parada". Isso é máquina de estados (saga), não request.
- **Correção honesta registrada:** a 1ª justificativa da Opção 3 ("humano pode levar 3h") estava ERRADA — o Brian cravou que, do 1º contato em diante, **o prazo é definido e curto** (5 min já é muito; 3h não existe). A Opção 3 se sustenta por outro motivo, mais firme: a coleta de escalação é o **1º passo da MESMA saga** que rege a partida inteira; o timeout do adversário só tem casa dentro da saga. Prazo curto joga A FAVOR: é um **cronômetro**, e segurar cronômetro é o que a saga faz por natureza (a duração é só o número que se põe nele).
- Descartada a **Opção 1 (evento/`Publish`)** pelos motivos do parágrafo do cosplay acima.

**Decisão: SEM `Publish` no marco 1 — mas o PUBLISH TEM CASA CRAVADA (fato → plateia).** ✅
- Contexto (importante): **o objetivo declarado do projeto é aprender `Publish`.** Isso reordena a prioridade — garantir que exista publish REAL vale mais que qualquer marco. Mas a saída NÃO é fingir publish no aperto de mão (isso ensinaria publish na forma degenerada, um-pra-um).
- **A mira certa:** publish nunca foi sobre os DOIS jogadores (esses são os dois *participantes* de uma negociação → sempre `Send`). Publish é sobre os **observadores de um FATO** — quem reage quando algo acontece, sem o emissor saber quem são. O nº de jogadores ser 2 é irrelevante: publish cresce com o nº de **reações a um fato**, não com o nº de lados da conversa.
- **Correção honesta registrada:** no turno anterior eu disse "publish talvez nunca entre" — aquilo valia sob a suposição de ZERO observadores. O objetivo do Brian desfaz a suposição: planta-se um observador real e o publish nasce real.
- **Primeiro observador (FECHADO): serviço de CLASSIFICAÇÃO/LIGA**, reagindo ao evento **`PartidaEncerrada`** → atualiza a tabela. (Candidatos alternativos, todos trabalho real: estatística, notificação, histórico — ficam como plateia futura.)
- **Por que é barato (≠ juiz neutro):** o juiz sentava no CAMINHO CRÍTICO (a partida esperava por ele → travava a fronteira). O observador só **ESCUTA**: a partida anuncia `PartidaEncerrada` e segue; se a classificação estiver fora do ar, a partida NÃO trava (natureza do publish: atira e esquece). Entra na lateral, não no meio de campo — não mata o esqueleto que anda.
- **Publish correto mesmo com 1 assinante só:** o serviço do jogo NÃO deve saber que a classificação existe — ele anuncia o fato e segue; quem quiser que escute. A intenção do desenho ("anuncio o fato, não rastreio quem consome") é o que faz ser publish honesto, diferente do mural vazio do aperto de mão.
- **Timing:** estreia em marco próprio (NÃO o marco 1, que fica `Send` puro). Como aprender publish é objetivo, é legítimo PRIORIZAR esse marco pra vir cedo (logo após o motor rodar uma partida), em vez de "quando der".

> **Régua aplicada (barato adiar × caro adiar):** a INFRA de publicar (broker, MassTransit, fiação) já está de pé no dia 1 de graça — `Send` e `Publish` usam a mesma infra. O que falta não é módulo, é um EVENTO real com PLATEIA. Adicionar publish quando o 1º observador existir custa quase nada → adia (igual ao juiz neutro). Forçar publish no aperto de mão custaria desfazer depois → não fazer.

### Sub-fio (b) — Formato dos pacotes (o projeto `Contracts`) ✅ FECHADO (sessão 7)

O QUE cruza a fronteira, campo a campo. Fechado em 8 martelos, todos filtrados por "caro de
mudar depois × barato adiar" e pelas 3 fronteiras da 3A(c) (database-per-service · motor num
lugar só · contrato ≠ entidade de domínio).

**1. Contrato NASCE COMPLETO (não mínimo).** ✅ O marco 1 é aperto de mão puro e NÃO simula —
mas a mensagem de escalação já carrega tudo que o motor vai ler no marco seguinte. Motivo:
reformar uma mensagem que os dois serviços já consomem é caro (a dor "caro de mudar depois");
e o que o motor precisa já está fechado (Seção 3 + paleta gorda) → é alvo conhecido, não chute.
O "esqueleto que anda" continua fino no que importa (o marco 1 segue sem rodar partida); ele é
sobre não erguer camadas isoladas, NÃO sobre espremer campos de uma mensagem só.
Descartada a opção "mínimo agora, reformo quando o motor chegar" (valor pedagógico real de
sentir o retrofit, mas perde pro custo de reabrir contrato).

**2. FORMA da escalação = os atributos que o motor lê.** ✅ Descartadas:
- **Só IDs (referências):** id sozinho não diz nada ao serviço do jogo — ele NÃO lê o banco do
  adversário (database-per-service). Só funcionaria com catálogo compartilhado (banco comum ou
  3º serviço) → fura database-per-service e mete peça nova. Gambiarra.
- **Força já calculada (Ataque/Defesa prontos):** calcular força É trabalho do motor, e o motor
  mora num lugar só. Obrigaria o adversário a ter uma CÓPIA da agregação → duas fórmulas prontas
  pra desencontrar; ao recalibrar (BaseGoals/DifferenceScale/pesos) os lados divergem calados.
- **Escolhida:** o adversário manda os 11 com os atributos; o motor (no serviço do jogo) recebe
  como "Time B" e calcula as forças com o MESMO código do Time A (fonte de verdade única da
  agregação). A tabela de pesos por posição fica no motor, NÃO na mensagem — nem cruza.

**3. IDENTIDADE do jogador no pacote = `PlayerId` opaco, escopo da interação.** ✅ (redação do
Brian, adotada literal) Cada jogador enviado carrega um `PlayerId` opaco **definido pelo serviço
dono do elenco**. O receptor usa pra identificar o jogador **naquela interação**, mas NÃO assume
identidade global compartilhada entre serviços. Descartado o esquema de identidade global agora
(GUID universal) — cerimônia paga hoje por um problema `[H]` que pode nunca vir; com clube dono
do próprio elenco, referência cruzando serviço é SEMPRE escopo-de-partida, nunca entidade de 1ª
classe. Se o horizonte cobrar identidade global, a feature daquela época paga — e o contrato já
estará versionado, então nem conta como retrofit do dia 1.

**4. NENHUMA entidade de domínio nem Value Object cruza o broker.** ✅ Os atributos viajam como
**números simples efetivos** (um valor 0–100 por gordo), não como o VO `Rating` nem o `Jogador`
do domínio jogado cru na fila (mandar o `Jogador` inteiro grudaria os serviços nos internos um
do outro — 3A(c)). O contrato da escalação é uma FORMA desenhada pra cruzar.
- *Reparo do Brian registrado:* a palavra é atributos **efetivos**, não "atributos". No dia 1
  efetivo = cru (nada modula: aptidão desligada, sem cansaço, sem forma). Só passa a diferir de
  cru num degrau futuro, e aí nasce a pergunta "quem aplica o modificador antes de pôr na fila"
  — **bandeira plantada, não é fio de hoje**. O campo do contrato é um número 0–100 de qualquer jeito.

**5. Conteúdo da mensagem 2 (`EnviarEscalacao`), fechado:** ✅
- por jogador → `PlayerId` (opaco) + nome + posição + os 6 gordos (números efetivos 0–100);
- no time → a formação.

**6. CLUBE do adversário NÃO entra na mensagem — decisão ESTRUTURAL e PERMANENTE (≠ parqueado).**
✅ (redação do Brian, adotada) No modelo atual cada manager — humano ou robô — comanda
**exatamente um clube**; logo o clube adversário é **derivável do destinatário** da solicitação e
não precisa integrar o contrato. Participação em grupos/SAFs multiclubes NÃO altera isso.
- *Reparo do Brian (importante):* eu tinha etiquetado o clube como "parqueado, vira necessário se
  o adversário tiver vários clubes" — ERRADO. Ele separou **comando ≠ propriedade**: a mensagem
  enxerga *manager → clube*, que é **1:1 por invariante**. Propriedade pode ser 1:muitos (grupo,
  SAF), mas é o grafo de *dono*, não o de *comando* — a partida fala com quem *comanda*. Clube
  sai do contrato pra sempre, não é bandeira aditiva.
- *Flag de horizonte (OUTRO eixo, não mexe nisto):* se um dia um processo hospedar VÁRIOS
  managers (serviço-robô multi-inquilino), o `Send` precisa endereçar o *manager* certo → seria
  um identificador de **manager** pra ROTEAMENTO, nunca o clube no corpo. Roteamento ≠ conteúdo
  de contrato. Anotado.

**7. Correlation ID (crachá de rastreio) viaja no HEADER, não como campo do contrato.** ✅
Descartado pôr como propriedade da classe da mensagem (enfiaria assunto transversal de infra
dentro do contrato de domínio — a fronteira que a 3A(c) separa; e cada mensagem nova teria que
lembrar de carregar o campo na mão → furo no rastro). Escolhido: anda como metadado no header;
o MassTransit tem lugar próprio e propaga de mensagem em mensagem. Nasce/recebe na borda HTTP e
segue pelo publish e pelo consumo. Casa na letra com a 3A ("propagado ponta a ponta"; a palavra
grifada é *propagado*). MatchId, esse sim, é dado de domínio → vai no CORPO (a saga usa ele pra
achar a partida certa quando a resposta volta).

**8. Conteúdo da mensagem 1 (`SolicitarEscalacao`), fechado:** ✅
- **É um pedido só — NÃO parte em dois.** No marco 1 não há intervalo entre "vai ter partida" e
  "manda a escalação" (o robô responde na hora); partir viraria ou dois `Send` colados (cerimônia
  vazia) ou um `Publish` "partida criada" — que reabriria o que o (c) fechou (sem publish no
  aperto de mão; a casa do publish é `PartidaEncerrada` em marco próprio).
- **Bônus (barato-de-adiar):** partir só ganha o pão no horizonte, quando "partida agendada" e
  "montar o XI" forem dois momentos separados no tempo. Aí o conserto é **aditivo** — acrescenta
  uma mensagem de anúncio ANTES, sem tocar na de pedido. Manter a msg 1 como **pedido puro** (sem
  fundir "anúncio" no corpo) é o que deixa esse futuro ser aditivo; se nascesse fundindo
  anúncio+pedido, partir depois seria ARRANCAR metade → aí sim quebra de contrato.
- **Prazo fica FORA.** Quem segura o cronômetro é a saga; o robô responde na hora. Mandar o prazo
  só serviria pra mostrar "você tem 5 min" numa tela de humano → UI de horizonte.
- **Assenta:** corpo = `MatchId`; header = crachá.

**Nomes das mensagens (comando = verbo-substantivo, português):** ✅
- Msg 1 (serviço do jogo → adversário, pede): **`SolicitarEscalacao`**
- Msg 2 (adversário → serviço do jogo, entrega): **`EnviarEscalacao`**

**Versão do contrato: ADIAR de propósito (Opção A).** ✅ Sem marcador de versão agora. Enquanto é
um dev subindo os dois serviços juntos, os dois mudam ao mesmo tempo — nunca há pacote "velho"
viajando. Versão só ganha utilidade quando os serviços forem atualizados **separadamente**
(sistema no ar). Registrado como decisão CONSCIENTE, não esquecimento (pro dia que alguém abrir o
código e perguntar "cadê o versionamento?"). Encaixe futuro é aditivo, receita pronta.

> **Nota de conduta (Regra 0, registrada na sessão 7):** a conversa com o Brian derrapou pro
> "dialeto dos documentos" (apelidos empilhados, quase taquigrafia de máquina) — fura a Regra 0
> (linguagem simples, todo termo explicado). Correção acordada: **os apelidos ficam DENTRO dos
> documentos** (é o idioma deles); **na conversa com o Brian, português normal e tudo explicado.**
> Documento é uma língua, conversa é outra.

### Sub-fio (d) — Pausar-e-reagir virando mensagem ✅ FECHADO (sessão 7)

COMO a mecânica de pausa-e-troca vira tráfego ENTRE serviços. Não é recuperação de
partida caída (isso é persistência, já fechada: cai = reinicia). É coordenação.

**1. Dois canais que NÃO se misturam numa pausa.** ✅
- **Gestor → serviço dele, pela tela:** HTTP. O clique "troca o 9 pelo 15" é borda do
  sistema, **não é mensagem de broker**. Fora do (d).
- **Serviço → serviço, pelo RabbitMQ:** *esse* é o assunto do (d).

**2. Opção A — desenho ASSIMÉTRICO pro mundo de hoje, com mensagens NEUTRAS.** ✅
- No dia 1 o serviço do jogo é o cérebro: roda a simulação, segura o relógio **e** faz o
  papel de mandante. Então a reação do gestor da CASA **não cruza o broker** — a tela dele
  fala HTTP com o serviço do jogo, que aplica na simulação que ele mesmo roda. Só o lado
  **visitante** mora do outro lado da fronteira → **só as reações do visitante viram
  mensagem entre serviços**. A pausa é assimétrica: um lado é conversa dentro de casa, o
  outro é carta pelo correio.
- **O pulo do gato (o que faz A ser seguro):** desenhar as mensagens **sem carimbar
  "adversário"** nelas — chamar de "reação de um gestor" (qualquer lado), não "reação do
  adversário". Assim o assimétrico é o **fato de hoje** (mandante local, visitante remoto),
  não o **contrato**. Quando o juiz neutro nascer (hoje adiado de propósito) e os dois lados
  ficarem remotos, as MESMAS mensagens servem pros dois, sem reescrever.
- Descartada a **Opção B (simétrico já)**: construir pra uma topologia que ainda não existe
  (o juiz está adiado) e fazer o mandante — que está dentro do próprio serviço do jogo — dar
  uma volta pelo broker à toa. Tecnologia onde não faz nada útil. O conserto de A, quando o
  juiz vier, é custo do marco do juiz — que já está adiado de qualquer forma (mesma régua:
  barato adiar).

**3. A mensagem de pausa (serviço do jogo → gestor remoto) LEVA A FOTO DA PARTIDA junto.** ✅
- Conteúdo: além do aviso "pausou e por quê", vai a situação (placar, minuto, quem está em
  campo, quem se machucou) — a foto que o gestor precisa pra decidir.
- Motivo: o gestor remoto é OUTRO serviço e **não lê o banco do serviço do jogo**
  (database-per-service — a mesma fronteira do sub-fio b). Se a mensagem chegasse magra
  ("só pausou"), seria obrigatório construir um caminho de CONSULTA de volta pra ele
  perguntar como está o jogo → mais um caminho pra erguer e manter.
- Descartada a **mensagem magra + consulta depois**: uma carta dizendo "me liga que eu te
  conto", quando dava pra mandar a carta já com a foto dentro.

**4. A REAÇÃO de volta reusa o `EnviarEscalacao` (Opção 1).** ✅
- Quando o gestor troca, ele manda **o time inteiro de novo, já com a troca feita**, usando
  o MESMO contrato do aperto de mão (`EnviarEscalacao`). Um contrato só. O serviço do jogo
  recebe o time novo e **recalcula a força** com o mesmo código de sempre (fonte de verdade
  única da agregação — casa com o martelo 2 do sub-fio b; e com o λ(t) já fechado, que
  recalcula quando muda quem está em campo).
- Descartada a **Opção 2 (mensagem nova enxuta "tira o X, põe o Y")**: um 2º contrato,
  trafega menos, mas é mais peça e o serviço do jogo teria que aplicar a troca sobre o time
  que já guarda. Vira útil só quando o volume de trocas importar — e aí é **aditiva**
  (acrescenta a mensagem específica sem tocar na de escalação). Mandar 11 números quando
  mudou 1 não pesa no começo.

**5. Amarração com o (c) — é a MESMA saga.** ✅ Essas mensagens são passos mais adiante da
saga que começou no aperto de mão (pedir escalação foi o passo 1; pausar-e-reagir são os
passos seguintes). O cronômetro da pausa é da saga — como o (c) já fechou, **não é decisão
nova** (levantado nesta sessão e reconhecido como redundante).

> **Régua aplicada (barato adiar × caro adiar):** o assimétrico de hoje foi desenhado pra
> NÃO virar dívida — mensagens neutras ("gestor", não "adversário") deixam o dia do juiz
> neutro ser só "religar o outro lado no mesmo cano", não reescrita. Mesma lógica do juiz e
> do publish: a porta já fica encaixada.

### Sub-fio (e) — Outbox / idempotência / DLQ ✅ FECHADO (sessão 8)

As três "joias de aprendizado". Régua central que rege o sub-fio inteiro: **cada padrão remenda uma
falha DIFERENTE, num lugar diferente do fluxo — e cada joia entra ONDE TEM TRABALHO DE VERDADE,
não onde caberia.** Forçar um padrão num lugar sem alvo é a mesma cosplay de arquitetura que já
recusamos no publish degenerado do aperto de mão. Fechado em uma decisão de base + três martelos.

**0. Base — a SAGA vive na MEMÓRIA no marco 1 (não gravada em banco).** ✅
- Coerente com a persistência já fechada (partida na memória; cai = reinicia do zero). O fluxo da
  saga estava parqueado À PARTE daquela decisão, então era genuinamente aberto — agora fechado.
- **Preocupação "deploy no meio da espera mata o convite pendente" foi DERRUBADA pelo Brian:**
  deploy é parada PLANEJADA (ele escolhe a hora, num momento sem jogo no ar). Isso removeu o motivo
  que eu tinha dado pra gravar a saga já no marco 1. Sobra o **crash** (parada não planejada), mas
  no marco 1 o adversário é o ROBÔ (responde na hora → a espera é um piscar, não os 5 min), e o que
  se perderia é um aperto de mão que **ainda não virou partida** → re-dispara e pronto (o próprio
  "cai = reinicia", um andar acima, e barato — não há estado de partida pra perder).
- **Consequência direta:** sem saga durável, NÃO nasce a costura "grava estado E publica" no aperto
  de mão → decide o destino do Outbox (martelo 1).
- **Durabilidade da saga volta com DENTES num marco bem mais tarde:** quando o adversário for
  HUMANO e a espera de 5 min for real — aí perder um convite pendente desperdiça o turno de uma
  PESSOA (não de um robô), e reabre "grava a saga?". Plantado pra lá.

**1. OUTBOX — FORA do marco 1.** ✅
- Outbox protege a costura "grava no meu banco **E** publica, e as duas têm que valer juntas" (dois
  sistemas — banco + broker — sem transação compartilhada; cai no meio → mensagem perdida ou fato
  fantasma). No marco 1 (aperto de mão puro, não roda partida, saga na memória) **não existe essa
  costura** → nada pra proteger → forçá-lo seria cosplay.
- **Casa provável = FIM DA PARTIDA:** serviço do jogo grava o resultado **e** publica
  `PartidaEncerrada` pra liga. **Ressalva honesta:** essa casa só é real SE o serviço do jogo
  guardar algo durável ali (histórico); se um dia for "só publica, não guarda", o Outbox procura
  casa noutro lugar. A confirmar quando esse marco chegar.

**2. IDEMPOTÊNCIA — só a ESTRUTURAL no marco 1; trava de EFEITO só na liga.** ✅
- Duas idempotências distintas (separar mata a dúvida):
  - **Estrutural (de fluxo):** vem da FORMA do desenho — saga + Correlation ID no cabeçalho (ambos
    já fechados no (b)/(c)). Uma cópia repetida cai numa saga que já sabe onde está → reconhece
    "esse aperto de mão eu já terminei" e ignora. *Quanto o MassTransit entrega sozinho vs. o que se
    encaixa na mão = detalhe do sub-fio (f), a verificar na fonte.*
  - **De efeito:** quando reprocessar MUDA dado ("somei o gol", "creditei o ponto"). Não há forma
    estrutural que salve → precisa de trava EXPLÍCITA (guardar "a mensagem de id X eu já apliquei",
    pular na 2ª).
- **Marco 1 quase só tem a estrutural:** coletar escalação não tem efeito que "duplicado dá errado";
  e a única mensagem com cara de efeito (a reação de troca) **já foi desarmada no sub-fio (d),
  martelo 4** — reenvia o time INTEIRO, a última mensagem sobrescreve, e como o time é idêntico o
  estado é o mesmo. Ou seja: **trava de efeito SEM ALVO no marco 1** (parte da idempotência a gente
  já desenhou sem chamar pelo nome).
- **Casa cravada da trava explícita: serviço de LIGA consumindo `PartidaEncerrada`** — se o evento
  chega 2× e ele faz "pontos do vencedor += 3", em dobro CORROMPE a tabela. Cai no mesmo marco do
  publish e (provavelmente) do Outbox — os três se encontram ali.
- **Nuance vs. Outbox:** forçar Outbox no marco 1 é cosplay PURO (não há nada pra proteger); a trava
  de efeito só não tem ALVO natural no aperto de mão. Legítimo PRIORIZAR o marco da liga cedo (pra
  aprender a joia com trabalho real), mas isso é ordem de roadmap, não decisão de agora.

**3. DLQ — cheiro OPOSTO: já vem de graça; RETRY BÁSICO ligado no marco 1.** ✅
- Ao contrário das duas de cima, a falha que a DLQ protege JÁ existe no marco 1 (mensagem que chega
  e não processa — malformada, bug no handler) E o estacionamento vem de fábrica. **Verificado na
  doc (busca, sessão 8):** por padrão o MassTransit move a mensagem que estourou uma exceção pra
  fila `_error` (detalhes do erro nos cabeçalhos + evento de falha publicado), sem configurar nada.
- Duas camadas distintas (formatos de falha diferentes — coexistem, não é uma OU outra):
  - **Estacionar (a DLQ em si):** falha que NÃO se cura sozinha (malformada, bug). Guarda de lado em
    vez de jogar fora (perde o rastro) ou reentregar pra sempre — a "mensagem venenosa" que trava a
    fila (loop infinito confirmado num issue do repo deles quando a fila de mortas some). **Vem de
    graça.**
  - **Retentar (retry):** falha PASSAGEIRA (rede piscou, banco travou meio segundo). NÃO vem por
    padrão — sem configurar, a mensagem falha na 1ª e já vai pro estacionamento.
- **Decisão marco 1:** **estacionamento de graça** (só SABER que existe; um aperto de mão que falha
  cai lá; NADA de ferramenta de consumir/alertar/reprocessar a fila de erro — isso é trabalho real,
  pra depois, quando houver volume e falha de verdade) **+ RETRY BÁSICO LIGADO.**
- **Registro honesto:** o retry ligado no marco 1 é ESCOLHA do Brian e **divergiu do meu palpite**
  ("deixar sem retry — o esqueleto só precisa provar que a mensagem atravessa a fronteira, e um
  soluço se re-dispara na mão"). Ele optou por ligar pra pôr a mão na mecânica cedo — legítimo, é
  joia que ele quer aprender (mesma lógica de priorizar o publish/a liga). Decisão final é dele.
- **Formato exato do retry** (nº de tentativas, retry imediato × com intervalo, second-level retry)
  = detalhe do sub-fio (f)/portão de código, a confirmar na doc do MassTransit, não afirmar de
  memória.

> **Régua aplicada (onde tem trabalho × cosplay × engenharia prematura):** Outbox e trava-de-efeito
> ficam FORA do marco 1 porque não têm ALVO lá — inventar alvo falso pra "usar o padrão" é a cosplay
> que o briefing proíbe. DLQ é diferente: a falha já existe e o backstop é grátis, então fica; o
> excesso a evitar NELA não é cosplay, é engenharia prematura (ferramenta de monitorar/reprocessar
> sem volume). Retry entra cedo por ESCOLHA de aprendizado, não por necessidade do esqueleto.

### Sub-fio (f) — MassTransit como a biblioteca ✅ FECHADO (sessão 8)

Último fio da mensageria. Quase todo CONFIRMAÇÃO (transporte RabbitMQ, semântica Send/Publish,
saga e as 3 joias já estavam fechados; a MassTransit é a ferramenta concreta embaixo de tudo).
Fatos de ferramenta conferidos por busca (ago/2026), não afirmados de memória.

**1. Biblioteca e versão: MassTransit v8 (Apache 2.0).** ✅
- A nota antiga marcava "v8 MIT serve" com bomba-relógio ("fim de manutenção fim de 2026"). A data
  chegou (estamos em ago/2026) → conferido na fonte. **Correção:** o v8 é **Apache 2.0**, não MIT.
- O v8 segue grátis e roda pra sempre; o que acaba no fim de 2026 é o SUPORTE OFICIAL (patches),
  não o funcionamento. O v9 virou comercial (~US$400/mês mínimo).
- **Por que a data quase não morde no nosso caso:** (a) aprendizado pessoal, não produção com dado
  sensível → falta de patch futuro é risco ~zero; (b) v8 grátis pra sempre; (c) o broker está atrás
  de um contrato DE PROPÓSITO → trocar de biblioteca depois é mudança limitada, não reescrita.
- **Peso a favor da v8 (aprendizado):** é *a* referência de mensageria no mundo .NET — doc, tutorial,
  fórum e os próprios conceitos (Send/Publish/saga/outbox/inbox) são moldados nela. Ecossistema mais
  rico = mais valor pro objetivo "aprender o que as empresas usam".
- **Considerados e descartados (justos):** OpenTransit (fork comunitário da v8, aberto, mira .NET
  10+ — mas anunciado dez/2025, jovem demais pra ser porto seguro); Wolverine/Rebus (abertas/MIT,
  mas API diferente → troca o ecossistema rico por proteção contra risco que mal existe aqui);
  NServiceBus e v9 (comerciais). Escolha do Brian.

**2. Formato do retry: `Immediate(3)`, sem redelivery no marco 1.** ✅
- **Distinção que decide o desenho (da doc):** *retry* ≠ *redelivery*. Retry = tentar de novo NA
  HORA, algumas vezes — e roda em memória segurando um lock na mensagem, então só serve pra erro
  CURTO e passageiro (rede piscou, banco ocupado meio segundo); retry longo TRAVA o consumidor.
  Redelivery = re-tentar DEPOIS DE MINUTOS (5/15/30…), pra outage longo (banco caiu de vez).
- Decisão: `Immediate(3)` (três tentativas na hora; falhou as três → vai pro `_error`, a DLQ grátis
  do (e)). **Sem redelivery** no marco 1 — é pra falha demorada, over-engineering pro esqueleto. O
  "3" é default de bom senso; trocar pra 5 é uma linha, não muda o desenho.

**3. Repositório da saga: em memória no marco 1.** ✅ A MassTransit tem repositório de saga em
memória pronto → casa direto com o "saga na memória" fechado no (e) martelo 0. Troca pra um durável
(via EF Core) quando o marco do humano/durabilidade chegar. (Descartado: já durável agora —
contradiz o (e), esqueleto não precisa.)

**4. Idempotência estrutural = DE GRAÇA (confirma o (e) martelo 2).** ✅
- A saga liga cada mensagem à partida certa pelo Correlation ID (o crachá do sub-fio b), e a máquina
  de estados só aceita cada evento no estado certo → cópia atrasada bate numa saga que já passou
  daquele ponto e é ignorada por natureza. Isso é a idempotência estrutural, sem código extra.
- `UseInMemoryOutbox` no consumidor evita republicar os avisos de saída quando há retry (não dispara
  efeito de saída em dobro na re-tentativa).
- O que **não** vem de graça: a trava fina "já vi a mensagem de id X" → vem junto do recurso de
  Inbox/Outbox DURÁVEL, que já foi adiado pro marco do Outbox. Bate certinho com o (e): estrutural
  agora, trava de efeito depois.

**5. Outbox transacional embutido (nota, não ação).** ✅ A MassTransit já traz o Outbox transacional
pronto (com EF Core). Quando a casa do Outbox chegar (fim de partida: grava resultado + publica
`PartidaEncerrada`), é LIGAR, não construir na mão. Nada a configurar agora.

---

## HISTÓRICO — Mensageria Game ↔ Opponent (a→f) ❌ SUPERADA COMO TOPOLOGIA

Todos os seis sub-fios do fluxo de mensagens estão fechados: (a) quais mensagens cruzam · (b)
formato dos contratos · (c) estilo (comando/`Send` + saga) · (d) pausar-e-reagir · (e) Outbox/
idempotência/DLQ · (f) MassTransit v8 + detalhes de ferramenta. **Não há mais sub-fio de mensageria
aberto.** O próximo passo do projeto é o **ROADMAP PULVERIZADO** (ver "FASE PÓS-SABATINA" abaixo).

### Guardado (aberto, não é de nenhum sub-fio ainda):
- ⬜ **O que fazer se o adversário FURA o prazo da escalação** (humano): escala a última usada? sorteia? W.O.? joga com o que tem? — detalhe de pausar-e-reagir (4C) / comportamento do robô. Anotado pra não perder.
- ⬜ **Quem aplica o modificador antes da fila** (quando "efetivo" ≠ "cru", degrau futuro): o atributo efetivo é calculado por quem MONTA a mensagem, antes de publicar. Bandeira plantada no martelo 4 do sub-fio (b). Não é fio de hoje.
- ⬜ **Endereçar o manager certo** se um processo hospedar vários managers (multi-inquilino): identificador de manager pra ROTEAMENTO, não no corpo. Flag do martelo 6 do (b).

---

## HISTÓRICO — Fase pós-sabatina antiga ❌ ROADMAP SUBSTITUÍDO PELA FASE A

Quando a sabatina fechar, ANTES de codar, produzir:

1. **Documentação consolidada** do jogo (juntar este arquivo + o mapa-jogador +
   decisões que vierem, num desenho coeso).
2. **Roadmap pulverizado em etapas evolutivas** — o plano de entregas. Vocabulário
   de empresa (pro Brian aprender o nome das coisas):
   - **Roadmap** = o mapa das entregas em ordem.
   - **Milestone / marco** = cada entrega fechada (a Seção 4I já tem 8 rascunhados).
   - **Issue / tarefa** = o "o que fazer hoje" dentro de um marco.
   - **Definition of Done (definição de pronto)** = a lista do que precisa estar
     verdadeiro pra o marco contar como terminado. Para este projeto (Seção 5):
     código + teste + doc/ADR + CI verde.
3. **Objetivo do Brian com isso:** saber EXATAMENTE o que fazer em cada etapa, sem
   ambiguidade. Isso É a prática de empresas boas — ele pediu a coisa certa.
