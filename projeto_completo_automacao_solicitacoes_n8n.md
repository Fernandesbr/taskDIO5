# Projeto Prático: Automação de Solicitações com n8n

**Tipo de atividade:** prática educacional  
**Plataforma de automação:** n8n  
**Versão do projeto:** 1.0  
**Data:** 19/10/2026

---

## 1. Resumo do projeto

Desenvolver uma automação no n8n para receber solicitações enviadas por formulário, validar os dados, registrar cada solicitação em uma planilha e enviar notificações por e-mail aos envolvidos.

O projeto será construído por etapas, começando por uma versão básica e evoluindo para validação de dados, prevenção de duplicidades e tratamento de falhas.

## 2. Problema a resolver

Atualmente, o recebimento e o encaminhamento de solicitações podem depender de atividades manuais: conferir os campos enviados, copiar informações para uma planilha e avisar o responsável pelo atendimento. Isso consome tempo e pode gerar esquecimentos, erros de digitação ou registros repetidos.

A automação deverá reduzir essas tarefas e padronizar o processo.

## 3. Objetivo

Criar um workflow no n8n que:
1. Detecte novas respostas de um formulário.
2. Valide os campos obrigatórios.
3. Evite processar novamente uma solicitação já registrada.
4. Registre os dados em uma planilha.
5. Envie uma notificação ao responsável.
6. Envie uma confirmação ao solicitante.
7. Permita identificar erros e realizar novas tentativas controladas.

## 4. Público e partes interessadas

- **Solicitante:** pessoa que preenche o formulário.
- **Responsável pelo atendimento:** pessoa que recebe e trata a solicitação.
- **Gestor do processo:** pessoa que acompanha os registros e os resultados.
- **Desenvolvedor do workflow:** aluno responsável por configurar e testar a automação.

## 5. Ferramentas

- **n8n:** orquestração do workflow.
- **Google Forms:** coleta das solicitações.
- **Google Sheets:** armazenamento das respostas e dos registros processados.
- **Gmail:** envio das notificações.

As contas e credenciais reais devem ser configuradas pelo aluno. Este documento não contém credenciais nem representa uma execução real.

## 6. Escopo

### Incluído
- Preparação de formulário.
- Planilha de respostas e planilha de controle.
- Gatilho de novas linhas no Google Sheets.
- Organização e validação dos dados.
- Verificação de duplicidade.
- Registro da solicitação com status inicial.
- Notificação por e-mail ao responsável e ao solicitante.
- Tratamento básico de erros.
- Testes e documentação.

### Fora do escopo inicial
- Portal web próprio.
- Integração com sistemas corporativos.
- Autenticação avançada de usuários finais.
- Painel gerencial.
- Uso de inteligência artificial.
- Gestão completa do ciclo de atendimento e atualização automática do status após resolução.

## 7. Requisitos funcionais

**RF01 — Receber solicitação:** o workflow deve detectar uma nova resposta armazenada pelo formulário.

**RF02 — Validar dados:** verificar nome, e-mail, categoria e descrição.

**RF03 — Identificar duplicidade:** utilizar um identificador estável da resposta original para impedir a criação repetida do mesmo registro.

**RF04 — Registrar solicitação:** salvar os dados na aba de controle com data, identificador e status inicial `Pendente`.

**RF05 — Notificar responsável:** enviar e-mail com os detalhes da solicitação depois de confirmar o registro.

**RF06 — Confirmar recebimento:** enviar e-mail ao solicitante depois de confirmar o registro.

**RF07 — Registrar falhas:** permitir identificar se ocorreu erro de validação, gravação ou envio de e-mail.

## 8. Requisitos não funcionais

- Utilizar permissões mínimas necessárias para as integrações.
- Não armazenar senhas ou tokens em campos da planilha.
- Manter o registro da solicitação caso um e-mail falhe.
- Evitar que uma nova tentativa gere uma segunda solicitação.
- Documentar as configurações que dependem da versão do n8n.
- Utilizar dados fictícios nos testes.

## 9. Modelo de dados

### 9.1 Formulário

Criar os seguintes campos:
- Nome completo — obrigatório.
- E-mail — obrigatório.
- Categoria — obrigatório, com opções como `Acesso a sistema`, `Problema técnico` e `Solicitação de informação`.
- Descrição — obrigatório.

Vincular o formulário a uma planilha do Google Sheets. As respostas devem ser armazenadas em uma aba própria, sem misturar respostas brutas com registros processados.

### 9.2 Aba `Solicitacoes`

Criar as colunas:

| Coluna | Finalidade |
|---|---|
| ID | Identificador estável e único da solicitação |
| Data_Recebimento | Data e hora de recebimento |
| Nome | Nome do solicitante |
| Email | E-mail do solicitante |
| Categoria | Categoria selecionada |
| Descricao | Descrição da solicitação |
| Status | Status do atendimento, inicialmente `Pendente` |
| Email_Responsavel | Estado do envio: `Pendente`, `Enviado` ou `Falhou` |
| Email_Solicitante | Estado do envio: `Pendente`, `Enviado` ou `Falhou` |
| Ultimo_Erro | Descrição resumida de eventual falha |

O identificador deve ser derivado de um valor estável da resposta original, como um identificador único fornecido pela origem. Se não houver um ID próprio, pode-se usar uma chave consistente baseada na origem e no número da linha, documentando suas limitações. Não gerar um novo ID aleatório a cada execução.

### 9.3 Exemplo fictício

| ID | Data_Recebimento | Nome | Email | Categoria | Descricao | Status |
|---|---|---|---|---|---|---|
| RESP-0001 | 19/10/2026 | Carlos Oliveira | carlos.oliveira@example.com | Acesso a sistema | Solicito acesso ao sistema de gestão logística. | Pendente |

O endereço `example.com` é ilustrativo e não deve ser utilizado para validar o envio real de mensagens.

## 10. Arquitetura do workflow

Fluxo lógico proposto:

`Google Sheets Trigger → Edit Fields → IF (dados válidos?) → consulta de duplicidade → IF (já existe?) → registro em Solicitacoes → envio de e-mails → atualização do resultado das notificações`

Ramificações:
- Dados inválidos → registrar ou sinalizar a rejeição.
- Solicitação já registrada → encerrar sem inserir outro registro.
- Falha ao registrar → registrar a falha e não enviar notificações.
- Falha em um e-mail → manter o registro e marcar a notificação correspondente como `Falhou`.

### Escolha do gatilho

Para este projeto, a abordagem inicial recomendada é o **Google Sheets Trigger**, configurado para detectar `Row Added` na aba de respostas do formulário. A documentação do n8n descreve os eventos `Row Added`, `Row Updated` e `Row Added or Updated`. Consulte a documentação oficial: https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.googlesheetstrigger/

O gatilho é baseado em verificação periódica, portanto a execução pode não ser instantânea. A frequência depende das opções disponíveis na versão e na instalação utilizadas.

## 11. Implementação detalhada

### Nó 1 — Google Sheets Trigger

**Objetivo:** detectar uma nova resposta na aba vinculada ao formulário.

Configurar:
1. Criar ou selecionar a credencial Google apropriada.
2. Selecionar a planilha de respostas.
3. Selecionar a aba correta.
4. Escolher o evento `Row Added`.
5. Executar um teste com uma resposta fictícia.
6. Confirmar os nomes e valores dos campos que chegam ao próximo nó.

Documentação: https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.googlesheetstrigger/

### Nó 2 — Edit Fields (Set)

**Objetivo:** padronizar os dados recebidos e facilitar o mapeamento.

Preparar campos equivalentes a:
- `id_resposta`
- `data_recebimento`
- `nome`
- `email`
- `categoria`
- `descricao`

Os nomes exatos das colunas recebidas devem ser conferidos na execução de teste. Mapear os campos de acordo com a saída real do gatilho, sem presumir nomes que não tenham sido verificados.

### Nó 3 — IF: validação dos campos

**Objetivo:** separar solicitações válidas das inválidas.

Verificações:
- Nome não está vazio.
- E-mail não está vazio e apresenta estrutura básica válida.
- Categoria não está vazia.
- Descrição não está vazia.

Saída `true`: seguir para a consulta de duplicidade.  
Saída `false`: direcionar para tratamento de dados inválidos.

A validação de formato não confirma que o endereço existe ou que a pessoa controla essa caixa postal.

### Nó 4 — Google Sheets: consultar registros

**Objetivo:** procurar o `id_resposta` na aba `Solicitacoes`.

Usar a operação de leitura/consulta compatível com a versão do nó Google Sheets. A configuração deve procurar uma linha cujo ID corresponda ao ID recebido.

Documentação das operações: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/

### Nó 5 — IF: verificar duplicidade

**Objetivo:** decidir se a solicitação já foi registrada.

- Se o identificador existir, encerrar o caminho sem inserir outra solicitação.
- Se não existir, seguir para o registro.

A lógica de consulta e inserção é adequada para uma atividade introdutória, mas não oferece garantia absoluta contra duplicidade em execuções simultâneas. Para um cenário de produção, seria necessário um mecanismo de idempotência mais robusto ou um armazenamento com restrição de unicidade.

### Nó 6 — Google Sheets: adicionar registro

**Objetivo:** adicionar uma linha na aba `Solicitacoes`.

Mapear:
- ID.
- Data de recebimento.
- Nome.
- E-mail.
- Categoria.
- Descrição.
- Status = `Pendente`.
- Email_Responsavel = `Pendente`.
- Email_Solicitante = `Pendente`.
- Ultimo_Erro = vazio.

Só continuar após o nó confirmar que o registro foi gravado com sucesso.

Documentação: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/

### Nó 7 — Gmail: notificar responsável

**Objetivo:** avisar que uma nova solicitação foi registrada.

Assunto sugerido: `Nova solicitação recebida — {{ID}}`

Modelo de mensagem:

Olá,

Uma nova solicitação foi registrada.

Protocolo: {{ID}}  
Solicitante: {{Nome}}  
Categoria: {{Categoria}}  
Descrição: {{Descricao}}  
Status: Pendente

Por favor, avalie a solicitação e providencie o atendimento.

Mensagem automática enviada pelo workflow do n8n.

O destinatário deve ser configurado pelo responsável pelo projeto. Não inventar nem fixar um endereço real no modelo.

Documentação: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.gmail/

### Nó 8 — Gmail: confirmar ao solicitante

**Objetivo:** confirmar o recebimento após a gravação bem-sucedida.

Assunto sugerido: `Confirmação de recebimento — {{ID}}`

Modelo de mensagem:

Olá, {{Nome}}!

Sua solicitação foi recebida e registrada.

Protocolo: {{ID}}  
Categoria: {{Categoria}}  
Status: Pendente

Esta mensagem confirma o recebimento, mas não significa que o atendimento foi concluído.

Mensagem automática.

### Nó 9 — Google Sheets: atualizar resultado dos e-mails

**Objetivo:** atualizar a linha da solicitação após cada tentativa de envio.

- Se o e-mail ao responsável for enviado, marcar `Email_Responsavel = Enviado`.
- Se o e-mail ao solicitante for enviado, marcar `Email_Solicitante = Enviado`.
- Se ocorrer falha, manter o registro e marcar o campo correspondente como `Falhou`, preenchendo `Ultimo_Erro` com uma descrição resumida.

A atualização deve localizar a linha pelo ID único. Se a versão utilizada permitir, separar o envio de cada e-mail e a atualização de seu status para que a falha de uma notificação não apague o resultado da outra.

## 12. Tratamento de falhas

### Falha de validação
Não enviar notificações normais nem registrar a solicitação como válida. Registrar o motivo da rejeição em um controle de erros ou em uma aba separada.

### Falha ao consultar ou gravar no Google Sheets
Sinalizar a falha. Não enviar e-mails de confirmação. Permitir uma nova tentativa controlada e verificar a existência do registro antes de tentar inserir novamente.

### Falha ao enviar e-mail
Preservar a solicitação. Marcar a notificação correspondente como `Falhou` e permitir reenvio sem criar outra solicitação.

### Observação sobre a ordem
Se o workflow enviar um e-mail e falhar antes de gravar o status `Enviado`, uma nova execução pode enviar o mesmo e-mail novamente. Para a versão educacional, documentar essa limitação; para produção, implementar controle de tentativas e idempotência de notificações.

## 13. Plano de testes

| Nº | Cenário | Resultado esperado |
|---|---|---|
| 1 | Todos os campos válidos | Criar um registro e tentar enviar as duas notificações |
| 2 | Nome vazio | Rejeitar ou sinalizar; não enviar notificações normais |
| 3 | E-mail com formato inválido | Rejeitar ou sinalizar |
| 4 | Categoria vazia | Rejeitar ou sinalizar |
| 5 | Mesmo ID processado novamente | Não criar outro registro |
| 6 | Google Sheets indisponível durante a gravação | Sinalizar erro; não enviar confirmação |
| 7 | Falha no e-mail do responsável | Preservar registro e marcar falha dessa notificação |
| 8 | Falha no e-mail do solicitante | Preservar registro e marcar falha dessa notificação |
| 9 | Reenvio após falha | Reutilizar o registro existente, sem duplicá-lo |
| 10 | Solicitação válida após uma inválida | Processar corretamente a nova solicitação |

Para cada teste, registrar: entrada utilizada, resultado esperado, resultado observado e ajuste realizado.

## 14. Critérios de aceite

O projeto será aceito quando:
- O gatilho detectar novas respostas.
- Os dados forem mapeados corretamente.
- Campos obrigatórios forem validados.
- A consulta de duplicidade funcionar nos testes.
- Os registros forem gravados com o status correto.
- As notificações forem enviadas em um cenário de sucesso.
- Falhas de registro impedirem o envio de confirmações.
- Falhas de e-mail não apagarem o registro.
- Os resultados dos testes forem documentados.

## 15. Entregáveis

1. Formulário de teste.
2. Planilha com respostas brutas e aba de solicitações processadas.
3. Workflow do n8n.
4. Evidências de execução e testes.
5. Documento com decisões, limitações e instruções de operação.

## 16. Plano de execução para a aula

### Fase 1 — Fluxo básico
- Preparar o formulário e a planilha.
- Configurar o gatilho.
- Mapear os campos.
- Inserir a solicitação na aba de controle.

### Fase 2 — Regras e notificações
- Validar campos.
- Verificar duplicidade.
- Enviar e-mails.
- Atualizar o estado das notificações.

### Fase 3 — Testes e confiabilidade
- Testar entradas inválidas.
- Simular falhas.
- Implementar tratamento de erros.
- Documentar as limitações e os resultados.

## 17. Limitações e premissas

- A automação não foi executada neste documento; as configurações precisam ser realizadas no ambiente do aluno.
- Os nomes exatos de campos, menus e operações podem variar conforme a versão do n8n.
- O acesso aos serviços Google exige credenciais autorizadas.
- A frequência do gatilho pode introduzir atraso entre o envio do formulário e o início do workflow.
- A prevenção básica de duplicidades não é garantia absoluta contra concorrência.
- O envio de e-mails depende das permissões e dos limites da conta utilizada.
- Dados pessoais reais devem ser evitados nos testes; usar dados fictícios.

## 18. Referências técnicas

- Documentação do Google Sheets Trigger: https://docs.n8n.io/integrations/builtin/trigger-nodes/n8n-nodes-base.googlesheetstrigger/
- Documentação do nó Google Sheets: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/
- Documentação do nó Gmail: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.gmail/

---

## Prompt para orientar a construção do workflow

Atue como especialista em n8n e instrutor para iniciantes. Ajude-me a implementar este projeto de automação de solicitações passo a passo, sem presumir que as credenciais ou integrações já estão configuradas.

Comece pela Fase 1 e explique:
1. Como preparar o formulário e as abas da planilha.
2. Como configurar o Google Sheets Trigger.
3. Como testar a entrada de dados.
4. Como mapear os campos e inserir o registro na aba `Solicitacoes`.
5. Como confirmar que a primeira versão funciona antes de avançar.

Informe o nome dos nós, os parâmetros que precisam ser configurados, as expressões necessárias e como validar cada etapa. Quando houver diferenças entre versões do n8n, explique como identificar a opção equivalente. Não invente resultados de execução: diferencie instruções de configuração de testes realmente realizados.
