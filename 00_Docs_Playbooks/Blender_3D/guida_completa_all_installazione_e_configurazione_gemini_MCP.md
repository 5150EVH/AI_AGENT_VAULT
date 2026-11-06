# Guida Completa all'Installazione e Configurazione dell'Ambiente
## Architettura: Obsidian Vault + GitHub + Conda + Blender 5.2.x + Gemini CLI + MCP Bridge

---

### NOTA INTRODUTTIVA CRITICA: Modelli LLM e Modellazione 3D

In questo setup viene utilizzato come modello di default della CLI un modello snello ad alta velocità (come **`gemini-3.1-flash-lite`** o **`gemini-2.5-flash`**).

#### Perché usiamo questo modello:
1. **Evitare errori di rate-limit (HTTP 503 / 429):** I modelli Pro e di fascia alta subiscono spesso picchi di saturazione su API tier gratuiti.
2. **Latenza minima:** I modelli Flash rispondono in pochi secondi, consentendo un'interazione fluida nel terminale.

#### Limiti operativi nella modellazione 3D:
* **Assenza di reale ragionamento geometrico/spaziale:** I modelli leggeri non "visualizzano" lo spazio euclideo 3D continuo. Ragionano per probabilità sintattica di codice Python.
* **Approssimazione tramite primitive:** Quando viene richiesta una mesh complessa (es. un'automobile, un albero o un animale low-poly), il modello tende a creare cubi, cilindri e sfere separati e accostarli nello spazio.
* **Mancanza di topologia coerente:** Senza istruzioni rigide di modellazione poligonale (tramite la libreria `bmesh`, modificatori Mirror/Subsurf, Loop Cut ed estrusione di facce) o senza un loop visivo continuo (come il tool `mcp_blender_look`), le mesh generate non hanno una topologia professionale unificata.

---

## FASE 1: Creazione dell'Infrastruttura Dati (Obsidian & Cartelle)

### 1.1 Installazione di Obsidian
1. Scarica Obsidian dal sito ufficiale (`https://obsidian.md`) e installalo su Windows.
2. All'avvio seleziona **Create new vault**.
3. Denomina il Vault: `AI_AGENT_VAULT`.
4. Posizionalo in un percorso di root dedicato per evitare conflitti con spazi e permessi Windows:
   `C:\AI_VAULT\AI_AGENT_VAULT`
5. Clicca su **Create**.

---

## FASE 2: GitHub, Git e Controllo Versione

### 2.1 Creazione Account e Repository GitHub
1. Accedi a [github.com](https://github.com/) (o crea un nuovo account gratuito).
2. Clicca sull'icona **+** in alto a destra e seleziona **New repository**.
3. Configura il repository:
   * **Repository name:** `AI_AGENT_VAULT`
   * **Visibility:** Private (o Public a tua scelta)
   * **Initialize with:** Deseleziona README, .gitignore e licenza (li collegheremo dalla cartella locale).
4. Clicca su **Create repository**. Copia l'URL HTTPS generato:
   `https://github.com/<TUO_USERNAME>/AI_AGENT_VAULT.git`

### 2.2 Installazione e Configurazione di Git su Windows
1. Scarica Git per Windows da [git-scm.com](https://git-scm.com/) e completa l'installazione guidata mantenendo le opzioni predefinite.
2. Apri il terminale Windows (`cmd`) e imposta la tua identità globale:
   ```cmd
   git config --global user.name "Tuo Nome"
   git config --global user.email "tua-email@example.com"
   ```

### 2.3 Collegamento del Vault a GitHub
1. Apri il prompt dei comandi e spostati nella cartella del Vault:
   ```cmd
   cd C:\AI_VAULT\AI_AGENT_VAULT
   ```
2. Inizializza il repository Git locale:
   ```cmd
   git init
   git branch -M main
   ```
3. Crea un file `.gitignore` per escludere file temporanei, cache e credenziali:
   ```cmd
   (
   echo .obsidian/workspace*
   echo node_modules/
   echo .gemini/credentials.json
   echo *.blend1
   echo __pycache__/
   ) > .gitignore
   ```
4. Collega il remote repository di GitHub e fai il primo commit:
   ```cmd
   git remote add origin https://github.com/<TUO_USERNAME>/AI_AGENT_VAULT.git
   git add .
   git commit -m "feat: setup iniziale vault e tracking"
   git push -u origin main
   ```
   *(Inserisci credenziali o Personal Access Token di GitHub se richiesto dal browser).*

---

## FASE 3: Ambiente Conda, Node.js e Gemini CLI

### 3.1 Installazione di Miniconda / Anaconda
1. Se non presente, installa **Miniconda** (o Anaconda) per Windows 64-bit selezionando l'opzione standard.
2. Apri **Anaconda Prompt** dal menu Start di Windows.

### 3.2 Creazione dell'Ambiente Dedicato
1. Crea un ambiente isolato con Python 3.13 e Node.js:
   ```cmd
   conda create -n blender-ai python=3.13 nodejs -y
   ```
2. Attiva l'ambiente:
   ```cmd
   conda activate blender-ai
   ```
   *Verifica:* Il prefisso del prompt mostrerà `(blender-ai)`.

### 3.3 Installazione dei Gestori e Pacchetti CLI
1. Installa `uv` (gestore pacchetti Python ultra-veloce):
   ```cmd
   pip install uv
   ```
2. Installa globalmente nell'ambiente la CLI ufficiale di Gemini:
   ```cmd
   npm install -g @google/gemini-cli
   ```
3. *Verifica installazioni:*
   ```cmd
   python --version
   uv --version
   gemini --version
   ```

### 3.4 Configurazione Chiave Google AI Studio
1. Vai su [aistudio.google.com](https://aistudio.google.com/).
2. Clicca su **Get API key** > **Create API key** e copia la chiave generata (`AIzaSy...`).
3. Impostala nella variabile d'ambiente di Windows:
   ```cmd
   setx GEMINI_API_KEY "INCOLLA_QUI_LA_TUA_CHIAVE"
   ```

---

## FASE 4: Blender, Versione Python e Plugin MCP

### 4.1 Installazione di Blender
1. Scarica e installa **Blender 5.2.x LTS** da [blender.org](https://www.blender.org/).
2. Avvia Blender.

### 4.2 Come Individuare la Versione Python di Blender
Blender include un interprete Python interno indipendente da quello di Conda o di sistema. Per verificare quale versione usa:
1. In Blender, cambia l'area di lavoro in alto selezionando la scheda **Scripting**.
2. Nel riquadro in basso a sinistra (**Python Console**), leggi la prima riga di testo. Mostrerà una dicitura simile a:
   `PYTHON INTERACTIVE CONSOLE 3.14.x (default, ...) [MSC v.19xx 64 bit (AMD64)]`
3. In alternativa digita nella console di Blender:
   ```python
   import sys; print(sys.version)
   ```
   *Nota:* Sapere questo ti garantisce che gli script generati non utilizzino moduli incompatibili con l'interprete integrato.

### 4.3 Installazione del Server / Plugin MCP per Blender
1. Apri Anaconda Prompt nell'ambiente `blender-ai` e lancia il comando di installazione dell'add-on fornito dal bridge:
   ```cmd
   uvx mcp-for-blender install-addon
   ```
   *Verifica:* Il comando confermerà che l'add-on è stato copiato nella cartella script di Blender (`%APPDATA%\Blender Foundation\Blender\5.2\scripts\addons\blender_mcp.py`).
2. Apri Blender:
   * Vai in **Edit** > **Preferences** > **Add-ons** (o **Get Extensions**).
   * Cerca nella casella di testo: `Blender MCP` o `MCP for Blender`.
   * Spunta la casella per abilitare l'add-on.
3. Nella 3D Viewport di Blender, premi il tasto **N** per aprire la barra laterale destra.
4. Seleziona la scheda **MCP for Blender** e clicca sul pulsante **Connect**.
   *Verifica:* Deve comparire la scritta verde:
   `✔ Connected on localhost:9876` e il pulsante deve mostrare la scritta `Disconnect`.

---

## FASE 5: Configurazione dei File di Connessione MCP

Affinché Gemini CLI sappia come comunicare con Blender, serve il file di configurazione `settings.json`.

### 5.1 Creazione del file `settings.json` Utente e Locale
Apri VS Code ed effettua i seguenti salvataggi:

1. Crea un nuovo file, incolla questo JSON:
   ```json
   {
     "mcpServers": {
       "blender": {
         "command": "uvx",
         "args": ["mcp-for-blender"]
       }
     }
   }
   ```
2. Fai **File > Save As...** e salvalo nei seguenti due percorsi:
   * Percorso Globale Utente: `C:\Users\<TUO_UTENTE>\.gemini\settings.json`
   * Percorso Locale Vault: `C:\AI_VAULT\AI_AGENT_VAULT\.gemini\settings.json`
   *(Se la cartella `.gemini` non esiste, creala).*

### 5.2 Creazione del file `GEMINI.md` per le Istruzioni di Sistema
Crea un file denominato `GEMINI.md` direttamente nella radice del Vault (`C:\AI_VAULT\AI_AGENT_VAULT\GEMINI.md`):

```markdown
# Istruzioni Operative per Blender MCP

- Quando l'utente chiede modifiche alla scena 3D di Blender, usa direttamente il tool `mcp_blender_execute_blender_code`.
- Non limitarti a scrivere codice testuale: esegui sempre l'azione tramite lo strumento.
- Regole sintattiche obbligatorie per Blender (bpy):
  - Usa `bpy.context.active_object` e MAI `bpy.active_object`.
  - Assicurati che ogni materiale abbia `mat.use_nodes = True` e assegna i valori al nodo `Principled BSDF`.
  - Non sovrapporre primitive vuote: se viene richiesta una forma articolata, usa la manipolazione topologica o unisci le mesh applicando i modificatori appropriati.
```

---

## FASE 6: Check-up di Verifica Finale del Sistema

Esegui questi tre controlli per assicurarti che tutto sia allineato prima di lavorare:

1. **Controllo Porta Blender:**
   Blender aperto con pannello laterale (tasto **N**) in stato `✔ Connected on localhost:9876`.
2. **Controllo Bridge uvx:**
   Nel terminale Conda esegui:
   ```cmd
   uvx mcp-for-blender --help
   ```
   Non deve restituire errori di moduli mancanti.
3. **Controllo Riconoscimento Tool MCP:**
   Nel terminale Conda posizionati nel Vault ed esegui:
   ```cmd
   cd C:\AI_VAULT\AI_AGENT_VAULT
   gemini --skip-trust -m gemini-3.1-flash-lite
   ```
   All'interno dell'interfaccia digita `/mcp list`.
   *Esito corretto:* Deve mostrare `🟢 blender - Ready (9 tools, 1 resource)`. Digita `/exit` per uscire.