# Orchestra Match - Specifica Completa

## 0) Scopo, obiettivi e principi

**Obiettivo prodotto**
Mettere in contatto **orchestre/direttori** con **musicisti** per coprire posizioni (servizi, concerti, tournée, sostituzioni, audizioni), con:

* candidatura semplice
* selezione efficiente
* notifiche puntuali
* affidabilità/trust (verifiche + rating moderato)

**Principi**

* **Manuale-first** (MVP): l'orchestra decide, il sistema aiuta con **filtri**. Il ranking automatico è previsto per Release 2.
* **Privacy by default**: contatti diretti solo dopo match **E** consenso `contact_sharing` attivo da entrambe le parti.
* **Rating "safe"**: double-blind, contestabile, visibile solo dopo soglia minima.
* **Auditabilità**: log eventi (offerte, cancellazioni, no-show) per gestire dispute.

---

## 1) Architettura prodotto (client + ruoli)

### Raccomandazione

* **Un solo account** con **profili multipli** e **switch ruolo** (Musicista / Orchestra member).
* **Client**:

  * **App mobile** (musicisti)
  * **Web dashboard** (consigliata per orchestre: pipeline, filtri, export, gestione team)

> Il report propone "una sola app": ok come *prodotto*. A livello pratico, la dashboard web riduce attrito lato orchestra.

### Ruoli e permessi (RBAC)

* **Musicista**
* **Orchestra Admin** (gestisce org, team, posizioni)
* **Orchestra Member** (casting manager/segretaria: gestisce posizioni/candidature)
* **Moderatore/Admin piattaforma**

> Nota: Il ruolo "Direttore" è stato rimosso per semplificare. Orchestra e Direttore sono la stessa entità (chi pubblica annunci). Un direttore può semplicemente creare una propria "orchestra" personale.

---

## 2) Oggetti principali (data model alto livello)

* **User** (account, auth, preferenze notifiche)
* **Orchestra (Organization)** + **OrchestraTeamMember**
* **MusicianProfile**
* **MediaAsset** (audio/video/link, CV PDF)
* **JobPost (Posizione/Annuncio)**
* **Application (Candidatura)**
* **Offer (Proposta)**
* **Engagement (Incarico confermato)**
* **Thread/Chat + Message**
* **Review (Recensione) + Dispute (Contestazione)**
* **Verification (Badge, step di verifica)**
* **Notification + NotificationPreference**
* **AuditLog (eventi tracciati)**
* **CalendarEvent / Availability** (disponibilità e blocchi)
* **PriorityRoster (Lista Prioritaria)** + **PriorityRosterTier (Livello)** + **PriorityRosterMember (Musicista in lista)**
* **EngagementQuota (Limite chiamate)** — tracciamento giorni di ingaggio per coppia orchestra/musicista

---

## 3) Dati da raccogliere (campi completi, con note “MVP”)

### 3.1 Musicista — profilo

**Identità & base**

* Nome, cognome (MVP)
* Foto profilo professionale (MVP)
* Email verificata (MVP), telefono (opzionale)
* Città + paese, raggio spostamenti (MVP)
* Lingue parlate (MVP)

**Professione**

* Strumento principale + varianti (es. “Clarinetto in Sib”, “Clarinetto basso”) (MVP)
* Strumenti secondari
* Ruoli: fila / prima parte / spalla / solista / concertino (MVP)
* Anni esperienza (MVP)
* Formazione: conservatorio, masterclass, accademie
* Esperienze orchestrali precedenti (orchestra/teatro, periodo, ruolo)
* Repertorio/competenze: sinfonico, operistico, contemporaneo, barocco, cameristico (+ tag) (MVP)
* Direttori con cui ha lavorato (opzionale)

**Portfolio media**

* Clip audio/video: max durata (es. 3 min), max numero (es. 5) (MVP consigliato)
* Link esterni (YouTube/SoundCloud/sito)
* CV PDF scaricabile (MVP)

**Disponibilità e preferenze**

* Calendario disponibilità (manuale MVP; sync esterna Release 2)
* Disponibilità geografica: locale/nazionale/internazionale (MVP)
* Disponibilità tournée: sì/no + durata max
* Tipo ingaggio preferito: concerto singolo/serie/stagione/sostituzione/tournée (MVP)
* Compenso minimo richiesto (range, opzionale)
* Preavviso minimo richiesto (MVP)

**Documenti “verificabili” (opzionali, con consenso)**

* Diploma/attestati
* “Posso fatturare / posso fare ricevuta” (flag operativo)
* Altri documenti (solo se davvero necessari)

> Nota GDPR/anti-discriminazione: data di nascita/nazionalità **non obbligatori** in MVP.

---

### 3.2 Orchestra/Ente — profilo organizzazione

**Identità**

* Nome, tipologia (sinfonica/camera/ensemble/coro+orch/teatro d’opera/festival) (MVP)
* Anno fondazione (opzionale)
* Sede principale + sedi frequenti prove/concerto (MVP)
* Logo/foto
* Sito web + social
* Contatti ufficiali (email/telefono) (MVP)

**Organizzazione**

* Referente audizioni/casting (nome, ruolo, contatto) (MVP)
* Dimensione organico (opzionale)
* Stagione (concerti/anno) (opzionale)
* Repertorio tipico (tag)
* Sale/teatri di riferimento (opzionale)

**Verifica (badge)**

* Step: email dominio → telefono → documenti (release successive)
* P.IVA/CF ente (opzionale, solo per “Verified”)
* Documento referente / lettera su carta intestata (solo se vuoi badge alto)

---

## 3.3 Lista Prioritaria (Priority Roster) — struttura

Ogni orchestra **deve** gestire una lista prioritaria di musicisti prima di poter distribuire un annuncio. La lista è organizzata a **livelli/tier** (Livello 1 = prima scelta, Livello 2 = seconda scelta, ecc.).

**Struttura**

* **PriorityRoster**: contenitore per orchestra, uno per strumento/ruolo (o generico)
* **PriorityRosterTier**: livello dentro il roster (1, 2, 3, …) con configurazione timer
* **PriorityRosterMember**: musicista assegnato a un tier

**Regole di business**

* Quando l'orchestra pubblica un annuncio, il sistema cerca il roster compatibile (strumento + ruolo)
* Le notifiche partono **solo** ai musicisti del **Livello 1**
* Ogni livello ha una **finestra temporale** (`WindowHours`) configurata dall'orchestra
* Allo scadere della finestra senza che la posizione sia stata coperta, il sistema notifica automaticamente il **livello successivo**
* La distribuzione procede in sequenza finché la lista non è esaurita
* **Solo dopo** l'esaurimento della lista (tutti i livelli notificati + scaduti) la posizione viene aperta alle **candidature libere** (feed pubblico)
* L'orchestra può decidere di aprire anticipatamente le candidature libere in qualsiasi momento

**Import massivo da Excel**

* L'orchestra può scaricare un **template Excel** precompilato (nome, cognome, email, strumento, livello)
* L'upload del template compilato crea/aggiorna i musicisti nella lista
* Il sistema valida le email, segnala duplicati e musicisti già presenti in piattaforma
* I musicisti non ancora registrati ricevono un **invito a iscriversi**

---

## 3.4 Limiti di Ingaggio (Engagement Quota)

Per evitare che i musicisti vengano contattati troppo frequentemente dalle stesse orchestre (fonte di lamentele nel settore), il sistema traccia i limiti di ingaggio.

**Regole di business**

* Per ogni coppia orchestra–musicista, il sistema conta i **giorni di ingaggio effettivi** in una finestra temporale mobile (default: ultimi 365 giorni)
* L'orchestra configura una **soglia massima** di giorni di ingaggio per musicista (es. 90 giorni su 365)
* Nella pipeline di selezione e nella lista prioritaria, i musicisti che si avvicinano o superano la soglia vengono evidenziati con un **indicatore visivo** (warning)
* L'orchestra può **skippare** i musicisti a rischio di sovra-utilizzo durante la distribuzione sequenziale
* Il conteggio include solo gli engagement completati o in corso (non quelli cancellati)
* La soglia è configurabile per orchestra e opzionalmente per strumento/ruolo

**Parametri configurabili**

| Parametro | Default | Descrizione |
|-----------|---------|-------------|
| `quota_window_days` | 365 | Finestra temporale mobile in giorni |
| `quota_max_engagement_days` | 90 | Giorni massimi di ingaggio nella finestra |
| `quota_warning_threshold` | 80% | Percentuale della soglia oltre cui mostrare warning |

---

## 4) Posizioni/Annunci (JobPost) — struttura completa

**Informazioni base (MVP)**

* Titolo annuncio
* Strumento richiesto + variante
* Ruolo (fila/prima parte/spalla/solista)
* Numero posizioni

**Dettagli ingaggio**

* Tipo: concerto singolo / serie / sostituzione / stagione / audizione stabile
* Date strutturate:

  * prove (una o più date)
  * concerti
  * tournée (range + tappe opzionali)
* Luogo/luoghi
* Compenso: fisso o range; lordo/netto (campo + note)
* Rimborsi: viaggio / vitto / alloggio (sì/no/parziale + note)
* Note logistiche: call time, parcheggio, strumenti forniti, ecc.

**Requisiti**

* Esperienza minima (anni o “livello”)
* Repertorio specifico
* Dress code
* Documenti richiesti (solo se indispensabili)
* Must-have vs nice-to-have (consigliato)

**Programma musicale**

* Opere/brani
* Direttore
* Solisti (se noti)
* Allegati: parti/estratti/studio

**Selezione & scadenze**

* Deadline candidatura
* Data prevista esito
* “Data limite conferma” (in caso di offerta)
* Metodo selezione: manuale / ranking suggerito / inviti diretti

**Visibilità annuncio**

* Pubblico (feed)
* Solo su invito (private call)
* Programmato (pubblica da data X)

---

## 5) Stati ufficiali (fondamentali per sviluppo/QA)

### JobPost

* Draft → **Distributing** → Published → InSelection → OfferSent → Filled/Closed → Archived

> **Distributing**: nuovo stato attivo durante la distribuzione sequenziale dalla lista prioritaria. Se l'orchestra non ha un roster compatibile, si salta direttamente a Published. Durante Distributing i musicisti dei livelli attivi possono già candidarsi e ricevere offerte.

### Application

* Submitted → Viewed → Shortlisted → Rejected
* … oppure: Offered → Accepted / Declined / **Expired**
* Withdrawn (ritirata dal musicista)

### Offer

* **Pending** → Accepted / Declined / **Countered** / Expired / **Withdrawn**

> Nota terminologia: "Pending" (non "Sent"), "Countered" (non "CounterProposed"), "Withdrawn" (non "Cancelled")

### Engagement

* Confirmed → InProgress → Completed
* Cancelled (con campo `CancelledBy`: Orchestra / Musician)

> **NoShowStatus** è un campo separato, non uno stato dell'Engagement:
> - NULL = nessun no-show
> - Reported → Contested → Confirmed / Reversed
>
> L'Engagement.Status finale dopo no-show confermato resta "Completed" con NoShowStatus="Confirmed"

---

## 6) Flussi operativi (end-to-end)

### Flusso A — Onboarding

**Musicista**

1. signup + verifica email
2. strumento/ruolo + città + disponibilità base
3. portfolio minimo (link o 1 clip)
4. preferenze notifiche

**Orchestra**

1. signup + verifica email
2. crea org + inserisce dati minimi
3. invita team (opzionale)
4. crea prima posizione

---

### Flusso B — Pubblicazione posizione e distribuzione sequenziale

Orchestra → Dashboard → Nuova posizione (wizard)

* Step 1: ruolo/strumento/date/luogo
* Step 2: compenso + rimborsi + logistica
* Step 3: requisiti + programma + allegati
* Step 4: preview + pubblica (ora o programmata)

Sistema — **distribuzione a livelli dalla lista prioritaria**:

* indicizza annuncio
* cerca il roster compatibile (strumento + ruolo) dell'orchestra
* **Fase 1 — Distribuzione prioritaria (obbligatoria se roster presente)**:
  * invia notifiche ai musicisti del **Livello 1** della lista prioritaria
  * avvia **timer** per il Livello 1 (`WindowHours` configurato nel tier)
  * i musicisti notificati possono candidarsi o rifiutare
  * allo scadere del timer: se posizione non coperta → notifica **Livello 2**, avvia nuovo timer
  * ripetere fino a esaurimento livelli
  * durante la distribuzione, i musicisti vicini alla **soglia limite di ingaggio** vengono evidenziati con warning; l'orchestra può decidere di saltarli
* **Fase 2 — Candidature aperte**:
  * solo dopo l'esaurimento della lista (o decisione manuale dell'orchestra), l'annuncio diventa visibile nel feed pubblico
  * invia notifiche a musicisti compatibili (hard match: strumento + date + area)

---

### Flusso C — Candidatura musicista

Musicista → feed/ricerca → dettaglio annuncio → “Candidati”

* messaggio di presentazione
* selezione clip (o carica estratto se richiesto)
* conferma disponibilità su tutte le date
* invio

Sistema:

* notifica orchestra
* candidatura entra in pipeline

---

### Flusso D — Gestione candidature (pipeline orchestra)

Orchestra → annuncio → lista candidati
Azioni:

* view profilo + ascolta clip + scarica CV
* tag: Preferito / Da valutare / Non idoneo
* shortlist
* chat 1:1 (abilitata dopo candidatura o invito)
* invia richiesta info (domande rapide)
* invia offerta

Extra consigliato:

* **Q&A annuncio** (domande pubbliche: utile per tutti i candidati)

---

### Flusso E — Offerta / Accettazione / Negoziazione

Orchestra → “Proponi ingaggio”

* conferma/importo finale
* date finali
* note
* scadenza risposta (es. 48h)
* allegati (parti/contratto/info)

Musicista:

* Accetta → crea Engagement + blocco calendario
* Rifiuta → motivazione opzionale
* Controproposta → negoziazione in chat (opzionale MVP)
* Timeout → offerta Expired, orchestra notificata

---

### Flusso F — Post-accettazione (Engagement)

Match confermato:

* email riepilogo dettagli
* chat attiva
* contatti diretti sbloccati (se policy attiva)
* reminder: 7 giorni prima + 1 giorno prima (configurabile)
* checklist: call time, dress code, documenti, parti

Post-evento:

* orchestra marca “Completed”
* dopo 3 giorni: richiesta recensione (finestra 3–30 giorni)

---

### Flusso G — Cancellazioni / modifiche / no-show (con tracciamento)

**Orchestra cancella**

* notifica musicista
* jobpost torna InSelection (se vuoi riaprire)
* evento registrato in AuditLog
* feedback su cancellazione (se previsto)

**Musicista si ritira**

* notifica orchestra
* riapertura automatica posizione (opzionale)
* tracciamento affidabilità (solo con regole chiare)

**No-show**

* orchestra segnala no-show → `NoShowStatus = Reported` (Engagement.Status resta "Completed")
* musicista può contestare entro X giorni → `NoShowStatus = Contested`
* moderazione decide → `NoShowStatus = Confirmed` o `Reversed`
* evento registrato in ReliabilityEvents

---

## 7) Matching e ricerca

### Hard filters (MVP)

Filtri obbligatori disponibili da subito:

* strumento/variante
* disponibilità date
* distanza/trasferta compatibile (se definita)
* eventuali requisiti minimi

### Soft scoring / ranking (Release 2)

> **Nota**: Il ranking automatico con score è previsto per Release 2. L'MVP usa solo i filtri sopra.

Fattori di scoring:

* distanza
* esperienza vs requisiti
* repertorio compatibile
* completezza e qualità portfolio (proxy)
* rating (solo dopo soglia minima)
* storico collaborazioni positive (se esiste)
* compenso in range (peso basso)

**Spiegabilità**
Mostra "Perché consigliato": 3–5 motivi (trasparenza).

### Ricerca avanzata

* Orchestre: filtri per strumento, zona, disponibilità, rating, esperienza, tag repertorio
* Musicisti: filtri per date, compenso, distanza, tipologia orchestra, repertorio

---

## 8) Notifiche (push/email) + preferenze

**Eventi principali**

* **invito da lista prioritaria** (musicista — notifica livello con scadenza timer)
* **avanzamento livello lista prioritaria** (orchestra — notifica passaggio al livello successivo)
* **apertura candidature libere** (musicisti compatibili — annuncio ora pubblico)
* **warning limite ingaggio** (orchestra — musicista vicino alla soglia)
* nuova posizione compatibile (musicista)
* nuova candidatura (orchestra)
* shortlist/view (musicista – opzionale)
* offerta inviata / accettata / rifiutata / scaduta
* nuovo messaggio chat
* reminder prove/concerto
* richiesta recensione
* contestazione recensione / esito contestazione

**Preferenze**

* canale: push / email / entrambi
* digest email: immediato / giornaliero / settimanale
* quiet hours

---

## 9) Rating & recensioni (moderato, double-blind)

### Struttura recensione (come report)

**Orchestra → Musicista**

* puntualità, preparazione, professionalità, collaborazione (1–5)
* commento (max 500)

**Musicista → Orchestra**

* organizzazione, rispetto accordi economici, condizioni lavoro, comunicazione (1–5)
* commento (max 500)

### Regole

* solo dopo Engagement completato
* finestra: 3–30 giorni
* **double-blind**: visibili solo quando entrambe inviate o scade il tempo
* rating pubblico solo dopo **min 3 recensioni**
* filtro parolacce + segnalazioni

### Dispute/Appello

* contesta entro X giorni
* ticket + motivazione
* moderazione: conferma / rimozione / oscuramento parti / rimozione entrambe
* 1 appello entro 7 giorni

### Anti-abuso

* rilevazione anomalie (tutti 1/tutti 5 ripetuti)
* possibilità di risposta pubblica (1 sola) *solo se vuoi*
* separa “rating qualità” da “affidabilità operativa” (cancellazioni/no-show)

---

## 10) Verifica profili (trust & safety)

* Badge stepwise:

  * Email verificata (base)
  * Telefono verificato (medio)
  * Orchestra “Verified” (documenti/controllo manuale) (alto)
  * Musicista “Verified” (diploma/attestato) (alto, opzionale)

---

## 11) Media (audio/video) — regole e moderazione

* supporto: upload + link esterni
* limiti: durata, numero, dimensione
* policy contenuti (privacy/copyright)
* segnalazioni + revisione admin
* (opzionale) streaming-only/watermark

---

## 12) Admin/Moderazione (requisiti minimi)

* pannello segnalazioni (utenti/media/recensioni)
* gestione dispute recensioni
* gestione verifiche (approve/reject)
* ban/sospensioni
* audit log eventi critici
* configurazione criteri matching (pesi, soglie)

### Audit Log - Eventi Obbligatori

Il sistema deve loggare obbligatoriamente i seguenti eventi:

| Categoria | Eventi |
|-----------|--------|
| **Offerte** | OfferSent, OfferAccepted, OfferDeclined, OfferExpired, OfferWithdrawn |
| **Engagement** | EngagementCreated, EngagementCancelled, EngagementCompleted |
| **No-Show** | NoShowReported, NoShowContested, NoShowConfirmed, NoShowReversed |
| **Recensioni** | ReviewCreated, ReviewDisputed, ReviewModerated |
| **Utenti** | UserBanned, UserSuspended, UserReactivated |
| **Verifiche** | VerificationApproved, VerificationRejected |
| **Lista Prioritaria** | RosterCreated, TierNotified, TierExpired, OpenCandidacyActivated |
| **Import Excel** | BulkImportStarted, BulkImportCompleted, BulkImportFailed |
| **Limiti Ingaggio** | QuotaWarningTriggered, QuotaExceeded |

### Prevenzione Overbooking

Per evitare di assegnare più musicisti delle posizioni disponibili:

```
Regola: PositionsOffered + PositionsFilled <= PositionsAvailable
```

- `PositionsOffered`: numero di offerte in stato Pending
- `PositionsFilled`: numero di engagement confermati
- Il check è transazionale: se il vincolo non è rispettato, l'invio offerta fallisce

### Soglie Affidabilità

| Parametro | Valore Default | Descrizione |
|-----------|----------------|-------------|
| `last_minute_days` | 7 | Cancellazione < X giorni = "last minute" |
| `last_minute_weight` | 2.0 | Peso penalità per cancellazione last-minute |
| `warning_threshold` | 70% | Sotto questa soglia, mostra warning nel profilo |
| `min_events_for_score` | 3 | Eventi minimi per calcolare score pubblico |

---

## 13) Privacy & GDPR (minimo indispensabile)

* consensi espliciti per: media, documenti, condivisione contatti, marketing
* export dati utente
* cancellazione account:

  * soft delete per storico engagement/recensioni (pseudonimizzato)

### Policy Contact Sharing (UNIVOCA)

I contatti (email, telefono) sono visibili **solo se**:

1. Engagement creato (offerta accettata)
2. **E** entrambe le parti hanno consenso `contact_sharing` attivo

Se una delle due condizioni non è soddisfatta:
- I contatti restano nascosti
- La comunicazione avviene solo via chat in-app

> Nota: non esiste toggle manuale. Lo sblocco è automatico al verificarsi delle condizioni.

### Time Zone

- Tutte le date sono salvate in **UTC**
- Il frontend converte in locale usando il fuso orario dell'utente (campo `Users.TimeZone`)
- Default: `Europe/Rome`

---

## 14) Roadmap consigliata (senza monetizzazione)

### MVP

* account + ruoli + switch profilo
* profili base + link/1 media + CV PDF
* **liste prioritarie a livelli** (roster per strumento/ruolo, gestione tier)
* **distribuzione sequenziale con timer** (notifiche per livello, apertura candidature aperte dopo esaurimento lista)
* **import massivo da template Excel** (download template, upload, validazione, inviti)
* **tracciamento limiti di ingaggio** (giorni per coppia orchestra/musicista, warning soglia)
* jobpost + feed + candidatura
* **filtri manuali** (NO ranking automatico)
* pipeline manuale + offerta + accetta/rifiuta + timeout
* proposta diretta (senza candidatura)
* email + push essenziali
* chat base
* verifica orchestra semplice (manuale o dominio)
* segnalazioni utente + pannello admin base

### Release 2

* **ranking suggerito + match score + spiegabilità**
* calendario + blocchi disponibilità + sync esterna
* template annunci
* inviti diretti evoluti
* upload media completo + file in chat
* liste musicisti preferiti

### Release 3

* rating pubblico double-blind + dispute
* affidabilità avanzata + regole no-show automatiche
* automazioni (auto-suggestions)

> **Nota allineamento**: Il ranking/matching è in **Release 2**, non MVP.
> L'MVP usa solo filtri manuali senza score automatico.