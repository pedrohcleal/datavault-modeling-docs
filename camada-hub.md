Na modelagem **Data Vault**, os **Hubs** são um dos três componentes principais, ao lado dos **Links** e **Satélites**. Eles são fundamentais para organizar os dados de forma a permitir a escalabilidade e flexibilidade na construção de data warehouses. 

### Definição de Hubs:
- **Hubs** representam os **nós de negócio** principais do sistema, geralmente associados às **entidades de negócios** que são de interesse na organização.
- Eles contêm informações **únicas e identificadoras** sobre essas entidades, como **chaves naturais** (ex.: números de contas, IDs de clientes, IDs de produtos, etc.).
- O objetivo de um Hub é fornecer uma estrutura consistente e centralizada para que outras informações (como os Links e Satélites) possam se referenciar de maneira eficiente.
  
### Características dos Hubs:
1. **Chave única**: Cada Hub contém uma chave única (geralmente um identificador natural) que permite identificar de forma exclusiva uma instância de uma entidade de negócio.
   
2. **Atributos estáticos**: Os Hubs normalmente armazenam apenas a chave da entidade e metadados relacionados, como:
   - Chave natural (ex.: ID do cliente).
   - Identificador gerado automaticamente (geralmente um **hash** ou **surrogate key**).
   - Atributos de rastreamento, como a **data de carga** (quando o dado foi inserido).
   
3. **Imutabilidade**: Os Hubs são imutáveis após sua criação. Ou seja, uma vez que uma chave é inserida, ela não muda ao longo do tempo, mesmo que os dados associados a essa chave no futuro mudem.

4. **Referência a Links e Satélites**: Eles são usados para estabelecer relacionamentos com outras tabelas:
   - **Links** conectam Hubs (e podem também conter informações sobre os relacionamentos entre diferentes entidades).
   - **Satélites** armazenam as características ou atributos descritivos de um Hub, podendo ser alterados com o tempo (por exemplo, endereços de clientes, status de contas, etc.).

### Exemplo:
Imaginando uma empresa que possui um sistema de clientes, produtos e vendas:
- O **Hub de Clientes** conteria um identificador único de cliente (ID do cliente), além de informações de controle, como a data de carga.
- O **Hub de Produtos** teria um identificador único de produto.
- O **Hub de Vendas** teria um identificador único para cada venda, e assim por diante.

### Vantagens dos Hubs:
- **Consistência e simplicidade**: Eles ajudam a centralizar as chaves de negócio em um único local, o que facilita o acesso e a rastreabilidade.
- **Escalabilidade**: Hubs permitem que novas entidades sejam facilmente adicionadas ao modelo de dados, sem a necessidade de reformular outras partes do sistema.
- **Facilidade de integração de dados**: Como os Hubs guardam apenas identificadores, eles são agnósticos em relação a fontes de dados e facilitam a integração de diferentes sistemas, mantendo a integridade dos dados.

Em resumo, os Hubs na modelagem Data Vault são cruciais para criar uma estrutura flexível e escalável, que permita o armazenamento eficiente de dados históricos enquanto mantém um design claro e organizado.

### Exemplos

Aqui estão alguns exemplos práticos de Hubs dentro de um modelo **Data Vault**:

### 1. **Hub de Clientes**

Imaginemos que estamos modelando os dados de uma empresa que possui um sistema de CRM. O Hub de Clientes seria responsável por armazenar as informações principais sobre cada cliente.

#### Estrutura do Hub de Clientes:
| **cliente_id (Hash Key)** | **cliente_id_natural (ID Cliente)** | **data_carga**   |
|---------------------------|-------------------------------------|------------------|
| 1                         | 1001                                | 2025-03-17      |
| 2                         | 1002                                | 2025-03-17      |
| 3                         | 1003                                | 2025-03-17      |

- **cliente_id (Hash Key)**: Chave única gerada para o Hub (pode ser um valor gerado a partir de um hash do `cliente_id_natural`).
- **cliente_id_natural (ID Cliente)**: A chave natural ou o identificador real do cliente na origem dos dados.
- **data_carga**: Data em que o cliente foi inserido no Data Warehouse.

### 2. **Hub de Produtos**

Agora, imagine que a empresa também tem um Hub de Produtos, que representa os itens vendidos.

#### Estrutura do Hub de Produtos:
| **produto_id (Hash Key)** | **produto_id_natural (ID Produto)** | **data_carga**   |
|---------------------------|-------------------------------------|------------------|
| 1                         | 2001                                | 2025-03-17      |
| 2                         | 2002                                | 2025-03-17      |
| 3                         | 2003                                | 2025-03-17      |

- **produto_id (Hash Key)**: Chave única gerada para o Hub (também pode ser um hash do `produto_id_natural`).
- **produto_id_natural (ID Produto)**: Chave natural ou identificador real do produto na origem dos dados.
- **data_carga**: Data em que o produto foi inserido no Data Warehouse.

### 3. **Hub de Vendas**

O Hub de Vendas pode conter os identificadores das transações realizadas pela empresa.

#### Estrutura do Hub de Vendas:
| **venda_id (Hash Key)** | **venda_id_natural (ID Venda)** | **data_carga**   |
|-------------------------|---------------------------------|------------------|
| 1                       | 3001                            | 2025-03-17      |
| 2                       | 3002                            | 2025-03-17      |
| 3                       | 3003                            | 2025-03-17      |

- **venda_id (Hash Key)**: Chave única gerada para o Hub.
- **venda_id_natural (ID Venda)**: Identificador natural da venda.
- **data_carga**: Data em que a venda foi inserida no Data Warehouse.

### 4. **Exemplo de Hub com Links e Satélites**

Vamos também criar um exemplo onde o **Hub de Vendas** se conecta a um **Hub de Clientes** através de um **Link** e armazena informações descritivas (como os detalhes da venda) em um **Satélite**.

#### Link de Vendas e Clientes (relacionando Hubs):
| **link_id (Hash Key)** | **venda_id (Hash Key)** | **cliente_id (Hash Key)** | **data_carga**   |
|------------------------|-------------------------|---------------------------|------------------|
| 1                      | 1                       | 1                         | 2025-03-17      |
| 2                      | 2                       | 2                         | 2025-03-17      |
| 3                      | 3                       | 3                         | 2025-03-17      |

- **link_id (Hash Key)**: Chave única gerada para o Link entre os Hubs.
- **venda_id (Hash Key)**: Chave estrangeira para o Hub de Vendas.
- **cliente_id (Hash Key)**: Chave estrangeira para o Hub de Clientes.

#### Satélite de Detalhes de Vendas (informações descritivas):

| **venda_id (Hash Key)** | **produto_id (Hash Key)** | **quantidade** | **valor_total** | **data_carga**   |
|-------------------------|---------------------------|----------------|-----------------|------------------|
| 1                       | 1                         | 2              | 100.00          | 2025-03-17      |
| 2                       | 2                         | 1              | 50.00           | 2025-03-17      |
| 3                       | 3                         | 3              | 150.00          | 2025-03-17      |

- **venda_id (Hash Key)**: Chave estrangeira para o Hub de Vendas.
- **produto_id (Hash Key)**: Chave estrangeira para o Hub de Produtos.
- **quantidade**: A quantidade do produto na venda.
- **valor_total**: O valor total da venda.
- **data_carga**: Data em que os detalhes da venda foram inseridos no Data Warehouse.

### Resumo:

- **Hubs** armazenam entidades centrais com suas chaves e metadados.
- **Links** conectam diferentes Hubs, representando os relacionamentos entre as entidades.
- **Satélites** armazenam os atributos ou detalhes históricos relacionados às entidades e seus relacionamentos.

Esses exemplos demonstram como os Hubs são usados para organizar dados centrais e como Links e Satélites complementam esse modelo, permitindo um Data Warehouse flexível e escalável.
