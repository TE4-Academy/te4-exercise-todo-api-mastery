# Session 6: Connect The Dots 🔗

**Tid:** 1 timme  
**Tema:** Från tanke → implementation → test → deploy  
**Mål:** Egen metodik och självförtroende

---

## 🤔 TÄNK FÖRST (10 min)

### **Du har nu lärt dig:**

✅ Session 1: Server fungerar (ping/echo)  
✅ Session 2: READ - hämta data  
✅ Session 3: CREATE - skapa data  
✅ Session 4: UPDATE/DELETE - ändra/ta bort data  
✅ Session 5: SEARCH/FILTER - hitta specifik data

**Detta är HELA CRUD + search!**

### **Denna session handlar om:**

1. **Din process** - Hur tänker DU när du ska bygga något nytt?
2. **Planera före kod** - Vad behövs? Vilket endpoint? Vilken data?
3. **Testa först** - Postman/curl INNAN fetch-kod
4. **Använd AI rätt** - För syntax, inte för flödet

**Frågor att fundera på:**

1. Vad gör jag FÖRST när jag får en ny uppgift?
2. Hur vet jag vilken HTTP method jag ska använda?
3. När ska jag fråga AI? När ska jag inte?
4. Vad är min "happy path" process?

**Skriv ner dina tankar - detta är viktigast i hela övningen!** ✏️

---

## 🎯 Vad ska vi uppnå?

Efter denna session ska du ha:

✅ Din egen process för att bygga nya features  
✅ Implementerat en ny feature från scratch  
✅ Testat systematiskt  
✅ Reflektion över hela övningen  
✅ Handlingsplan för framtiden

---

## 📝 ÖVNING 1: Din process (20 min)

### **Scenario:**

Du ska bygga en ny feature: **"Prioritera todos (låg/medium/hög)"**

### **Steg 1: TÄNK - Bryt ner problemet**

**Innan du kodar, svara på:**

1. **Vad behövs i Airtable?**

   - Nytt fält: Priority (Single select: Low, Medium, High)

2. **Vilka endpoints påverkas?**

   - GET /api/todos - visa priority
   - POST /api/todos - sätt priority när skapad
   - PATCH /api/todos/:id - ändra priority

3. **Vad behövs i UI?**

   - Dropdown när skapar todo
   - Visa priority-färg/ikon i listan
   - Knapp för att ändra priority

4. **Hur testar jag?**
   - Lägg till fält i Airtable först
   - Testa POST med priority i Postman
   - Testa GET - syns priority?
   - Sedan implementera i kod

**Skriv ner din plan innan du börjar koda!**

---

### **Steg 2: TESTA - Postman först**

#### **2a. Lägg till Priority-fält i Airtable**

1. Öppna din Todos-tabell
2. **Add field**
3. Type: **Single select**
4. Name: **Priority**
5. Options: `Low` (grön), `Medium` (gul), `High` (röd)
6. **Save**

#### **2b. Testa POST med priority (Postman)**

1. Öppna "Create Todo" request
2. Uppdatera body:

```json
{
  "fields": {
    "Task": "Test priority",
    "Done": false,
    "Priority": "High"
  }
}
```

3. **Send**
4. Kontrollera response - finns Priority?
5. Kontrollera Airtable - visas priority?

#### **2c. Testa GET (Postman)**

1. Öppna "Get Todos" request
2. **Send**
3. Se response - finns Priority i fields?

**✅ Fungerar i Postman? Då fungerar det i kod också!**

---

### **Steg 3: IMPLEMENTERA - Kod**

#### **3a. Uppdatera Express POST endpoint**

<details>
<summary>Kodexempel</summary>

```javascript
app.post("/api/todos", async (req, res) => {
  try {
    const { task, priority } = req.body; // Lägg till priority

    if (!task) {
      return res.status(400).json({ error: "Task is required" });
    }

    const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}`;

    const fields = {
      Task: task,
      Done: false,
    };

    // Lägg till priority om angiven
    if (priority) {
      fields.Priority = priority;
    }

    const response = await fetch(url, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${AIRTABLE_PAT}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ fields }),
    });

    const data = await response.json();
    res.json(data);
  } catch (error) {
    console.error("Error creating todo:", error);
    res.status(500).json({ error: "Failed to create todo" });
  }
});
```

</details>

#### **3b. Uppdatera browser UI**

<details>
<summary>Kodexempel</summary>

```html
<!-- Formulär med priority -->
<form id="createForm">
  <input type="text" id="taskInput" placeholder="New task..." required />
  <select id="prioritySelect">
    <option value="">No priority</option>
    <option value="Low">Low</option>
    <option value="Medium">Medium</option>
    <option value="High">High</option>
  </select>
  <button type="submit">Add Todo</button>
</form>

<script>
  document
    .getElementById("createForm")
    .addEventListener("submit", async (e) => {
      e.preventDefault();

      const task = document.getElementById("taskInput").value;
      const priority = document.getElementById("prioritySelect").value;

      const body = { task };
      if (priority) body.priority = priority;

      await fetch("http://localhost:3000/api/todos", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      });

      document.getElementById("taskInput").value = "";
      document.getElementById("prioritySelect").value = "";
      loadTodos();
    });

  // Visa priority i lista
  function loadTodos() {
    // ... tidigare kod ...

    data.records.forEach((record) => {
      const priority = record.fields.Priority || "None";

      // Lägg till priority-badge
      const badge = document.createElement("span");
      badge.textContent = priority;
      badge.style.padding = "2px 8px";
      badge.style.borderRadius = "4px";
      badge.style.fontSize = "12px";
      badge.style.marginLeft = "8px";

      if (priority === "High") badge.style.background = "#ffcccb";
      else if (priority === "Medium") badge.style.background = "#ffffcc";
      else if (priority === "Low") badge.style.background = "#ccffcc";

      li.appendChild(badge);
    });
  }
</script>
```

</details>

#### **3c. Testa din implementation**

1. **Skapa todo med priority**
2. **Refresh listan** - syns priority?
3. **Kontrollera Airtable** - sparad korrekt?
4. **F12 Network tab** - inspect request/response

---

## 💭 ÖVNING 2: Helhetsutvärdering (20 min)

### **Jämför din notes app med denna övning:**

| Aspekt            | Före övning               | Efter övning                    |
| ----------------- | ------------------------- | ------------------------------- |
| **Process**       | "Börja koda direkt"       | "Planera → Testa → Koda"        |
| **Felsökning**    | "Panik, copy-paste"       | "Systematisk: curl/Postman/F12" |
| **CRUD**          | "Varje operation är unik" | "Alla är variationer"           |
| **AI-användning** | "Fråga om allt"           | "Syntax ja, flöde nej"          |
| **Självkänsla**   | "Jag kan inte"            | "Jag vet hur jag tänker"        |

### **Skriv ner:**

1. **Vad har förändrats i hur du tänker?**

2. **Vad tar du med dig till notes app?**

3. **Vilken del var svårast? Varför?**

4. **Vilken del gav mest aha-upplevelse?**

---

## 📋 ÖVNING 3: Din egen feature (20 min)

### **Välj EN feature att implementera:**

**Alternativ 1: Due Date (förfallodatum)**

- Lägg till Date-fält i Airtable
- Visa datum i lista
- Highlight överdue todos (röd bakgrund)

**Alternativ 2: Categories/Tags**

- Lägg till Category-fält (Single line text)
- Filter på category
- Visa category-badge

**Alternativ 3: Edit Task text**

- Implementera PATCH för Task-fältet
- Edit-knapp i listan
- Inline edit eller modal

### **Följ din process:**

1. ✏️ **Planera** (5 min)

   - Vad behövs i Airtable?
   - Vilka endpoints?
   - Hur testa?

2. 🧪 **Testa i Postman** (5 min)

   - Fungerar POST/GET/PATCH?

3. 🔨 **Implementera** (10 min)
   - Express endpoint
   - Browser UI

**Använd AI för syntax, men DU bestämmer flödet!**

---

## 💭 FINAL REFLEKTION (15 min)

### **Hela övningen - 6 timmar**

Fyll i `final-reflection.md` med:

1. **Vad har du lärt dig?** (Inte bara tekniskt - också om dig själv)

2. **Hur har din process förändrats?**

3. **Vad känner du dig trygg i nu?**

4. **Vad behöver du öva mer på?**

5. **Hur kommer du använda AI framåt?**

6. **Vad är din action plan för notes app?**

7. **Meddelande till dig själv om 1 månad:**

---

## 🤖 AI-MOMENT (5 min)

### **Ställ strategiska frågor:**

**INTE:** "Hur fixar jag min notes app?"  
**UTAN:** "Beskriv processen för att debugga en PATCH-request som failar"

**INTE:** "Skriv kod för delete-funktion"  
**UTAN:** "Förklara skillnaden mellan DELETE i REST API och i DOM manipulation"

**Be om:**

- Koncept-förklaringar
- MDN-länkar
- Debugging-strategier

**INTE:**

- Färdig kod
- "Fixa åt mig"

---

## ✅ Session 6 & Övningen KLAR! 🎉

### **Du har nu:**

✅ Full CRUD mot Airtable  
✅ Search & filter  
✅ Systematisk process  
✅ Debugging-strategi  
✅ AI-användning som stöd, inte krycka  
✅ Självkänsla i flödet

---

## 🚀 Nästa steg

### **För notes app:**

1. **Lägg fram en plan** (innan kod!)
2. **Testa i Postman** (verifiera endpoints)
3. **Implementera steg-för-steg** (en feature i taget)
4. **Fråga AI om syntax** (inte om flöde)

### **För framtida projekt:**

- Använd samma process
- Happy path först, alltid
- Testa verktyg → kod, inte tvärtom
- En sak i taget

---

## 📝 Sista uppgiften

**Skriv klart `final-reflection.md` innan du lämnar!**

Detta är den viktigaste filen - din egen kunskap om hur DU lär dig bäst.

**Grattis! Du har klarat hela övningen! 🎓**
