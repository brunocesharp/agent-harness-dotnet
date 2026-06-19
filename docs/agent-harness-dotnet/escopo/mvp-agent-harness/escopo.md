# MVP — Agent Harness

**Situação:** Rascunho
**Data de criação:** 18/06/2026
**Última atualização:** 18/06/2026
**Projeto:** Agent Harness Dotnet
**Repositório:** https://github.com/brunocesharp/agent-harness-dotnet

---

## Resumo Executivo

Este escopo cobre a implementação do MVP do Agent Harness Dotnet, entregando a fundação técnica para orquestração de modelos de IA na empresa. O MVP abrange a seleção de modelos, a estrutura base do agent harness e o controle de skills — sem interface web, autenticação ou integrações externas.

---

## Contexto

A empresa não possui nenhum processo ou ferramenta para controlar e utilizar diferentes modelos de IA. Este escopo entrega a primeira camada de orquestração centralizada, construída em .NET Core, que permitirá a toda a empresa começar a usar e alternar entre modelos de IA de forma estruturada.

### Situação Atual

Não existe nenhum processo, ferramenta ou fluxo para uso de modelos de IA internamente. Cada time que precisar usar IA precisa criar sua própria solução do zero.

### Situação Desejada

Um agent harness em .NET Core que permite selecionar e trocar modelos de IA dinamicamente, com controle de skills plugável — acessível via API por devs, analistas, time de produto e designers.

---

## Abrangência

### Inclui (In Scope)

- Escolha e troca de modelo de IA em runtime
- Estrutura base do agent harness (contratos, ciclo de vida, execução)
- Controle de skills (registro, ativação e execução via agent)

### Não Inclui (Out of Scope)

- Interface web — fora do MVP; pode ser adicionada em escopo futuro
- Integrações externas — sem conectores com sistemas de terceiros neste momento
- Autenticação — sem camada de auth neste MVP; será endereçada em escopo futuro

### Fronteiras com Outros Escopos

| Escopo relacionado | O que ele cobre | Fronteira com este escopo |
|--------------------|-----------------|---------------------------|
| Escopo futuro (auth) | Autenticação e autorização | Este escopo não implementa segurança de acesso |
| Escopo futuro (UI) | Interface web | Este escopo entrega apenas API |

---

## Usuários Afetados

| Perfil | Quantidade estimada | Como é afetado | Nível de impacto |
|--------|---------------------|----------------|------------------|
| Desenvolvedores | a definir | Integram e orquestram modelos via API | Alto |
| Analistas | a definir | Utilizam modelos para análises e processos | Alto |
| Time de Produto | a definir | Exploram capacidades de IA nos produtos | Médio |
| Designers | a definir | Usam IA como suporte criativo | Médio |

---

## Oportunidades Endereçadas

> *Mapeamento de oportunidades (opportunity.md) não realizado. Seção preenchida com base na visão do projeto.*

| # | Oportunidade | Prioridade | Como este escopo endereça |
|---|--------------|------------|---------------------------|
| 1 | Ausência de qualquer processo para uso de modelos de IA | A | Entrega a fundação técnica para uso e controle de modelos |
| 2 | Impossibilidade de trocar entre modelos conforme a necessidade | A | Implementa seleção e troca de modelo em runtime |

### Oportunidades NÃO Endereçadas

| Oportunidade | Por que ficou de fora | Quando será endereçada |
|--------------|----------------------|------------------------|
| Acesso seguro e controlado aos modelos | Auth está fora do MVP | Escopo futuro |
| Uso via interface visual | UI está fora do MVP | Escopo futuro |

---

## Funcionalidades e Comportamentos Esperados

### Escolha de Modelo

| # | Funcionalidade | Descrição | Critério de Aceite | Prioridade |
|---|----------------|-----------|-------------------|------------|
| F1.1 | Listagem de modelos disponíveis | Retorna os modelos registrados e disponíveis para uso | Endpoint retorna lista com nome e status de cada modelo | Must |
| F1.2 | Seleção de modelo | Permite definir qual modelo será utilizado pelo agent | Agent instancia e utiliza o modelo selecionado | Must |
| F1.3 | Troca de modelo em runtime | Permite trocar o modelo ativo sem reiniciar o agent | Troca ocorre sem perda de contexto ou erro | Should |
| F1.4 | Modelo padrão configurável | Permite definir um modelo padrão via configuração | Agent inicia com modelo padrão quando nenhum é especificado | Should |

### Estrutura do Agent Harness

| # | Funcionalidade | Descrição | Critério de Aceite | Prioridade |
|---|----------------|-----------|-------------------|------------|
| F2.1 | Contrato base do harness | Interface/abstração que define o comportamento de um agent | Qualquer implementação de agent segue o contrato definido | Must |
| F2.2 | Ciclo de vida do agent | Gerencia criação, execução e encerramento do agent | Agent pode ser iniciado, executado e encerrado de forma controlada | Must |
| F2.3 | Execução de prompt via agent | Envia um prompt ao modelo ativo e retorna a resposta | Agent recebe prompt, chama o modelo e retorna resposta | Must |
| F2.4 | Configuração de agent via injeção de dependência | Harness configurável via DI do .NET | Agent configurado e registrado via `IServiceCollection` | Should |

### Controle de Skills

| # | Funcionalidade | Descrição | Critério de Aceite | Prioridade |
|---|----------------|-----------|-------------------|------------|
| F3.1 | Registro de skills | Permite registrar skills disponíveis para o agent | Skill registrada fica disponível para execução | Must |
| F3.2 | Execução de skill via agent | Agent identifica e executa a skill adequada ao contexto | Skill é executada com os parâmetros corretos e retorna resultado | Must |
| F3.3 | Ativação e desativação de skills | Permite habilitar/desabilitar skills sem removê-las | Skill desativada não é considerada pelo agent | Should |
| F3.4 | Contrato base de skill | Interface que define o comportamento de uma skill | Qualquer skill implementa o contrato e é reconhecida pelo harness | Must |

### Legenda de Prioridade (MoSCoW)

- **Must:** Obrigatório para o lançamento
- **Should:** Importante, mas pode ser ajustado
- **Could:** Desejável se houver tempo
- **Won't (this time):** Não será feito neste escopo

---

## Requisitos Não-Funcionais

| Categoria | Requisito | Meta | Como medir |
|-----------|-----------|------|------------|
| Tecnologia | Stack em .NET Core | 100% .NET Core | Revisão de código |
| Performance | Tempo de resposta do harness (overhead) | < 100ms sobre o tempo do modelo | Benchmark local |
| Extensibilidade | Novos modelos e skills adicionáveis sem alterar o core | Zero alteração no harness core | Revisão de arquitetura |

---

## Hipóteses Críticas Associadas

> *Mapeamento de hipóteses (assumptions.md) não realizado. Hipóteses listadas com base na visão e no escopo.*

| # | Hipótese | Impacto se falsa | Situação | Plano de validação |
|---|----------|------------------|----------|-------------------|
| H1 | Os modelos de IA a serem suportados possuem SDKs ou APIs compatíveis com .NET Core | Pode inviabilizar integrações ou aumentar o esforço | A validar | Pesquisar SDKs disponíveis antes de implementar |
| H2 | Uma API sem autenticação é aceitável para o MVP interno | Exposição de dados indevida | A validar | Confirmar com stakeholders se há restrição de segurança |
| H3 | O conceito de "skill" é suficientemente genérico para cobrir os casos de uso iniciais | Necessidade de redesign da abstração | A validar | Prototipar com 2-3 skills reais |

### Hipóteses Aceitas como Premissa

- .NET Core é a tecnologia obrigatória para este projeto
- O acesso inicial será restrito a usuários internos da empresa

---

## Stakeholders Envolvidos

| Stakeholder | Papel no escopo | Interesse | Influência | Precisa aprovar? |
|-------------|-----------------|-----------|------------|------------------|
| brunocesharp | Responsável técnico | Alto | Alta | Sim |
| Times internos (devs, analistas, produto, design) | Usuários finais | Alto | Média | Não |

### Matriz de Aprovação

| Decisão | Quem aprova | Quem é consultado | Quem é informado |
|---------|-------------|-------------------|------------------|
| Escopo geral | brunocesharp | Times internos | Empresa |
| Mudanças de funcionalidade | brunocesharp | Times internos | a definir |

---

## Métricas de Sucesso

### North Star deste Escopo

| Métrica | Baseline | Meta | Prazo |
|---------|----------|------|-------|
| Times utilizando o harness para interagir com modelos de IA | 0 times | 1+ time usando em ambiente de desenvolvimento | a definir |

### Métricas de Suporte

| Categoria | Métrica | Baseline | Meta | Prazo |
|-----------|---------|----------|------|-------|
| Técnica | Modelos suportados pelo harness | 0 | 2+ modelos | a definir |
| Técnica | Skills registráveis e executáveis | 0 | 3+ skills | a definir |
| Usuário | Feedback positivo de adoção interna | N/A | Aprovação dos times piloto | a definir |

### Critérios de Lançamento (Go/No-Go)

- [ ] Harness consegue selecionar e trocar modelos em runtime
- [ ] Pelo menos 2 modelos de IA integrados e funcionando
- [ ] Sistema de skills registrável e executável via agent
- [ ] Documentação mínima de uso para times internos

---

## Riscos do Escopo

| # | Risco | Probabilidade | Impacto | Mitigação | Responsável |
|---|-------|---------------|---------|-----------|-------------|
| R1 | SDKs dos modelos escolhidos não serem compatíveis com .NET Core | Média | Alto | Validar compatibilidade antes de iniciar implementação | brunocesharp |
| R2 | Abstração de skills não cobrir casos de uso reais | Média | Alto | Prototipar com casos concretos antes de finalizar contrato | brunocesharp |
| R3 | Baixa adoção interna por falta de documentação ou suporte | Baixa | Médio | Incluir documentação mínima como critério de lançamento | brunocesharp |

---

## Restrições e Dependências

### Restrições

| Tipo | Restrição | Impacto no escopo |
|------|-----------|-------------------|
| Tecnologia | .NET Core obrigatório | Stack definida; bibliotecas devem ser compatíveis |
| Prazo | Sem data de conclusão definida | Sem pressão de entrega no momento |

### Dependências

| Dependência | Tipo | Status | Responsável | Impacto se atrasar |
|-------------|------|--------|-------------|-------------------|
| Definição dos modelos de IA a suportar | Técnica | Pendente | brunocesharp | Bloqueia F1.1 e F1.2 |

---

## Decisões Pendentes

| # | Decisão | Opções | Quem decide | Prazo | Impacto de não decidir |
|---|---------|--------|-------------|-------|------------------------|
| D1 | Quais modelos de IA serão suportados no MVP | Claude, GPT-4, Gemini, outros | brunocesharp | Antes de iniciar desenvolvimento | Bloqueia implementação de F1.x |
| D2 | Formato de exposição da API (REST, gRPC, biblioteca) | REST API, gRPC, NuGet Package | brunocesharp | Antes de iniciar desenvolvimento | Define arquitetura do harness |

---

## Premissas do Escopo

- O harness será consumido por usuários internos da empresa
- Não há restrições de segurança que impeçam o uso sem autenticação no MVP
- Os modelos de IA a serem integrados possuem APIs acessíveis via .NET Core
- O escopo pode evoluir após validação com os times internos

---

## Estimativa de Esforço (Alto Nível)

| Fase | Esforço estimado | Confiança |
|------|------------------|-----------|
| Especificação | a definir | Baixa |
| Desenvolvimento | a definir | Baixa |
| Testes | a definir | Baixa |
| Homologação | a definir | Baixa |
| **Total** | **a definir** | **Baixa** |

*Nota: Estimativas serão definidas após validação das decisões pendentes D1 e D2.*

---

## Histórico de Situação

| Data | Situação | Responsável | Observação |
|------|----------|-------------|------------|
| 18/06/2026 | Rascunho | brunocesharp | Criação inicial |

---

## Anexos e Referências

- [Discovery - Vision](./discovery/vision.md)
