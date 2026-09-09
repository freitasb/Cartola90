# Briefing — Jogo Manager de Futebol
### Visão e princípios consolidados após a Fase A

> **O que é este documento.** É o terreno do projeto: guarda a visão do jogo, os
> princípios que orientam as decisões e as fronteiras já fechadas. Ele **não** é a
> especificação detalhada do sistema e **não** substitui `decisoes-jogo-futebol.md`,
> `sistema-dia-1.md` ou `roadmap-jogo-futebol.md`.
>
> A sabatina do núcleo inicial já terminou. O que foi fechado na Fase A não deve
> voltar a aparecer aqui como pergunta em aberto. As áreas ainda futuras continuam
> apenas como bandeiras, sem desenho prematuro.

---

## 0. Regras de conversa e de evolução do projeto

Estas regras continuam valendo para qualquer nova rodada de desenho.

- **Linguagem simples, sempre.** Termo técnico deve ser explicado quando for
  necessário para entender a decisão.
- **Honestidade direta.** Se uma ideia for gambiarra, enfeite ou "cosplay de
  arquitetura", dizer isso claramente.
- **Uma coisa de cada vez.** Puxar um fio, esgotar o que realmente importa e só
  depois seguir.
- **Ajudar a refletir, não decidir pelo Brian.** Quando houver bifurcação real,
  apresentar custos e benefícios; quando a resposta decorrer de algo já fechado,
  afirmar em vez de reabrir discussão.
- **Separar princípio, padrão e ferramenta.** Ex.: separar a necessidade de
  consistência, o padrão Outbox e a ferramenta MassTransit.
- **Não forçar tecnologia onde não cabe.** Fila, banco, serviço ou framework só
  entram quando resolvem um problema real.
- **Cobertura total, peso proporcional.** Todo detalhe relevante deve ser visto,
  mas nem todo detalhe merece uma rodada inteira de discussão.
- **Não reabrir decisões da Fase A sem contradição real.** O trabalho posterior
  deve construir por cima delas.
- **Não antecipar áreas futuras.** Competition, autenticação, cloud, contratos,
  eventos ricos e outras evoluções só devem ser desenhados quando chegar a vez.

---

## 1. Quem é o Brian e qual é o objetivo

- **Objetivo real:** aprender, na prática, o que uma empresa costuma precisar na
  maioria dos projetos comuns de software. O jogo é a desculpa boa para exercitar
  isso; portfólio é consequência.
- **Método de aprendizado:** aprender amplo sobre um assunto → fazer exercícios →
  aplicar no projeto. Cada tecnologia pode ter seu próprio ciclo de estudo.
- **De onde vem:** o ERP360 ensinou uma base de C#/.NET, domínio, persistência,
  mensageria e organização. O Manager deve aproveitar o que é útil, com mais rigor
  documental e de testes desde o começo.
- **Jeito de trabalhar:** entender o porquê antes de aplicar, registrar decisões e
  crescer o projeto em pedaços pequenos e reais.

---

## 2. O que JÁ está fechado — a moldura atual

Isto é visão vigente. Não relitigar sem contradição real.

1. **É um manager de futebol.** A fantasia é comandar e preparar um clube.

2. **Jogador e clube importam de verdade.** Elenco, posições, atributos e o lado
   humano do futebol são parte central do domínio. O jogo não deve reduzir o clube
   a um número decorativo.

3. **A partida acontece de verdade no servidor, ao longo do tempo.** Não é um
   placar instantâneo nem uma fita pré-calculada para ser reproduzida depois.

4. **A representação pode ser simples.** Relógio, placar, eventos e decisões do
   técnico bastam. Não existe obrigação de campo 3D ou jogadores se movendo na tela.

5. **O usuário acompanha uma partida interativa.** A execução progride minuto a
   minuto e pode ser pausada. Mais tarde, intervenções esportivas alteram apenas o
   que acontecerá dali para frente.

6. **O Game é a autoridade da partida.** A execução pertence ao servidor, não ao
   navegador. Fechar a aba ou perder a conexão da tela não deve destruir uma
   partida que continua viva no Game.

7. **O MatchEngine é componente interno do Game.** Ele é a fonte única da
   matemática da partida e fica isolado de React, SignalR, RabbitMQ, Inbox,
   Competition e banco de dados.

8. **O Manager AI também é componente interno do Game.** No recorte inicial sua
   responsabilidade real é montar a escalação da CPU. Ele não existe em serviço
   separado apenas para justificar mensageria.

9. **A tela roda em React no navegador.** Comandos seguem para o Game pela API web;
   atualizações da partida são empurradas para a tela por SignalR.

10. **RabbitMQ continua no projeto, mas só atravessa fronteira real.** O primeiro
    caso fechado é o fato `PartidaEncerrada`, publicado pelo Game e consumido pelo
    Manager Inbox.

11. **Manager Inbox é processo separado.** Ele possui responsabilidade e
    persistência próprias. A súmula continua pertencendo ao Game.

12. **Competition é futura.** Game sabe como uma partida acontece; Competition
    saberá como uma competição acontece. Campeonato não deve ser redesenhado agora.

13. **Construção por walking skeleton.** Cada etapa deve ser pequena, real e
    atravessar um fluxo útil. Menos funcionalidades é aceitável; funcionalidades
    falsas ou temporárias fingindo o desenho final, não.

14. **Cloud é posterior.** Haverá deploy real em nuvem, mas provedor e topologia só
    serão escolhidos quando a aplicação existente mostrar o que precisa ser
    hospedado.

---

## 3. Arquitetura inicial em alto nível

### Game

O Game é dono, no recorte atual, de:

- Club;
- Player;
- Manager;
- vínculos necessários ao jogo;
- TeamSetup;
- execução da partida;
- histórico/súmula;
- MatchEngine;
- Manager AI.

### Manager Inbox

É o primeiro processo separado que recebe um fato esportivo do Game.

O primeiro caso é simples e real:

> uma partida terminou → o Game anuncia `PartidaEncerrada` → o Inbox cria um
> `MatchResult` para **cada Manager da partida**.

### Competition

Fica fora do recorte inicial.

Quando existir, sua fronteira será de competição, não de simulação da partida.

---

## 3A. Stack e práticas herdadas do ERP360

O Manager reaproveita princípios e ferramentas úteis, mas não transforma a stack
antiga numa lista obrigatória.

### Base que continua fazendo sentido

- C#/.NET e ASP.NET Core Web API;
- Clean Architecture;
- DDD onde há regra real para proteger;
- casos de uso explícitos na Application;
- EF Core e banco relacional;
- Result Pattern para falhas esperadas;
- testes de domínio, aplicação e integração;
- Docker/docker-compose;
- CI desde cedo;
- logging estruturado e Correlation ID;
- React na interface;
- SignalR para atualizações da partida;
- RabbitMQ + MassTransit quando houver integração entre processos;
- Outbox e idempotência quando existir um efeito concreto que precise deles;
- ADR/notinha para decisões arquiteturais significativas.

### O que não entra por catálogo

Não adicionar automaticamente MediatR, Redis, Kubernetes, OpenTelemetry, Event
Sourcing, Saga, AutoMapper, Specification, Unit of Work custom ou novos
microsserviços apenas porque são padrões conhecidos.

O problema vem primeiro; a ferramenta vem depois.

---

## 4. O domínio do clube e do jogador

O clube deve ganhar profundidade ao longo do projeto, mas o recorte atual precisa
ser utilizável sem tentar construir um Football Manager inteiro de uma vez.

### Entidades independentes

Club, Player e Manager são conceitos independentes.

Um jogador ou técnico pode existir sem clube e mudar de vínculo ao longo do tempo.
O vínculo atual com o clube é um conceito próprio; contratos completos ficam para
uma fase futura.

### Manager Human e CPU

Existe uma única entidade Manager, diferenciada pelo tipo de controle:

- Human;
- Cpu.

No recorte inicial há um único Human e podem existir vários CPU. Isso é regra da
versão inicial, não uma proibição arquitetural de multiplayer futuro. Preferências
do técnico, como a formação preferida da CPU, não devem ser confundidas com o
`TeamSetup` persistente do clube.

### Elenco inicial

O primeiro mundo jogável possui dois clubes e 16 jogadores por clube, permitindo
11 titulares e 5 reservas. O domínio não deve ficar eternamente limitado a 16.

### Player atual

O jogador já possui atributos que o motor realmente consome. Para jogadores de
linha são seis atributos gerais; goleiros possuem seis atributos próprios.

Posição natural e posições secundárias também fazem parte do modelo atual.
Improvisação é uma situação derivada do slot utilizado, não um valor persistido.

O `mapa-jogador.md` preserva atributos finos e estados futuros sem obrigar sua
implementação agora.

---

## 5. TeamSetup e preparação da equipe

A escalação do usuário não nasce apenas dentro de uma partida.

Existe uma configuração persistente do trabalho atual do técnico no clube:
formação + jogadores distribuídos nos slots.

Essa configuração permanece para jogos seguintes até ser alterada.

Quando uma partida começa, ela recebe sua própria fotografia da configuração válida
daquele momento. Alterar o TeamSetup depois não altera retroativamente a partida já
iniciada.

O Manager AI prepara o lado CPU usando as mesmas posições e qualidades reconhecidas
pelo domínio, priorizando encaixes naturais/secundários e improvisando apenas quando
necessário.

---

## 6. O motor da partida

O MatchEngine recebe os dois lados preparados e produz a evolução da partida.

### Princípio central

O motor trabalha com probabilidades, não com destino predeterminado.

Um time melhor deve deslocar a distribuição a seu favor, mas ainda pode empatar ou
perder.

### Modelo inicial

O primeiro motor usa:

- qualidades derivadas dos jogadores;
- pesos conscientes da posição;
- tratamento separado do goleiro;
- mando de campo;
- modelo de gols baseado em Poisson;
- sorteio controlado por uma fonte única de aleatoriedade;
- progressão minuto a minuto.

As fórmulas, parâmetros e calibrações fechadas ficam em
`decisoes-jogo-futebol.md` e, depois, na documentação técnica apropriada.

### λ dinâmico

A probabilidade futura muda quando muda uma entrada efetiva da conta.

Uma substituição, por exemplo, altera quem está em campo e pode alterar a força e o
λ dos minutos seguintes. O passado não é recalculado.

Pausa, evento visual ou passagem do tempo não devem modificar o λ por magia.

### Evoluções possíveis

Continuam como bandeiras, sem compromisso de implementação imediata:

- xG/PSxG;
- duelos individuais;
- fases e posicionamento;
- funções táticas;
- contexto de formação;
- atributos mais finos;
- eventos esportivos mais ricos.

---

## 7. A partida interativa

A experiência pretendida é próxima de uma simulação interativa no estilo
Brasfoot/FIFA em modo de simulação, sem campo animado.

O usuário acompanha:

- passagem dos minutos;
- placar;
- eventos;
- estados da partida;
- oportunidades de intervenção quando a mecânica correspondente existir.

### Pausa

O recorte inicial possui pausa manual com limite próprio.

Pausas obrigatórias por lesão grave, expulsão ou outros eventos são uma mecânica
futura e não devem ser antecipadas no primeiro motor.

### Primeira intervenção esportiva

A primeira intervenção real já escolhida para uma etapa posterior do núcleo é a
substituição do técnico humano durante pausa ou intervalo.

A troca altera a composição em campo e, portanto, pode mudar força e probabilidades
futuras.

Mudança tática livre, cansaço, cartão, lesão e decisões esportivas mais profundas da
CPU permanecem futuras.

---

## 8. Tela e comunicação em tempo real

O React não executa a partida.

Fluxo conceitual:

- comandos do usuário → HTTP → Game;
- atualizações da execução → Game → SignalR → React.

Ao voltar para uma partida que continua viva, a tela deve primeiro recuperar uma
fotografia atual no Game e depois acompanhar as mudanças seguintes.

SignalR é canal de comunicação, não motor nem persistência.

---

## 9. Persistência e lifecycle — princípio de visão

A identidade histórica de uma execução existe desde o início da partida.

Ao mesmo tempo, o estado vivo minuto a minuto continua em memória.

Isso permite duas regras coexistirem:

1. existe histórico de que aquela execução nasceu;
2. não existe checkpoint/resume da simulação viva.

Se o Game morrer, a execução em memória não é reconstruída. O registro histórico
pode ser encerrado como não concluído e uma nova tentativa será outra execução.

Detalhes de status, dados persistidos e transações pertencem ao documento de
decisões e ao `sistema-dia-1.md`.

---

## 10. Mensageria e fronteiras

A justificativa antiga de manter um adversário em processo separado foi descartada.

A regra atual é mais forte:

> mensageria só existe quando uma responsabilidade real cruza processos.

Primeiro fluxo fechado:

`Game → PartidaEncerrada → Manager Inbox`

O fato de uma partida ter terminado pode futuramente ganhar outros consumidores,
como Competition, notificações ou estatísticas, mas eles só devem nascer quando
existir trabalho real para eles.

Outbox, idempotência, retry e fila de erro continuam conceitos importantes, porém
cada um entra no ponto em que resolve uma falha concreta do fluxo.

---

## 11. Fundamentação — ciência, calibração e heurística

Manter honestidade sobre a origem de cada regra.

### Base mais forte

- Poisson/Dixon-Coles para modelagem de gols;
- xG em motores futuros;
- estudos de mando;
- curvas de idade;
- métricas de goleiro e outros dados quando a mecânica exigir.

### Heurística / game design

- pesos exatos por posição;
- percentual exato do goleiro;
- futuras penalizações por improvisação;
- psicologia, moral e personalidade;
- modificadores táticos ainda não calibrados.

Mesmo uma fórmula com boa base estatística precisa de calibração para produzir um
jogo plausível.

---

## 12. O que continua fora do recorte atual

Manter como bandeiras, sem redesenhar nesta fase:

- Competition/campeonato;
- vários clubes jogando entre si;
- calendário, rodadas e classificação;
- lesões, cartões, expulsões e pênaltis;
- pausas obrigatórias;
- cansaço;
- CPU mais inteligente durante a partida;
- mudança tática/formação dinâmica;
- instruções condicionais;
- autenticação/autorização;
- multiplayer entre humanos;
- contratos de jogadores e técnicos;
- diretoria, torcida e finanças profundas;
- evolução de jogador;
- dados reais externos;
- motor mais detalhado;
- calibração séria com liga-alvo real;
- cloud/deploy.

---

## 13. O que foi explicitamente superado

Não voltar a tratar como arquitetura atual:

- Opponent Service obrigatório no início;
- robô separado para justificar RabbitMQ;
- `SolicitarEscalacao` / `EnviarEscalacao` pelo broker no primeiro skeleton;
- saga do cumprimento;
- `PartidaCriada` como primeiro Publish;
- IA fake/fixa;
- clube com somente 11 jogadores;
- escolher os 11 melhores gerais antes de encaixar posição;
- proibir qualquer improvisação;
- polling como solução principal da partida ao vivo;
- súmula completa dentro de `PartidaEncerrada`;
- Seed/EngineVersion como conteúdo do evento de integração;
- partida em andamento sem qualquer registro histórico no banco;
- interpretação antiga de "assíncrono" como xadrez por carta e fundamento do
  produto inicial.

O histórico dessas decisões e o que as substituiu ficam em
`decisoes-jogo-futebol.md`.

---

## 14. Alvo deste briefing

Este documento cumpriu seu papel quando um desenvolvedor consegue entender, sem
abrir a especificação inteira:

- que produto estamos construindo;
- onde mora a simulação;
- qual é a relação entre Game, Manager AI, Inbox e Competition futura;
- por que React, SignalR e RabbitMQ têm papéis diferentes;
- que o Player já importa para o motor;
- que a partida é progressiva e pertence ao servidor;
- que o projeto cresce por fatias reais;
- e quais áreas estão deliberadamente adiadas.

Os detalhes fechados de matemática, contratos, persistência e regras operacionais
ficam no documento de decisões e, em forma consolidada de sistema/implementação, no
`sistema-dia-1.md` e no `roadmap-jogo-futebol.md`.

---

## Glossário rápido

- **MatchEngine:** componente interno do Game que contém a matemática da partida.
- **Manager AI:** componente interno do Game que toma decisões de técnico para CPU.
- **SignalR:** canal usado pelo servidor para empurrar atualizações da partida ao React.
- **RabbitMQ:** broker usado quando uma mensagem precisa cruzar processos reais.
- **Evento de integração:** fato que cruza a fronteira entre processos.
- **Outbox:** padrão para manter persistência e intenção de publicar consistentes.
- **Idempotência:** garantir que uma entrega repetida não duplique o efeito.
- **Poisson / xG:** modelos estatísticos ligados à geração/qualidade de gols e chances.
- **Seed:** valor técnico usado para inicializar a sequência aleatória de uma execução.
- **Walking skeleton:** primeira fatia pequena e real que atravessa o fluxo necessário.
- **ADR:** registro curto do motivo de uma decisão arquitetural significativa.
