# Dateline

*[English version](README.md)*

Dateline viser den viktigste hendelsen som skjedde på en gitt dato i historien, med alle andre år som deler datoen listet opp under.

**Nettside:** https://andersbjorkh.github.io/dateline/

## Bruk

- Bla mellom dager med knappene ← / → eller piltastene, velg en dato i kalenderen, eller gå tilbake til i dag med **Today**.
- Datoen ligger i adressen (`#07-20`), så alle dager kan lenkes til eller bokmerkes.
- Tidslinjen plasserer alle hendelsene på datoen etter årstall. Klikk på et merke for å hoppe til hendelsen.
- **Curated only** skjuler alt unntatt Wikipedias utvalgte («selected») hendelser. **Oldest first** / **Newest first** bestemmer rekkefølgen. Begge valgene huskes i `localStorage`.
- **Make headline** løfter en hvilken som helst hendelse til toppen for dette besøket. **Restore pick** setter det redaksjonelle valget tilbake.
- Lyst og mørkt tema følger systeminnstillingen, og kan byttes med temaknappen (valget huskes). Animasjoner tar hensyn til innstillingen for redusert bevegelse.

Selve grensesnittet er på engelsk.

## Hvordan det fungerer

- `data/MM.json` inneholder én måned med hendelser fra Wikipedias [On this day](https://api.wikimedia.org/wiki/Feed_API/Reference/On_this_day)-feed, uten duplikater. Hver hendelse har en poengsum: antall Wikipedia-språkutgaver som har en artikkel om hovedtemaet (fra Wikidata-sitelinks).
- Hovedsaken for hver dato er et redaksjonelt valg lagret i `picks.json`. Popularitet alene valgte stadig artikler som «World War II» eller en landside, så `shortlist.py` snevrer inn hver dato til rundt 15 kandidater, og hovedsaken velges for hånd blant dem. En dato uten valg faller tilbake på høyeste poengsum, med et lite tillegg for utvalgte hendelser.
- Bare bildet til hovedsaken beholdes, nedskalert til 300 px og lagt inn som en JPEG-data-URI, så siden gjør ingen forespørsler mot Wikimedia mens den kjører.
- `index.html` er en statisk side uten byggesteg og uten avhengigheter. Den laster bare måneden den trenger.

### Dataformat

Hver `data/MM.json` knytter en tosifret dag til et objekt som dette, her for `07.json` → `"20"`:

```jsonc
{
  "top": 55,                 // indeksen til hovedsaken i `events`
  "img": "data:image/jpeg;base64,…",  // bilde til hovedsaken, eller null
  "events": [                // sortert etter år, eldste først
    {
      "y": 1969,
      "text": "The Apollo 11 Lunar Module Eagle landed on the Sea of Tranquility…",
      "score": 105,          // antall Wikidata-sitelinks for hovedartikkelen
      "sel": true,           // med i Wikipedias utvalgte («selected») liste
      "main": { "t": "Apollo 11", "desc": "First crewed Moon landing (1969)", "url": "https://en.wikipedia.org/wiki/Apollo_11" },
      "links": [{ "t": "…", "url": "…" }]   // opptil fem relaterte artikler
    }
  ]
}
```

`picks.json` knytter `MM-DD` til `{ "year", "text" }`. `text` må være helt lik teksten til en hendelse.

## Kjøre lokalt

Siden henter data med `fetch`, så den må serveres over HTTP i stedet for å åpnes direkte som fil:

```sh
python3 -m http.server 8000
# åpne http://localhost:8000/
```

## Bygge dataene på nytt

```sh
pip install requests pillow
python3 build_data.py   # henter feeden og Wikidata-tall til .cache/, skriver data/
python3 shortlist.py    # valgfritt: skriver kandidatlister til .cache/shortlist.txt (krever .cache/ fra build_data.py)
```

API-svar mellomlagres i `.cache/` (ignorert av git), så senere kjøringer går raskt. Slett mappen for å hente ferske data.

For å endre en hovedsak: rediger `picks.json` og kjør `build_data.py` på nytt.

## Publisering

Nettstedet er et sett statiske filer som GitHub Pages serverer fra roten av repoet. `.nojekyll` slår av Jekyll-behandling, slik at filene publiseres som de er.

## Kreditering

Hendelsestekstene kommer fra Wikipedia under CC BY-SA 4.0. Bildene kommer fra Wikimedia Commons.
