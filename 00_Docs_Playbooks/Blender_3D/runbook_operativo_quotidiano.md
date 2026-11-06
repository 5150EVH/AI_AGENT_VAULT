# Runbook Operativo: Avvio Sessione e Smoke Test
## Procedura quotidiana di esecuzione da computer appena acceso

---

### Prerequisiti
* Computer Windows avviato.
* Nessun processo background bloccato su porte di rete.

---

## Procedura Guidata Passo-Passo

### 1. Avviare Blender 5.2 e attivare il Listener
1. Apri l'applicazione **Blender**.
2. Se compare la schermata di Splash, fai un clic all'interno dell'area di lavoro 3D per chiuderla.
3. Premi il tasto **N** sulla tastiera (apre la barra laterale destra della 3D Viewport).
4. Clicca sulla scheda **MCP for Blender**.
5. Controlla lo stato:
   * Se vedi `Not connected`, clicca sul pulsante **Connect**.
   * Lo stato deve diventare: `✔ Connected on localhost:9876`.
   * Il pulsante sotto deve mostrare la scritta **Disconnect**.

> **IMPORTANTE:** Lascia la finestra di Blender aperta sullo sfondo o affiancata al terminale. Non chiuderla mai durante la sessione di generazione.

---

### 2. Aprire Anaconda Prompt e attivare l'ambiente
1. Apri il menu Start di Windows e cerca **Anaconda Prompt** (o Miniconda Prompt).
2. Attiva l'ambiente virtuale configurato con le dipendenze:
   ```cmd
   conda activate blender-ai
   ```
3. Spostati nella cartella del tuo Vault di lavoro:
   ```cmd
   cd C:\AI_VAULT\AI_AGENT_VAULT
   ```
   *Verifica del prompt:* La riga di comando deve indicare:
   `(blender-ai) C:\AI_VAULT\AI_AGENT_VAULT>`

---

### 3. Avviare Gemini CLI con i flag corretti
Lancia Gemini impostando i flag collaudati per bypassare conferme e blocchi su Windows:

```cmd
gemini --skip-trust -y -m gemini-3.1-flash-lite
```

#### Significato dei parametri:
* `--skip-trust`: Salta il controllo di sicurezza sulla cartella locale, caricando immediatamente i server MCP in `settings.json`.
* `-y`: Modalità YOLO (approvazione automatica di tool e codice, senza fermarsi ad attendere conferme invisibili).
* `-m gemini-3.1-flash-lite`: Utilizza il modello Flash a bassa latenza che non va in blocco per picchi di traffico (errore 503).

*Verifica:* Si apre la schermata della CLI con il testo in basso:
`Type your message or @path/to/file` e l'indicazione `1 MCP server` attivo.

---

### 4. Esecuzione dello Smoke Test: Sfera Rossa al Centro
Nella riga di input di Gemini, digita o incolla il comando di test:

```text
Usa mcp_blender_execute_blender_code per pulire la scena corrente e creare una sfera rossa al centro degli assi.
```

Premi **Invio**.

#### Cosa accade:
1. Gemini invoca il tool MCP `execute_blender_code`.
2. Il codice Python `bpy` viene inviato sul socket `localhost:9876`.
3. Nel terminale compare la notifica di esecuzione con spunta verde (`Code executed successfully`).
4. **Verifica visiva su Blender:**
   * Tutti gli oggetti preesistenti (cubo di default, lampada, camera) vengono cancellati.
   * Al centro della scena (coordinate `0, 0, 0`) compare una sfera (UV Sphere).
   * Se attivi la modalità di visualizzazione Material Preview o Rendered in Blender (tasto **Z** > *Material Preview*), la superficie della sfera sarà di colore rosso vivo.

---

### Scorciatoia Rapida da Singola Riga (Headless)
Se desideri eseguire un comando direttamente dal terminale di Windows senza entrare nell'interfaccia interattiva della CLI, con Blender già connesso sul socket 9876, puoi lanciare direttamente:

```cmd
gemini --skip-trust -y -m gemini-3.1-flash-lite -p "Usa mcp_blender_execute_blender_code per pulire la scena corrente e creare una sfera rossa al centro degli assi."
```

Il comando eseguirà l'azione e restituirà il controllo al terminale una volta completata la sfera in Blender.