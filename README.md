# Pombo Correio

Plataforma de automação de relacionamento com clientes integrada ao ERP.

O Pombo Correio não tem como objetivo ser um CRM completo. O foco é automatizar ações de pós-venda, follow-up, reativação, cross-sell e ofertas com base nos clientes, serviços e atendimentos registrados no ERP.

## Objetivo do produto

Permitir que a empresa mantenha contato com seus clientes de forma automática, usando eventos do negócio como gatilhos para o envio de mensagens.

Exemplo:

1. Um atendimento de **Unha** é finalizado.
2. O cliente entra em uma campanha de pós-venda.
3. Após 1 dia, o sistema envia uma mensagem usando um template configurado.
4. Após 15 dias, o sistema pode enviar uma segunda mensagem.

## Escopo inicial

O produto será organizado nos seguintes módulos:

- Clientes
- Serviços
- Campanhas
- Templates de mensagem
- Integrações

O módulo de conversas dentro do sistema é opcional e não faz parte do núcleo inicial do produto.

## Clientes

O módulo de Clientes funciona como a central de consulta do relacionamento de cada cliente, sem substituir o cadastro principal do ERP.

Ele contempla:

- dados cadastrais sincronizados do ERP;
- atualização de contato a partir de serviços realizados mais recentes;
- tratamento e validação do telefone;
- último atendimento e próximo agendamento;
- última mensagem enviada e próxima mensagem programada;
- permissão individual para campanhas automáticas;
- bandeiras automáticas e manuais;
- campanhas em andamento e encerradas;
- motivo obrigatório em todo encerramento de participação;
- timeline de eventos confiáveis do ERP e do Pombo Correio.

A especificação funcional completa está disponível em [docs/modulo-clientes.md](docs/modulo-clientes.md).

## Serviços

O módulo de serviços deve permitir:

### Cadastro

- Cadastro e edição do serviço.
- Identificação do serviço correspondente no ERP.
- Ativação e desativação do serviço.

### Histórico de atendimentos

Exibir os atendimentos relacionados ao serviço.

### Histórico de campanhas

Exibir todas as campanhas relacionadas ao serviço, incluindo o status de cada campanha.

Ao abrir uma campanha dentro do histórico do serviço, deve ser possível consultar a lista de clientes impactados por ela.

## Campanhas

Campanhas concentram tanto os disparos manuais quanto os fluxos automáticos. Uma automação é tratada como uma campanha que possui um gatilho e uma ou mais ações programadas.

### Informações gerais

Cada campanha deve possuir:

- Nome.
- Descrição opcional.
- Status: ativa ou inativa.
- Serviço relacionado, quando aplicável.

### Gatilho

O gatilho define quando o cliente entra na campanha.

Exemplo:

> Atendimento do serviço **Unha** finalizado.

Outros gatilhos poderão ser adicionados conforme a evolução do produto, como cliente sem atendimento há determinado período ou compra de um serviço específico.

### Ações

Uma campanha pode possuir uma ou mais ações.

Cada ação deve possuir:

- Tempo de espera após o gatilho ou após a ação anterior.
- Template de mensagem selecionado.
- Ordem de execução.

Exemplo:

1. Enviar o template **Pós-venda Unha** após 1 dia.
2. Enviar o template **Retorno Unha** após 15 dias.

### Ativação e desativação

- Uma campanha ativa pode receber novos clientes e executar suas ações.
- Uma campanha inativa não deve receber novos clientes nem executar novos disparos.
- O comportamento dos clientes que já estavam em uma campanha no momento da desativação será definido posteriormente.

### Histórico e resultados

Cada campanha deve permitir consultar:

- Status da campanha.
- Clientes impactados.
- Ações programadas para cada cliente.
- Mensagens enviadas.
- Falhas de envio.

Métricas e relatórios mais avançados não fazem parte do MVP inicial.

### Regras de parada

Será necessário definir regras para impedir mensagens inadequadas ou duplicadas.

Possíveis regras:

- Parar as próximas ações quando o cliente realizar um novo atendimento.
- Parar manualmente a participação de um cliente.
- Evitar que o mesmo evento inclua o cliente duas vezes na mesma campanha.

A interrupção automática quando o cliente responder dependerá da capacidade da integração com o WhatsApp e será detalhada posteriormente.

## Templates de mensagem

O módulo de templates deve permitir:

- Cadastro e edição de mensagens reutilizáveis.
- Ativação e desativação de templates.
- Uso de variáveis do cliente, serviço e atendimento.
- Seleção de um template em cada ação de campanha.

Exemplos de variáveis:

- Nome do cliente.
- Nome do serviço.
- Data do atendimento.
- Nome da empresa.

## Integrações

### ERP

A integração com o ERP deve fornecer ao Pombo Correio os dados necessários para o funcionamento das campanhas a partir de três fontes principais:

- cadastro de clientes;
- serviços realizados;
- agendamentos.

Ela também é responsável por atualizar cadastros, normalizar dados de contato, consolidar último atendimento e próximo agendamento, gerar eventos para campanhas e registrar falhas de sincronização.

O ERP continua sendo a fonte principal dos dados comerciais.

A especificação completa da integração está disponível em [docs/integracao-erp.md](docs/integracao-erp.md).

### WhatsApp

A integração com o WhatsApp será responsável pelo envio das mensagens configuradas nas campanhas.

Inicialmente, o representante poderá continuar utilizando o mesmo número diretamente no WhatsApp para conversar com o cliente. Uma caixa de entrada de conversas dentro do Pombo Correio é considerada uma funcionalidade opcional e futura.

## Fora do escopo inicial

- CRM comercial completo.
- Funil de vendas.
- Dashboard geral.
- Tela de configurações complexa.
- Notas internas.
- Caixa de entrada de conversas.
- Múltiplos canais de comunicação.
- Inteligência artificial.
- Relatórios avançados.

## Pontos em aberto

Os seguintes itens serão detalhados durante a evolução da documentação:

- Campos completos do cadastro de serviços.
- Lista definitiva de gatilhos disponíveis.
- Regras de parada das campanhas.
- Comportamento ao desativar uma campanha em andamento.
- Provedor e regras da integração com o WhatsApp.
- Status possíveis para campanhas, ações e mensagens.
- Regras de reentrada de clientes em campanhas.

## Princípio do MVP

O MVP deve implementar apenas o necessário para responder às seguintes perguntas:

- Para qual cliente enviar?
- Qual evento iniciou a campanha?
- Quando enviar?
- Qual mensagem enviar?
- O envio funcionou?

Funcionalidades que não contribuam diretamente para esse fluxo devem ser avaliadas para versões futuras.
