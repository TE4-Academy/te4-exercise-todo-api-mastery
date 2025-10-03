# Session 1: I Am Alive 🟢

**Tid:** 1 timme

### **Installera dependencies**

```cmd
npm install express dotenv
```

förstå att varje del kan testas isolerat först  
**Mål:** Självkänsla att "jag kan alltid börja enkelt och verifiera steg för steg"

---

## 🤔 TÄNK FÖRST (5 min)

### **Läs detta innan du kodar:**

När något är komplext och svårt att förstå, vad gör vi då?  
**Vi börjar med det enklaste möjliga som kan fungera.**

En server som bara säger "I am alive" är nog det enklaste.  
En server som ekar tillbaka vad du skickar är nästan lika enkelt.

**Frågor att fundera på:**

1. Hur vet jag att min server faktiskt körs och lyssnar?
2. Hur vet jag att min klient (browser/curl) kan nå servern?
3. Hur vet jag att data kan gå fram och tillbaka?
4. Vad är skillnaden mellan att testa med curl och med browser?

**Skriv ner dina svar** (ingen kodning ännu!) ✏️

---

## 🎯 Vad ska vi uppnå?

Efter denna session ska du ha:

✅ En server som svarar på `/ping` med `{ status: "alive" }`  
✅ En server som ekar tillbaka data på `/echo`  
✅ Testat med curl/PowerShell  
✅ Testat med Postman  
✅ Testat med en enkel HTML-sida (fetch)  
✅ Förstått express.static

---

## 🛠️ Setup

### **1. Skapa projektmapp**

```cmd
mkdir todo-api-mastery
cd todo-api-mastery
```

### **2. Initiera npm**

```cmd
npm init -y
```

### **3. Installera dependencies**

```cmd
npm install express dotenv
```

### **4. Skapa grundfiler**

```cmd
echo "" > server.js
echo "" > .env
echo "" > .gitignore
```

### **5. .gitignore**

Öppna `.gitignore` och lägg till:

```
node_modules/
.env
```

---

## 📝 CHECKPOINT 1: Minimal Server (15 min)

### **Din uppgift:**

Skapa en minimal Express-server som:

- Lyssnar på port 3000
- Har endpoint `GET /ping` som returnerar `{ status: "alive" }`
- Serverar statiska filer från `public`-mappen

### Tänk innan du kodar:

- Vad behöver importeras? (`express`)
- Hur startar man en Express-server? (`app.listen()`)
- Hur skapar man en GET-route? (`app.get()`)
- Vad ska returneras? (JSON: `res.json()`)

### **Kodexempel (om du kör fast):**

<details>
<summary>Klicka för att se exempel</summary>

```javascript
// server.js
import express from "express";

const app = express();
const PORT = 3000;

// Middleware
app.use(express.static("public")); // Servera HTML från public-mappen
app.use(express.json());

// Endpoint: Ping
app.get("/ping", (req, res) => {
  res.json({ status: "alive" });
});

// Starta server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

</details>

### **⚠️ Glöm inte:**

Lägg till i `package.json`:

```json
{
  "type": "module",
  "scripts": {
    "start": "node server.js"
  }
}
```

### **Starta servern:**

```cmd
npm start
```

Du ska se: `Server running on http://localhost:3000`

---

## 🧪 CHECKPOINT 2: Testa med curl (10 min)

### **Öppna en NY terminal** (CMD eller PowerShell)

**Testa ping-endpoint:**

```cmd
curl http://localhost:3000/ping
```

**Förväntat svar:**

```json
{ "status": "alive" }
```

### **Om det inte fungerar:**

1. ✅ Körs servern? (Se första terminalen)
2. ✅ Rätt URL? (`http://localhost:3000/ping`)
3. ✅ Stavfel i koden? (kolla `app.get('/ping', ...)`)

### **Windows-specifikt:**

Om curl inte finns i CMD, använd PowerShell:

```powershell
Invoke-RestMethod -Uri http://localhost:3000/ping
```

### **Lyckat?** ✅

**Skriv ner:** Vad hände? Vad returnerade servern? Hur kändes det?

---

## 🧪 CHECKPOINT 3: Testa med Postman (5 min)

### **Öppna Postman Desktop**

1. **New Request**
2. **Method:** GET
3. **URL:** `http://localhost:3000/ping`
4. **Send**

**Förväntad response:**

- Status: 200 OK
- Body: `{ "status": "alive" }`
- Time: < 100ms

### **Vad är skillnaden mot curl?**

- Curl = terminal, snabbt, scriptbart
- Postman = visuellt, lättare att se detaljer (headers, time, status)

**Båda gör exakt samma HTTP GET request!**

---

## 📝 CHECKPOINT 4: Echo endpoint (15 min)

### **Nu ska du:**

Lägg till ett nytt endpoint `POST /echo` som:

- Tar emot JSON i body
- Skickar tillbaka samma data

### **Tänk:**

- POST istället för GET
- `req.body` för att läsa data
- `res.json()` för att returnera data

### **Kodexempel:**

<details>
<summary>Klicka för att se exempel</summary>

```javascript
// Lägg till i server.js (efter /ping)

app.post("/echo", (req, res) => {
  const receivedData = req.body;
  res.json({
    message: "Echo!",
    youSent: receivedData,
  });
});
```

</details>

### **Testa med curl:**

```cmd
curl -X POST http://localhost:3000/echo ^
  -H "Content-Type: application/json" ^
  -d "{\"test\": \"hej\"}"
```

**PowerShell:**

```powershell
$body = @{ test = "hej" } | ConvertTo-Json
Invoke-RestMethod -Method POST -Uri http://localhost:3000/echo -Body $body -ContentType "application/json"
```

**Förväntad response:**

```json
{
  "message": "Echo!",
  "youSent": {
    "test": "hej"
  }
}
```

---

## 📝 CHECKPOINT 5: Testa från browser (10 min)

### **Skapa index.html**

```cmd
type nul > index.html
```

### **Minimal HTML:**

```html
<!DOCTYPE html>
<html lang="sv">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Test Server</title>
  </head>
  <body>
    <h1>Test Server Connection</h1>
    <button id="pingBtn">Test Ping</button>
    <button id="echoBtn">Test Echo</button>
    <pre id="result"></pre>

    <script>
      const resultEl = document.getElementById("result");

      document.getElementById("pingBtn").addEventListener("click", async () => {
        const response = await fetch("http://localhost:3000/ping");
        const data = await response.json();
        resultEl.textContent = JSON.stringify(data, null, 2);
      });

      document.getElementById("echoBtn").addEventListener("click", async () => {
        const response = await fetch("http://localhost:3000/echo", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ test: "från browser" }),
        });
        const data = await response.json();
        resultEl.textContent = JSON.stringify(data, null, 2);
      });
    </script>
  </body>
</html>
```

### Öppna i browser:

1. Gå till `http://localhost:3000/` (servern serverar index.html automatiskt)
2. Klicka **Test Ping**
3. Klicka **Test Echo**

**Notera:** URL:en är `/ping` (inte `http://localhost:3000/ping`) eftersom HTML serveras från samma server.

---

## REFLEKTERA (10 min)

### **Pausa och fundera:**

1. **Vad fungerade bra?**

   - Vilken testmetod kändes mest naturlig? (curl/Postman/browser)

2. **Vad var svårt?**

   - Stavfel? Glömde något? Förvirrande felmeddelanden?

3. **Vad lärde du dig?**

   - Hur vet du att servern lever?
   - Vad är skillnaden mellan GET och POST i praktiken?
   - Varför servera HTML från Express?

4. **Hur skulle du förklara för någon annan?**
   - "Först gör jag..." (skriv ner stegen)

### **Skriv ner dina svar** (antingen här eller i `reflection.md`)

---

## 🤖 AI-MOMENT (5 min)

### **Fråga Claude:**

Om du fastnade på något syntaktiskt, fråga nu:

**Exempel-frågor:**

- "Varför behövs `express.json()` middleware?"
- "Förklara skillnaden mellan `res.send()` och `res.json()`"
- "Vad gör `express.static()` exakt?"

**Be om MDN-länkar för fördjupning!**

---

## ✅ Session 1 Klar!

### **Checklista:**

- [ ] Server svarar på `/ping`
- [ ] Server ekar data på `/echo`
- [ ] Testat med curl/PowerShell
- [ ] Postman
- [ ] Testat från browser (http://localhost:3000/)
- [ ] Förstått express.static
- [ ] Skrivit reflektioner

### **Nästa session:**

**Session 2: One Thing at a Time (READ)**  
Nu ska vi koppla till Airtable och fokusera på GET/READ!

**Ta en paus! Du har gjort jättebra! 🎉**
