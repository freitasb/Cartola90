# Manual de Implementação — Cartola90

## Estado atual

```text
Documentação oficial ........ ✅ Fechada
Mapa Mestre .................. ✅ Fechado (emendado em 17/09/2026)
Manual de Implementação ...... 🔄 Em construção
Implementação oficial ........ 🔄 Em andamento — Parte A

Parte A — Preparação da oficina

A1  Criar e preparar o repositório ......... ✅ executado
A2  Criar a solution ....................... ✅ executado
A3  Criar os quatro projetos do backend .... ✅ executado
A4  Configurar referências ................. ⬜ BLOCO ATUAL
A5  Fixar o SDK .NET ....................... ✅ executado (antecipado, ver nota)
A6  Preparar PostgreSQL local .............. ⬜
A7  Ligar EF Core e Npgsql ................. ⬜
A8  Subir a API mínima ..................... ⬜
A9  Criar o frontend real vazio ............ ⬜
A10 Preparar a estratégia de testes ........ ⬜
A11 Preparar Docker Compose ................ ⬜
A12 Preparar CI ............................ ⬜
```

Este Manual é um documento operacional derivado.

Sua função é explicar **como executar** cada bloco definido pelo Mapa Mestre. Ele não cria novas regras de produto e não substitui os documentos oficiais.

Precedência:

```text
5 documentos oficiais
        >
Mapa Mestre
        >
Manual de Implementação
        >
código existente
```

Se o Manual contradisser uma fonte superior, o Manual deve ser corrigido.

---

## Nota de consolidação — 17/09/2026

Os blocos A1, A2, A3 e A5 foram originalmente escritos como quatro arquivos separados, mantidos fora do repositório. Isso contrariava a própria seção 9 do A1, que define o Manual como **um único arquivo versionado em `docs/`**, crescendo bloco a bloco.

Consequência prática da separação: o Manual era o único documento do projeto sem histórico em Git. Os cinco documentos oficiais e o Mapa Mestre tinham rastreabilidade; o documento que registra *como cada passo foi executado* não tinha.

Esta consolidação corrige isso e aplica três correções factuais, todas decorrentes de decisões tomadas em 17/09/2026 e registradas em `decisoes-jogo-futebol_FINAL.md` §25:

1. **Plataforma:** o projeto passa a ser **.NET 10**, não .NET 8. O bloco A5 foi reescrito; o texto anterior existia para impedir que o SDK 10 fosse selecionado, e perdeu função.
2. **Nomes:** repositório e solution são **`Cartola90`** e **`Cartola90.slnx`**, não `manager-football` / `ManagerFootball.sln`.
3. **Formato da solution:** o projeto adota **`.slnx`**. O bloco A2 original protegia o `.sln` por uma razão concreta — o SDK do .NET 8 não lê arquivos `.slnx`. Com .NET 10, essa razão deixou de existir.

Os blocos A1–A3 e A5 aparecem abaixo já corrigidos e marcados como executados. Eles são mantidos por inteiro, e não resumidos, porque descrevem decisões operacionais que precisam continuar consultáveis.

---

# Parte A — Preparação da oficina

# A1 — Criar e preparar o repositório ✅

## 1. Objetivo

Criar a baseline oficial do projeto antes de qualquer implementação.

Ao concluir A1 devem existir:

```text
Cartola90/
│
├── README.md
│
└── docs/
    ├── briefing-jogo-futebol_FINAL.md
    ├── decisoes-jogo-futebol_FINAL.md
    ├── mapa-jogador_FINAL.md
    ├── sistema-dia-1_FINAL.md
    ├── roadmap-jogo-futebol_FINAL.md
    ├── mapa-mestre-implementacao.md
    └── manual-implementacao.md
```

Além disso:

```text
✓ diretório é um repositório Git
✓ existe uma branch principal
✓ existe um commit inicial oficial
✓ documentação está versionada
✓ README está versionado
✓ repositório pode ser clonado novamente
✓ clone contém a mesma baseline documental
```

A1 **não cria aplicação**.

## 2. Por que A1 existe

O objetivo não é simplesmente executar `git init`.

A1 estabelece o ponto a partir do qual o desenvolvimento oficial passa a ter uma história confiável.

Houve scaffolding experimental antes desta baseline. Esse material permanece fora da autoridade da implementação oficial. Código anterior pode continuar preservado separadamente como referência histórica, mas não determina a arquitetura nem entra automaticamente nesta baseline.

## 3. Limite de A1

Durante A1, não criar solution, projetos .NET, `global.json`, PostgreSQL, EF Core, testes, CI, nem qualquer entidade de domínio. Cada peça possui ponto próprio de nascimento:

```text
A2  → Cartola90.slnx
A3  → projetos .NET e .gitignore
A5  → global.json
A6  → PostgreSQL
A9  → Game.Web
A12 → CI
```

A estrutura física aprovada do repositório representa o **destino da Parte A**, não a obrigação de criar tudo no primeiro commit.

## 4. Lacuna operacional registrada — `.gitignore`

Nenhuma fonte superior determina em qual bloco o `.gitignore` deve nascer.

Proposta operacional do Manual, mantida: nascer em **A3**, primeiro momento em que passam a existir artefatos gerados (`bin/`, `obj/`). Quando o frontend nascer em A9, o mesmo arquivo será ampliado para cobrir o ecossistema Node.

Esta é proposta operacional, não decisão vinda dos documentos oficiais.

## 5. Nome do repositório

```text
Cartola90
```

O nome da solution acompanha o do repositório e aparece em A2:

```text
Cartola90.slnx
```

> **Correção 17/09/2026.** A versão anterior deste bloco adotava `manager-football` para o diretório e `ManagerFootball` para a solution. O nome `Cartola90` foi confirmado como definitivo. O `README.md` versionado ainda se intitula "Manager Football" e deve ser ajustado — pendência aberta, registrada na seção de pendências no fim deste Manual.

## 6. Branch principal

As fontes exigem uma branch principal, mas não fixam seu nome.

Adotado: `main`. Convenção atual amplamente utilizada, sem requisito do projeto que justifique outro nome. Não é decisão arquitetural e não exige ADR.

## 7. Criar o diretório local

```bash
mkdir Cartola90
cd Cartola90
```

## 8. Inicializar o Git

```bash
git init -b main
git branch --show-current     # esperado: main
git status                    # ainda não deve existir commit
```

Alternativa se a versão instalada do Git não aceitar `-b`:

```bash
git init
git branch -M main
```

## 9. Criar a pasta de documentação

Criar `docs/` e colocar nela os cinco documentos oficiais, o Mapa Mestre e o Manual.

### Estado diferente dos arquivos

Os seis primeiros documentos estão fechados. O `manual-implementacao.md` é diferente: entra no repositório desde A1, mas está **em construção incremental**. No primeiro commit ele contém apenas o trabalho consolidado até aquele momento, e será ampliado nos commits seguintes.

```text
estar em docs/
≠
estar fechado
```

### Sobre o handoff

O arquivo `handoff-proximo-chat-manual-implementacao.md` é artefato de transição entre conversas. Ele não pertence à estrutura documental permanente aprovada, e seu conteúdo relevante deve estar absorvido pelo Manual ou pelas fontes apropriadas.

> **Situação em 17/09/2026:** o handoff foi marcado como histórico. Ele permanece no repositório como registro, mas não deve mais ser consultado como fonte de decisão técnica.

## 10. Criar o README inicial

O README de A1 deve ser curto e verdadeiro. Ele ainda não deve ensinar a executar a API, subir PostgreSQL, iniciar React, executar migrations ou rodar testes, porque nenhuma dessas capacidades existe.

O README deverá evoluir quando existirem comandos reais que um desenvolvedor possa executar. Não documentar hoje um procedimento futuro como se já funcionasse.

## 11. Identidade Git

Antes do primeiro commit:

```bash
git config user.name
git config user.email
```

Se não retornarem valores válidos, o commit falha. Configurar globalmente (`--global`) ou apenas neste repositório, conforme preferência da máquina. A escolha não pertence à arquitetura do Manager.

## 12. Primeiro commit e remoto

```bash
git add README.md docs
git status
git commit -m "docs: establish project baseline"
```

Criar no GitHub um repositório remoto **vazio**, sem pedir README, `.gitignore` ou licença gerados automaticamente — o README já existe localmente, e criar arquivos remotamente produziria histórico inicial desnecessário antes do primeiro push.

```bash
git remote add origin <URL-DO-REPOSITORIO>
git remote -v
git push -u origin main
```

## 13. Verificação de A1

A conferência principal é feita por **clone limpo**, fora do diretório original:

```bash
git clone <URL-DO-REPOSITORIO>
cd Cartola90
git status                 # esperado: working tree clean
git branch --show-current  # esperado: main
git log --oneline
```

Confirmar a presença de `README.md`, de `docs/` e, dentro dele, dos cinco oficiais, do Mapa Mestre e do Manual.

**A1 não possui teste automatizado.** Isso é deliberado: ainda não existem solution, projetos, CI ou aplicação. Criar um teste artificial apenas para afirmar que arquivos existem contrariaria a regra de não introduzir peças sem função real. A prova de A1 é operacional — commit, push, clone limpo, estado Git correto.

## 14. Checklist de aceitação

```text
[x] repositório local nasceu do zero
[x] branch principal é main
[x] README existe
[x] cinco documentos oficiais estão em docs/
[x] Mapa Mestre está em docs/
[x] Manual está em docs/ na versão consolidada          ← fechado em 17/09/2026
[x] está explícito que o Manual continua em construção
[x] primeiro commit oficial existe
[x] identidade Git permitiu criar o commit
[x] remoto aponta para o repositório correto
[x] main foi enviada ao remoto
[x] clone em diretório limpo funciona
[x] clone contém README e documentação esperada
[x] git status do clone está limpo
[x] ausência de teste automatizado em A1 foi conscientemente aceita
```

O `.gitignore` não faz parte deste checklist, porque sua posição não é normativa. Ele nasce em A3.

## 15. Troubleshooting

**`git` não é reconhecido.** Verificar `git --version`. Se não existir, corrigir a instalação antes de prosseguir.

**`git init -b main` não funciona.** Instalação antiga. Preferencialmente atualizar o Git; para não travar, usar `git init` seguido de `git branch -M main` e conferir com `git branch --show-current`.

**O commit falha pedindo identidade** (`Author identity unknown`). Configurar `user.name` e `user.email` conforme a seção 11 e repetir.

**O remoto já possui um primeiro commit.** Não usar `push --force` como reação automática. Confirmar antes se o repositório remoto correto foi criado, quem criou o commit e quais arquivos existem nele. O fluxo normal de A1 pressupõe remoto vazio.

**Apareceu código ou artefato inesperado antes do commit.** Parar antes do `git add`. A1 deve conter apenas README e documentação.

**Algum documento está faltando.** Não avançar para A2. A baseline documental é o próprio produto de A1.

## 16. Resultado de A1

```text
Cartola90/
├── README.md
└── docs/
```

Existe uma baseline Git reproduzível e documentalmente correta. Ainda não existe software executável — e isso é exatamente o resultado esperado.

---

# A2 — Criar a solution ✅

## 1. Objetivo

Fazer nascer a solution .NET oficial do Game:

```text
Cartola90.slnx
```

Ao terminar A2, a raiz passa a ter:

```text
Cartola90/
│
├── Cartola90.slnx
├── README.md
└── docs/
```

A solution deve existir na **raiz do repositório** e ser reconhecida pela CLI .NET. Ainda não existem projetos dentro dela.

## 2. Por que a solution nasce antes dos projetos

A solution é o contêiner de organização dos projetos que nascerão em A3. Separar os dois passos permite confirmar, em ordem:

```text
CLI .NET disponível
        ↓
solution válida
        ↓
projetos reais
```

sem misturar problemas de criação da solution com problemas de templates, referências ou compilação.

## 3. O que uma solution é — e o que ela não é

Ela organiza projetos .NET e permite operações sobre o conjunto (build, test, gerenciamento dos projetos). Mas a solution:

- não é projeto compilável por si própria;
- não contém regra de domínio;
- não implementa Clean Architecture;
- não define as dependências entre Domain, Application, Infrastructure e Api;
- não escolhe o Target Framework dos projetos.

```text
Solution
    ≠
arquitetura
```

As fronteiras arquiteturais só se materializam quando os `.csproj` existem, e as dependências entre eles só nascem em **A4**.

## 4. Pré-condição

A1 concluído, branch `main`, estado Git conhecido. A2 não deve ser usado para corrigir silenciosamente uma execução incompleta de A1.

## 5. Verificar a CLI .NET

```bash
dotnet --version
dotnet --list-sdks
```

O primeiro informa **qual SDK a CLI está selecionando agora**; o segundo lista tudo que está instalado. São perguntas diferentes.

O ambiente do projeto exige um SDK **10.0.x** instalado. Ter outros SDKs instalados na mesma máquina não é problema.

### 5.1 `dotnet --version` e A5 não são a mesma coisa

Neste momento ainda não existe `global.json`, e isso é intencional. Ele nasce em **A5** e é ele que fixa formalmente qual SDK a CLI seleciona dentro do repositório.

Em A2 perguntamos apenas: *a máquina possui o ambiente necessário para continuar?* Ainda não dizemos: *este repositório está formalmente fixado neste SDK.*

## 6. Formato do arquivo — `.slnx`

> **Decisão revista em 17/09/2026.** A versão anterior deste bloco exigia `.sln` e trazia troubleshooting instruindo a apagar qualquer `.slnx` gerado por engano. A razão era concreta: **o SDK do .NET 8 não consegue abrir arquivos `.slnx`**, e a stack era .NET 8.
>
> Com o projeto em .NET 10, essa razão deixou de existir. O `.slnx` é o formato padrão do SDK 10, é XML legível, e resolve um problema real do formato antigo: a `.sln` clássica guarda GUIDs por projeto e blocos de aninhamento que produzem conflitos de merge difíceis de resolver.
>
> **Requisito derivado, para não esquecer:** o runner do GitHub Actions em **A12** precisará de um SDK 10, senão `dotnet build` não reconhecerá o arquivo. Isso está anotado na seção de pendências no fim deste Manual.

A partir do SDK 10, `dotnet new sln` gera `.slnx` por padrão. O formato antigo continua disponível via `--format sln`, e não é o que este projeto usa.

## 7. Criar a solution

Na raiz do repositório:

```bash
dotnet new sln --name Cartola90
```

Resultado esperado:

```text
Cartola90.slnx
```

## 8. Conferir fisicamente

A raiz deve conter:

```text
Cartola90/
│
├── Cartola90.slnx
├── README.md
└── docs/
```

Ainda não devem existir `src/Game.Domain/`, `src/Game.Application/`, `src/Game.Infrastructure/` nem `src/Game.Api/`. Eles pertencem a A3.

## 9. Fazer a CLI reconhecer a solution

```bash
dotnet sln Cartola90.slnx list
```

A CLI deve abrir a solution sem erro e indicar que ainda não existem projetos cadastrados.

```text
0 projetos
```

não significa falha. Significa que A2 criou o contêiner e A3 ainda não aconteceu.

## 10. O que não fazer para "provar" a solution

Não criar projeto temporário para adicionar e remover depois. Nada de `FakeProject`, `HelloWorld`, `Dummy` ou `TesteSolution`.

A solution vazia já pode ser validada pela CLI. Criar um `.csproj` descartável violaria a regra do projeto:

> não criar feature ou estrutura fake apenas para provar infraestrutura.

## 11. Verificação manual

1. **Estrutura física:** `Cartola90.slnx` existe e está na raiz — não em `src/` nem em `docs/`.
2. **Reconhecimento pela CLI:** `dotnet sln Cartola90.slnx list` roda sem erro de parsing e retorna zero projetos.

Essa é a conferência definida pelo Mapa Mestre: *a CLI .NET reconhece a solution sem erro.*

## 12. Verificação automatizada

**Não existe teste automatizado em A2**, e isso é esperado. Ainda não existem projetos de produção, projetos de teste, regra de domínio ou CI.

A conferência necessária é uma validação da própria ferramenta — verificação operacional, não teste de produto. Não criar projeto xUnit para verificar que a solution existe. A estratégia oficial permanece: projetos de teste só nascem quando ganham responsabilidade real.

## 13. Versionamento

As fontes não determinam um commit obrigatório por subetapa. A1 teve commit próprio por uma razão específica: sua responsabilidade era estabelecer a baseline Git oficial. Isso não vira silenciosamente "cada item A deve gerar um commit".

A granularidade dos commits é escolha operacional, não critério de aprovação do bloco.

## 14. Troubleshooting

**`dotnet` não é reconhecido.** Executar `dotnet --info`. Se nem esse comando funcionar, a CLI não está disponível no ambiente. Não prosseguir.

**Não existe SDK 10 instalado.** Instalar antes de avançar. A escolha da versão exata que será fixada no repositório pertence a A5.

**A solution nasceu com outro nome** (por exemplo, derivado da pasta). Se o arquivo acabou de nascer e está vazio, remover e recriar com `dotnet new sln --name Cartola90`.

**A solution foi criada dentro de `src/`.** Corrigir a localização antes de A3. A estrutura aprovada coloca a solution na raiz.

**`dotnet sln ... list` informa que não há projetos.** Esperado. Não corrigir um estado correto adicionando projeto fictício.

**`dotnet sln ... list` diz que o arquivo não existe.** Conferir diretório atual, nome e extensão do arquivo.

## 15. ADRs em A2

Nenhum ADR novo é necessário. Não criar ADR para o uso de uma solution, para o nome `Cartola90`, para o comando utilizado ou para a diferença operacional entre gerações da CLI. A existência e o nome da solution vêm do desenho aprovado; a adaptação do comando é compatibilidade operacional, não arquitetura do produto.

A escolha `.slnx` sobre `.sln` também não é ADR — é consequência direta da decisão de plataforma, que **essa sim** está registrada em `decisoes-jogo-futebol_FINAL.md` §25.

## 16. Checklist de aceitação

```text
[x] A1 estava em estado válido antes de começar
[x] CLI dotnet funciona
[x] SDK .NET 10 instalado
[x] foi identificada a versão atualmente selecionada da CLI
[x] Cartola90.slnx existe
[x] Cartola90.slnx está na raiz do repositório
[x] a CLI reconhece Cartola90.slnx sem erro
[x] a solution ainda possuía zero projetos ao fim do bloco
[x] nenhum projeto fake foi criado para testá-la
[x] ausência de testes automatizados em A2 foi conscientemente aceita
```

## 17. Linha de parada

Quando `dotnet sln Cartola90.slnx list` reconhecer a solution vazia:

```text
A2 — Criar a solution ................. ✅
```

PARAR. Não executar ainda `dotnet new classlib`, `dotnet new webapi` ou `dotnet sln add` — pertencem ao próximo bloco.

> **Nota de ordem de execução.** Neste projeto, A5 foi executado entre A2 e A3, deliberadamente. A justificativa está no próprio A5.

---

# A5 — Fixar o SDK .NET ✅ *(executado entre A2 e A3)*

> **Correção de ordem.** Este bloco permanece identificado como A5 no Mapa Mestre, mas foi executado imediatamente após A2 e antes de A3.
>
> Motivo: os primeiros projetos .NET não devem ser gerados enquanto o repositório não controla qual SDK executa a CLI.

> **Reescrita em 17/09/2026.** A versão anterior deste bloco existia para garantir que `dotnet --version` resolvesse `8.0.x`, impedindo que um SDK 10 instalado na máquina fosse selecionado silenciosamente. Com .NET 10 adotado, esse objetivo se inverteu, e o bloco ficou mais curto.

## 1. Objetivo

Criar na raiz:

```text
global.json
```

e garantir que, dentro do repositório, `dotnet --version` resolva um SDK `10.0.x`.

A partir desse momento, `dotnet new`, `dotnet restore` e `dotnet build` deixam de depender silenciosamente de qualquer SDK que esteja instalado na máquina.

## 2. Por que este bloco precisa acontecer antes de A3

Há dois conceitos diferentes, e essa distinção vale para sempre:

```text
SDK selecionado pela CLI     ≠     Target Framework do projeto
```

Uma máquina pode ter vários SDKs instalados. Sem `global.json`, a CLI escolhe o mais novo. Informar `--framework net10.0` na criação de um projeto garante o **alvo** daquele projeto, mas não controla **qual ferramenta** o criou e o compila.

Como A3 é o primeiro momento em que templates geram `.csproj` e código real, a fixação precisa existir antes dele.

## 3. Pré-condições

```text
A1 ........ ✅
A2 ........ ✅
```

Devem existir `Cartola90.slnx`, `README.md` e `docs/`. Ainda **não** devem existir os quatro projetos de backend.

## 4. Escolher a versão

```bash
dotnet --list-sdks
```

Procurar uma versão estável `10.0.x`. A regra operacional é escolher a **versão estável 10.0.x mais recente instalada naquele momento**, nunca preview ou release candidate.

Não copiar números deste Manual. A versão exata é escolhida na máquina, no momento da execução.

## 5. Política de roll-forward

Além da versão base, é preciso definir o que acontece quando outra máquina não possui exatamente aquele patch.

Adotado:

```text
rollForward = latestFeature
```

Isso permite usar uma feature band ou patch posterior **dentro do 10.0**, sem avançar silenciosamente para uma geração seguinte. O equilíbrio buscado é:

```text
reprodutibilidade
+
não exigir o mesmo patch exato eternamente
+
permanecer dentro do .NET 10
```

Isso é configuração operacional, não ADR de arquitetura.

## 6. Criar o `global.json`

Na raiz, substituindo pela versão real escolhida:

```bash
dotnet new globaljson --sdk-version <SDK_10_ESCOLHIDO> --roll-forward latestFeature
```

`version` exige uma versão completa e existente. Não preencher manualmente com um número aproximado.

## 7. Resultado esperado

```text
Cartola90/
│
├── Cartola90.slnx
├── global.json
├── README.md
└── docs/
```

Estrutura conceitual do arquivo:

```json
{
  "sdk": {
    "version": "<SDK_10_ESCOLHIDO>",
    "rollForward": "latestFeature"
  }
}
```

**Estado atual no repositório:** `10.0.401` com `rollForward: latestFeature`.

## 8. Verificação principal

Na raiz:

```bash
dotnet --version
```

O resultado deve começar por `10.0.`.

### 8.1 Confirmar de dentro de uma subpasta

Entrar em `docs/` e executar `dotnet --version` novamente. O resultado deve continuar sendo um SDK 10 compatível.

Isso comprova que o resolvedor encontrou o `global.json` subindo pela hierarquia do repositório — que é o comportamento que A5 precisa provar. Depois, voltar à raiz.

## 9. O que não fazer

Não criar ainda `Game.Domain`, `Game.Application`, `Game.Infrastructure` ou `Game.Api`. Não executar ainda `dotnet new classlib` nem `dotnet new webapi`. Eles pertencem a A3.

A função deste bloco é exclusivamente garantir que, quando A3 começar, esses comandos já rodem sob uma versão controlada da CLI.

## 10. Relação entre SDK e Target Framework

A partir daqui existem duas proteções complementares, e vale fixar a diferença:

| | Controlado por | Responde |
|---|---|---|
| **SDK** | `global.json` | qual geração da ferramenta `dotnet` opera este repositório? |
| **Target Framework** | `.csproj` | para qual framework este projeto é compilado? |

Precisamos das duas. Uma controla a ferramenta, a outra controla o alvo.

## 11. Verificação automatizada

**Não existe teste automatizado de produto em A5**, e isso continua esperado. Não há domínio, aplicação nem projetos de teste.

A verificação executável deste bloco é a própria resolução da CLI: `dotnet --version`. Ela prova uma configuração de ambiente, não uma regra de negócio. **Não criar teste xUnit para `global.json`.**

Posteriormente o CI também deverá usar uma versão compatível do SDK, mas essa integração pertence a **A12**.

## 12. Troubleshooting

**`dotnet --version` continua mostrando outra geração.** Confirmar que o `global.json` está na raiz do repositório e que o comando está sendo executado de dentro dele. O resolvedor sobe a hierarquia de diretórios procurando o arquivo.

**A versão fixada não existe na máquina.** `dotnet --version` falha com mensagem indicando que o SDK não foi encontrado. Conferir com `dotnet --list-sdks` e corrigir o número no `global.json` para uma versão realmente instalada.

**Outra máquina não tem exatamente esse patch.** É para isso que existe `rollForward: latestFeature`. Se ainda assim falhar, a outra máquina provavelmente não tem nenhum SDK 10.0 — instalar.

## 13. Checklist de aceitação

```text
[x] A1 e A2 estavam válidos antes de começar
[x] dotnet --list-sdks mostra um SDK 10 instalado
[x] global.json existe na raiz
[x] version contém uma versão completa e real
[x] rollForward está definido como latestFeature
[x] dotnet --version dentro do repositório resolve 10.0.x
[x] o mesmo vale a partir de uma subpasta
[x] nenhum projeto .NET foi criado neste bloco
[x] ausência de teste automatizado em A5 foi conscientemente aceita
```

---

# A3 — Criar os quatro projetos do backend ✅

## 1. Objetivo

Criar a estrutura inicial do backend:

```text
src/
├── Game.Domain/
├── Game.Application/
├── Game.Infrastructure/
└── Game.Api/
```

Todos com:

```text
.NET 10
Target Framework: net10.0
```

Ao final, os quatro projetos devem estar registrados em `Cartola90.slnx` e a solution deve compilar.

> **Correção 17/09/2026.** A versão anterior deste bloco usava `Cartola90.slnx` no passo de adicionar projetos, mas `ManagerFootball.sln` no passo de compilar — dois nomes de solution no mesmo documento. Corrigido para `Cartola90.slnx` em todos os passos.

## 2. Criar a pasta `src`

Na raiz:

```bash
mkdir src
```

## 3. Criar o `.gitignore`

Agora começam a surgir artefatos como `bin/` e `obj/`, então o `.gitignore` passa a ter função concreta — é o ponto de nascimento proposto em A1.

O SDK fornece um template oficial:

```bash
dotnet new gitignore
```

**Estado atual no repositório:** o `.gitignore` em uso não é o template puro. Ele cobre .NET, Visual Studio, Rider, VS Code, Node/React/Vite, arquivos de ambiente, logs, saída de testes e cobertura, pacotes NuGet, arquivos de sistema operacional e temporários. A cobertura antecipada do ecossistema Node é intencional e será exercida em A9.

## 4. Criar os três projetos de biblioteca

```bash
dotnet new classlib -n Game.Domain         -o src/Game.Domain         -f net10.0
dotnet new classlib -n Game.Application    -o src/Game.Application    -f net10.0
dotnet new classlib -n Game.Infrastructure -o src/Game.Infrastructure -f net10.0
```

Remover o arquivo inicial do template em cada um:

```text
src/Game.Domain/Class1.cs
src/Game.Application/Class1.cs
src/Game.Infrastructure/Class1.cs
```

Resultado: cada pasta contém apenas o seu `.csproj`.

## 5. Criar `Game.Api`

Usar explicitamente o template baseado em Controllers:

```bash
dotnet new webapi -n Game.Api -o src/Game.Api -f net10.0 --use-controllers --no-openapi
```

`--use-controllers` determina que o projeto use Controllers em vez de Minimal APIs — decisão já fechada pelo Mapa Mestre.

Remover os exemplos gerados pelo template:

```text
src/Game.Api/WeatherForecast.cs
src/Game.Api/Controllers/WeatherForecastController.cs
```

O projeto fica com a estrutura ASP.NET Core, mas sem endpoint de demonstração.

> **Nota sobre `Game.Api.http`.** O template gera também um arquivo `.http`, usado para disparar requisições direto da IDE. Ele foi mantido. Hoje não tem função, porque não existe endpoint, mas passará a ter uso real em **A8**. Não é feature fake: é ferramenta de desenvolvimento, não parte do produto.

> **Nota sobre `Program.cs`.** O template gera `app.UseAuthorization()`. Não existe autenticação no projeto e ela está explicitamente fora do recorte inicial. Essa linha é ruído de template e será removida em **A8**, quando o pipeline da API for montado de forma deliberada.

## 6. Conferir os Target Frameworks

Nos quatro `.csproj`, confirmar:

```xml
<TargetFramework>net10.0</TargetFramework>
```

## 7. Adicionar os projetos à solution

Na raiz:

```bash
dotnet sln Cartola90.slnx add src/Game.Domain/Game.Domain.csproj
dotnet sln Cartola90.slnx add src/Game.Application/Game.Application.csproj
dotnet sln Cartola90.slnx add src/Game.Infrastructure/Game.Infrastructure.csproj
dotnet sln Cartola90.slnx add src/Game.Api/Game.Api.csproj
```

## 8. Conferir a solution

```bash
dotnet sln Cartola90.slnx list
```

Devem aparecer os quatro projetos.

## 9. Compilar

```bash
dotnet build Cartola90.slnx
```

Resultado esperado:

```text
Build succeeded.
0 Error(s)
```

Neste ponto **ainda não existem referências entre os quatro projetos**. Elas entram em A4. Uma build bem-sucedida aqui prova que os quatro projetos são válidos individualmente, não que a arquitetura está montada.

## 10. Conferir o Git

```bash
git status
```

Os diretórios gerados pelo build (`bin/`, `obj/`) **não** devem aparecer entre os arquivos a versionar. Se aparecerem, o `.gitignore` está incorreto ou não está na raiz — corrigir antes de commitar, porque remover arquivo do histórico depois é bem mais trabalhoso do que não adicioná-lo.

## 11. Checklist de aceitação

```text
[x] quatro projetos net10.0
[x] API criada com Controllers
[x] Class1.cs removidos
[x] exemplos WeatherForecast removidos
[x] projetos registrados na solution
[x] solution compila
[x] .gitignore ativo e bin/obj fora do git status
```

## 12. Resultado de A3

```text
Cartola90/
│
├── Cartola90.slnx
├── global.json
├── .gitignore
├── README.md
│
├── docs/
│
└── src/
    ├── Game.Domain/
    │   └── Game.Domain.csproj
    ├── Game.Application/
    │   └── Game.Application.csproj
    ├── Game.Infrastructure/
    │   └── Game.Infrastructure.csproj
    └── Game.Api/
        ├── Game.Api.csproj
        ├── Game.Api.http
        ├── Program.cs
        ├── appsettings.json
        ├── appsettings.Development.json
        ├── Controllers/
        └── Properties/launchSettings.json
```

## 13. Linha de parada

```text
A3 — Criar os quatro projetos do backend .... ✅
```

PARAR. O próximo bloco é A4.

---

# A4 — Configurar referências entre projetos ⬜ BLOCO ATUAL

## 1. O problema que este bloco resolve

Hoje, no repositório, existe uma frase e não existe um fato.

A frase está no Mapa Mestre §6: `Game.Domain` não pode depender de `Game.Application`, de `Game.Infrastructure` nem de `Game.Api`. É a regra central da Clean Architecture neste projeto — a regra de futebol não depende de banco, de HTTP nem de broker.

O fato não existe porque os quatro projetos estão **soltos**. Nenhum referencia nenhum. Se alguém escrever hoje, dentro do `Game.Domain`, uma linha que use `Game.Infrastructure`, nada acontece de especial: o código simplesmente não compila porque o tipo não é encontrado — pelo motivo errado, por acidente, e não porque a arquitetura foi defendida.

A4 é o bloco que transforma a frase em estrutura. Depois dele, a dependência permitida existe porque foi declarada, e a proibida é impossível porque não foi.

## 2. Objetivo

Aplicar exatamente as dependências aprovadas pelo Mapa Mestre §6:

```text
Game.Application    → Game.Domain

Game.Infrastructure → Game.Application
Game.Infrastructure → Game.Domain

Game.Api            → Game.Application
Game.Api            → Game.Infrastructure
```

E **não** criar nenhuma outra. Continuam proibidas:

```text
Game.Domain         → Application / Infrastructure / Api
Game.Application    → Infrastructure / Api
Game.Infrastructure → Api
```

Repare no desenho: `Game.Domain` não recebe nenhuma referência. Ele é o único projeto que não conhece ninguém. Isso não é coincidência nem economia — é a definição de domínio independente.

## 3. Pré-condições

```text
A1 ✅   A2 ✅   A3 ✅   A5 ✅
```

Confirmar antes de começar:

```bash
cd "C:\Tapagoh!\Cartola90"
git status
dotnet build Cartola90.slnx
```

O build deve passar com 0 erros. Se não passar, o problema é anterior a A4 e precisa ser resolvido primeiro.

## 4. Executar

Na raiz do repositório, um comando por linha:

```bash
dotnet add src/Game.Application/Game.Application.csproj reference src/Game.Domain/Game.Domain.csproj

dotnet add src/Game.Infrastructure/Game.Infrastructure.csproj reference src/Game.Application/Game.Application.csproj
dotnet add src/Game.Infrastructure/Game.Infrastructure.csproj reference src/Game.Domain/Game.Domain.csproj

dotnet add src/Game.Api/Game.Api.csproj reference src/Game.Application/Game.Application.csproj
dotnet add src/Game.Api/Game.Api.csproj reference src/Game.Infrastructure/Game.Infrastructure.csproj
```

Leia o comando assim: `dotnet add <quem depende> reference <de quem ele depende>`. A ordem importa e inverter os dois caminhos cria exatamente a dependência proibida.

### 4.1 Por que declarar referências que já viriam sozinhas

`Game.Infrastructure` referencia `Game.Application`, que por sua vez referencia `Game.Domain`. Na prática, o `Game.Domain` já chegaria ao `Game.Infrastructure` por **transitividade** — o .NET propaga referências de projeto automaticamente. O mesmo vale para `Game.Api → Game.Application`, que já viria por meio de `Game.Infrastructure`.

Então por que declarar as duas explicitamente, como manda o Mapa Mestre?

Por duas razões práticas:

1. **Intenção visível.** Abrir o `.csproj` e ler que `Infrastructure` depende de `Domain` é diferente de descobrir isso deduzindo uma cadeia. O arquivo passa a dizer o que o projeto quer, não apenas o que ele herda.
2. **Robustez a mudanças futuras.** Se um dia `Application` deixar de referenciar `Domain`, quem dependia só por transitividade quebra de repente, longe da causa. A referência explícita sobrevive a essa mudança.

O custo é zero: declarar uma referência que já existiria não duplica nada no build.

## 5. Conferência manual

### 5.1 Ler os arquivos

Abrir os quatro `.csproj` e confirmar:

| Projeto | Deve conter `ProjectReference` para |
|---|---|
| `Game.Domain.csproj` | **nenhum** |
| `Game.Application.csproj` | Game.Domain |
| `Game.Infrastructure.csproj` | Game.Application, Game.Domain |
| `Game.Api.csproj` | Game.Application, Game.Infrastructure |

O `Game.Domain.csproj` continuar sem nenhuma referência **é o resultado esperado**, não um passo esquecido.

O trecho gerado tem esta forma:

```xml
<ItemGroup>
  <ProjectReference Include="..\Game.Domain\Game.Domain.csproj" />
</ItemGroup>
```

### 5.2 Compilar

```bash
dotnet build Cartola90.slnx
```

Esperado: `Build succeeded. 0 Error(s)`.

### 5.3 Conferir o Git

```bash
git status
```

Devem aparecer como modificados apenas três arquivos:

```text
src/Game.Application/Game.Application.csproj
src/Game.Infrastructure/Game.Infrastructure.csproj
src/Game.Api/Game.Api.csproj
```

`Game.Domain.csproj` **não** deve aparecer. Se aparecer, algo foi referenciado dentro dele e precisa ser desfeito.

## 6. Experimento opcional de diagnóstico

Este passo é opcional e serve para você **ver** a proteção funcionando, em vez de apenas ler sobre ela.

Criar temporariamente, dentro de `src/Game.Domain/`, um arquivo `Teste.cs`:

```csharp
// Arquivo descartável — apenas para observar o erro de compilação.
namespace Game.Domain;

internal sealed class Teste
{
    // Game.Infrastructure não é alcançável a partir do Domain.
    private readonly Game.Infrastructure.Class1? _proibido;
}
```

Rodar `dotnet build Cartola90.slnx` e observar o erro: o compilador não encontra o namespace, porque não existe caminho de referência do Domain para a Infrastructure.

**Apagar o arquivo em seguida e não commitá-lo.**

Isso não viola a regra de "não criar peça fake": nada está sendo criado que finja ser funcionalidade, nada entra no histórico e nada permanece. É um experimento de diagnóstico, do mesmo tipo que desligar um cabo para confirmar qual luz apaga.

## 7. Verificação automatizada — e o limite honesto deste bloco

**Não existe teste automatizado em A4.**

Mas o motivo aqui é diferente dos blocos anteriores, e vale entender bem, porque é a primeira vez no projeto em que essa distinção aparece de verdade.

O compilador passa a proteger a arquitetura **por ausência**: o Domain não alcança a Infrastructure porque a referência não existe. Essa proteção é real e é forte.

Só que ela tem um buraco preciso: **o compilador não impede alguém de adicionar a referência errada.** Se amanhã alguém rodar

```bash
dotnet add src/Game.Domain/Game.Domain.csproj reference src/Game.Infrastructure/Game.Infrastructure.csproj
```

o build passa. A regra do Mapa Mestre foi violada e nada reclama.

Fechar esse buraco é papel do **teste arquitetural**, e o Mapa Mestre diz que ele nasce "quando houver a primeira regra de dependência que queremos proteger executavelmente". Essa regra acabou de nascer, aqui em A4. O teste que a protege pertence a **A10**.

Resumindo a divisão de trabalho:

```text
A4  → a dependência proibida não existe
A10 → a dependência proibida não pode voltar a existir
```

Não antecipar o teste agora. A ordem do Mapa Mestre é deliberada, e A10 tem seu próprio bloco.

## 8. Troubleshooting

**Erro de dependência circular no build.** A mensagem cita uma cadeia de projetos que se referenciam em ciclo. Causa quase certa: algum comando foi executado com os dois caminhos invertidos. Localizar a referência errada no `.csproj` e removê-la — pela CLI:

```bash
dotnet remove <projeto>.csproj reference <referencia-errada>.csproj
```

**`dotnet add ... reference` reclama que o arquivo não existe.** Conferir se você está na raiz do repositório e se os caminhos começam em `src/`. Os comandos deste bloco pressupõem a raiz como diretório atual.

**O build passa, mas o `.csproj` não mudou.** Provavelmente a referência já existia e a CLI a ignorou silenciosamente. Não é erro.

**Adicionei uma referência proibida por engano.** Remover com `dotnet remove ... reference ...`, rodar `dotnet build` e conferir o `git status`. Se ainda não houve commit, `git restore <arquivo>` devolve o `.csproj` ao estado anterior.

## 9. Checklist de aceitação

```text
[ ] Game.Application referencia Game.Domain
[ ] Game.Infrastructure referencia Game.Application
[ ] Game.Infrastructure referencia Game.Domain
[ ] Game.Api referencia Game.Application
[ ] Game.Api referencia Game.Infrastructure
[ ] Game.Domain não referencia ninguém
[ ] nenhuma dependência proibida foi criada
[ ] dotnet build Cartola90.slnx passa com 0 erros
[ ] git status mostra apenas os três .csproj modificados
[ ] ausência de teste automatizado em A4 foi conscientemente aceita
[ ] ficou entendido que a proteção executável da regra pertence a A10
```

## 10. Linha de parada

Quando o build passar e os quatro `.csproj` estiverem conforme a tabela:

```text
A4 — Configurar referências ................. ✅
```

PARAR.

Não executar ainda nada relacionado a Docker, PostgreSQL, EF Core ou `DbContext`. Como A5 já foi executado antecipadamente, o próximo bloco real é:

```text
A6 — Preparar PostgreSQL local
```

---

# Pendências abertas do Manual

Itens conhecidos, com o bloco em que devem ser resolvidos. Não são esquecimentos: são compromissos datados.

| # | Pendência | Resolver em |
|---|---|---|
| 1 | `README.md` ainda se intitula "Manager Football"; o nome oficial é `Cartola90` | próximo commit de documentação |
| 2 | `app.UseAuthorization()` é ruído de template no `Program.cs` | A8 |
| 3 | `Game.Api.http` existe sem endpoints para exercitar | A8 |
| 4 | O runner do GitHub Actions precisará de SDK 10 para entender `.slnx` | A12 |
| 5 | Teste arquitetural que impede a criação de dependência proibida | A10 |
| 6 | `.gitignore` precisará ser revisitado quando o frontend nascer | A9 |
