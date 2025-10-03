# Session 1 Checkpoints

## ✅ Checkpoint 1: Minimal Server (15 min)

**Mål:** Server körs och svarar på `/ping`

### **Innan du testar:**
- [ ] `npm install express dotenv` körts
- [ ] `package.json` har `"type": "module"`
- [ ] `server.js` finns och har grundkod
- [ ] `.gitignore` innehåller `node_modules/` och `.env`

### **När du startar:**
- [ ] `npm start` körs utan fel
- [ ] Konsolen visar "Server running on http://localhost:3000"
- [ ] Servern kraschar inte direkt

### **Felsökning:**
- **"Cannot find module 'express'"** → Kör `npm install`
- **"SyntaxError: Cannot use import"** → Lägg till `"type": "module"` i package.json
- **Port redan används** → Ändra PORT till 3001 eller stäng andra servrar

---

## ✅ Checkpoint 2: curl test (10 min)

**Mål:** curl kan nå servern och få svar

### **CMD (Windows):**
```cmd
curl http://localhost:3000/ping
```

### **PowerShell:**
```powershell
Invoke-RestMethod -Uri http://localhost:3000/ping
```

### **Förväntat resultat:**
```json
{"status":"alive"}
```

### **Felsökning:**
- **"Failed to connect"** → Är servern startad? Rätt port?
- **404 Not Found** → Stavfel i `/ping`? Kontrollera server.js
- **Inget svar** → Kolla firewall/antivirus

---

## ✅ Checkpoint 3: Postman test (5 min)

**Mål:** Samma request i Postman

### **Setup:**
- [ ] Postman Desktop öppen
- [ ] New Request skapad
- [ ] Method: GET
- [ ] URL: `http://localhost:3000/ping`

### **Resultat:**
- [ ] Status: 200 OK (grön)
- [ ] Body: `{ "status": "alive" }`
- [ ] Time: < 100ms (normalt)

### **Jämför:**
Är resultatet exakt samma som curl? **Ja!** Det är samma HTTP request.

---

## ✅ Checkpoint 4: Echo endpoint (15 min)

**Mål:** POST /echo fungerar

### **curl test (CMD):**
```cmd
curl -X POST http://localhost:3000/echo ^
  -H "Content-Type: application/json" ^
  -d "{\"message\": \"hej\"}"
```

### **PowerShell test:**
```powershell
$body = @{ message = "hej" } | ConvertTo-Json
Invoke-RestMethod -Method POST -Uri http://localhost:3000/echo -Body $body -ContentType "application/json"
```

### **Förväntat svar:**
```json
{
  "message": "Echo!",
  "youSent": {
    "message": "hej"
  }
}
```

### **Postman test:**
- [ ] Method: POST
- [ ] URL: `http://localhost:3000/echo`
- [ ] Headers: `Content-Type: application/json`
- [ ] Body → raw → JSON:
```json
{
  "message": "test från postman"
}
```

### **Felsökning:**
- **req.body är undefined** → Glömt `app.use(express.json())`?
- **400 Bad Request** → Fel JSON-syntax i body

---

## ✅ Checkpoint 5: Browser test (10 min)

**Mål:** Fetch från HTML fungerar

### **index.html skapad:**
- [ ] Fil finns i projektmappen
- [ ] Innehåller knappar och fetch-kod
- [ ] Öppnad i browser

### **Test:**
1. [ ] Klicka "Test Ping" → ser `{"status":"alive"}`
2. [ ] Klicka "Test Echo" → ser echat data

### **Felsökning:**

**Fetch error:**
```
Failed to fetch
```
**Lösning:**
- Är servern startad?
- Rätt URL? (http://localhost:3000)
- Kolla F12 → Network tab för detaljer

---

## 📊 Sammanfattning

### **Vad fungerade?**
- [ ] Server startar utan fel
- [ ] curl GET /ping
- [ ] curl POST /echo  
- [ ] Postman GET /ping
- [ ] Postman POST /echo
- [ ] Browser fetch GET /ping
- [ ] Browser fetch POST /echo

### **Om något inte fungerade:**
Skriv ner vad och hur du löste det (eller be om hjälp nästa session).

---

## 🎯 Redo för Session 2?

**Du är redo om:**
- ✅ Du kan starta servern utan fel
- ✅ Du kan testa endpoints med curl/Postman
- ✅ Du förstår express.static
- ✅ Du känner dig bekväm med GET vs POST

**Nästa:** Vi lägger till riktiga Airtable-anrop! 🚀
