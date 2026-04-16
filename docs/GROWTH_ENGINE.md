# NEP3 + AIRDROP PULSE — GROWTH ENGINE

> **Conceito:** Cada plataforma se auto-divulga. Usuários nos divulgam também.  
> Automação social + Share dialogs virais + i18n nativo.  
> *Status: Arquitetado. Aguardando chaves de API de redes sociais por plataforma.*

---

## VISÃO GERAL

O Whale Signals já provou o modelo:
- **WhaleTube** → gera vídeos e posta no YouTube automaticamente
- **XWhale** → monitora Twitter e responde em threads relevantes
- **Sentinela** → engaja com comentários em vídeos do YouTube

**O mesmo padrão se replica para cada plataforma:**

| Plataforma | Twitter/X | YouTube | Telegram | Status |
|-----------|-----------|---------|----------|--------|
| Whale Signals | @WhaleSignalsAPP ✅ | ✅ WhaleTube | — | Operando |
| Airdrop Pulse | A criar → `@AirdropPulseApp` | A criar | A criar | **Aguardando chaves** |
| NEP3 | A criar → `@nep3app` | A criar | A criar | **Aguardando chaves** |

**Quando o usuário fornecer as chaves → infra já está pronta, é só plugar.**

---

## PARTE 1: AUTOMAÇÃO SOCIAL POR PLATAFORMA

### 1.1 — Airdrop Pulse Social Engine

#### AP-XBot (equivalente ao XWhale)
**Twitter/X — Conta:** `@AirdropPulseApp`  
**Função:** Engajar com a comunidade de airdrops e novos tokens

```
Comportamentos automáticos:
├── Quando novo token é lançado via AP:
│     "🚀 New token just launched on Airdrop Pulse!
│      $TOKEN — [chain] | Supply: [X]M | Creator: [0x...]
│      📊 View on airdroppulse.app/token/[address]
│      #DeFi #Airdrop #[chain]"
│
├── Quando airdrop verificado é adicionado:
│     "🪂 New verified airdrop detected!
│      Project: [name] | Potential: [$est]
│      ✅ Verified by Neptune AI
│      Claim at airdroppulse.app/airdrop/[id]"
│
├── Monitor de keywords no Twitter:
│     ["airdrop hunter", "new token launch", "gem alert",
│      "100x token", "free airdrop", "#airdrop"]
│     → Reply com: "Check @AirdropPulseApp — verified airdrops only 🪂"
│
└── Daily digest (9AM UTC):
      "📅 Airdrop Pulse Daily:
       • [N] new tokens launched today
       • [N] airdrops verified
       • Top: $TOKEN (+X%)
       🔗 airdroppulse.app"
```

#### AP-Tube (equivalente ao WhaleTube)
**YouTube — Canal:** `Airdrop Pulse`  
**Função:** Conteúdo automático sobre airdrops e token launches

```
Vídeos automáticos gerados e postados:
├── "Top 5 Airdrops da Semana" (toda segunda-feira)
├── "Novo Token Lançado: $TOKEN — Análise Rápida" (cada launch com volume > $10K)
├── "Airdrop Alert: [nome] — Como Participar" (cada airdrop verificado)
└── "Review Semanal Airdrop Pulse — Métricas e Destaques"
```

#### AP-Sentinela (equivalente ao Sentinela)
**YouTube Comments**  
**Função:** Engajar em vídeos sobre criação de tokens e airdrops

```
Keywords alvo em vídeos:
["how to create token", "token launch tutorial", "airdrop guide 2026",
 "how to airdrop crypto", "pump.fun tutorial", "create meme token"]

Comentário padrão:
"Great video! If you want to launch your own token without writing a single line 
of code, check out Airdrop Pulse 🚀 Multi-chain (Solana, Base, BNB) in 60 seconds.
👉 airdroppulse.app"
```

---

### 1.2 — NEP3 Social Engine

#### NEP3-XBot
**Twitter/X — Conta:** `@nep3app`  
**Função:** Posicionar o NEP3 como referência de super app DeFi

```
Comportamentos automáticos:
├── Quando bot executa trade vencedor (Copy Trade / Sniper):
│     "🤖 Neptune Bot just caught a gem!
│      $TOKEN +[X]% in [Y] minutes
│      Copy trade whales automatically → nep3.app/bots
│      #DeFi #CopyTrading #[chain]"
│
├── Monitor de keywords de competidores:
│     ["jupiter swap", "1inch swap", "bridge cross chain", "best dex aggregator"]
│     → Reply: "Have you tried Neptune Swap? Best routes, 0 subscription, 
│               all chains in one place 🔱 nep3.app"
│
├── Milestone automático de volume:
│     A cada $1M em volume via plataforma:
│     "🌊 $[X]M swapped through Neptune!
│      Zero subscription. 0.5% fee only when you use.
│      The ocean is open 🔱 nep3.app"
│
└── Weekly stats (sexta-feira):
      "📊 Neptune Weekly:
       • $[X]M volume
       • [N] bot executions  
       • [N] wallets active
       • [N] tokens launched via Forge
       🔱 nep3.app"
```

#### NEP3-Tube
**YouTube — Canal:** `Neptune Crypto SaaS`  
**Conteúdo automático:**
```
├── "Neptune Bot Made +X% in 24h — Copy Trade Strategy Explained" (por execução destacada)
├── "How to Swap [Token] Cross-Chain in 30 seconds — Neptune" (por novo par popular)
├── "Weekly DeFi Alpha: Neptune's Best Opportunities" (toda semana)
└── "Token Launch Alert: $TOKEN on Neptune Forge" (por launch com tração)
```

---

## PARTE 2: INFRAESTRUTURA DE AUTOMAÇÃO (ARQUITETURA)

### Padrão replicado do Whale Signals (comprovado)

```
Supabase (cada plataforma tem suas próprias tabelas):

Airdrop Pulse:
├── ap_social_queue      → tweets/vídeos agendados para postar
├── ap_comment_queue     → comentários YouTube agendados
├── ap_twitter_monitor   → keywords monitoradas e respostas
└── ap_social_stats      → métricas de engajamento

NEP3:
├── nep3_social_queue
├── nep3_comment_queue
├── nep3_twitter_monitor
└── nep3_social_stats

Edge Functions (mesmo padrão do Whale Signals):
├── ap-twitter-poster    → posta tweets da ap_social_queue
├── ap-twitter-monitor   → monitora keywords, enfileira replies
├── ap-comment-poster    → posta comentários YT (ap_comment_queue)
├── ap-tube              → gera e posta vídeos automáticos
├── nep3-twitter-poster  → análogo para NEP3
├── nep3-twitter-monitor → análogo para NEP3
└── nep3-tube            → análogo para NEP3
```

### Gatilhos de conteúdo

```typescript
// Webhook: novo token lançado no AP → enfileira tweet automático
// api/launch-token.ts (PATCH final) → dispara:
await supabase.from('ap_social_queue').insert({
  type: 'new_token_launch',
  content: generateTweetContent(tokenMeta),
  scheduled_for: new Date().toISOString(),
  platform: 'twitter',
  status: 'scheduled'
})

// Webhook: bot do NEP3 executa trade com >5% lucro → enfileira tweet
await supabase.from('nep3_social_queue').insert({
  type: 'bot_win',
  content: generateBotWinTweet(execution),
  scheduled_for: new Date().toISOString(),
  platform: 'twitter',
  status: 'scheduled'
})
```

### Secrets necessários por plataforma

```bash
# Airdrop Pulse — Twitter/X (@AirdropPulseApp)
AP_TWITTER_API_KEY=...
AP_TWITTER_API_SECRET=...
AP_TWITTER_ACCESS_TOKEN=...
AP_TWITTER_ACCESS_TOKEN_SECRET=...
AP_TWITTER_BEARER_TOKEN=...

# Airdrop Pulse — YouTube
AP_YOUTUBE_CLIENT_ID=...
AP_YOUTUBE_CLIENT_SECRET=...
AP_YOUTUBE_REFRESH_TOKEN=...

# Airdrop Pulse — Telegram Bot
AP_TELEGRAM_BOT_TOKEN=...
AP_TELEGRAM_CHANNEL_ID=...

# NEP3 — Twitter/X (@nep3app)
NEP3_TWITTER_API_KEY=...
NEP3_TWITTER_API_SECRET=...
NEP3_TWITTER_ACCESS_TOKEN=...
NEP3_TWITTER_ACCESS_TOKEN_SECRET=...
NEP3_TWITTER_BEARER_TOKEN=...

# NEP3 — YouTube
NEP3_YOUTUBE_CLIENT_ID=...
NEP3_YOUTUBE_CLIENT_SECRET=...
NEP3_YOUTUBE_REFRESH_TOKEN=...

# NEP3 — Telegram Bot
NEP3_TELEGRAM_BOT_TOKEN=...
NEP3_TELEGRAM_CHANNEL_ID=...
```

**⚠️ Infra pode ser deployada e testada antes das chaves. Quando chaves chegarem → adicionar no Supabase Secrets → funcionamento imediato.**

---

## PARTE 3: SHARE DIALOGS VIRAIS (Usuários nos divulgam)

### Conceito
Cada ação significativa → pop-up/modal de compartilhamento.  
Pré-preenchido. 1 clique para postar. Imagem gerada dinamicamente.

### AP — Share Events

#### Token Lançado
```
Modal: "Seu token está ao vivo! Compartilhe com o mundo 🚀"

Twitter/X pré-preenchido:
"Just launched $[SYMBOL] on @AirdropPulseApp! 🚀
Chain: [Solana/Base/BNB] | Supply: [X]M
Zero code. 60 seconds. Real fees for creators 💰
👉 airdroppulse.app/token/[address]
#DeFi #[chain] #NewToken"

Imagem gerada (OG card):
├── Logo do token (upload do usuário)
├── $SYMBOL em destaque
├── "Launched via Airdrop Pulse"
├── Chain badge
└── QR code para a página do token
```

#### Airdrop Participado
```
Twitter/X pré-preenchido:
"Just claimed [X] $[TOKEN] via @AirdropPulseApp 🪂
Estimated value: $[USD]
Find verified airdrops → airdroppulse.app
#Airdrop #FreeCrypto #DeFi"
```

### NEP3 — Share Events

#### Bot Win
```
Modal: "Seu bot acertou! Compartilhe o resultado 🤖"

Twitter/X pré-preenchido:
"My @nep3app bot just made +[X]% on $[TOKEN]! 🤖
Copy trading whales on [chain] automatically.
Zero subscription. Only 0.3% on wins.
Set it up → nep3.app/bots
#CopyTrading #DeFiBot #[chain]"

Imagem gerada:
├── "+X%" em destaque (verde)
├── Token logo
├── "Neptune Bot Win"
└── "Powered by Whale Signals data 🐳"
```

#### Swap Executado (volume alto > $1K)
```
Twitter/X pré-preenchido:
"Just swapped $[X] from [tokenA] to [tokenB] in [Y] seconds!
Best route across [N] DEXes via @nep3app 🔱
Multi-chain. Zero subscription. Real rates.
👉 nep3.app/swap
#DeFi #Swap #[chain]"
```

### Infraestrutura de OG Images Dinâmicas

```typescript
// api/og/token.tsx — Vercel Edge Function (Satori)
import { ImageResponse } from "@vercel/og"

export default function TokenOG({ symbol, chain, logo, price }) {
  return new ImageResponse(
    <div style={{ background: "#0A0E1A", display: "flex", ... }}>
      <img src={logo} width={80} height={80} style={{ borderRadius: 40 }} />
      <h1 style={{ color: "#00D4FF" }}>${symbol}</h1>
      <p style={{ color: "#fff" }}>Launched via Airdrop Pulse</p>
      <div style={{ color: "#8B9CB6" }}>{chain} • {price}</div>
    </div>,
    { width: 1200, height: 630 }
  )
}

// URL do OG tag:
// <meta property="og:image" content="https://airdroppulse.app/api/og/token?symbol=XYZ&chain=solana&..." />
```

### Plataformas de Share
```
Cada share dialog terá botões para:
├── 𝕏 Twitter/X (principal)
├── ✈️ Telegram (segundo mais importante no crypto)
├── 💬 WhatsApp
├── 🔗 Copiar link (com UTM tracking)
└── 📸 Baixar imagem (para Instagram Stories)
```

---

## PARTE 4: I18N — INTERNACIONALIZAÇÃO NATIVA

### Estratégia

**Não depender apenas de "Google Translate" do browser.**  
Servir o conteúdo já traduzido para cada idioma.

### Idiomas Prioritários (cobertura do mercado crypto)

| Idioma | Código | Mercado alvo | Prioridade |
|--------|--------|-------------|-----------|
| Inglês | `en` | Global | 🔴 P0 — Base |
| Português | `pt` | Brasil (2º maior mercado crypto) | 🔴 P0 |
| Espanhol | `es` | LATAM | 🟡 P1 |
| Chinês Simplificado | `zh` | China/Asia | 🟡 P1 |
| Coreano | `ko` | Coreia do Sul | 🟡 P1 |
| Japonês | `ja` | Japão | 🟠 P2 |
| Russo | `ru` | CIS | 🟠 P2 |
| Turco | `tr` | Turquia (top crypto per capita) | 🟠 P2 |

### Implementação Técnica

```bash
# Instalar
npm install react-i18next i18next i18next-browser-languagedetector i18next-http-backend
```

```typescript
// src/lib/i18n.ts
import i18n from "i18next"
import { initReactI18next } from "react-i18next"
import LanguageDetector from "i18next-browser-languagedetector"
import Backend from "i18next-http-backend"

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: "en",
    supportedLngs: ["en", "pt", "es", "zh", "ko", "ja", "ru", "tr"],
    detection: {
      order: ["navigator", "htmlTag", "cookie", "localStorage"],
      // Detecta automaticamente o idioma do browser
    },
    backend: {
      loadPath: "/locales/{{lng}}/{{ns}}.json"
    },
    ns: ["common", "swap", "bots", "airdrop", "launch"],
    defaultNS: "common"
  })
```

```
public/locales/
├── en/
│   ├── common.json    → { "swap": "Swap", "connect": "Connect Wallet", ... }
│   ├── swap.json
│   ├── bots.json
│   ├── airdrop.json
│   └── launch.json
├── pt/
│   ├── common.json    → { "swap": "Trocar", "connect": "Conectar Carteira", ... }
│   └── ...
├── es/
├── zh/
└── ko/
```

```tsx
// Uso nos componentes
import { useTranslation } from "react-i18next"

function ConnectButton() {
  const { t } = useTranslation()
  return <button>{t("connect")}</button>
  // Mostra "Connect Wallet" para EN, "Conectar Carteira" para PT, etc.
}
```

### Seletor de Idioma Manual
```tsx
// Componente <LanguageSwitcher /> no Header
// Dropdown com bandeiras: 🇺🇸 🇧🇷 🇪🇸 🇨🇳 🇰🇷 🇯🇵 🇷🇺 🇹🇷
// Persiste em localStorage
// Atualiza <html lang="XX"> dinamicamente
```

### SEO Multi-idioma (hreflang)
```html
<!-- No <head> de cada página -->
<link rel="alternate" hreflang="en" href="https://nep3.app/?lang=en" />
<link rel="alternate" hreflang="pt" href="https://nep3.app/?lang=pt" />
<link rel="alternate" hreflang="es" href="https://nep3.app/?lang=es" />
<link rel="alternate" hreflang="zh" href="https://nep3.app/?lang=zh" />
<link rel="alternate" hreflang="ko" href="https://nep3.app/?lang=ko" />
<link rel="alternate" hreflang="x-default" href="https://nep3.app/" />
```

### Estratégia de Tradução (custo zero)
1. **Fase 1:** Inglês + Português (feito manualmente — são as línguas do dono)
2. **Fase 2:** Usar DeepL API free tier (500K chars/mês) para ES, ZH, KO
3. **Fase 3:** Community translations (usuários nativos revisam e contribuem)
4. **Fase 4:** OpenAI GPT-4o para novas strings automaticamente

### Para o Airdrop Pulse (mesmo padrão)
```
Aplicar o MESMO sistema react-i18next.
Prioridade: EN + PT (fase 1)
Depois ES + ZH (fase 2)
AP tem mais texto de UI → tradução manual do EN/PT primeiro.
```

---

## PARTE 5: CHECKLIST DE IMPLEMENTAÇÃO

### Pré-requisitos (usuário deve fornecer)
- [ ] Conta Twitter/X `@AirdropPulseApp` criada
- [ ] Conta Twitter/X `@nep3app` criada
- [ ] Canal YouTube "Airdrop Pulse" criado
- [ ] Canal YouTube "Neptune Crypto SaaS" criado
- [ ] Bot Telegram para AP criado (@BotFather)
- [ ] Bot Telegram para NEP3 criado (@BotFather)
- [ ] Keys Twitter (Consumer Key/Secret + Access Token) para cada conta
- [ ] YouTube OAuth credentials para cada canal

### Implementação (assim que chaves chegarem)

#### Para Airdrop Pulse
```bash
# 1. Criar tabelas no Supabase do AP (Firebase → ou novo projeto Supabase para AP?)
# 2. Adicionar Edge Functions: ap-twitter-poster, ap-twitter-monitor, ap-tube, ap-sentinela
# 3. Adicionar secrets no Supabase/Vercel
# 4. Deploy → funcionamento imediato
```

#### Para NEP3
```bash
# 1. Criar projeto Supabase para NEP3 (ou compartilhar com AP)
# 2. Adicionar Edge Functions: nep3-twitter-poster, nep3-twitter-monitor, nep3-tube
# 3. Adicionar secrets
# 4. Deploy
```

#### i18n (independe de chaves externas — pode fazer agora)
```bash
# Airdrop Pulse:
cd Airdrop-Pulse
npm install react-i18next i18next i18next-browser-languagedetector
# Criar src/lib/i18n.ts
# Criar public/locales/en/ e public/locales/pt/
# Substituir strings hardcoded por t("key") nos componentes

# NEP3: já iniciar com i18n desde o setup (Fase 0)
```

---

## RESUMO ESTRATÉGICO

```
CADA PLATAFORMA = PRODUTO + MEGAFONE AUTOMÁTICO

Airdrop Pulse:
  ┌─ Produto: criar tokens, descobrir airdrops
  └─ Megafone: @AirdropPulseApp (Twitter) + YouTube + Telegram
               Share dialogs nos momentos de launch/claim
               i18n: PT/EN/ES/ZH

NEP3:
  ┌─ Produto: swap, bots, DeFi, tudo multi-chain
  └─ Megafone: @nep3app (Twitter) + YouTube + Telegram
               Share dialogs em bot wins, swaps grandes, milestones
               i18n: PT/EN/ES/ZH/KO

Whale Signals: já funciona. Serve de prova de conceito para os demais.

RESULTADO:
  ✅ Plataforma se divulga 24/7 automaticamente
  ✅ Usuários satisfeitos amplificam organicamente
  ✅ Cobertura global via i18n
  ✅ Custo: $0 (APIs gratuitas + Supabase free tier)
```
