# Teksten in `content/`, foto's en downloads in `public/`

Collega's kunnen een site bijwerken via KF AI (de Claude-app). De assistent
mag daar **alleen** bestanden in `content/` en `public/` wijzigen, met een
vaste lijst bestandstypen. Een tekst die in een `.tsx`-bestand staat, kan via
de Claude-app dus niet veranderd worden. Daarom is dit verplicht voor elke
site, ook als niemand er nu om vraagt.

## Wat waar staat

| Map | Wat erin hoort | Toegestaan via de Claude-app |
|---|---|---|
| `content/` | Alle teksten die een bezoeker leest: pagina's, koppen, knopteksten, titels en metabeschrijvingen, openingstijden, adres, acties | `.md`, `.json`, `.yml`, `.yaml`, `.txt`, `.csv` |
| `public/` | Foto's, logo's, iconen, downloads (folders, prijslijsten) | `.png`, `.jpg`, `.jpeg`, `.webp`, `.avif`, `.gif`, `.ico`, `.svg`, `.pdf`, `.txt` |

De code (`app/`, `components/`, `functions/`, `infra/`, configuratie) blijft
bij de ontwikkelaar in Claude Code. Een vraag over opmaak of code gaat niet
via de Claude-app.

Een indeling die goed werkt:

```
content/
  instellingen.yml      bedrijfsnaam, telefoon, adres, openingstijden, menu
  home.md
  over-ons.md
  contact.md
  acties/
    voorjaarsactie.md   één bestand per actie: de lijstpagina leest de map
public/
  fotos/
  downloads/
```

Regels voor de namen: kleine letters, cijfers en streepjes, geen spaties of
leestekens. De assistent weigert vreemde paden, en een collega moet een naam
in een zin kunnen noemen ("pas content/contact.md aan").

## Een pagina in Markdown

```markdown
---
titel: Contact
beschrijving: Bel, mail of kom langs in onze showroom in Utrecht.
---

# Kom langs

We zijn open van dinsdag tot en met zaterdag.
```

- **Titel en metabeschrijving staan in de frontmatter**, zodat een collega ze
  ook kan aanpassen. De SEO-lat geldt nog steeds per pagina.
- **Geen HTML in Markdown.** Wat tussen `<` en `>` staat, wordt weggelaten.
  Zeg dat in de README, zodat niemand zich afvraagt waar zijn tekst bleef.
- Foto's in een tekst verwijzen naar `public/`: `![Showroom](/fotos/showroom.webp)`.
  Alleen eigen foto's: de Content-Security-Policy laat geen afbeeldingen van
  andere sites toe (`img-src 'self'`), en dat moet zo blijven.

## De pagina's lezen `content/` bij de build

Next.js met `output: "export"` leest de bestanden tijdens `npm run build`. Er
draait daarna niets meer; de uitvoer is gewone HTML.

```ts
// lib/inhoud.ts — leest content/ bij de build. Veiligheidskeuzes: zie hieronder.
import { readFile } from "node:fs/promises";
import path from "node:path";
import { parse } from "yaml";
import { z } from "zod";

const MAP = path.join(process.cwd(), "content");

const Kop = z.object({
  titel: z.string().min(1).max(70),
  beschrijving: z.string().min(1).max(160),
});

export async function leesPagina(naam: string) {
  const bestand = path.join(MAP, `${naam}.md`);
  const ruw = await readFile(bestand, "utf8");
  const m = /^---\r?\n([\s\S]*?)\r?\n---\r?\n?/.exec(ruw);
  const kop = Kop.safeParse(m ? parse(m[1]) : {});
  if (!kop.success) {
    // Deze melding ziet een collega in de Claude-app als de build faalt.
    throw new Error(`content/${naam}.md: ${kop.error.issues.map((i) => i.path.join(".") + " " + i.message).join("; ")}`);
  }
  return { ...kop.data, tekst: m ? ruw.slice(m[0].length) : ruw };
}
```

```tsx
// Een pagina toont de tekst zonder ruwe HTML.
import Markdown from "react-markdown";
import remarkGfm from "remark-gfm";

<Markdown remarkPlugins={[remarkGfm]} skipHtml>
  {pagina.tekst}
</Markdown>
```

Installeer de nieuwste stabiele versies (`npm view <pakket> version`), niet
uit je geheugen.

## Veiligheid: waarom precies zo

Iedereen in de groep websitebouwers kan via de Claude-app bestanden in
`content/` zetten, en een tekst uit een mail kan een prompt bevatten. Wat in
`content/` staat, moet dus **gegevens blijven en nooit code worden**, niet op
de site en niet tijdens de build:

- **Geen `rehype-raw`**, geen `dangerouslySetInnerHTML`, geen
  `allowDangerousHtml`. Anders zet een tekst een `<script>` of een nep-
  inlogformulier op de site.
- **Laat `urlTransform` van react-markdown op de standaard staan.** Die haalt
  `javascript:`-links weg.
- **Geen MDX** (`.mdx`, `next-mdx-remote`, `@next/mdx` voor `content/`): MDX
  is code. De Claude-app weigert `.mdx` ook.
- **Geen `gray-matter`** voor de frontmatter: dat pakket voert een
  frontmatter die met `---js` begint uit als JavaScript, tijdens de build.
  Gebruik het `yaml`-pakket zoals hierboven; `parse` maakt alleen gegevens.
- **JSON** met `JSON.parse`, **CSV** met een eenvoudige parser; nooit `eval`,
  `new Function` of `import()` met een pad uit `content/`.
- **Een tekst kiest geen code.** Mag een pagina in de frontmatter een
  sjabloon kiezen (`indeling: breed`), dan via een vaste lijst in de code
  (`z.enum([...])`), nooit als bestandsnaam of componentnaam.
- **Valideer met zod** en laat de build falen met een duidelijke melding.
  Een kapotte build zet niets live; de oude site blijft staan, en de
  collega ziet in de Claude-app welk bestand niet klopt.

## Controle

- Zoek in `app/` en `components/` naar zichtbare zinnen die nog in de code
  staan (koppen, knoppen, alt-teksten, meta). Alles wat een bezoeker leest,
  hoort in `content/`. Uitzondering: technische teksten als "Pagina niet
  gevonden" mogen in de code, maar zet ze liever ook in
  `content/instellingen.yml`.
- Pas een tekst in `content/` aan, bouw, en kijk of hij op de site verandert.
