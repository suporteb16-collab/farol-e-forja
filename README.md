# 🔦 Farol e Forja — Dashboard B16

Dashboard de marketing digital para o lançamento **Farol e Forja** do cliente **Juliano Cazarré**, desenvolvido pela [Agência B16](https://agenciab16.com.br).

---

## 📊 Visão Geral

O dashboard lê dados em tempo real de duas fontes:

| Aba | Fonte | Conteúdo |
|---|---|---|
| `meta_ads` | Exportação Meta Ads | Campanhas, criativos, investimento, funil |
| `tamborete` | Webhook (Tamborete) | Vendas confirmadas, faturamento, UTMs |

---

## 🏗️ Infraestrutura

```
Tamborete Webhook
      ↓
Cloudflare Worker (farol-e-forja-webhook)
      ↓
Google Sheets (privada) ← Service Account GCP
      ↓
Dashboard HTML (GitHub Pages)
```

### Cloudflare Worker

- **URL do webhook:** `https://farol-e-forja-webhook.<subdominio>.workers.dev?secret=fef_whk_2026`
- **Evento disparado:** pagamento aprovado (`status: APPROVED`)
- **Secrets configurados no Worker:**

| Secret | Descrição |
|---|---|
| `SHEET_ID` | ID da planilha Google Sheets |
| `GOOGLE_CLIENT_EMAIL` | E-mail da Service Account GCP |
| `GOOGLE_PRIVATE_KEY` | Chave privada da Service Account |
| `WEBHOOK_SECRET` | Token de validação do webhook |

### GCP — Service Account

- **Projeto:** `farol-e-forja-b16`
- **API ativada:** Google Sheets API
- **Permissão na planilha:** Editor

---

## 📋 Planilha Google Sheets

**ID:** `1-TyOdUeK0akbwY7Rj9lOjTaBnmbLj5ZtElO7-YZlAss`

### Aba `meta_ads`

Exportação manual dos dados do Meta Ads. Colunas esperadas:

```
Date | Campaign Name | Ad Name | Adset Name |
Spend (Cost, Amount Spent) | Reach (Estimated) | Impressions |
Inline Link Clicks | Action Landing Page View |
Action FB Pixel Initiate Checkout (Offsite Conversion) |
Action FB Pixel Purchase (Offsite Conversion) |
Website Purchase Roas | Action Leads
```

### Aba `tamborete`

Preenchida automaticamente pelo Cloudflare Worker a cada venda aprovada:

```
order_ref | order_number | order_status | amount | currency |
payment_method | installments | card_brand | card_last4 |
product_name | product_price | quantity |
authorization_code | authorization_nsu |
utm_source | utm_medium | utm_campaign | utm_content | utm_term |
customer_name | customer_email | customer_phone |
created_at | raw_payload
```

---

## 🚀 Deploy

### 1. Configurar a URL do Worker no dashboard

No arquivo `index.html`, localize e substitua:

```javascript
const WORKER_URL = 'https://farol-e-forja-webhook.SEU-SUBDOMINIO.workers.dev';
```

### 2. Habilitar GitHub Pages

1. Acesse **Settings → Pages** no repositório
2. Source: `Deploy from a branch`
3. Branch: `main` / `root`
4. O dashboard ficará disponível em: `https://suporteb16-collab.github.io/farol-e-forja/`

### 3. Atualizar dados Meta Ads

1. Exporte os dados do Meta Ads em CSV
2. Cole na aba `meta_ads` da planilha (mantendo o cabeçalho)
3. O dashboard atualiza automaticamente ao recarregar

---

## 📅 Filtros

- **Período padrão:** `20/06/2026` até hoje
- **Auto-refresh:** a cada 2 horas
- Filtros de data disponíveis no header do dashboard

---

## 🎨 Seções do Dashboard

| Seção | Descrição |
|---|---|
| Big Numbers | Investimento, Vendas, Faturamento, CAC, Melhor CAC |
| Funil de Conversão | Impressões → Alcance → Cliques → LPV → Checkout → Vendas |
| Evolução Diária | 4 gráficos: Inv×Fat, Vendas/dia, CAC/dia, Pago vs Orgânico |
| Distribuição por Origem | Vendas por UTM + Investimento por público (Frio/Quente) |
| Criativos e Distribuição | Investimento e Vendas por criativo |
| Performance por Criativo | Tabela com Investimento, Vendas e CAC |
| CAC por Público | Frio ❄️ vs Quente 🔥 |

---

## 👥 Responsável

**Agência B16** — [henrscard@gmail.com](mailto:henrscard@gmail.com)
