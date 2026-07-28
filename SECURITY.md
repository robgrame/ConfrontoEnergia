# 🔐 Segnalazione di vulnerabilità

Grazie per il tempo che dedichi a rendere più sicuro un servizio pubblico e gratuito.

## ⚠️ Non aprire una issue

Le issue di GitHub sono pubbliche: una vulnerabilità descritta lì è immediatamente
visibile a chiunque, compreso chi potrebbe sfruttarla, mentre la correzione non è ancora
pronta. Usa uno dei due canali riservati qui sotto.

## 📬 Come segnalare

| Canale | Come |
|---|---|
| 🥇 **GitHub Security Advisory** | Scheda **Security** → *Report a vulnerability*. Resta privato fino alla pubblicazione ed è il canale preferito. |
| ✉️ **Posta elettronica** | **abuse@confrontoenergia.it** |

Sono utili, quando li hai: il tipo di problema, il percorso o l'endpoint interessato,
i passi per riprodurlo, e l'impatto che ritieni possibile. Anche una segnalazione
parziale è meglio di nessuna segnalazione.

## ⏱️ Cosa aspettarti

Il progetto è portato avanti da una sola persona nel tempo libero: non c'è un SLA e non
posso garantire tempi di risposta. L'impegno è di **riscontrare entro 7 giorni** e di
tenerti aggiornato sull'avanzamento della correzione. Se lo desideri, il tuo nome viene
citato nell'advisory; altrimenti la segnalazione resta anonima.

**Non è previsto alcun compenso**: il servizio è gratuito, senza pubblicità e non
genera ricavi.

## 🎯 Cosa rientra nel perimetro

- I siti `confrontoenergia.it`, `www.confrontoenergia.it` e l'API `api.confrontoenergia.it`,
  compreso il server MCP.

Il codice non è pubblico, ma **questo non è un ostacolo alla segnalazione**: interessano i
comportamenti osservabili del servizio in esercizio, non la lettura del sorgente.

**Fuori perimetro**, perché non sono sotto il mio controllo: i servizi Azure sottostanti
(vanno segnalati al [MSRC](https://msrc.microsoft.com/report)), le librerie di terze parti
(segnalale a monte), le fonti dati ARERA, e i risultati di scanner automatici privi di un
impatto dimostrato.

## 🙅 Per favore evita

Test di carico o denial of service, invio di spam attraverso i moduli, e qualunque
accesso a dati che non siano tuoi. Se durante una prova ti imbatti in dati personali di
altri, **interrompi subito** e dimmelo: non scaricarli e non conservarli.

## 🛡️ Come è protetto il servizio

- Nessun segreto nel codice: la configurazione arriva da Azure App Configuration e i
  segreti da Key Vault, risolti con Managed Identity.
- Nessuna registrazione, nessun account, nessuna profilazione: non esiste una base dati
  di utenti da violare.
- Le bollette caricate transitano solo per l'elaborazione e vengono cancellate al termine;
  una regola di lifecycle le rimuove comunque entro 24 ore.
- Dipendenze verificate a ogni build (`dotnet list package --vulnerable`).

Per il trattamento dei dati personali vedi l'[informativa privacy](https://confrontoenergia.it/privacy.html)
o scrivi a **privacy@confrontoenergia.it**.
