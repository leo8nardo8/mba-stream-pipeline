# Stream Processing Pipeline — Criptomoedas em Tempo Real

**MBA em Engenharia de Dados | Disciplina: Stream Processing Pipelines**
Prof. Rafael Tsuji Matsuyama | Abril 2026

---

## Visão Geral

Pipeline de streaming que consome cotações de criptomoedas em tempo real da **CoinGecko API**, processa os eventos com **Apache Spark Structured Streaming** no Databricks, e persiste os resultados em **Delta Lake (Parquet)**.

```
CoinGecko API (JSON, sem autenticação)
    │  polling a cada 15s
    ▼
Landing Zone — DBFS /landing (JSON bruto)
    │  readStream (Spark Structured Streaming)
    ▼
[Bônus 1] Schema estruturado + filtro de registros inválidos
    │
    ▼
Transformações: cast de tipos, spread_24h, trend_24h
    │
    ├── [Base]     writeStream → Delta Lake /output
    ├── [Bônus 2]  groupBy + agg → Delta Lake /aggregations
    └── [Bônus 3]  window() sliding 60min + tumbling 30min → /windows
```

## Estrutura do Repositório

```
├── notebooks/
│   └── stream_pipeline_coingecko.ipynb   ← notebook principal (10 células comentadas)
├── databricks.yml                         ← Bônus 4: deploy como Asset Bundle
├── .github/
│   └── workflows/
│       └── deploy_pipeline.yml            ← Bônus 4: CI/CD via GitHub Actions
└── README.md
```

## Entregáveis por Critério

| Critério | Arquivo | Status |
|----------|---------|--------|
| Enunciado base (ingestão + ETL + Parquet) | `stream_pipeline_coingecko.ipynb` células 4–6 | ✅ |
| Bônus 1 — Schema e validação | Célula 3 + filtros na célula 6 | ✅ |
| Bônus 2 — Agregação | Célula 8 | ✅ |
| Bônus 3 — Window Functions | Célula 9 (sliding + tumbling) | ✅ |
| Bônus 4 — Deploy CI/CD | `databricks.yml` + `deploy_pipeline.yml` | ✅ |

## Como Executar (Databricks Community Edition)

1. Criar cluster com **Databricks Runtime 13.3 LTS** ou superior
2. Importar `notebooks/stream_pipeline_coingecko.ipynb` via Workspace > Import
3. Executar célula a célula — cada célula tem comentários explicativos
4. A célula 5 gera os dados (aguarda ~75s para 5 batches)
5. Verificar outputs em `dbfs:/tmp/crypto_pipeline/`

Nenhuma API key necessária. A CoinGecko API é pública no tier gratuito.

## Nota sobre o Bônus 4 — Deploy CI/CD

O workflow `.github/workflows/deploy_pipeline.yml` é composto por dois jobs:

- **validate** — roda em todo push e Pull Request, sem nenhuma credencial. Valida a sintaxe do notebook, testa a conectividade com a CoinGecko API e verifica a estrutura do repositório. ✅ Funciona no Community Edition.

- **deploy** — faz o deploy do pipeline no Databricks via Asset Bundle. Requer `DATABRICKS_TOKEN` e `DATABRICKS_HOST` configurados nos secrets do repositório. No Databricks Community Edition, tokens de acesso pessoal não estão disponíveis — o job é marcado com `continue-on-error: true` e emite um aviso explicativo sem quebrar o pipeline. Em um ambiente Enterprise ou Pro, basta adicionar os secrets e o deploy passa a funcionar automaticamente a cada merge na branch `main`.
