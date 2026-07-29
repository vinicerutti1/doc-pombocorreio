# Módulo de Clientes

## 1. Visão geral

O módulo de Clientes é a central de consulta do relacionamento entre a empresa e cada cliente dentro do Pombo Correio.

Ele não tem como objetivo substituir o cadastro do ERP nem transformar o produto em um CRM comercial completo. O ERP continua sendo a fonte principal dos dados cadastrais, serviços realizados e agendamentos. O Pombo Correio mantém uma representação local desses dados para permitir consultas, filtros e automações de relacionamento.

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

Os dados comerciais e cadastrais são originados no ERP. O Pombo Correio não será a interface principal de manutenção desses dados.

Os campos sincronizados devem ser exibidos como somente leitura. Alterações cadastrais devem ser realizadas no ERP e refletidas no Pombo Correio após a sincronização.

### 2.2. O Pombo Correio possui identidade própria

Cada cliente deve possuir um identificador interno próprio no Pombo Correio.

O identificador do ERP deve ser armazenado como referência de integração, mas não deve substituir o identificador interno.

Estrutura conceitual mínima:

- `id`: identificador interno do Pombo Correio.
- `erp_customer_id`: identificador do cliente no ERP.

O sistema seguirá a base de clientes do ERP. Tratamento de duplicidades, mesclagem de cadastros e deduplicação automática não fazem parte do MVP.

### 2.3. O módulo não depende de respostas do WhatsApp

No MVP, o sistema não deve assumir que possui controle confiável sobre as respostas enviadas pelo cliente no WhatsApp.

Por isso:

- não existe campo de última resposta do cliente;
- não existe indicador de última interação recebida;
- respostas não são usadas como gatilho de parada;
- a timeline não deve registrar respostas que o sistema não consiga confirmar;
- o campo de relacionamento deve ser chamado de **última mensagem enviada**, e não de último contato.

## 3. Origem dos dados

O módulo de Clientes será alimentado por três conjuntos principais de dados vindos do ERP:

1. Cadastro de clientes.
2. Serviços realizados.
3. Agendamentos.

Cada fonte possui responsabilidades diferentes.

### 3.1. Cadastro de clientes

É a fonte principal para os dados cadastrais atuais.

Campos esperados, conforme disponibilidade no ERP:

- identificador do cliente no ERP;
- nome;
- telefone;
- e-mail;
- data de nascimento.

### 3.2. Serviços realizados

É a fonte do histórico de atendimentos e também pode trazer dados cadastrais mais recentes.

Sempre que um novo serviço realizado for importado, o sistema deve verificar se o registro contém informações de contato atualizadas. Isso é necessário porque o telefone ou outro dado pode ter sido corrigido no momento do atendimento.

A sincronização deve permitir que dados presentes em um serviço mais recente atualizem o cadastro local do cliente, desde que o valor seja válido e esteja presente.

Campos potencialmente atualizáveis:

- nome;
- telefone;
- e-mail.

O sistema deve armazenar internamente:

- data da última atualização cadastral;
- origem da última atualização cadastral;
- data do registro que originou a atualização, quando disponível.

Esses metadados são importantes para rastreabilidade técnica, mas não precisam ocupar espaço de destaque na tela principal.

### 3.3. Agendamentos

É a fonte dos compromissos futuros do cliente.

O módulo deve identificar o próximo agendamento válido do cliente e utilizá-lo para:

- exibição na ficha;
- criação de bandeira automática;
- aplicação de filtros em campanhas;
- execução de regras de parada, quando configuradas.

Agendamentos cancelados não devem ser considerados como próximos agendamentos ativos.

## 4. Regras de sincronização cadastral

### 4.1. Criação do cliente

Quando um cliente do ERP ainda não possuir vínculo com um cliente interno, o sistema deve criar um novo registro interno e associá-lo ao identificador do ERP.

### 4.2. Atualização do cliente

Quando o cliente já existir, os campos sincronizados devem ser atualizados conforme os dados recebidos.

A prioridade exata entre o relatório de cadastro e o relatório de serviços dependerá da disponibilidade de datas de atualização no ERP. Como regra funcional:

- o cadastro de clientes representa a fonte principal;
- um serviço mais recente pode atualizar dados de contato quando trouxer informações mais atuais;
- valores vazios não devem apagar valores válidos já armazenados;
- valores inválidos não devem substituir valores válidos;
- a origem e a data da atualização devem ser registradas internamente.

### 4.3. Ausência de exclusão ou inativação

O MVP não terá fluxo de exclusão, arquivamento ou inativação manual de clientes.

O cliente permanece disponível para consulta conforme os dados sincronizados e os históricos gerados pelo Pombo Correio.

## 5. Dados do cliente

### 5.1. Dados cadastrais sincronizados

A ficha do cliente deve apresentar:

- nome;
- código ou identificador no ERP;
- telefone;
- situação do telefone;
- e-mail;
- data de nascimento;
- data e hora da última sincronização.

Não fazem parte do MVP:

- unidade;
- profissional responsável;
- primeiro atendimento;
- quantidade de atendimentos;
- valor total gasto;
- ticket médio;
- notas internas.

### 5.2. Dados de relacionamento calculados

A ficha também deve apresentar informações derivadas dos dados sincronizados e das automações:

- último atendimento, com data e serviço realizado;
- próximo agendamento;
- última mensagem enviada pelo sistema;
- próxima mensagem programada;
- campanhas em andamento;
- campanhas encerradas;
- bandeiras automáticas;
- bandeiras manuais;
- permissão para receber campanhas automáticas.

## 6. Último atendimento e último serviço

Último atendimento e último serviço realizado representam partes do mesmo evento.

O sistema não precisa tratar essas informações como dois registros independentes. A interface deve exibir o evento mais recente com seus dados:

```text
Último atendimento
10/07/2026
Unha em Gel
```

Internamente, o registro do atendimento deve possuir ao menos:

- identificador interno;
- identificador do atendimento no ERP, quando disponível;
- cliente;
- serviço;
- data e hora;
- status necessário para determinar se o serviço foi efetivamente realizado.

## 7. Tratamento do telefone

O telefone é um dado crítico porque determina se o cliente pode receber mensagens.

### 7.1. Normalização

Antes de salvar ou atualizar o telefone, o sistema deve normalizar o valor, removendo diferenças de formatação como:

- espaços;
- parênteses;
- traços;
- caracteres não numéricos;
- prefixos duplicados.

A representação exibida pode ser formatada, mas o valor operacional deve permanecer normalizado.

### 7.2. Situações do telefone

O sistema deve conseguir distinguir, no mínimo:

- telefone disponível e aparentemente válido;
- telefone ausente;
- telefone com formato inválido;
- falha de envio registrada para o telefone.

Essas situações não serão representadas como bandeiras. Elas devem ser exibidas junto ao próprio campo de telefone.

Exemplos:

```text
(11) 99999-9999
Telefone disponível para envio
```

```text
Sem telefone cadastrado
```

```text
(11) 9999-999
Formato de telefone inválido
```

### 7.3. Validação antes do envio

Antes de incluir o cliente em uma ação de envio, o sistema deve verificar:

- existência do telefone;
- formato válido;
- permissão para campanhas automáticas;
- existência de alguma condição de parada da participação.

A validação de existência real de conta no WhatsApp dependerá das capacidades da integração escolhida.

### 7.4. Atualização por serviço realizado

Quando um serviço realizado trouxer um telefone mais recente e válido, esse telefone pode atualizar o cadastro local do cliente.

A atualização deve respeitar as regras gerais:

- não substituir um telefone válido por valor vazio;
- não substituir um telefone válido por um valor inválido;
- registrar a origem e a data da atualização;
- usar o telefone atualizado nos próximos envios ainda não executados.

Mensagens já enviadas devem preservar o telefone utilizado no momento do envio para fins de histórico.

## 8. Permissão para campanhas automáticas

Cada cliente deve possuir uma configuração simples:

**Receber campanhas automáticas: Sim ou Não.**

Essa configuração pertence ao Pombo Correio e não depende do ERP.

### 8.1. Quando habilitada

O cliente pode:

- entrar em novas campanhas automáticas, caso atenda aos gatilhos e filtros;
- continuar participando de campanhas já iniciadas;
- receber ações futuras programadas.

### 8.2. Quando desabilitada

O cliente:

- não pode entrar em novas campanhas automáticas;
- deve ter suas participações automáticas em andamento encerradas;
- não deve receber novas mensagens automáticas.

A configuração não exige motivo, justificativa, autor ou histórico específico de alteração.

Entretanto, quando a mudança encerrar uma participação em campanha, o encerramento da participação deve seguir a regra geral de rastreabilidade e registrar o motivo padronizado, por exemplo:

> Campanhas automáticas desativadas para o cliente.

### 8.3. Campanhas manuais

A aplicação dessa configuração a disparos manuais deve ser definida pelo módulo de Campanhas. Como princípio seguro, o MVP deve respeitar a configuração também nos disparos manuais, evitando mensagens indesejadas.

## 9. Bandeiras

Bandeiras servem para classificar clientes e facilitar consultas e filtros de campanha.

Existem dois tipos:

- automáticas;
- manuais.

### 9.1. Bandeiras automáticas

São calculadas pelo sistema e não podem ser atribuídas ou removidas manualmente.

No MVP, devem estar focadas em condições comerciais derivadas do último atendimento e do próximo agendamento.

Exemplos:

- Agendamento marcado.
- Sem atendimento há 30 dias.
- Sem atendimento há 90 dias.
- Sem atendimento há 180 dias.

Não devem ser usadas como bandeiras automáticas:

- cliente em campanha;
- telefone inválido;
- possível duplicidade;
- campanhas silenciadas;
- falha de envio.

### 9.2. Regra das faixas de tempo

As bandeiras relacionadas ao tempo desde o último atendimento devem ser mutuamente exclusivas.

Exemplo: um cliente sem atendimento há 112 dias deve possuir apenas a bandeira **Sem atendimento há 90 dias**, e não simultaneamente as bandeiras de 30 e 90 dias.

A regra deve considerar a faixa mais alta já alcançada e ainda não superada pela próxima faixa configurada.

Exemplo conceitual:

- 30 a 89 dias: Sem atendimento há 30 dias.
- 90 a 179 dias: Sem atendimento há 90 dias.
- 180 dias ou mais: Sem atendimento há 180 dias.

Os períodos podem ser fixos no MVP. Configuração dinâmica das faixas pode ser avaliada posteriormente.

### 9.3. Bandeira de agendamento

O cliente deve receber a bandeira automática **Agendamento marcado** quando possuir pelo menos um agendamento futuro válido.

A bandeira deve ser removida automaticamente quando:

- o agendamento for cancelado;
- o agendamento for concluído e não houver outro futuro;
- a data passar e o registro deixar de ser considerado futuro;
- a sincronização indicar que não existe mais agendamento ativo.

### 9.4. Bandeiras manuais

As bandeiras manuais são gerenciadas pelo administrador.

O administrador deve poder:

- criar uma bandeira;
- editar o nome;
- escolher uma identificação visual simples;
- desativar uma bandeira;
- atribuir uma bandeira a um cliente;
- remover uma bandeira de um cliente.

Exemplos:

- VIP.
- Prioritário.
- Cliente antigo.
- Atendimento especial.

As bandeiras manuais devem estar disponíveis como critérios de filtro nas campanhas.

Uma bandeira já utilizada não deve ser excluída definitivamente no MVP. Ela deve poder ser desativada, preservando os vínculos existentes e evitando inconsistências históricas.

### 9.5. Uso em campanhas

Campanhas podem utilizar bandeiras automáticas e manuais como filtros de entrada.

Exemplos:

- incluir clientes com a bandeira manual VIP;
- incluir clientes com a bandeira automática Sem atendimento há 90 dias;
- excluir clientes com agendamento marcado.

A regra de filtro pertence à campanha. O módulo de Clientes apenas disponibiliza as bandeiras e seus vínculos.

## 10. Campanhas na ficha do cliente

A ficha deve exibir as participações do cliente em campanhas, separando participações em andamento e encerradas.

### 10.1. Participações em andamento

Para cada participação ativa, exibir:

- nome da campanha;
- status da participação;
- etapa atual;
- quantidade total de ações;
- próxima ação;
- data e hora da próxima ação programada;
- opção de encerramento manual.

Exemplo:

```text
Campanha: Pós-venda Unha
Situação: Em andamento
Etapa atual: 2 de 3
Próxima ação: Lembrete de retorno
Programada para: 01/08/2026 às 10:00
```

### 10.2. Participações encerradas

Para cada participação encerrada, exibir:

- nome da campanha;
- data e hora de entrada;
- data e hora de encerramento;
- etapa em que foi encerrada;
- tipo de encerramento: manual ou automático;
- motivo do encerramento.

### 10.3. Reentrada na mesma campanha

A regra que determina se um cliente pode participar novamente da mesma campanha pertence à configuração da campanha.

O módulo de Clientes apenas deve refletir o histórico de todas as participações.

Possíveis configurações da campanha:

- não permitir nova participação;
- permitir nova participação após o encerramento anterior;
- permitir nova participação após um intervalo definido.

Para o MVP, recomenda-se inicialmente:

- não permitir nova participação; ou
- permitir nova participação após o encerramento anterior.

## 11. Encerramento de participação em campanha

Sempre que a participação de um cliente em uma campanha for encerrada, manual ou automaticamente, o motivo deve ser registrado e exibido.

Nenhum encerramento pode ocorrer sem rastreabilidade.

### 11.1. Dados obrigatórios

O encerramento deve registrar:

- campanha;
- cliente;
- tipo de encerramento: manual ou automático;
- motivo;
- data e hora;
- etapa ou ação em que a participação foi interrompida;
- usuário responsável, quando o encerramento for manual e essa informação estiver disponível.

### 11.2. Motivos automáticos

Exemplos de motivos padronizados:

- novo agendamento identificado;
- novo atendimento realizado;
- campanhas automáticas desativadas para o cliente;
- telefone inválido;
- falha definitiva no envio;
- campanha desativada ou encerrada;
- cliente deixou de atender aos critérios da campanha;
- regra de parada configurada acionada.

A lista definitiva de motivos deve acompanhar as regras de parada disponíveis no módulo de Campanhas.

### 11.3. Encerramento manual

Ao escolher a ação **Encerrar participação**, o usuário deve informar um motivo.

Motivos sugeridos:

- cliente solicitou interrupção;
- contato realizado por outro canal;
- campanha não se aplica mais;
- cadastro incorreto;
- outro.

Ao selecionar **Outro**, deve ser possível informar uma descrição complementar.

### 11.4. Exibição

Na ficha do cliente:

```text
Reativação 180 dias
Encerrada em 28/07/2026 às 14:32
Motivo: Novo agendamento identificado
```

Na timeline:

```text
28/07/2026 às 14:32
Participação encerrada automaticamente
Campanha: Reativação 180 dias
Motivo: Novo agendamento identificado
```

## 12. Mensagens

### 12.1. Última mensagem enviada

A ficha deve mostrar a última mensagem enviada pelo Pombo Correio.

Informações mínimas:

- data e hora;
- campanha;
- template utilizado;
- resultado do envio.

O conteúdo completo da mensagem pode ser exibido em detalhe ou na timeline.

### 12.2. Próxima mensagem programada

Quando houver uma ação futura ativa, a ficha deve exibir a próxima mensagem programada.

Informações mínimas:

- campanha;
- ação;
- template;
- data e hora previstas.

Antes do envio, o sistema deve revalidar:

- permissão para campanhas;
- telefone;
- estado da participação;
- regras de parada;
- variáveis obrigatórias do template.

### 12.3. Falhas de envio

Falhas devem ser registradas na timeline e associadas à mensagem.

Informações esperadas:

- data e hora;
- campanha;
- ação;
- telefone utilizado;
- motivo retornado pela integração;
- classificação como falha temporária ou definitiva, quando possível.

Uma falha definitiva pode encerrar a participação na campanha, desde que a campanha possua essa regra. Nesse caso, o motivo do encerramento deve ser exibido.

## 13. Timeline do cliente

A timeline é o histórico geral do cliente dentro do Pombo Correio.

Ela deve reunir eventos do ERP e eventos controlados pelo sistema, em ordem cronológica.

### 13.1. Eventos do ERP

- cliente sincronizado;
- serviço realizado;
- agendamento criado;
- agendamento alterado;
- agendamento cancelado;
- dados cadastrais atualizados, quando relevante.

### 13.2. Eventos do Pombo Correio

- entrada em campanha;
- ação programada;
- mensagem enviada;
- falha no envio;
- participação encerrada;
- bandeira manual atribuída;
- bandeira manual removida.

Bandeiras automáticas podem ser recalculadas sem gerar eventos na timeline, evitando excesso de ruído.

### 13.3. Eventos não suportados no MVP

Não registrar como fatos confirmados:

- cliente respondeu no WhatsApp;
- cliente leu a mensagem;
- última interação do cliente;
- atendimento manual realizado no WhatsApp fora do sistema.

Esses eventos só poderão ser adicionados futuramente se a integração fornecer dados confiáveis.

### 13.4. Estrutura mínima de evento

Cada evento deve conter:

- identificador interno;
- cliente;
- tipo;
- data e hora do evento;
- origem: ERP, Pombo Correio ou integração;
- referência ao objeto relacionado, quando aplicável;
- dados necessários para apresentação.

## 14. Listagem de clientes

A tela de listagem deve permitir localizar e filtrar clientes sem funcionar como um funil comercial.

### 14.1. Colunas recomendadas

- nome;
- telefone;
- último atendimento;
- próximo agendamento;
- bandeiras;
- permissão para campanhas.

### 14.2. Busca

Permitir busca por:

- nome;
- telefone;
- código do ERP.

### 14.3. Filtros

Filtros recomendados:

- possui ou não possui telefone;
- telefone válido ou inválido;
- recebe ou não recebe campanhas automáticas;
- possui agendamento futuro;
- bandeira automática;
- bandeira manual;
- período desde o último atendimento;
- campanha atual, quando necessário.

A presença em campanha pode existir como filtro operacional, mas não como bandeira automática do cliente.

### 14.4. Ordenação

Ordenações úteis:

- nome;
- último atendimento mais recente ou mais antigo;
- próximo agendamento;
- última mensagem enviada.

## 15. Estrutura sugerida da ficha

### 15.1. Cabeçalho

- nome;
- código do ERP;
- telefone e situação;
- e-mail;
- data de nascimento;
- permissão para campanhas;
- bandeiras.

### 15.2. Resumo de relacionamento

- último atendimento e serviço;
- próximo agendamento;
- última mensagem enviada;
- próxima mensagem programada.

### 15.3. Campanhas

- participações em andamento;
- participações encerradas;
- motivo dos encerramentos;
- ação de encerramento manual.

### 15.4. Timeline

Histórico cronológico de serviços, agendamentos, campanhas e mensagens.

### 15.5. Sincronização

Exibir de forma discreta:

```text
Dados sincronizados do ERP
Última atualização: 28/07/2026 às 14:30
```

## 16. Permissões administrativas

No MVP, o administrador deve poder:

- consultar qualquer cliente;
- ativar ou desativar o recebimento de campanhas automáticas;
- criar, editar e desativar bandeiras manuais;
- atribuir e remover bandeiras manuais;
- encerrar manualmente a participação de um cliente em uma campanha;
- consultar o motivo de todos os encerramentos.

A edição direta dos dados sincronizados do ERP não deve ser permitida.

## 17. Regras de negócio consolidadas

1. Todo cliente possui ID interno do Pombo Correio.
2. O identificador do ERP é uma referência externa.
3. O ERP é a fonte principal dos dados cadastrais e comerciais.
4. O sistema não trata duplicidades no MVP.
5. Dados vazios ou inválidos não substituem dados válidos durante a sincronização.
6. Um serviço mais recente pode atualizar dados de contato do cliente.
7. O telefone deve ser normalizado e validado.
8. A situação do telefone é exibida junto ao campo e não como bandeira.
9. O cliente pode permitir ou bloquear campanhas automáticas.
10. Cliente com campanhas bloqueadas não entra em novas campanhas e tem participações automáticas atuais encerradas.
11. O módulo não registra motivo para a configuração de bloqueio, apenas o valor da configuração.
12. Todo encerramento de participação em campanha registra e exibe um motivo.
13. O sistema não depende de respostas do WhatsApp no MVP.
14. Último atendimento e último serviço são exibidos como um único evento.
15. Bandeiras automáticas representam tempo desde o último atendimento ou existência de agendamento.
16. Bandeiras de tempo são mutuamente exclusivas.
17. Bandeiras manuais são criadas e gerenciadas pelo administrador.
18. Bandeiras automáticas e manuais podem ser usadas como filtros de campanhas.
19. A regra de reentrada pertence à campanha, não ao cliente.
20. Clientes não possuem fluxo de exclusão ou inativação no MVP.
21. Dados vindos do ERP são somente leitura na interface.
22. A timeline inclui apenas eventos conhecidos de forma confiável.

## 18. Modelo conceitual de dados

A implementação pode variar, mas o domínio precisa representar ao menos as entidades abaixo.

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
- data da última atualização cadastral;
- origem da última atualização cadastral.

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

### Vínculo cliente-bandeira

- cliente;
- bandeira;
- data de atribuição;
- origem automática ou manual.

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
- motivo do encerramento.

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

## 19. Critérios de aceite do MVP

O módulo será considerado funcional quando:

1. Clientes do ERP forem sincronizados com ID interno próprio.
2. A ficha exibir os dados cadastrais definidos.
3. Um serviço realizado puder atualizar dados de contato válidos.
4. O telefone for normalizado e sua situação for exibida.
5. O último atendimento e o próximo agendamento forem exibidos corretamente.
6. O administrador puder habilitar ou desabilitar campanhas automáticas por cliente.
7. Bandeiras automáticas forem calculadas conforme atendimento e agendamento.
8. O administrador puder criar e atribuir bandeiras manuais.
9. Bandeiras puderem ser utilizadas como filtros de campanha.
10. A ficha exibir campanhas em andamento e encerradas.
11. Todo encerramento de participação exibir um motivo.
12. A última mensagem enviada e a próxima mensagem programada forem exibidas.
13. A timeline reunir os eventos suportados em ordem cronológica.
14. Dados do ERP forem apresentados como somente leitura.
15. Nenhuma funcionalidade depender de respostas recebidas no WhatsApp.

## 20. Fora do escopo do módulo no MVP

- edição principal do cadastro do ERP;
- deduplicação e mesclagem de clientes;
- exclusão, arquivamento ou inativação manual;
- notas internas;
- unidade e profissional;
- métricas financeiras do cliente;
- primeiro atendimento;
- contagem total de atendimentos;
- caixa de entrada do WhatsApp;
- leitura ou confirmação de resposta do cliente;
- funil de vendas;
- tarefas comerciais;
- score avançado de cliente;
- relatórios analíticos avançados.

## 21. Decisões futuras

Os pontos abaixo podem ser definidos após o MVP:

- prioridade técnica detalhada entre fontes cadastrais do ERP;
- validação de existência do número no WhatsApp;
- configuração dinâmica das faixas das bandeiras automáticas;
- regras avançadas de reentrada em campanhas;
- aplicação diferenciada do bloqueio em campanhas manuais;
- políticas de repetição após falhas temporárias;
- paginação, volume e retenção da timeline;
- inclusão de respostas e leitura de mensagens caso a integração passe a fornecer dados confiáveis.
