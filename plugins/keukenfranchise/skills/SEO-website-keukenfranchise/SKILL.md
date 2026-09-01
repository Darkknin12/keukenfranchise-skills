---
name: SEO-website-keukenfranchise
description: De SEO-werkwijze voor elke website die met website-bouwen-keukenfranchise gemaakt wordt - de on-page-lat per pagina, en de volledige SEO- en concurrentieanalyse met Ahrefs. Gebruik deze skill zodra de pagina's van een site vorm krijgen, altijd vóór de site live gaat, en wanneer iemand vraagt om een site beter vindbaar te maken of om een SEO- of concurrentieanalyse.
---

# On-page SEO — elke pagina langs deze lat

Je werkt voor iemand die niet technisch is en die "vindbaar in Google" wil
zonder te weten wat daarvoor nodig is. Alles hieronder is on-page: het staat
volledig onder jouw controle, kost niets, en hoort dus gewoon te gebeuren —
de gebruiker hoeft er niet om te vragen.

## Eerst: waarover moet deze site gevonden worden?

Vraag de gebruiker in gewone taal: **"Als iemand jou in Google zoekt, wat
typt diegene dan in?"** Vraag door tot je per pagina één hoofdonderwerp hebt
(bijvoorbeeld "cateraar Breda", "bruiloftscatering", "bedrijfslunch
bestellen"). Dat is de snelle versie; het echte zoekwoordenonderzoek met
Ahrefs-cijfers staat verderop. De regel blijft in beide gevallen dezelfde:
**één onderwerp per pagina.** Twee onderwerpen op één pagina concurreren met
elkaar; maak er twee pagina's van.

## De lat per pagina

Loop dit voor élke pagina langs, niet alleen de homepage:

- **Titel (`<title>`)**: uniek per pagina, maximaal ±60 tekens, het
  hoofdonderwerp vooraan, de bedrijfsnaam achteraan. "Bruiloftscatering in
  Breda | Van der Berg Catering" — niet "Home" en niet twee keer dezelfde
  titel op de site.
- **Metabeschrijving**: uniek, ±150 tekens, geschreven als reden om te
  klikken — met het onderwerp erin en iets concreets ("Vraag binnen één dag
  een offerte aan"). Dit is advertentietekst, geen samenvatting.
- **Eén `<h1>` per pagina**, met het hoofdonderwerp erin, en daarna een
  logische kopjesvolgorde (`h2` onder `h1`, `h3` onder `h2` — nooit een
  niveau overslaan omdat het er mooier uitziet; grootte regel je met CSS).
- **Leesbare adressen**: kort, kleine letters, koppeltekens, het onderwerp
  erin — `/bruiloftscatering`, niet `/pagina2` of `/diensten?id=3`. Eenmaal
  live is een adres een belofte: verander je hem later, dan hoort er een
  301-doorverwijzing bij.
- **Echte tekst, voldoende tekst.** Een pagina die gevonden moet worden
  heeft minimaal een paar honderd woorden nodig die de vraag van de bezoeker
  echt beantwoorden. Tekst in afbeeldingen bestaat voor Google niet.
- **Afbeeldingen**: beschrijvende bestandsnamen (`bruidstaart-catering.webp`,
  niet `IMG_4032.webp`) en een alt-tekst die zegt wat er te zien is. De
  alt-tekst is er voor wie het beeld niet ziet — geen zoekwoordenlijstje.
- **Interne links met betekenis**: verwijs tussen pagina's met een
  beschrijvende linktekst ("bekijk onze bruiloftscatering"), nooit "klik
  hier". Elke pagina moet vanaf de homepage in maximaal twee klikken
  bereikbaar zijn.
- **Structured data (JSON-LD)**: op zijn minst `LocalBusiness` (of
  `Organization`) met naam, adres, telefoonnummer en openingstijden op de
  contactpagina. Heeft de site een veelgestelde-vragenblok, zet er dan
  `FAQPage`-markup op. Controleer het resultaat met de Rich Results Test
  van Google.
- **Social preview**: `og:title`, `og:description` en een `og:image` van
  1200×630 per pagina, plus de Twitter-kaartvarianten. Een gedeelde link
  zonder afbeelding wordt aantoonbaar minder aangeklikt.
- **Canonical** op elke pagina naar zichzelf, en één schrijfwijze van de site
  (met of zonder `www` — kies er één en verwijs de ander door).

## De lat voor de site als geheel

- **`sitemap.xml` en `robots.txt`**, statisch meegegenereerd. De sitemap
  bevat alleen pagina's die er echt toe doen.
- **Snelheid is een rankingfactor.** De statische opzet van
  website-bouwen-keukenfranchise is al snel; verpest dat niet met zware
  afbeeldingen of onnodige scripts. Controleer met Lighthouse (in Chrome:
  F12 → Lighthouse) en neem de meting serieus: onder de ~90 voor Performance
  is er echt iets te verbeteren.
- **Mobiel is de maatstaf** — Google beoordeelt de mobiele weergave, niet de
  desktop.
- **Lokaal vindbaar**: heeft het bedrijf een fysieke locatie, adviseer de
  gebruiker dan (in gewone taal) een gratis Google-bedrijfsprofiel aan te
  maken op business.google.com, met exact dezelfde naam, adres en
  telefoonnummer als op de site. Dat is voor lokale zoekopdrachten
  belangrijker dan al het bovenstaande — maar het is een account van de
  gebruiker, dus jij begeleidt en hij klikt.
- **Na livegang**: laat de gebruiker de site aanmelden bij Google Search
  Console (search.google.com/search-console, verificatie kan via een
  DNS-record dat jij in Terraform zet) en dien de sitemap daar in. Vertel
  erbij dat resultaat weken tot maanden duurt — beloof nooit een positie.

## Ahrefs: de meetdata achter de keuzes

Voor een echte analyse heb je Ahrefs-data nodig — zonder meting is elk
SEO-advies een mening. Zet dat zo op:

**De koppeling (eenmalig per laptop).** Ahrefs heeft een officiële
MCP-koppeling waarmee jij rechtstreeks in hun data kunt kijken. Vereist een
Ahrefs-abonnement (Lite of hoger) — vraag de gebruiker éérst of de zaak dat
heeft, want zonder abonnement heeft de koppeling geen zin. Toevoegen:

```
claude mcp add ahrefs --transport http https://api.ahrefs.com/mcp/mcp
```

Bij het eerste gebruik opent de browser een Ahrefs-inlogpagina; dat inloggen
is aan de gebruiker (het is zíjn account), daarna onthoudt de koppeling het.
Werkt de koppeling niet of is er geen abonnement op deze laptop, dan is het
alternatief simpel: vraag de gebruiker de rapporten als CSV te exporteren uit
Ahrefs zelf (elke tabel daar heeft een exportknop) en aan jou te geven, of
laat hem schermafbeeldingen plakken. Zeg er dan bij wélk rapport je nodig
hebt — "Site Explorer → Organic keywords voor domein X" — zodat hij niet
hoeft te zoeken.

**Wat je de gebruiker altijd eerst vraagt**, in gewone taal en één vraag
tegelijk:

1. Het eigen domein (of dat er nog geen is — dan is de analyse alleen de
   concurrentiekant).
2. Wie hij zelf als concurrenten ziet: "welke drie bedrijven komen tegen bij
   klanten, of staan in Google waar jij wilt staan?" Vul dat daarna aan met
   wat Ahrefs als organische concurrenten meet — die twee lijsten verschillen
   vaak, en dat verschil is zelf al een bevinding.
3. Waar de klanten vandaan komen (heel Nederland, één regio, ook België?) —
   dat bepaalt of lokale zoektermen zwaarder wegen.
4. Wat een klant wáárd is, grofweg. Tien zoekers per maand op "keuken op maat
   Breda" is veel als één keuken duizenden euro's oplevert; hetzelfde getal
   is niets voor een webshop met marges van een tientje.

## Zoekwoordenonderzoek: jij levert de woorden, Ahrefs de cijfers

De taakverdeling is vast: **jij bedenkt de kandidaat-zoekwoorden, Ahrefs
levert alleen de cijfers erbij.** Vraag de gebruiker dus nooit om zelf
zoekwoorden te verzinnen — hij weet wat zijn bedrijf doet, jij weet hoe
mensen zoeken. Zo loopt het:

**Stap 1 — jij schrijft de kandidatenlijst.** Op basis van wat het bedrijf
doet, de regio en de vragen die klanten stellen, bedenk je zelf de
kandidaten: per dienst de kale term, de varianten en synoniemen, de
plaats-combinaties, de koopgerichte vormen ("offerte", "prijs", "kosten",
"laten maken") en de vraagvormen ("wat kost", "hoe lang duurt").

**Stap 2 — groepeer ze per onderwerp, klaar om te plakken.** Eén groep per
pagina-kandidaat, en de zoekwoorden binnen een groep als één regel met
komma's — zodat de gebruiker (of jij, via de koppeling) ze in één keer in
Ahrefs plakt in plaats van stuk voor stuk. Zeg er per groep bij wélk
rapport van Keywords Explorer erop moet, en waarom:

- **Overview**: altijd, voor élke groep — plak de hele regel en je hebt per
  zoekwoord volume, moeilijkheid en intentie terug. Dit is de basisdata.
- **Matching terms**: alleen op de één à twee kernwoorden van een groep —
  vindt alle langere varianten die het kernwoord bevatten en die jij niet
  bedacht hebt.
- **Related terms**: ook op de kernwoorden, als het onderwerp breder kan
  zijn dan de letterlijke term — vindt woorden die er inhoudelijk bij horen
  zonder het kernwoord te bevatten (bij "keuken op maat" bijvoorbeeld
  "maatwerkkeuken").

De vorm die je aanlevert, letterlijk zo:

```
Groep: Bruiloftscatering  →  Overview op alles; Matching + Related op "bruiloftscatering"
bruiloftscatering, catering bruiloft, bruiloft catering kosten, trouwerij catering, catering trouwfeest, bruiloftscatering breda

Groep: Bedrijfscatering  →  Overview op alles; Matching op "bedrijfscatering"
bedrijfscatering, lunch kantoor bestellen, catering zakelijk, bedrijfslunch, kantoorcatering breda
```

Vergeet de instellingen niet: land **Nederland** (google.nl) en laat de
gebruiker bij een export álle kolommen meenemen.

**Stap 3 — jij leest de cijfers en kiest.** Per groep wijs je één
hoofdzoekwoord aan (beste verhouding tussen volume, moeilijkheid en
koopintentie) en een handvol ondersteunende termen. Wees eerlijk over wat
de data zegt: een groep waar niemand op zoekt, sneuvelt — ook als de
gebruiker hem leuk vond. En andersom: vindt Matching of Related iets groots
dat jij miste, dan wordt dát het hoofdzoekwoord.

**Stap 4 — jij bouwt de pagina's en teksten erop.** Dit is waar het
onderzoek voor bestond, dus doe het ook: per groep schrijf (of herschrijf)
je de pagina die dat hoofdzoekwoord target — het hoofdzoekwoord in titel,
h1 en de eerste alinea, de ondersteunende termen verwerkt in tussenkoppen
en lopende tekst waar ze natuurlijk passen, en de inhoud beantwoordt de
vraag achter het zoekwoord echt (wie "bruiloftscatering kosten" zoekt, wil
bedragen zien, geen sfeerverhaal). Geen woord vaker dan het natuurlijk
leest: de lat per pagina hierboven blijft gelden, en een tekst die naar
zoekwoorden ruikt verliest van een tekst die de vraag beantwoordt.

## De SEO-analyse, stap voor stap

Altijd in deze volgorde, en meld per stap wat je vond in gewone taal:

1. **Nulmeting eigen site**: organisch verkeer, aantal zoekwoorden waar de
   site op scoort, de autoriteit (DR) en de best presterende pagina's. Dit is
   de foto van vandaag waar elke latere meting tegen afgezet wordt — sla hem
   op in het project (`seo/nulmeting-<datum>.md`).
2. **Quick wins**: zoekwoorden waar de site op positie 4-15 staat. Daar is
   één betere pagina vaak genoeg voor de eerste pagina van Google, en dat is
   het goedkoopste verkeer dat er bestaat. Benoem per zoekwoord wélke pagina
   het is en wat eraan moet gebeuren.
3. **Technische staat**: loop de on-page-lat hierboven langs en gebruik de
   site-auditdata als die er is. Kapotte links en ontbrekende titels eerst —
   dat is gratis herstel.
4. **Backlinkprofiel**: hoeveel verwijzende sites, en vooral: zitten er
   rommelige tussen. Geen linkjacht adviseren; wel benoemen waar natuurlijke
   kansen liggen (brancheverenigingen, leveranciers, lokale media).

## De concurrentieanalyse, stap voor stap

1. **Kies twee of drie concurrenten** — uit de vraag aan de gebruiker plus
   wat de data zegt. Meer niet: tien concurrenten analyseren levert een
   dik rapport op waar niemand iets mee doet.
2. **Het gat in de inhoud** (content gap): op welke zoekwoorden scoren zíj
   wel en deze site niet? Filter op wat realistisch én relevant is — een
   zoekwoord met lage moeilijkheid en echte koopintentie wint van een groot
   zoekvolume waar de site nooit op gaat winnen.
3. **Hun beste pagina's**: welke pagina's brengen hun het meeste organische
   verkeer, en wat voor soort pagina's zijn dat (prijspagina's, blogs,
   regiopagina's)? Dat zegt wat er in deze branche wérkt.
4. **De krachtsverhouding eerlijk benoemen**: is hun autoriteit veel hoger,
   zeg dan dat de kop-aan-kop-zoektermen voorlopig niet te winnen zijn en
   richt het advies op de niches en de regio waar het wél kan.
5. **Eindig met een top vijf**: de vijf acties met het meeste effect voor de
   minste moeite, in gewone taal, met per actie wat het concreet oplevert.
   Een analyse zonder prioriteiten is een encyclopedie, geen advies.

## Vastleggen voor de volgende sessie — verplicht

Een SEO-analyse is duur (Ahrefs-tegoed, tijd van de gebruiker) en een nieuwe
chat weet er niets van. Zonder vastlegging doet de volgende sessie het
onderzoek dunnetjes over, of erger: kiest ándere hoofdzoekwoorden en
herschrijft pagina's die net goed stonden. Daarom:

- **Alles wat uit Ahrefs komt en elke keuze die je erop baseert, gaat in de
  map `seo/` van het project**, met de datum in de bestandsnaam:
  `seo/nulmeting-2026-09-01.md`, `seo/zoekwoordenonderzoek-2026-09-01.md`,
  `seo/concurrentieanalyse-2026-09-01.md`. Geen datum verzinnen — de datum
  is de houdbaarheid van de data.
- **Het belangrijkste document is `seo/zoekwoordkeuzes.md`**: per pagina het
  gekozen hoofdzoekwoord, de ondersteunende termen, en in één zin waarom
  (de cijfers die de doorslag gaven). Dit is geen archief maar de actuele
  waarheid: verandert een keuze, werk dit bestand bij — de gedateerde
  onderzoeken blijven staan als onderbouwing.
- **Zet in de `CLAUDE.md` van het project één verwijzing**: dat de map
  `seo/` bestaat, wat het laatste onderzoek was en van wanneer. Een nieuwe
  sessie die aan SEO begint, leest éérst die map — staat er een onderzoek
  van minder dan een paar maanden oud, dan bouw je daarop voort in plaats
  van opnieuw te beginnen.
- Werk deze bestanden **in dezelfde beurt** bij als waarin je de keuze
  maakt, niet achteraf. Een sessie kan elk moment stoppen; wat alleen in
  het gesprek staat, is dan weg.

## Meetbaar afronden

Klaar is pas klaar als je dit gecontroleerd hebt en het resultaat aan de
gebruiker meldt:

- [ ] Elke pagina heeft een unieke titel en metabeschrijving (loop de lijst
      echt langs, meld het aantal).
- [ ] De Rich Results Test accepteert de structured data.
- [ ] Lighthouse: Performance, Accessibility, Best Practices en SEO alle
      vier gemeten; noem de vier getallen eerlijk.
- [ ] De sitemap staat op /sitemap.xml en klopt met de echte pagina's.
- [ ] Een gedeelde link toont titel, tekst én afbeelding (test met een
      preview-tool of in een chat-app).
- [ ] De map `seo/` en `seo/zoekwoordkeuzes.md` zijn bijgewerkt en de
      `CLAUDE.md` verwijst ernaar — een nieuwe chat kan verder zonder het
      onderzoek over te doen.
