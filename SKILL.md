# Café Matinal — App Knowledge Skill

## Visão Geral

**Café Matinal** é um sistema de produtividade pessoal single-page app (SPA) para o Vasco (BBTOP20 / Borja on Stocks + Investidor Prudente com César Borja).

- **URL produção:** https://vasco-cell.github.io/cafe-matinal/
- **Repositório:** https://github.com/vasco-cell/cafe-matinal (branch `main`, GitHub Pages)
- **Stack:** HTML único + React 18 UMD + Tabler Icons CDN (sem bundler, sem build step)

---

## Arquitectura Técnica

### Stack
```
index.html (ficheiro único ~2300 linhas)
├── <style>         — CSS vars + classes utilitárias
├── React 18 UMD    — carregado de jsDelivr CDN
├── Tabler Icons    — cdn.jsdelivr.net/npm/@tabler/icons-webfont
├── <script>        — toda a lógica da app em React.createElement (ce())
└── data/           — ficheiros JSON no repo (sincronização GitHub)
    ├── notes.json, mem.json, habits.json, convs.json
    ├── tasks.json, reps.json, pessoal.json, fin.json, prios.json
```

### Funções de Storage Globais
| Função | O que faz |
|--------|-----------|
| `sg(key)` | Lê de GitHub raw + localStorage como fallback |
| `ss(key, value)` | Guarda em localStorage + escreve `data/*.json` via GitHub API |
| `aiC(msgs, sys, max)` | Chama Claude Haiku directo via browser, chave de `localStorage("bj-ak")` |
| `whF(params)` | Webhook Make.com para acções ClickUp |
| `fetchData(silent?)` | Carrega tarefas do ClickUp API + sync silencioso a cada 60s |

### Credenciais (NUNCA no código — só em localStorage ou fragmentado)
- **GitHub token:** fragmentado no código como `['ghp', '_i69NK4x5cr', 'swgJ7Amo3Km', 'iJm4Qky8T4c5Up7']`
- **ClickUp API key:** em `localStorage("cu_key")` — inserida pelo utilizador
- **Claude API key:** em `localStorage("bj-ak")` — inserida pelo utilizador nas Definições
- **Make.com webhook:** `https://hook.eu1.make.com/77msmctjelcxsi2zss5yttvuyys29rlu`
- **ClickUp workspace:** `24309030`, user ID `206530598`
- **Spotify playlist:** `3Xea2Lx3iyn9C2lpzC0MVy`

---

## Separadores / Componentes (19 total)

| Separador | Componente | O que faz | Dados |
|-----------|------------|-----------|-------|
| Hoje | `PHoje` | Dashboard: top 3 IA, tarefas urgentes, hábitos hoje, saldo mês | ClickUp + localStorage |
| Prioridades | `PPrios` | Lista de prioridades ordenáveis, marcar como feito | `data/prios.json` |
| Assistente | `PAssist` | Chat LLM completo, histórico de conversas, arquivar/apagar | `data/convs.json` |
| Tarefas | `PTarefas` | Lista tarefas ClickUp, filtros, modal criar (lista+data+prioridade) | ClickUp API |
| Calendário | `PCal` | 3 vistas (Mês/Semana/Agenda), criar tarefa/evento por dia | ClickUp + Google Cal |
| Inbox | `PInbox` | Captura rápida: tarefa, nota, memória | localStorage |
| Foco | `PFoco` | Timer Pomodoro (25/45/60/90min) + Spotify embed persistente | localStorage |
| Notas | `PNotas` | Notas com AI expand/resumo, editar (EditModal), apagar | `data/notes.json` |
| Memória | `PMemoria` | Base de conhecimento pessoal por categorias, editar/apagar | `data/mem.json` |
| Revisão Semanal | `PRevisao` | Template guiado de revisão semanal | localStorage |
| Relatórios | `PRelatorios` | Geração algorítmica: resumo, conquistas, urgentes, finanças, hábitos | localStorage → `data/reps.json` |
| Previsões | `PPrevisoes` | Previsão algorítmica 7 dias com risco, foco, alertas | localStorage |
| Templates | `PTemplates` | Templates de tarefas reutilizáveis | localStorage |
| Rotinas | `PRotinas` | Rotinas diárias/semanais como template | localStorage |
| Pessoal | `PPessoal` | Notas pessoais por categoria (Família, Saúde, Carreira…) | `data/pessoal.json` |
| Finanças | `PFinancas` | Receitas/despesas por mês, saldo, gráfico | `data/fin.json` |
| Analytics | `PAnalytics` | Métricas: tarefas, hábitos, sessões foco, finanças | computed |
| Hábitos | `PHabitos` | Grid 5 semanas (Seg→Dom), streak, marcar hoje, criar/apagar | `data/habits.json` |
| Definições | `PDefs` | Chave API Claude, estado integrações | localStorage |

---

## Componentes Globais

### FloatingAssistant (Beirafonte)
- Botão flutuante `✦` arrastável (posição em `localStorage("bj-fab-pos")`)
- Mini LLM directo à API Claude Haiku
- Contexto automático: tarefas, hábitos, finanças, eventos
- Acções: `criar_tarefa`, `fechar_tarefa`, `navegar`
- Badge de mensagens não lidas
- Sugestões rápidas pré-definidas

### EditModal (universal)
- Props: `item, onSave, onDelete, onClose, noTitle, label`
- Usado em: Notas, Memória
- Tem botão "Eliminar" vermelho + "Guardar"

### NotificationsPanel
- Sino 🔔 na topbar com badge vermelho
- Alertas: tarefas atrasadas, urgentes, eventos hoje, hábitos pendentes, amanhã

### GlobalSearch
- Tecla `/` abre
- Busca: tarefas, notas, memórias, prioridades, páginas

---

## CSS / Design System

```css
:root {
  --bg-page: #FAF9F7;    --bg-app: #fff;         --bg-nav: #fff;
  --bg-surface: #FAF9F7; --border: #E5E3DC;       --border-light: #F0EEE8;
  --text-1: #1A1A2E;     --text-2: #6B6860;       --text-3: #9B9890;
  --purple: #7C3AED;     --purple-bg: #EDE9FE;    --accent: #1A1A2E;
}
html.dark {
  --bg-page: #0F0F0F;    --bg-app: #1A1A2E;       --bg-nav: #111118;
  --bg-surface: #1E1E2E; --border: #2D2D45;
  --text-1: #E8E6F0;     --text-2: #9B99B0;
  --purple: #A78BFA;     --accent: #7C3AED;
}
```

**Classes utilitárias:** `.cd` (card), `.bt` (button secondary), `.bt.dk` (button primary dark), `.ip` (input), `.pt` (page title), `.ps` (page subtitle), `.sp` (spinner), `.bn` (nav button)

---

## Mobile / Responsive

- **Breakpoint:** 820px
- Sidebar escondida por defeito no mobile, abre com ☰ (`mob-ham`)
- Overlay escuro ao abrir sidebar (`mob-overlay`)
- Nav fecha automaticamente ao clicar num item
- Hábitos grid: `clamp(34px, 9.5vw, 80px)` por célula
- Spotify player: 380px centrado, `translateX(-50%)`
- Foco mode buttons: `overflowX: auto`

---

## Sincronização de Dados

| Dado | Onde guarda | Sincroniza entre dispositivos? |
|------|-------------|-------------------------------|
| Tarefas | `data/tasks.json` + ClickUp | ✅ Sim |
| Notas | `data/notes.json` | ✅ Sim |
| Memória | `data/mem.json` | ✅ Sim |
| Hábitos + logs | `data/habits.json` | ✅ Sim |
| Conversas AI | `data/convs.json` | ✅ Sim |
| Relatórios | `data/reps.json` | ✅ Sim |
| Prioridades | `data/prios.json` | ✅ Sim |
| Finanças | `data/fin.json` | ✅ Sim |
| Pessoal | `data/pessoal.json` | ✅ Sim |
| Chave API Claude | `localStorage("bj-ak")` | ❌ Por dispositivo |
| Posição FAB | `localStorage("bj-fab-pos")` | ❌ Por dispositivo |
| Brief diário | `localStorage("bj-brief-sent")` | ❌ Por dispositivo |

---

## Integrações

| Serviço | Como | O que faz |
|---------|------|-----------|
| **ClickUp** | REST API directo | Tarefas, criar/fechar, prioridades |
| **Google Calendar** | Via Make.com webhook | Eventos no calendário |
| **Claude (Haiku 4.5)** | API directo via browser | Assistente, briefings, análise |
| **Make.com** | Webhook | Proxy ClickUp actions, email briefing |
| **Microsoft 365** | MCP via API call | Email (em desenvolvimento) |
| **Spotify** | Embed iframe | Player de música no Foco |
| **GitHub** | API REST | Storage de todos os dados JSON |

---

## Atalhos de Teclado

| Tecla | Acção |
|-------|-------|
| `/` | Pesquisa global |
| `N` | Nova tarefa |
| `A` | Abrir assistente flutuante |
| `D` | Toggle dark mode |
| `ESC` | Fecha modais/pesquisa |

---

## Limitações Conhecidas / Backlog

| Área | Limitação / Melhoria potencial |
|------|-------------------------------|
| AI email | Microsoft 365 MCP no browser requer aprovação OAuth — não finalizado |
| Relatórios | Apenas algorítmico; versão com AI dependia de make.com não configurado |
| Previsões | Apenas algorítmico; prompt AI existe mas estava a falhar JSON parse |
| Rotinas | Criar novas rotinas funciona mas remover ainda não implementado |
| Templates | Criar/usar funciona; editar template individual em falta |
| Analytics | Dados históricos só desde início do uso da app (sem histórico ClickUp) |
| Spotify | Funciona mas requer conta Spotify premium para playback |
| Offline | Sem service worker; sem internet a app não carrega |
| Segurança | GitHub token fragmentado no código (acesso público ao repo) |
| Emails diários | Cenário Make.com envia 1 briefing/dia — possível duplicar se localStorage limpo |

---

## Deploy

```bash
# Método preferido: git directo
git clone https://TOKEN@github.com/vasco-cell/cafe-matinal.git
# editar index.html
git add index.html && git commit -m "..." && git push origin main
# GitHub Actions faz deploy automático para GitHub Pages
```

**ATENÇÃO:** GitHub Push Protection bloqueia commits com chaves API.  
Nunca incluir `sk-ant-api03-...` no código — usar sempre `localStorage`.

---

## Contexto de Negócio (BBTOP20)

- **Vasco Neves** — marketing, conteúdo e automações
- **César Borja** — fundador, criador de conteúdo (Borja on Stocks, Investidor Prudente)
- **Equipa:** Filipa, Catarina, Diana, André
- **Email principal:** vasco@borjaonstocks.com
- **Marcas:** BBTOP20, Borja on Stocks, Investidor Prudente, Camaradas Investidores, Borja Insights
- **Produto recente:** livro "Rico em Ações" com sessões Feira do Livro Lisboa

