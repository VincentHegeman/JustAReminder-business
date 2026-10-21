# JustAReminder Business — bouwpakket v1.0
**Datum:** 21 oktober 2026  
**Status:** werkende interactieve demo + Supabase-startimplementatie; **nog niet productierijp**.

## Inhoud

| Bestand | Doel |
|---|---|
| `business-dashboard.html` | Business-dashboard voor computer, iPad en telefoon: taken, expliciete herinneringen, agenda, team, activiteit, instellingen. |
| `config.js` | Standaard **demoMode: true**. Zet pas op live nadat SQL en beveiliging gecontroleerd zijn. |
| `config.example.js` | Lege/veilige configuratiekopie. |
| `01_business_database.sql` | Supabase-schema, Row Level Security, audit en gecontroleerde team-RPC's. Nieuwe `jar_business_*`-tabellen. |
| `02_controlequeries.sql` | Alleen-lezen controles op installatie en policies. |
| `PROJECTSAMENVATTING.md` | Samenvatting van relevante afspraken uit beschikbare JustAReminder-chats, plus open punten. |
| `TECHNISCHE_GRENZEN_EN_VOLGENDE_STAPPEN.md` | Wat al werkt, wat expres nog niet is gekoppeld en testvolgorde. |

## 1. Direct de demo openen — zonder Supabase
Open `business-dashboard.html` in een browser terwijl `config.js` in **dezelfde map** staat. Bij `demoMode: true` verschijnen voorbeeldtaken en fictieve collega's. Je kunt taken toevoegen, wijzigen, verwijderen, herinneringen plannen, een team bekijken, voorkeuren aanpassen en door de agenda bladeren. De demo gebruikt **uitsluitend geheugen in de browser**: na vernieuwen gaan wijzigingen verloren. Het dashboard kan eveneens als statische pagina op Vercel worden gehost.

## 2. Database in testomgeving installeren
1. Maak een **back-up** van je bestaande Supabase-project en probeer dit bij voorkeur eerst in een los testproject.
2. Open de **Supabase SQL Editor**, maak een nieuwe query en plak **het gehele** bestand `01_business_database.sql` erin. Voer één keer uit.
3. Controleer de aanwezigheid van de `jar_business_*`-tabellen en RLS-policies met `02_controlequeries.sql`.
4. Gebruik de gewone Supabase Authentication (e-mail/wachtwoord). E-mailbevestiging wordt gestuurd als dit in je Supabase-project is ingeschakeld.
5. Controleer in Authentication > URL Configuration dat je eigen domein/redirect-URL klopt.

### BELANGRIJK
Deze SQL **wijzigt geen Privé-, Family- of Mollie-tabellen**, geen betaalprijzen en geen live-betaalvlaggen. Bestaande rechten en abonnementen worden niet automatisch gekoppeld. Een nieuwe Business-werkruimte heeft bewust `billing_status='not_configured'` en `plan_code='business_unconfigured'`.

## 3. Overschakelen naar echte accounts en database
Open `config.js` in een teksteditor en verander:

```javascript
window.JAR_BUSINESS_CONFIG = {
  demoMode: false,
  supabaseUrl: "https://ECHTE-PROJECT-URL.supabase.co",
  supabaseAnonKey: "JOUW_PUBLIEKE_SUPABASE_ANON_OF_PUBLISHABLE_KEY"
};
```

Gebruik de **publieke** publishable/anon-key uit je project, **nooit** `service_role`, een geheime Supabase-sleutel, Mollie API key, webhooksecret of andere administratieve sleutel. Deze bestanden worden openbaar zodra je ze online zet. Kopieer geen authentieke klantdata naar de demo.

Zet `business-dashboard.html` en `config.js` op je statische hosting onder dezelfde map (bijvoorbeeld `public/business/` met dashboard hernoemd naar `index.html`, zodat `https://justareminder.nl/business/` werkt). Bewaar Privé en Family op hun bestaande routes. Controleer Vercel-rewrites als je project nu al een aangepaste routing heeft.

## 4. Eerste Business-gebruiker testen
1. Maak een aparte testgebruiker aan via Business > Registreren (of log in met een bestaande Supabase Auth-gebruiker).
2. Klik **+ Bedrijf** en maak een fictieve bedrijfswerkruimte aan.
3. Voeg een taak toe zonder herinneringstaal: dit moet `kind='task'` opleveren.
4. Klik op **Herinneringen** en voeg een herinnering met een datum/tijd toe.
5. Open **Team**: genereer als eigenaar of manager een eenmalige uitnodigingscode.
6. Gebruik een tweede **apart ingelogd testaccount** om met de code aan te sluiten.
7. Controleer rolwijziging, taaktoewijzing, privacy tussen twee afzonderlijke bedrijven en zichtbaarheid op desktop/mobiel.
8. Controleer dat een medewerker **geen** onbevoegde leden kan toevoegen of de abonnementsgegevens kan wijzigen.

## 5. Productiecheck: eerst afronden
- **Servergeplande meldingen ontbreken nog.** De instelling 'vooraf waarschuwen' staat in de database, maar verstuurt nog geen push, e-mail of sms. Daardoor is dit nog geen volledige live-herinneringendienst.
- **Business-betalingen zijn niet aangesloten.** Er is geen Business-prijs gekozen in de beschikbare projectcontext. Er wordt geen Mollie-checkout gestart en geen bestaand Privé/Family-abonnement omgezet.
- **RLS/RPC moeten met twee of meer echte testaccounts worden getest.** De HTML is lokaal in demo-stand getest, maar er is in deze sessie geen toegang geweest tot jouw actieve Supabase- of Mollie-omgeving.
- **De slimme zinnenherkenning is bewust een offline, regelgebaseerde eerste implementatie.** Hij onthoudt niets en heeft geen AI API-kosten, maar is nog niet betrouwbaar voor alle talen, grammatica's en complexe herhalingen.
- **Privacy/compliance:** configureer verwerkersafspraken, toegangsbeperking, back-ups, logretentie, toegestane tijdzones, uitnodigingsafhandeling en EU-privacyregels vóór commerciële uitrol.
- **PWA/app-installatie:** dit pakket bevat geen nieuwe Android-APK, service worker of manifest. Koppel het later veilig aan jullie bestaande PWA na test op Android, desktop en iPad.

## Wat ik wel en niet heb kunnen meenemen
Ik heb de in het project **beschikbare gespreksinformatie** en het bestand `Stap 12 proefbetaling.txt` verwerkt. De volledige onbewerkte bron van alle vroegere dashboardbestanden en alle SQL-migraties was in deze sessie **niet beschikbaar**. Daarom is dit een nieuwe, afgebakende Business-module en **geen gegarandeerde drop-in wijziging** van jullie huidige webapp. Voer dit niet blind in een live omgeving uit.
