# Part1

## HK - HashKeys
Para identificar se uma coluna pode ser uma **Hash Key (HK)** na modelagem **Data Vault**, você deve seguir alguns critérios fundamentais. Aqui estão as principais diretrizes para escolher uma coluna que pode ser usada como HK:

---

### 🔎 **Critérios para ser uma Hash Key (HK)**
1. **Identificador único e estável**  
   - Deve ser um **identificador natural** que represente unicamente uma entidade no sistema de origem.
   - Não deve mudar frequentemente (evitar colunas voláteis como e-mails ou telefones).

2. **Pertencer a um HUB**  
   - As Hash Keys são criadas para **Hubs** (entidades principais do modelo).
   - O HK de um **Hub** é gerado a partir da chave natural da entidade.

3. **Usada para relacionamento**  
   - Se a coluna será usada para vincular registros entre Hubs, ela pode ser usada como HK.
   - Exemplo: Um **HUB_CLIENTE** pode ter `HK_Cliente`, que será referenciado por um **LINK_CLIENTE_CONTRATO**.

4. **Vem de diferentes fontes e precisa ser padronizada**  
   - Se os dados vêm de múltiplos sistemas que usam chaves diferentes (`id_cliente`, `client_number`, `customer_code`), criar uma **HK única** evita problemas de integração.

5. **Evita chaves sequenciais do banco**  
   - Se a tabela tem uma **chave primária autoincremental**, **NÃO** use essa coluna diretamente como HK.  
   - Prefira colunas naturais e aplique um **hash** para garantir unicidade.

---

### ⚡ **Exemplo Prático: Identificando uma HK**
Imagine que temos uma tabela de **clientes** com as colunas:

| id_cliente | cpf        | nome  | email             |
|------------|-----------|-------|------------------|
| 1          | 12345678901 | João  | joao@email.com  |
| 2          | 98765432100 | Maria | maria@email.com |

#### **Qual coluna escolher para ser a Hash Key (HK)?**
- `id_cliente` ❌ (Gerado pelo banco, não é confiável entre fontes diferentes)
- `cpf` ✅ (Identificador natural único, pode ser usado como base para HK)

Gerando a Hash Key:
```sql
SELECT HASHBYTES('SHA1', cpf) AS HK_Cliente FROM Clientes;
```
Agora, `HK_Cliente` pode ser usado como chave primária do **HUB_CLIENTE** e para relacionar com **Satelites** e **Links**.

---

### 🚀 **Resumo: Como identificar uma coluna para HK**
✅ Deve ser **única e estável**  
✅ Normalmente vem da **chave natural** da entidade  
✅ É usada para **relacionar entidades no modelo**  
✅ Ajuda a integrar **dados de múltiplas fontes**  
✅ **Evita** chaves sequenciais do banco de dados  

## BK
O **BK (Business Key)** no Data Vault é o **dado original e sem hashear** que identifica de forma única uma entidade dentro do sistema de origem. Ele é usado como a chave natural da entidade e serve de referência para a geração da **HK (Hash Key)**.

---

## 🔹 Diferença entre **BK (Business Key) e HK (Hash Key)**
| Característica      | **BK (Business Key)** | **HK (Hash Key)** |
|---------------------|----------------------|--------------------|
| Tipo de dado       | Valor original (ex.: CPF, CNPJ, Código Cliente) | Valor hasheado (SHA-1, MD5, etc.) |
| Função             | Identifica a entidade no sistema de origem | Identifica a entidade dentro do Data Vault |
| Formato            | Texto, número, código, etc. | Hash binário ou string hexadecimal |
| Variação entre fontes | Pode ter variações (ex.: "00123" vs "123") | Unificado via hash |
| Usado para joins?  | Não recomendado (pode ter variações) | Sim, melhora a performance |

---

## 🔹 Exemplo Prático

### 🎯 **Tabelas com BK e HK**
Imagine que temos um sistema de **Clientes** com o seguinte dado:

| **BK (Business Key)** | **Nome**  |
|----------------------|--------|
| `12345678901`       | João   |
| `98765432100`       | Maria  |

Agora, no Data Vault, criamos o **HUB_CLIENTE**, gerando a **HK (Hash Key)** a partir do BK:

| **HK_Cliente (SHA-1 do BK)** | **BK_Cliente (CPF)** |
|-----------------------------|----------------------|
| `0x1A2B3C...`               | `12345678901`       |
| `0x9F8E7D...`               | `98765432100`       |

---

## 🔹 Como gerar o HK a partir do BK?

Em **SQL Server**, usando `HASHBYTES()`:
```sql
SELECT 
    HASHBYTES('SHA1', cpf) AS HK_Cliente,
    cpf AS BK_Cliente
FROM Clientes;
```

Em **Python**, usando `hashlib`:
```python
import hashlib

bk_cliente = "12345678901"
hk_cliente = hashlib.sha1(bk_cliente.encode()).hexdigest()

print(f"BK: {bk_cliente} -> HK: {hk_cliente}")
```

---

## 🔹 Quando usar BK e HK?
✅ **BK**: Serve para rastrear a chave original e entender de onde o dado veio.  
✅ **HK**: Serve para **relacionamentos internos no Data Vault**, garantindo eficiência e padronização.

## Hashdiff
No **Data Vault**, uma **hashdiff** é um hash gerado a partir de todas as colunas descritivas de uma **satélite**. Ele é usado para detectar mudanças nos dados de forma eficiente e evitar armazenar registros duplicados. 

### 📌 **Como funciona?**
1. **Criação**: O **hashdiff** é gerado utilizando uma função de hash (como `MD5`, `SHA1` ou `SHA256`) sobre todas as colunas não-chave do satélite.
2. **Comparação**: Quando um novo registro chega, seu hashdiff é comparado ao último hashdiff armazenado. Se forem diferentes, significa que houve uma alteração nos dados e um novo registro deve ser inserido.
3. **Otimização**: O hashdiff evita a necessidade de comparar cada coluna individualmente, tornando a detecção de mudanças mais eficiente.

### 📌 **Exemplo no `dbt`**
No `dbt`, um **hashdiff** pode ser criado usando o `hash()` ou `md5()` do SQL, como:

```sql
SELECT 
    hub_id,
    md5(concat_ws('|', col1, col2, col3)) AS hashdiff,
    col1,
    col2,
    col3,
    load_date
FROM raw_data
```

📌 **Benefícios do hashdiff no Data Vault:**
- Melhora a eficiência das comparações.
- Evita armazenar dados desnecessários.
- Reduz a necessidade de verificações linha a linha.

Se estiver usando `dbt` com `dbt_utils`, também pode usar `dbt_utils.generate_surrogate_key()`:

```sql
SELECT 
    hub_id,
    {{ dbt_utils.generate_surrogate_key(['col1', 'col2', 'col3']) }} AS hashdiff,
    col1,
    col2,
    col3,
    load_date
FROM raw_data
```

Isso ajuda a garantir que os valores sejam tratados corretamente, mesmo se forem `NULL`.

## Quais Colunas Devem Fazer Parte do Hashdiff?
Apenas os **atributos de negócio** devem ser incluídos no hashdiff. **Colunas de auditoria e controle não devem ser utilizadas.**

### ✅ **Incluídas no Hashdiff**
- Atributos descritivos e relevantes para a história do objeto modelado.
- Colunas que impactam a representação do dado na camada de negócio.

### ❌ **Excluídas do Hashdiff**
- **`dt_ref`**: Data de referência do dado. Ela pode estar presente na tabela, mas não deve ser usada no hashdiff.
- **`file_key`** e **`etag`**: Usadas apenas na Raw Layer para controle e auditoria.
- **`last_modified` (`load_datetime`)**: Timestamp da última modificação na fonte. Pode ser utilizada para controle, mas **não deve fazer parte do hashdiff**.

## Como Construir o Hashdiff no dbt?
No **dbt**, utilizamos a função `dbt_utils.generate_surrogate_key()` para gerar o hashdiff.

### 📌 **Exemplo de Implementação**
```sql
SELECT
    dt_ref,  -- Fica antes do hashdiff, mas não participa dele
    dbt_utils.generate_surrogate_key(ARRAY[
        tipo_instrumento_hk,
        tipo_instrumento_bk,
        emissor_hk,
        emissor_bk,
        apelido,
        ranking
    ]) AS hashdiff
FROM prep_raw.dados_fonte
```

### 🔹 **Explicação do Código:**
- A coluna **`dt_ref`** é mantida no conjunto de dados, mas **fora do hashdiff**.
- Os atributos descritivos são passados como array para `dbt_utils.generate_surrogate_key()`, que gera um **hash SHA256**.
- O hashdiff é calculado com base nos atributos do negócio para identificar mudanças no satélite.

## 4. Benefícios do Uso do Hashdiff
✅ **Detecção eficiente de mudanças** sem necessidade de comparar todas as colunas individualmente.  
✅ **Melhora a performance** nas consultas de versionamento.  
✅ **Garante integridade e rastreabilidade** ao histórico dos dados.  

