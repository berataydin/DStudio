# Qwen — checkpoint per riprendere il lavoro

Questo documento distingue ciò che funziona, ciò che è verificato e ciò che
resta da completare. La tranche corrente riguarda **Qwen3.6-35B-A3B e
Qwen3.8-Flash-Next già integrati su macOS Metal**, non tutti i requisiti Qwen
del piano di lavoro locale.

## Punto di arresto

**Consolidamento Qwen3.6/3.8 verificato il 7 settembre 2026; pausa richiesta.**
Fermarsi qui: gli altri capitoli non devono partire automaticamente. Questo
chiude la tranche dei due modelli già integrati, non la qualificazione completa
della famiglia o del piano.
Questa tranche è il perimetro dell'aggiornamento Qwen richiesto per GitHub,
insieme al README e alle dipendenze di caricamento/build condivise.
La pubblicazione non riprende gli altri capitoli del piano; la .app dell'utente
non è stata sostituita o riavviata.

| Checkpoint | Stato | Risultato / limite |
| --- | --- | --- |
| Q-01 — strumenti e avvio | Verificato su percorsi mirati | Chat e Agent/Cowork sperimentali; strumenti nativi, file creati e riletti, motore corretto. Non è qualità generale. |
| Q-02 — nuova sessione | Verificato nei replay mirati | Agent e Cowork passano su entrambi i modelli reali: progresso nativo, annullamento con contesto conservato, reset completo e rilettura. |
| Q-03 — checkpoint su disco | Capacità distinta | Qwen3.6 non serializza lo stato ricorrente completo: checkpoint disabilitati, cronologia dell'app conservata. Qwen3.8 mantiene il proprio formato nativo. |
| Q-04 — finestra desktop | Da completare | HTTP e UI simulata non sostituiscono la prova della vera .app in primo piano. |
| Q-05 — qualità generale | Da completare | Baseline Qwen3.6 11/12, Qwen3.8 12/12. Restano almeno 100 casi comuni per modello e riferimenti numerici. |
| Q-06 — altre modalità e nuovo 27B | Da implementare/qualificare | Learn/thinking, Design, visione/PDF e Qwen27B non diventano supportati per effetto di questa tranche. |

## Modifiche da preservare

- [Patch native](../patch/ds4-agent-jsonl/README.md), manifest **91**: restano
  tokenizer, kernel e formato degli strumenti dei rispettivi fork.
- Il reset prepara al massimo una sessione KV candidata con lo stesso contesto
  configurato e pesi condivisi, non una seconda copia del modello. Il vecchio
  contesto resta valido fino alla pubblicazione. Errori restituiti dall'API e
  annullamento non devono distruggerlo.
- Un salvataggio richiesto deve riuscire prima del reset. Se riesce e il reset
  viene annullato, quel salvataggio rimane. Un salvataggio fallito non assegna
  una nuova identità alla sessione.
- Riutilizzo della cache del prompt solo se compatibile, nessuna scrittura
  speculativa. Preparazione e rilascio dei vecchi buffer fuori dal mutex.
  Nessun abbassamento automatico del contesto.
- La UI aspetta l'esito del reset prima di collegare il motore alla nuova
  conversazione. Una ricevuta nativa di errore non è un successo. Richieste
  diventate obsolete durante un cambio vista non vengono inviate.
- Reset legacy main/Laguna e CLI TTY restano interventi separati; le loro
  varianti non sono state cambiate o qualificate per deduzione.

## Identità

| Componente | Revisione |
| --- | --- |
| Qwen3.6, vagrillo/ds4 | 60fca11f0c8b16ca50c757324dddd717ba043098 |
| Qwen3.8, ivanfioravanti/ds4-metal | 66b0e3fc3bf0f548db1ec0c0dd19f4e43567a7f8 |
| Adattatori DStudio | patch/ds4-agent-jsonl/manifest, versione 91 |

Hash di sorgente, patch e file derivato: [bases.json](../patch/ds4-agent-jsonl/bases.json).
Un checkout da archivio non eredita l'identità Git della cartella padre.
Non aggiornare fork o pesi durante un replay senza registrare una nuova base.

Ultimi replay: Mac M2 Max con 96 GB, Metal, contesto **16.384 token**, thinking
disattivato e impostazioni di campionamento native. Qwen3.6 usa il Q6_K_XL da
31,8 GB residente; Qwen3.8 il Q4K/MXFP4 da 73,4 GB residente più il PLE Q4_1
da 32,0 GB su SSD. Expert streaming disattivato in entrambi. Nessuna prova a
128k implicita: capacità configurata e quantità di prompt elaborata sono distinte.
Identità complete dei file e binari nelle ricevute private dei replay.

## Test da ripetere

Comandi dalla radice del progetto; sostituire i percorsi degli engine con i
checkout esatti già compilati. Non caricano pesi:

    make test-engine-setup-unit test-launch-preflight test-agent-spawn
    make test-qwen-session-reset QWEN35_AGENT_TREE=/path/to/ds4-qwen35 QWEN38_AGENT_TREE=/path/to/ds4-qwen38
    make test-qwen35-agent QWEN35_AGENT_TREE=/path/to/ds4-qwen35 QWEN35_AGENT_FLAGS=--sanitize
    make test-qwen38-agent QWEN38_AGENT_TREE=/path/to/ds4-qwen38 QWEN38_AGENT_FLAGS=--sanitize
    DSTUDIO_AGENT_QWEN35_DIR=/path/to/ds4-qwen35 DSTUDIO_AGENT_QWEN38_DIR=/path/to/ds4-qwen38 make test-agent-patch-migration
    node tests/unit/agent_session_capability_test.mjs
    node tests/unit/frontend_behavior_test.mjs
    node tests/unit/qwen38_tool_oracle_test.mjs
    node tests/browser/ui_agent_design_playwright_test.mjs
    DSTUDIO_TEST_BROWSER=webkit node tests/browser/ui_agent_design_playwright_test.mjs

Il test di reset esegue handler e thread nativi con inferenza simulata e barriera
deterministica: annullamento, errore, successo, allocazione fallita, annullamento
tardivo, duplicati; Qwen3.8 comprende salvataggio fallito e riferimenti immagini.
ASan/UBSan coprono Agent/helper, non gli oggetti core già compilati.

## Ricevute disponibili

Percorsi relativi a `tests/.artifacts/`, ignorati da Git. Le prove iniziali
fallite sono conservate, non sostituite dai retry.

| Prova | Ricevuta | Esito / perimetro |
| --- | --- | --- |
| Reset Qwen3.6 originale | `agent-session-reset/run-g340ve` | 3/3 casi rossi attesi: riproduce il difetto |
| Reset Qwen3.8 originale | `agent-session-reset/run-NKTNJv` | 3/3 casi rossi attesi: riproduce il difetto |
| Reset Qwen3.6, patch 91 | `agent-session-reset/run-mHCmsD` | 6/6 PASS, inferenza simulata e thread nativi |
| Reset Qwen3.8, patch 91 | `agent-session-reset/run-wZAYRH` | 7/7 PASS, inferenza simulata e thread nativi |
| Applicazione delle patch | `agent-patch-migration/run-gk25s5` | 5/5 PASS, inclusi ripetizione, ripristino, drift e modifiche estranee |
| Parser e strumenti Qwen3.6 | `qwen35-agent/run-uWi4Ah` | PASS, ASan/UBSan sugli adattatori, filesystem reale |
| Parser e strumenti Qwen3.8 | `qwen38-agent/run-B3Hnd8` | PASS, ASan/UBSan sugli adattatori, filesystem reale |
| Setup Qwen3.6 ripetuto, patch 91 | `qwen35-http-dqrmY1` | PASS, CLI e HTTP reali, nessun peso o inferenza |
| Primo replay reset Qwen3.6 | `qwen35-host-live/run-KaB607` | FAIL conservato: il test non ricomponeva il testo fra gli eventi di progresso |
| Secondo replay reset Qwen3.6 | `qwen35-host-live/run-l76E5r` | FAIL conservato: il test includeva la ricevuta interna nascosta dalla UI |
| Replay Qwen3.6 con decoder corretto | `qwen35-host-live/run-rTxWZs` | 2/2 PASS: Agent e Cowork, annullamento con ricordo esatto, reset completato e lettura reale |
| Primo replay reset Qwen3.8 | `qwen38-host-live/run-uAb5iW` | FAIL conservato: prompt di 4.358 token, tutto in un blocco; reset riuscito ma nessun progresso intermedio osservabile |
| Secondo replay reset Qwen3.8 | `qwen38-host-live/run-QQtql2` | FAIL conservato: annullamento e codice corretti, ma il test includeva la riga nativa di autosave nella risposta |
| Replay Qwen3.8 con fixture e decoder corretti | `qwen38-host-live/run-pebGA0` | 2/2 PASS: Agent e Cowork, progresso parziale, annullamento con ricordo esatto, reset completo e lettura reale |

In entrambi i replay falliti, la ricevuta di annullamento era corretta e il
testo visibile conteneva esattamente il codice originario. L'oracolo corretto
ha verificato entrambe le ricevute senza modificarle; ora viene usato per
entrambi i Qwen. Le regressioni escludono l'eco della domanda, ricompongono
gli eventi frammentati e non accettano codici errati o testo aggiuntivo.
Questo non risolve il diverso fallimento sul conteggio `run-MJEBA6`.

Il fork Qwen3.8 al pin usa blocchi nativi prefill da 8.192 token e chiama il
callback di avanzamento dopo il blocco. Il primo fixture ne conteneva soltanto
4.358: non poteva qualificare un annullamento fra blocchi. Il replay successivo
usa più testo inerte nel solo workspace di test, senza cambiare chunk, contesto,
engine o deadline. Il grader condiviso ora richiede `0 < done < total`;
tutti e quattro i reset del replay Qwen3.6 già conservato rispettano anche questa
condizione più stretta. Nuove regressioni rifiutano un progresso solo finale.

Nel secondo replay Qwen3.8 il reset viene annullato dopo 8.192 dei 9.958 token
e la risposta ricorda il codice esatto. Il grader iniziale includeva però il
messaggio `saved session … (… tokens)`: la UI lo classifica come elemento di
sistema. Il decoder condiviso ora esclude quella forma terminale esatta; una
regressione esegue anche le funzioni reali `splitUserTurns`/`segmentAgent` e
verifica la separazione risposta/sistema per codice corretto ed errato.
La ricevuta fallita è conservata e verificata senza riscriverne l'esito.

Bilancio di questa campagna di reset: **sei tentativi di script**, quattro
iniziali falliti per i difetti del test/fixture sopra descritti e due replay
finali completi riusciti, ognuno con Agent e Cowork. Non è un tasso di qualità
del modello. Il decoder finale è stato riapplicato alle risposte Qwen3.6 già
conservate, senza riscriverle: i controlli semantici continuano a passare.

Altri controlli mirati: preflight **28/28**, avvio su pipe/processi **16/16**,
coda sessioni UI **34/34**, regressioni frontend e oracolo degli strumenti PASS.
Browser Chromium e WebKit PASS con motore simulato, inclusa la ricevuta di
reset fallito in Agent e Cowork. Non sono prove della vera finestra `.app`.

## Replay con modelli reali

Verifica esplicita e pesante, **un modello alla volta**:

    make tests/.build/dstudio-server-test
    node tests/live/qwen38_host_smoke.mjs /path/to/ds4-qwen35 /path/to/qwen36.gguf --qwen35 --reset-lifecycle
    node tests/live/qwen38_host_smoke.mjs /path/to/ds4-qwen38 /path/to/qwen38.gguf /path/to/ple.gguf --reset-lifecycle

Richiede pesi già presenti nello store condiviso del checkout. Verifica
strumenti/file, annullamento durante il reset, ricordo di un codice presente
soltanto nella conversazione precedente, reset completato e nuova lettura.
Usa profilo/workspace isolati e termina solo i propri processi.

Le ricevute dettagliate in tests/.artifacts/ sono locali e ignorate da Git.
Questi replay non sono benchmark di velocità, qualità held-out o prove desktop.
I fixture del reset sono volutamente diversi per esercitare le due granularità
native: Qwen3.6 aggiorna per token, Qwen3.8 per blocco. Su quest'ultimo il
contatore può non avanzare per la durata di un blocco; il test non qualifica
interruzioni all'interno di un kernel GPU né aggiorna percentuali inventate.
La vecchia prova --controls e il suo fallimento iniziale sul conteggio restano
distinti: --reset-lifecycle non li sostituisce.

Qwen3.6: nel replay finale entrambi i reset annullati conservano il contesto;
gli altri due completano e consentono la lettura del file precedente. Le attese
di reset completo sono state 314 s per Agent e 553 s per Cowork: misure di questa
singola prova di sviluppo, non benchmark o confronto di prestazioni. Il reset
rimane costoso, ma i controlli continuano a rispondere. Host e motori del test
sono stati chiusi e verificati assenti; la `.app` utente è rimasta intatta.

Qwen3.8: entrambi i modi osservano un blocco completato mentre rimane lavoro,
annullano il reset e ricordano il codice precedente senza tool. Il secondo
reset completa e permette una nuova lettura. L'autosave nativo produce le sue
ricevute; questo non sostituisce una matrice completa di ripristino dei
checkpoint su disco. Host e motori del replay finale sono stati verificati
assenti dopo la chiusura. Nessun modello del test è lasciato in esecuzione.

I gate mirati sono stati rieseguiti al termine: setup/preflight/spawn/oracoli
PASS, sintassi del runner e `git diff --check` PASS. I test Chromium/WebKit e
quelli nativi con sanitizzatori restano le prove mirate elencate sopra; non è
stato dichiarato un nuovo `check-fast` completo, un profilo prestazionale
completo o una qualificazione release della `.app`.

## Verifica del pacchetto per GitHub

Il 7 settembre il solo contenuto selezionato per il commit è stato ricostruito
in un worktree isolato: le modifiche degli altri capitoli rimaste nella copia
di lavoro non possono far passare questi controlli. Il pin main pubblicato
rimane **f4d03f6**; l'aggiornamento main/GLM, Goals, il restyling PDF e i risultati
Design non fanno parte di questo push. Sono incluse le dipendenze condivise
necessarie a installare e avviare i runtime Qwen, compreso l'adattamento del
protocollo comune nel consumer Design, non la sua integrazione con Qwen.

| Verifica ripetuta sul pacchetto | Esito e perimetro |
| --- | --- |
| Host, setup, preflight e spawn | Compilazione nativa PASS; preflight 28/28, spawn 16/16 e sessioni UI 34/34 |
| Reset e strumenti Qwen | 6/6 Qwen3.6 e 7/7 Qwen3.8; parser/tool PASS con ASan/UBSan sugli adattatori e inferenza simulata |
| Patch e build private | 39 controlli applicatore, 5 basi Agent e 9 basi web/server PASS; 8 scenari build Agent e 17 Design PASS con compilatore simulato |
| Build native condivise main/Laguna | 23 scenari PASS: compilazione, strumenti, renderer e consumer Design; nessun peso caricato |
| Avvio e controlli | 11 scenari HTTP PASS; 5 Chromium e 5 WebKit PASS, più regressioni Agent/Cowork; motori simulati |
| Catalogo, PLE e metriche | Catalogo Qwen3.6 e ciclo patch PLE Qwen3.8 PASS; serializzatori JSON/SSE reali e ciclo patch su tre sorgenti main/Laguna PASS |
| Routing backend | 16/16 PASS con compilatori/linker simulati; non qualifica CUDA, ROCm o Vulkan reali |

Oltre ai comandi precedenti, sono stati eseguiti:

    make test-unified-patch test-agent-build test-design-build-freshness
    make test-launch-control test-ui-launch test-steering
    make test-agent-native-build AGENT_MAIN_TREE=/path/to/main-f4d03f6 AGENT_LAGUNA_TREE=/path/to/laguna
    make test-runtime-patch-migration
    node tests/integration/backend_link_test.mjs /path/to/main /path/to/laguna /path/to/qwen38 /path/to/qwen35
    make test-qwen35-catalog QWEN35_DIR=/path/to/qwen35
    make test-qwen38-inspect QWEN38_DIR=/path/to/qwen38
    make test-server-metrics-patch test-main-decode-metrics METRICS_MAIN_DIR=/path/to/main-git-history LAGUNA_DIR=/path/to/laguna
    node tests/unit/quality_baseline_test.mjs

Le migrazioni richiedono le sorgenti esatte dichiarate nei rispettivi harness;
il controllo delle metriche richiede anche l'oggetto Git main `f4d03f6`.
Le ricevute di questa verifica sono locali sotto
`tests/.artifacts/qwen-publish.MZN3J1/tree/tests/.artifacts/`.
Sono conservati anche i primi tentativi falliti: fixture sorgenti incomplete,
archivio privo della cronologia Git richiesta dal test delle metriche e un
consumer Design omesso dalla prima selezione del commit. Dopo aver fornito
le fixture richieste e incluso quell'adattamento condiviso, i gate interessati
sono stati ripetuti integralmente, senza indebolire le asserzioni.

I runner di accettazione includono i due eseguibili Qwen strutturati e il nuovo
pin nel primo avvio. La verifica di pubblicazione non ha ripetuto download,
inferenza reale o avvio della `.app`: i replay reali rimangono quelli documentati
sopra. Nessun nuovo benchmark di velocità o punteggio di qualità viene dichiarato.

## Dopo la pausa

1. Q-04: vera .app, menu, cambio modello, Stop, nuova conversazione e ripresa.
   Conservare la precedente prova Chat desktop come baseline; ripetere i
   percorsi interessati sulla nuova build isolata, senza attribuire alla nuova
   patch l'intera matrice storica. Riferimento locale: P7 di `PLAN.MD`.
2. Q-05: corpus comune di 100 domande per modello, controlli numerici e indagine
   sul conteggio terminato anticipatamente. Il retry riuscito non ne dimostra
   la causa né la correzione.
   Congelare casi/oracoli prima dei run e conservare errori e timeout nel
   denominatore. Distribuzione e criteri restano in P6 del piano locale.
3. Q-06: Learn/thinking, Design, visione, nuovo Qwen27B e installer/backend.
   CUDA/Vulkan/ROCm richiedono prove distinte sull'hardware appropriato.
   Candidati e lacune concrete Metal/CUDA restano in P3 del piano locale;
   nessun nuovo peso o motore è stato scaricato per anticipare questa fase.
4. Altri capitoli del piano locale: altri motori, PDF/visione, nove
   design system, confronti Agent, profilazione, build finale e pubblicazione
   dei benchmark con Matplotlib.

Non segnare P3/P6/P7 o l'intero piano come completati per il solo passaggio
della tranche corrente. Prima di un push rivedere il worktree e i dati privati,
con l'autorizzazione prevista.
