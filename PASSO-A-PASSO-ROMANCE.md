# 📘 PASSO A PASSO — App de Romance Stories (RomanceFlix+)

> Modelo "Netflix" para histórias de romance narradas — assinatura única via Stripe no site, app Android + iOS só consome.
> Inspirado nos canais: @romancestoriesdavid, @marcusstories123, @liebesgeschichten1, @romancestories231

---

## 🎯 ESTRATÉGIA — Por que vai funcionar

O nicho de **romance stories narradas** explodiu no YouTube nos últimos anos. Os canais que você mandou são exemplos vivos:
- Audiência fiel (mulheres 25-55, alta retenção)
- Histórias longas (30-60 min) que viciam
- Formato "audiobook visual" — pessoas escutam fazendo outras coisas
- Subgêneros viciantes: bilionário, segunda chance, amor proibido, secret baby

**Sua vantagem com app próprio:**
- YouTube limita alcance e tira 45% do que paga
- Conteúdo "mais quente" / sensual é frequentemente desmonetizado
- Você não tem lista de assinantes
- Sem app próprio, está sempre nas mãos do algoritmo

---

## 🏗️ ARQUITETURA EM 1 IMAGEM

```
                    ┌─────────────────────────┐
                    │   YOUTUBE (gratuito)    │  ← Marketing / isca
                    │  Trailers e episódios   │
                    │  curtos. Comentário     │
                    │  fixado: "história      │
                    │   completa no app"      │
                    └────────────┬────────────┘
                                 ▼
   ┌──────────────────────────────────────────────────────┐
   │           SITE (romanceflix.com.br)                  │
   │  - Cadastro / login                                  │
   │  - 1 ÚNICO PLANO: R$ 19,90/mês via Stripe            │
   │  - Player web + modo só áudio                        │
   └──────────┬─────────────────────────────────┬─────────┘
              │                                 │
              ▼                                 ▼
   ┌──────────────────┐               ┌──────────────────┐
   │   APP ANDROID    │               │     APP iOS      │
   │  (Play Store)    │               │  (App Store)     │
   │  Login → Player  │               │  Login → Player  │
   │  NÃO cobra aqui  │               │  NÃO cobra aqui  │
   └──────────┬───────┘               └────────┬─────────┘
              │                                 │
              └──────────────┬──────────────────┘
                             ▼
          ┌──────────────────────────────────────┐
          │  BACKEND (API) + STRIPE              │
          │  - Valida assinatura ativa           │
          │  - Gera URL assinada do vídeo        │
          └────────────┬─────────────────────────┘
                       ▼
            ┌──────────────────────────┐
            │   BUNNY STREAM / MUX     │  ← Vídeos hospedados
            │   HLS protegido + DRM    │
            └──────────────────────────┘
```

**Modelo de negócio:** **PLANO ÚNICO** — R$ 19,90/mês com 7 dias grátis. Simples, sem confundir o cliente.

---

## 📋 FASE 0 — DECISÕES INICIAIS (1 semana)

### 0.1 — Marca
- [ ] **Nome do app** (sugestões: "RomanceFlix+", "StoryLove", "Histórias de Amor+", "Paixões")
- [ ] **Tagline** (ex: "As histórias de amor mais envolventes, agora só pra você")
- [ ] **Logo** elegante, feminina, em rosa/dourado (Canva pro ou designer freelancer)
- [ ] **Domínio** — registrar `.com.br` ou `.com` no Cloudflare
- [ ] **Cores e identidade visual** (sugiro rose gold + bordô + off-white)

### 0.2 — Modelo de negócio (PLANO ÚNICO)
- [ ] **Plano único:** R$ 19,90/mês
- [ ] **Trial grátis:** 7 dias (regra de ouro pra romance — converte ~40%)
- [ ] **Cancelamento:** a qualquer momento, sem multa
- [ ] **Forma de pagamento:** cartão, Pix recorrente e boleto (todos via Stripe)

### 0.3 — Conteúdo de lançamento
- [ ] **Mínimo de 15-20 histórias prontas** pra lançar
- [ ] **Subgêneros para variar** o catálogo:
  - 💼 Bilionário / CEO
  - 💔 Segunda chance / Reencontro
  - 🚫 Amor proibido
  - 👶 Secret baby
  - 💍 Casamento de conveniência / arranjado
  - 🏰 Príncipe e plebeia
  - 🌹 Amigos viram amantes
- [ ] **Frequência:** mínimo 3 episódios novos por semana (engajamento alto)
- [ ] **Duração ideal:** 30-60 min por episódio (formato similar aos canais de referência)

### 0.4 — Empresa
- [ ] **CNPJ** (MEI até R$ 81 mil/ano, ou ME após isso)
- [ ] **Conta bancária PJ**
- [ ] **Termos de Uso e Política de Privacidade** (LGPD obrigatório)
- [ ] **Classificação etária** — definir se vai ter conteúdo +18 (afeta as lojas)

⚠️ **Atenção sobre conteúdo +18:** Apple e Google são restritivos. Se o conteúdo for sensual/adulto, pode rejeitar o app. Mantenha "spicy mas sem explícito" — igual livros romance bestsellers fazem.

---

## 🛠️ FASE 1 — INFRAESTRUTURA E HOSPEDAGEM (1 semana)

### 1.1 — Bunny Stream (recomendado)
1. Criar conta em https://bunny.net
2. Ativar **Bunny Stream** (US$ 5/mês mínimo)
3. Criar uma **Video Library** chamada "Romances"
4. **Por que Bunny:**
   - Vídeo protegido (token signing) — não dá pra baixar
   - HLS adaptativo
   - Modo "audio only" nativo (perfeito pra romance que pessoas escutam)
   - Custo MUITO menor que Vimeo

### 1.2 — Servidor (backend + site)
- **Hospedagem:** Hetzner (€ 4-10/mês), DigitalOcean (US$ 6/mês) ou Railway (US$ 5/mês)
- **Domínio + SSL:** Cloudflare (grátis)

---

## 💳 FASE 2 — STRIPE (3-5 dias)

### 2.1 — Criar conta Stripe Brasil
1. Cadastrar em https://stripe.com/br com CNPJ
2. Ativar **Stripe Billing** (assinaturas recorrentes)
3. Ativar **Pix** (suporte recorrente)
4. Pegar as chaves de API

### 2.2 — Criar 1 ÚNICO Produto no Stripe
No painel da Stripe → Products → Add Product:
- Produto: "Acesso Total" → Preço R$ 19,90/mês recorrente
- Adicionar **trial period** de 7 dias

Anotar o `price_id` (price_xxxxx) — vamos usar no código.

### 2.3 — Configurar Webhook
- URL: `https://romanceflix.com.br/api/stripe/webhook`
- Eventos a escutar:
  - `customer.subscription.created`
  - `customer.subscription.updated`
  - `customer.subscription.deleted`
  - `invoice.payment_succeeded`
  - `invoice.payment_failed`

---

## 💻 FASE 3 — SITE / WEB APP (4-6 semanas)

### 3.1 — Stack recomendada para romance
**Opção A — Mais Rápido (recomendado):**
- WordPress + plugin **Paid Memberships Pro** (open source, GPL)
- Stripe nativo, restrição de conteúdo, plano único pronto
- Player Bunny Stream embutido via shortcode
- Pronto em 2-3 semanas

**Opção B — Mais Customizável:**
- Next.js (frontend + backend) + PostgreSQL + Stripe SDK
- Forkar `justdjango/video-membership` ou `sadmann7/netflx-web`
- 6-10 semanas

### 3.2 — Funcionalidades obrigatórias do site
1. **Landing page** atraente com trailers do YouTube embedados
2. **Página única de plano** (R$ 19,90 com botão grande)
3. **Cadastro / Login** (email + senha + Google login)
4. **Checkout Stripe** (botão → Stripe Checkout → volta com sessão ativa)
5. **Área da assinante:**
   - Catálogo organizado por subgênero
   - Player web (com modo só áudio — diferencial!)
   - "Continuar de onde parou"
   - Lista de favoritos
   - Página de conta (cancelar, trocar cartão)
6. **Webhook handler** (recebe eventos da Stripe)
7. **Termos de Uso / Privacidade / LGPD**

### 3.3 — Diferencial: MODO ÁUDIO
A maior parte das suas assinantes vai ouvir as histórias enquanto:
- Cozinha
- Dirige
- Trabalha
- Antes de dormir

**Implementar:**
- Botão "modo só áudio" no player
- Capa fica grande na tela
- Continua tocando com tela bloqueada (PWA + Media Session API)
- Equivalente do Spotify pra histórias

### 3.4 — Fluxo de proteção de vídeo
1. Frontend pede ao backend: "quero ver vídeo X"
2. Backend confere: "esse user tem assinatura ativa?"
3. Se sim, gera **URL assinada** do Bunny Stream (válida por 1 hora)
4. Frontend toca a URL no player

Isso impede pirataria.

---

## 📱 FASE 4 — APP ANDROID (3-5 semanas)

### 4.1 — Estratégia: TWA primeiro, nativo depois
**TWA (Trusted Web Activity)** = empacota o site como app Android.

**Vantagens:**
- 1-2 dias de trabalho
- 100% código compartilhado
- Atualizações instantâneas

**Como fazer:**
1. Garantir que o site é **PWA** (manifest.json + service worker)
2. Usar **Bubblewrap**: `npx @bubblewrap/cli init --manifest=https://romanceflix.com.br/manifest.json`
3. Gera APK/AAB pronto pra Play Store
4. Publicar (US$ 25 taxa única)

### 4.2 — Evolução para nativo
Quando tiver 200-500 assinantes:
- **React Native** (recomendado — mesmo código pra iOS)
- Bibliotecas: `react-native-video` ou Bunny Stream SDK
- **Recurso essencial:** notificação push quando sair episódio novo (aumenta retenção 30%+)

### 4.3 — REGRA DO GOOGLE PLAY (MUITO IMPORTANTE)
Como você cobra no SITE:
- ✅ App é "**streaming service**" (igual Netflix, Spotify)
- ❌ **NÃO** pode ter botão "assinar" dentro do app
- ❌ **NÃO** pode mencionar preços ou link pra pagamento
- ✅ Pode dizer: "Para assinar, visite romanceflix.com.br"

Se seguir, funciona — **é exatamente o modelo Netflix**.

---

## 🍎 FASE 5 — APP iOS (3-4 semanas)

### 5.1 — Conta Apple Developer
- US$ 99/ano (https://developer.apple.com)
- Precisa CNPJ e D-U-N-S Number (gratuito)

### 5.2 — Reader App Entitlement
Apple criou categoria especial pra apps tipo Netflix/Kindle/Audible:
- Categoria: **"Reader Apps"**
- Permite NÃO oferecer compra no app
- Aplica-se: livros, vídeos, **áudio (audiobooks)**, magazines
- Solicitar: https://developer.apple.com/contact/request/reader-app-link

**Romance histórias narradas se enquadram perfeitamente** — é tipo audiobook.

### 5.3 — Build com React Native
Se já tem o app Android em React Native, o iOS é "compilar e ajustar":
- 80% código compartilhado
- Testar no TestFlight antes de publicar

---

## 📊 FASE 6 — LANÇAMENTO E MARKETING (contínuo)

### 6.1 — Estratégia de aquisição via YouTube
Você JÁ vai criar conteúdo no YouTube — esse é seu motor de aquisição.

**Conteúdo público (YouTube):**
- Trailers de 3-5 min ou "primeiros 10 minutos" de cada história
- Comentário fixado: "História completa no app: romanceflix.com.br/historia"
- Card final no vídeo direcionando ao site
- Descrição do vídeo com link

**Funil de conversão:**
```
YouTube (grátis) → Site (cadastro) → 7 dias trial → Assinante R$ 19,90/mês
```

**Conversão típica em romance:**
- 2-5% das visualizações do YouTube clicam no link
- 30-50% dos cadastros começam o trial
- 30-50% dos trials viram assinantes pagos
- Resultado: a cada 10.000 views no YouTube, ~30-100 assinantes pagos

### 6.2 — Métricas para acompanhar
- Conversão YouTube → cadastro (meta: 3%+)
- Conversão trial → pago (meta: 40%+)
- Churn mensal (meta: < 7%)
- LTV por assinante (meta: R$ 100+ — ou seja, 5+ meses na média)

### 6.3 — Retenção (chave do modelo de assinatura)
- **3 episódios novos toda semana** sem falta
- **Pesquisa com assinantes:** "que história você quer ouvir?"
- **Notificações push** quando sair episódio novo
- **Cliffhangers** entre episódios da mesma série
- **Comunidade Telegram/Discord** exclusiva pra assinantes
- **Conteúdo "votado pelas fãs"** — engajamento brutal

---

## 💰 ESTIMATIVA DE CUSTOS MENSAIS

### Lançamento (0-100 assinantes)
| Item | Custo |
|---|---|
| Servidor (Hetzner/DigitalOcean) | R$ 30 |
| Bunny Stream (vídeo + CDN) | R$ 50 |
| Domínio + Cloudflare | R$ 10 |
| Stripe (taxa por transação) | 4% + R$ 0,40 |
| Apple Developer | R$ 50 (US$ 99/ano ÷ 12) |
| Google Play | R$ 0 (taxa única US$ 25) |
| **Total fixo** | **~R$ 140/mês** |

### Crescimento (1.000 assinantes pagantes)
| Item | Custo |
|---|---|
| Servidor maior | R$ 100 |
| Bunny Stream (mais tráfego) | R$ 400 |
| Email transacional | R$ 50 |
| Stripe (~R$ 800 em taxas) | R$ 800 |
| **Total** | **~R$ 1.350/mês** |

**Receita com 1.000 assinantes a R$ 19,90:** R$ 19.900/mês
**Margem líquida:** ~R$ 18.000/mês

### Projeção realista de crescimento (com canal YouTube já ativo)
| Mês | Assinantes | Receita | Margem |
|---|---|---|---|
| 1 | 50 | R$ 995 | R$ 800 |
| 3 | 250 | R$ 4.975 | R$ 4.300 |
| 6 | 700 | R$ 13.930 | R$ 12.500 |
| 12 | 1.500 | R$ 29.850 | R$ 27.000 |

---

## ⚠️ RISCOS E COMO MITIGAR

| Risco | Mitigação |
|---|---|
| Apple/Google rejeita o app | Conteúdo "spicy mas não explícito" + Reader App Guidelines |
| Pirataria (gravarem a tela) | URLs assinadas + watermark dinâmico do email do user |
| Stripe bloqueia (conteúdo adulto) | Manter dentro do "romance" sem sexo explícito; backup com Mercado Pago |
| Churn alto (cancelamento) | 3 episódios novos/semana + cliffhangers + comunidade |
| Concorrência (canais YouTube grátis) | Histórias EXCLUSIVAS + qualidade superior + sem propaganda |

---

## ✅ CHECKLIST GERAL

### Você (criador):
- [ ] Definir nome, logo, identidade visual
- [ ] Abrir CNPJ
- [ ] Conta bancária PJ
- [ ] Roteirizar e gravar 15+ histórias (subgêneros variados)
- [ ] Comprar domínio
- [ ] Criar contas: Stripe, Bunny Stream, Google Play, Apple Developer
- [ ] Termos de Uso e Política de Privacidade

### Seu parceiro (ou contratado dev):
- [ ] Implementar site (Fase 3) — 4-6 semanas
- [ ] Implementar app Android (Fase 4) — 2-5 semanas
- [ ] Implementar app iOS (Fase 5) — 3-4 semanas
- [ ] Configurar webhooks Stripe e proteção de vídeo
- [ ] Publicar nas lojas

### Total de tempo até iOS publicado: **3 a 4 meses**

---

## 🚀 PRÓXIMO PASSO IMEDIATO

1. **Aprovar a apresentação visual** (`romance.html`)
2. **Definir nome e marca** do app
3. **Decidir entre Opção A (WordPress + PMP) ou Opção B (custom)**
4. **Começar pela Fase 1 (infra)** — leva 1 semana

Quando você decidir, posso te ajudar a:
- Configurar o WordPress + Paid Memberships Pro do zero
- Ou clonar e adaptar o `justdjango/video-membership`
- Ou criar do zero em Next.js
- Configurar a Stripe e o produto do plano único
- Configurar Bunny Stream e fazer o primeiro upload protegido
