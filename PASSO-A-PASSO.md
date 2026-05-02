# 📘 PASSO A PASSO COMPLETO — App de Streaming Bíblico

> Modelo "Netflix" — assinatura via Stripe no site, app só consome conteúdo.
> Vídeos hospedados em servidor próprio (Bunny Stream / Mux), YouTube usado só para divulgação.

---

## 🎯 ARQUITETURA EM 1 IMAGEM (texto)

```
                    ┌─────────────────────────┐
                    │   YOUTUBE (gratuito)    │  ← Marketing / isca
                    │  Trailers, episódios    │
                    │  Diz: "veja completo no │
                    │   nosso app"            │
                    └────────────┬────────────┘
                                 ▼
   ┌──────────────────────────────────────────────────────┐
   │             SITE (historias-eternas.com)             │
   │  - Cadastro / login                                  │
   │  - Stripe Checkout (assina aqui)                     │
   │  - Player web                                        │
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

**Regra de ouro:** o assinante PAGA NO SITE. O app só faz login. Isso evita a comissão de 30% da Apple e Google e se enquadra na regra de "Reader App" (igual Netflix).

---

## 📋 FASE 0 — DECISÕES INICIAIS (1 semana)

Antes de escrever uma linha de código, definir:

### 0.1 — Marca
- [ ] **Nome do app** (ex: "Histórias Eternas+", "Bíblia em 1ª Pessoa", "Testemunhos+")
- [ ] **Logo** (Canva, Figma ou contratar designer)
- [ ] **Domínio** (.com.br ou .com — registrar no Registro.br ou Cloudflare)
- [ ] **Cores e identidade visual**

### 0.2 — Modelo de negócio
- [ ] **Preço dos planos:**
  - Mensal: R$ 19,90
  - Anual: R$ 149 (R$ 12,40/mês — desconto)
  - Família: R$ 24,90 (4 telas)
- [ ] **Trial grátis:** 7 dias
- [ ] **Política de cancelamento:** a qualquer momento

### 0.3 — Conteúdo
- [ ] **Quantos vídeos no lançamento?** (mínimo 10-15 episódios)
- [ ] **Frequência de novos episódios** (1 por semana?)
- [ ] **Roteiros prontos** dos primeiros 10 episódios

### 0.4 — Empresa
- [ ] **CNPJ** (MEI ou ME) — necessário para abrir conta Stripe
- [ ] **Conta bancária PJ**
- [ ] **Termos de Uso e Política de Privacidade** (LGPD obrigatório)

---

## 🛠️ FASE 1 — INFRAESTRUTURA E HOSPEDAGEM DE VÍDEO (1 semana)

### 1.1 — Conta Bunny Stream (recomendado)
1. Criar conta em https://bunny.net
2. Ativar **Bunny Stream** (US$ 5/mês mínimo + US$ 0,005 por GB entregue)
3. Criar uma **Video Library**
4. **Por que Bunny e não YouTube/Vimeo:**
   - Vídeo protegido (token signing) — não dá pra baixar
   - HLS adaptive streaming (4K → 360p automático)
   - CDN global rápida
   - Custo MUITO menor que Vimeo/Mux
   - SDK Android e iOS prontos

### 1.2 — Alternativas a Bunny
| Opção | Preço | Quando usar |
|---|---|---|
| **Bunny Stream** | ~US$ 5-50/mês | ✅ Recomendado pra começar |
| **Mux** | US$ 0,005/min armazenado + US$ 0,012/min entregue | Quando crescer (analytics avançado) |
| **Cloudflare Stream** | US$ 5/mês cada 1.000 min armazenados | Boa alternativa simples |
| **AWS S3 + CloudFront + MediaConvert** | Variável | Maior controle, mais complexo |

### 1.3 — Servidor (backend + site)
- **Hospedagem:** Hetzner (€ 4-10/mês), DigitalOcean (US$ 6/mês) ou Railway (US$ 5/mês)
- **Domínio + SSL:** Cloudflare (grátis)

---

## 💳 FASE 2 — STRIPE (3-5 dias)

### 2.1 — Criar conta Stripe Brasil
1. Acessar https://stripe.com/br
2. Cadastrar com CNPJ e dados bancários PJ
3. Ativar **Stripe Billing** (assinaturas recorrentes)
4. Ativar **Pix** (Stripe já suporta no Brasil)
5. Pegar as **chaves de API** (publishable key + secret key)

### 2.2 — Criar Produtos no Stripe Dashboard
No painel da Stripe → Products → Add Product:
- Produto 1: "Plano Mensal" → Preço R$ 19,90/mês recorrente
- Produto 2: "Plano Anual" → Preço R$ 149/ano recorrente
- Produto 3: "Plano Família" → Preço R$ 24,90/mês recorrente

Anotar os `price_ids` (price_xxxxx) — vamos usar no código.

### 2.3 — Configurar Webhook
- URL: `https://seusite.com.br/api/stripe/webhook`
- Eventos a escutar:
  - `customer.subscription.created`
  - `customer.subscription.updated`
  - `customer.subscription.deleted`
  - `invoice.payment_succeeded`
  - `invoice.payment_failed`

---

## 💻 FASE 3 — SITE / WEB APP (4-6 semanas)

### 3.1 — Stack recomendada
**Opção A — Mais Rápido (recomendado para começar):**
- WordPress + plugin **Paid Memberships Pro** (open source, GPL)
- Stripe nativo, restrição de conteúdo, planos prontos
- Player Bunny Stream embutido via shortcode
- Pronto em 2-3 semanas

**Opção B — Mais Customizável:**
- Next.js (frontend + backend) + PostgreSQL + Stripe SDK
- Forkar **justdjango/video-membership** ou **sadmann7/netflx-web** como base
- 6-10 semanas

### 3.2 — Funcionalidades obrigatórias do site
1. **Landing page** (home pública com trailers do YouTube)
2. **Página de planos** (3 cards de preço)
3. **Cadastro / Login** (email + senha, ou Google login)
4. **Checkout Stripe** (botão → Stripe Checkout → volta com sessão)
5. **Área do assinante:**
   - Catálogo de vídeos
   - Player web (Bunny Stream embed)
   - "Continuar assistindo"
   - Página de conta (cancelar assinatura, trocar cartão)
6. **Webhook handler** (recebe eventos da Stripe e atualiza banco)
7. **Termos de Uso / Privacidade / LGPD**

### 3.3 — Fluxo de proteção de vídeo
Quando o usuário clica em "assistir":
1. Frontend pede ao backend: "quero ver vídeo X"
2. Backend confere: "esse user tem assinatura ativa?"
3. Se sim, gera **URL assinada** do Bunny Stream (válida por 1 hora)
4. Frontend toca a URL no player

Isso impede pirataria — o link expira e não é compartilhável.

---

## 📱 FASE 4 — APP ANDROID (3-5 semanas)

### 4.1 — Estratégia: TWA primeiro, nativo depois
**TWA (Trusted Web Activity)** = empacotar o site como app Android. É o que o **Twitter/X**, **Starbucks** e muitos outros usam.

**Vantagens:**
- 1-2 dias de trabalho
- 100% código compartilhado com o site
- Atualizações instantâneas (sem republicar na Play Store)

**Como fazer TWA:**
1. Garantir que o site é **PWA** (Progressive Web App) — manifest.json + service worker
2. Usar **Bubblewrap** (ferramenta do Google): `npx @bubblewrap/cli init --manifest=https://seusite.com.br/manifest.json`
3. Gera APK/AAB pronto pra Play Store
4. Publicar na Play Console (US$ 25 taxa única)

### 4.2 — App nativo (depois que validar mercado)
Quando tiver os primeiros 200-500 assinantes, evoluir para:
- **React Native** (recomendado — mesmo código pra iOS)
- ou **Flutter**
- ou **Kotlin nativo**

Bibliotecas:
- `react-native-video` ou Bunny Stream Mobile SDK
- `@stripe/stripe-react-native` (se for cobrar dentro do app via Play Billing — opcional)

### 4.3 — REGRA CRÍTICA do Google Play
Como você cobra a assinatura no SITE (e não no app), você precisa se enquadrar nas exceções:
- ✅ App é "**streaming service**" (igual Netflix, Spotify)
- ✅ Assinante já existe — app só faz login
- ❌ **NÃO** pode ter botão "assinar" dentro do app
- ❌ **NÃO** pode mencionar preços ou link pra pagamento dentro do app
- ✅ Pode dizer: "Para assinar, visite nosso site"

Isso é EXATAMENTE como Netflix faz. **Funciona** desde que siga a regra.

---

## 🍎 FASE 5 — APP iOS (3-4 semanas)

### 5.1 — Conta Apple Developer
- US$ 99/ano (https://developer.apple.com)
- Precisa CNPJ e D-U-N-S Number (gratuito em https://www.dnb.com)

### 5.2 — Reader App Entitlement
Apple criou uma categoria especial para apps tipo Netflix/Kindle/Spotify:
- Categoria: **"Reader Apps"**
- Permite NÃO oferecer compra no app, desde que o conteúdo já tenha sido pago externamente
- Aplica-se: livros, vídeos, áudio, magazines, news, cloud storage
- Solicitar via: https://developer.apple.com/contact/request/reader-app-link

### 5.3 — Build com React Native
Se já tem o app Android em React Native, o iOS é "compilar e ajustar":
- 80% do código é compartilhado
- Ajustes: fontes, ícones, splash screen, navegação iOS
- Testar no TestFlight antes de publicar

---

## 📊 FASE 6 — LANÇAMENTO E MARKETING (contínuo)

### 6.1 — Estratégia de aquisição via YouTube
Você JÁ tem audiência no YouTube — esse é seu trunfo.

**Conteúdo público (YouTube):**
- Trailers / "primeiros 3 minutos" dos episódios exclusivos
- Comentário fixado: "Episódio completo no app: historias-eternas.com"
- Card final no vídeo direcionando ao site

**Funil de conversão:**
```
YouTube (grátis) → Site (cadastro) → Trial 7 dias grátis → Assinante pago
```

### 6.2 — Métricas para acompanhar
- Taxa de conversão YouTube → cadastro
- Taxa de conversão cadastro → trial
- Taxa de conversão trial → pago
- Churn rate (% que cancela por mês) — meta: < 5%
- LTV (Lifetime Value) por assinante

### 6.3 — Suporte ao cliente
- Email de suporte
- WhatsApp Business (opcional mas converte muito no Brasil)
- FAQ no site
- Chatbot simples (opcional)

---

## 💰 ESTIMATIVA DE CUSTOS MENSAIS

### Lançamento (0-100 assinantes)
| Item | Custo |
|---|---|
| Servidor (Hetzner/DigitalOcean) | R$ 30 |
| Bunny Stream (vídeo + CDN) | R$ 50 |
| Domínio + Cloudflare | R$ 10 |
| Stripe (taxa por transação) | 4% + R$ 0,40 por cobrança |
| Apple Developer | R$ 50 (US$ 99/ano ÷ 12) |
| Google Play | R$ 0 (taxa única US$ 25) |
| **Total fixo** | **~R$ 140/mês** |

### Crescimento (1.000 assinantes pagantes)
| Item | Custo |
|---|---|
| Servidor maior | R$ 100 |
| Bunny Stream (mais tráfego) | R$ 400 |
| Email transacional (SendGrid/Resend) | R$ 50 |
| Stripe (~R$ 800 em taxas) | R$ 800 |
| **Total** | **~R$ 1.350/mês** |

**Receita com 1.000 assinantes a R$ 19,90:** R$ 19.900/mês
**Margem líquida:** ~R$ 18.000/mês

---

## ⚠️ RISCOS E COMO MITIGAR

| Risco | Mitigação |
|---|---|
| Apple/Google rejeita o app | Estudar caso Netflix/Spotify, seguir Reader App Guidelines |
| Pirataria dos vídeos | URLs assinadas + DRM (Bunny suporta) |
| Stripe bloqueia conta | Diversificar: ativar também Mercado Pago como backup |
| Churn alto (cancelamentos) | Lançar episódios semanais (assinante fica esperando próximo) |
| LGPD / multa | Termos de uso bem feitos + consentimento explícito |

---

## ✅ CHECKLIST GERAL — O QUE VOCÊ PRECISA FAZER

### Você (criador):
- [ ] Definir nome, logo, identidade visual
- [ ] Abrir CNPJ (MEI ou ME)
- [ ] Conta bancária PJ
- [ ] Roteirizar e gravar 10+ episódios para o lançamento
- [ ] Comprar domínio
- [ ] Criar conta Stripe e Bunny Stream
- [ ] Conta Google Play Developer (US$ 25 única)
- [ ] Conta Apple Developer (US$ 99/ano)
- [ ] Termos de Uso e Política de Privacidade (advogado ou template LGPD)

### Seu parceiro (ou contratado dev):
- [ ] Implementar site (Fase 3) — 4-6 semanas
- [ ] Implementar app Android (Fase 4) — 2-5 semanas
- [ ] Implementar app iOS (Fase 5) — 3-4 semanas
- [ ] Configurar webhooks Stripe e proteção de vídeo
- [ ] Publicar nas lojas

### Total de tempo até iOS publicado:
**3 a 4 meses** (com 1 dev fullstack dedicado)

---

## 🚀 PRÓXIMO PASSO IMEDIATO

1. **Aprovar esta proposta** (HTML que enviei)
2. **Definir nome e marca** do app
3. **Decidir entre Opção A (WordPress + PMP) ou Opção B (custom)**
4. **Começar pela Fase 1 (infra)** — leva 1 semana e libera tudo

Quando você decidir, posso te ajudar a:
- Configurar o WordPress + Paid Memberships Pro do zero
- Ou clonar e adaptar o `justdjango/video-membership` (Django)
- Ou criar do zero em Next.js
- Configurar a conta Bunny Stream e fazer o primeiro upload protegido
- Configurar a Stripe e os planos

Me avisa qual caminho quer seguir e a gente põe a primeira fase no ar nesta semana!
