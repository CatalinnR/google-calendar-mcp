# Google Calendar MCP — Fișă de proiect

## Ce este acest proiect?

Integrarea **Google Calendar** direct în **Claude Code** prin protocolul MCP (Model Context Protocol).  
Permite lui Claude să creeze, citească, modifice și șteargă evenimente din calendarul tău Google, fără să deschizi browser-ul.

---

## Cum funcționează?

```
Tu (în Claude Code)
      ↓
   Claude
      ↓
  MCP Server (google-calendar)
      ↓
  Google Calendar API
      ↓
  Calendarul tău Google
```

Claude trimite comenzi către un server MCP local, care comunică cu API-ul Google prin credențialele tale OAuth.

---

## Configurare

| Element | Locație |
|---------|---------|
| Credențiale OAuth | `D:\ClaudeCode\Claude Code Calendar\` |
| Configurare MCP | Settings globale Claude Code |
| Cont conectat | catalin.rat96@gmail.com |

> La prima utilizare după restart, necesită autentificare OAuth (se deschide browser pentru confirmare).

### ⚠️ Există DOUĂ fișiere de configurare, nu unul

MCP-ul e configurat separat pentru Claude Desktop și pentru sesiunile Claude Code CLI. La orice schimbare a căii către fișierul de credențiale, ambele trebuie actualizate:

| Config | Locație |
|--------|---------|
| Claude Desktop (Windows) | `C:\Users\catal\AppData\Local\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json` |
| Claude Code CLI (WSL) | `/home/catal/.claude.json` → secțiunea `mcpServers.google-calendar` |

Ambele au cheia `env.GOOGLE_OAUTH_CREDENTIALS` cu calea către fișierul JSON.

---

## Ce poate face Claude prin acest MCP?

| Acțiune | Funcționează? |
|---------|--------------|
| Creare eveniment | ✅ |
| Citire evenimente | ✅ |
| Modificare eveniment | ✅ |
| Ștergere eveniment | ✅ |
| Setare reminder popup | ✅ |
| Setare reminder email | ✅ |
| Verificare oră curentă | ✅ |
| Verificare disponibilitate (free/busy) | ✅ |

---

## Tipuri de notificări

| Tip | Cum ajunge la tine | Funcționează offline? |
|-----|-------------------|----------------------|
| **popup** | Notificare pe ecran (browser sau app Google Calendar pe telefon) | ❌ Nu |
| **email** | Email în Gmail | ❌ Nu |
| **Alarmă nativă telefon** | App Ceas Android/iOS | ✅ Da |

### ⚠️ Important despre notificări

- **Popup și email necesită internet activ** în momentul alertei
- Dacă ești offline când se declanșează alerta → **nu primești nimic în timp real**
- Când te reconectezi, Google Calendar îți arată notificările ratate — dar momentul a trecut deja
- Pentru o alarmă **garantată offline** → setează manual în app-ul Ceas de pe telefon

---

## Exemplu de utilizare

```
"Adaugă un eveniment mâine la 15:00 cu titlul Ședință supervizare,
 durată 1 oră, reminder 30 de minute înainte"
```

Claude va:
1. Verifica ora curentă (timezone Europe/Bucharest)
2. Crea evenimentul în calendarul primary
3. Seta reminder popup la 30 min înainte

---

## Sesiunea de testare — 2 iunie 2026

- Eveniment creat: **Test eveniment MCP**, 22:00–23:00
- Reminder: popup la 21:45 (15 min înainte)
- Rezultat: ✅ Eveniment apărut imediat în Google Calendar

---

## Sesiunea de reparare — 20 septembrie 2026

- **Problemă:** MCP-ul dădea eroare de autentificare. Investigare a arătat că nu era un token expirat, ci clientul OAuth din Google Cloud Console fusese **șters** (`Eroare 401: deleted_client`)
- **Cauză reală:** clientul OAuth individual dispăruse din proiectul Google Cloud "Claude Calendar" (proiectul însuși era sănătos, nemarcat pentru ștergere)
- **Fix:**
  1. Creat client OAuth nou (tip Desktop app) în același proiect
  2. Descărcat fișier JSON nou în `D:\ClaudeCode\Claude Code Calendar\`
  3. Actualizate **ambele** fișiere de configurare (Claude Desktop + Claude Code CLI — vezi secțiunea de mai sus) cu noua cale
  4. Autentificare finalizată direct din terminal: `GOOGLE_OAUTH_CREDENTIALS="<cale>" npx -y @cocal/google-calendar-mcp auth`
- **Rezultat:** ✅ funcțional, testat cu `list-calendars` (5 calendare) și creare evenimente reale (remindere Declarația Unică + ședință supervizare 25 sep)
- **Lecție:** dacă eroarea viitoare menționează explicit "deleted_client", nu insista cu restart — trebuie client OAuth nou din Google Cloud Console

---

## Limitări cunoscute

- Claude **nu poate** seta alarme native pe telefon sau PC (nu există API pentru asta)
- Notificările popup necesită **app Google Calendar instalat pe telefon** și notificări activate
- Necesită conexiune activă la internet pentru orice operațiune
