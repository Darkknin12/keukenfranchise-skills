# Een bestaande site klaarmaken voor KF AI

Een site die vóór november 2026 met deze skill gebouwd is, mist vaak twee
dingen: de Git-integratie van Cloudflare Pages, en teksten in `content/`.
Zonder die twee kan niemand de site via de Claude-app bijwerken. Doe de stappen
in deze volgorde, en werk na elke stap de `CLAUDE.md` bij.

## 1. Nagaan hoe de site er nu voor staat

- **Git-integratie?** `npx wrangler pages project list` toont per project een
  kolom "Git Provider". In het dashboard: project → Settings → Builds; staat
  daar een gekoppelde repository, dan is het goed. Staat er "Direct Upload",
  of werd de site tot nu toe met `wrangler pages deploy` live gezet, dan heeft
  hij het niet.
- **Staat het project in de Terraform van de site?** Zoek in `infra/` naar
  `cloudflare_pages_project`. Heeft het een `source`-blok?
- **Teksten in `content/`?** Kijk of de pagina's hun teksten uit `content/`
  lezen of dat ze in `.tsx`-bestanden staan.
- **Wat beheert de Terraform nog meer?** DNS als lijst of per naam, domeinen,
  mailrecords.

Vertel de gebruiker in twee of drie zinnen wat je vond en wat er moet
gebeuren, voordat je iets verandert.

## 2. Teksten naar `content/`

Zie `inhoud-in-content.md`. Werk pagina voor pagina:

1. Maak een schermafdruk van de pagina (desktop en mobiel).
2. Verhuis de teksten naar `content/<pagina>.md` (of `instellingen.yml` voor
   wat op elke pagina staat) en laat de pagina ze bij de build lezen.
3. Bouw en vergelijk met de schermafdruk: er mag niets zichtbaar veranderen.

Doe dit op een tak `website/code-teksten-naar-content` met een PR, en merge
pas als het voorbeeld klopt.

## 3. De Git-integratie

**Het project heeft al Git-integratie en staat in Terraform:** voeg de
instellingen uit `git-integratie-en-terraform.md` toe (`preview_deployment_setting
= "custom"`, include `website/*`) en draai `terraform plan`. Een gewone wijziging
is prima. Staat er `must be replaced` bij het project, stop dan: overleg met de
gebruiker, want vervangen betekent een nieuw project en een domeinverhuizing.

**Het project heeft geen Git-integratie, of staat niet in Terraform:** maak
een **nieuw** project met Git-integratie via Terraform, met een nieuwe naam
(bijvoorbeeld `<oude-naam>-git`). Niet importeren: een nieuw project is
voorspelbaar, en de stappen daarna zijn hetzelfde.

1. Nieuw project met `source` (zie `git-integratie-en-terraform.md`), en
   eerst nog **zonder** domeinen.
2. Zet de geheimen opnieuw (`wrangler pages secret put … --project-name
   <nieuw>`), voor production en preview: ze verhuizen niet mee.
3. Push naar `main` en wacht op een geslaagde productiebuild. Controleer de
   site op `<nieuw>.pages.dev`: pagina's, formulier (testmail), headers.
4. **Het domein verhuizen**, op een rustig moment, en zeg de gebruiker vooraf
   dat de site enkele minuten onbereikbaar kan zijn terwijl Cloudflare een
   nieuw certificaat maakt:
   - `cloudflare_pages_domain` van de kale naam en `www` naar het nieuwe
     project (`project_name`); het plan toont dan een vervanging van die twee
     koppelingen, en dat is hier de bedoeling;
   - de CNAME's van de kale naam en `www` naar het nieuwe
     `cloudflare_pages_project.site.subdomain`.
   - Laat de gebruiker het plan zien en pas daarna `apply`. Controleer de site
     daarna op het echte adres.
5. Het oude project laat je een week staan als terugvaloptie. Daarna
   verwijdert de gebruiker het in het dashboard (of, als het in Terraform
   stond, haal je het daar weg na zijn uitdrukkelijke akkoord).

**Stond de DNS of stonden de domeinen als één lijst in de Terraform**, zet ze
dan om naar één resource per naam, voordat de site in KF AI komt. Lees het plan
goed: het mag tijdens het omzetten geen record weghalen dat daarna niet
terugkomt.

## 4. De rest

- De Access-app voor de voorbeelden (`git-integratie-en-terraform.md`).
- De GitHub-instellingen door de mens (`github-ruleset.md`), en de apps
  "Cloudflare Workers and Pages" en `ykai-websites` op de repo.
- De regel over KF AI in de `CLAUDE.md` van de site (zie de SKILL.md).
- Een AI-beheerder zet de site in het beheerscherm van KF AI
  (**Koppelingen → Websites**), met de repo, het Pages-project, de
  productietak en de zone. Het beheerscherm controleert dat het project aan
  precies die repo en tak hangt; klopt `owner`, `repo_name` of
  `production_branch` in het `source`-blok niet, dan bestaat de site voor de
  assistent niet.

Klaar is pas klaar als een collega in de groep websitebouwers via de
Claude-app een tekst heeft aangepast, het voorbeeld (na de Microsoft-login)
heeft gezien, en `terraform plan` daarna nog steeds "No changes" geeft.
