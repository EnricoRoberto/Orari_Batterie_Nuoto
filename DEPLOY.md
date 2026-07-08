# Deploy

## Opzione A — CI/CD da GitHub (deploy automatico ad ogni push)

1. Crea repo GitHub, push del contenuto di questa cartella (`deploy/`) come root del repo.
   ```
   cd deploy
   git init
   git add .
   git commit -m "init"
   git branch -M main
   git remote add origin <URL_REPO>
   git push -u origin main
   ```

2. Genera un service account per il CI:
   - Firebase Console → ⚙️ Project settings → **Service accounts** → **Generate new private key** → scarica il JSON.
   - Il service account di default ("Firebase Admin SDK") ha già i permessi necessari (Hosting Admin + Realtime Database Admin).

3. Su GitHub: repo → **Settings → Secrets and variables → Actions → New repository secret**
   - Nome: `FIREBASE_SERVICE_ACCOUNT`
   - Valore: incolla il contenuto **completo** del file JSON scaricato al punto 2.

4. Push su `main` → il workflow `.github/workflows/deploy.yml` esegue `firebase deploy` (hosting + database rules) in automatico. Log su GitHub → tab **Actions**.

Da qui in poi: modifichi `public/index.html` o `database.rules.json`, fai commit+push su `main`, il deploy parte da solo.

## Opzione B — deploy manuale da terminale

```
npm install -g firebase-tools
cd deploy
firebase login
firebase deploy
```

Al termine, l'URL sarà tipo:
`https://swimstarttime.web.app` (o `.firebaseapp.com`)


## Cosa fa `firebase deploy`
- Pubblica `public/index.html` su Firebase Hosting (URL pubblico, raggiungibile da chiunque).
- Applica `database.rules.json` al Realtime Database esistente (`swimstarttime-default-rtdb`), abilitando lettura/scrittura pubblica sui nodi `session` e `monitorati`.

## Note sicurezza
Le regole DB sono aperte (`.read`/`.write: true`) perché l'app non ha login: chiunque abbia l'URL può caricare un nuovo PDF (sovrascrivendo la sessione corrente) o modificare gli orari. Va bene per un uso interno/condiviso con lo staff gara; se in futuro serve limitare l'accesso, si può aggiungere un semplice PIN condiviso o l'auth anonima di Firebase.

## Aggiornare l'app in futuro
Basta modificare `public/index.html` e rilanciare `firebase deploy` (o solo `firebase deploy --only hosting` se non tocchi le regole DB).

## Struttura dati Realtime Database

```
/session
  /meta
    fileName, uploadedAt, corsie, staccoBatteria, staccoSpecialita
  /meeting
    [ { parte, data, oraInizioOriginale, eventi: [ { specialita, batterie: [ { numero, totale, orario (ISO), manuale (bool), atleti: [...] } ] } ] } ]
  /lastEdit
    { tipo, dettaglio, timestamp, operatore }

/monitorati
  { "NOME_COGNOME": "Nome Cognome", ... }
```

Non serve creare nulla a mano: la prima volta che qualcuno carica un PDF, l'app scrive automaticamente questa struttura.
