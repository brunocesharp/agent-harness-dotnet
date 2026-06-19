# Especificação Funcional — MVP Agent Harness

> Gerado em: 18/06/2026
> Versão: 1.0
> Status: Rascunho
> Referência: docs/agent-harness-dotnet/escopo/mvp-agent-harness/escopo.md

---

## Contexto

Este documento especifica o comportamento funcional do MVP do Agent Harness Dotnet. O harness permite selecionar, trocar e orquestrar modelos de IA em runtime via .NET Core, com suporte a ciclo de execução stateful, histórico de conversação, controle de contexto, memória e interface web.

---

## Features Especificadas

- [Feature 1: Escolha de Modelo](#feature-1-escolha-de-modelo)
- [Feature 2: Estrutura do Agent Harness](#feature-2-estrutura-do-agent-harness)
- [Feature 3: Query Loop / Ciclo de Execução](#feature-3-query-loop--ciclo-de-execução)
- [Feature 4: Histórico de Conversação](#feature-4-histórico-de-conversação)

---

## Feature 1: Escolha de Modelo

### Narrativa

```
Feature: Escolha de Modelo
  In order to usar o modelo de IA mais adequado para cada tarefa sem reiniciar ou reconfigurar o sistema
  As desenvolvedor ou usuário do harness
  I want que o sistema liste os modelos disponíveis, permita selecionar um e troque em runtime
```

### Regras de Negócio

- **RN-1.1** — Modelos disponíveis são carregados exclusivamente do appsettings
- **RN-1.2** — Cada modelo expõe nome, tamanho em parâmetros e versão
- **RN-1.3** — Somente modelos registrados no appsettings podem ser selecionados
- **RN-1.4** — Ao trocar de modelo durante sessão ativa, o contexto é preservado e transferido pela gestão de contexto
- **RN-1.5** — Se nenhum modelo for selecionado explicitamente, o modelo marcado como padrão no appsettings é carregado
- **RN-1.6** — Se não houver modelos configurados no appsettings, o envio de prompts não é possível

### Cenários

---

#### Cenário 1.1: Listar modelos disponíveis

```gherkin
Dado que o appsettings possui modelos configurados
  (ex: "Claude Sonnet 4.6 — 175B — v4.6", "Llama 3 — 70B — v3.0")
Quando o dev solicita a listagem de modelos
Então o sistema retorna nome, tamanho em parâmetros e versão de cada modelo registrado
```

---

#### Cenário 1.2: Selecionar um modelo da lista

```gherkin
Dado que os modelos disponíveis incluem "Claude Sonnet 4.6" e "Llama 3"
Quando o dev seleciona "Llama 3"
Então o modelo "Llama 3 — 70B — v3.0" é carregado e fica pronto para receber prompts
```

---

#### Cenário 1.3: Listar quando nenhum modelo está configurado

```gherkin
Dado que o appsettings não possui nenhum modelo configurado
Quando o dev solicita a listagem de modelos
Então o sistema retorna uma lista vazia
  E não é possível enviar prompts até que um modelo seja configurado
```

---

#### Cenário 1.4: Trocar modelo com sessão ativa

```gherkin
Dado que o modelo "Claude Sonnet 4.6" está ativo com uma sessão em andamento
Quando o dev seleciona "Llama 3" durante a sessão
Então o modelo "Llama 3" é carregado como ativo
  E o contexto da sessão é transferido para o novo modelo pela gestão de contexto
```

---

#### Cenário 1.5: Usar modelo padrão sem seleção explícita

```gherkin
Dado que o appsettings define "Claude Sonnet 4.6" como modelo padrão
  E que nenhum modelo foi selecionado explicitamente
Quando o harness é iniciado
Então o modelo "Claude Sonnet 4.6" é carregado automaticamente como modelo ativo
```

### Pontos em Aberto

_Nenhum._

---

## Feature 2: Estrutura do Agent Harness

### Narrativa

```
Feature: Estrutura do Agent Harness
  In order to ter uma base padronizada e extensível para construir agents sem reimplementar infraestrutura a cada vez
  As desenvolvedor que integra o harness
  I want que o sistema forneça contratos de agent, ciclo de vida controlado e execução de prompts via injeção de dependência
```

### Regras de Negócio

- **RN-2.1** — Agent é registrado e configurado via injeção de dependência (.NET DI)
- **RN-2.2** — Iniciar um agent significa incluí-lo no loop — ele passa a operar e ficar disponível para receber prompts
- **RN-2.3** — Executar um prompt significa o agent processar contexto de chat + prompt e devolver resposta em texto
- **RN-2.4** — Encerrar um agent significa removê-lo do loop — ele deixa de receber prompts
- **RN-2.5** — Enquanto nenhum agent estiver ativo, o envio de prompt fica bloqueado
- **RN-2.6** — Iniciar um agent já em execução é uma operação sem efeito
- **RN-2.7** — Executar prompt em agent encerrado é uma operação sem efeito

### Cenários

---

#### Cenário 2.1: Registrar e iniciar agent via DI

```gherkin
Dado que o agent está registrado no container de injeção de dependência
Quando o dev inicia o agent
Então o agent é incluído no loop e fica operacional para receber prompts
```

---

#### Cenário 2.2: Executar prompt via agent ativo

```gherkin
Dado que o agent está ativo no loop
  E que há um contexto de chat em andamento
Quando o dev envia um prompt ao agent
Então o agent processa o contexto + prompt
  E devolve uma resposta em texto
```

---

#### Cenário 2.3: Encerrar agent

```gherkin
Dado que o agent está ativo no loop
Quando o dev encerra o agent
Então o agent é removido do loop
  E deixa de receber novos prompts
```

---

#### Cenário 2.4: Tentar enviar prompt sem nenhum agent ativo

```gherkin
Dado que nenhum agent está ativo no loop
Quando o dev tenta enviar um prompt
Então o envio fica bloqueado
  E a ação de confirmar o envio permanece desativada até que um agent seja iniciado
```

---

#### Cenário 2.5: Iniciar agent já em execução

```gherkin
Dado que o agent já está ativo no loop
Quando o dev tenta iniciá-lo novamente
Então o sistema não realiza nenhuma ação
  E o agent continua operando normalmente
```

---

#### Cenário 2.6: Executar prompt em agent encerrado

```gherkin
Dado que o agent foi encerrado e removido do loop
Quando o dev tenta enviar um prompt para esse agent
Então o sistema não realiza nenhuma ação
```

### Pontos em Aberto

_Nenhum._

---

## Feature 3: Query Loop / Ciclo de Execução

### Narrativa

```
Feature: Query Loop / Ciclo de Execução
  In order to manter conversas coerentes com múltiplos turnos sem perder o fio da conversa
  As usuário do harness
  I want que o sistema execute um ciclo stateful que acumule contexto entre turnos e saiba quando encerrar
```

### Regras de Negócio

- **RN-3.1** — O loop é iniciado por chamada de API
- **RN-3.2** — O estado acumulado entre turnos inclui: histórico de mensagens, contador de turnos, modelo ativo, skills ativas, memória de sessão, status da sessão e tokens consumidos
- **RN-3.3** — Após cada resposta, a sessão entra em stand-by aguardando o próximo prompt
- **RN-3.4** — Uma sessão em stand-by retoma o contexto completo ao receber novo prompt
- **RN-3.5** — Múltiplas sessões podem estar em stand-by simultaneamente
- **RN-3.6** — Ao atingir o limite de turnos configurado, o processo de encolhimento de contexto é executado automaticamente
- **RN-3.7** — A sessão é encerrada explicitamente pelo usuário

### Cenários

---

#### Cenário 3.1: Iniciar sessão via chamada de API

```gherkin
Dado que um agent está ativo no loop
Quando o dev chama a API para iniciar uma nova sessão
Então o loop cria a sessão com estado inicial
  E a sessão fica em stand-by aguardando o primeiro prompt
```

---

#### Cenário 3.2: Processar turno e acumular estado

```gherkin
Dado que a sessão está em stand-by
Quando o usuário envia um prompt
Então o agent processa o prompt com o contexto acumulado
  E o estado da sessão é atualizado com a nova mensagem, o contador de turnos e os tokens consumidos
  E a sessão retorna ao stand-by aguardando o próximo prompt
```

---

#### Cenário 3.3: Retomar sessão em stand-by com contexto completo

```gherkin
Dado que a sessão está em stand-by após 3 turnos anteriores
Quando o usuário envia um novo prompt
Então o agent retoma a sessão com o histórico completo dos 3 turnos anteriores
  E processa o novo prompt no contexto acumulado
```

---

#### Cenário 3.4: Múltiplas sessões em stand-by simultaneamente

```gherkin
Dado que existem 3 sessões diferentes em stand-by
Quando o usuário envia prompt para a sessão 2
Então apenas a sessão 2 é processada
  E as sessões 1 e 3 permanecem em stand-by com seus contextos preservados
```

---

#### Cenário 3.5: Encolhimento de contexto ao atingir limite de turnos

```gherkin
Dado que a sessão atingiu o limite de turnos configurado
Quando o usuário envia um novo prompt
Então o processo de encolhimento de contexto é executado automaticamente
  E o contexto é reduzido antes de processar o novo prompt
  E a sessão continua sem ser encerrada
```

---

#### Cenário 3.6: Usuário encerra sessão explicitamente

```gherkin
Dado que a sessão está em stand-by
Quando o usuário encerra a sessão
Então a sessão muda de status para encerrada
  E deixa de receber novos prompts
  E o estado da sessão é descartado
```

### Pontos em Aberto

_Nenhum._

---

## Feature 4: Histórico de Conversação

### Narrativa

```
Feature: Histórico de Conversação
  In order to manter a coerência das conversas e permitir revisitar interações anteriores
  As usuário do harness
  I want que o sistema registre, persista e recupere o histórico completo de mensagens de cada sessão
```

### Regras de Negócio

- **RN-4.1** — O histórico é persistido automaticamente no banco de dados a cada turno
- **RN-4.2** — Cada mensagem armazena texto e um identificador único para recuperação de informações completas
- **RN-4.3** — A recuperação do histórico retorna as N últimas mensagens da sessão, sendo N configurável
- **RN-4.4** — Solicitar o histórico de uma sessão sem mensagens retorna vazio
- **RN-4.5** — Limpar o histórico arquiva as mensagens no banco — nenhuma mensagem é apagada permanentemente

### Cenários

---

#### Cenário 4.1: Salvar mensagem automaticamente após cada turno

```gherkin
Dado que a sessão está em andamento
Quando o usuário envia um prompt e o agent responde
Então a mensagem do usuário e a resposta do agent são persistidas no banco de dados
  E cada mensagem recebe um identificador único
```

---

#### Cenário 4.2: Recuperar as N últimas mensagens de uma sessão

```gherkin
Dado que a sessão possui 20 mensagens registradas no banco
  E que o limite configurado é de 10 mensagens
Quando o usuário solicita o histórico da sessão
Então o sistema retorna as 10 últimas mensagens
  E cada mensagem retorna texto e identificador único
```

---

#### Cenário 4.3: Solicitar histórico de sessão sem mensagens

```gherkin
Dado que a sessão foi iniciada mas nenhum prompt foi enviado
Quando o usuário solicita o histórico da sessão
Então o sistema retorna vazio
```

---

#### Cenário 4.4: Limpar histórico de uma sessão

```gherkin
Dado que a sessão possui mensagens registradas no banco
Quando o usuário limpa o histórico da sessão
Então as mensagens são arquivadas no banco
  E o histórico ativo da sessão fica vazio
  E as mensagens arquivadas permanecem acessíveis pelo identificador
```

### Pontos em Aberto

_Nenhum._

---

## Próximos Passos Sugeridos

- **spec-functional** — especificar as demais features: Estrutura do Harness, Query Loop, Histórico, Contexto, Memória, Prompt, Skills, Interface Web
- **refinement-api** — detalhar os contratos de endpoint para listagem e seleção de modelos

---

## Histórico

| Data | Versão | Alteração | Autor |
|------|--------|-----------|-------|
| 18/06/2026 | 1.0 | Versão inicial — Feature 1: Escolha de Modelo | brunocesharp |
| 18/06/2026 | 1.1 | Feature 2: Estrutura do Agent Harness | brunocesharp |
| 18/06/2026 | 1.2 | Feature 3: Query Loop / Ciclo de Execução | brunocesharp |
| 18/06/2026 | 1.3 | Feature 4: Histórico de Conversação | brunocesharp |
