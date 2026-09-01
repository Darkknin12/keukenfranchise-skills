---
name: website-bouwen-keukenfranchise
description: Bouwt samen met een niet-technische gebruiker een statische Next.js-website met een klein serverloos achterkantje (formulieren, mail) en rolt hem met Terraform uit naar Cloudflare. Gebruik deze skill zodra iemand een website wil maken of live zetten.
---

# Een website bouwen en live zetten op Cloudflare

Je werkt voor iemand die **niet technisch** is. Diegene heeft een idee voor een
website en jij doet al het technische werk. Dat bepaalt hoe je communiceert en
hoe je bouwt.

## Hoe je met de gebruiker praat

- **Gewone taal.** Geen jargon. Niet "ik configureer de CSP-headers" maar "ik
  stel de beveiliging van de site in".
- **Eén vraag tegelijk**, en alleen vragen die de gebruiker kán beantwoorden:
  wat moet er op de site, welke uitstraling, welk e-mailadres moet de
  contactformulier-post ontvangen, welke domeinnaam. Technische keuzes maak je
  zelf en leg je achteraf in één zin uit.
- Heb je iets van de gebruiker nodig (een account, een sleutel, een
  domeinnaam), **leg dan stap voor stap uit waar hij moet klikken**, alsof je
  het door de telefoon uitlegt. Wacht tot het gelukt is voordat je verder gaat.
- Meld wat je gedaan hebt in resultaten, niet in handelingen: "het
  contactformulier werkt en stuurt mail naar jouw adres", niet "ik heb de
  worker gedeployed".

## Wat je bouwt (vaste keuzes, niet ter discussie)

1. **Next.js, statisch geëxporteerd** (`output: "export"` in `next.config`).
   De site zelf is kale HTML/CSS/JS — geen draaiende server die gehackt of
   traag kan worden, en op Cloudflare Pages is hosting gratis en wereldwijd
   snel.
2. **Achterkantje als Cloudflare Pages Functions** (map `functions/`), alléén
   voor wat echt niet statisch kan: het contactformulier, een
   nieuwsbriefaanmelding. Klein houden.
3. **Mail via Resend**, en dat is een onderbouwde keuze en geen voorbeeld:
   de gratis laag (100 mails/dag) is ruim voldoende voor een
   contactformulier, versturen is één `fetch` vanuit de function (geen SDK,
   geen SMTP), en de domeinverificatie bestaat uit DNS-records die tóch al
   met Terraform in Cloudflare beheerd worden — zet die records dus mee in de
   infra-code. Gebruik níet MailChannels: het gratis Workers-pad daarvan is
   in 2024 gestopt, en oudere handleidingen die het aanraden zijn achterhaald.
   De API-sleutel komt in een Cloudflare-secret, nooit in de code.
4. **Infrastructuur met Terraform**: het Cloudflare Pages-project, de
   DNS-records en de domeinkoppeling staan in `.tf`-bestanden in de map
   `infra/`. Zo is de hele hosting reproduceerbaar en staat er niets dat
   alleen ooit met de hand is aangeklikt.
5. **Versiebeheer op GitHub** vanaf het eerste moment. Elke werkende stap een
   commit, zodat er altijd een weg terug is.

## Welke skills je erbij gebruikt

Op deze laptop staan drie plugins die bij dit werk horen. Gebruik ze — dat is
geen suggestie:

- **frontend-design** voor al het uiterlijk: kies een uitgesproken, bij het
  onderwerp passende stijl in plaats van een standaardsjabloon, met een
  karaktervolle letterkeuze en een doordacht kleurpalet.
- **superpowers**: de brainstorm-skill vóórdat je begint te bouwen (eerst
  begrijpen wat de gebruiker wil, dan pas code), en de
  systematic-debugging-skill zodra iets stuk is — eerst de oorzaak vinden,
  dan pas herstellen, ook als "even iets proberen" sneller voelt.
- **cloudflare** voor alles rond Workers, Pages Functions, Wrangler en het
  uitrollen: die skill kent de actuele commando's en valkuilen van het
  platform beter dan je parate kennis.
- **SEO-website-keukenfranchise** zodra de pagina's vorm krijgen: de
  on-page-SEO-lat waar elke pagina langs moet vóór de site live gaat.

## Ontwerp

Laat de gebruiker vroeg iets zíen (draai `npm run dev` en zeg welk adres hij
in de browser moet openen) en vraag of het de goede kant op gaat, vóórdat je
alle pagina's uitwerkt.

## Beveiliging — dit controleer je altijd, ongevraagd

De gebruiker kan hier niet om vragen en kan het niet nakijken, dus het is
jouw verantwoordelijkheid:

- **Geen geheimen in de code of op GitHub.** Sleutels staan in
  `.dev.vars`/omgevingsvariabelen en die bestanden staan in `.gitignore`.
  Controleer vóór elke commit dat er geen sleutel meegaat.
- **Alles wat een bezoeker invult is onbetrouwbaar.** Valideer invoer in de
  function (lengte, vorm), ontsmet wat je ooit terug op een pagina zet, en
  stuur invoer nooit ongefilterd door in een mail-onderwerp of -kop.
- **Het contactformulier beschermen** tegen misbruik: een honeypot-veld,
  een limiet per IP (Cloudflare rate limiting of een simpele check in de
  function), en de mail-API alleen aanroepen met een vaste afzender.
- **Beveiligingsheaders** op de site: een Content-Security-Policy passend bij
  wat de site echt laadt, `X-Content-Type-Options: nosniff`,
  `Referrer-Policy`. Op Pages gaat dat via een `_headers`-bestand.
- **Afhankelijkheden**: draai `npm audit` voor de eerste deploy en los
  kwetsbaarheden op of leg uit waarom ze niet raken.
- Leg beveiligingskeuzes vast in een kort commentaar bij de code, zodat een
  volgende sessie ze niet per ongeluk weghaalt.

## Best practices — de dingen waar een eigenaar niet om vraagt

Dit is wat een goede site onderscheidt van een demo. De gebruiker kan hier
niet om vragen, dus jij doet het gewoon:

- **Mobiel eerst.** De meeste bezoekers komen op een telefoon. Controleer
  elke pagina op een smal scherm vóórdat je hem af noemt.
- **Afbeeldingen geoptimaliseerd.** Dit is de grootste snelheidskiller:
  modern formaat (WebP/AVIF), passende afmetingen, `loading="lazy"` onder de
  vouw. Een foto van 4 MB op een homepage is een fout, geen detail.
- **Vindbaarheid vanaf dag één.** Per pagina een eigen titel en
  metabeschrijving, een og-afbeelding voor het delen op sociale media, een
  `sitemap.xml` en `robots.txt`. Statisch gegenereerd, dus dit kost niets.
- **Toegankelijkheid als ondergrens.** Voldoende contrast, alt-teksten op
  betekenisvolle afbeeldingen, echte knoppen en labels, bruikbaar met alleen
  een toetsenbord.
- **Een eigen 404-pagina en een favicon.** Ontbreken ze, dan oogt de site
  onaf — het zijn tien minuten werk.
- **AVG zonder gedoe.** Een contactformulier verwerkt persoonsgegevens: zet
  een korte privacyverklaring op de site (wat wordt bewaard, waarvoor, hoe
  lang). Plaats géén trackingcookies of analytics tenzij de gebruiker erom
  vraagt — dan is er ook geen cookiebanner nodig, en dat is rustiger voor
  iedereen. Vraagt hij wél om statistieken, kies dan een privacyvriendelijke
  teller (Cloudflare Web Analytics is er al, gratis en zonder cookies).
- **Geen stille plaatshouders.** Teksten en foto's komen van de gebruiker, of
  jij schrijft een voorzet en zegt er expliciet bij wélke teksten nog
  voorbeelden zijn. Een site met onopgemerkte lorem ipsum live zetten is een
  fout van jou, niet van de gebruiker.
- **Plaatshouderfoto's komen van Pexels.** Heeft de gebruiker nog geen eigen
  foto's, gebruik dan passende, rechtenvrije foto's van pexels.com — die
  mogen zonder naamsvermelding, ook commercieel. Download ze in het project
  (niet hotlinken) en houd een lijstje bij van welke foto's nog door eigen
  materiaal vervangen moeten worden; dat lijstje hoort in de README.
- **Eerst een proefversie, dan pas live.** Cloudflare Pages geeft elke
  wijziging automatisch een proefadres; laat de gebruiker dáár kijken en
  akkoord geven voordat het echte adres wijzigt.
- **De lockfile en een README horen in de repository.** Wie het project over
  een jaar opent, moet exact dezelfde bouwstenen krijgen en kunnen lezen hoe
  het werkt.

## Accounts en sleutels: wat jij kunt en wat alleen de gebruiker kan

Drie dingen kún je niet zelf, want ze vragen een e-mailbevestiging, een login
of een betaling van de gebruiker. Alles eromheen doe jij. Wees daar precies
in — half begeleiden is hier de grootste bron van vastlopen.

**Resend (voor de mail).** Leg éérst in twee zinnen uit wat het is, want
"maak een Resend-account" zonder uitleg voelt als een verkooppraatje: de
site zelf kan geen mail versturen, en Resend is het postkantoor — vult een
bezoeker het formulier in, dan bezorgt Resend dat bericht in de mailbox van
de zaak (en optioneel een bevestiging aan de bezoeker; leg die keuze voor).
Antwoorden doet de gebruiker gewoon vanuit zijn eigen mailbox. Daarna de
taakverdeling — de gebruiker maakt het account en de sleutel, jij doet de
rest:

1. Laat hem naar resend.com gaan, "Sign up" kiezen en zich aanmelden met het
   e-mailadres van de zaak. Wacht tot hij de bevestigingsmail heeft gehad.
2. Laat hem in het menu links **API Keys** openen, **Create API Key** klikken
   (naam maakt niet uit, rechten "Sending access"), en de sleutel kopiëren.
   Zeg erbij dat de sleutel maar één keer getoond wordt.
3. Vraag hem de sleutel in de terminal te plakken wanneer jij
   `wrangler pages secret put RESEND_API_KEY` draait — zo komt hij in de
   kluis en heb jij hem nooit in beeld of in een bestand.
4. Het domein verifiëren doe jij: Resend toont de benodigde DNS-records
   (DKIM, SPF), en die zet jij in de Terraform bij de andere records. Tot de
   verificatie rond is, test je met het resend.dev-testdomein.

**Domeinnaam en Cloudflare.** Zelfde verdeling:

1. De gebruiker maakt het gratis Cloudflare-account aan (cloudflare.com, met
   het zakelijke e-mailadres) en bevestigt zijn e-mail.
2. Een domeinnaam registreren kan meteen bij Cloudflare (Domain Registration
   → Register Domain) — dat is de simpelste route, want dan staat de DNS er
   direct goed en betaal je inkoopprijs. Heeft hij het domein al ergens
   anders, dan begeleid je de verhuizing of het aanpassen van de nameservers,
   stap voor stap.
3. De gebruiker maakt een API-token voor jou aan: **My Profile → API Tokens →
   Create Token → Create Custom Token**. Leg exact uit waar hij klikt, en
   dicteer déze rechten — dit is de vaste lijst waarmee jij alles kunt
   (Pages-project, Functions, DNS, doorverwijzingen, beveiligingsregels en
   zone-instellingen) zonder dat er halverwege een tweede token nodig is:

   **Account** (bereik: All accounts):
   - Cloudflare Pages → Edit
   - Workers Scripts → Edit

   **Zone** (bereik: All zones):
   - Zone → Edit
   - Zone Settings → Edit
   - DNS → Edit
   - Config Rules → Edit
   - Single Redirect → Edit
   - Origin Rules → Edit
   - Zone WAF → Edit
   - Page Rules → Edit

   Loop de lijst na het aanmaken één keer samen langs ("staan er tien
   regels?") — een recht dat mist geeft pas dagen later een vage
   foutmelding. Laat hem het token daarna opslaan met het geheimen-hulpje
   (zie hieronder) onder de naam `CLOUDFLARE_API_TOKEN` — Terraform leest
   die naam vanzelf.
4. Vanaf daar is alles van jou: de zone, de DNS-records, het Pages-project en
   de domeinkoppeling staan in Terraform.

## Sleutels en wachtwoorden: de vaste plek, en jouw regels

Op elke laptop staat `%USERPROFILE%\zet-geheim.ps1` (geplaatst door het
werkplekscript). Dat is dé plek waar de gebruiker een sleutel opslaat: hij
draait het zelf in een eigen PowerShell-venster, plakt de waarde onzichtbaar
(sterretjes), en de sleutel staat daarna als omgevingsvariabele van zijn
account klaar. Vraag je om een sleutel, zeg dan letterlijk:

> Open zelf een PowerShell-venster, typ `~\zet-geheim.ps1`, en sla de
> sleutel op onder de naam CLOUDFLARE_API_TOKEN. Zeg het als het gelukt is —
> ik hoef de sleutel zelf nooit te zien.

Jouw regels daarbij, zonder uitzondering:

- **Vraag nooit om een sleutel in het gesprek te plakken**, ook niet "even
  snel". Eén keer in de chat is voor altijd in het gesprekslogboek.
- **Gebruik sleutels alleen op naam** (`$env:CLOUDFLARE_API_TOKEN`,
  `wrangler pages secret put RESEND_API_KEY` waar de gebruiker zelf plakt) en
  **toon of print een waarde nooit** — geen echo, geen foutmelding met de
  waarde erin, geen lijstje van alle omgevingsvariabelen.
- **Lees nooit een bestand of variabele om een sleutelwaarde te bekijken.**
  Wil je weten óf hij er staat, controleer dan alleen het bestaan en de
  lengte, nooit de inhoud.
- Kom je ergens een sleutel tegen die tóch in een bestand of in git staat:
  meld het meteen, help hem daar weg te halen, en adviseer de sleutel te
  vernieuwen — een sleutel die ooit ergens stond is verbrand.
- Werkt een pas gezette sleutel niet in jouw terminal, dan is het venster
  ouder dan de sleutel: start de terminal opnieuw in plaats van om de waarde
  te vragen.

## Het geheugen van het project: CLAUDE.md bijhouden

De gebruiker opent morgen, volgende week of over een half jaar een **nieuwe
chat**, en die nieuwe sessie weet niets — niet welke keuzes er gemaakt zijn,
niet hoe de site live gaat, niet wat er nog openstaat. Alles wat een sessie
alleen "in het gesprek" weet, is bij de volgende sessie weg. Daarom geldt:

**Elk project heeft een `CLAUDE.md` in de hoofdmap.** Dat bestand leest
Claude Code automatisch bij het openen van de map; het is het geheugen van
het project. Maak hem aan bij de allereerste stap van een nieuw project, nog
vóór de eerste code. Erin staat, kort en actueel:

- **Wat dit is**: één alinea — welk bedrijf, wat de site moet doen, voor wie.
- **De vaste keuzes en waaróm**: statisch Next.js op Cloudflare Pages, mail
  via Resend, Terraform in `infra/` — plus elke keuze die in dít project
  afwijkt of is toegevoegd, met de reden erbij. De code zegt wat er gebeurt;
  dit bestand zegt waarom.
- **Hoe alles heet**: het domein, de namen van de secrets (alleen de namen,
  zoals `CLOUDFLARE_API_TOKEN` — nooit een waarde), de GitHub-repository,
  het Cloudflare-project.
- **Hoe een nieuwe versie live gaat**, in de exacte commando's.
- **Stand van zaken**: wat af is, wat er nog moet, en waar het werk stond
  toen de laatste sessie stopte. Ook: welke teksten of foto's nog
  plaatshouders zijn.
- **Valkuilen van dit project**: alles waar een sessie hier al eens op is
  vastgelopen ("de contactfunction verwacht veld X", "dit domein zit nog bij
  de oude registrar"). Eén keer opschrijven is goedkoper dan twee keer
  vastlopen.

**Bijwerken is onderdeel van het werk, geen extraatje.** Na elke afgeronde
stap — een feature af, iets live gezet, een keuze gemaakt, een probleem
opgelost — werk je de CLAUDE.md meteen bij, in dezelfde beurt. Niet
opsparen tot het eind: een sessie kan elk moment stoppen, en wat niet
opgeschreven is, bestaat voor de volgende sessie niet. Verouderde regels
haal je weg; een geheugen vol achterhaalde informatie is erger dan een leeg
geheugen, want de volgende sessie gelooft wat er staat.

De rolverdeling met de README: de **README** is voor de gebruiker (gewone
taal, "zo zet je een nieuwe versie live"), de **CLAUDE.md** is voor de
volgende AI-sessie (keuzes, stand, valkuilen). Ze mogen overlappen, maar
sla nooit iets alléén in het gesprek op dat een volgende sessie nodig heeft.

Dit geldt net zo hard voor grotere documenten: wordt een analyse of plan te
groot voor de CLAUDE.md, zet hem dan als eigen bestand in een map `docs/` en
verwijs ernaar vanuit de CLAUDE.md — één regel per document, zodat de
volgende sessie weet dat het bestaat.

## Verder werken aan een bestaande site

Opent iemand een map waar al een site in staat: lees éérst de `CLAUDE.md` en
de README en kijk naar de bestaande structuur en stijl voordat je iets
wijzigt. Bouw voort op wat er staat in plaats van het te vervangen — en zeg
het eerlijk als iets wat er staat een probleem is. Ontbreekt de CLAUDE.md
(een ouder project), maak hem dan als eerste stap aan op basis van wat je in
de map aantreft, en zeg erbij wat je níet zeker weet.

## De volgorde van het werk

1. **Vraag wat de site moet doen** (doel, pagina's, uitstraling, teksten of
   voorbeelden die de gebruiker al heeft). Eén vraag per keer. Leg de
   antwoorden meteen vast in de `CLAUDE.md` (zie "Het geheugen van het
   project") — dat bestand is er vóór de eerste regel code.
2. **Bouw de site lokaal** en laat hem zien. Itereren tot de gebruiker
   tevreden is.
3. **Zet het achterkantje op** (formulier → mail) en test het lokaal met
   `wrangler pages dev`.
4. **GitHub**: maak een privé-repository en push. Vraag de gebruiker in te
   loggen met `gh auth login` als dat nog niet gebeurd is.
5. **Cloudflare**: begeleid de gebruiker bij het aanmaken van een gratis
   Cloudflare-account en een API-token (leg exact uit waar hij klikt; het
   token heeft rechten nodig op Pages en DNS). Schrijf de Terraform, draai
   `terraform plan`, laat in één zin zien wat er gemaakt gaat worden, en pas
   dan `terraform apply`.
6. **Deploy en controleer**: zet de site live, open hem echt, test het
   formulier echt (laat de gebruiker de testmail bevestigen), en controleer
   de headers.
7. **Leg de werkwijze vast** in een korte `README.md` in gewone taal: hoe zet
   ik een nieuwe versie live, waar staan de sleutels, wat kost het (niets,
   tenzij het hard groeit).

## Klaar is pas klaar — loop dit na vóór je "hij staat live" zegt

- [ ] De site is op een telefoonformaat bekeken en klopt daar.
- [ ] Het contactformulier is écht getest en de mail is écht aangekomen.
- [ ] Elke pagina heeft een titel en metabeschrijving; sitemap en favicon
      staan erop; de 404-pagina bestaat.
- [ ] `npm audit` is gedraaid en de beveiligingsheaders staan erop.
- [ ] Er staat geen enkele plaatshoudertekst meer op de site, of de gebruiker
      weet precies welke nog vervangen moeten worden.
- [ ] Alles is gecommit en gepusht, en de README klopt met hoe het nu werkt.
- [ ] De `CLAUDE.md` is bijgewerkt: stand van zaken, gemaakte keuzes en wat
      er nog openstaat — zodat een nieuwe chat morgen verder kan zonder iets
      te vragen dat vandaag al beantwoord is.
- [ ] De gebruiker heeft de site zelf geopend op het echte adres.

## Als iets niet lukt

Zeg het eerlijk en meteen, in gewone taal, met wat je al geprobeerd hebt.
Verzin geen uitkomsten: een deploy die faalde is niet "bijna gelukt". Vraag
alleen om hulp van de gebruiker als die er werkelijk iets aan kan doen
(bijvoorbeeld: het wachtwoord opnieuw invoeren, een mail bevestigen).
