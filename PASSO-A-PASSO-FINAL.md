# 📘 PASSO A PASSO — App de Romance Stories (limpo, sem design)

> Documento APENAS de execução. A aparência do app está em `app.html` e a landing em `romance.html` — separados.

---

## 🎯 OBJETIVO

Construir uma plataforma de assinatura para histórias de romance narradas, modelo "Netflix":
- Site público (landing) → vende a assinatura
- Área de membros (web + app Android + app iOS) → consome o conteúdo
- Vídeos hospedados em servidor próprio (não YouTube)
- Stripe para pagamento

**Inspiração:** mystoriesromance.com (US$ 14,99/mês), canais @romancestoriesdavid, @marcusstories123, @liebesgeschichten1, @romancestories231 — mas com qualidade visual e UX MUITO superiores.

---

## 📐 ARQUITETURA TÉCNICA

```
                    ┌─────────────────────────┐
                    │  YOUTUBE (gratuito)      │  ← Marketing
                    │  Trailers, Book 1 free   │
                    └────────────┬─────────────┘
                                 ▼
┌─────────────────────────────────────────────────────────────┐
│            LANDING (romanceflix.com.br)                     │
│   - Apresenta produto, preço, depoimentos                   │
│   - Cadastro + Stripe Checkout (R$ 19,90/mês)               │
└──────────────────────────────┬──────────────────────────────┘
                               ▼
┌─────────────────────────────────────────────────────────────┐
│      ÁREA DE MEMBROS (app.romanceflix.com.br + app)         │
│   - Catálogo (Mafia Boss, Bilionário, etc.)                 │
│   - Player com Books (Book 1 grátis, Book 2+ pago)          │
│   - Modo audiobook (só áudio)                               │
│   - Continue assistindo, minha lista                        │
└──────────────────────────────┬──────────────────────────────┘
                               ▼
        ┌──────────────────────────────────────┐
        │   API + STRIPE WEBHOOK + DATABASE     │
        │   - Valida assinatura                 │
        │   - Gera URL assinada do vídeo        │
        └────────────┬──────────────────────────┘
                     ▼
       ┌────────────────────────────────┐
       │   BUNNY STREAM / MUX            │  ← Vídeos
       │   HLS protegido + DRM           │
       └────────────────────────────────┘
```

---

## 🗓️ CRONOGRAMA DE 12 SEMANAS (3 meses)

### Semana 1 — Decisões + Empresa
- Definir nome final, logo, cores, domínio
- Abrir CNPJ
- Conta bancária PJ
- Comprar domínio (.com.br no Registro.br ou .com no Cloudflare)
- Termos de Uso + Política de Privacidade (LGPD)

### Semana 2 — Infraestrutura
- Criar conta Bunny Stream → criar Video Library
- Criar conta Stripe (CNPJ) → ativar Stripe Billing + Pix
- Criar 1 produto Stripe: "Acesso Total" R$ 19,90/mês com 7 dias trial
- Provisionar servidor (Hetzner/DigitalOcean/Railway)
- Configurar domínio + Cloudflare + SSL

### Semanas 3-6 — Backend + Site
- Banco de dados (PostgreSQL): tabelas users, subscriptions, stories, books, watch_progress
- Autenticação (email/senha + Google login)
- Integração Stripe Checkout + webhook
- API endpoints: lista de histórias, detalhes, URL assinada do vídeo
- Frontend da landing (vender)
- Frontend da área de membros (catálogo + player)

### Semanas 7-9 — App Android
- Configurar PWA do site (manifest.json + service worker)
- Empacotar como TWA usando Bubblewrap
- Conta Google Play Developer (US$ 25 única)
- Submeter na Play Store
- (Opcional) Migrar pra React Native quando validar tração

### Semanas 10-12 — App iOS
- Conta Apple Developer (US$ 99/ano)
- Solicitar D-U-N-S Number (gratuito)
- Aplicar para Reader App Entitlement
- Build do app (TWA não funciona pra iOS — usar React Native ou Capacitor)
- Submeter na App Store

---

## ✅ CHECKLIST POR ÁREA

### A. CRIADOR (você)
- [ ] Nome, logo, identidade visual
- [ ] CNPJ aberto
- [ ] Conta bancária PJ
- [ ] Domínio comprado
- [ ] Mínimo de 15 histórias prontas pra lançamento (cada com Book 1, 2, 3)
- [ ] Roteiro definido para próximas 12 semanas (3 episódios/semana)
- [ ] Termos de Uso + Política de Privacidade (advogado ou template LGPD)
- [ ] Conta Apple Developer (US$ 99/ano)
- [ ] Conta Google Play (US$ 25 única)

### B. INFRAESTRUTURA (dev junior pode fazer)
- [ ] Servidor provisionado (Hetzner ou DigitalOcean)
- [ ] Domínio + Cloudflare + SSL configurados
- [ ] Email transacional (Resend ou SendGrid)
- [ ] Bunny Stream: Video Library criada
- [ ] Bunny Stream: chave de signing token gerada
- [ ] Stripe: produto criado, price_id anotado
- [ ] Stripe: webhook endpoint registrado

### C. BACKEND (dev fullstack)
- [ ] Banco PostgreSQL com schema:
  - users (id, email, password_hash, stripe_customer_id, created_at)
  - subscriptions (id, user_id, stripe_sub_id, status, current_period_end)
  - stories (id, slug, title, description, genre, cover_url)
  - books (id, story_id, number, title, video_id, duration, is_free)
  - watch_progress (user_id, book_id, position_seconds, completed_at)
  - favorites (user_id, story_id, added_at)
- [ ] Auth (JWT ou sessions)
- [ ] Endpoint POST /api/stripe/create-checkout-session
- [ ] Endpoint POST /api/stripe/webhook (validar assinatura)
- [ ] Endpoint GET /api/stories (lista + filtros)
- [ ] Endpoint GET /api/stories/:slug (detalhes + books)
- [ ] Endpoint GET /api/videos/:bookId/url (gera URL assinada do Bunny)
- [ ] Endpoint POST /api/progress (salva continue assistindo)
- [ ] Middleware: bloquear vídeo pago se assinatura inativa

### D. FRONTEND — Landing (`romance.html` é o protótipo)
- [ ] Hero impactante com CTA
- [ ] Seção "como funciona"
- [ ] Catálogo amostra (com cadeado em Books pagos)
- [ ] Página de pricing (R$ 19,90/mês — plano único)
- [ ] Depoimentos (assim que tiver)
- [ ] FAQ
- [ ] Cadastro + Stripe Checkout

### E. FRONTEND — Área de Membros (`app.html` é o protótipo)
- [ ] Top bar com busca + avatar + notificações
- [ ] Sidebar com categorias e gêneros
- [ ] Hero rotativo (4 histórias destaque)
- [ ] Row "Continue Assistindo" com barra de progresso
- [ ] Row "Top 10" com numeração grande
- [ ] Rows por gênero (Mafia Boss, Bilionário, Segunda Chance, Amor Proibido)
- [ ] Grid de "humor" (Quente, Dark, Pra Chorar, etc.)
- [ ] Modal de detalhes ao clicar em card (com lista de Books)
- [ ] Player page com:
  - [ ] Vídeo principal (Bunny Stream HLS)
  - [ ] Botão "modo só áudio" (esconde vídeo)
  - [ ] Lista de Books da história
  - [ ] Botão próximo Book auto-play
  - [ ] Comentários (opcional na v1)
  - [ ] Recomendados ao final

### F. APP ANDROID
- [ ] PWA do site (manifest.json + service worker para offline básico)
- [ ] Bubblewrap pra empacotar como TWA: `npx @bubblewrap/cli init`
- [ ] Ícone e splash screen
- [ ] Submeter Play Store (ficha de loja, screenshots, descrição)
- [ ] **Atenção:** sem botão de assinar dentro do app (regra Google)

### G. APP iOS
- [ ] D-U-N-S Number solicitado
- [ ] Conta Apple Developer ativa
- [ ] Pedido de Reader App Entitlement
- [ ] Build com Capacitor (mais simples) ou React Native
- [ ] TestFlight (testar internamente)
- [ ] Submeter App Store
- [ ] **Atenção:** sem botão de assinar dentro do app (regra Apple)

---

## 💰 CUSTOS

### Custos fixos mensais (lançamento)
| Item | Valor |
|---|---|
| Servidor (Hetzner) | R$ 30 |
| Bunny Stream | R$ 50 |
| Domínio + Cloudflare | R$ 10 |
| Email transacional | R$ 0 (free tier) |
| Apple Developer | R$ 50 (US$ 99/ano ÷ 12) |
| **Total** | **~R$ 140/mês** |

### Custos variáveis
- Stripe: 4% + R$ 0,40 por transação (no Brasil)
- Bunny Stream: ~US$ 0,005 por GB entregue
- Google Play: US$ 25 (taxa única)

### Custos de desenvolvimento (1 dev fullstack contratado)
- Site + backend: R$ 8.000 a R$ 25.000
- App Android (TWA): R$ 1.000 a R$ 3.000
- App iOS: R$ 5.000 a R$ 12.000
- **Total:** R$ 14.000 a R$ 40.000

---

## 📊 MÉTRICAS-CHAVE (acompanhar desde o dia 1)

| Métrica | O que é | Meta inicial |
|---|---|---|
| Conversão YT → cadastro | % das views YouTube que viraram cadastro | 2-5% |
| Conversão cadastro → trial | % dos cadastros que iniciam trial | 30-50% |
| Conversão trial → pago | % dos trials que viram pagantes | 30-50% |
| Churn mensal | % que cancelam no mês | < 7% |
| LTV | Valor médio gerado por cliente até cancelar | R$ 100+ (5 meses) |
| CAC | Custo de aquisição (se rodar ads) | < R$ 30 |

---

## 🚫 PEGADINHAS PARA EVITAR

1. **NÃO** colocar botão de assinar dentro dos apps (Google/Apple proibirão)
2. **NÃO** subir vídeos sem URL assinada — vai ser pirateado em 24h
3. **NÃO** começar sem 15+ histórias prontas — assinante cancela se acha que tem pouco conteúdo
4. **NÃO** usar Stripe sem ativar webhook — assinaturas vão "quebrar" sem você saber
5. **NÃO** prometer conteúdo +18 explícito — Apple/Google rejeitam o app
6. **NÃO** misturar marketing (landing) com app (área logada) no mesmo deploy — separar desde o início

---

## 🎯 PRÓXIMOS 3 PASSOS IMEDIATOS

1. **Validar a apresentação** com seu parceiro (mostrar `romance.html` + `app.html`)
2. **Definir nome final** e comprar o domínio
3. **Abrir CNPJ + Stripe + Bunny** (paralelo, leva 1 semana)

Quando estiver pronto pra começar, me chama que eu inicio o setup técnico (escolher entre WordPress+PMP rápido ou stack custom Next.js).
