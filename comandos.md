# Comandos do DBT

## Introdução
Esta documentação fornece um guia básico sobre os comandos utilizados no DBT (Data Build Tool) para modelagem e execução de dados.

## Comandos básicos

### 1. Limpar diretórios temporários
```bash
dbt clean
```
Este comando remove diretórios temporários criados pelo DBT, como `target` e `dbt_packages`, garantindo um ambiente limpo antes de iniciar o processo.

### 2. Instalar dependências
```bash
dbt deps
```
Este comando baixa e instala as dependências do projeto definidas no arquivo `packages.yml`.

### 3. Gerar a documentação
```bash
dbt docs generate
```
Este comando gera a documentação do projeto DBT com base nos modelos definidos.

### 4. Processar manifest
```bash
python manifest_treatment.py
```
Este comando não faz parte diretamente do DBT, mas pode ser usado para processar o arquivo `manifest.json` gerado pelo DBT.

## Compilação e Execução

### 5. Compilar os modelos
```bash
dbt compile
```
Este comando compila os modelos do DBT e verifica se há erros de sintaxe ou referências inválidas.
Se houver erros, será necessário corrigi-los antes de seguir para a execução.

### 6. Executar um modelo específico
```bash
dbt run --select nome_do_model
```
Este comando executa um modelo específico do DBT, materializando os dados no banco.

### 7. Validar no banco de dados
Após a execução, é recomendado verificar no DBeaver se a tabela foi criada conforme esperado.

### 6. Executar um modelo específico
```bash
dbt run --full-refresh --select tag:dev_fiduciarios_oliveiratrust
```

```bash
dbt test --select tag:dev_fiduciarios_oliveiratrust
```

## Considerações finais
O processo de execução dos modelos no DBT pode ser considerado uma operação "em produção", pois impacta diretamente os dados do banco. Portanto, é importante validar a sintaxe e os modelos antes de executar comandos que modifiquem os dados.


