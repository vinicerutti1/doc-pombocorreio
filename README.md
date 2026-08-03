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

O módulo de Serviços representa o catálogo do ERP dentro do Pombo Correio.

Ele contempla:

- criação e atualização pelo relatório 0033;
- dados somente leitura;
- nome, descrição e categoria;
- ID interno do Pombo Correio;
- identificação pelo nome normalizado, enquanto o ERP não fornecer código estável;
- criação de serviços provisórios a partir de atendimentos;
- atualização posterior do provisório quando aparecer no catálogo;
- histórico de atendimentos;
- campanhas relacionadas com nome, status e atalho;
- uso de categorias em gatilhos e filtros de campanhas.

O valor do serviço não é utilizado pelo módulo.

A especificação funcional completa está disponível em [docs/modulo-servicos.md](docs/modulo-servicos.md).

## Campanhas

O módulo de Campanhas é o núcleo de automação do Pombo Correio.

Uma campanha possui uma ou mais regras independentes, estruturadas como:

```text
Evento
  ↓
Deslocamento de tempo, quando aplicável
  ↓
Ação
```

O módulo contempla:

- campanhas automáticas baseadas em serviço realizado ou agendamento;
- múltiplas regras na mesma campanha;
- envios antes ou depois de agendamentos;
- envios somente depois de serviços realizados;
- seleção de template em cada ação de envio;
- ações de encerramento por novo serviço ou novo agendamento;
- cancelamento das ações pendentes após o encerramento;
- motivo obrigatório em todo encerramento;
- configuração de reentrada por campanha;
- limite global de campanhas simultâneas por cliente;
- arquitetura preparada para disparo em massa.

A especificação funcional completa está disponível em [docs/modulo-campanhas.md](docs/modulo-campanhas.md).

## Templates de mensagem

O módulo de Templates mantém mensagens reutilizáveis para as ações das campanhas.

Ele contempla:

- listagem com nome e status;
- identificador interno único e imutável;
- editor de texto simples;
- atalhos para variáveis fixas;
- validação de tags e variáveis;
- preview em tempo real simulando o WhatsApp;
- vínculo das ações de campanhas pelo ID interno do template;
- preservação do conteúdo final das mensagens já enviadas;
- ativação e inativação sem exclusão definitiva.

A especificação funcional completa está disponível em [docs/modulo-templates.md](docs/modulo-templates.md).

## Integrações

### ERP

A integração com o ERP utiliza quatro relatórios, nesta ordem:

1. **0033 — Tabela de preços dos serviços**;
2. **0004 — Lista de dados de todos os clientes**;
3. **0031 — Serviços realizados no período**;
4. **0051 — Clientes com agendamentos**.

Depois das importações, o sistema consolida último atendimento, bandeiras, próximo agendamento e eventos de campanhas.

O ERP continua sendo a fonte principal dos dados comerciais.

A especificação completa está disponível em [docs/integracao-erp.md](docs/integracao-erp.md).

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

- comportamento ao inativar uma campanha em andamento;
- provedor e regras da integração com o WhatsApp;
- janela de horários permitida para envios;
- tratamento de remarcações;
- política de repetição após falhas temporárias;
- experiência de seleção de público no disparo em massa.

## Princípio do MVP

O MVP deve implementar apenas o necessário para responder às seguintes perguntas:

- Para qual cliente enviar?
- Qual evento iniciou a campanha?
- Quando enviar?
- Qual mensagem enviar?
- O envio funcionou?

Funcionalidades que não contribuam diretamente para esse fluxo devem ser avaliadas para versões futuras.
