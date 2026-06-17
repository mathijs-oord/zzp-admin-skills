# Boekhoud-skills — ZZP-administratie met Claude Code

Een lichtgewicht boekhouding voor Nederlandse ZZP'ers/freelancers, volledig op **platte
bestanden** (CSV + JSON) en aangedreven door [Claude Code](https://claude.com/claude-code).
Je praat in gewone taal ("maak een factuur voor klant X", "verwerk deze bon"); de skills
redeneren en stellen voor, en een klein deterministisch programma (`lib/admin.mjs`) doet het
exacte rekenwerk en schrijft naar de bestanden.

> **Kernprincipe:** *AI bepaalt WAT, deterministische code bepaalt HOE.* Factuurnummers,
> BTW-berekening, koersconversie en PDF-generatie lopen nooit "uit het hoofd" van de AI, maar
> altijd via code. Zo blijft de administratie reproduceerbaar en controleerbaar.

Deze repo is een **Claude Code plugin**: hij levert alléén de skills + code. Jouw administratie
(facturen, bonnen, CSV's) blijft in je **eigen map** staan — die bezit je zelf, kun je backuppen,
en blijft staan als de plugin een update krijgt.

## Wat kun je ermee?

| Commando | Wat het doet |
|---|---|
| `/setup` | Eenmalige onboarding: je map initialiseren + bedrijfsgegevens en standaard betaaltermijn instellen. |
| `/factuur` | Een verkoopfactuur maken (omzet). Vult tarief/omschrijving aan op basis van eerdere facturen, kent het juiste oplopende factuurnummer toe en genereert een PDF. |
| `/zelffactuur` | Een factuur importeren die je opdrachtgever namens jou maakte (self-billing). Telt óók als omzet. |
| `/bon` | Een bon of inkoopfactuur als kostenpost verwerken. Classificeert de BTW (aftrekbaar / verlegd / niet-aftrekbaar) en rekent vreemde valuta om op de ECB-dagkoers. |
| `/betaald` | Een verkoopfactuur op betaald zetten (status + betaaldatum). |
| `/aangifte` | Een kwartaal-BTW-overzicht samenstellen (rubrieken 1a t/m 5g + ICP-opgaaf) ter voorbereiding van je aangifte. |

> In het slash-menu heten de skills `boekhoud:factuur`, `boekhoud:bon`, enz. (de plugin-naam is de
> prefix). De korte vorm `/factuur` werkt ook gewoon — Claude herkent de bedoeling, ook midden in
> een zin.

> ⚠️ Dit is een **voorbereidend** hulpmiddel. De skills berekenen en ordenen; de daadwerkelijke
> BTW-aangifte dien je zelf in op **Mijn Belastingdienst Zakelijk**. Geen vervanging voor een
> boekhouder of fiscaal advies — controleer de uitkomsten.

## Vereisten

- **[Claude Code](https://claude.com/claude-code)** (CLI, desktop of VS Code-extensie).
- **[Node.js](https://nodejs.org)** 18 of nieuwer (`node --version`). Géén npm-packages nodig.
- **Google Chrome** (of Chromium/Brave/Edge) voor het genereren van PDF's. Staat Chrome niet op de
  standaard macOS-locatie, zet dan de env-var `ADMIN_CHROME`:
  ```bash
  export ADMIN_CHROME="/Applications/Chromium.app/Contents/MacOS/Chromium"
  ```

## Installeren (voor gebruikers)

Eenmalig, in Claude Code:

```text
/plugin marketplace add mathijs-oord/zzp-admin-skills
/plugin install boekhoud@boekhoud-marketplace
```

Daarna:

1. **Maak (of kies) een map** voor je administratie, bv. `~/Administratie`, en **open die map in
   Claude Code** (`cd ~/Administratie` en start Claude Code daar). Hier komt jouw data te staan.
2. Typ **`/setup`**. Dat zet bij eerste gebruik de lege templates klaar in je map (`data/`-CSV's,
   `business.example.json`, een project-`CLAUDE.md` en de uitvoer-mappen) en vraagt vervolgens je
   bedrijfsgegevens (naam, adres, KvK, BTW-id, IBAN) + standaard betaaltermijn.
3. **Probeer het uit**, bijvoorbeeld:
   - `/factuur` → "Maak een factuur voor Acme B.V., 20 uur consultancy à €100."
   - `/bon` → "Verwerk deze bon" (+ pad naar een PDF/foto).
   - `/aangifte` → "Stel de aangifte voor dit kwartaal samen."

Werk je voor meerdere administraties? Maak per administratie een aparte map en draai `/setup` in
elk. De plugin is overal beschikbaar; de data blijft per map gescheiden.

## Updates ontvangen

Bij **third-party marketplaces staat auto-update standaard uit**. Twee opties:

- **Handmatig** (altijd de nieuwste versie ophalen):
  ```text
  /plugin marketplace update boekhoud-marketplace
  ```
- **Auto-update aanzetten**: open `/plugin` → tabblad **Marketplaces** → kies
  `boekhoud-marketplace` → **Enable auto-update**. Updates komen dan bij het opstarten van Claude
  Code binnen.

Updates raken alleen de plugin-code/skills/regels — **nooit je data**. Je `data/`, `facturen/`,
`bonnen/` en `aangiftes/` blijven onaangeroerd.

## Hoe het is opgebouwd

**De plugin (deze repo — wordt geüpdatet):**

```
zzp-admin-skills/
├── .claude-plugin/
│   ├── marketplace.json     marketplace-definitie (deze repo = marketplace)
│   └── plugin.json          plugin-manifest (naam: boekhoud)
├── skills/                  de skills: setup, factuur, zelffactuur, bon, betaald, aangifte
├── lib/admin.mjs            deterministische kern (factuurnr, BTW, FX, PDF, CSV-I/O)
├── templates/               HTML-templates voor de factuur- en aangifte-PDF's
├── rules/                   Nederlandse BTW-regels in markdown — de kennisbron van de skills
└── seed/                    lege templates die /setup naar je eigen map kopieert
```

**Jouw map (per gebruiker — blijft van jou):**

```
~/Administratie/
├── CLAUDE.md                projectconventies (door /setup neergezet)
├── data/                    je administratie (CSV's + business.json)
├── facturen/<YYYY-QN>/      gegenereerde + geïmporteerde factuur-PDF's, per kwartaal
├── bonnen/<YYYY-QN>/        kopie van je bonbestanden, per kwartaal (bewaarplicht)
└── aangiftes/               de aangifte-PDF's per kwartaal
```

## Je data & privacy

- Alles staat lokaal in je eigen map. Er gaat niets naar een externe boekhoud-dienst.
- De plugin-repo bevat **nooit** jouw administratie — alleen generieke code + lege templates.
- Eén externe aanroep: bij vreemde valuta haalt `/bon` de ECB-dagkoers op via de open
  [frankfurter.app](https://www.frankfurter.app)-API.

## Veiligheid by design

De skills **verwijderen of overschrijven nooit** automatisch een factuur of bon. Corrigeren of
verwijderen doe je bewust met de hand in de CSV-bestanden. Zo kan een verkeerd begrepen opdracht
nooit stilletjes je administratie wissen.

---

## Voor de maintainer (updates uitbrengen)

Deze repo is tegelijk de **marketplace** én de **plugin** (één-repo-opzet, plugin op de root via
`"source": "./"`).

**Een update uitbrengen** = pushen naar de default branch:

```bash
git add -A
git commit -m "..."
git push
```

Het `version`-veld is bewust **weggelaten** uit `plugin.json`, dus elke commit telt als een nieuwe
versie. Gebruikers halen 'm op met `/plugin marketplace update boekhoud-marketplace` (of automatisch
als ze auto-update aan hebben). Wil je later vaste releases (semver), zet dan `version` in
`plugin.json` en hoog die per release op.

**Lokaal testen vóór je pusht** — in een aparte testmap, niet in deze repo:

```text
/plugin marketplace add /Users/<jij>/Documents/zzp-admin-skills
/plugin install boekhoud@boekhoud-marketplace
```

Daarna in een lege testmap `/setup` draaien. Pas je de plugin aan, herlaad met
`/plugin marketplace update boekhoud-marketplace` (lokaal pad) en `/plugin reload-plugins`.

**Validatie:**

```bash
claude plugin validate .
```

**Architectuur kort:** code/skills/regels/templates leven in de plugin en worden via
`$CLAUDE_PLUGIN_ROOT` aangeroepen; data wordt door `lib/admin.mjs` weggeschreven relatief aan de
map waar Claude Code draait (`process.cwd()` / `CLAUDE_PROJECT_DIR`, te overriden met
`ADMIN_DATA_DIR`). Zo overleeft gebruikersdata elke plugin-update.

## Herkomst

Afgeleid van de business-logica van **Vink**, een Mac-boekhoud-app waarvan de ontwikkeling is
gestopt. De fiscale rekenregels (BTW-routing, rubrieken, factuureisen) komen daaruit voort.

---

*Gebruik op eigen verantwoordelijkheid; dit is geen fiscaal advies.*
