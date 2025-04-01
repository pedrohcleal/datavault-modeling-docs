# 🔍 **Camada Raw no Data Vault**  

A **Camada Raw** (ou **Raw Layer**) é a **primeira camada** no **Data Vault** e tem a função de armazenar os dados exatamente como vieram da fonte, sem transformações ou alterações.  

---

## 📌 **Principais Características**
✅ **Dados Brutos**: Os dados são armazenados no seu formato original, sem mudanças de estrutura ou valores.  
✅ **Auditável**: Mantém informações sobre a origem, data de ingestão e metadados para rastreamento.  
✅ **Imutável**: Dados não são atualizados ou sobrescritos—se houver mudanças, um novo registro é adicionado.  
✅ **Rastreável**: Cada dado pode ser ligado diretamente à sua origem, garantindo transparência.  

---

## 🏗 **Papel da Camada Raw no Data Vault**  
A **Raw Layer** serve como **ponte entre a fonte de dados e o Data Vault**. Sua principal função é garantir que qualquer erro ou inconsistência possa ser identificado e corrigido antes que os dados sejam utilizados para análise ou processamento.  

### 🔄 **Fluxo de Dados**  
1️⃣ Os dados são **extraídos** de diferentes fontes (APIs, bancos de dados, arquivos CSV, etc.).  
2️⃣ Eles são **armazenados na camada Raw** **sem modificações**, preservando sua estrutura original.  
3️⃣ **Metadados de auditoria** são adicionados para garantir rastreabilidade.  
4️⃣ Os dados são **carregados no Data Vault**, onde são modelados em **Hubs, Links e Satélites**.  

---

## 🛠 **Estrutura da Camada Raw**  
Os dados são armazenados em tabelas ou arquivos com informações adicionais para rastreamento.  

### 📁 **Exemplo de Tabela Raw**  
```sql
CREATE TABLE raw_meus_dados (
    file_key VARCHAR,         -- ID do arquivo/lote processado
    etag VARCHAR,             -- Código de versão (exemplo para APIs)
    dt_ref DATE,              -- Data de referência do dado
    load_datetime TIMESTAMP,  -- Data e hora do carregamento
    source_system VARCHAR,    -- Nome do sistema de origem
    processing_status VARCHAR, -- Status do processamento (SUCESSO, ERRO)
    data JSONB                -- Dados brutos em formato JSON
);
```

### 📜 **Exemplo de Logs de Ingestão**  
```sql
CREATE TABLE raw_ingestion_log (
    ingestion_id SERIAL PRIMARY KEY,
    file_key VARCHAR,
    source_system VARCHAR,
    load_datetime TIMESTAMP DEFAULT NOW(),
    record_count INT,
    error_message TEXT
);
```

---

## 🎯 **Por que a Camada Raw é Importante?**  
🔹 **Garantia de Rastreabilidade** → Podemos sempre verificar de onde um dado veio.  
🔹 **Facilidade de Auditoria** → Se houver erro no processamento, os dados brutos estão preservados.  
🔹 **Resiliência** → Permite reprocessamento de dados sem perder a integridade original.  
🔹 **Independência da Fonte** → Evita dependência de transformações feitas antes da ingestão.  

---

## 🚀 **Resumo**  
- A camada **Raw** é essencial no Data Vault para armazenar dados sem modificações.  
- Mantém **metadados de auditoria** para rastrear a origem e o status do carregamento.  
- **Evita perda de informações** e permite auditorias e reprocessamentos seguros.  

Se precisar de mais detalhes ou exemplos, me avise! 🚀


## Auditoria 

### 🔍 **Auditoria na Camada Raw (Raw Layer Audit)**  

A auditoria na camada **Raw** é o processo de garantir a **integridade, rastreabilidade e confiabilidade** dos dados logo após a sua ingestão, antes de serem transformados ou modelados no **Data Vault**.  

---

### 📂 **O que é a Camada Raw?**
A **Raw Layer** é a primeira camada do **Data Warehouse** ou **Data Lake**. Ela recebe os dados **exatamente como vieram da fonte**, sem transformação, para garantir rastreabilidade e auditoria.

---

### 📊 **Objetivos da Auditoria na Camada Raw**
1. **Garantir a integridade dos dados**  
   - Evitar dados corrompidos, truncados ou duplicados.  
   - Verificar se os dados carregados batem com a fonte original.  

2. **Monitorar o processo de ingestão**  
   - Registrar **quando**, **de onde** e **como** os dados foram carregados.  
   - Identificar falhas, latência e problemas de ingestão.  

3. **Permitir rastreabilidade completa**  
   - Armazenar metadados como `file_key`, `etag`, timestamps e hashes.  
   - Facilitar a reprocessamento e auditoria posterior.  

---

### 🛠 **Como fazer Auditoria na Camada Raw?**
1. **Adicionar colunas de metadados**  
   - `file_key`: Identifica o arquivo ou lote carregado.  
   - `etag`: Código único para verificar se os dados mudaram (geralmente em APIs).  
   - `load_datetime`: Timestamp do carregamento dos dados.  
   - `source_system`: Origem do dado (ex.: API, CSV, Banco X).  
   - `processing_status`: Status do processamento (`SUCESSO`, `ERRO`, etc.).  

2. **Registrar logs de carga**  
   - Criar tabelas de logs (`raw_ingestion_log`) com detalhes sobre arquivos e execução.  
   - Registrar erros e inconsistências.  

3. **Aplicar controles de qualidade**  
   - Contar registros carregados e comparar com a fonte.  
   - Validar integridade referencial.  

---

### 🎯 **Exemplo de Estrutura de Auditoria**
#### 📁 **Tabela Raw (com metadados de auditoria)**  
```sql
CREATE TABLE raw_meus_dados (
    file_key VARCHAR,         -- ID do arquivo/lote
    etag VARCHAR,             -- Identificador de versão
    dt_ref DATE,              -- Data de referência do dado
    load_datetime TIMESTAMP,  -- Data de carregamento
    source_system VARCHAR,    -- Fonte do dado
    processing_status VARCHAR, -- Status da carga (SUCESSO, ERRO)
    data JSONB                -- Dados originais (exemplo)
);
```
#### 📜 **Tabela de Logs de Auditoria**
```sql
CREATE TABLE raw_ingestion_log (
    ingestion_id SERIAL PRIMARY KEY,
    file_key VARCHAR,
    source_system VARCHAR,
    load_datetime TIMESTAMP DEFAULT NOW(),
    record_count INT,
    error_message TEXT
);
```

---

### 🏛 **Por que a Auditoria é Importante no Data Vault?**
- No **Data Vault**, confiamos que os dados da camada Raw são **completos e corretos**.  
- Se houver erro na ingestão, os **Hubs, Links e Satélites** podem ser impactados.  
- Manter a auditoria na camada Raw facilita a **rastreabilidade** e evita perda de dados.  

Se precisar de mais detalhes ou exemplos, me avise! 🚀
