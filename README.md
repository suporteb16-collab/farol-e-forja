# 🔦 Farol e Forja — Dashboard B16

Dashboard de marketing digital para o lançamento **Farol e Forja** do cliente **Juliano Cazarré**, desenvolvido pela [Agência B16](https://agenciab16.com.br).

Página única (`index.html`) publicada via **GitHub Pages**, lendo dados em tempo real do Google Sheets através de um Cloudflare Worker.

---

## 📊 Visão Geral

O dashboard lê **duas** abas em tempo real:

| Aba | Fonte | Conteúdo |
|---|---|---|
| `meta_ads` | Exportação Meta Ads | Campanhas, criativos, adsets, investimento, funil (por Anúncio × Adset × Dia) |
| `Vendas_Totais` | Consolidação (tamborete + kiwify) | Todas as vendas confirmadas, faturamento, UTMs, gateway |

> A antiga aba `Vendas Online` (Kiwify) foi **removida** — as vendas online agora vivem dentro de `Vendas_Totais` (coluna `gateway = kiwify`).

---

## 🏗️ Infraestrutura

```
Tamborete / Kiwify → Consolidação → aba Vendas_Totais
                                          ↓
Meta Ads (export manual) ─────────→ aba meta_ads
                                          ↓
                        Cloudflare Worker (GET ?sheet=<aba>)
                                          ↓
                          Dashboard HTML (GitHub Pages)
```

### Cloudflare Worker

- **URL:** `https://farol-e-forja-webhook.henrscard.workers.dev`
- **GET `?sheet=<aba>`:** lê qualquer aba do Sheets e devolve CSV (é como o dashboard consome os dados).
- **POST (webhook Tamborete):** grava vendas aprovadas na aba `tamborete_silver`.

| Secret | Descrição |
|---|---|
| `SHEET_ID` | ID da planilha Google Sheets |
| `GOOGLE_CLIENT_EMAIL` | E-mail da Service Account GCP |
| `GOOGLE_PRIVATE_KEY` | Chave privada da Service Account |
| `WEBHOOK_SECRET` | Token de validação do webhook |

### GCP — Service Account
- **Projeto:** `farol-e-forja-b16` · **API:** Google Sheets API · **Permissão na planilha:** Editor

---

## 📋 Planilha Google Sheets

**ID:** `1-TyOdUeK0akbwY7Rj9lOjTaBnmbLj5ZtElO7-YZlAss`

### Aba `meta_ads`
Exportação manual do Meta Ads (colunas em inglês, por Anúncio × Adset × Dia):

```
Date | Campaign Name | Ad Name | Adset Name |
Spend (Cost, Amount Spent) | Reach (Estimated) | Impressions |
Inline Link Clicks | Action Landing Page View |
Action FB Pixel Initiate Checkout (Offsite Conversion) |
Action FB Pixel Purchase (Offsite Conversion) |
Website Purchase Roas | Action Leads | Instagram Profile Visits
```

> ⚠️ Os números vêm formatados em pt-BR (ponto de milhar, ex.: `2.765`). O parser do dashboard trata isso; não reformatar as colunas.

### Aba `Vendas_Totais`
Fonte unificada de vendas (tamborete + kiwify):

```
id_transacao | status_transacao | data_transacao |
id_produto | nome_produto | qtd_produto |
source | medium | campaign | content | term |
valor_transacao | gateway
```

---

## 🧮 Regras de cálculo importantes

- **Impressões / cliques / LPV:** somados de `meta_ads` (parser trata ponto de milhar). Total do período de anúncios = **879.620**.
- **Alcance:** valor **dedup a nível de conta** (não é somável). Definido manualmente na constante `REACH_DEDUP` do `index.html`.
- **CAC (Meta):** considera **somente as vendas online (kiwify)** contra o **investimento do site** (`[B16] [VENDAS] [SITE] [F&F]`), alinhado por data. Vendas presenciais (Âncora etc.) **não** entram no CAC, pois não têm mídia.
- **Melhor CAC por criativo:** cruza `Ad Name` (Meta) × `content` (utm_content das vendas).
- **Impostos Meta:** o "Valor usado" é 87,5% do total; 12,5% é imposto (calculado no card *Composição do Investimento*).
- **Seguidores no Instagram (1.068):** estáticos no código (`SEGUIDORES_AD` / `SEGUIDORES_CAMP`), vindos dos exports do Meta (Campanhas/Anúncios). Não estão na planilha.

---

## 🚀 Deploy

O GitHub Pages publica **direto do branch `main`** — cada `git push` vai ao ar.

1. **Pages:** Settings → Pages → Deploy from a branch → `main` / `root`.
2. **URL:** `https://suporteb16-collab.github.io/farol-e-forja/`
3. **Atualizar dados:** colar o novo export na aba correspondente do Sheets; o dashboard atualiza ao recarregar.

> `.gitignore` exclui `*.csv` e `.mcp.json` — dados de campanha não vão para o repositório público.

---

## 📅 Filtros
- **Período padrão:** `27/04/2026` (1ª venda) até hoje. *Obs.: os anúncios só têm dado a partir de 24/06.*
- **Auto-refresh:** a cada 1 hora.

---

## 🎨 Abas e Seções

### Aba **Visão Geral**
| Seção | Descrição |
|---|---|
| Big Numbers | Investido, Total de Vendas, Faturamento, **CAC Online**, Melhor CAC (criativo) |
| Vendas: Presencial × Online | Split por gateway (vendas, faturamento e ticket médio) |
| Composição do Investimento | Rosca Mídia (87,5%) × Impostos (12,5%) + investimento por objetivo |
| Seguidores no Instagram | Total (1.068) + tabela com scroll e toggle Anúncio/Campanha |
| Funil de Conversão | Impressões → Alcance → Cliques → LPV → Checkout → Vendas |
| Evolução Diária | Investimento × Faturamento, Vendas/dia, CAC Online/dia, Com/Sem Tracking |
| Distribuição por Origem | Vendas por canal + investimento por público (Frio/Quente) |
| Criativos / Adsets / Campanhas | Tabelas de performance (100% Meta Ads) |

### Aba **Levantamento F&F**
Resumo de produção de conteúdo e resultados orgânicos (259 peças, ~9 mil → 281 mil seguidores, views, insights) — fonte: documento do cliente.

---

## 👥 Responsável

**Agência B16** — [henrscard@gmail.com](mailto:henrscard@gmail.com)
