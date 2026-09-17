# Mapa Mestre de Implementação — Manager de Futebol

> Documento de navegação da construção. Ele não substitui as cinco fontes oficiais; organiza o caminho de implementação para que seja possível saber **onde estamos, o que já existe, o que vem depois e como reconhecer que um bloco terminou**.

---

## VOCÊ ESTÁ AQUI

```text
Documentação oficial ........ ✅ Fechada
Mapa Mestre .................. ✅ Fechado em 07/09/2026 (emendado em 17/09/2026)
Manual de Implementação ...... 🔄 Em construção
Implementação oficial ........ 🔄 Em andamento — Parte A

Parte A:
A1 ✅   A2 ✅   A3 ✅   A5 ✅ (antecipado)

Bloco atual:
A4 — Configurar referências entre projetos

Próximo:
A6 — Preparar PostgreSQL local
```

> O código experimental criado antes deste mapa permanece suspenso e não define a baseline oficial.

---

## Emenda de 17/09/2026

Três decisões foram tomadas e este Mapa foi corrigido para refletí-las. O registro com
justificativa está em `decisoes-jogo-futebol_FINAL.md` §25.

1. **Plataforma: .NET 10**, substituindo .NET 8. Consequência direta: a biblioteca
   `UUIDNext` sai do projeto, porque o `Guid.CreateVersion7()` passa a ser nativo.
2. **Nomes oficiais: `Cartola90` e `Cartola90.slnx`**, substituindo `manager-football`
   e `ManagerFootball.sln`.
3. **Formato da solution: `.slnx`.** O `.sln` era exigido porque o SDK do .NET 8 não lê
   `.slnx`; com .NET 10 essa restrição desapareceu.

Também foi reconhecido que os blocos A1, A2, A3 e A5 já haviam sido executados entre
09/09 e 11/09/2026, fora da ordem prevista — A5 antes de A3, com justificativa legítima
registrada no Manual. O marcador acima foi corrigido para descrever o estado real.

---

# 1. Hierarquia documental

A documentação tem papéis diferentes.

## 1.1 Fontes oficiais — definem a verdade do produto

1. `briefing-jogo-futebol_FINAL.md`
2. `decisoes-jogo-futebol_FINAL.md`
3. `mapa-jogador_FINAL.md`
4. `sistema-dia-1_FINAL.md`
5. `roadmap-jogo-futebol_FINAL.md`

Esses cinco documentos definem produto, regras, fronteiras, decisões e ordem oficial dos marcos.

## 1.2 Mapa Mestre — organiza a construção

Este documento responde:

> **O que será construído, em qual ordem e como reconhecer que cada parte chegou ao resultado esperado?**

Ele pode repetir nomes de peças, dependências e fronteiras estáveis, mas evita duplicar números, contratos e regras detalhadas que já possuem fonte normativa.

## 1.3 Manual de Implementação — ensina a executar o bloco atual

O futuro `manual-implementacao.md` responderá:

> **Como executar, passo a passo, a parte em que estamos trabalhando agora?**

Ele conterá comandos, arquivos, verificações, troubleshooting e explicações de implementação.

## 1.4 Regra de precedência

Se houver conflito:

```text
Documento oficial
        >
Mapa Mestre
        >
Manual de Implementação
```

O documento derivado deve ser corrigido. Código existente não ganha autoridade por inércia.

---

# 2. Visão do caminho inteiro

```text
5 DOCUMENTOS OFICIAIS
        │
        ▼
PREPARAÇÃO DA OFICINA
        │
        ▼
MARCO 1
Cadastros e preparação dos times
        │
        ▼
MARCO 2
Primeira partida interativa
        │
        ▼
MARCO 3
Lifecycle de abandono e crash
        │
        ▼
MARCO 4
Mensageria confiável no fim da partida
        │
        ▼
MARCO 5
Manager Inbox ponta a ponta
        │
        ▼
MARCO 6
Primeira intervenção esportiva
```

O Mapa Mestre mostra todos os marcos. Apenas a preparação e o Marco 1 recebem agora profundidade suficiente para virarem Manual de Implementação. Nos Marcos 2–6, **resumir decisão já existente é permitido; desenhar nova decisão não é**.

---

# PARTE A — Preparação da oficina do Marco 1

## 3. Objetivo

Preparar o repositório, a estrutura da aplicação e as ferramentas mínimas para começar a construir funcionalidades reais do Marco 1.

Ao terminar:

- backend compila e sobe;
- frontend compila e abre;
- PostgreSQL sobe;
- API alcança o banco;
- fronteiras dos quatro projetos .NET estão montadas;
- testes já existentes executam;
- CI verifica o repositório;
- o ambiente local é reproduzível.

Ainda não existe `Club`, `Player`, `Manager` ou qualquer feature do domínio.

---

## 4. Decisões técnicas fechadas

### Backend

- C#;
- .NET 10;
- ASP.NET Core Web API;
- Controllers;
- Clean Architecture;
- `Game.Domain`;
- `Game.Application`;
- `Game.Infrastructure`;
- `Game.Api`.

### Persistência

- PostgreSQL;
- EF Core 10;
- Npgsql;
- migrations;
- IDs representados como `Guid`;
- geração UUIDv7 nativa via `Guid.CreateVersion7()`.

> **Emenda 17/09/2026.** A versão anterior previa a biblioteca `UUIDNext` porque o .NET 8
> não gera UUIDv7 sozinho. Com .NET 10 o recurso é nativo, e a dependência foi removida
> antes de existir: o projeto não deve adicioná-la.

### Frontend

- React;
- TypeScript;
- Vite;
- pnpm;
- Node 24 LTS.

### Testes

- xUnit;
- FluentAssertions;
- Moq quando mocks forem úteis;
- Coverlet;
- Testcontainers quando infraestrutura real precisar ser provada;
- testes arquiteturais quando as fronteiras ganharem a primeira verificação executável.

### Execução futura

No recorte inicial, apenas uma Match ficará viva simultaneamente. Isso é limitação operacional inicial, não lei estrutural do domínio.

---

## 5. Estrutura física do repositório

```text
Cartola90/
│
├── Cartola90.slnx
├── README.md
├── global.json
├── docker-compose.yml
│
├── docs/
│   ├── briefing-jogo-futebol_FINAL.md
│   ├── decisoes-jogo-futebol_FINAL.md
│   ├── mapa-jogador_FINAL.md
│   ├── sistema-dia-1_FINAL.md
│   ├── roadmap-jogo-futebol_FINAL.md
│   ├── mapa-mestre-implementacao.md
│   └── manual-implementacao.md
│
├── src/
│   ├── Game.Domain/
│   ├── Game.Application/
│   ├── Game.Infrastructure/
│   ├── Game.Api/
│   └── Game.Web/
│
├── tests/
│   └── projetos de teste nascem quando ganham função real
│
└── .github/
    └── workflows/
        └── ci.yml
```

---

## 6. Dependências entre projetos .NET

### Permitidas

```text
Game.Application → Game.Domain

Game.Infrastructure → Game.Application
Game.Infrastructure → Game.Domain

Game.Api → Game.Application
Game.Api → Game.Infrastructure
```

### Proibidas

```text
Game.Domain → Application / Infrastructure / Api
Game.Application → Infrastructure / Api
Game.Infrastructure → Api
```

`MatchEngine` e `Manager AI` continuam componentes internos do Game; não recebem `.csproj` próprio agora.

---

## 7. Ambiente local

Ferramentas esperadas na máquina do desenvolvedor:

- Git;
- SDK .NET 10;
- Node 24 LTS;
- pnpm;
- Docker;
- IDE/editor;
- acesso ao GitHub.

No desenvolvimento inicial:

```text
Game.Api  → executado na máquina
Game.Web  → executado na máquina
PostgreSQL → Docker
```

API e Web ficam fora do container para preservar debug, hot reload e ciclo de aprendizado simples. A composição completa poderá existir quando houver necessidade real de provar o ambiente inteiro em containers.

---

## 8. Sequência de montagem da oficina

### A1 — Criar e preparar o repositório ✅

**Faz nascer:** repositório Git, branch principal, README e documentação versionada.

**Conferência:** clonar em diretório limpo e confirmar que a documentação e o README estão presentes.

---

### A2 — Criar a solution ✅

**Faz nascer:** `Cartola90.slnx`.

**Conferência:** a CLI .NET reconhece a solution sem erro.

---

### A3 — Criar os quatro projetos do backend ✅

```text
Game.Domain
Game.Application
Game.Infrastructure
Game.Api
```

**Não criar agora:** projeto de MatchEngine, Manager AI, Contracts, Competition, Inbox ou mensageria.

**Conferência:** os quatro projetos aparecem na solution e compilam vazios.

---

### A4 — Configurar referências ⬜ BLOCO ATUAL

Aplicar somente as dependências aprovadas na seção 6.

**Conferência:** build da solution passa; dependências proibidas não existem.

**Limite deste bloco:** o compilador passa a impedir a dependência proibida *por ausência
de referência*, mas não impede que alguém adicione a referência errada depois. Fechar esse
buraco é papel do teste arquitetural, que pertence a A10.

---

### A5 — Fixar o SDK .NET ✅ *(executado entre A2 e A3)*

Criar `global.json` com uma versão válida do SDK .NET 10 escolhida no momento da execução.

**Conferência:** uma máquina com o SDK compatível resolve a versão esperada ao executar `dotnet --version` dentro do repositório.

> **Nota de ordem.** Este bloco foi deliberadamente executado antes de A3. Motivo: A3 é o
> primeiro momento em que templates geram `.csproj` reais, e eles não devem ser criados
> enquanto o repositório não controla qual SDK a CLI seleciona. A numeração foi mantida
> para não invalidar referências cruzadas.

---

### A6 — Preparar PostgreSQL local

Criar o serviço PostgreSQL no Docker.

**Não criar:** tabela fictícia para “provar” EF.

**Conferência:** container sobe saudável e aceita conexão com as credenciais configuradas para desenvolvimento.

---

### A7 — Ligar EF Core e Npgsql

`Game.Infrastructure` recebe o mecanismo de persistência e o `DbContext` inicial.

**Conferência:** a API inicia com a infraestrutura registrada e consegue alcançar o PostgreSQL. A primeira migration de negócio só nasce quando existir a primeira entidade real persistível.

---

### A8 — Subir a API mínima

A base operacional deve possuir apenas o necessário para existir e ser diagnosticada:

- Controllers;
- DI;
- configuração por ambiente;
- Swagger/OpenAPI;
- health check;
- logging estruturado;
- CorrelationId.

**Conferência manual:** abrir Swagger e health check; ambos respondem sem depender de feature fake.

---

### A9 — Criar o frontend real vazio

Criar `Game.Web` com React + TypeScript + Vite + pnpm.

**Conferência manual:** dependências restauram, TypeScript compila, Vite inicia e a página base abre no navegador.

---

### A10 — Preparar a estratégia de testes

Os projetos de teste não nascem todos vazios.

- teste arquitetural: nasce quando houver a primeira regra de dependência que queremos proteger executavelmente;
- `Game.Domain.Tests`: nasce com a primeira regra de domínio;
- `Game.Application.Tests`: nasce com o primeiro caso de uso;
- `Game.IntegrationTests`: nasce com a primeira persistência real.

**Conferência:** qualquer projeto de teste que exista possui pelo menos uma responsabilidade real e executa no build/CI.

---

### A11 — Preparar Docker Compose

No começo, o Compose representa apenas a topologia real disponível. Para desenvolvimento, o único serviço obrigatório é PostgreSQL.

Não adicionar RabbitMQ, Manager Inbox ou Competition antes de seus marcos.

**Conferência:** `docker compose up` sobe somente os componentes previstos para este momento.

---

### A12 — Preparar CI

O GitHub Actions verifica apenas o que existe:

- restore/build backend;
- testes já existentes;
- restore/check/build frontend.

Não configurar antecipadamente deploy, cloud, RabbitMQ, Kubernetes ou Sonar.

**Conferência:** um push limpo obtém pipeline verde.

---

## 9. A oficina está pronta quando

É possível partir de um clone limpo e obter:

```text
✓ backend restaura e compila
✓ API inicia
✓ Swagger abre
✓ health check responde
✓ PostgreSQL sobe
✓ API alcança o PostgreSQL
✓ React restaura dependências
✓ TypeScript compila
✓ frontend abre
✓ testes existentes executam
✓ CI fica verde
✓ nenhuma feature falsa foi criada para provar infraestrutura
```

A partir daqui começa o domínio real do Marco 1.

---

# PARTE B — Marco 1: Cadastros e preparação dos times

## 10. Missão

Construir tudo que precisa existir para responder:

> **Há dois clubes reais, com jogadores e técnicos reais, os vínculos estão corretos, o Human possui TeamSetup válido e a CPU consegue preparar sua equipe?**

O Marco 1 termina imediatamente antes de nascer a primeira Match.

---

## 11. Ordem funcional

```text
OFICINA PRONTA
      │
      ▼
M1.1 — Núcleo cadastral
Club + Player + Manager
      │
      ▼
M1.2 — Vínculos atuais
Player ↔ Club / Manager ↔ Club
      │
      ▼
M1.3 — Modelo esportivo
atributos + posições + formações + ManagerProfile
      │
      ▼
M1.4 — Elenco utilizável
cadastro individual + lote + 16 por clube
      │
      ▼
M1.5 — React do núcleo
      │
      ▼
M1.6 — TeamSetup Human
      │
      ▼
M1.7 — SimpleManagerAI
      │
      ▼
M1.8 — Ready to Play
      │
      ▼
MARCO 1 CONCLUÍDO
```

### Correção explícita em relação ao roadmap anterior

A ordem anterior colocava React antes de `NaturalPosition`, embora a própria tela de jogadores já exigisse filtro por posição natural. O roadmap oficial foi corrigido para fazer posições/formações nascerem antes do React. Isso remove uma dependência invertida sem alterar o escopo do Marco 1.

Fonte normativa: `roadmap-jogo-futebol_FINAL.md` — Marco 1.

---

## 12. M1.1 — Núcleo cadastral

### Objetivo

Criar as três identidades independentes:

```text
Club
Player
Manager
```

Player e Manager podem existir sem clube. Club pode existir sem jogadores e sem Manager atual.

### Não antecipar

- contratos;
- salários;
- estádio;
- torcida;
- finanças;
- hierarquias `HumanManager`/`CpuManager`.

Manager usa `ControlType = Human | Cpu`.

### Onde existe trabalho

- Domain;
- Application;
- Infrastructure;
- API;
- banco;
- testes correspondentes.

### Como você confere

**Manual via Swagger:** criar e consultar pelo menos um Club, um Player e um Manager como entidades independentes.

**Automatizado:** testar invariantes que já existirem nesse bloco.

### Resultado observável

Você consegue demonstrar, sem acesso direto ao código, que as três entidades existem e podem ser persistidas/consultadas independentemente.

Fonte normativa: `sistema-dia-1_FINAL.md` — §3.1 e §3.2; `decisoes-jogo-futebol_FINAL.md` — §§4–5.

---

## 13. M1.2 — Vínculos atuais

### Objetivo

Representar separadamente pertencimento atual e existência da entidade.

```text
PlayerClubAssignment
ManagerClubAssignment
```

### Invariantes principais

- Player: no máximo um clube atual;
- Manager: no máximo um clube atual;
- Club: no máximo um Manager atual;
- Player e Manager podem ficar sem clube.

Não criar lista persistida duplicada de jogadores dentro de `Club`.

### Como você confere

**Swagger:** criar entidades, vincular, consultar vínculo atual, desvincular e tentar uma operação que violaria a cardinalidade atual. O backend deve rejeitar a violação.

**Testes:** invariantes de vínculo e persistência onde necessário.

### Resultado observável

O sistema responde de forma confiável quem pertence atualmente a quem e rejeita dois vínculos atuais incompatíveis.

Fonte normativa: `sistema-dia-1_FINAL.md` — §3; `decisoes-jogo-futebol_FINAL.md` — §4.

---

## 14. M1.3 — Modelo esportivo atual + ManagerProfile

### Objetivo

Dar ao Player apenas os dados esportivos usados pelo recorte inicial e criar as informações de preferência do Manager necessárias à preparação CPU.

### Jogador de linha

Atributos diretos:

```text
Finalização
Drible
Passe
Ritmo
Defesa
Físico
```

Todos no intervalo 0–100.

Derivações:

```text
QualidadeAtaque =
(Finalização + Drible + Passe + Ritmo + Físico) / 5

QualidadeDefesa =
(Defesa + Ritmo + Físico) / 3
```

### Goleiro

Atributos diretos:

```text
Mergulho
Manuseio
Chute
Reflexos
Velocidade
Posicionamento
```

Derivação:

```text
QualidadeGoleiro =
(Mergulho + Manuseio + Chute + Reflexos + Velocidade + Posicionamento) / 6
```

### Conhecimento posicional

```text
NaturalPosition
SecondaryPositions
```

As posições e formações suportadas são as definidas nos documentos oficiais. `Improvised` é condição derivada, nunca posição persistida.

### ManagerProfile

Nasce neste bloco porque `PreferredFormation` depende do conjunto de formações reconhecidas:

```text
ManagerProfile
- ManagerId
- PreferredFormation
```

Regra:

- CPU: `PreferredFormation` obrigatória;
- Human: opcional.

`ManagerProfile` não é `TeamSetup`.

### Como você confere

**Swagger:** criar/editar jogadores com atributos e posições válidos; provocar entradas fora de 0–100 e combinações posicionais inválidas; cadastrar perfil de Manager CPU com formação suportada.

**Testes:** validar as três fórmulas, limites 0–100 e invariantes de posições/formações.

Se uma derivação não estiver exposta na API, a prova dessa fórmula fica nos testes automatizados, não em endpoint artificial.

### Resultado observável

Você consegue mostrar que Player e Manager já carregam a informação esportiva necessária aos próximos blocos e que entradas inválidas são rejeitadas.

Fonte normativa: `mapa-jogador_FINAL.md` — “O QUE O RECORTE INICIAL REALMENTE USA”; `decisoes-jogo-futebol_FINAL.md` — §§7 e 7A; `sistema-dia-1_FINAL.md` — §§3.3, 5.2, 5.3 e 6.

---

## 15. M1.4 — Elenco utilizável

### Objetivo

Montar os dois elencos do primeiro mundo jogável por operações reais.

### Capacidades

- criação individual de Player;
- criação em lote;
- mesmas validações no individual e no lote;
- lote atômico;
- clube comum opcional no lote;
- consulta global com busca/filtros/paginação necessários;
- consulta do elenco de um clube.

O recorte inicial utiliza 16 jogadores por clube. Isso não limita estruturalmente Club para sempre.

### Como você confere

**Swagger:** criar jogadores individualmente e em lote; provar que um lote inválido não cria parcialmente os demais; consultar um clube e confirmar seu elenco atual.

### Resultado observável

Os dois clubes podem ter elencos completos montados sem seed como única forma de uso, e o cadastro em lote respeita atomicidade.

Fonte normativa: `roadmap-jogo-futebol_FINAL.md` — Marco 1, tarefa 4; `sistema-dia-1_FINAL.md` — §§4 e 5.1.

---

## 16. M1.5 — React do núcleo

### Objetivo

Operar visualmente os conceitos já existentes no backend.

Áreas iniciais:

- Clubes;
- Jogadores;
- Técnicos;
- Escalação.

O React não contém a regra oficial de negócio. Pode orientar/avisar, mas o backend permanece autoridade.

### Como você confere

Pelo navegador, executar os fluxos reais de cadastro, edição, consulta, busca, filtro, vínculo/desvínculo e lote já construídos nos blocos anteriores.

### Resultado observável

O primeiro mundo pode ser montado pela interface real do produto, sem depender do Swagger para o uso cotidiano.

Fonte normativa: `sistema-dia-1_FINAL.md` — §4; `decisoes-jogo-futebol_FINAL.md` — §10.

---

## 17. M1.6 — TeamSetup Human

### Objetivo

Permitir que o Manager Human prepare e persista seu trabalho atual no clube.

```text
TeamSetup
- ManagerId
- ClubId
- Formation
- Slots
```

A formação define os slots; Player conhece posições; TeamSetup coloca jogadores específicos nos slots.

Regras detalhadas permanecem nas fontes oficiais. Neste mapa, o essencial é que o Human consegue montar 11 titulares e os 5 restantes formam o banco do recorte inicial.

### Como você confere

No React, montar uma escalação válida, salvar, sair da tela, reabrir e confirmar a mesma configuração. Alterar o vínculo de um jogador escalado e confirmar que o TeamSetup não é silenciosamente reescrito e passa a impedir preparação válida até correção.

### Resultado observável

A escalação do Human é persistente, reabrível e protegida contra inconsistências relevantes.

Fonte normativa: `sistema-dia-1_FINAL.md` — §5.4; `decisoes-jogo-futebol_FINAL.md` — §9.

---

## 18. M1.7 — SimpleManagerAI

### Objetivo

Preparar automaticamente o lado CPU com o elenco real do clube.

O algoritmo completo e o `ScoreNoSlot` permanecem normativos nas fontes oficiais. O Mapa Mestre não os replica além do necessário para navegação.

### Fronteiras

SimpleManagerAI:

- usa elenco, posições, atributos e `PreferredFormation`;
- escolhe titulares e banco;
- respeita as formações suportadas;
- retorna falha real quando nenhuma formação válida puder ser produzida;
- não calcula λ;
- não simula Match;
- não conhece RabbitMQ.

### Como você confere

Acionar pela aplicação a preparação/validação do lado CPU e observar a escalação gerada. Testes automatizados cobrem cenários de preferência, fallback e impossibilidade de montagem.

A forma exata da operação HTTP/UI será decidida no Manual do bloco; não criar endpoint artificial apenas para expor método interno.

### Resultado observável

Um clube CPU válido produz uma escalação real sem fixture fixa/fake, e um elenco impossível produz falha explícita.

Fonte normativa: `sistema-dia-1_FINAL.md` — §5.5; `decisoes-jogo-futebol_FINAL.md` — §8.

---

## 19. M1.8 — Ready to Play

### Objetivo

Responder de forma confiável:

> **cada lado está realmente preparado para uma futura Match?**

A Match ainda não nasce.

### Human

Depende de Club, Manager Human, elenco e TeamSetup válidos.

### CPU

Depende de Club, Manager CPU, elenco, ManagerProfile e capacidade real de o SimpleManagerAI gerar preparação válida.

### Como você confere

Montar o mundo real dos dois lados e executar a validação de preparação. Depois quebrar propositalmente uma condição relevante — por exemplo, remover um jogador necessário ou invalidar o setup — e confirmar `Not Ready` com causa explícita.

### Resultado observável

```text
Human = READY
CPU   = READY
```

Esse é o ponto exato em que o Marco 1 termina.

Fonte normativa: `roadmap-jogo-futebol_FINAL.md` — Marco 1, tarefa 8.

---

## 20. Linha de chegada completa do Marco 1

Fluxo de prova:

```text
criar Club Human
      ↓
criar/vincular Manager Human
      ↓
criar/vincular elenco
      ↓
montar TeamSetup
      ↓
Human = READY

criar Club CPU
      ↓
criar/vincular Manager CPU + ManagerProfile
      ↓
criar/vincular elenco
      ↓
SimpleManagerAI prepara equipe
      ↓
CPU = READY
```

Tudo pela aplicação real, com PostgreSQL, API, React e testes/CI correspondentes.

---

## 21. De propósito, não entra no Marco 1

- Match;
- MatchEngine;
- relógio da partida;
- SignalR da partida;
- pausas;
- histórico da partida;
- RabbitMQ;
- MassTransit;
- Outbox;
- Manager Inbox;
- substituição;
- Competition.

---

# PARTE C — Mapa global dos Marcos 2–6

## 22. Regra desta parte

Esta seção é **mapa de destino**, não especificação de implementação.

Ela pode resumir decisões já fechadas para que o caminho seja compreensível. Quando uma regra exigir número, contrato ou algoritmo detalhado, a fonte oficial vence e deve ser consultada.

Não tomar novas decisões de baixo nível para um marco futuro apenas porque ele aparece neste mapa.

---

## 23. Marco 2 — Primeira partida interativa

### Pergunta

> Dados dois lados `READY`, conseguimos executar uma partida real do início ao fim?

### Peças que entram

- `Match` como execução concreta;
- persistência do nascimento e do histórico inicial;
- `MatchEngine` interno;
- `IRandomSource`;
- execução viva em memória;
- relógio de simulação;
- estados vivos `Running`, `Paused` e `Halftime`;
- HTTP para comandos;
- SignalR para atualizações;
- pausas e intervalo conforme regras oficiais;
- finalização `Finished`;
- histórico/súmula consultável no Game.

`Finished` é status persistido/finalização; não faz parte da lista de estados vivos acima.

### Resultado

Uma Match nasce, é acompanhada ao vivo, termina normalmente e pode ser consultada depois.

### Fonte normativa

- `sistema-dia-1_FINAL.md` — §§7–11, 13;
- `decisoes-jogo-futebol_FINAL.md` — §§11–15;
- `roadmap-jogo-futebol_FINAL.md` — Marco 2.

---

## 24. Marco 3 — Lifecycle de interrupção

### Pergunta

> O que acontece quando uma execução nasce, mas não chega normalmente ao fim?

### Peças que entram

- `Sair` explícito;
- distinção entre desconexão e saída;
- `NotCompleted`;
- detecção/reconciliação de `InProgress` órfã após crash;
- nova identidade/Seed para nova tentativa;
- manutenção da decisão de **não** haver checkpoint/resume.

### Resultado

O histórico distingue `Finished` de `NotCompleted`, e uma queda do processo não deixa execução órfã parecendo válida indefinidamente.

### Fonte normativa

- `sistema-dia-1_FINAL.md` — §12;
- `decisoes-jogo-futebol_FINAL.md` — §16;
- roadmap — Marco 3.

---

## 25. Marco 4 — Mensageria confiável de `PartidaEncerrada`

### Pergunta

> Uma partida terminada consegue comunicar seu fato a outros processos sem colocar o resultado em risco?

### Peças que entram

- RabbitMQ;
- MassTransit;
- primeiro contrato de integração real;
- MassTransit Transactional Outbox com EF Core;
- publicação de `PartidaEncerrada`;
- comportamento correto quando broker ou persistência falham.

Não criar consumidor fake apenas para justificar o broker.

### Resultado

Finalização da Match e intenção de publicação ficam consistentes; indisponibilidade temporária do broker não invalida uma Match já terminada.

### Fonte normativa

- `sistema-dia-1_FINAL.md` — §§14–15;
- `decisoes-jogo-futebol_FINAL.md` — §§17–18 e 21A;
- roadmap — Marco 4.

---

## 26. Marco 5 — Manager Inbox E2E

### Pergunta

> Um processo independente consegue reagir a `PartidaEncerrada` e produzir valor real ao usuário?

### Peças que entram

- processo Manager Inbox;
- banco próprio do Inbox;
- consumo de `PartidaEncerrada`;
- `InboxItem`;
- idempotência de efeito;
- retry/fila de erro conforme política oficial;
- lista/detalhe no React;
- leitura `false → true`;
- “Ver súmula” consultando o Game pelo `MatchId`.

Não duplicar a súmula no Inbox.

### Resultado

Uma Match terminada gera mensagens úteis para os Managers envolvidos e o fluxo pode ser acompanhado ponta a ponta.

### Fonte normativa

- `sistema-dia-1_FINAL.md` — §16;
- `decisoes-jogo-futebol_FINAL.md` — §§19 e 21A;
- roadmap — Marco 5.

---

## 27. Marco 6 — Primeira intervenção esportiva

### Pergunta

> O Manager Human consegue alterar efetivamente o futuro de uma Match em andamento?

### Peças que entram

- composição viva alterável;
- substituição humana;
- invariantes da troca;
- contadores oficiais de pausa/parada/substituição;
- `MatchEvent Substitution`;
- recálculo da força/λ apenas para o futuro quando a composição efetivamente muda;
- atualização de React, SignalR e súmula.

Números e regras completas dos contadores permanecem nas fontes oficiais para evitar duplicação normativa.

### Resultado

Uma substituição válida muda quem está em campo, é registrada, pode alterar o cálculo futuro e aparece na história final da Match.

### Fonte normativa

- `sistema-dia-1_FINAL.md` — §17;
- `decisoes-jogo-futebol_FINAL.md` — §§15 e 20;
- roadmap — Marco 6.

---

# 28. Matriz global de trabalho

## Semântica

- **TRABALHO**: a peça nasce ou recebe alteração relevante naquele marco.
- **USA**: o marco depende dela, mas ela não é o foco de construção daquela etapa.
- **—**: não participa de forma relevante do marco.

| Capacidade | Preparação | M1 | M2 | M3 | M4 | M5 | M6 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Estrutura Clean do Game | TRABALHO | USA | USA | USA | USA | — | USA |
| PostgreSQL do Game | TRABALHO | TRABALHO | TRABALHO | TRABALHO | TRABALHO | USA | USA |
| React + TypeScript | TRABALHO | TRABALHO | TRABALHO | USA | — | TRABALHO | TRABALHO |
| Club / Player / Manager | — | TRABALHO | USA | — | — | — | USA |
| TeamSetup | — | TRABALHO | USA | — | — | — | USA |
| SimpleManagerAI | — | TRABALHO | USA | — | — | — | — |
| Match | — | — | TRABALHO | TRABALHO | TRABALHO | USA | TRABALHO |
| MatchEngine | — | — | TRABALHO | USA | — | — | TRABALHO |
| Execução viva em memória | — | — | TRABALHO | TRABALHO | — | — | TRABALHO |
| SignalR | — | — | TRABALHO | USA | — | — | TRABALHO |
| Histórico / súmula | — | — | TRABALHO | USA | — | USA | TRABALHO |
| NotCompleted | — | — | — | TRABALHO | — | — | — |
| RabbitMQ / MassTransit | — | — | — | — | TRABALHO | USA | — |
| Transactional Outbox | — | — | — | — | TRABALHO | USA | — |
| PartidaEncerrada | — | — | — | — | TRABALHO | USA | — |
| Manager Inbox | — | — | — | — | — | TRABALHO | USA |
| Banco do Inbox | — | — | — | — | — | TRABALHO | USA |
| Idempotência do consumidor | — | — | — | — | — | TRABALHO | USA |
| Substituição humana | — | — | — | — | — | — | TRABALHO |

---

# 29. Evolução da topologia

## Preparação / Marco 1

```text
Game.Web
   │
   ▼
Game.Api
   │
   ▼
PostgreSQL
```

## Marco 2 / Marco 3

```text
Game.Web
   │ HTTP + SignalR
   ▼
Game
   │
   ▼
PostgreSQL

+ estado vivo de Match em memória
```

## Marco 4

```text
Game.Web
   │
   ▼
Game
 ┌─┴─────────┐
 ▼           ▼
Game DB    RabbitMQ
```

## Marco 5 em diante

```text
                Game DB
                  ▲
                  │
Game.Web ───────► Game
                  │
                  ▼
               RabbitMQ
                  │
                  ▼
             Manager Inbox
                  │
                  ▼
               Inbox DB
```

Marco 6 aprofunda comportamento esportivo dentro do Game; não adiciona novo processo.

---

# 30. Progressão de problemas aprendidos

```text
M1
Domínio + persistência + API + React
        ↓
M2
Motor + tempo + estado vivo + SignalR
        ↓
M3
Lifecycle + falhas de processo
        ↓
M4
Transação + Outbox + broker
        ↓
M5
Fronteira de processo + consumer + idempotência
        ↓
M6
Estado esportivo mutável durante execução
```

O objetivo não é usar tecnologias por catálogo. Cada tecnologia entra quando existe um problema real que a justifica.

---

# 31. Depois do Marco 6

Continuam nomeadas, mas não desenhadas em detalhe neste Mapa:

- Competition, campeonato, calendário e classificação;
- mais clubes;
- CPU mais inteligente durante jogo;
- mudanças táticas/formações dinâmicas;
- cartões, lesões, cansaço;
- evolução de jogador;
- contratos/diretoria/torcida;
- dados reais;
- autenticação/autorização;
- multiplayer humano;
- calibração avançada;
- cloud/deploy.

Essas áreas só serão detalhadas quando houver decisão explícita de trazê-las para o caminho real.

---

# 32. Próximo documento

Quando este Mapa Mestre for considerado fechado, o próximo artefato é:

```text
manual-implementacao.md
```

Ele começa no primeiro passo executável da Parte A e aprofunda **somente o trabalho atual**.

O Manual deve permitir que o desenvolvedor:

- saiba o que fazer;
- entenda por que está fazendo;
- saiba em qual projeto/arquivo trabalhar;
- execute comandos quando necessários;
- confira manual e automaticamente o resultado;
- saiba onde parar;
- saiba o que investigar antes de avançar quando o resultado divergir.
