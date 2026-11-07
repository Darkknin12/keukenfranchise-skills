# keukenfranchise-skills

De Claude Code-skills van Your Kitchen, als plugin-marketplace. Collega's
installeren dit één keer (het werkplekscript doet dat al); daarna komen
updates vanzelf binnen zodra hier gepusht wordt — niemand hoeft iets te doen.

## Wat erin zit

Eén plugin, `keukenfranchise`, met drie skills:

| Skill                               | Doet                                                                |
| ----------------------------------- | -------------------------------------------------------------------- |
| `website-bouwen-keukenfranchise`    | Een statische Next.js-site bouwen en met Terraform naar Cloudflare, klaar voor KF AI. |
| `SEO-website-keukenfranchise`       | On-page SEO, zoekwoordenonderzoek en concurrentieanalyse met Ahrefs. |
| `kantoor-documenten-keukenfranchise`| Excel, Word, PowerPoint, PDF en CSV voor de administratie.           |

## Installeren (voor wie het handmatig doet)

```
claude plugin marketplace add Darkknin12/keukenfranchise-skills
claude plugin install keukenfranchise@keukenfranchise-skills
```

## Onderhoud (voor de beheerder)

De skills onderhoud je **hier** — dit is de enige bron. Een skill aanpassen:

1. Bewerk `plugins/keukenfranchise/skills/<naam>/SKILL.md`.
2. Commit en push naar `main`.
3. Klaar: Claude Code ververst gekoppelde marketplaces zelf. Wie het meteen
   wil: `claude plugin update keukenfranchise`.

Een vierde skill toevoegen = een nieuwe map onder
`plugins/keukenfranchise/skills/` met een `SKILL.md` erin (met `name` en
`description` in de frontmatter). Er hoeft verder niets geregistreerd te
worden.
