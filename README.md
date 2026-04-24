# 🏗️ Documentação Técnica: Backend - Cidade de Refúgio

Esta documentação descreve a arquitetura de backend do projeto **Cidade de Refúgio**, detalhando os fluxos de dados, componentes e a infraestrutura baseada em AWS.

---

## 1. 🌟 Visão Geral
O sistema utiliza uma arquitetura **Serverless** e orientada a eventos para gerir o ciclo de vida dos dados dos alunos. A solução abrange desde a ingestão via API, processamento de documentos com OCR (Textract), armazenamento em Data Lake (S3) e bases NoSQL (DynamoDB), até a disponibilização de dados para Business Intelligence (Athena).

---

## 2. 🧩 Componentes da Arquitetura

### 2.1 🚪 Camada de Interface (Ingress)
- **🌐 API Gateway (`cidade_refugio_api`)**: Interface REST que comunica com o Frontend para operações de criação (POST), atualização (PUT) e consulta (GET).

### 2.2 ⚙️ Camada de Processamento (Compute)
- **⚡ AWS Lambda**:
  - `lbd-salva-documentos`: Responsável por receber e persistir ficheiros de imagem.
  - `lbd-cadastro-alunos`: Gere os dados cadastrais enviados via formulários.
  - `lbd-pertences`: Regista e consulta itens na tabela de pertences.
  - `lbd-gerador-pdf`: Cria documentos e declarações baseados em templates pré-definidos.
  - `lbd-leitura-documentos`: Orquestra a extração inteligente de dados.
  - `lbd-get-alunos`: Consolida informações de diversas fontes para exibição no cliente.

### 2.3 🧠 Camada de Inteligência e Extração
- **🔍 Amazon Textract**: Utilizado para extrair texto e dados estruturados de imagens de documentos de forma automatizada (*on-demand*).

### 2.4 📂 Camada de Armazenamento (Storage)
- **🪣 Amazon S3 (`cidade-refugio-data-lake`)**:
  - `raw/alunos/imagens`: Zona de landing para ficheiros brutos.
  - `process/alunos/imagens`: Imagens após processamento.
  - `alunos/pdf`: Armazenamento de documentos gerados e PDFs finais.
  - `SOT_alunos`: *Single Source of Truth* (Fonte Única de Verdade) para dados de alunos.
- **💾 Amazon DynamoDB**:
  - `tabela_pertences`: Armazenamento de alta performance para metadados de pertences.
  - `templates-PDF`: Repositório de esquemas para a geração dinâmica de documentos.

### 2.5 📊 Camada de Analytics e BI
- **🛠️ AWS Glue Job**: Processamento batch agendado para transformação de dados.
- **🕷️ AWS Glue Crawler**: Mapeia os esquemas de dados no S3 para o Data Catalog.
- **⚖️ Amazon Athena**: Permite a execução de consultas SQL diretamente sobre os dados armazenados no S3.

---

## 3. 🔄 Fluxos de Dados Principais

### 📥 Fluxo de Ingestão e OCR
1. O utilizador envia um documento via Frontend.
2. O **API Gateway** aciona a Lambda de salvamento.
3. O ficheiro é depositado no prefixo `raw` do **S3**.
4. A Lambda de leitura envia o ficheiro ao **Textract**.
5. O resultado estruturado é armazenado no Data Lake e disponibilizado para consulta.

### 📄 Fluxo de Geração de Documentos
1. Uma requisição de declaração é feita via API.
2. A Lambda `lbd-gerador-pdf` consome um template do **DynamoDB**.
3. Os dados do aluno são fundidos ao template.
4. O PDF resultante é guardado no **S3** e o link/ficheiro é retornado.

---

## 4. 📝 Análise e Observações Técnicas

- **🚀 Escalabilidade**: Sendo 100% Serverless, o sistema escala automaticamente conforme a procura, sem necessidade de gestão de servidores.
- **🛡️ Resiliência**: O uso de múltiplas zonas de armazenamento no S3 (raw, process, SOT) permite a reprocessagem de dados em caso de falha na lógica de negócio.
- **🔐 Segurança**: Recomenda-se a implementação de políticas de IAM restritas por prefixo de bucket e a utilização de encriptação em repouso (KMS) para dados sensíveis dos alunos.
- **💰 Custos**: O uso de Glue Jobs por agendamento ajuda no controlo de custos analíticos, enquanto o Textract é faturado apenas pelo uso efetivo.

---
