# Todo API
## Lärmål

Efter denna övning ska du:

- Känna dig trygg i att testa API:er steg-för-steg (curl → Postman → kod)
- Förstå CRUD-mönster och kunna kopiera dem mellan operationer
- Veta hur du felsöker systematiskt när något inte fungerar
- Ha en process från tanke → test → implementation
- Använda AI rätt - för syntax, inte för att förstå flödet- 6 timmar progressiv övning

## Översikt

Denna fil är en **översikt och guide** till övningen. De faktiska stegen hittar du i varje session-mapp.

Övningen består av 6 sessioner à 1 timme där du bygger en komplett todo-app mot Airtable API.

## Syfte

Bygga självkänsla och helhetssyn på API-arbete genom att knyta ihop säcken.  
**Inte:** "Jag kan all syntax utantill"  
**Utan:** "Jag förstår flödet, kan bryta ner problem, och vet när/hur jag frågar AI"

---

## Lärmål

Efter denna övning ska du:

✅ **Känna dig trygg** i att testa API:er steg-för-steg (curl → Postman → kod)  
✅ **Förstå CRUD-mönster** och kunna kopiera dem mellan operationer  
✅ **Veta hur du felsöker** systematiskt när något inte fungerar  
✅ **Ha en process** från tanke → test → implementation  
✅ **Använda AI rätt** - för syntax, inte för att förstå flödet

---

## 📚 Övningens struktur (6 sessioner à 1 timme)

| Session | Tema                   | Fokus                                  |
| ------- | ---------------------- | -------------------------------------- |
| **1**   | I Am Alive             | Testa att grunden fungerar             |
| **2**   | One Thing at a Time    | READ - en operation perfekt            |
| **3**   | Copy The Pattern       | CREATE - kopiera mönstret              |
| **4**   | Debug Like a Detective | UPDATE/DELETE - systematisk felsökning |
| **5**   | Search & Filter        | Query parameters och filter            |
| **6**   | Connect The Dots       | Hela flödet och egen process           |

---

## Verktyg & Förutsättningar

### Du behöver ha installerat:

- Windows 11
- VS Code
- Node.js (med npm)
- Postman Desktop

### Du kan redan:

- Grundläggande ES6 JavaScript
- Node.js och Express
- Client-server koncept
- F12 debugging i browser

---

## Todo App - Datamodell

Vi bygger en minimal todo-app mot Airtable.

### Airtable Table: "Todos"

| Fält | Typ | Exempel |
| ---- | --- | ------- |

| **Task** | Single line text | "Handla mjölk" |
| **Done** | Checkbox | ☐ / ☑ |
| **Created** | Created time (auto) | 2025-01-03 14:23 |

**Det är allt!** Fokus på process, inte komplex data.

---

## Setup (20 min innan Session 1)

### 1. Skapa Airtable Base

1. Gå till https://airtable.com
2. Skapa ny Base: "Todo App"
3. Döp om tabellen till: **Todos**
4. Skapa fält:
   - `Task` (Single line text)
   - `Done` (Checkbox)
   - `Created` (Created time - auto)
5. Lägg till 2-3 test-todos manuellt

### **2. Skapa Personal Access Token (PAT)**

1. Airtable → Developer Hub → Personal Access Tokens
2. **Create token**
3. Name: "TE4 Todo Exercise"
4. Scopes:
   - `data.records:read`
   - `data.records:write`
5. Access: Välj din "Todo App" base
6. **Create token** → Spara säkert (visas bara en gång!)

### **3. Hitta Base ID**

1. Öppna din Base i Airtable
2. Klicka **Help** → **API documentation**
3. Kopiera Base ID (börjar med `app...`)

### 4. Skapa .env fil (används från Session 2)

Gör detta i Session 1 under steget SETUP (inte nu)
Skapa `.env` i projektmappen:

```
AIRTABLE_PAT=patXXXXXXXXXXXXXX
AIRTABLE_BASE=appXXXXXXXXXXXXXX
AIRTABLE_TABLE=Todos
PORT=3000
```

### **5. Installera dependencies (Session 1)**

Gör detta i Session 1 under steget SETUP (inte nu)

```cmd
npm init -y
npm install express dotenv
```

---

## Hur du använder övningen

Varje session (session-1 till session-6) följer samma struktur:

1. **TÄNK** (5 min) - Läs intro, fundera på frågor, skriv ner tankar
2. **TESTA ENKELT FÖRST** (15 min) - curl/Postman innan kod
3. **BYGG STEG-FÖR-STEG** (25 min) - Följ checkpoints, testa ofta
4. **REFLEKTERA** (10 min) - Vad lärde du dig? Vad var svårt?
5. **AI-MOMENT** (5 min) - Fråga om syntaxdetaljer du fastnat på

---

## Övningens filosofi

**Happy Path First** - Få det att fungera enkelt innan du lägger till felhantering

**Test Before Code** - Testa i curl/Postman innan du skriver fetch-kod

**One Step at a Time** - Lägg till EN sak i taget, testa, sen nästa

**Copy-Paste is OK** - Men förstå vad du kopierar, läs koden högt

**AI är Assistent** - Använd för "hur skriver jag detta?" inte "vad ska jag göra?"

---

## Filstruktur

```
te4-exercise-todo-api-mastery/
├── README.md (denna fil - översikt)
├── session-1-i-am-alive/
│   ├── README.md
│   ├── checkpoints.md
│   └── reflection.md
├── session-2-read/
│   ├── README.md
│   ├── checkpoints.md
│   ├── curl-examples.md
│   └── reflection.md
├── session-3-create/
│   ├── README.md
│   ├── checkpoints.md
│   └── reflection.md
├── session-4-update-delete/
│   ├── README.md
│   ├── checkpoints.md
│   └── reflection.md
├── session-5-search-filter/
│   ├── README.md
│   ├── checkpoints.md
│   └── reflection.md
└── session-6-connect-dots/
    ├── README.md
    ├── checkpoints.md
    └── final-reflection.md
```

---

## Kom igång

1. Slutför Setup ovan (Airtable Base, PAT)
2. Gå till `session-1-i-am-alive/`
3. Öppna `README.md` och följ stegen
4. Fortsätt sedan med session-2, session-3, osv.

---

## Troubleshooting

### Airtable 403 error

- Kontrollera PAT scopes: `data.records:read` + `data.records:write`
- Verifiera att rätt Base valdes när PAT skapades
- Testa PAT i CMD/PowerShell först (se Session 2)

### "Module not found"

- Kör `npm install` i projektmappen
- Kontrollera att `package.json` finns

### Environment variables fungerar inte

- Kontrollera att `.env` finns i projektets rot
- Ladda dotenv i server.js: `import 'dotenv/config';`
- Starta om servern efter .env-ändringar

---

**Lycka till!**
