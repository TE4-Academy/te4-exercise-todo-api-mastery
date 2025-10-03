# Session 3: Copy The Pattern (CREATE) ✏️

**Tid:** 1 timme  
**Tema:** När du förstår GET, är POST bara "samma fast med body"  
**Mål:** Inser att CRUD är repetitiva mönster, inte unika problem

---

## 🤔 TÄNK FÖRST (5 min)

### **Jämför GET och POST:**

| Aspekt      | GET           | POST                         |
| ----------- | ------------- | ---------------------------- |
| **URL**     | Samma         | Samma                        |
| **Headers** | Authorization | Authorization + Content-Type |
| **Body**    | Ingen         | JSON med data                |
| **Syfte**   | Hämta         | Skapa                        |

**Insikt:** POST är nästan identiskt med GET + body!

**Frågor att fundera på:**

1. Vad ska skickas i body för att skapa en todo?
2. Hur vet jag att POST lyckas? (status code?)
3. Kan jag testa POST i curl/Postman först?
4. Hur skiljer sig fetch för POST vs GET?

**Skriv ner dina tankar** ✏️

---

## 🎯 Vad ska vi uppnå?

Efter denna session ska du ha:

✅ Testat POST till Airtable med curl  
✅ Testat samma i Postman  
✅ Skapat Express endpoint `POST /api/todos`  
✅ Formulär i browser som skapar todos  
✅ Sett ny todo dyka upp i Airtable  
✅ Förstått request body struktur

---

## 🧪 CHECKPOINT 1: POST med curl (15 min)

### **Airtable POST struktur:**

```
POST https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}
Headers:
  Authorization: Bearer {PAT}
  Content-Type: application/json
Body:
{
  "fields": {
    "Task": "Din todo text",
    "Done": false
  }
}
```

**Notera:**

- Method: POST (inte GET)
- Header: `Content-Type: application/json`
- Body: `fields` objekt med Task och Done

### **CMD:**

```cmd
curl -X POST https://api.airtable.com/v0/appXXXXX/Todos ^
  -H "Authorization: Bearer patXXXXX" ^
  -H "Content-Type: application/json" ^
  -d "{\"fields\":{\"Task\":\"Test från curl\",\"Done\":false}}"
```

### **PowerShell:**

```powershell
$headers = @{
    "Authorization" = "Bearer patXXXXX"
    "Content-Type" = "application/json"
}

$body = @{
    fields = @{
        Task = "Test från PowerShell"
        Done = $false
    }
} | ConvertTo-Json

Invoke-RestMethod -Method POST -Uri "https://api.airtable.com/v0/appXXXXX/Todos" -Headers $headers -Body $body
```

### **Förväntat svar:**

```json
{
  "id": "recNEWXXX",
  "createdTime": "2025-01-03T15:30:00.000Z",
  "fields": {
    "Task": "Test från curl",
    "Done": false,
    "Created": "2025-01-03T15:30:00.000Z"
  }
}
```

**✅ Lyckat?** Öppna Airtable i browser - ser du den nya raden?

---

## 🧪 CHECKPOINT 2: POST i Postman (10 min)

### **Setup:**

1. **New Request** (eller kopiera GET request)
2. **Method:** POST (ändra från GET)
3. **URL:** `https://api.airtable.com/v0/{BASE_ID}/Todos`
4. **Headers:**
   - `Authorization: Bearer {PAT}`
   - `Content-Type: application/json`
5. **Body → raw → JSON:**

```json
{
  "fields": {
    "Task": "Test från Postman",
    "Done": false
  }
}
```

6. **Send**

### **Resultat:**

- Status: **200 OK** (Airtable returnerar 200, inte 201)
- Body: Ny record med `id`
- Kontrollera Airtable browser tab!

### **Spara i Collection:**

- Save → "Airtable Todos" collection
- Namn: "Create Todo"

---

## 📝 CHECKPOINT 3: Express POST endpoint (20 min)

### **Din uppgift:**

Skapa `POST /api/todos` som:

1. Tar emot `{ task: "text" }` från browser
2. Formaterar om till Airtable-struktur
3. POST:ar till Airtable
4. Returnerar skapade todo

### **Tänk:**

- Samma URL som GET (bara annan method)
- `req.body` för att läsa data från browser
- `fetch` till Airtable med `method: 'POST'` och `body`

### **Kodexempel:**

<details>
<summary>Klicka för att se exempel</summary>

```javascript
app.post("/api/todos", async (req, res) => {
  try {
    const { task } = req.body;

    if (!task) {
      return res.status(400).json({ error: "Task is required" });
    }

    const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}`;

    const response = await fetch(url, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${AIRTABLE_PAT}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        fields: {
          Task: task,
          Done: false,
        },
      }),
    });

    if (!response.ok) {
      throw new Error(`Airtable error: ${response.status}`);
    }

    const data = await response.json();
    res.json(data);
  } catch (error) {
    console.error("Error creating todo:", error);
    res.status(500).json({ error: "Failed to create todo" });
  }
});
```

</details>

### **Testa med curl:**

```cmd
curl -X POST http://localhost:3000/api/todos ^
  -H "Content-Type: application/json" ^
  -d "{\"task\":\"Test från egen server\"}"
```

**PowerShell:**

```powershell
$body = @{ task = "Test från egen server" } | ConvertTo-Json
Invoke-RestMethod -Method POST -Uri http://localhost:3000/api/todos -Body $body -ContentType "application/json"
```

---

## 📝 CHECKPOINT 4: Formulär i browser (15 min)

### **Uppdatera index.html:**

```html
<h2>Create Todo</h2>
<form id="createForm">
  <input type="text" id="taskInput" placeholder="New task..." required />
  <button type="submit">Add Todo</button>
</form>

<h2>Todos</h2>
<button id="loadTodos">Refresh</button>
<ul id="todoList"></ul>

<script>
  // Create todo
  document
    .getElementById("createForm")
    .addEventListener("submit", async (e) => {
      e.preventDefault();

      const task = document.getElementById("taskInput").value;

      const response = await fetch("http://localhost:3000/api/todos", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ task }),
      });

      const newTodo = await response.json();
      console.log("Created:", newTodo);

      // Clear input
      document.getElementById("taskInput").value = "";

      // Reload list
      document.getElementById("loadTodos").click();
    });

  // Load todos (från Session 2)
  document.getElementById("loadTodos").addEventListener("click", async () => {
    const response = await fetch("http://localhost:3000/api/todos");
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

  // Load on page load
  document.getElementById("loadTodos").click();
</script>
```

### **Test:**

1. Öppna index.html
2. Skriv "Handla mjölk" i formulär
3. Klicka "Add Todo"
4. Se listan uppdateras
5. Kontrollera Airtable browser tab!

---

## 💭 REFLEKTERA (10 min)

### **Pausa och fundera:**

1. **Vad är skillnaden mellan GET och POST?**

   - Method + Headers + Body

2. **Varför "fields" i Airtable body?**

   - Airtable API-struktur, alltid `{ fields: {...} }`

3. **Varför 200 OK istället för 201 Created?**

   - Airtable API-design, inte standard REST

4. **Kopiera-klistra mönstret:**

   - GET kod → POST kod: Vad ändrades? (method, headers, body)

5. **Vad skulle UPDATE och DELETE behöva?**
   - UPDATE: POST + record ID
   - DELETE: GET + record ID + DELETE method

### **Skriv ner i reflection.md!**

## Skapa en kopia av reflection.md från Session 2

## 🤖 AI-MOMENT (5 min)

### **Fråga Claude:**

**Exempel-frågor:**

- "Förklara HTTP methods: GET vs POST vs PATCH vs DELETE"
- "Varför Content-Type: application/json?"
- "MDN artikel om fetch API POST request"

---

## ✅ Session 3 Klar!

### **Checklista:**

- [ ] curl POST till Airtable fungerar
- [ ] Postman POST till Airtable fungerar
- [ ] Express `POST /api/todos` skapar todo
- [ ] Formulär i browser fungerar
- [ ] Ny todo syns i Airtable
- [ ] Förstått request body struktur

### **Nästa session:**

**Session 4: Debug Like a Detective (UPDATE/DELETE)**  
Nu tar vi itu med PATCH och DELETE - med systematisk felsökning!

**Ta en paus! Du kan CREATE nu! ✅**
