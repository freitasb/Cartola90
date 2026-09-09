# Handoff — Proposta de melhoria Manager
## Próximo chat: início do Manual de Implementação

**Data de fechamento deste chat:** 07/09/2026

---

# 1. Onde o projeto está

A fase documental de produto/arquitetura está fechada e o **Mapa Mestre de Implementação** também foi consolidado.

Estado ao migrar:

```text
5 documentos oficiais ........ ✅ Fechados
Mapa Mestre ................... ✅ Fechado em 07/09/2026
Manual de Implementação ....... ⬜ Ainda não iniciado
Implementação oficial ......... ⬜ Ainda não iniciada
```

O código/scaffolding criado experimentalmente em um momento anterior foi **suspenso** e não é baseline oficial. Não deve ser usado para justificar decisões retroativamente.

---

# 2. Arquivos que o próximo chat deve receber

## Cinco fontes oficiais

1. `briefing-jogo-futebol_FINAL.md`
2. `decisoes-jogo-futebol_FINAL.md`
3. `mapa-jogador_FINAL.md`
4. `sistema-dia-1_FINAL.md`
5. `roadmap-jogo-futebol_FINAL.md`

## Documento de navegação

6. `mapa-mestre-implementacao.md`

## Este handoff

7. `handoff-proximo-chat-manual-implementacao.md`

O roadmap entregue aqui já contém a correção de ordem do Marco 1 feita em 07/09/2026: posições/formações e `ManagerProfile` vêm antes dos cadastros React porque a UI já depende de `NaturalPosition`.

---

# 3. Hierarquia documental

```text
5 documentos oficiais
        >
Mapa Mestre
        >
Manual de Implementação
        >
código existente
```

Se houver conflito, a fonte superior vence e o documento/código derivado deve ser corrigido.

O Mapa Mestre é guia de construção, não segunda fonte normativa. Números, contratos e regras detalhadas devem preferencialmente apontar para os documentos oficiais em vez de serem copiados.

---

# 4. Objetivo do próximo chat

Criar o **Manual de Implementação** no nível necessário para que o usuário consiga executar sozinho, acompanhar o projeto e saber:

- o que fazer;
- por que fazer;
- onde fazer;
- em que ordem;
- como verificar manualmente;
- como verificar por testes;
- onde parar;
- o que investigar se o resultado esperado não ocorrer.

O Manual começa pela **Parte A — preparação da oficina**, a partir de A1, e aprofunda apenas o bloco atual.

Não detalhar antecipadamente Marcos 2–6. Eles ficam apenas no nível do Mapa Mestre até virarem o próximo trabalho real.

---

# 5. Forma de trabalho exigida pelo usuário

O usuário quer participar ativamente e manter o mesmo nível de consciência do projeto que o assistente.

Portanto:

1. não sair implementando sozinho;
2. antes de materializar decisão relevante em código, explicar a decisão e seus impactos;
3. acelerar conceitos de Clean Architecture/.NET que o usuário já domina, mas não retirar sua participação;
4. parar apenas em trade-offs reais, decisões novas ou conceitos específicos do Manager;
5. quando uma lacuna não estiver respondida pelos documentos, não inventar silenciosamente;
6. o usuário autoriza explicitamente quando a implementação oficial começar.

O usuário não quer governança digital complexa nem automação de status. O Mapa/Manual existem principalmente para **consciência humana, navegação e execução manual**.

---

# 6. Decisões técnicas já fechadas para a oficina

## Backend

- .NET 8 — escolha explícita do usuário, mesmo com conhecimento de que o suporte oficial termina em novembro de 2026;
- ASP.NET Core Web API;
- Controllers;
- Clean Architecture;
- quatro projetos:
  - `Game.Domain`
  - `Game.Application`
  - `Game.Infrastructure`
  - `Game.Api`

## Banco

- PostgreSQL;
- EF Core 8;
- Npgsql;
- migrations.

## Identidade

- tipo usado pelo domínio/aplicação: `Guid`;
- geração UUIDv7;
- no .NET 8, decisão atual: biblioteca `UUIDNext`;
- não implementar UUIDv7 manualmente.

## Frontend

- React;
- TypeScript;
- Vite;
- pnpm;
- Node 24 LTS.

## Testes

- xUnit;
- FluentAssertions;
- Moq quando fizer sentido;
- Coverlet;
- Testcontainers quando houver infraestrutura real a provar;
- testes arquiteturais quando existir a primeira regra concreta a proteger.

## Execução futura

- recorte inicial aceita uma única Match viva simultaneamente;
- isso não deve ser modelado como lei eterna do domínio ou singleton conceitual.

---

# 7. Estrutura física aprovada

```text
manager-football/
│
├── ManagerFootball.sln
├── README.md
├── global.json
├── docker-compose.yml
│
├── docs/
│   ├── 5 documentos oficiais
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
│   └── projetos nascem quando ganham função real
│
└── .github/workflows/ci.yml
```

Dependências:

```text
Application → Domain
Infrastructure → Application + Domain
Api → Application + Infrastructure
```

Proibidas:

```text
Domain → Application/Infrastructure/Api
Application → Infrastructure/Api
Infrastructure → Api
```

`MatchEngine` e `Manager AI` continuam componentes internos do Game, sem `.csproj` próprio neste recorte.

---

# 8. Ambiente local decidido

Durante desenvolvimento inicial:

```text
Game.Api   → máquina
Game.Web   → máquina
PostgreSQL → Docker
```

A razão é manter debug/hot reload/ciclo de aprendizado simples.

O `docker-compose` cresce conforme a topologia real cresce. Não adicionar RabbitMQ antes do Marco 4.

---

# 9. Ponto inicial exato do Manual

Começar em:

> **Parte A / A1 — Criar e preparar o repositório**

O Mapa Mestre já define A1–A12. O Manual deve transformar A1 em instruções executáveis antes de avançar para A2.

Não é necessário replanejar os seis marcos.

---

# 10. Regras conceituais permanentes já consolidadas

- problema antes de ferramenta;
- uma tecnologia legítima ainda precisa justificar adoção;
- último momento responsável;
- risco conhecido pode ser aceito, risco invisível não;
- infraestrutura não deve redesenhar o core;
- arquitetura deve se defender por compiler/tests/CI quando razoável;
- não fabricar precisão;
- não criar feature fake;
- spike só para incerteza concreta/material;
- roadmap organiza produto, risco organiza atenção;
- ADR apenas para decisões significativas;
- consenso por argumento;
- desafiar premissas quando necessário;
- usuário deve entender por quê, não só o quê.

---

# 11. Correções documentais feitas neste chat

1. O roadmap oficial foi corrigido para remover a dependência invertida React → `NaturalPosition` inexistente.
2. `ManagerProfile` ganhou ponto explícito de nascimento junto do modelo de posições/formações no Marco 1.
3. O Mapa Mestre traz as fórmulas atuais de `QualidadeAtaque`, `QualidadeDefesa` e `QualidadeGoleiro` na parte do Marco 1 em que o usuário precisa conferi-las.
4. Blocos backend do Marco 1 têm conferência manual preferencialmente via Swagger, sem antecipar React só para validar backend.
5. `Finished` foi removido da lista de estados vivos; os estados vivos são `Running`, `Paused`, `Halftime`.
6. A matriz global passou a distinguir `TRABALHO` de `USA`.
7. O Mapa Mestre passou a apontar para regras numéricas/contratos detalhados em vez de copiá-los indiscriminadamente.
8. Foi adicionada hierarquia documental e marcador `VOCÊ ESTÁ AQUI`.

---

# 12. Prompt curto de continuação

> Estamos continuando o projeto **Proposta de melhoria Manager**. Leia os cinco documentos oficiais, o `mapa-mestre-implementacao.md` e este handoff. A documentação oficial e o Mapa Mestre estão fechados; a implementação oficial ainda não começou. Nosso próximo trabalho é criar o **Manual de Implementação**, começando pela Parte A / A1 — Criar e preparar o repositório. O objetivo do Manual é permitir que eu execute sozinho e acompanhe cada decisão, com passos, motivo, localização, verificação e troubleshooting. Não implemente código até eu autorizar explicitamente. Não replaneje os Marcos 2–6; use o Mapa Mestre como navegação e detalhe apenas o bloco atual.
