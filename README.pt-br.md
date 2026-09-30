# Pipeline de Dados PDF + IA com n8n

<p align="center">
  🇧🇷 <strong>Português</strong> &nbsp;|&nbsp;
  🇺🇸 <a href="README.md">English</a>
</p>

## Visão geral

Um **pipeline automatizado de extração de dados de PDF e ETL**, desenvolvido com **n8n**, **Anthropic Claude**, **Google Drive** e **Google Sheets**.

O workflow lê um relatório em PDF, localiza a tabela desejada, extrai os registros estruturados com IA, valida o JSON retornado, transforma os registros para uma estrutura padronizada, verifica se o relatório já foi processado e carrega os dados validados no Google Sheets.

O projeto demonstra uma aplicação prática de **processamento de documentos com IA, ETL, qualidade de dados, deduplicação e automação de workflows**.

## Arquitetura

```text
Google Drive
     │
     ▼
Documento PDF
     │
     ▼
Verificação de duplicidade
     │
     ├── DUPLICATE ──► Notificação
     │
     └── NEW
          │
          ▼
   Anthropic Claude
          │
          ▼
    Validação dos dados
          │
          ├── FAILED ──► Notificação de erro
          │
          └── PASSED
                │
                ▼
       Transformação dos registros
                │
                ▼
          Google Sheets
                │
                ▼
       Notificação de processamento
```

## Tecnologias

- **n8n** — orquestração do workflow
- **Anthropic Claude** — extração de dados do PDF com IA
- **Google Drive** — origem dos documentos
- **Google Sheets** — destino dos dados
- **JavaScript** — validação e transformação
- **JSON** — contrato de dados estruturados
- ETL
- Validação de qualidade dos dados
- Deduplicação

## Extração dos dados

A etapa de IA localiza a seção **"2. Registros de Atendimento"** do PDF e retorna:

```json
{
  "atendimentos": [
    {
      "cnes": "...",
      "placa": "...",
      "tipologia": "...",
      "compet": "...",
      "uf": "...",
      "municipio": "...",
      "oci": "...",
      "quantidade": 0
    }
  ]
}
```

Cada linha da tabela representa exatamente um registro de atendimento.

O prompt de extração instrui o modelo a:

- Extrair todas as linhas.
- Não agrupar registros.
- Não realizar cálculos.
- Não fazer inferências.
- Preservar os valores encontrados no PDF.
- Retornar somente a estrutura JSON esperada.

## Validação dos dados

O node `VALIDATE — Data Quality` verifica a resposta da IA antes que os dados continuem pelo pipeline.

São verificados:

- JSON válido.
- Estrutura do objeto.
- Existência do array `atendimentos`.
- Campos obrigatórios.
- Campos vazios.
- Tipo e valor da quantidade.
- Campos inesperados.
- Existência de pelo menos um registro extraído.

Isso cria uma camada de validação entre a extração realizada pela IA e o carregamento dos dados.

## Transformação dos dados

Após a validação, o workflow cria um item do n8n para cada registro de atendimento.

A estrutura normalizada contém:

```text
cnes
placa
tipologia
compet
uf
municipio
oci
qtd
atendimento
id_relatorio
```

Metadados técnicos também podem ser adicionados aos registros de processamento.

## Deduplicação

Antes de enviar o PDF para a etapa de extração com IA, o workflow verifica se o relatório já foi registrado.

### Novo documento

```text
NEW
 ↓
Processar PDF
 ↓
Extrair registros
 ↓
Validar
 ↓
Transformar
 ↓
Carregar no Google Sheets
```

### Documento já processado

```text
DUPLICATE
 ↓
Ignorar processamento
 ↓
Enviar notificação
```

Isso evita o processamento repetido do mesmo relatório.

## Tratamento de erros

Caso a resposta da IA não atenda ao contrato de dados esperado, o workflow direciona a execução para o caminho de erro da validação em vez de carregar registros inválidos.

O workflow separa claramente:

```text
Extração
    ↓
Validação
    ↓
Transformação
    ↓
Carregamento
```

## Estrutura do repositório

```text
n8n-pdf-ai-data-pipeline/
│
├── workflow/
│   └── pdf-ai-extraction.json
│
├── screenshots/
│
├── README.md
├── README.pt-br.md
└── .gitignore
```

## Configuração

1. Importe `workflow/pdf-ai-extraction.json` no n8n.
2. Configure as credenciais do Google Drive.
3. Configure as credenciais do Google Sheets.
4. Configure a credencial da Anthropic.
5. Configure o Gmail caso as notificações por e-mail sejam utilizadas.
6. Substitua os IDs e o endereço de e-mail de exemplo pelos seus valores.
7. Revise as colunas do Google Sheets.
8. Teste o workflow com um PDF de exemplo.
9. Teste os cenários `NEW` e `DUPLICATE`.
10. Ative o workflow somente após validar o processamento.

> O workflow incluído neste repositório é uma versão sanitizada para portfólio. Credenciais reais, documentos privados e identificadores pessoais de contas foram intencionalmente removidos.

## Contexto de portfólio

Este projeto demonstra conhecimentos práticos em:

- Data Analytics
- Business Intelligence
- ETL
- Qualidade de dados
- Processamento de dados com IA
- Automação de workflows
- Transformação de JSON
- Construção de pipelines orientados ao negócio

## Licença

Este projeto está disponível para fins educacionais e de portfólio.
