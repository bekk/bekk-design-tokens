# Migrering av Kilden til de nye tokenene

Plan for designsystem-faggruppa: hva som må gjøres for at [Kilden](https://github.com/bekk/designsystem) skal bruke tokenstrukturen i dette repoet i stedet for dagens `@kilden/design-tokens`.

Tallene under er målt mot Kilden på `main` (`3fd8c21`) og `dist/tokens.css` i dette repoet, 25. oktober 2026. Det som er målt står som tall. Det som er en beslutning eller en anbefaling er merket slik.

## Kort fortalt

- Størrelsen er overkommelig. Kilden bruker 117 forskjellige `var(--…)`. Bare 38 finnes ikke i de nye tokenene og er ikke definert lokalt i Kilden. 13 av dem er ikke tokens (8 fra Radix, 3 fra Shiki og 2 fragmenter i strenger). De 25 reelle er 18 typografivariabler som har fått nytt navn, 6 elevation-variabler og `--dimension-0`.
- **Rollebyttet er den største jobben.** Gamle `dominant` (temafargen) heter nå `accent`, og gamle `accent` (lime) heter nå `dominant`. Det er 104 klassetreff i 48 filer, og de må byttes samtidig.
- **`data-color-scheme` må alltid settes.** I nye tokens finnes de semantiske fargene bare inne i `[data-color-scheme]`. Kilden setter i dag attributtet bare ved eksplisitt lys eller mørk modus.
- **Skygger (elevation) følger ikke med.** De ligger i kode og må bli værende i Kilden.
- **Fargene `neutral-…-hover/active`, chip og noen andre er redesign.** De kan ikke oversettes mekanisk. Se [Redesign](#redesign-som-går-parallelt).
- To ting er ikke avgjort og må avklares før arbeidet starter: hva som skjer med merkene `basic`, `geeky` og `goofy`, og hvordan tokenene pakkes. Se [Åpne spørsmål](#åpne-spørsmål).

## Del 1: Gjøres i tokens og Figma

### Ferdig

- Roller `neutral`, `dominant`, `accent`, `interactive` og status, med temaene `bekk-vann`, `bekk-jord`, `bekk-ild` og `bekk-luft`. Se [README](../README.md).
- Surface-skalaen, `neutral` med `disabled`-tilstander, `backdrop`, motion og typografi (inkludert bredde og kursiv).
- Fast spacing-skala på 4px-rutenett, med halve og negative steg (`--spacing-1-5`, `--spacing-neg-1`).
- Token Spark: `rem` per token, `duration`-type for motion, og avvisning av `.` i tokennavn.

### Gjenstår

| Hva | Hvor | Status |
| --- | --- | --- |
| Fjerne `primitive.dimension.*` og gjøre `spacing.unit` til en fast verdi (4) | Figma, deretter Token Spark → Push | `dist/tokens.css` har fortsatt `--primitive-dimension-*`, og `--spacing-unit` peker på `--primitive-dimension-step` |
| Skygger som tokens (Figma Effect Styles ⇄ DTCG) | Eget prosjekt | Ikke startet. Skygger ligger i kode i mellomtiden |
| Push regenererer ikke `dist/` hvis bare `metadata.json` endres | Token Spark | Kjent begrensning. Omgås ved at en vanlig token-endring følger etterpå |

## Del 2: Gjøres i Kilden

Rekkefølgen er foreslått slik at hvert steg kan merges for seg uten å bryte noe.

### 1. HTML-attributter (`apps/docs/app/root.tsx`)

| I dag | Nytt |
| --- | --- |
| `data-theme="bekk-<tema>"` | Uendret. Nye tokens bruker samme navn |
| Standardtema `ild` settes i skriptet | Uendret. Siden skriptet alltid setter temaet, trengs ingen `defaultTheme` i tokens |
| `data-color-scheme` settes bare for `light` og `dark` | **Settes alltid**: `light`, `dark` eller `auto` (følger operativsystemet). Uten attributtet finnes ingen semantiske farger |
| `data-brand` | Finnes ikke i nye tokens. Se [Åpne spørsmål](#åpne-spørsmål) |
| `.dark`-klassen | Beholdes. Den driver Tailwind-varianten `dark:` (`@custom-variant dark` i `app.css`, 4 filer bruker `dark:`) og er uavhengig av tokenene |

Tre ting å vite:
- `data-theme` alene endrer ikke semantiske farger. For en seksjon med eget tema må både `data-theme` og `data-color-scheme` stå på den.
- En seksjon kan låses til én modus med `data-color-scheme="light"` eller `"dark"`. Det erstatter egne tokens for «alltid lys».
- Standardtemaet har ingen egen `[data-theme]`-blokk. En seksjon kan derfor ikke bytte tilbake til det fra et annet tema.

### 2. Fargene og rollebyttet

Gamle `dominant-*` blir `accent-*` og omvendt. Gjør det som ett søk-og-erstatt med mellomliggende plassholder, ellers ender alt på samme rolle. Omfang: 104 klassetreff (`bg-`, `text-`, `border-`, `outline-`, `ring-`, `shadow-` med flere) i 48 filer, fordelt på komponenter, CSS og dokumentasjon (`packages/designsystem/skills`).

Kilden bruker 42 forskjellige fargevariabler. Av dem er 23 uendret, 8 er rene rollebytter, 10 har avgjort mapping, og 1 (chip) er redesign. Alle nye navn i tabellen er kontrollert mot `dist/tokens.css`. Fullstendig tabell står i [vedlegget](#vedlegg-fargemapping).

Avgjort mapping der navnet ikke finnes i nye tokens:

| Gammel | Bruk | Ny | Hvorfor |
| --- | --- | --- | --- |
| `neutral-border-subtle` | 7: skillelinjer, accordion, tabs, tabeller, menyer | `neutral-border-disabled` | Skillelinjer er svake linjer |
| `neutral-surface-tinted` | skeleton | `neutral-surface-disabled` | Samme visuelle styrke |
| `{status}-background-default` | alert og toast | `{status}-surface-default` | Toast og alert bruker alvorlighetsfargen. Nøytral toast bruker `neutral-surface-default` |
| `neutral-surface-hover` / `-active` | 17: knapper, menyer, tabeller, toast | `dominant-surface-hover` / `-active` | `neutral` har ikke hover. Komponenten henter hover fra `dominant` eller `accent` |
| `neutral-border-strong` | secondary-knapp ved trykk | `accent-border-default` | `dominant-border-default` og `neutral-border-default` er begge `#1f1f1f` i lys modus, så trykket ville ikke vises |
| `dominant-border-subtle` | skygge på aktiv tab | `accent-border-default` | Eller dropp skyggen i redesignet |

### 3. Typografi

- Variabelnavnene er endret: `--typography-<stil>-font-size` → `--typography-<stil>-fontSize`, og `-line-height` → `-lineHeight`. Det gjelder 18 variabler i `styles.css` (`--text-*` og `--text-*--line-height`).
- Linjehøyde var tidligere en lengde i `rem`. Den er nå et tall uten enhet (1.3 og 1.4). Verifiser at `--text-*--line-height` gir riktig resultat i Tailwind.
- Nye variabler finnes per stil: `-letterSpacing`, `-fontWeight`, `-fontStretch`, `-textCase`. Bruk dem der Kilden i dag hardkoder disse (`uppercase` i `button.tsx` er et eksempel).
- Stilene er `heading-lg`, `heading-md`, `heading-sm`, `leading`, `body`, `title`, `small`, `caption` og `action`.
- `--font-weight-light`, `-regular` og `-medium` har samme navn som før.

**Fonter, risiko:** `font-action` bruker `--font-family-action`. I nye tokens er den `GT America`, og utvidet bredde beskrives som `--typography-action-fontStretch: 125%`. I Kilden er «utvidet medium» en egen font (`GT America Expanded` i `fonts/bekk.css`). Uten endring mister knappene og etikettene den utvidede skriften. Foreslått løsning: deklarer den utvidede filen som `font-family: "GT America"` med `font-stretch: 125%`, og sett `font-stretch` i `font-action`. Alternativt overstyrer Kilden `--font-action` lokalt.

### 4. Spacing og dimension

- `--spacing: var(--spacing-unit)` (Tailwind) er uendret, og `--spacing-unit` finnes i nye tokens (4px, og `0.25rem` etter Figma-endringen og PR #41). Tailwinds `p-4` og `gap-2` gir altså samme verdier som før.
- `primitive.dimension.*` og `--dimension-N` er fjernet, fordi komponentene ikke bruker dem. `tokens.dimension` forsvinner. Den eneste bruken jeg fant er testen i `src/lib/tokens.test.ts` som sjekker `var(--dimension-0)`, og den må fjernes.
- Skalaen kunne dokumenteres på nytt som `--spacing-0-5 … --spacing-24` (n × 4px) i `spacing.md` og `spacing-token-table.tsx`. Verdiene tilsvarer Tailwinds multipler.
- Tokenene gjør ikke lenger spacing proporsjonal med skriftstørrelsen mellom mobil og desktop. Det gjorde den gamle formelen, men komponentene brukte den ikke.

### 5. Elevation (skygger)

`styles.css` kobler `--shadow-*` til seks `--elevation-*`-variabler (`subtle`, `floating`, `overlay`, `raised-default`, `raised-hover`, `raised-pressed`). De er ikke i nye tokens og må defineres i Kilden selv. Foreslått løsning: flytt verdiene fra dagens bygg til en egen CSS-fil i `packages/designsystem`. Temabyggerens valg av skyggestil bruker de samme variablene.

### 6. Motion og backdrop

`--motion-duration-{fast,default,slow,slower}`, `--motion-easing-{standard,enter,exit}` og `--color-backdrop` har samme navn som `styles.css` allerede bruker. Det som gjenstår er å kontrollere verdiene (150/200/300/500ms og easing-kurvene).

### 7. Dokumentasjon og tester

- `packages/designsystem/skills/kilden/references/tokens/` (`borders`, `colors`, `elevation`, `motion`, `radius`, `spacing`, `typography`), `references/token-klasser.md` og `guider/egne-temaer.md`.
- `apps/docs`: `color-token-table.tsx`, `spacing-token-table.tsx`, `search-dialog.tsx` og `sidebar.tsx` nevner tokens som endres.
- `classes.test.ts` fanger ikke klasser som peker på en udefinert variabel (en slik klasse gir fortsatt CSS). Kjør derfor også `tokens-contract.test.ts`, eller dets etterfølger.

## Redesign som går parallelt

Disse kan ikke oversettes mekanisk, og komponentene må tegnes om mot den nye strukturen:

- **`neutral` er en standardfarge uten hover.** Komponenter blander `neutral` med `dominant` eller `accent` i stedet for å låse seg til én stil. Hover og active kommer fra `dominant` (anbefalt, konstant rebell) eller `accent`.
- **Chip** (`dominant-background-default` brukes for tekst på valgt chip). For en valgfri chip (choice chip) er utgangspunktet: standard `surface-tinted`, valgt `base-default` med `base-contrast-default` på teksten. De andre variantene avgjøres per variant.
- **Surface-tilstandene:** `surface.default`, `tinted`, `hover` og `active` bruker egne steg (lys 50/100/200/300, mørk 800/700/600/900), så de er alltid forskjellige. Tegn hover og active etter det.
- **Verdiene er endret med vilje** (farger og typografi). Forvent visuelle forskjeller, og kontroller alle fire temaer i lys og mørk modus.

## Åpne spørsmål

1. **Merkene `basic`, `geeky` og `goofy`.** De ligger i `packages/design-tokens/src/brands/`, og docs-appen setter `data-brand` for dem. Nye tokens beskriver bare Bekk. De skal enten leve videre (egne tokensett eller lokale overstyringer) eller utgå. Dette er ikke diskutert, og det påvirker temabyggeren.
2. **Pakking av tokens.** I dag leverer `@kilden/design-tokens` både CSS, JS-eksporten (`tokens`) og `contract.json`, som `tokens-contract.test.ts` og temabyggeren er avhengige av. Valget står mellom en npm-pakke og å kopiere `dist/tokens.css` inn i repoet. Det er ikke avgjort. Hvis pakken byttes ut, må `contract.json` og temabyggeren erstattes eller skrives om.
3. **Fonten for `action`:** egen `GT America Expanded`-fil, eller utvidet bredde via `font-stretch` (se Typografi).
4. **Skygger:** hvem eier dem på sikt (kode i Kilden, eller et eget prosjekt for Figma Effect Styles)?

## Rekkefølge

1. Avklar åpne spørsmål 1 og 2.
2. Del 1: kjør Figma-endringen (fjerne dimension), merge PR #41 og push, slik at `dist/tokens.css` er ferdig.
3. Kilden: attributter (steg 1). Liten endring som kan merges først.
4. Kilden: kodemod for roller og farger (steg 2) i én PR, med visuell sjekk av alle temaer.
5. Kilden: typografi og fonter (steg 3), spacing og dimension (steg 4), elevation (steg 5), motion (steg 6).
6. Dokumentasjon og tester (steg 7).
7. Redesign av komponenter går parallelt og leveres per komponent.

## Vedlegg: fargemapping

Alle 42 fargevariablene Kilden bruker. «Bruk» er antall treff i `packages/designsystem/src` og `apps/docs/app`. Alle nye navn finnes i `dist/tokens.css`.

| Gammel (`--color-…`) | Bruk | Ny (`--color-…`) | Type |
| --- | ---: | --- | --- |
| `neutral-text-default` | 50 | `neutral-text-default` | uendret |
| `dominant-base-default` | 39 | `accent-base-default` | rollebytte |
| `neutral-text-subtle` | 29 | `neutral-text-subtle` | uendret |
| `neutral-border-default` | 28 | `neutral-border-default` | uendret |
| `neutral-background-default` | 18 | `neutral-background-default` | uendret |
| `neutral-surface-hover` | 16 | `dominant-surface-hover` | avgjort mapping |
| `neutral-surface-default` | 11 | `neutral-surface-default` | uendret |
| `dominant-base-contrast-default` | 9 | `accent-base-contrast-default` | rollebytte |
| `neutral-border-subtle` | 7 | `neutral-border-disabled` | avgjort mapping |
| `accent-base-default` | 6 | `dominant-base-default` | rollebytte |
| `danger-border-default` | 4 | `danger-border-default` | uendret |
| `accent-base-contrast-default` | 4 | `dominant-base-contrast-default` | rollebytte |
| `danger-text-default` | 3 | `danger-text-default` | uendret |
| `success-background-default` | 2 | `success-surface-default` | avgjort mapping |
| `success-border-default` | 2 | `success-border-default` | uendret |
| `success-text-default` | 2 | `success-text-default` | uendret |
| `warning-background-default` | 2 | `warning-surface-default` | avgjort mapping |
| `warning-border-default` | 2 | `warning-border-default` | uendret |
| `warning-text-default` | 2 | `warning-text-default` | uendret |
| `danger-background-default` | 2 | `danger-surface-default` | avgjort mapping |
| `danger-base-default` | 2 | `danger-base-default` | uendret |
| `danger-base-contrast-default` | 2 | `danger-base-contrast-default` | uendret |
| `accent-text-default` | 2 | `dominant-text-default` | rollebytte |
| `dominant-border-default` | 1 | `accent-border-default` | rollebytte |
| `dominant-border-subtle` | 1 | `accent-border-default` | avgjort mapping |
| `info-background-default` | 1 | `info-surface-default` | avgjort mapping |
| `info-border-default` | 1 | `info-border-default` | uendret |
| `info-text-default` | 1 | `info-text-default` | uendret |
| `dominant-background-default` | 1 | REDESIGN (chip) | redesign |
| `success-base-default` | 1 | `success-base-default` | uendret |
| `success-base-contrast-default` | 1 | `success-base-contrast-default` | uendret |
| `warning-base-default` | 1 | `warning-base-default` | uendret |
| `warning-base-contrast-default` | 1 | `warning-base-contrast-default` | uendret |
| `info-base-default` | 1 | `info-base-default` | uendret |
| `info-base-contrast-default` | 1 | `info-base-contrast-default` | uendret |
| `accent-base-hover` | 1 | `dominant-base-hover` | rollebytte |
| `accent-base-active` | 1 | `dominant-base-active` | rollebytte |
| `neutral-border-strong` | 1 | `accent-border-default` | avgjort mapping |
| `neutral-surface-active` | 1 | `dominant-surface-active` | avgjort mapping |
| `danger-base-hover` | 1 | `danger-base-hover` | uendret |
| `danger-base-active` | 1 | `danger-base-active` | uendret |
| `neutral-surface-tinted` | 1 | `neutral-surface-disabled` | avgjort mapping |
