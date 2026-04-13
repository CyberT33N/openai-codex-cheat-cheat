# 🧠 MCP 

## 🧩 Grundprinzip

**Model Context Protocol (MCP)** verbindet das Modell mit:

* lokalen Tools (z. B. Filesystem, CLI)
* externen Services (z. B. Browser, APIs)

👉 Konfiguration erfolgt **ausschließlich über `config.toml`**
👉 Gilt **global (`~/.codex/config.toml`) oder projektbezogen (`.codex/config.toml`)**

---

# ⚙️ Unterstützte MCP-Server-Typen (wichtig!)

Codex unterstützt aktuell:

| Typ                 | Beschreibung                         |
| ------------------- | ------------------------------------ |
| **STDIO**           | Lokaler Prozess (`command` + `args`) |
| **Streamable HTTP** | HTTP-basierter MCP-Endpunkt (`url`)  |

❌ **Nicht unterstützt (in Codex aktuell):**

* `type: "sse"` (Server-Sent Events)

---

# ✅ Deine funktionierenden MCP-Server

## 1. 📁 filesystem-extended (STDIO)

### Zweck

* Zugriff auf lokales Filesystem
* Suchen, Lesen, Analysieren von Dateien

### Konfiguration

```toml
[mcp_servers."filesystem-extended"]
command = "node"
args = [ "...", "..."]
enabled = true

enabled_tools = [ ... ]
disabled_tools = [ ... ]
```

### Fähigkeiten (aus deiner Config)

* Directory Traversal
* File Search (Regex / Glob)
* File Metadata & Checksums
* Content Reading mit Zeilennummern

### Einschränkungen (bewusst gesetzt)

* ❌ Kein Schreiben (`create_files`, `patch_files`, etc.)
* 👉 Read-only Fokus → **sicher & kontrolliert**

---

## 2. 🌐 playwright (Streamable HTTP)

### Zweck

* Browser-Automation
* UI-Interaktion
* Web-Scraping / Testing

### Konfiguration

```toml
[mcp_servers.playwright]
url = "http://127.0.0.1:8932/mcp"
enabled = true

enabled_tools = [ ... ]
```

### Fähigkeiten

* Navigation (`browser_navigate`)
* DOM-Interaktion (`browser_click`)
* JS-Ausführung (`browser_evaluate`)
* Screenshots (`browser_take_screenshot`)
* Netzwerkanalyse

👉 Das ist im Prinzip ein **Headless-Browser als Tool für das LLM**

---

# ❌ Deine NICHT funktionierenden MCP-Server

## 3. fetcher (SSE)

```json
"type": "sse"
"url": "http://127.0.0.1:3001/sse"
```

## 4. crawl4ai (SSE)

```json
"type": "sse"
"url": "http://127.0.0.1:11236/mcp/sse"
```

---

## 🚫 Problem: Transport-Type

Diese beiden nutzen:

```json
"type": "sse"
```

👉 **Problem:**
Codex unterstützt **kein SSE als MCP-Transport**

---

## 🧠 Was bedeutet das technisch?

| Transport       | Verhalten                                |
| --------------- | ---------------------------------------- |
| STDIO           | Lokaler Prozess via stdin/stdout         |
| Streamable HTTP | Request/Response + Streaming             |
| SSE             | Server pusht Events → ❌ nicht kompatibel |

👉 SSE ist **push-basiert**, Codex erwartet aber:

* Pull / Request-Response oder
* kontrolliertes Streaming

---

## 🛠️ Lösungsmöglichkeiten

### Option 1 (empfohlen)

➡️ Server auf **Streamable HTTP MCP** umstellen

### Option 2

➡️ Wrapper/Proxy bauen:

* SSE → HTTP Bridge

### Option 3

➡️ Alternative MCP-Server nutzen

---

# 🧱 Architektur — Wichtige Konzepte

## 1. 🔌 MCP = Tool Layer für das Modell

Das Modell selbst:

* kann **keine echten Aktionen ausführen**

MCP:

* erweitert es um **externe Fähigkeiten**

👉 Architektur:

```
LLM
 ↓
Codex Agent
 ↓
MCP Server (Tools)
 ↓
System / APIs / Browser
```

---

## 2. 🧩 Entkopplung durch MCP

Vorteil:

* Tools sind **austauschbar**
* Modell bleibt gleich

Beispiel:

* Playwright → kann durch Chrome DevTools ersetzt werden
* Filesystem → durch Remote FS

---

## 3. 🔐 Sicherheitsmodell

Wichtige Mechanismen:

### Tool Whitelisting

```toml
enabled_tools = [ ... ]
```

👉 Nur diese Tools darf das Modell nutzen

### Tool Blacklisting

```toml
disabled_tools = [ ... ]
```

👉 Sicherheitsbarriere

---

## 4. ⚖️ Kontrolle vs. Autonomie

Deine Config zeigt ein gutes Pattern:

| Bereich     | Entscheidung |
| ----------- | ------------ |
| Filesystem  | read-only    |
| Browser     | kontrolliert |
| Write-Tools | deaktiviert  |

👉 Best Practice:
**LLM darf lesen + analysieren, aber nicht unkontrolliert schreiben**

---

## 5. 🧠 Tool-Semantik ist entscheidend

Das Modell versteht Tools über:

* Namen (`browser_click`)
* Beschreibung (vom Server)
* Kontext

👉 Gute MCP-Server:

* haben **klare, atomare Tools**
* sind **deterministisch**

---

## 6. ⚡ Runtime-Verhalten

Wichtig zu verstehen:

* MCP-Server werden **zur Laufzeit gestartet / verbunden**
* Wenn sie nicht sichtbar sind:

  * Config invalid ❌
  * Server startet nicht ❌
  * Extension neu laden nötig ⚠️

👉 Dein Fall:
➡️ Neustart → Config wurde neu geladen → funktioniert

---

# ⚠️ Typische Fehlerquellen

## 1. TOML Syntax

* Backslashes falsch escaped
* Keys nach Tables (invalid)

## 2. Pfade (Windows!)

* `"C:\..."` ❌
* `'C:\...'` ✅

## 3. Server startet nicht

* `node` nicht im PATH
* Script crasht

## 4. Falscher Transport

* SSE ❌
* HTTP/STDIO ✅

---

# 🧭 Best Practices

✅ Nur notwendige Tools aktivieren
✅ Schreibzugriff minimieren
✅ MCP-Server lokal testen
✅ HTTP bevorzugen für Remote
✅ STDIO für lokale Tools

---

# 🧾 TL;DR

* MCP wird über **`config.toml`** konfiguriert
* Nur **STDIO + Streamable HTTP** funktionieren
* Deine funktionierenden Server:

  * ✅ filesystem-extended
  * ✅ playwright
* Deine nicht funktionierenden:

  * ❌ fetcher (SSE)
  * ❌ crawl4ai (SSE)
* Architektur:

  * MCP = Tool-Layer für das LLM
  * Fokus auf Sicherheit + Kontrolle


## Full example
```toml
model = "gpt-5.4"
model_reasoning_effort = "medium"

[windows]
sandbox = "unelevated"

[projects.'C:\Projects\utils\ai\voice\VoiceTyper']
trust_level = "trusted"

[mcp_servers."filesystem-extended"]
command = "node"
args = [
  "c:\\Projects",
  "C:\\Projects",
]
enabled = true
enabled_tools = [
  "list_directories",
  "directory_trees",
  "search_regexes",
  "search_globs",
  "list_directory_entries",
  "get_file_infos",
  "get_path_metadata",
  "get_file_checksums",
  "list_allowed_directories",
  "find_files_by_glob",
  "find_paths_by_name",
  "count_lines",
  "read_files_with_line_numbers",
  "search_file_contents_by_regex"
]
disabled_tools = [
  "patch_files",
  "create_files",
  "append_files",
  "replace_file_line_ranges"
]

[mcp_servers.playwright]
url = "http://127.0.0.1:8932/mcp"
enabled = true
enabled_tools = [
  "browser_close",
  "browser_tabs",
  "browser_network_requests",
  "browser_run_code",
  "browser_navigate",
  "browser_snapshot",
  "browser_click",
  "browser_evaluate",
  "browser_take_screenshot"
]

```

