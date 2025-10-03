# Session 2 Curl Examples

## 📘 Alla curl-kommandon för Session 2

---

## 1. Direkt till Airtable (GET)

### **CMD (Windows):**

```cmd
curl -H "Authorization: Bearer patXXXXXXXXXXXXXX" https://api.airtable.com/v0/appXXXXXXXXXXXXXX/Todos
```

### **PowerShell:**

```powershell
$headers = @{ "Authorization" = "Bearer patXXXXXXXXXXXXXX" }
Invoke-RestMethod -Uri "https://api.airtable.com/v0/appXXXXXXXXXXXXXX/Todos" -Headers $headers
```

---

## 2. Via din Express-server (GET)

### **CMD:**

```cmd
curl http://localhost:3000/api/todos
```

### **PowerShell:**

```powershell
Invoke-RestMethod -Uri http://localhost:3000/api/todos
```

### **Med verbose (se headers):**

```cmd
curl -v http://localhost:3000/api/todos
```

---

## 3. Formattera JSON output

### **CMD med jq:**

```cmd
curl http://localhost:3000/api/todos | jq .
```

### **PowerShell (automatiskt formaterat):**

```powershell
Invoke-RestMethod -Uri http://localhost:3000/api/todos | ConvertTo-Json -Depth 10
```

---

## 4. Spara response till fil

### **CMD:**

```cmd
curl http://localhost:3000/api/todos -o todos.json
```

### **PowerShell:**

```powershell
Invoke-RestMethod -Uri http://localhost:3000/api/todos | ConvertTo-Json -Depth 10 | Out-File todos.json
```

---

## 5. Testa med olika endpoints

### **Ping (från Session 1):**

```cmd
curl http://localhost:3000/ping
```

### **Echo (från Session 1):**

```cmd
curl -X POST http://localhost:3000/echo ^
  -H "Content-Type: application/json" ^
  -d "{\"test\": \"hej\"}"
```

### **Todos:**

```cmd
curl http://localhost:3000/api/todos
```

---

## 🔍 Felsökning med curl

### **Verbose mode (se all HTTP traffic):**

```cmd
curl -v http://localhost:3000/api/todos
```

**Visar:**

- Request headers
- Response headers
- Status code
- Timing

### **Bara headers (ingen body):**

```cmd
curl -I http://localhost:3000/api/todos
```

### **Follow redirects:**

```cmd
curl -L http://localhost:3000/api/todos
```

---

## 💡 Tips

### **Aliaser för PowerShell**

Lägg till i PowerShell-profil för snabbare kommandon:

```powershell
function Get-Todos {
    Invoke-RestMethod -Uri http://localhost:3000/api/todos
}

# Använd:
Get-Todos
```

### **CMD batch script**

Skapa `test-todos.bat`:

```batch
@echo off
curl http://localhost:3000/api/todos
pause
```

### **PowerShell script**

Skapa `test-todos.ps1`:

```powershell
$response = Invoke-RestMethod -Uri http://localhost:3000/api/todos
$response.records | ForEach-Object {
    $task = $_.fields.Task
    $done = if ($_.fields.Done) { "✓" } else { "☐" }
    Write-Host "$done $task"
}
```

Kör:

```cmd
powershell -ExecutionPolicy Bypass -File test-todos.ps1
```

---

## 🆘 Vanliga fel

### **"curl: command not found"**

**CMD:** curl finns i Windows 10+ (1803 och nyare)  
**Lösning:** Använd PowerShell `Invoke-RestMethod` istället

### **"Failed to connect to localhost"**

**Orsak:** Servern körs inte  
**Lösning:** Kör `npm start` i annan terminal

### **403 Forbidden (från Airtable)**

**Orsak:** PAT har fel scopes eller access  
**Lösning:** Se huvudövningens Setup-sektion

### **Escape-tecken krånglar**

**CMD:** Använd `^` för line breaks, `\"` för quotes i JSON  
**PowerShell:** Använd `'` för strings, `` ` `` för line breaks

---

**Spara denna fil för snabbreferens! 📚**
