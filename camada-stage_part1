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

Se precisar de mais detalhes sobre a implementação no seu contexto, me avise! 🚀
