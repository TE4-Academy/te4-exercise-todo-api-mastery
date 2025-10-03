# Session 5: Search & Filter 🔎

**Tid:** 1 timme  
**Tema:** Att söka är bara GET med extra parametrar  
**Mål:** Avmystifierar "avancerade" features - det är bara fler parametrar

---

## 🤔 TÄNK FÖRST (5 min)

### **Vad är query parameters?**

```
https://api.example.com/todos?done=false&limit=10
                            ↑
                    Query parameters
```

**Det är bara data I URL:en!**

**Airtable `filterByFormula`:**

- Airtables sätt att filtrera records
- Skrivs som formel: `{Done} = FALSE()`
- URL-encodas när skickat

**Frågor att fundera på:**

1. Hur skiljer sig filtered GET från vanlig GET? (Bara URL!)
2. Vem gör filtreringen? (Backend/Airtable, inte frontend!)
3. Vad är URL encoding? Varför behövs det?
4. Hur bygger jag en sökfunktion? (Input → query parameter → GET)

**Skriv ner dina tankar** ✏️

---

## 🎯 Vad ska vi uppnå?

Efter denna session ska du ha:

✅ Förstått query parameters  
✅ Använt Airtable filterByFormula  
✅ Filtrerat på Done status (visa bara ogjorda/gjorda)  
✅ Sökt i Task text  
✅ Filter-knappar i browser  
✅ Sökfält i browser

---

## 🧪 CHECKPOINT 1: Filter med curl (15 min)

### **Airtable filterByFormula syntax:**

**Visa bara ogjorda:**

```
filterByFormula={Done} = FALSE()
```

**Visa bara gjorda:**

```
filterByFormula={Done} = TRUE()
```

**Sök i Task text:**

```
filterByFormula=SEARCH("mjölk", {Task})
```

### **URL encoding:**

Spaces och special characters måste encodas:

- Space ` ` → `%20` eller `+`
- `{` → `%7B`
- `}` → `%7D`
- `=` → `%3D`

**Tips:** Postman gör detta automatiskt!

### **CMD (manuell encoding):**

```cmd
curl -H "Authorization: Bearer patXXXXX" ^
  "https://api.airtable.com/v0/appXXXXX/Todos?filterByFormula=%7BDone%7D%20%3D%20FALSE()"
```

**För svårt!** Använd PowerShell eller Postman istället.

### **PowerShell (automatisk encoding):**

```powershell
$headers = @{ "Authorization" = "Bearer patXXXXX" }
$filter = "{Done} = FALSE()"
$encodedFilter = [System.Web.HttpUtility]::UrlEncode($filter)

Invoke-RestMethod -Uri "https://api.airtable.com/v0/appXXXXX/Todos?filterByFormula=$encodedFilter" -Headers $headers
```

**Enklare: Låt Postman hantera det!**

---

## 🧪 CHECKPOINT 2: Filter i Postman (15 min)

### **Visa bara ogjorda todos:**

1. **Kopiera GET Todos request**
2. **Params tab** (under URL)
3. **Add param:**
   - Key: `filterByFormula`
   - Value: `{Done} = FALSE()`
4. **Send**

**Postman encodar automatiskt!** Se URL:

```
.../Todos?filterByFormula=%7BDone%7D%20%3D%20FALSE()
```

### **Visa bara gjorda:**

- Value: `{Done} = TRUE()`

### **Sök efter "mjölk":**

- Value: `SEARCH("mjölk", {Task})`

### **Tips: Spara som separate requests:**

- "Get Undone Todos"
- "Get Done Todos"
- "Search Todos"

---

## 📝 CHECKPOINT 3: Express filter endpoint (20 min)

### **Strategi 1: Separate routes**

```javascript
// Bara ogjorda
app.get("/api/todos/undone", async (req, res) => {
  const filter = encodeURIComponent("{Done} = FALSE()");
  const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}?filterByFormula=${filter}`;
  // ... fetch och returnera
});

// Bara gjorda
app.get("/api/todos/done", async (req, res) => {
  const filter = encodeURIComponent("{Done} = TRUE()");
  const url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}?filterByFormula=${filter}`;
  // ... fetch och returnera
});
```

### **Strategi 2: Query parameter (flexiblare!)**

```javascript
app.get("/api/todos", async (req, res) => {
  try {
    const { status, search } = req.query;

    let url = `https://api.airtable.com/v0/${AIRTABLE_BASE}/${AIRTABLE_TABLE}`;

    // Bygg filter
    let filter = "";
    if (status === "done") {
      filter = "{Done} = TRUE()";
    } else if (status === "undone") {
      filter = "{Done} = FALSE()";
    } else if (search) {
      filter = `SEARCH("${search}", {Task})`;
    }

    if (filter) {
      url += `?filterByFormula=${encodeURIComponent(filter)}`;
    }

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

### **Testa:**

```cmd
# Alla
curl http://localhost:3000/api/todos

# Bara ogjorda
curl "http://localhost:3000/api/todos?status=undone"

# Bara gjorda
curl "http://localhost:3000/api/todos?status=done"

# Sök
curl "http://localhost:3000/api/todos?search=mjolk"
```

**PowerShell:**

```powershell
Invoke-RestMethod -Uri "http://localhost:3000/api/todos?status=undone"
```

---

## 📝 CHECKPOINT 4: Filter UI i browser (15 min)

### **Uppdatera index.html:**

```html
<h2>Todos</h2>
<div>
  <button id="showAll">All</button>
  <button id="showUndone">Undone</button>
  <button id="showDone">Done</button>
</div>

<div>
  <input type="text" id="searchInput" placeholder="Search..." />
  <button id="searchBtn">Search</button>
</div>

<ul id="todoList"></ul>

<script>
  async function loadTodos(params = "") {
    const url = `http://localhost:3000/api/todos${params}`;
    const response = await fetch(url);
    const data = await response.json();

    const list = document.getElementById("todoList");
    list.innerHTML = "";

    if (data.records.length === 0) {
      list.innerHTML = "<li>No todos found</li>";
      return;
    }

    data.records.forEach((record) => {
      // ... samma som tidigare (checkbox, task, delete)
    });
  }

  // Filter buttons
  document.getElementById("showAll").addEventListener("click", () => {
    loadTodos();
  });

  document.getElementById("showUndone").addEventListener("click", () => {
    loadTodos("?status=undone");
  });

  document.getElementById("showDone").addEventListener("click", () => {
    loadTodos("?status=done");
  });

  // Search
  document.getElementById("searchBtn").addEventListener("click", () => {
    const search = document.getElementById("searchInput").value;
    if (search) {
      loadTodos(`?search=${encodeURIComponent(search)}`);
    } else {
      loadTodos();
    }
  });

  // Initial load
  loadTodos();
</script>
```

### **Test:**

1. Klicka "Undone" → se bara ogjorda
2. Klicka "Done" → se bara gjorda
3. Klicka "All" → se alla
4. Sök "mjölk" → se matchande

---

## 💭 REFLEKTERA (10 min)

### **Pausa och fundera:**

1. **Vad är query parameters egentligen?**

   - Data i URL efter `?`
   - Key-value pairs: `?key=value&key2=value2`

2. **Vem gör filtreringen?**

   - Backend (Airtable i detta fall)
   - Frontend skickar bara parametrar

3. **Varför URL encoding?**

   - Special characters (`{}=()`) kan inte vara direkt i URL
   - `encodeURIComponent()` fixar det

4. **Hur bygger man en sökfunktion?**

   - Input → query parameter → GET med filter → visa resultat

5. **Är detta "avancerat"?**
   - Nej! Det är bara GET med extra URL-parametrar!

### **Skriv ner i reflection.md!**

## Skapa en kopia av reflection.md från Session 4

## 🤖 AI-MOMENT (5 min)

### **Fråga Claude:**

**Exempel-frågor:**

- "Förklara URL query parameters"
- "Vad är URL encoding och varför behövs det?"
- "Airtable filterByFormula syntax exempel"
- "MDN artikel om URLSearchParams"

---

## ✅ Session 5 Klar!

### **Checklista:**

- [ ] Förstått query parameters
- [ ] Testat filterByFormula i Postman
- [ ] Express endpoint med filter
- [ ] Filter-knappar i browser fungerar
- [ ] Sökfält fungerar
- [ ] Förstått att backend filtrerar, inte frontend

### **Nästa session:**

**Session 6: Connect The Dots**  
Hela flödet från tanke till implementation!

**Ta en paus! Du kan söka och filtrera! 🔎**
