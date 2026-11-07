# GitHub-instellingen van een website-repo (door de mens)

Via KF AI maakt de app `ykai-websites` een tak `website/…`, een commit en een
PR, en een publiceerder zet de wijziging live door te mergen. De ruleset op
`main` zorgt dat dat alleen kan met een geslaagde build van precies die
wijziging, ook als iemand de assistent misbruikt.

**Jij zet deze instellingen niet zelf**, ook niet met `gh api` of een token:
een agent wijzigt geen rulesets, branchbeleid of bypass-lijsten. Een mens met
beheerrechten op de repo doet het. Jij legt het uit, wacht, en controleert het
daarna alleen-lezen.

## Vooraf

- **Eén keer een build gehad.** GitHub toont een check pas in de keuzelijst
  als hij de afgelopen week gedraaid heeft. Laat eerst een voorbeeld bouwen:
  push een tak `website/proef` en open er een PR voor.
- **De apps staan op de repo** (door een eigenaar van de organisatie):
  - "Cloudflare Workers and Pages", met **Only select repositories**: alleen
    de website-repo's;
  - `ykai-websites`: deze repo toevoegen aan de installatie.

## Stap voor stap (dicteer dit)

**1. De ruleset.** Repo → **Settings** → **Rules** → **Rulesets** → **New
ruleset** → **New branch ruleset**.

- **Ruleset Name:** `main beschermen`
- **Enforcement status:** Active
- **Bypass list:** leeg laten. In elk geval nooit `ykai-websites` erin: dan kan
  de app zonder geslaagde build mergen.
- **Target branches:** Add target → **Include default branch**.
- Vink aan:
  - **Restrict deletions**
  - **Block force pushes**
  - **Require a pull request before merging**
    - Required approvals: `0`. De bevestiging in de Claude-app, met de
      samenvatting en het voorbeeld, is de controle; een tweede persoon in
      GitHub is bewust niet gekozen.
    - Allowed merge methods: alleen **Squash**.
  - **Require status checks to pass**
    - **Require branches to be up to date before merging** aan. Dan is wat
      live gaat precies wat in het voorbeeld stond.
    - **Add checks** → zoek op "Cloudflare" en kies de check van Cloudflare
      Pages (hij heet doorgaans "Cloudflare Pages"). Kies bij de bron de app
      "Cloudflare Workers and Pages" als GitHub dat vraagt.
- **Create**.

**2. Mergen.** Repo → **Settings** → **General** → **Pull Requests**:

- alleen **Allow squash merging** aan (merge commits en rebase uit);
- **Automatically delete head branches** aan. De assistent verwijdert zelf
  nooit een tak; zonder deze instelling blijven alle `website/…`-takken staan.

## Gevolg voor de ontwikkelaar

Cloudflare bouwt alleen takken die met `website/` beginnen, en een tak zonder
build krijgt geen check. Een PR vanaf een andere tak kan dus nooit gemerged
worden. Werk daarom ook zelf op een tak `website/…`, bijvoorbeeld
`website/code-nieuw-menu`. De Claude-app ziet zo'n PR wel, maar weigert hem te
publiceren zodra er een bestand buiten `content/` en `public/` in zit; die
merge je zelf in GitHub. Hetzelfde geldt voor Dependabot: zet zo'n update over
op een `website/`-tak.

## Controleren (alleen lezen)

```
gh api repos/<eigenaar>/<repo>/rules/branches/main
gh api repos/<eigenaar>/<repo> --jq "{squash: .allow_squash_merge, merge: .allow_merge_commit, rebase: .allow_rebase_merge, takkenOpruimen: .delete_branch_on_merge}"
gh api repos/<eigenaar>/<repo>/rulesets
gh api repos/<eigenaar>/<repo>/rulesets/<id uit de vorige regel> --jq ".bypass_actors"
```

Je wilt zien: `deletion`, `non_fast_forward`, `pull_request` en
`required_status_checks` (met `strict_required_status_checks_policy: true` en
de Cloudflare-check) op `main`, een lege bypass-lijst, alleen squash, en
`delete_branch_on_merge: true`. Mist er iets, meld het en vraag de mens het aan
te passen.
