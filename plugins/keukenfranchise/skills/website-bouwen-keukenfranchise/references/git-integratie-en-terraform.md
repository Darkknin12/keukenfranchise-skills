# Pages-project met Git-integratie, Access en DNS in Terraform

Dit is de Terraform die bij elke site hoort. Hij is zo opgezet dat collega's de
site via KF AI (de Claude-app) kunnen bijwerken, zonder dat Terraform en de
assistent elkaar in de weg zitten. Lees eerst waarom; de code staat eronder.

## Waarom zo

- **Publiceren is mergen.** Het Pages-project hangt aan de GitHub-repository.
  Cloudflare bouwt zelf: een merge naar `main` geeft een productiebuild, een
  push op een tak `website/…` geeft een voorbeeld. Mislukt een productiebuild,
  dan blijft de oude site staan. Wat live staat, is altijd terug te vinden in
  `main`.
- **Nooit Direct Upload** (`wrangler pages deploy`). Een project dat zo begint,
  kan volgens Cloudflare later niet meer aan Git gekoppeld worden. Dan moet er
  een nieuw project komen en verhuist het domein (zie
  `bestaande-site-migreren.md`).
- **Voorbeelden alleen voor `website/*`.** De assistent maakt al zijn takken
  met dat voorvoegsel. Andere takken (Dependabot, oude proeven) kosten dan
  geen builds.
- **Voorbeelden alleen na een Microsoft-login.** Een voorbeeld kan teksten
  bevatten die nog niet openbaar mogen. De Access-app op
  `*.<project>.pages.dev` vraagt een login met het werkaccount en laat alleen
  leden van de websitegroepen door. Het productieadres `<project>.pages.dev`
  en de eigen domeinen vallen er niet onder: daar staat de echte site, en
  Cloudflare kan een eigen domein niet koppelen als er al een Access-regel op
  staat.
- **Eén eigenaar per naam.** Terraform beheert het hoofddomein (de kale naam
  en `www`), de mailrecords van Resend en de Access-app. Extra namen
  (`actie.<domein>`) koppelt de assistent via de Claude-app; zijn records
  hebben het DNS-commentaar `ykai:<site>`. Terraform raakt alleen wat in zijn
  eigen state staat, dus zolang elke naam één eigenaar heeft, is er geen
  drift. Daarom staat elke naam in een **eigen resource**. Een resource die
  alle records van een zone of alle domeinen van een project als één lijst
  beheert, zou de namen van de assistent als drift zien en weghalen.

## Wat de mens eerst moet doen

Dit kan jij niet; leg het stap voor stap uit en wacht tot het gelukt is.

1. **De site staat in het Cloudflare-account van het bedrijf**, het account met
   het Zero Trust-team (de Microsoft-login voor de voorbeelden). In een eigen
   account werken de voorbeelden achter de login en de koppeling met KF AI
   niet.
2. **De GitHub-repository staat in de organisatie waar de app `ykai-websites`
   op staat.** Vraag de beheerder welke organisatie dat is als het niet in de
   `CLAUDE.md` staat.
3. **De Cloudflare-app op GitHub.** In Cloudflare: Workers & Pages → Create →
   Pages → Connect to Git, GitHub kiezen en de app "Cloudflare Workers and
   Pages" installeren met **Only select repositories** en alleen de
   website-repo's. Het project zelf maak je níet in het dashboard (dat doet
   Terraform); de stap is alleen nodig om GitHub aan Cloudflare te koppelen.
   Zonder die koppeling faalt `terraform apply` op het `source`-blok.
4. **Het token** heeft naast de vaste lijst uit de SKILL.md ook
   **Account → Access: Apps and Policies → Edit**, voor de Access-app.
5. **De twee ID's van de Access-regel en de aanmeldbron.** Die komen uit de
   Terraform `infra/cloudflare-access` in de repo van de kennisassistent
   (outputs `access_regel_id` en `aanmeldbron_id`). Vraag ze aan de beheerder.
   Het zijn geen geheimen; ze mogen in `infra/terraform.tfvars`.

## Provider

Zoek de nieuwste stabiele versie live op (registry.terraform.io, provider
`cloudflare/cloudflare`; op 7-11-2026 was dat 5.28.1) en controleer het schema
in de docs van díe versie. De voorbeelden hieronder volgen het v5-schema
(attributen met `=`, geen blokken).

```hcl
terraform {
  required_providers {
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 5.28" # live opgezocht; verhoog bij elke nieuwe stabiele versie
    }
  }
}
```

## Variabelen

```hcl
variable "cloudflare_account_id" { type = string }
variable "zone_id" { type = string }         # of cloudflare_zone.site.id als de zone hier staat
variable "domein" { type = string }          # de kale naam, bijvoorbeeld "keukenfranchise.nl"
variable "project_naam" { type = string }    # kleine letters, cijfers en streepjes
variable "github_eigenaar" { type = string } # de organisatie
variable "github_repo" { type = string }

# Uit de outputs van infra/cloudflare-access (repo van de kennisassistent).
variable "access_regel_id" {
  type        = string
  description = "Herbruikbare Access-regel 'AI-websites' (output access_regel_id)"
}
variable "aanmeldbron_id" {
  type        = string
  description = "Entra ID-aanmeldbron in Access (output aanmeldbron_id)"
}
```

## Het Pages-project

```hcl
resource "cloudflare_pages_project" "site" {
  account_id        = var.cloudflare_account_id
  name              = var.project_naam
  production_branch = "main"

  build_config = {
    build_command   = "npm run build"
    destination_dir = "out" # Next.js met output: "export"
    root_dir        = ""
    build_caching   = true
  }

  source = {
    type = "github"
    config = {
      owner                          = var.github_eigenaar
      repo_name                      = var.github_repo
      production_branch              = "main"
      production_deployments_enabled = true
      pr_comments_enabled            = true

      # Alleen takken die met website/ beginnen krijgen een voorbeeld.
      # Cloudflare: eerst excludes, dan includes, wat nergens op past wordt
      # overgeslagen. Met alleen een include en geen excludes valt al het
      # andere dus vanzelf af (voorbeeld 1 in "Branch deployment controls").
      # Zet géén exclude "*": volgens dezelfde volgorde zou die ook website/*
      # kunnen tegenhouden.
      preview_deployment_setting = "custom"
      preview_branch_includes    = ["website/*"]
    }
  }

  # De hele site hangt aan dit project; een vervanging is een nieuwe
  # subdomein en een domeinverhuizing. Liever een harde stop in het plan.
  lifecycle {
    prevent_destroy = true
  }
}
```

Let op:

- `deployments_enabled` is in v5 verouderd; gebruik
  `production_deployments_enabled` en `preview_deployment_setting`.
- De Node-versie zet je in de repo (`.node-version`) of als `NODE_VERSION`;
  kies de huidige LTS, live opgezocht.
- Geheimen (zoals `RESEND_API_KEY`) zet de gebruiker zelf met
  `wrangler pages secret put`; die horen niet in de Terraform. Zet ze voor
  **production én preview**, anders werkt het formulier in het voorbeeld niet.
- `subdomain` van het project is niet altijd `<naam>.pages.dev`: is de naam
  al bezet, dan plakt Cloudflare er iets achter. Verwijs daarom altijd naar
  `cloudflare_pages_project.site.subdomain` en nooit naar een zelfgemaakte
  string.

## De Access-app voor de voorbeelden

```hcl
resource "cloudflare_zero_trust_access_application" "voorbeelden" {
  account_id = var.cloudflare_account_id
  name       = "Voorbeelden ${var.project_naam}"
  type       = "self_hosted"

  # *.<project>.pages.dev dekt de unieke voorbeeldadressen en de takadressen,
  # maar niet <project>.pages.dev zelf en geen eigen domein.
  domain = "*.${cloudflare_pages_project.site.subdomain}"
  destinations = [{
    type = "public"
    uri  = "*.${cloudflare_pages_project.site.subdomain}"
  }]

  # Alleen de Microsoft-login; geen andere aanmeldbronnen.
  allowed_idps              = [var.aanmeldbron_id]
  auto_redirect_to_identity = true
  app_launcher_visible      = false
  session_duration          = "24h"

  policies = [{
    id         = var.access_regel_id
    precedence = 1
  }]
}
```

Zet **nooit** Access op het productieadres of op een eigen domein, en gebruik
niet de schakelaar "Enable access policy" van Pages: die laat alleen leden van
het Cloudflare-account toe, niet de collega's.

## DNS en domeinen: één resource per naam

```hcl
# De kale naam. Een CNAME op de kale naam mag bij Cloudflare (CNAME flattening).
resource "cloudflare_dns_record" "kaal" {
  zone_id = var.zone_id
  name    = var.domein
  type    = "CNAME"
  content = cloudflare_pages_project.site.subdomain
  proxied = true
  ttl     = 1 # automatisch; verplicht bij proxied
  comment = "terraform:${var.project_naam}"
}

resource "cloudflare_dns_record" "www" {
  zone_id = var.zone_id
  name    = "www.${var.domein}"
  type    = "CNAME"
  content = cloudflare_pages_project.site.subdomain
  proxied = true
  ttl     = 1
  comment = "terraform:${var.project_naam}"
}

resource "cloudflare_pages_domain" "kaal" {
  account_id   = var.cloudflare_account_id
  project_name = cloudflare_pages_project.site.name
  name         = var.domein
}

resource "cloudflare_pages_domain" "www" {
  account_id   = var.cloudflare_account_id
  project_name = cloudflare_pages_project.site.name
  name         = "www.${var.domein}"
}
```

- `cloudflare_pages_domain` maakt zelf geen DNS-record; daarom staan de
  CNAME's erbij.
- Het commentaar `terraform:<project>` laat in het dashboard zien wie de
  eigenaar is. Gebruik **nooit** `ykai:` als commentaar: dat merkteken is van
  de assistent, en een record met dat merkteken kan hij als het zijne
  aanzien.
- **Mailrecords van Resend** (MX, SPF, DKIM, DMARC): elk record een eigen
  `cloudflare_dns_record`, met de waarden die Resend toont, ook met
  `comment = "terraform:<project>"`.
- De doorverwijzing van `www` naar de kale naam (of andersom, zie de
  SEO-skill) is geen DNS en geen domein; die mag in een eigen regel.

**Nooit** in deze Terraform:

- een resource of module die alle records van een zone in één keer beheert
  (een lijst records, een zonebestand, "authoritative" DNS);
- een lijst domeinen op het project;
- een naam die de assistent al gekoppeld heeft (DNS-commentaar `ykai:<site>`).
  Wil iemand zo'n naam in Terraform, dan ontkoppelt een publiceerder hem eerst
  via de Claude-app, en pas daarna komt hij erin. Niet importeren: Cloudflare
  weigert een tweede CNAME op dezelfde naam, dus de `apply` faalt dan
  zichtbaar in plaats van stil iets over te nemen.

## Controle na elke koppeling via de Claude-app

Heeft een publiceerder een extra naam gekoppeld via de Claude-app, draai dan
`terraform plan` in `infra/`. Dat moet **"No changes"** geven. Wil het plan
iets weghalen of wijzigen dat je niet zelf veranderd hebt, stop dan en zoek
uit waarom; pas niets toe.

## `terraform plan` en `apply`

- Lees het plan altijd en vat het in één zin samen voor de gebruiker.
- Staat er `destroy` of `must be replaced` bij het Pages-project of een
  domein, stop dan en leg uit wat dat betekent (de site kan even weg zijn).
  `prevent_destroy` houdt een vervanging van het project tegen; haal die regel
  alleen weg na uitdrukkelijk akkoord van de gebruiker.
- Controleer na de eerste `apply` in het dashboard (project → Settings →
  Builds → Branch control) dat er "Custom branches" staat met alleen
  `website/*`, en dat een push op een tak `website/proef` een voorbeeld geeft
  dat om een Microsoft-login vraagt. Een tak zonder dat voorvoegsel hoort geen
  build te krijgen.
