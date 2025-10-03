# Session 4: Debug Like a Detective 🔍

**Tid:** 1 timme  
**Tema:** Använd verktygen för att förstå VAD som är fel  
**Mål:** Systematisk felsökningsprocess istället för panik

---

## 🤔 TÄNK FÖRST (5 min)

### **När något inte fungerar - vad gör du?**

❌ **Dålig strategi:**

- Panik och copy-paste från Stack Overflow
- Ändra massa kod på måfå
- "Det fungerar inte!"

✅ **Bra strategi:**

- Läs felmeddelandet (vad säger det?)
- Kontrollera request (F12 Network, Postman Console, curl -v)
- Verifiera data (är ID korrekt? Rätt format?)
- Testa steg-för-steg (curl först, sedan kod)

**Frågor att fundera på:**

1. Vad är skillnaden mellan UPDATE (PATCH) och CREATE (POST)?
2. Hur vet jag VILKET record jag ska uppdatera? (ID!)
3. Vad är skillnaden mellan PATCH och PUT?
4. Varför är DELETE "lätt" när man förstår mönstret?

**Skriv ner dina tankar** ✏️

---

## 🎯 Vad ska vi uppnå?

Efter denna session ska du ha:

✅ UPDATE (PATCH) todo (markera done/undone)  
✅ DELETE todo  
✅ Systematisk felsökningsprocess  
✅ Förstått hur record ID fungerar  
✅ Checkbox-toggle i browser  
✅ Delete-knapp i browser

---

## 🧪 CHECKPOINT 1: UPDATE med curl (15 min)

### **PATCH vs PUT:**

- **PATCH** = Uppdatera DELAR av record (rekommenderat)
- **PUT** = Ersätt HELA record

Airtable stödjer båda, vi använder PATCH.

### **Airtable PATCH struktur:**

```
PATCH https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}
Headers:
  Authorization: Bearer {PAT}
  Content-Type: application/json
Body:
{
  "fields": {
    "Done": true
  }
}
```

**Notera:**

- URL innehåller **record ID** (recXXXXXXXXXXXXXX)
- Body innehåller BARA fält som ska ändras

### **Hitta ett record ID:**

1. **Från GET response (Session 2):**

```cmd
curl http://localhost:3000/api/todos
```

Kopiera ett `"id": "recXXXXXXXXXXXXXX"`

### **CMD:**

```cmd
curl -X PATCH https://api.airtable.com/v0/appXXXXX/Todos/recXXXXX ^
  -H "Authorization: Bearer patXXXXX" ^
  -H "Content-Type: application/json" ^
  -d "{\"fields\":{\"Done\":true}}"
```

### **PowerShell:**

```powershell
$headers = @{
    "Authorization" = "Bearer patXXXXX"
    "Content-Type" = "application/json"
}

$body = @{
    fields = @{
        Done = $true
    }
} | ConvertTo-Json

$recId = "recXXXXXXXXXXXXXX"  # Ersätt med riktigt ID
Invoke-RestMethod -Method PATCH -Uri "https://api.airtable.com/v0/appXXXXX/Todos/$recId" -Headers $headers -Body $body
```

### **Förväntat svar:**

```json
{
  "id": "recXXXXXXXXXXXXXX",
  "createdTime": "...",
  "fields": {
    "Task": "...",
    "Done": true, // Ändrat!
    "Created": "..."
  }
}
```

**✅ Kontrollera Airtable:** Är checkboxen ikryssad?

---

## 🧪 CHECKPOINT 2: DELETE med curl (10 min)

### **Airtable DELETE struktur:**

```
DELETE https://api.airtable.com/v0/{BASE_ID}/{TABLE_NAME}/{RECORD_ID}
Headers:
  Authorization: Bearer {PAT}
```

**Enklaste operationen!** Ingen body, bara ID.

### **CMD:**

```cmd
curl -X DELETE https://api.airtable.com/v0/appXXXXX/Todos/recXXXXX ^
  -H "Authorization: Bearer patXXXXX"
```

### **PowerShell:**

```powershell
$headers = @{ "Authorization" = "Bearer patXXXXX" }
$recId = "recXXXXXXXXXXXXXX"

Invoke-RestMethod -Method DELETE -Uri "https://api.airtable.com/v0/appXXXXX/Todos/$recId" -Headers $headers
```

### **Förväntat svar:**

```json
{
  "deleted": true,
  "id": "recXXXXXXXXXXXXXX"
}
```

**✅ Kontrollera Airtable:** Är raden borta?

---

## 🧪 CHECKPOINT 3: Postman PATCH & DELETE (10 min)

### **PATCH i Postman:**

1. **Kopiera GET request** → Duplicera
2. **Ändra method:** PATCH
3. **URL:** `https://api.airtable.com/v0/{BASE}/{TABLE}/{RECORD_ID}`
4. **Body → raw → JSON:**

```json
{
  "fields": {
    "Done": false
  }
}
```

5. **Send**

**Testa toggle:** Kör samma request igen med `"Done": true`

### **DELETE i Postman:**

1. **New Request**
2. **Method:** DELETE
3. **URL:** `https://api.airtable.com/v0/{BASE}/{TABLE}/{RECORD_ID}`
4. **Headers:** `Authorization: Bearer {PAT}`
5. **Ingen body!**
6. **Send**

**Tips:** Skapa en test-todo först (POST), sedan DELETE den.

---

## 📝 CHECKPOINT 4: Express PATCH endpoint (15 min)

### **Din uppgift:**

Skapa `PATCH /api/todos/:id` som:

1. Tar emot record ID från URL parameter
2. Tar emot `{ done: boolean }` från body
3. PATCH:ar till Airtable
4. Returnerar uppdaterad todo

### **Kodexempel:**

<details>
<summary>Klicka för att se exempel</summary>

```javascript
app.patch("/api/todos/:id", async (req, res) => {
  try {
    const { id } = req.params; // URL parameter
    const { done } = req.body;

    if (done === undefined) {
      return res.status(400).json({ error: "Done field is required" });
    }

    const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}/${id}`;

    const response = await fetch(url, {
      method: "PATCH",
      headers: {
        Authorization: `Bearer ${AIRTABLE_PAT}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        fields: {
          Done: done,
        },
      }),
    });

    if (!response.ok) {
      throw new Error(`Airtable error: ${response.status}`);
    }

    const data = await response.json();
    res.json(data);
  } catch (error) {
    console.error("Error updating todo:", error);
    res.status(500).json({ error: "Failed to update todo" });
  }
});
```

</details>

### **Testa med curl:**

```cmd
curl -X PATCH http://localhost:3000/api/todos/recXXXXX ^
  -H "Content-Type: application/json" ^
  -d "{\"done\":true}"
```

---

## 📝 CHECKPOINT 5: Express DELETE endpoint (10 min)

### **Kodexempel:**

<details>
<summary>Klicka för att se exempel</summary>

```javascript
app.delete("/api/todos/:id", async (req, res) => {
  try {
    const { id } = req.params;

    const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}/${id}`;

    const response = await fetch(url, {
      method: "DELETE",
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
    console.error("Error deleting todo:", error);
    res.status(500).json({ error: "Failed to delete todo" });
  }
});
```

</details>

### **Testa med curl:**

```cmd
curl -X DELETE http://localhost:3000/api/todos/recXXXXX
```

---

## 📝 CHECKPOINT 6: Browser UI (15 min)

### **Uppdatera todo-listan:**

```html
<script>
  document.getElementById("loadTodos").addEventListener("click", async () => {
    const response = await fetch("http://localhost:3000/api/todos");
    const data = await response.json();

    const list = document.getElementById("todoList");
    list.innerHTML = "";

    data.records.forEach((record) => {
      const li = document.createElement("li");
      const id = record.id;
      const task = record.fields.Task;
      const done = record.fields.Done;

      // Checkbox
      const checkbox = document.createElement("input");
      checkbox.type = "checkbox";
      checkbox.checked = done;
      checkbox.addEventListener("change", async () => {
        await fetch(`http://localhost:3000/api/todos/${id}`, {
          method: "PATCH",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ done: checkbox.checked }),
        });
      });

      // Task text
      const span = document.createElement("span");
      span.textContent = task;
      span.style.textDecoration = done ? "line-through" : "none";

      // Delete button
      const deleteBtn = document.createElement("button");
      deleteBtn.textContent = "🗑️";
      deleteBtn.addEventListener("click", async () => {
        await fetch(`http://localhost:3000/api/todos/${id}`, {
          method: "DELETE",
        });
        li.remove();
      });

      li.appendChild(checkbox);
      li.appendChild(span);
      li.appendChild(deleteBtn);
      list.appendChild(li);
    });
  });
</script>
```

### **Test:**

1. Klicka checkbox → se strike-through
2. Kontrollera Airtable → checkad?
3. Klicka 🗑️ → todo försvinner
4. Refresh Airtable → borta?

---

## 💭 REFLEKTERA (10 min)

### **Felsökningsstrategi:**

1. **Felmeddelande säger vad?**
2. **Kolla F12 Network tab** - Vilken status code? Request correct?
3. **Testa i Postman först** - Fungerar det där?
4. **curl verbose (-v)** - Se exakt vad som skickas
5. **console.log()** - Logga ID, body, response

### **CRUD-mönstret:**

- CREATE (POST) - ingen ID, bara body
- READ (GET) - ingen body
- UPDATE (PATCH) - ID + body
- DELETE (DELETE) - bara ID

**Alla är variationer på samma tema!**

### **Skriv ner i reflection.md!**

## Skapa en kopia av reflection.md från Session 3

## ✅ Session 4 Klar!

### **Checklista:**

- [ ] PATCH med curl/Postman
- [ ] DELETE med curl/Postman
- [ ] Express PATCH & DELETE endpoints
- [ ] Checkbox toggle i browser
- [ ] Delete button i browser
- [ ] Förstått systematisk felsökning

### **Nästa session:**

**Session 5: Search & Filter**  
Query parameters och filterByFormula!

**Ta en paus! Du kan hela CRUD nu! 🎉**
