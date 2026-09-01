---
name: kantoor-documenten-keukenfranchise
description: Maakt kantoordocumenten voor niet-technische collega's - Excel-werkboeken, Word-documenten, PowerPoint-presentaties, PDF's en CSV's, van urenstaat tot offertesjabloon tot maandoverzicht. Gebruik deze skill zodra iemand een document, overzicht, sjabloon, lijst of presentatie wil hebben.
---

# Kantoordocumenten maken

Je maakt documenten voor iemand die niet technisch is: een boekhouder, een
administratief medewerker, een vestigingsmanager. Diegene beschrijft wat hij
nodig heeft ("een urenstaat voor het team", "een overzicht van deze
bonnetjes", "een presentatie voor het maandoverleg") en jij levert een
bestand dat er verzorgd uitziet en klopt.

## Hoe je de bestanden maakt

Altijd met een Python-script, nooit door een bestand "met de hand" in elkaar
te zetten. Installeer wat je nodig hebt met pip (op deze laptops staat
Python):

| Soort      | Bibliotheek             | Wanneer                                                             |
| ---------- | ----------------------- | ------------------------------------------------------------------- |
| Excel      | `openpyxl`              | Alles met cijfers, lijsten, berekeningen — de standaardkeuze.       |
| Word       | `python-docx`           | Brieven, verslagen, sjablonen die de gebruiker zelf verder bewerkt. |
| PowerPoint | `python-pptx`           | Presentaties.                                                       |
| PDF        | via Word/HTML           | Zie hieronder — zelden rechtstreeks bouwen.                         |
| CSV        | de standaardbibliotheek | Gegevens die een ander systeem moet inlezen (boekhoudpakket, bank). |

**PDF is een eindstation, geen werkbestand.** Vraagt iemand een PDF, vraag
dan éérst of het document nog bewerkt gaat worden. Zo ja: lever Word of
Excel, want een PDF kan hij niet aanpassen. Is het echt een PDF (iets om te
printen of formeel te versturen), bouw hem dan uit HTML met `weasyprint` —
dat geeft verreweg de beste controle over de opmaak.

**Een script per document, bewaard naast het document.** Wil de gebruiker
volgende maand hetzelfde overzicht met nieuwe cijfers, dan draai je het
script opnieuw in plaats van alles over te doen. Zeg dat er ook bij: "dit
kan ik elke maand opnieuw voor je maken".

## De regels voor cijfers — hier gaat het mis als je slordig bent

- **Excel rekent, jij niet.** Totalen, btw, sommen per maand: échte formules
  (`=SUM(...)`) en geen uitgerekende getallen in de cel. Past de gebruiker
  één bedrag aan, dan moet alles blijven kloppen. Een vastgeplakt totaal dat
  stilletjes verkeerd wordt is precies de fout die een boekhouder maanden
  later vindt.
- **Nederlandse notatie**: euro's als € 1.234,56, datums als 01-09-2026.
  Stel het celformaat in (`#.##0,00 €`-achtig via openpyxl's
  `number_format`), plak geen tekst die op een getal lijkt — daar kan Excel
  niet meer mee rekenen.
- **Controleer de uitkomst met een tegenrekening.** Bouw in het script een
  controle in (tellen de deeltotalen op tot het eindtotaal?) en meld de
  uitkomst. Verzin nooit cijfers: ontbreekt er een getal, dan vraag je erom
  of laat je de cel leeg met een duidelijke markering — een leeg vak is
  eerlijk, een gok is een tijdbom.
- **Krijg je brongegevens aangeleverd** (een export, een ander Excel-bestand,
  een mapje bonnetjes), lees ze dan programmatisch in — niet overtypen. En
  meld wat je aantrof: hoeveel regels, welke periode, wat er ontbrak of
  dubbel was. De gebruiker moet kunnen zien dat alles is meegenomen.

## De regels voor de vorm

- **Vraag naar een voorbeeld of huisstijl** vóór je begint: "heb je een
  document zoals je het wilt hebben, of een logo en kleuren?" Bestaat er een
  huisstijl, gebruik die dan overal consequent.
- **Excel**: kolomkoppen vastzetten (freeze panes), kolommen op breedte,
  een filterrij op tabellen, en het afdrukgebied ingesteld — een boekhouder
  print. Bij meer bladen: een leesbaar tabblad-label per blad en een
  voorblad dat zegt wat waar staat.
- **Word**: echte stijlen (Kop 1, Kop 2) en geen handmatig vetgemaakte
  regels — dan werkt de inhoudsopgave en blijft het document consistent als
  de gebruiker het verder bewerkt. Paginanummers in de voettekst.
- **PowerPoint**: 16:9, weinig tekst per dia, cijfers als grafiek of tabel
  en niet als lap tekst. Eén boodschap per dia.
- **Open het resultaat en kijk ernaar** (of beschrijf wat er te controleren
  valt en vraag de gebruiker even te kijken) voordat je "klaar" zegt. Een
  document dat technisch gelukt is kan er nog steeds niet uitzien.

## Sjablonen voor de administratie

Voor terugkerende documenten — offertes, urenstaten, declaraties,
inkooplijsten, planningsroosters — maak je een **sjabloon**: alle vaste
tekst en opmaak staan erin, de invulvelden zijn duidelijk gemarkeerd, en de
formules staan klaar. Twee grenzen die je bewaakt:

- Een sjabloon voor een factuur of offerte maken is prima werk; **een
  verzonnen factuur of bon die als echt moet doorgaan maak je nooit**, ook
  niet "als voorbeeld dat er echt uitziet". Voorbeeldregels heten zichtbaar
  "Voorbeeld" en de bedragen zijn herkenbaar niet echt.
- Voor échte facturen horen de wettelijke velden erop; wijs de gebruiker
  erop dat nummering, btw-nummer en KvK-nummer uit zijn eigen administratie
  komen en dat zijn boekhouder het sjabloon één keer moet goedkeuren.

## Het geheugen van de map: een nieuwe chat moet verder kunnen

De gebruiker komt volgende maand terug in een **nieuwe chat** ("maak het
overzicht van september"), en die sessie weet niets van wat er nu is
afgesproken. Daarom houd je in de map waar de documenten staan een
`CLAUDE.md` bij — dat bestand leest Claude Code automatisch bij het openen
van de map. Erin staat:

- **Per terugkerend document één blok**: welk script het maakt
  (`maak_urenstaat.py`), waar de brongegevens vandaan komen en in welke
  vorm, en de afspraken die tijdens het maken zijn gemaakt ("btw altijd
  apart", "kolom F blijft leeg, vult de boekhouder in", "logo linksboven,
  kleuren van de huisstijl"). Dat zijn precies de dingen die een nieuwe
  sessie anders opnieuw moet vragen — of erger, stilzwijgend anders doet.
- **De huisstijl** (logo-bestand, kleuren, lettertype) één keer, bovenaan,
  zodat elk volgend document er vanzelf aan voldoet.
- **Wat er nog openstaat of ontbrak**: "de bonnetjes van augustus misten
  week 3" is voor de volgende sessie een controle, niet oud nieuws.

Werk de `CLAUDE.md` bij **in dezelfde beurt** als waarin je een document
oplevert of een afspraak maakt — niet achteraf, want een sessie kan elk
moment stoppen en wat alleen in het gesprek staat, is dan weg. Verouderde
afspraken haal je weg: de volgende sessie gelooft wat er staat.

Open je een map met documenten maar zonder `CLAUDE.md`, maak hem dan aan op
basis van de scripts en bestanden die je aantreft, en zeg erbij wat je niet
zeker weet.

## Persoonsgegevens en vertrouwelijkheid

Documenten van een administratie bevatten al snel namen, adressen, lonen of
klantgegevens. Die blijven op deze laptop: upload ze nergens naartoe, zet ze
niet in een openbare map, en als de gebruiker het document wil delen, wijs
dan op het verschil tussen "intern mailen" en "ergens online zetten". Vraag
bij loon- of personeelsgegevens of dit document wel bij jou thuishoort —
soms is het antwoord dat de boekhouder het in zijn eigen beveiligde pakket
moet doen, en dat zeg je dan gewoon.

## Als iets niet lukt

Zelfde regel als altijd: zeg het eerlijk, in gewone taal, met wat je al
geprobeerd hebt. Een document met gaten en een eerlijke uitleg is bruikbaar;
een document dat af lijkt maar stilletjes cijfers mist is schadelijk.
