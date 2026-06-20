# 🦘 Caderninho

Gerador de **planejamento semanal com IA** para professoras da Educação Infantil (BNCC).
A professora digita o tema da semana e o app preenche automaticamente o resumo, os
objetivos de aprendizagem e a rotina dos cinco dias — tudo editável e pronto para
imprimir ou exportar para Word.

Faz parte do ecossistema **Walladm** e funciona como porta de entrada (funil) para o
sistema de gestão escolar.

---

## ✨ O que faz

- Preenchimento automático do planejamento a partir de um tema (ex.: "Os animais da fazenda").
- Objetivos da **BNCC** corretos por faixa etária (Berçário `EI01`, Maternal `EI02`, Pré `EI03`).
- Semanário completo: 5 dias × 6 momentos da rotina (acolhida, atividade dirigida,
  brincadeiras, cuidado, movimento, história).
- Tudo editável; botão "Refazer dia" para regenerar um dia específico.
- Exportar **Word (.doc)** e **PDF** direto do navegador.
- Cota gratuita por semana + tela de **contribuição via Pix** ao atingir o limite.

---

## 🧱 Stack

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js 15 (App Router) · React · TypeScript · Tailwind |
| IA | Anthropic API — Claude Sonnet (`@anthropic-ai/sdk`) |
| Rate limit / cota | Upstash Redis (`@upstash/ratelimit`) |
| Pagamento | Pix estático (copia-e-cola + QR) |
| Deploy | Vercel |

---

## 🔄 Como funciona (arquitetura)

```
Navegador (Caderninho)
        │  POST /api/gerar  { prompt }
        ▼
/api/gerar  (Vercel · serverless)   ← a ANTHROPIC_API_KEY vive AQUI, nunca no client
        │
        ├─ checa cota por IP (Upstash Redis): 3 planejamentos grátis/semana
        │
        ├─ dentro da cota ─► Claude Sonnet (1 chamada de resumo/objetivos + 1 por dia)
        │                    └─► devolve JSON que preenche a tela
        │
        └─ cota cheia ─────► resposta 402 → tela de contribuição (Pix) + CTA Walladm

Exportar Word / PDF  →  acontece 100% no navegador (não passa pelo servidor)
```

Pontos-chave:

- **A chave da IA nunca vai para o navegador.** O front só conhece a rota `/api/gerar`.
- **A cota é validada no servidor (por IP).** O contador no client é apenas UX — a
  proteção real dos tokens está no backend.
- **1 geração = 1 unidade de cota** (não por chamada à IA).

---

## 🚀 Rodando localmente

Pré-requisitos: Node 18+ e uma conta na [Upstash](https://upstash.com) (free tier).

```bash
git clone https://github.com/<seu-usuario>/caderninho.git
cd caderninho
npm install
cp .env.example .env.local   # preencha as variáveis (abaixo)
npm run dev                  # http://localhost:3000
```

---

## 🔑 Variáveis de ambiente

Crie `.env.local` a partir de `.env.example`:

| Variável | Descrição |
|---|---|
| `ANTHROPIC_API_KEY` | Chave da API da Anthropic (server-side). |
| `UPSTASH_REDIS_REST_URL` | URL do Redis na Upstash. |
| `UPSTASH_REDIS_REST_TOKEN` | Token REST da Upstash. |
| `PIX_KEY` | Sua chave Pix (para exibição). |
| `PIX_PAYLOAD` | Payload Pix copia-e-cola (com CRC16 válido). |
| `NEXT_PUBLIC_WALLADM_URL` | URL do Walladm para o CTA/funil. |
| `FREE_LIMIT` | Planejamentos grátis por janela (default `3`). |

> ⚠️ O `PIX_PAYLOAD` precisa terminar com o CRC16 correto, senão o banco recusa o
> código. Use um gerador confiável de Pix copia-e-cola estático.

---

## ☁️ Deploy na Vercel

1. Importe o repositório na Vercel.
2. Em **Settings → Environment Variables**, adicione todas as variáveis acima.
3. Deploy. A rota `/api/gerar` roda como serverless function automaticamente.

```bash
# ou via CLI
vercel
vercel --prod
```

---

## 💰 Monetização

O núcleo de geração é desacoplado da monetização, então a mesma base cobre três
modelos simultâneos:

1. **Funil do Walladm** *(principal)* — CTA em pontos estratégicos levando escolas ao
   sistema de gestão. A conversão de valor real é a escola virar cliente.
2. **Contribuição via Pix** — após a cota gratuita, tela de contribuição de baixa
   fricção (a partir de R$5). Pix estático, sem webhook.
3. **Anúncio** — um único slot discreto (rodapé e/ou após a geração). Começar com
   house ads (Walladm + afiliados de material pedagógico) antes de redes externas.
   **Nunca** inserir anúncio no meio do fluxo de geração.

### Roadmap de monetização (standalone)

- Login simples (magic link) + plano **Pro**: planejamentos ilimitados, sem anúncio,
  `.docx` com a marca da escola, histórico salvo.
- Trocar Pix estático por cobrança recorrente (**Asaas**).
- Webhook de pagamento para desbloqueio automático.

---

## 🗺️ Roadmap técnico

- [ ] `/api/docx` — geração de `.docx` real no servidor (lib `docx`) com logo e
      semanário em paisagem.
- [ ] Autenticação e contas.
- [ ] Histórico de planejamentos por usuário.
- [ ] Suporte a mais formatos de planejamento (quinzenal, mensal).

---

## 📁 Estrutura

```
caderninho/
├─ app/
│  ├─ page.tsx              # UI principal
│  ├─ api/
│  │  ├─ gerar/route.ts     # proxy da IA + checagem de cota
│  │  └─ docx/route.ts      # (roadmap) geração de .docx real
├─ components/              # componentes da UI
├─ lib/                     # prompts, ratelimit, helpers
├─ public/
│  └─ walladm-logo.svg
├─ .env.example
└─ README.md
```

---

Feito com 💚 no ecossistema **Walladm** · Rio de Janeiro, RJ.
