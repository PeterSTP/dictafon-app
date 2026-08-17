# Diktafón & Asistent — prototyp

Jednoduchá webová appka (PWA), ktorá:
- prepisuje diktovaný text naživo (Web Speech API, slovenčina),
- v texte rozpoznáva príkazy typu „zapíš do kalendára…“ a vytvorí odkaz na pridanie
  udalosti do Google Kalendára / Outlooku, alebo `.ics` súbor pre Apple Kalendár a iné appky,
- voliteľne synchronizuje poznámky medzi telefónom a počítačom cez vlastný free Firebase projekt.

Celá appka je statická (HTML/JS/CSS), nepotrebuje žiadny vlastný server.

## 1. Ako appku spustiť a vyskúšať

**Najrýchlejšie (na počítači):** rozbaľ priečinok a dvojklikom otvor `index.html` v Chrome.
Diktovanie by malo fungovať aj takto lokálne v desktopovom Chrome.

**Pre telefón a pre spoľahlivé fungovanie mikrofónu je ale potrebné appku hosťovať cez HTTPS**
(mobilné prehliadače prístup k mikrofónu cez web bez HTTPS neumožňujú). Najjednoduchšia
bezplatná cesta bez zakladania účtu:

1. Choď na **https://app.netlify.com/drop**
2. Pretiahni tam celý priečinok `dictafon-app` (so všetkými súbormi vrátane `index.html`).
3. Netlify okamžite vygeneruje verejnú `https://...netlify.app` adresu — tú isto otvor
   v telefóne aj v počítači (napr. si ju pošli sebe SMS-kou/e-mailom, alebo si vygeneruj QR kód).
4. Na telefóne appku otvor v Chrome (Android) alebo Safari (iPhone) a cez menu prehliadača
   zvoľ „Pridať na plochu“ / „Add to Home Screen“ — appka sa potom správa ako nainštalovaná appka.

*(Alternatívy: GitHub Pages, Vercel, Cloudflare Pages — princíp je rovnaký: nahrať tieto súbory,
dostať verejnú HTTPS adresu.)*

## 2. Diktovanie — dôležité obmedzenie na iPhone

Web Speech API (priebežné rozpoznávanie reči v appke) plnohodnotne funguje najmä v **Chrome**
(Android aj Windows/Mac) a v Edge. Na **iPhone/iPad (Safari aj Chrome, ktorý tam beží na tom
istom systémovom engine)** je podpora obmedzená alebo nefunguje spoľahlivo — to je obmedzenie
Apple, nie appky.

Riešenie: na iPhone jednoducho klepni do textového poľa v appke a použi **mikrofón priamo na
systémovej klávesnici** (ikonka mikrofónu vedľa medzerníka) — to je natívne diktovanie od Apple,
funguje veľmi dobre aj po slovensky. Appka aj tak v napísanom/nadiktovanom texte nájde príkazy
pre kalendár, keď klikneš na „🔎 Nájsť príkazy v texte“.

## 3. Ako fungujú kalendárové príkazy

Appka počúva/skenuje text a hľadá frázy ako:

> „zapíš do kalendára…“, „pridaj do kalendára…“, „vytvor udalosť…“, „naplánuj…“,
> „dohodni schôdzku…“, „nastav pripomienku…“

Pokúsi sa z vety vytiahnuť dátum (dnes, zajtra, pozajtra, v pondelok…, 20.8., 20. augusta) a čas
(o 15:00, o 9, o desiatej). Výsledok sa zobrazí ako karta, kde si dátum/čas/názov **pred pridaním
môžeš skontrolovať a opraviť** — rozpoznávanie reči aj parsovanie textu nie je 100% spoľahlivé,
preto je táto kontrola dôležitá.

Z karty potom jedným klikom:
- otvoríš predvyplnenú udalosť v **Google Kalendári** (prihlásený Google účet v prehliadači),
- otvoríš predvyplnenú udalosť v **Outlook na webe** (prihlásený Microsoft účet),
- stiahneš **.ics súbor**, ktorý po otvorení pridá udalosť do **Apple Kalendára** (alebo
  ktoréhokoľvek iného kalendára, ktorý .ics vie importovať).

Toto zámerne nepoužíva priame prepojenie/API s tvojím účtom (to by vyžadovalo zakladanie
vývojárskej appky v Google/Microsoft konzole a schvaľovací proces) — namiesto toho appka len
predpripraví odkaz/súbor a posledný krok (uloženie) urobíš potvrdením v danom kalendári, presne
ako keby si to tam pridával ručne.

## 4. Synchronizácia poznámok medzi telefónom a počítačom

Appka sama osebe (bez ďalšieho nastavenia) ukladá poznámky **len lokálne v prehliadači** daného
zariadenia — telefón a počítač teda navzájom nič nevidia.

Pre živú synchronizáciu je v appke sekcia „Synchronizácia“, kde si pripojíš **vlastnú bezplatnú
Firebase databázu** (Google, cca 5 minút nastavenie, návod je aj priamo v appke):

1. `console.firebase.google.com` → nový projekt.
2. Pridaj „Web app“, skopíruj `firebaseConfig`.
3. Firestore Database → „Create database“ → „Start in test mode“.
4. Config vlož do appky na oboch zariadeniach + zadaj rovnaký „párovací kód“ (ľubovoľné heslo,
   ktoré poznáš len ty, napr. `peter-tajny-kod-2026`).

Od tej chvíle sa poznámky uložené na jednom zariadení objavia aj na druhom (naživo).

**Poznámka k bezpečnosti:** „test mode“ vo Firestore je dočasne verejne zapisovateľný (cca 30 dní),
čo je v poriadku na vyskúšanie. Pre trvalé používanie treba nastaviť Firestore Security Rules
(napr. viazané na prihlásenie), inak by teoreticky ktokoľvek so znalosťou tvojho párovacieho kódu
mohol dáta čítať/meniť. Ak appku budeš chcieť používať dlhodobo, viem pomôcť toto dotiahnuť.

## 5. Čo appka NEROBÍ (zámerne, v tejto fáze prototypu)

- Nezapisuje priamo cez API do tvojho reálneho Google/Outlook/Apple kalendára bez potvrdenia —
  posledný krok (uložiť udalosť) vždy potvrdzuješ ty v danom kalendári.
- Nie je publikovaná v App Store / Google Play ako natívna appka.
- Nemá vlastný účet/prihlasovanie — identitu zariadení rieši len párovací kód pre Firebase.

Toto všetko sa dá v ďalšom kroku doplniť, ak prototyp bude fungovať tak, ako potrebuješ — napr.
plné prepojenie s Google Kalendárom cez OAuth (aby sa udalosť zapísala úplne automaticky bez
klikania), vlastné prihlasovanie, alebo natívne appky.
