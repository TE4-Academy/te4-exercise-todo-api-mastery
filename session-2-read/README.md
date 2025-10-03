# Session 2: One Thing at a Time (READ)

**Tid:** 1 timme  
**Tema:** Bygg EN operation perfekt först, kopiera sen mönstret  
**Mål:** Förstå att verktyg är bara olika sätt att göra HTTP-anrop

---

## TÄNK FÖRST (5 min)

### Läs detta innan du kodar:

Vad är skillnaden mellan att anropa ett API med curl, Postman och fetch?

**Spoiler: INGET!** Det är samma HTTP GET request, bara olika verktyg.

Denna session fokuserar på EN sak: READ/GET från Airtable

När du behärskar detta, är resten bara variationer på samma tema.

**Frågor att fundera på:**

1. Vad behövs för att anropa Airtable API? (URL, headers, method)
2. Hur vet jag att anropet lyckas? (status code, response data)
3. Varför ska jag testa i curl/Postman INNAN jag kodar fetch?
4. Vad är "happy path"? Varför börja där?

**Skriv ner dina tankar**

---

## Vad ska vi uppnå?

Efter denna session ska du ha:

- Testat Airtable GET med curl/PowerShell
- Testat samma request i Postman
- Skapat Express endpoint som proxy till Airtable
- Anropat din egen server från browser
- Förstått request → server → Airtable → server → response flödet
- Listat todos i browser

---

## Förberedelse: Airtable credentials

### Från huvudövningens README:

Du ska ha:

- Airtable Base med "Todos" tabell
- Personal Access Token (PAT)
- Base ID (börjar med `app...`)
- `.env` fil med credentials

### Kontrollera .env:

```
AIRTABLE_PAT=patXXXXXXXXXXXXXX
AIRTABLE_BASE=appXXXXXXXXXXXXXX
AIRTABLE_TABLE=Todos
PORT=3000
```

---

## CHECKPOINT 1: Testa Airtable med curl (15 min)

### Innan du testar:

Förstå anatomin av ett Airtable API-anrop:

```
GET https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}
Headers:
  Authorization: Bearer {YOUR_PAT}
```

**Tre delar:**

1. **URL** - Base ID + Table namn
2. **Method** - GET för READ
3. **Header** - Authorization med PAT

### CMD:

```cmd
curl -H "Authorization: Bearer patXXXXXXXXXXXXXX" https://api.airtable.com/v0/appXXXXXXXXXXXXXX/Todos
```

**Ersätt:**

- `patXXXXX` med din PAT
- `appXXXXX` med ditt Base ID

### PowerShell:

```powershell
$headers = @{ "Authorization" = "Bearer patXXXXXXXXXXXXXX" }
Invoke-RestMethod -Uri "https://api.airtable.com/v0/appXXXXXXXXXXXXXX/Todos" -Headers $headers
```

### Förväntat svar:

```json
{
  "records": [
    {
      "id": "recXXX",
      "createdTime": "2025-01-03T10:00:00.000Z",
      "fields": {
        "Task": "Handla mjölk",
        "Done": false,
        "Created": "2025-01-03T10:00:00.000Z"
      }
    }
  ]
}
```

### Om du får fel:

**403 Forbidden:**

- PAT har fel scopes (behöver `data.records:read`)
- PAT har inte access till denna Base
- Felaktig PAT (kopiera igen från Airtable)

**404 Not Found:**

- Fel Base ID
- Fel Table namn (case-sensitive! "Todos" inte "todos")

**401 Unauthorized:**

- PAT saknas eller är fel

### **✅ Lyckat?**

**Skriv ner:** Vad såg du i responsen? Hur många todos har du? Vilka fält?

---

## CHECKPOINT 2: Testa i Postman (10 min)

### Nu samma request i Postman:

1. **New Request**
2. **Method:** GET
3. **URL:** `https://api.airtable.com/v0/{BASE_ID}/Todos`
4. **Headers tab:**
   - Key: `Authorization`
   - Value: `Bearer {YOUR_PAT}`
5. **Send**

### **Jämför med curl:**

- Samma status code? (200 OK)
- Samma data i response?
- Samma headers?

**Poäng:** Det är EXAKT samma HTTP request! Postman är bara ett GUI för curl.

### **Tips: Spara i Collection**

- **Save** → "Airtable Todos" collection
- Användbara för snabba tester senare

---

## CHECKPOINT 3: Express proxy endpoint (20 min)

### Varför en proxy?

Browser bör inte anropa Airtable direkt (säkerhet - PAT exponeras).  
Lösning: Express-server som proxy.

```
Browser → fetch → din server → Airtable → din server → Browser
```

### **Din uppgift:**

1. Installera `node-fetch` (för att anropa Airtable från Node)
2. Lägg till route `GET /api/todos`
3. Denna route anropar Airtable och returnerar data

### **Installera node-fetch:**

```cmd
npm install node-fetch
```

### **Ladda .env credentials:**

I `server.js` (längst upp):

```javascript
import "dotenv/config";

const AIRTABLE_PAT = process.env.AIRTABLE_PAT;
const AIRTABLE_BASE = process.env.AIRTABLE_BASE;
const AIRTABLE_TABLE = process.env.AIRTABLE_TABLE;
```

### **Skapa GET /api/todos:**

<details>
<summary>Klicka för kodexempel</summary>

```javascript
import fetch from "node-fetch";

// ... tidigare kod ...

app.get("/api/todos", async (req, res) => {
  try {
    const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}`;

    const response = await fetch(url, {
      headers: {
        Authorization: `Bearer ${AIRTABLE_PAT}`,
      },
    });

    if (!response.ok) {
      throw new Error(`Airtable error: ${response.status}`);
    }

    const data = await response.json();
    res.json(data);
  } catch (error) {
    console.error("Error fetching todos:", error);
    res.status(500).json({ error: "Failed to fetch todos" });
  }
});
```

</details>

### **Testa med curl:**

```cmd
curl http://localhost:3000/api/todos
```

**Förväntat:** Samma data som direkt från Airtable!

### **Testa i Postman:**

- Method: GET
- URL: `http://localhost:3000/api/todos`
- Ingen Authorization header behövs! (servern hanterar det)

---

## CHECKPOINT 4: Visa i browser (15 min)

### Uppdatera public/index.html:

Lägg till knapp och lista:

```html
<h2>Todos</h2>
<button id="loadTodos">Load Todos</button>
<ul id="todoList"></ul>

<script>
  document.getElementById("loadTodos").addEventListener("click", async () => {
    const response = await fetch("/api/todos");
    const data = await response.json();

    const list = document.getElementById("todoList");
    list.innerHTML = "";

    data.records.forEach((record) => {
      const li = document.createElement("li");
      const task = record.fields.Task;
      const done = record.fields.Done ? "✓" : "☐";
      li.textContent = `${done} ${task}`;
      list.appendChild(li);
    });
  });
</script>
```

### **Test:**

1. Öppna index.html i browser
2. Klicka "Load Todos"
3. Se listan populeras!

### **F12 Network tab:**

- Öppna F12 → Network
- Klicka "Load Todos" igen
- Se requesten till `/api/todos`
- Inspektera response

**Vad ser du?** Status, headers, timing, preview?

---

## REFLEKTERA (10 min)

### Pausa och fundera:

1. **Vad är skillnaden mellan curl, Postman och fetch?**

   - Samma HTTP request, olika verktyg!

2. **Varför inte anropa Airtable direkt från browser?**

   - Säkerhet: PAT skulle exponeras i frontend-kod

3. **Vad gör Express-servern?**

   - Proxy: tar emot browser request → anropar Airtable → returnerar data

4. **Vad är "records" i Airtable response?**

   - Array av todos. Varje record har `id`, `fields`, `createdTime`

5. **När ska jag testa i curl/Postman innan jag kodar?**
   - ALLTID! Verifiera endpoint fungerar först.

### **Skriv ner i reflection.md!**

## Skapa en kopia av reflection.md från Session 1

## AI-MOMENT (5 min)

### Fråga Claude:

**Exempel-frågor:**

- "Förklara Airtable API response-struktur (records, fields)"
- "Varför inte exponera API-nycklar i frontend?"
- "Vad är skillnaden mellan fetch i Node och browser?"
- "MDN artikel om Authorization header"

---

## Session 2 Klar!

### Checklista:

- [ ] curl/PowerShell GET till Airtable fungerar
- [ ] Postman GET till Airtable fungerar
- [ ] Express `/api/todos` returnerar data
- [ ] Browser fetch listar todos (http://localhost:3000/)
- [ ] Förstått request-flödet: browser → server → Airtable
- [ ] Inspekterat Network tab i F12

### Nästa session:

**Session 3: Copy The Pattern (CREATE)**  
Nu ska vi lägga till todos - POST är bara GET fast med body!

**Ta en paus!**
