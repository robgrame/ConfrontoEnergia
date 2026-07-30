<div align="center">

# ⚡ ConfrontoEnergia

**Confronta le offerte luce e gas del mercato libero italiano — senza dare i tuoi dati a nessuno.**

[![Sito](https://img.shields.io/badge/🌐_confrontoenergia.it-online-2ea44f?style=for-the-badge)](https://confrontoenergia.it)
[![API](https://img.shields.io/badge/🔌_api.confrontoenergia.it-MCP_%2B_REST-0078D4?style=for-the-badge)](https://confrontoenergia.it/api.html)

[![.NET](https://img.shields.io/badge/.NET-10.0-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![C#](https://img.shields.io/badge/C%23-14-239120?logo=csharp&logoColor=white)](https://learn.microsoft.com/dotnet/csharp/)
[![Azure](https://img.shields.io/badge/Azure-App_Service-0078D4?logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)
[![Azure OpenAI](https://img.shields.io/badge/Azure_OpenAI-GPT-412991?logo=openai&logoColor=white)](https://azure.microsoft.com/products/ai-services/openai-service)
[![Document Intelligence](https://img.shields.io/badge/Document_Intelligence-OCR-0078D4?logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/products/ai-services/ai-document-intelligence)
[![MCP](https://img.shields.io/badge/MCP-Model_Context_Protocol-000000?logo=anthropic&logoColor=white)](https://modelcontextprotocol.io/)
[![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-7952B3?logo=bootstrap&logoColor=white)](https://getbootstrap.com/)
[![QuestPDF](https://img.shields.io/badge/QuestPDF-report-CC0000)](https://www.questpdf.com/)
[![Versione](https://img.shields.io/badge/versione-1.11.260730-0f766e)](https://confrontoenergia.it)
[![xUnit](https://img.shields.io/badge/test-724_passing-2ea44f?logo=xunit&logoColor=white)](#-test)
[![Dati](https://img.shields.io/badge/dati-ARERA_Open_Data-004990)](https://www.ilportaleofferte.it/)
[![Indipendente](https://img.shields.io/badge/100%25-indipendente_·_no_ads-e83e8c)](https://confrontoenergia.it/chi-siamo.html)

</div>

---

> 📖 **Questo repository è una vetrina, non contiene il codice.**
> Racconta com'è fatto e come ragiona ConfrontoEnergia — fonti dati, metodo di calcolo,
> architettura e scelte di privacy — perché chi usa il servizio possa capirlo prima di
> fidarsene. Il codice è sviluppato in un repository privato: le regole di calcolo, la
> metodologia e i limiti dichiarati restano però descritti qui per intero, ed è quella la
> parte che conta per giudicare i numeri che il sito mostra.
> Il servizio in esercizio è su **[confrontoenergia.it](https://confrontoenergia.it)**.

---

> 🧭 **Principio architetturale**
> **AI per estrarre, spiegare e conversare. Matematica deterministica per prezzi, consumi e classifiche.**
> Nessun numero mostrato all'utente è mai stato "generato" da un modello linguistico.

---

## 🎯 Perché esiste

ARERA pubblica già i dati e mette a disposizione un comparatore, ma il suo portale è
istituzionale e pieno di frizioni. Il valore aggiunto qui è tutto **attorno** al dato:

| 😖 Attrito sul Portale ARERA | ✅ Risposta di ConfrontoEnergia |
|---|---|
| Devi conoscere a memoria kWh/anno, fasce, potenza | 📄 Carichi le bollette → profilo auto-compilato |
| Non sa qual è la tua offerta attuale | 💸 La estrae → confronto "quanto risparmi davvero" |
| Condizioni contrattuali in legalese | ⚠️ Segnalazione automatica delle offerte **limitanti** |
| Filtri a form rigidi | 💬 Ricerca in **linguaggio naturale** |
| Difficile valutare fisso contro variabile | 📉 Scenari deterministici su consumo e PUN/PSV |
| Il contratto è difficile da confrontare | 📑 Passaporto con condizioni, scadenze ed evidenze |
| Dopo il cambio non sai cosa controllare | ✅ Verifica prima bolletta, reclami e Conciliazione |
| Nessuna lettura d'insieme dei consumi | 📈 **Resoconto annuale** con grafici e PDF |
| Chiuso ai programmi | 🤖 **Server MCP** pubblico: lo interroga anche un assistente AI |

E soprattutto: gli altri comparatori commerciali chiedono nome, email e telefono, poi
rivendono il contatto. Qui **non c'è registrazione, non c'è pubblicità, non c'è profilazione**.

## 📊 Fonte dati

File XML bulk giornalieri del **Portale Offerte** (ARERA / Acquirente Unico):

```
https://www.ilportaleofferte.it/portaleOfferte/resources/opendata/csv/offerteML/{ANNO}_{MESE}/PO_Offerte_{E|G}_MLIBERO_{YYYYMMDD}.xml
```

- `E` = energia elettrica (~3.500 offerte/giorno, ~20 MB), `G` = gas
- Namespace `http://www.acquirenteunico.it/schemas/SII_AU/OffertaRetail/01`, elemento ripetuto `offerta`
- Parsing **streaming** (`XmlReader`) per non caricare in memoria l'intero file
- Il worker salva una copia compressa del catalogo e uno **snapshot immutabile** con
  hash SHA-256, data fonte, versione del parser e licenza
- Il portale continua a usare l'ultima fotografia ufficiale valida durante le
  indisponibilità ARERA e mostra sempre la data del listino
- Il CSV scaricato manualmente dal **Portale Consumi** viene elaborato solo nel browser:
  granularità reale, buchi segnalati, nessuna interpolazione e nessun upload

## 🧮 Metodologia di calcolo

Oneri di sistema, trasporto/distribuzione e imposte sono **identici per tutte le offerte**
a parità di consumo: non spostano la classifica. Il confronto usa quindi la **spesa parte
venditore**, l'unica componente su cui i fornitori competono davvero:

```
spesaVenditoreAnnua = Σ quote fisse (€/anno) + (Σ componenti €/kWh) × consumoAnnuo
```

- Un'offerta può avere **più** componenti €/kWh (es. corrispettivo energia + mercato capacità): vengono **sommate**.
- Per le offerte **variabili** il prezzo è lo *spread* su PUN/PSV; il parametro `prezzoMateriaPrima` permette di stimare la bolletta totale indicativa.
- Le bollette **infra-annuali** (bimestrali, trimestrali) vengono annualizzate sul periodo realmente coperto, non moltiplicate a occhio.
- Ogni regola su diritti, bonus e vulnerabilità mostra la **fonte normativa** distinta
  dalla guida operativa; soglie e criteri temporali dichiarano la loro validità.

## 🧩 Cosa offre

| | Cosa | Dove |
|---|---|---|
| 🌐 | **Portale web** — confronto, upload bollette, resoconto annuale, PDF | [confrontoenergia.it](https://confrontoenergia.it) |
| 💬 | **Copilota** — ricerca guidata, spiegazioni e fonti delle offerte | [confrontoenergia.it](https://confrontoenergia.it) |
| 📉 | **Scenari** — what-if deterministico su consumi e indice PUN/PSV | [confrontoenergia.it](https://confrontoenergia.it) |
| 📑 | **Passaporto del contratto** — condizioni, scadenze, evidenze e confronto | [confrontoenergia.it](https://confrontoenergia.it/#contratto) |
| ✅ | **Verifica prima bolletta** — prezzo, quota fissa e spesa materia contro il contratto | [confrontoenergia.it](https://confrontoenergia.it/#contratto) |
| 📊 | **Import Portale Consumi** — CSV locale, granularità reale e profilo annuale | [confrontoenergia.it](https://confrontoenergia.it) |
| 🧭 | **Cambio e diritti** — ripensamento, reclamo e percorso Conciliazione | [confrontoenergia.it](https://confrontoenergia.it/#diritti) |
| 🛡️ | **Bonus e vulnerabilità** — orientamento con norme e fonti ARERA datate | [confrontoenergia.it](https://confrontoenergia.it/#tutele) |
| ⚡ | **Simulatore casa elettrica** — pompa di calore, induzione, EV e autoconsumo | [confrontoenergia.it](https://confrontoenergia.it/#casa-elettrica) |
| 🔌 | **API REST** — gli stessi calcoli, via HTTP | [api.confrontoenergia.it](https://confrontoenergia.it/api.html) |
| 🤖 | **Server MCP** — 4 strumenti per assistenti AI | `https://api.confrontoenergia.it/mcp` |
| 📰 | **Novità** — cosa decide ARERA, spiegato a chi paga la bolletta | [confrontoenergia.it/notizie.html](https://confrontoenergia.it/notizie.html) |

### 📰 Come nascono le Novità

Un servizio in background legge i **comunicati ufficiali ARERA** (atti pubblici, non articoli di
giornale: riscrivere un articolo produrrebbe un'opera derivata, e la citazione della fonte non
sana la violazione). Il modello linguistico non parafrasa il comunicato: gli viene chiesto di
**spiegare che cosa comporta per chi paga la bolletta**, senza virgolettati.

Tre presidi, perché una notizia sbagliata vale meno di nessuna notizia:

1. **Nessuna cifra inventata** — ogni numero prodotto dal modello deve comparire nel comunicato
   di partenza, confrontato dopo normalizzazione dei separatori. Se non torna, la notizia viene
   **scartata**, non corretta a mano.
2. **Nulla si pubblica da solo** — ogni notizia nasce come **bozza**. La pubblicazione passa da
   un endpoint amministrativo protetto da chiave (confronto a tempo costante, *fail closed*: senza
   chiave configurata il gruppo non viene nemmeno mappato). Una bozza risponde con un redirect
   all'elenco, non con un 403, per non confermarne l'esistenza.
3. **La fonte è sempre in chiaro** — link al comunicato originale, `rel="nofollow"` e dichiarazione
   di non affiliazione con l'Autorità.

### 🤖 Strumenti MCP

| Strumento | Cosa fa |
|---|---|
| `stato_catalogo` | Data del listino ARERA in uso e quante offerte contiene |
| `confronta_offerte` | Classifica luce/gas per consumo annuo, con filtri (fisso/variabile, limitanti, top *N*) |
| `calcola_spesa` | Spesa annua materia energia di **una** offerta specifica |
| `dettaglio_offerta` | Struttura di prezzo completa: tutte le componenti, una per una |

Accesso con chiave via API Management (quote e rate limit). Configurazione client in
[`api.html`](https://confrontoenergia.it/api.html).

## 🔌 Endpoint REST

| Metodo | Rotta | Descrizione |
|---|---|---|
| `GET` | `/health` | Stato del servizio |
| `GET` | `/api/stato` | Offerte in cache e data del listino |
| `GET` | `/api/capabilities` | Se OCR e narrativa AI sono disponibili |
| `POST` | `/api/confronto` | 🧮 Confronto strutturato (core deterministico) |
| `POST` | `/api/scenari` | 📉 What-if deterministico su consumi e PUN/PSV |
| `POST` | `/api/chiedi` | 💬 Ricerca in linguaggio naturale |
| `POST` | `/api/consulenza` | 🧭 Lettura ragionata del profilo di consumo |
| `POST` | `/api/profilo-da-bolletta` | Estrazione profilo da testo bolletta |
| `POST` | `/api/bolletta-upload` · `/api/bollette-upload` | 📄 OCR PDF/foto → stima consumo annuo |
| `POST` | `/api/bollette-testo` | Come sopra, ma da testo già estratto |
| `POST` | `/api/report-upload` | 📈 Resoconto annuale (sincrono) |
| `POST` | `/api/report-async` | ⏳ Resoconto in background → id lavoro |
| `GET` | `/api/report-stato/{id}` | Stato del lavoro asincrono |
| `POST` | `/api/report-testo` · `/api/report-manuale` | Resoconto da testo o da dati inseriti a mano |
| `POST` | `/api/report-pdf` | 🖨️ Resoconto in PDF (QuestPDF) |
| `POST` | `/api/contratto/analizza-testo` · `/api/contratto/analizza-upload` | 📑 Passaporto del contratto con evidenze |
| `POST` | `/api/contratto/confronta` | 🔎 Confronto del contratto confermato con il catalogo |
| `POST` | `/api/contratto/verifica-bolletta-testo` · `/api/contratto/verifica-bolletta-upload` | ✅ Prima bolletta contro contratto |
| `GET` | `/api/notizie` · `/api/notizie/{slug}` | 📰 Novità pubblicate |
| `GET` | `/notizie/{slug}` | Pagina della notizia (resa lato server, con JSON-LD) |
| `GET` `POST` | `/api/admin/notizie/…` | 🔒 Bozze, pubblicazione, scarto (header `X-Admin-Key`) |
| `GET` | `/robots.txt` · `/sitemap.xml` | 🔍 SEO |

### 💻 Esempi

```bash
# 🧮 Confronto strutturato
curl -X POST https://api.confrontoenergia.it/api/confronto \
  -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: $KEY" \
  -d '{"commodity":"Elettrico","consumoAnnuo":2700,"tipoOfferta":"fisso","escludiLimitanti":true,"top":5}'

# 💬 Linguaggio naturale
curl -X POST https://api.confrontoenergia.it/api/chiedi \
  -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: $KEY" \
  -d '{"domanda":"luce a prezzo fisso senza vincoli, le migliori 3","consumoAnnuo":2700}'
```

## 🏗️ Architettura Azure

Il portale **non ha risorse proprie oltre alle due web app**: riusa l'infrastruttura
condivisa, perché è un servizio gratuito e senza SLA e una copia dedicata di gateway,
vault e configurazione sarebbe un costo fisso senza alcun beneficio.

| Risorsa | SKU / Tipo | Ruolo |
|---|---|---|
| 🖥️ App Service (Linux) | .NET 10 | Portale + API + MCP |
| 🧪 Deployment slot | Beta | Build, test e smoke automatici delle feature |
| ⚙️ App Service (Linux) | .NET 10 | Consumatore della coda dei resoconti |
| 📦 App Service Plan | P0v3 Linux | Calcolo condiviso con gli altri portali |
| 🚪 API Management | Developer | Chiavi, quote e dominio `api.confrontoenergia.it` |
| 🎛️ App Configuration | Standard | Configurazione, etichetta `ConfrontoEnergia` |
| 🔐 Key Vault | RBAC | Segreti e certificati |
| 💾 Storage | StorageV2 | Contenitori bollette e resoconti, coda dei lavori |
| 🌍 DNS | zona `confrontoenergia.it` | Record del sito e della posta |
| 🧠 AI | Azure AI Services | Document Intelligence (OCR) + OpenAI (narrativa) |

> I nomi delle risorse e gli identificativi di sottoscrizione sono volutamente omessi:
> l'infrastruttura è condivisa con altri portali e pubblicarne i nomi non aggiunge nulla
> a chi legge.

Regioni: applicazione, archiviazione e OCR in **Italia Settentrionale**. Il modello
linguistico usa una distribuzione *DataZoneStandard* dell'Unione Europea, non *Global*:
le distribuzioni Global instradano l'inferenza in qualunque regione Azure del mondo, e
per un servizio che riceve bollette non era accettabile.

### 🔒 Principi

- **Nessun segreto nel codice o nelle app settings.** L'unica impostazione è
  `AZURE_APPCONFIG_ENDPOINT`; il resto arriva da App Configuration, con i segreti
  risolti da Key Vault tramite *Key Vault references*.
- **Managed Identity + RBAC** per ogni accesso (App Configuration Data Reader,
  Key Vault Secrets User, Cognitive Services User, Storage Blob/Queue Data Contributor).
  Nessuna chiave condivisa: `disableLocalAuth` sulla risorsa AI.
- `DefaultAzureCredential` lato codice: Managed Identity in Azure, credenziali
  sviluppatore in locale.
- 🔁 I certificati TLS di `api.confrontoenergia.it` sono rinnovati da un **runbook
  Azure Automation** (Posh-ACME + DNS-01 su Azure DNS); i domini del sito usano i
  certificati gestiti di App Service.

### ⏳ Elaborazione asincrona

L'upload multiplo di bollette può superare il timeout HTTP, quindi il percorso è a coda:

```
POST /api/report-async → blob ce-bollette → coda ce-report-jobs → Worker → blob ce-report → GET /api/report-stato/{id}
```

Le bollette transitano **solo** per l'elaborazione e vengono cancellate al termine; una
regola di lifecycle le rimuove comunque **entro 24 ore**, perché la cancellazione
applicativa non deve essere l'unica garanzia su dati di questo tipo.

## 🧪 Test

**724 test automatici**, tutti verdi a ogni modifica, più test Node per i moduli
browser-only. Non coprono solo il calcolo:
presidiano anche le promesse fatte all'utente — che ogni pagina dichiari indipendenza e
assenza di pubblicità, che nessuna pagina serva script di tracciamento, che il testo del
piè di pagina resti **leggibile** (contrasto WCAG calcolato dai colori dichiarati nel CSS,
non una verifica di stringa) e che il vecchio marchio non ricompaia da nessuna parte.

Alcuni esempi di ciò che viene verificato a ogni rilascio:

- una bolletta trimestrale viene **annualizzata sul periodo realmente coperto**, non
  moltiplicata a occhio;
- un'offerta priva di prezzo energia utilizzabile **non entra in classifica**, altrimenti
  le sole quote fisse la porterebbero in cima falsando tutto;
- un'offerta riservata a certe zone non viene proposta a chi non può sottoscriverla;
- una notizia che contiene una cifra assente dal comunicato ARERA di partenza viene
  **scartata**, non corretta a mano.

## 🔐 Privacy

- 🚫 **Nessuna registrazione**: non chiediamo nome, email, telefono o indirizzo.
- 🚫 **Nessuna pubblicità, nessuna profilazione, nessun analytics nel browser**: nelle
  pagine non c'è alcuno script di tracciamento. Lato server c'è solo diagnostica tecnica
  (Application Insights): errori e tempi di risposta, senza cookie e senza ricostruire
  la navigazione.
- ⏱️ **Bollette caricate**: usate solo per produrre il tuo resoconto, poi cancellate;
  al massimo 24 ore in un'area di transito cifrata.
- 📑 **Contratto e prima bolletta**: elaborati nella singola richiesta; le evidenze
  tornano al browser e non diventano un archivio personale.
- 🧭 **Portale Consumi, reclami, bonus e simulatore casa elettrica**: elaborazione
  interamente nel browser, senza invio al server o persistenza locale.
- 🤝 **Mai condivise**: nessun dato va a fornitori di energia, intermediari o broker di contatti.
- ✍️ Chi preferisce non caricare nulla può **inserire i dati a mano** e ottenere lo stesso risultato.

Testo integrale: [Informativa privacy](https://confrontoenergia.it/privacy.html) ·
[Cookie policy](https://confrontoenergia.it/cookie.html)

## 📁 Com'è organizzata la soluzione

Due applicazioni .NET e una libreria di dominio condivisa. Lo schema serve a mostrare
**dove vive ciascuna responsabilità**: il calcolo della spesa, per esempio, sta in un
unico punto usato sia dal portale sia dall'API sia dagli strumenti MCP, così i numeri non
possono divergere fra i tre.

```
ConfrontoEnergia.Core/              🧠 logica di dominio, condivisa fra API e worker
  Modelli/      Offerta, ComponentePrezzo, ZoneItalia (tabella ISTAT), AreraCodici
  Servizi/      ClienteOpenDataArera        download + lettura in streaming
                ArchivioOfferte             cache giornaliera del listino
                PersistenzaCatalogo         copia compressa: i riavvii non riscaricano 20 MB
                CalcolatoreCosti            calcolo deterministico della spesa
                ServizioConfronto           filtri, vincoli territoriali e classifica
                AnalizzatoreLinguaggioNaturale   testo libero → filtri
                EstrattoreBollette          profilo di consumo dalla bolletta
                ServizioReport / ServizioReportPdf   resoconto annuale
                ConsulenteEnergetico / AnalizzatoreConsulenza   lettura ragionata
                ServizioNarrativa           narrativa AI
  Contratti/    EstrattoreContratto, ServizioPassaportoContratto,
                VerificatorePrimaBolletta   evidenze e controlli deterministici
  Elaborazione/ ArchivioLavori, coda e stato dell'elaborazione asincrona
  Notizie/      FonteArera                  lettura dei comunicati ufficiali
                GeneratoreNotizie           spiegazione AI + verifica delle cifre
                ArchivioNotizie             bozze e pubblicate su blob

ConfrontoEnergia.Api/               🌐 portale e API pubblica
  StrumentiMcp/ StrumentiConfronto, StrumentiCatalogo   strumenti del server MCP
  EndpointContratto  passaporto, confronto e verifica prima bolletta
  EndpointNotizie   rotte pubbliche e amministrative delle Novità
  Pagine/       modelli HTML resi lato server (fuori da wwwroot, non indicizzabili)
  PagineSeo     sostituzione dei segnaposto nelle pagine statiche
  Program       endpoint Minimal API
  wwwroot/      portale statico e moduli locali (Portale Consumi, diritti,
                tutele e simulatore casa elettrica)

ConfrontoEnergia.Worker/            ⚙️ coda degli upload multipli + monitoraggio ARERA
```

## 🏷️ Perché i nomi sono in italiano

I nomi dei tipi sono in italiano come il dominio che descrivono: il calcolo di una
bolletta si discute con chi la riceve, e un nome che coincide con la parola usata nella
conversazione riduce il numero di traduzioni mentali in cui si annidano gli errori.

## 🔐 Segnalazione di vulnerabilità

Non aprire una issue pubblica: usa la scheda **Security → Report a vulnerability**
oppure scrivi a **abuse@confrontoenergia.it**. Dettagli in [SECURITY.md](SECURITY.md).

## ⚖️ Codice, licenza e riuso

Il codice **non è pubblicato**: è sviluppato in un repository privato. La scelta non
cambia nulla per chi usa il servizio, perché ciò che permette di giudicare i numeri non è
la lettura del sorgente ma il **metodo**, ed è descritto qui per intero: da dove vengono i
dati, che cosa entra nel conto e che cosa ne resta fuori, quali offerte vengono escluse e
perché, che cosa fa l'AI e — soprattutto — che cosa **non** le è permesso fare.

Il progetto non è rilasciato sotto una licenza open source: valgono i termini di legge sul
diritto d'autore, **tutti i diritti riservati**. Copia, modifica, redistribuzione e riuso
richiedono un accordo esplicito. Se vuoi riusarne una parte o collaborare, scrivi a
**info@confrontoenergia.it**: la richiesta è benvenuta.

Le librerie di terze parti restano soggette alle rispettive licenze.

## 🙋 Domande, errori, suggerimenti

Se un numero non ti torna, se un'offerta ti sembra classificata male o se il sito si
comporta in modo strano, **apri una issue qui**: è il posto giusto e la segnalazione è
utile anche senza il codice sotto gli occhi. Per le vulnerabilità usa invece i canali
riservati descritti in [SECURITY.md](SECURITY.md).

---

<div align="center">

**Creato con il ❤️ da un cittadino per i cittadini**
Servizio 100% indipendente · non affiliato ad ARERA né ad alcun fornitore di energia

</div>
