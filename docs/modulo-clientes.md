# Módulo de Clientes

## 1. Visão geral

O módulo de Clientes é a central de consulta do relacionamento entre a empresa e cada cliente dentro do Pombo Correio.

Ele não substitui o cadastro do ERP nem transforma o produto em um CRM comercial completo. O ERP continua sendo a fonte principal dos dados cadastrais, serviços realizados e agendamentos. O Pombo Correio mantém uma representação local desses dados para permitir consultas, filtros e automações de relacionamento.

O módulo deve responder, de forma simples, às seguintes perguntas:

- Quem é o cliente?
- Qual foi o último serviço realizado?
- O cliente possui um próximo agendamento?
- Quando foi enviada a última mensagem automática?
- Existe alguma mensagem futura programada?
- De quais campanhas o cliente participa ou participou?
- Por que uma participação em campanha foi encerrada?
- O cliente está autorizado a receber campanhas automáticas?
- Quais bandeiras automáticas e manuais se aplicam ao cliente?

## 2. Princípios do módulo

### 2.1. O ERP é a fonte principal

Os dados cadastrais e comerciais são originados no ERP. Os campos sincronizados devem ser exibidos como somente leitura.

Alterações cadastrais devem ser realizadas no ERP e refletidas no Pombo Correio após a sincronização.

### 2.2. Identidade interna

Cada cliente deve possuir um identificador interno próprio no Pombo Correio.

O identificador do ERP deve ser armazenado apenas como referência técnica da integração e não precisa ser exibido na listagem de clientes.

O sistema seguirá a base do ERP. Deduplicação, mesclagem e correção de duplicidades não fazem parte do MVP.

### 2.3. O módulo não depende de respostas do WhatsApp

No MVP, o sistema não deve assumir que possui controle confiável sobre respostas recebidas pelo WhatsApp.

Por isso:

- não existe campo de última resposta;
- não existe indicador de última interação recebida;
- respostas não são usadas como gatilho de parada;
- a timeline não registra respostas que o sistema não consegue confirmar;
- o campo deve ser chamado de **última mensagem enviada**, e não de último contato.

## 3. Origem dos dados

O módulo será alimentado por três fontes do ERP:

1. cadastro de clientes;
2. serviços realizados;
3. agendamentos.

### 3.1. Cadastro de clientes

Fonte principal para:

- nome;
- telefone;
- e-mail;
- data de nascimento;
- identificador do cliente no ERP.

### 3.2. Serviços realizados

Fonte do histórico de atendimentos e de possíveis atualizações cadastrais mais recentes.

Quando um novo serviço realizado for importado, o sistema deve verificar se o registro contém nome, telefone ou e-mail mais atuais. Valores preenchidos e válidos podem atualizar o cadastro local.

O sistema deve armazenar internamente:

- origem da atualização;
- data de referência da origem;
- data da sincronização.

### 3.3. Agendamentos

Fonte dos compromissos futuros do cliente.

O sistema deve identificar o próximo agendamento válido e utilizá-lo para:

- exibição fixa na ficha;
- bandeira automática de agendamento;
- filtros de campanhas;
- regras de parada, quando configuradas.

Agendamentos cancelados não devem ser considerados.

## 4. Regras de sincronização

### 4.1. Criação

Quando um cliente do ERP ainda não possuir registro interno:

1. gerar um ID interno;
2. armazenar o identificador do ERP;
3. normalizar os dados de contato;
4. criar o cadastro local;
5. registrar a data da sincronização.

### 4.2. Atualização

Quando o cliente já existir:

- atualizar apenas os campos sincronizados;
- preservar dados próprios do Pombo Correio;
- ignorar valores vazios quando já existir um valor válido;
- impedir que valores inválidos substituam dados válidos;
- registrar origem e data da atualização.

A sincronização nunca deve sobrescrever:

- permissão para campanhas automáticas;
- bandeiras manuais;
- histórico de campanhas;
- timeline;
- mensagens programadas ou enviadas.

### 4.3. Ausência de exclusão ou inativação

O MVP não terá fluxo de exclusão, arquivamento ou inativação manual de clientes.

## 5. Estrutura da tela do cliente

A ficha do cliente deve possuir uma área fixa de resumo e três abas:

1. **Dados**;
2. **Campanhas e automações**;
3. **Timeline de eventos**.

```text
Cliente
├── Resumo fixo
│   ├── Último atendimento
│   └── Próximo agendamento
│
├── Dados
├── Campanhas e automações
└── Timeline de eventos
```

## 6. Resumo fixo

O resumo deve permanecer visível independentemente da aba selecionada.

### 6.1. Último atendimento

Exibir:

- data do atendimento;
- nome do serviço realizado.

Último atendimento e último serviço representam o mesmo evento e não devem ser tratados como dois campos independentes.

Exemplo:

```text
Último atendimento
10/07/2026
Unha em Gel
```

### 6.2. Próximo agendamento

Exibir:

- data e hora;
- serviço, quando disponível.

Quando não houver agendamento futuro válido, informar de forma simples que o cliente não possui próximo agendamento.

## 7. Aba Dados

A aba Dados concentra os dados cadastrais, configurações do cliente, bandeiras e informações resumidas de comunicação.

### 7.1. Dados cadastrais sincronizados

Exibir como somente leitura:

- nome;
- telefone;
- situação do telefone;
- e-mail;
- data de nascimento;
- data e hora da última sincronização.

O identificador do ERP pode ser armazenado internamente, mas não precisa ser destacado para o usuário.

Não fazem parte do MVP:

- unidade;
- profissional responsável;
- primeiro atendimento;
- quantidade de atendimentos;
- valor total gasto;
- ticket médio;
- notas internas.

### 7.2. Configuração de campanhas automáticas

A configuração **Receber campanhas automáticas: Sim ou Não** deve ficar nesta aba.

Essa configuração pertence ao Pombo Correio e não depende do ERP.

Quando habilitada, o cliente pode:

- entrar em campanhas automáticas;
- continuar participando de campanhas já iniciadas;
- receber ações futuras programadas.

Quando desabilitada, o cliente:

- não entra em novas campanhas automáticas;
- tem suas participações automáticas em andamento encerradas;
- não recebe novas mensagens automáticas.

A alteração da configuração não exige justificativa. Porém, quando ela encerrar uma participação, o encerramento deve registrar o motivo padronizado:

> Campanhas automáticas desativadas para o cliente.

### 7.3. Bandeiras

Exibir:

- bandeiras automáticas;
- bandeiras manuais.

#### Bandeiras automáticas

No MVP:

- Agendamento marcado;
- Sem atendimento há 30 dias;
- Sem atendimento há 90 dias;
- Sem atendimento há 180 dias.

As faixas de tempo devem ser mutuamente exclusivas. O cliente deve possuir apenas a maior faixa aplicável.

Não usar como bandeiras automáticas:

- cliente em campanha;
- telefone inválido;
- possível duplicidade;
- campanhas silenciadas;
- falha de envio.

#### Bandeiras manuais

O administrador deve poder:

- criar;
- editar o nome;
- definir uma identificação visual simples;
- desativar;
- atribuir a clientes;
- remover de clientes.

Exemplos:

- VIP;
- Prioritário;
- Cliente antigo;
- Atendimento especial.

As bandeiras automáticas e manuais devem estar disponíveis como filtros de campanhas.

### 7.4. Comunicação resumida

Exibir:

- última mensagem enviada;
- próxima mensagem programada.

Para a última mensagem enviada, mostrar:

- data e hora;
- campanha;
- template;
- resultado do envio.

Para a próxima mensagem programada, mostrar:

- campanha;
- ação;
- template;
- data e hora previstas.

## 8. Tratamento do telefone

O telefone é crítico para o envio de mensagens.

### 8.1. Normalização

Antes de salvar ou atualizar:

- remover espaços;
- remover parênteses;
- remover traços;
- remover caracteres não numéricos;
- tratar prefixos duplicados.

### 8.2. Situações do telefone

O sistema deve distinguir:

- disponível e aparentemente válido;
- ausente;
- formato inválido;
- falha de envio registrada.

Essas situações devem aparecer junto ao campo de telefone, não como bandeiras.

### 8.3. Validação antes do envio

Antes de programar ou executar uma ação, verificar:

- existência do telefone;
- formato válido;
- permissão para campanhas automáticas;
- estado da participação;
- regras de parada;
- variáveis obrigatórias do template.

### 8.4. Atualização por serviço realizado

Um telefone mais recente e válido recebido em um serviço realizado pode atualizar o cadastro local.

Não substituir:

- telefone válido por valor vazio;
- telefone válido por valor inválido.

Mensagens já enviadas devem preservar o telefone utilizado no momento do envio.

## 9. Aba Campanhas e automações

A aba deve exibir todas as participações do cliente, separando as atuais das encerradas.

### 9.1. Participações em andamento

Exibir:

- nome da campanha;
- status da participação;
- etapa atual;
- quantidade total de ações;
- próxima ação;
- data e hora da próxima ação;
- atalho para abrir a campanha;
- opção de encerramento manual.

### 9.2. Participações encerradas

Exibir:

- nome da campanha;
- data e hora de entrada;
- data e hora de encerramento;
- etapa em que foi encerrada;
- tipo de encerramento: manual ou automático;
- motivo do encerramento;
- atalho para abrir a campanha.

### 9.3. Reentrada

A regra que determina se o cliente pode participar novamente da mesma campanha pertence à configuração da campanha.

O módulo de Clientes apenas exibe o histórico das participações.

## 10. Encerramento de participação

Sempre que uma participação for encerrada, manual ou automaticamente, o motivo deve ser registrado e exibido.

Dados obrigatórios:

- campanha;
- cliente;
- tipo de encerramento;
- motivo;
- data e hora;
- etapa ou ação interrompida;
- usuário responsável, quando manual e disponível.

Motivos automáticos possíveis:

- novo agendamento identificado;
- novo atendimento realizado;
- campanhas automáticas desativadas para o cliente;
- telefone inválido;
- falha definitiva no envio;
- campanha encerrada;
- cliente deixou de atender aos critérios;
- regra de parada acionada.

No encerramento manual, o motivo é obrigatório.

## 11. Aba Timeline de eventos

A timeline é o histórico geral do cliente no Pombo Correio.

Ela deve reunir eventos confiáveis do ERP e do sistema, em ordem cronológica.

### 11.1. Eventos do ERP

- cliente sincronizado;
- serviço realizado;
- agendamento criado;
- agendamento alterado;
- agendamento cancelado;
- dados cadastrais atualizados, quando relevante.

### 11.2. Eventos do Pombo Correio

- entrada em campanha;
- ação programada;
- mensagem enviada;
- falha no envio;
- participação encerrada;
- bandeira manual atribuída;
- bandeira manual removida.

Bandeiras automáticas podem ser recalculadas sem gerar eventos, evitando ruído.

### 11.3. Eventos não suportados no MVP

Não registrar como fatos confirmados:

- cliente respondeu no WhatsApp;
- cliente leu a mensagem;
- última interação do cliente;
- atendimento manual realizado fora do sistema.

### 11.4. Estrutura mínima do evento

Cada evento deve conter:

- ID interno;
- cliente;
- tipo;
- data e hora;
- origem;
- referência ao objeto relacionado, quando aplicável;
- dados necessários para apresentação.

## 12. Listagem de clientes

A listagem deve ser simples e focada em localizar clientes.

### 12.1. Colunas

Exibir:

- nome;
- último atendimento;
- próximo agendamento;
- bandeiras;
- permissão para campanhas automáticas.

Não exibir:

- telefone;
- código do ERP;
- última mensagem;
- campanha atual.

### 12.2. Busca

Permitir busca por:

- nome.

A busca por telefone e código do ERP não faz parte da listagem do MVP.

### 12.3. Filtros

Filtros recomendados:

- recebe ou não recebe campanhas automáticas;
- possui ou não possui agendamento futuro;
- bandeira automática;
- bandeira manual;
- período desde o último atendimento.

Não incluir filtro por telefone.

### 12.4. Ordenação

Permitir ordenar por:

- nome;
- último atendimento mais recente ou mais antigo;
- próximo agendamento.

## 13. Permissões administrativas

No MVP, o administrador deve poder:

- consultar qualquer cliente;
- ativar ou desativar o recebimento de campanhas automáticas na aba Dados;
- criar, editar e desativar bandeiras manuais;
- atribuir e remover bandeiras manuais;
- encerrar manualmente uma participação;
- consultar o motivo de todos os encerramentos.

A edição direta dos dados sincronizados não deve ser permitida.

## 14. Regras de negócio consolidadas

1. Todo cliente possui ID interno do Pombo Correio.
2. O identificador do ERP é apenas uma referência técnica.
3. O ERP é a fonte principal dos dados cadastrais e comerciais.
4. O sistema não trata duplicidades no MVP.
5. Dados vazios ou inválidos não substituem dados válidos.
6. Um serviço mais recente pode atualizar dados de contato.
7. O telefone deve ser normalizado e validado.
8. A situação do telefone aparece junto ao campo e não como bandeira.
9. A configuração de campanhas automáticas fica na aba Dados.
10. Cliente bloqueado não entra em novas campanhas e tem participações automáticas atuais encerradas.
11. Todo encerramento de participação registra e exibe um motivo.
12. O sistema não depende de respostas do WhatsApp.
13. Último atendimento e próximo agendamento ficam fixos fora das abas.
14. Bandeiras automáticas representam tempo desde o último atendimento ou existência de agendamento.
15. Bandeiras de tempo são mutuamente exclusivas.
16. Bandeiras manuais são gerenciadas pelo administrador.
17. Bandeiras podem ser usadas como filtros de campanhas.
18. A regra de reentrada pertence à campanha.
19. Clientes não possuem exclusão ou inativação no MVP.
20. Dados vindos do ERP são somente leitura.
21. A timeline inclui apenas eventos conhecidos de forma confiável.
22. A listagem não exibe telefone nem código do ERP.

## 15. Modelo conceitual de dados

### Cliente

- ID interno;
- ID no ERP;
- nome;
- telefone original;
- telefone normalizado;
- situação do telefone;
- e-mail;
- data de nascimento;
- permissão para campanhas automáticas;
- data da última sincronização;
- origem e data da última atualização cadastral.

### Atendimento

- ID interno;
- ID no ERP;
- cliente;
- serviço;
- data;
- status.

### Agendamento

- ID interno;
- ID no ERP;
- cliente;
- serviço, quando disponível;
- data;
- status.

### Bandeira

- ID;
- nome;
- tipo: automática ou manual;
- identificação visual;
- ativa.

### Participação em campanha

- ID;
- cliente;
- campanha;
- data de entrada;
- situação;
- etapa atual;
- próxima ação;
- data de encerramento;
- tipo de encerramento;
- motivo.

### Mensagem

- ID;
- cliente;
- campanha;
- ação;
- template;
- telefone utilizado;
- conteúdo processado;
- data programada;
- data de envio;
- situação;
- motivo de falha.

### Evento da timeline

- ID;
- cliente;
- tipo;
- origem;
- data e hora;
- referência relacionada;
- dados de apresentação.

## 16. Critérios de aceite do MVP

O módulo será considerado funcional quando:

1. clientes do ERP forem sincronizados com ID interno próprio;
2. a listagem exibir somente os campos definidos;
3. a busca da listagem funcionar por nome;
4. a ficha possuir resumo fixo, aba Dados, aba Campanhas e automações e aba Timeline;
5. último atendimento e próximo agendamento permanecerem visíveis fora das abas;
6. a aba Dados exibir os dados cadastrais definidos;
7. a configuração de campanhas automáticas estiver na aba Dados;
8. o telefone for normalizado e sua situação exibida;
9. bandeiras automáticas forem calculadas corretamente;
10. o administrador puder criar e atribuir bandeiras manuais;
11. a aba Campanhas e automações exibir participações atuais e encerradas;
12. todo encerramento exibir um motivo;
13. a última mensagem enviada e a próxima mensagem programada forem exibidas;
14. a timeline reunir os eventos suportados em ordem cronológica;
15. dados do ERP forem apresentados como somente leitura;
16. nenhuma funcionalidade depender de respostas recebidas no WhatsApp.

## 17. Fora do escopo do MVP

- edição principal do cadastro do ERP;
- deduplicação e mesclagem de clientes;
- exclusão, arquivamento ou inativação;
- notas internas;
- unidade e profissional;
- métricas financeiras;
- primeiro atendimento;
- contagem total de atendimentos;
- caixa de entrada do WhatsApp;
- leitura ou confirmação de resposta;
- busca e filtros por telefone na listagem;
- exibição de código do ERP na listagem;
- funil de vendas;
- tarefas comerciais;
- score avançado;
- relatórios analíticos avançados.

## 18. Decisões futuras

- prioridade técnica detalhada entre fontes cadastrais;
- validação da existência do número no WhatsApp;
- configuração dinâmica das faixas das bandeiras;
- regras avançadas de reentrada;
- aplicação diferenciada do bloqueio em campanhas manuais;
- repetição após falhas temporárias;
- paginação e retenção da timeline;
- inclusão de respostas caso a integração forneça dados confiáveis.
