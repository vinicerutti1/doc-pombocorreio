# Módulo de Serviços

## 1. Objetivo

O módulo de Serviços representa o catálogo de serviços do ERP dentro do Pombo Correio.

Ele não substitui o cadastro do ERP e não permite criação, edição ou exclusão manual de serviços. Sua função é:

- disponibilizar os serviços para consulta;
- permitir que serviços e categorias sejam usados em gatilhos e filtros de campanhas;
- consolidar o histórico de atendimentos associados a cada serviço;
- mostrar as campanhas relacionadas ao serviço;
- preservar os vínculos com atendimentos e campanhas mesmo quando o serviço for sincronizado posteriormente.

O ERP permanece como fonte de verdade para nome, descrição e categoria.

## 2. Fonte principal

O relatório **0033 — Tabela de preços dos serviços** é a fonte principal para criação e atualização do catálogo de serviços.

Embora o relatório possua valor, o Pombo Correio não utilizará nem exibirá o preço do serviço no módulo.

Campos utilizados no MVP:

- nome do serviço;
- descrição, quando preenchida;
- categoria.

Campos ignorados:

- valor;
- demais informações que não sejam necessárias para campanhas e consulta.

## 3. Identidade do serviço

### 3.1. ID interno

Cada serviço deve possuir um ID interno gerado pelo Pombo Correio.

Esse ID é usado para:

- associar atendimentos ao serviço;
- associar campanhas ao serviço;
- manter os vínculos caso dados do ERP sejam atualizados;
- evitar dependência direta do texto exibido na interface.

### 3.2. Ausência de identificador do ERP

Até o momento, o ERP não apresenta um identificador estável do serviço nos relatórios analisados.

Por isso, a identificação inicial será feita pelo nome normalizado.

Exemplo:

```text
" Unha em Gel "
"UNHA EM GEL"
"unha em gel"
```

Todos devem resultar na mesma chave normalizada:

```text
unha em gel
```

A normalização deve, no mínimo:

- remover espaços no início e no fim;
- reduzir espaços duplicados;
- padronizar letras maiúsculas e minúsculas;
- tratar caracteres equivalentes de forma consistente.

Não deve existir correspondência aproximada no MVP. Nomes diferentes, como `Alongamento Gel` e `Alongamento em Gel`, devem ser tratados como serviços distintos para evitar associações incorretas.

## 4. Regra de criação e atualização

### 4.1. Criação pelo relatório 0033

Quando um serviço do relatório 0033 ainda não existir:

1. gerar um ID interno;
2. salvar o nome original;
3. gerar a chave normalizada;
4. salvar a descrição, quando disponível;
5. salvar a categoria;
6. registrar a origem e a data da sincronização.

### 4.2. Atualização pelo relatório 0033

Quando o serviço já existir:

1. localizar pela chave normalizada;
2. atualizar nome, descrição e categoria com os dados do ERP;
3. preservar o ID interno;
4. preservar os atendimentos associados;
5. preservar as campanhas relacionadas;
6. registrar a data da última sincronização.

Nenhum desses dados poderá ser editado manualmente no Pombo Correio.

## 5. Serviço provisório

### 5.1. Quando criar

Um serviço provisório deve ser criado quando o relatório **0031 — Serviços realizados no período** apresentar um serviço que ainda não existe no catálogo sincronizado pelo relatório 0033.

### 5.2. Dados do provisório

O serviço provisório deve possuir:

- ID interno;
- nome recebido no atendimento;
- chave normalizada;
- indicação técnica de que é provisório;
- data de criação;
- origem do registro.

Categoria e descrição podem permanecer vazias até a sincronização do relatório 0033.

### 5.3. Atualização posterior

Quando um serviço do relatório 0033 corresponder à chave normalizada de um serviço provisório:

1. atualizar o mesmo registro;
2. preencher descrição e categoria;
3. substituir o nome pela versão oficial do ERP;
4. retirar a indicação de provisório;
5. preservar todos os atendimentos e campanhas já associados.

Não deve ser criado um segundo serviço nessa situação.

### 5.4. Divergência de nomes

Se o nome do provisório não corresponder exatamente após normalização ao nome do relatório 0033, os registros permanecerão separados no MVP.

A resolução manual de serviços duplicados ou equivalentes não faz parte do escopo inicial.

## 6. Listagem de serviços

A listagem deve apresentar:

- nome;
- categoria.

Recursos da listagem:

- busca por nome;
- filtro por categoria;
- ordenação por nome;
- paginação, caso necessária.

Não fazem parte da listagem:

- valor;
- filtro por campanha;
- edição;
- exclusão;
- ativação ou inativação manual.

## 7. Tela de detalhes

A tela de detalhes deve ser somente leitura e dividida em três áreas.

### 7.1. Dados do serviço

Exibir:

- nome;
- descrição, quando disponível;
- categoria;
- data da última sincronização.

Quando o serviço ainda for provisório, a interface pode informar que os dados completos ainda não foram encontrados no catálogo do ERP.

### 7.2. Histórico de atendimentos

Exibir os atendimentos associados ao serviço.

Cada registro deve mostrar:

- cliente;
- data do atendimento;
- situação do atendimento;
- identificador do atendimento no ERP, quando disponível.

Não exibir:

- valor;
- profissional;
- unidade.

Filtros mínimos do histórico:

- período;
- situação do atendimento;
- busca por cliente, quando necessário.

### 7.3. Campanhas relacionadas

Exibir apenas:

- nome da campanha;
- status;
- atalho para acessar a campanha.

Não é necessário mostrar na ficha do serviço o tipo de vínculo com a campanha.

## 8. Uso da categoria nas campanhas

As categorias sincronizadas do ERP devem ficar disponíveis para configuração de gatilhos e filtros de campanhas.

Exemplo:

```text
Gatilho: atendimento finalizado
Categoria: Manicure e Pedicure
```

Uma campanha configurada por categoria deve considerar todos os serviços atualmente associados àquela categoria.

Se um serviço mudar de categoria no ERP, a nova categoria deve ser usada nas avaliações futuras. Participações e eventos históricos não devem ser reescritos retroativamente.

## 9. Relação com campanhas

Um serviço pode estar relacionado a várias campanhas.

Uma campanha também pode utilizar vários serviços ou uma categoria inteira.

As regras de pós-venda, reativação, cross-sell e ofertas pertencem ao módulo de Campanhas, não ao cadastro de Serviços.

O módulo de Serviços fornece apenas os dados usados pelas regras.

## 10. Relação com atendimentos

O relatório 0031 alimenta o histórico de atendimentos dos serviços.

Ao importar um atendimento:

1. localizar o serviço pelo nome normalizado;
2. criar um serviço provisório quando não houver correspondência;
3. associar o atendimento ao ID interno do serviço;
4. preservar a associação em sincronizações futuras;
5. gerar eventos de campanha somente quando a situação do atendimento for válida para isso.

## 11. Modelo conceitual

```mermaid
erDiagram
    SERVICE {
        uuid id PK
        string name
        string normalized_name UK
        string description
        string category
        boolean provisional
        datetime created_at
        datetime last_synced_at
    }

    SERVICE_RECORD {
        uuid id PK
        uuid service_id FK
        uuid customer_id FK
        string erp_record_id
        datetime performed_at
        string status
    }

    CAMPAIGN_SERVICE {
        uuid campaign_id FK
        uuid service_id FK
    }

    SERVICE ||--o{ SERVICE_RECORD : possui
    SERVICE ||--o{ CAMPAIGN_SERVICE : relacionado
```

## 12. Fluxo de sincronização

```mermaid
flowchart TD
    R0033[Relatório 0033 - Tabela de preços] --> NORMALIZAR[Normalizar nome]
    NORMALIZAR --> LOCALIZAR{Serviço encontrado?}
    LOCALIZAR -->|Não| CRIAR[Criar serviço]
    LOCALIZAR -->|Sim, provisório| COMPLETAR[Completar serviço provisório]
    LOCALIZAR -->|Sim, definitivo| ATUALIZAR[Atualizar dados do ERP]
    CRIAR --> SALVAR[(Catálogo de serviços)]
    COMPLETAR --> SALVAR
    ATUALIZAR --> SALVAR

    R0031[Relatório 0031 - Serviços realizados] --> BUSCAR{Serviço existe?}
    BUSCAR -->|Sim| VINCULAR[Vincular atendimento]
    BUSCAR -->|Não| PROVISORIO[Criar serviço provisório]
    PROVISORIO --> VINCULAR
    VINCULAR --> HISTORICO[Atualizar histórico]
```

## 13. Regras de negócio

1. O relatório 0033 cria e atualiza o catálogo oficial.
2. O valor do serviço é ignorado.
3. Nenhum dado sincronizado pode ser editado no Pombo Correio.
4. Cada serviço possui um ID interno.
5. A identificação externa é feita pelo nome normalizado enquanto não existir código estável no ERP.
6. Correspondência aproximada não será usada no MVP.
7. Serviços ausentes no catálogo podem ser criados provisoriamente a partir de atendimentos.
8. O serviço provisório deve ser atualizado, e não duplicado, quando aparecer no relatório 0033.
9. O histórico de atendimentos deve permanecer associado ao mesmo ID interno.
10. Categoria poderá ser usada em gatilhos e filtros de campanhas.
11. O módulo não contém regras de cross-sell ou automação.
12. Campanhas relacionadas devem mostrar nome, status e atalho.

## 14. Critérios de aceite

O módulo será considerado funcional quando:

1. importar os serviços do relatório 0033;
2. ignorar o valor do serviço;
3. impedir edição manual dos dados sincronizados;
4. localizar serviços por nome normalizado;
5. criar serviços provisórios para atendimentos sem correspondência;
6. completar serviços provisórios em sincronizações posteriores;
7. preservar vínculos com atendimentos e campanhas;
8. listar serviços com busca por nome e filtro por categoria;
9. exibir dados, histórico de atendimentos e campanhas relacionadas;
10. disponibilizar categoria para uso futuro em campanhas;
11. permitir acesso direto da ficha do serviço à campanha relacionada.

## 15. Fora do escopo do MVP

- edição manual de serviços;
- criação manual de serviços;
- exclusão de serviços;
- ativação ou inativação manual;
- uso ou exibição de valores;
- histórico de preços;
- mesclagem de serviços duplicados;
- correspondência aproximada de nomes;
- regras de cross-sell dentro do serviço;
- filtro por campanha na listagem;
- relatórios analíticos avançados.