# Roadmap — Jogo Manager de Futebol
### Os marcos de construção, em ordem, cada um com sua linha de chegada

> **O que é este arquivo.** O plano de construção do núcleo inicial do jogo: as decisões
> já fechadas nos documentos oficiais viradas em uma **sequência de marcos pequenos,
> cumulativos e termináveis**. Aqui mora o *o quê* e o *em que ordem construir*; o
> `sistema-dia-1.md` descreve como o sistema deve existir quando esse núcleo estiver
> completo, e o `decisoes-jogo-futebol.md` preserva o porquê das decisões.
>
> **Regra de precedência.** Este roadmap substitui a sequência antiga baseada em
> Opponent Service, cumprimento via RabbitMQ e reação do robô pelo broker. A arquitetura
> vigente começa com **Game + React + domínio persistente**, mantém o `Manager AI` dentro
> do Game e só introduz RabbitMQ quando aparece a primeira fronteira real de integração:
> `PartidaEncerrada` → `Manager Inbox`.
>
> **Como este arquivo é escrito.** A linguagem continua deliberadamente prática: cada
> marco informa o objetivo, a ordem de construção, a linha de chegada e o que fica de fora.
> O roadmap não redesenha regra já fechada e não antecipa features futuras só para “deixar
> espaço pronto”.

---

## Como ler

**O que é um marco.** Um bloco de trabalho que:

1. entrega uma capacidade nova que funciona de verdade;
2. se apoia no que já ficou pronto antes;
3. termina em um estado utilizável e testável;
4. não depende de feature fake para justificar tecnologia futura.

**A linha de chegada — igual para todo marco.** Um marco só está pronto quando:

1. **Código:** o comportamento novo funciona ponta a ponta dentro do escopo daquele marco.
2. **Teste:** existem testes automáticos suficientes para provar as regras novas.
3. **Decisão:** decisões arquiteturais caras de reverter recebem uma nota/ADR curta quando
   necessário; parâmetros de game design não viram ADR.
4. **Verde:** build e testes passam no CI.

**O chão que vale para todos — e não é um marco separado.**

- Clean Architecture: regra de futebol não depende de React, banco ou broker.
- DDD onde existe regra real a proteger; invariantes ficam junto do dono da regra.
- Result Pattern para falhas esperadas de negócio.
- Casos de uso explícitos na Application.
- EF Core por interfaces consumidas pela Application; Infrastructure implementa.
- DTO HTTP, domínio e contrato de integração não são o mesmo modelo.
- `IRandomSource` centraliza todo acaso do motor.
- minuto esportivo é estado da simulação; wall-clock entra apenas onde há tempo real.
- React envia comandos por HTTP; o servidor continua autoridade da partida.
- logging estruturado + `CorrelationId`.
- testes rápidos no domínio/motor; Testcontainers apenas quando banco/broker real for
  necessário.
- docker-compose cresce junto com a topologia real do produto.
- CI desde o começo.
- não introduzir MediatR, Saga, Redis, Kubernetes, OpenTelemetry, Event Sourcing,
  AutoMapper, Specification, Unit of Work custom ou novos microsserviços sem problema real
  que justifique a ferramenta.

---

## Os marcos, em ordem

1. **Marco 1 — Cadastros e preparação dos times.**
2. **Marco 2 — Primeira partida interativa.**
3. **Marco 3 — `NotCompleted`.**
4. **Marco 4 — Outbox + RabbitMQ + `PartidaEncerrada`.**
5. **Marco 5 — Manager Inbox E2E.**
6. **Marco 6 — Primeira intervenção esportiva.**

A sequência é intencional:

```text
preparar o mundo
      ↓
rodar uma partida real
      ↓
tratar execução interrompida
      ↓
publicar o fato de encerramento com consistência
      ↓
consumir esse fato em outro processo real
      ↓
permitir a primeira decisão esportiva durante o jogo
```

O RabbitMQ **não** é usado para preparar escalação da CPU, nem para fazer o Game conversar
com um Opponent Service. O `Manager AI` permanece dentro do Game.

---

# Marco 1 — Cadastros e preparação dos times

**Objetivo, em uma linha.** Construir o mundo mínimo jogável: dois clubes, jogadores,
técnicos, vínculos e escalações persistentes, com o lado humano preparado pelo usuário e o
lado CPU preparado pelo `SimpleManagerAI`. **Ainda não existe partida e ainda não existe
RabbitMQ.**

## Entrega funcional

Ao terminar este marco deve ser possível:

- cadastrar `Club`;
- cadastrar `Player` globalmente;
- cadastrar `Manager` globalmente;
- vincular/desvincular Player ↔ Club;
- vincular/desvincular Manager ↔ Club;
- diferenciar Manager `Human` e `Cpu` por `ControlType`;
- cadastrar `PreferredFormation` da CPU;
- manter 16 jogadores por clube no primeiro mundo jogável;
- montar e persistir o `TeamSetup` do lado humano;
- deixar o `SimpleManagerAI` montar a escalação inicial da CPU;
- validar que os dois lados estão prontos para uma futura Match.

## Tarefas, na ordem de construção

### 1. Subir o esqueleto real do produto

Criar a base executável do que já existe neste marco:

- Game / ASP.NET Core Web API;
- banco relacional do Game via EF Core + migrations;
- React;
- health check/Swagger/configuração básica;
- logging estruturado;
- CI;
- docker-compose apenas com os componentes que realmente existem agora.

Não subir RabbitMQ vazio. Não criar Manager Inbox vazio. Não criar Opponent Service.

**Saída desta tarefa:** backend, frontend e banco sobem; build e testes básicos passam.

### 2. Implementar `Club`, `Player`, `Manager` e os vínculos

Modelar como entidades independentes.

Vínculos atuais:

```text
PlayerClubAssignment
ManagerClubAssignment
```

Regras do recorte:

- Player tem no máximo um clube atual;
- Manager tem no máximo um clube atual;
- Club tem no máximo um Manager atual;
- Player e Manager podem existir sem clube.

Não criar contrato completo, salário, multa ou rescisão.

O `Player` já precisa nascer com os dados realmente usados pelo núcleo inicial:

- jogador de linha: Finalização, Drible, Passe, Ritmo, Defesa e Físico, todos 0–100;
- goleiro: Mergulho, Manuseio, Chute, Reflexos, Velocidade e Posicionamento, todos 0–100;
- `QualidadeAtaque`, `QualidadeDefesa` e `QualidadeGoleiro` são derivadas, não campos
  cadastrados como outra fonte de verdade.

Não implementar agora os atributos finos futuros do `mapa-jogador.md`.

**Saída desta tarefa:** domínio + persistência + casos de uso conseguem criar entidades com
os atributos efetivamente usados pelo jogo e alterar vínculos preservando invariantes.

### 3. Implementar posições reconhecidas pelo domínio e `ManagerProfile`

Player possui:

- `NaturalPosition`;
- zero ou mais `SecondaryPositions`.

Formações iniciais:

- 4-4-2;
- 4-3-3;
- 3-5-2.

Posições reconhecidas:

`GK · CB · RB/LB · RWB/LWB · DM · CM · AM · RM/LM · RW/LW · ST`

Regras:

- NaturalPosition não se repete em SecondaryPositions;
- improvisação é derivada, não persistida;
- não existe penalização matemática por improvisação neste recorte;
- jogador de linha pode ser improvisado;
- quando o Human improvisar, a UI deve avisar que o jogador está fora de Natural/Secondary;
- não criar multiplicador/penalização numérica apenas para deixá-lo “desligado”;
- slot GK exige goleiro real.

Com as formações reconhecidas pelo domínio, criar também `ManagerProfile`:

- associado ao `Manager`;
- `PreferredFormation` obrigatória para CPU;
- `PreferredFormation` opcional para Human;
- `ManagerProfile` representa preferência/configuração do técnico e não se confunde com
  `TeamSetup`.

**Saída desta tarefa:** o domínio sabe se um jogador está em posição natural, secundária ou
improvisada sem armazenar um “estado de improvisado”, e o perfil do Manager já possui a
preferência de formação necessária para a preparação futura do lado CPU.

> **Correção de ordem registrada em 07/09/2026.** Nesta versão, posições e formações
> reconhecidas pelo domínio vêm antes dos cadastros React. A versão anterior colocava o
> React primeiro, embora a própria tela de jogadores já exigisse filtro por
> `NaturalPosition`. A alteração remove essa dependência invertida; não muda o escopo do
> Marco 1.

### 4. Fechar o elenco do primeiro mundo jogável

Preparar os fluxos necessários para montar os dois clubes reais do recorte:

- criação individual de Player;
- criação em lote reutilizando as mesmas validações do fluxo individual;
- lote atômico: uma linha inválida cancela o lote inteiro;
- clube comum opcional no lote;
- um clube com Manager Human;
- um clube com Manager Cpu;
- 16 jogadores em cada clube;
- 11 titulares + 5 reservas na configuração de jogo;
- idealmente 2 goleiros no elenco.

No primeiro mundo jogável os dois clubes já são o par disponível; **não existe seleção de
adversário neste recorte**.

O número 16 é recorte inicial, não limitação estrutural eterna da entidade Club.

**Saída desta tarefa:** ambos os clubes podem ser montados por casos de uso reais, inclusive
em lote, e possuem elenco suficiente para preparação real.

### 5. Construir os cadastros React reais

A interface nasce como parte do produto, não como wizard descartável.

Áreas mínimas:

- Clubes;
- Jogadores;
- Técnicos;
- Escalação.

Jogadores:

- criar individualmente;
- criar em lote;
- editar;
- consultar;
- cadastrar e validar os seis atributos 0–100 aplicáveis ao jogador de linha ou ao goleiro;
- vincular/desvincular clube;
- busca por nome;
- filtro por clube atual ou sem clube;
- filtro por posição natural;
- paginação server-side simples na lista global;
- elenco do clube exibido sem paginação.

Cadastro em lote:

- funciona em grade;
- aceita clube comum opcional;
- reutiliza as mesmas validações do cadastro individual;
- é atômico: uma linha inválida cancela o lote inteiro.

Managers:

- identidade;
- `ControlType`;
- `ManagerProfile.PreferredFormation`;
- vínculo/desvínculo com Club.

A UI apenas opera conceitos que já existem no domínio e na aplicação. Em especial, o filtro
por `NaturalPosition` só aparece depois de o conhecimento posicional existir.

**Saída desta tarefa:** o primeiro mundo pode ser montado e conferido pela interface real,
sem seed manual como única forma de uso.

### 6. Implementar `TeamSetup` persistente do humano

`TeamSetup` representa o trabalho atual do técnico naquele clube:

```text
TeamSetup
- ManagerId
- ClubId
- Formation
- Slots
```

Regras:

- usuário escolhe formação suportada;
- monta 11 titulares;
- os 5 restantes formam o banco do recorte;
- configuração persiste entre partidas futuras até ser alterada;
- TeamSetup inválido bloqueia início de futura Match;
- se jogador escalado deixar o clube, não apagar silenciosamente a configuração.

**Saída desta tarefa:** o lado humano pode ser salvo e reaberto com a mesma configuração.

### 7. Implementar `SimpleManagerAI`

`SimpleManagerAI` é componente interno do Game e prepara o lado CPU.

Fluxo mínimo:

1. recebe elenco disponível;
2. tenta `PreferredFormation`;
3. cria slots;
4. busca 11 jogadores únicos;
5. prioriza Natural/SecondaryPositions;
6. improvisa linha apenas quando necessário;
7. compara candidatos possíveis com `ScoreNoSlot`;
8. exige goleiro real em GK;
9. define 11 titulares;
10. deixa os 5 restantes no banco;
11. se a formação preferida não puder ser montada, tenta as outras suportadas;
12. se nenhuma formação suportada produzir uma escalação válida, retorna falha de domínio.

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

O score reutiliza matemática já existente no domínio: não cria λ paralelo nem penalização de
improvisação.

Ele não calcula λ, não simula a partida e não conhece mensageria.

**Saída desta tarefa:** o lado CPU é montado de forma real, sem escalação fixa/fake, e uma
impossibilidade real de montagem aparece como falha de domínio explícita.

### 8. Criar a validação “pronto para jogar”

Antes de o Marco 2 existir, deixar explícito o que significa um lado estar pronto:

- Club válido;
- Manager atual válido;
- elenco do recorte disponível;
- TeamSetup humano válido ou escalação CPU gerável;
- 11 titulares únicos;
- 5 reservas;
- GK real no slot GK;
- todos os escalados pertencem ao clube atual.

**Saída desta tarefa:** o sistema consegue responder se os dois lados estão prontos para
uma Match, mesmo sem ainda criar uma.

## Pronto é quando…

- **Código:** é possível montar o primeiro mundo pelo React, persistir os vínculos,
  persistir o TeamSetup humano, gerar o setup CPU e obter dois lados válidos para jogar.
- **Teste:** existem testes de invariantes de vínculo, atributos 0–100 e qualidades
  derivadas, validações de posições, lote atômico, TeamSetup e SimpleManagerAI — inclusive
  falha de domínio quando nenhuma formação suportada puder ser montada; integração com banco
  apenas onde necessário.
- **Decisão:** ficam registradas, quando necessário, as decisões estruturais de entidades
  independentes, TeamSetup persistente e Manager AI interno.
- **Verde:** CI compila e roda os testes com sucesso.

## De propósito, NÃO entra aqui

- Match;
- MatchEngine;
- relógio de partida;
- SignalR da partida;
- pausas;
- histórico de partida;
- RabbitMQ;
- MassTransit;
- Outbox;
- Manager Inbox;
- substituição;
- Competition.

---

# Marco 2 — Primeira partida interativa

**Objetivo, em uma linha.** Pegar os dois lados preparados no Marco 1 e rodar a primeira
Match real no servidor: persistida como `InProgress` ao nascer, executada minuto a minuto em
memória, acompanhada ao vivo pelo React e concluída normalmente como `Finished` com
histórico e súmula. **Ainda sem substituição e ainda sem RabbitMQ.**

> **Nota de sequência.** O comando `Continue` pertence a este marco porque encerra a pausa
> normal. A semântica `Sair → NotCompleted` é implementada somente no Marco 3, junto do
> lifecycle de abandono/crash; assim não existe um botão de saída com comportamento
> provisório/fake no Marco 2.

## Entrega funcional

A primeira partida já deve ter:

- `MatchId` próprio;
- nova Seed por execução;
- EngineVersion para diagnóstico;
- `Match.Status = InProgress` persistido no nascimento;
- fotografia inicial dos relacionados;
- MatchEngine;
- execução minuto a minuto;
- 6 minutos reais de bola;
- 30 segundos reais de intervalo;
- SignalR;
- 3 pausas manuais de até 60 segundos;
- comando `Continue`;
- final normal;
- `Finished`;
- placar;
- eventos de `Goal`;
- histórico e súmula no Game.

## Tarefas, na ordem de construção

### 1. Criar a Match e persistir o nascimento

A Match identifica uma **execução concreta**.

Ordem:

1. validar os dois lados;
2. gerar MatchId;
3. gerar nova Seed;
4. registrar EngineVersion;
5. persistir Match como `InProgress`;
6. persistir os fatos iniciais do histórico;
7. criar estado vivo em memória.

Persistir no nascimento pelo menos:

- MatchId;
- `InProgress`;
- StartedAt;
- ClubId + ClubName histórico dos dois lados;
- ManagerIds;
- 16 relacionados de cada lado;
- PlayerId + PlayerName histórico;
- papel inicial `Starter` / `Substitute`;
- Seed;
- EngineVersion.

Não persistir minuto corrente, placar parcial, RNG corrente, pausa viva ou evento provisório.

**Saída desta tarefa:** uma Match existe historicamente antes de começar a correr.

### 2. Implementar o MatchEngine isolado

O MatchEngine é componente interno do Game e a única fonte da matemática.

Implementar e testar o que já está fechado no `sistema-dia-1.md`:

- qualidades de ataque/defesa por jogador;
- pesos posicionais;
- defesa com mistura 70% linha / 30% goleiro;
- `λ = 1,35 × 2^((Ataque − DefesaAdversária) / 30)`;
- mando aplicado sobre λ;
- `p = 1 − e^(−λ/90)` por minuto;
- clamps de λ;
- `IRandomSource` central;
- Seed reproduzível para diagnóstico;
- produção de `Goal`.

O motor não conhece React, SignalR, EF Core, RabbitMQ ou Manager Inbox.

**Saída desta tarefa:** dados dois times preparados e uma fonte de acaso, o motor consegue
avançar a simulação de forma testável.

### 3. Criar o runner vivo da partida

Separar matemática esportiva de cadência real.

Configuração inicial:

- minutos 1–45 em 3 minutos reais;
- 30 segundos de `Halftime`;
- minutos 46–90 em 3 minutos reais;
- equivalência inicial: 1 minuto esportivo = 4 segundos reais de bola correndo.

Estados vivos:

- `Running`;
- `Paused`;
- `Halftime`.

Status persistido continua `InProgress` durante os três.

**Saída desta tarefa:** o servidor consegue manter a execução viva em memória até o minuto
90, sem depender do navegador.

### 4. Ligar a experiência ao React com HTTP + SignalR

Fluxo fechado:

```text
comandos:
React → HTTP → Game

atualizações:
Game → SignalR → React
```

Atualizações mínimas:

- minuto/estado;
- placar;
- novos eventos;
- `Running`;
- `Paused`;
- `Halftime`;
- `Finished`.

Ao abrir/reabrir uma Match ainda viva:

```text
GET /matches/{id}/live
```

1. React busca fotografia atual por HTTP;
2. monta o estado atual;
3. conecta/reconecta no SignalR;
4. acompanha somente as mudanças seguintes.

F5, aba fechada ou queda do SignalR não matam nem pausam a execução.

**Saída desta tarefa:** a partida pode ser assistida de forma contínua sem o React calcular
nada esportivo.

### 5. Implementar as pausas manuais

Existem 3 pausas manuais por partida.

Cada pausa:

- dura no máximo 60 segundos reais;
- congela a simulação;
- não processa minuto;
- não avança RNG;
- não recalcula λ apenas por existir;
- consome uma pausa ao abrir;
- pode terminar antes por `Continue`;
- retoma automaticamente quando o tempo termina.

Neste marco a pausa **não possui substituição nem mudança tática**. Ela existe para fechar o
mecanismo de tempo/controle antes da primeira intervenção esportiva.

**Saída desta tarefa:** pausar e continuar funciona sem alterar a matemática passada ou
avançar a simulação por trás.

### 6. Implementar o intervalo

Ao fim do minuto 45:

- entrar em `Halftime`;
- congelar o avanço esportivo;
- contar 30 segundos reais;
- não consumir pausa manual;
- iniciar automaticamente o segundo tempo ao zerar.

Sem substituição ainda.

**Saída desta tarefa:** uma partida completa possui dois tempos com intervalo real.

### 7. Fechar a finalização normal

Ao terminar o minuto 90:

- fixar placar final;
- fixar eventos definitivos existentes neste marco (`Goal`);
- persistir `Match.Status = Finished`;
- persistir `FinishedAt`;
- encerrar o estado vivo;
- avisar o React por SignalR.

Neste marco ainda **não existe evento de integração**. A costura transacional com Outbox
entra no Marco 4.

**Saída desta tarefa:** a Match concluída é um resultado oficial persistido no Game.

### 8. Expor histórico e súmula

O Game é o dono do histórico esportivo.

Persistir conceitos reais:

- Match;
- RelatedPlayers;
- MatchEvents.

Todos os 16 relacionados de cada lado aparecem na súmula, inclusive reservas que nunca
entraram.

Preservar identidade histórica suficiente:

- PlayerId;
- PlayerName usado na partida;
- ClubId;
- ClubName usado na partida;
- papel inicial Starter/Substitute.

Consulta oficial:

```text
GET /matches/{MatchId}/report
```

Não criar agregado persistido duplicado `MatchReport` apenas para a visão.

**Saída desta tarefa:** uma Match `Finished` pode ser consultada depois, com placar,
relacionados e eventos.

### 9. Fechar a rede de testes do motor e da execução

Cobrir:

- funções matemáticas puras;
- invariantes de Match;
- mesma Seed + mesma EngineVersion quando usada para diagnóstico controlado;
- testes estatísticos da distribuição;
- relógio esportivo separado do wall-clock;
- pausa não avançando minuto/RNG;
- intervalo;
- transição normal `InProgress → Finished`;
- SignalR apenas em testes de integração onde fizer sentido.

**Saída desta tarefa:** a partida está protegida contra regressão matemática e de lifecycle
normal.

## Pronto é quando…

- **Código:** o usuário inicia uma Match válida, assiste aos 90 minutos em 6 minutos reais,
  passa pelo intervalo, pode usar as três pausas/Continue, recebe atualizações por SignalR e
  termina com Match `Finished`, placar, gols e súmula persistidos.
- **Teste:** motor, lifecycle normal, pausa, intervalo e histórico têm cobertura automática;
  a distribuição do motor possui testes estatísticos.
- **Decisão:** permanecem documentadas as decisões caras: servidor autoritativo, Match
  persistida no nascimento, estado vivo sem checkpoint e MatchEngine interno.
- **Verde:** CI passa.

## De propósito, NÃO entra aqui

- substituição;
- tática durante a partida;
- `NotCompleted` por saída/crash — entra no Marco 3;
- Outbox;
- RabbitMQ;
- `PartidaEncerrada`;
- Manager Inbox;
- Competition;
- lesão/cartão/pênalti/cansaço.

---

# Marco 3 — `NotCompleted`

**Objetivo, em uma linha.** Tornar explícito e auditável o destino de partidas que não
chegam ao fim: saída manual e crash encerram a execução sem inventar resultado, sem retomar
do meio e sem reutilizar a identidade antiga.

## Entrega funcional

- `Sair` explícito → `NotCompleted`;
- crash pode deixar `InProgress` órfã;
- startup reconhece órfã e marca `NotCompleted`;
- a execução viva não é reconstruída;
- a próxima tentativa recebe novo MatchId e nova Seed;
- a execução antiga continua no histórico;
- `NotCompleted` não é W.O.;
- `NotCompleted` não produz resultado oficial.

## Tarefas, na ordem de construção

### 1. Implementar `Sair`

O comando explícito:

- encerra a execução viva;
- muda o status histórico para `NotCompleted`;
- não cria placar oficial;
- não transforma em W.O.;
- não tenta continuar depois;
- mantém a Match antiga consultável como execução não concluída.

**Saída desta tarefa:** o usuário consegue abandonar sem apagar o fato de que aquela
execução existiu.

### 2. Diferenciar desconexão de saída

F5, aba fechada ou SignalR desconectado:

- não significam `Sair`;
- não mudam status;
- não pausam automaticamente;
- a execução continua no Game.

Ao voltar, o usuário recupera fotografia atual por HTTP e reconecta no SignalR.

**Saída desta tarefa:** falha de interface não vira abandono esportivo.

### 3. Tratar crash do Game

Se o processo morrer:

- estado vivo em memória se perde;
- não reconstruir minuto;
- não reconstruir placar parcial;
- não reconstruir eventos provisórios;
- não reconstruir cursor do RNG;
- não reconstruir escalação viva alterada.

**Saída desta tarefa:** a ausência de checkpoint/resume vira comportamento deliberado, não
acidente implícito.

### 4. Criar reconciliação de startup

Na subida do Game:

> Match persistida como `InProgress` sem execução viva correspondente → `NotCompleted`.

A reconciliação deve ser segura para rodar a cada startup.

**Saída desta tarefa:** nenhuma Match fica eternamente `InProgress` depois de crash.

### 5. Garantir nova identidade na próxima tentativa

Qualquer nova execução posterior:

- novo MatchId;
- nova Seed;
- novo StartedAt;
- novo snapshot inicial.

Não reutilizar a Match anterior.

**Saída desta tarefa:** MatchId continua significando execução concreta.

### 6. Testar cenários de interrupção

Cobrir:

- `Sair → NotCompleted`;
- desconexão do navegador não encerra;
- startup converte `InProgress` órfã;
- nova execução não reutiliza MatchId nem Seed;
- nenhum mecanismo tenta resume.

## Pronto é quando…

- **Código:** saída manual e crash deixam histórico coerente; órfãs são reconciliadas;
  nova tentativa nasce com nova identidade.
- **Teste:** abandono, desconexão, crash simulado/startup e nova execução estão cobertos.
- **Decisão:** fica explícito que não existe checkpoint/resume e que `NotCompleted` não é
  resultado esportivo.
- **Verde:** CI passa.

## De propósito, NÃO entra aqui

- W.O.;
- “reinício rápido” como feature especial;
- recuperação técnica da Match;
- publicação de evento de resultado;
- RabbitMQ;
- Manager Inbox.

---

# Marco 4 — Outbox + RabbitMQ + `PartidaEncerrada`

**Objetivo, em uma linha.** Introduzir a primeira integração real do sistema: quando uma
Match termina normalmente, o Game grava o resultado e a intenção de publicar o fato
`PartidaEncerrada` de forma consistente; RabbitMQ indisponível não invalida o resultado.

## Entrega funcional

- RabbitMQ entra pela primeira vez;
- MassTransit v8 entra pela primeira vez;
- MassTransit Transactional Outbox + EF Core no Game;
- finalização normal e intenção de publicação ficam na mesma transação;
- `PartidaEncerrada` usa `Publish`;
- resultado `Finished` continua válido mesmo com broker fora;
- não existe consumidor fake de produção apenas para justificar o broker.

## Tarefas, na ordem de construção

### 1. Adicionar RabbitMQ e MassTransit à topologia real

Atualizar a infraestrutura local apenas agora:

- RabbitMQ no docker-compose;
- configuração do MassTransit no Game;
- health/configuração necessária;
- CorrelationId propagado para a integração.

Não recriar Opponent Service. Não criar Saga.

**Saída desta tarefa:** o Game consegue usar a infraestrutura de mensageria sem ainda
alterar a semântica da Match.

### 2. Criar o primeiro contrato real em `Contracts`

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

Semântica:

- MatchId = execução concreta;
- EventId = ocorrência concreta do evento de integração.

**Saída desta tarefa:** `Contracts` continua mínimo e recebe somente o que realmente cruza
processos.

### 3. Integrar MassTransit Transactional Outbox com EF Core

Não construir Outbox caseira.

A finalização normal passa a persistir na mesma transação:

- placar final;
- eventos definitivos;
- `Match → Finished`;
- FinishedAt;
- intenção de publicar `PartidaEncerrada`.

**Saída desta tarefa:** resultado e intenção de publicação não podem ficar em “meio estado”
por uma queda entre salvar e publicar.

### 4. Publicar `PartidaEncerrada`

Após o commit, a infraestrutura entrega o evento pelo broker.

Regra:

> fato ocorrido → `Publish`.

Não usar `Send` para esse caso.

**Saída desta tarefa:** o fim normal da Match produz o primeiro evento de integração real.

### 5. Tratar broker indisponível

Se RabbitMQ estiver fora:

- a Match corretamente finalizada continua `Finished`;
- o resultado continua oficial;
- a intenção continua no Outbox;
- publicação pode acontecer depois.

Broker fora não converte Match em `NotCompleted`.

**Saída desta tarefa:** disponibilidade do broker não controla a verdade esportiva do Game.

### 6. Tratar falha da persistência final

Se a gravação final falhar enquanto o processo ainda vive:

- repetir a persistência do mesmo resultado já calculado quando apropriado;
- não rodar MatchEngine novamente;
- não gerar nova Seed.

Se o processo morrer antes do commit:

- a execução viva se perde;
- a Match inicial eventualmente será reconciliada como `NotCompleted` pelo Marco 3.

**Saída desta tarefa:** falha de persistência não cria uma segunda simulação do mesmo jogo.

### 7. Testar o contrato transacional

Cobrir com infraestrutura real onde necessário:

- Match termina → fica `Finished` + intenção registrada;
- evento é publicado;
- broker fora → Match continua válida e evento fica pendente;
- restabelecimento → evento é entregue;
- falha antes do commit não deixa resultado parcial oficialmente finalizado.

Usar Testcontainers para banco/broker nos testes que dependem deles.

## Pronto é quando…

- **Código:** uma Match normal termina, persiste resultado + Outbox atomicamente e gera
  `PartidaEncerrada`; desligar RabbitMQ não apaga nem invalida o resultado.
- **Teste:** há integração real provando Outbox + broker, inclusive indisponibilidade.
- **Decisão:** ficam registradas as decisões caras: primeiro Publish real, Transactional
  Outbox do MassTransit e broker fora da autoridade esportiva.
- **Verde:** CI passa.

## De propósito, NÃO entra aqui

- Manager Inbox em produção — entra no Marco 5;
- consumidor fake apenas para “provar RabbitMQ”;
- Saga;
- DLQ custom;
- redelivery atrasado;
- Competition.

---

# Marco 5 — Manager Inbox E2E

**Objetivo, em uma linha.** Colocar o primeiro consumidor real do evento de integração em
outro processo: o Manager Inbox recebe `PartidaEncerrada`, cria `MatchResult`, protege o
efeito contra duplicidade e permite ao técnico abrir a mensagem e consultar a súmula no
Game.

## Entrega funcional

- processo `Manager Inbox` separado;
- banco próprio do Inbox;
- consumo de `PartidaEncerrada`;
- `InboxItem` persistido;
- `MatchResult` como primeiro tipo;
- idempotência de efeito;
- retry `Immediate(3)`;
- fila `_error` padrão após falhas;
- lista;
- detalhe;
- `IsRead`;
- “Ver súmula” consultando o Game.

## Tarefas, na ordem de construção

### 1. Criar o processo Manager Inbox

Adicionar somente agora:

- serviço/processo separado;
- banco próprio;
- EF Core + migrations;
- configuração MassTransit;
- health/logging/configuração;
- docker-compose atualizado.

Game e Inbox não compartilham banco.

**Saída desta tarefa:** o primeiro consumidor real existe como fronteira independente.

### 2. Modelar `InboxItem`

Modelo mínimo:

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

Primeiro `Type`:

```text
MatchResult
```

Não incluir:

- EventId;
- CorrelationId;
- ReadAt;
- Seed;
- EngineVersion;
- cópia da súmula.

Não criar herança de classes por tipo de mensagem neste recorte.

**Saída desta tarefa:** o Inbox possui um modelo simples de produto, não uma réplica do
contrato de integração.

### 3. Consumir `PartidaEncerrada`

O consumer:

- recebe o fato;
- cria um `MatchResult` para `Home.ManagerId`;
- cria um `MatchResult` para `Away.ManagerId`;
- os dois itens usam o mesmo `MatchId`;
- cada item pertence ao respectivo `ManagerId`;
- persiste os itens no banco do Inbox.

A regra é **um item por Manager da partida**, não uma mensagem global única do confronto.
A idempotência continua calculada por Manager + Match + Type.

O Game não precisa saber como o Inbox monta título/texto.

**Saída desta tarefa:** terminar partida cria uma mensagem real para cada Manager da Match.

### 4. Implementar idempotência de efeito

Chave de negócio:

```text
ManagerId + MatchId + Type
```

Criar constraint UNIQUE.

Comportamento:

- primeira entrega cria a mensagem;
- duplicata equivalente vira no-op;
- não criar segunda mensagem.

**Saída desta tarefa:** entrega at-least-once do broker não duplica efeito de produto.

### 5. Configurar retry e `_error`

Política inicial:

```text
Immediate(3)
```

Sem redelivery atrasado.

Se falhar após as tentativas:

- a mensagem segue para a fila `_error` padrão do MassTransit.

Não construir monitor operacional sofisticado neste marco.

**Saída desta tarefa:** falha transitória tenta novamente; falha persistente fica isolada
sem loop infinito.

### 6. Construir lista e detalhe no React

Superfícies mínimas:

- ícone/entrada para Inbox;
- lista de mensagens;
- detalhe.

Endpoints já fechados:

```text
GET /managers/{managerId}/inbox
GET /managers/{managerId}/inbox/{inboxItemId}
```

Regras:

- nova mensagem nasce `IsRead = false`;
- listar não marca como lida;
- abrir detalhe muda `false → true`;
- depois de lida, não volta a `false`.

**Saída desta tarefa:** o Manager consegue usar a Inbox como feature real, não apenas ver
registro no banco.

### 7. Implementar “Ver súmula” via Game

Inbox guarda apenas `MatchId`.

Ao pedir a súmula:

```text
Inbox/React → Game → GET /matches/{MatchId}/report
```

Não copiar a súmula para o banco do Inbox.

**Saída desta tarefa:** Inbox referencia o fato; Game continua dono do histórico esportivo.

### 8. Fechar teste E2E da integração

Cenário mínimo real:

1. uma Match termina normalmente;
2. Outbox publica `PartidaEncerrada`;
3. RabbitMQ entrega;
4. Manager Inbox consome;
5. nasce um `InboxItem` para cada Manager da partida;
6. repetir a mesma entrega não duplica nenhum dos dois itens;
7. lista mostra mensagem não lida;
8. detalhe marca lida;
9. “Ver súmula” recupera o relatório no Game.

Usar banco e broker reais descartáveis no teste E2E.

## Pronto é quando…

- **Código:** terminar uma Match cria uma mensagem utilizável na Inbox; duplicata não
  duplica; falha de consumo segue retry/_error; abrir marca lido; súmula vem do Game.
- **Teste:** existe fluxo ponta a ponta Game → Outbox → RabbitMQ → Inbox → UI/consulta, com
  idempotência comprovada.
- **Decisão:** fica claro que Inbox é dono da mensagem e Game é dono da Match/súmula.
- **Verde:** CI passa.

## De propósito, NÃO entra aqui

- copiar súmula para Inbox;
- histórico esportivo duplicado;
- hierarquia de mensagens complexa;
- notificação push externa;
- outro consumidor artificial;
- Competition.

---

# Marco 6 — Primeira intervenção esportiva

**Objetivo, em uma linha.** Permitir a primeira decisão humana que realmente altera o
futuro da simulação: substituição durante pausa manual ou intervalo, respeitando os limites
do futebol e recalculando força/λ apenas para os minutos seguintes.

## Entrega funcional

- substituição humana;
- durante pausa manual ou intervalo;
- máximo de 5 substituições;
- máximo de 3 paradas com bola rolando;
- 3 pausas manuais continuam contador separado;
- troca altera quem está em campo;
- registra `MatchEvent Substitution`;
- recalcula forças afetadas;
- recalcula λ futuro quando necessário;
- passado não é recalculado;
- SignalR atualiza a tela;
- súmula reflete a troca.

## Tarefas, na ordem de construção

### 1. Introduzir composição viva alterável

Até aqui a fotografia inicial de titulares era suficiente para a partida toda.

Agora a execução viva precisa distinguir:

- quem está em campo agora;
- quem está no banco agora;
- quem já saiu e não pode retornar.

Isso continua em memória durante a execução; não vira checkpoint por minuto.

**Saída desta tarefa:** a Match consegue representar uma troca sem modificar o passado
histórico inicial.

### 2. Criar o comando de substituição

Fluxo:

1. usuário pausa ou usa o intervalo;
2. escolhe quem sai;
3. escolhe quem entra;
4. React envia HTTP;
5. Game valida;
6. altera composição viva;
7. registra `Substitution`;
8. recalcula forças/λ futuros;
9. SignalR publica a nova fotografia/estado.

**Saída desta tarefa:** a substituição atravessa UI → Application → domínio → execução viva
sem broker.

### 3. Implementar invariantes mínimas

- quem entra precisa estar relacionado e no banco;
- quem sai precisa estar em campo;
- quem saiu não volta;
- não pode existir jogador duplicado;
- continuam 11 jogadores em campo.

**Saída desta tarefa:** nenhuma substituição deixa a Match em estado impossível.

### 4. Implementar os três contadores separados

Nunca misturar:

1. pausas manuais: máximo 3;
2. paradas de substituição com bola rolando: máximo 3;
3. substituições totais: máximo 5.

Regras:

- abrir pausa consome 1 pausa manual;
- apenas consultar e continuar não consome parada de substituição;
- confirmar uma ou mais trocas na mesma pausa consome 1 parada;
- várias trocas na mesma pausa contam como uma única parada;
- no intervalo não consome pausa manual;
- no intervalo não consome parada;
- toda troca continua consumindo o limite total de 5.

**Saída desta tarefa:** a mecânica respeita os três saldos sem criar um único contador
ambíguo.

### 5. Registrar `MatchEvent Substitution`

O histórico passa a ter, neste recorte:

- `Goal`;
- `Substitution`.

A substituição precisa registrar informação suficiente para a súmula refletir quem entrou e
quem saiu.

Não criar cartão, lesão ou cansaço junto.

**Saída desta tarefa:** a troca não existe apenas no estado efêmero; o histórico final sabe
que ela ocorreu.

### 6. Recalcular apenas o futuro

A mudança de jogador altera uma entrada efetiva do cálculo.

Fluxo conceitual:

```text
substituição
    ↓
novo conjunto em campo
    ↓
novas forças, se afetadas
    ↓
novo λ(t)
    ↓
probabilidades dos minutos FUTUROS
```

Não recalcular:

- gols passados;
- minutos já processados;
- números já sorteados como se a troca tivesse acontecido antes.

A pausa em si não altera λ; a mudança efetiva de quem está em campo é que pode alterar.

**Saída desta tarefa:** a decisão humana muda a distribuição futura sem reescrever a
história da partida.

### 7. Atualizar React e SignalR

Durante pausa/intervalo, mostrar o necessário para escolher a troca.

Depois de confirmar:

- refletir novo time em campo;
- refletir banco atualizado;
- refletir limites restantes;
- refletir evento de substituição;
- seguir a partida do ponto atual.

**Saída desta tarefa:** a intervenção é visível e operável de ponta a ponta.

### 8. Atualizar súmula e testes

A súmula deve continuar listando todos os 16 relacionados e agora refletir participação por
substituição.

Testar:

- troca válida;
- tentativa com jogador não relacionado;
- tentativa com jogador que não está em campo;
- quem saiu não volta;
- máximo de 5;
- máximo de 3 paradas;
- várias trocas na mesma parada;
- intervalo não consumindo pausa/parada;
- pausa sem troca não consumindo parada;
- λ só muda a partir do momento da mudança efetiva;
- histórico registra `Substitution`.

## Pronto é quando…

- **Código:** o Manager Human consegue substituir durante pausa ou intervalo, os três
  contadores permanecem corretos, a composição muda, o MatchEvent é registrado e o motor
  usa as novas forças apenas para os minutos futuros.
- **Teste:** regras de substituição, contadores, histórico e recálculo futuro estão
  cobertos.
- **Decisão:** fica preservado que a primeira intervenção esportiva é local ao Game via
  HTTP e não cria mensageria artificial.
- **Verde:** CI passa.

## De propósito, NÃO entra aqui

- substituição inteligente da CPU;
- mudança tática/formação livre durante a Match;
- cansaço;
- lesão;
- cartão;
- expulsão;
- pênalti;
- pausa obrigatória;
- instruções condicionais.

---

# Depois do Marco 6 — nomeado, mas não desenhado neste roadmap

O núcleo inicial termina no Marco 6. Os tópicos abaixo continuam fora desta consolidação e
precisam de rodada própria de desenho antes de virar tarefa:

- Competition;
- campeonato;
- vários clubes;
- calendário, rodadas e classificação;
- autenticação/autorização;
- multiplayer humano;
- lesões;
- cartões/expulsões;
- pênaltis;
- cansaço;
- CPU fazendo substituições inteligentes;
- funções táticas;
- formação dinâmica durante a partida;
- instruções condicionais;
- contratos completos;
- diretoria;
- torcida;
- finanças profundas;
- evolução do jogador;
- dados reais externos;
- motor avançado por chutes/xG/PSxG;
- duelos/fases/posicionamento avançado;
- calibração séria contra liga real;
- deploy real em cloud.

**Cloud continua sem provedor escolhido.** AWS, Azure, ECS, RDS, Amazon MQ ou outra
arquitetura de produção só devem ser escolhidos quando a aplicação já existente mostrar o
que precisa ser hospedado.

---

# Critério de saúde do roadmap

A implementação continua coerente se, ao avançar os marcos, estas regras permanecerem
verdadeiras:

1. Marco 1 entrega produto real sem partida e sem RabbitMQ.
2. Manager AI é interno ao Game e não existe serviço adversário separado.
3. Marco 2 prova a partida interativa completa antes da primeira integração externa.
4. Match nasce persistida, mas o estado vivo não vira checkpoint.
5. Marco 3 trata interrupção sem resume e sem reutilizar identidade.
6. RabbitMQ só entra no Marco 4, com `PartidaEncerrada`.
7. Outbox protege persistência + intenção de publicar; broker não decide se o resultado é
   válido.
8. Manager Inbox é o primeiro consumidor real e só nasce no Marco 5.
9. A súmula continua pertencendo ao Game.
10. A primeira intervenção esportiva entra somente no Marco 6 e altera apenas o futuro.
11. Competition permanece fora até receber desenho próprio.
12. Nenhum marco cria feature fake apenas para exercitar uma tecnologia.

---

*Roadmap consolidado da Fase B2 — acompanha `briefing-jogo-futebol.md`,
`decisoes-jogo-futebol.md`, `mapa-jogador.md` e `sistema-dia-1.md`. Revisão cruzada final
dos cinco documentos oficiais concluída.*
