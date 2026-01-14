
# TODO – Web Builder (naken, browser-driven)

## Grundprinciper
- Browsern är state-maskinen
- History = undo / redo
- URL = dokument + inställningar
- Ingen egen persistence
- Minimal, läsbar logik
- Allt ska gå att förstå genom att läsa URL + koden

---

## Steg 0 – Rensa bort överflödigt ansvar

- [ ] Ta bort all användning av `localStorage`
- [ ] Ta bort `loadData()` / `saveData()`
- [ ] Acceptera:
  - reload = nytt tillstånd om ingen URL finns
  - browsern ansvarar för historik, inte appen

---

## Steg 1 – URL är enda sanningen

- [ ] Allt dokumentinnehåll kommer från URL (`?d=...`)
- [ ] Språkval kommer från URL (`?lang=en|sv`)
- [ ] `this.data` är endast en spegling av URL:n
- [ ] Ingen parallell intern state

> Appen ska alltid kunna rekonstrueras från URL + default.

---

## Steg 2 – Undo/redo via History API (browsern gör jobbet)

- [ ] Vid meningsfull ändring:
  - uppdatera URL
  - `history.pushState`
- [ ] Ingen egen undo-stack
- [ ] Ingen redo-logik
- [ ] Back/Forward = tidsnavigering

---

## Steg 3 – Minimal debounce (endast rimlighet)

Syfte:
- undvika ett history-steg per tangenttryckning
- inget mer

- [ ] En (1) timer
- [ ] Ingen abstraktion
- [ ] Ingen heuristik utöver ”vänta lite”

```js
scheduleUrlUpdate() {
  clearTimeout(this.urlTimer);

  this.urlTimer = setTimeout(() => {
    history.pushState(null, '', this.getShareUrl());
  }, 400);
}
````

Browsern:

* begränsar historik
* hanterar minne
* avgör beteende

---

## Steg 4 – popstate = återläs URL

* [ ] Lyssna på `popstate`
* [ ] Läs `?d=`
* [ ] Rendera exakt det som står i URL:n

```js
window.addEventListener('popstate', () => {
  const params = new URLSearchParams(location.search);
  const d = params.get('d');
  if (!d) return;

  const data = JSON.parse(decompress(d));
  app.data = data;
  app.elements.htmlEditor.value = data.html;
  app.elements.cssEditor.value = data.css;
  app.updatePreview();
});
```

Ingen cache. Ingen fallback. Ingen ”hjälp”.

---

## Steg 5 – Initiera första history-steget

* [ ] Vid start:

  * om `?d=` finns → använd den
  * annars:

    * använd tomt dokument (se Steg 7)
    * skriv första URL med `history.replaceState`

---

## Steg 6 – Språk: engelska och svenska

### Mål

* Enkel tvåspråkighet
* Ingen i18n-ram
* Språkval synligt och begripligt

### Beslut

* Språk styrs av:

  * URL: `?lang=en` eller `?lang=sv`
  * knapp i UI som uppdaterar URL

### Att göra

* [ ] Definiera ett enkelt språkobjekt:

```js
const strings = {
  en: { title: "Web Builder", view: "View site", save: "Save & Share" },
  sv: { title: "Webbbyggare", view: "Visa sida", save: "Spara & dela" }
};
```

* [ ] Läs språk från URL vid start
* [ ] Språkknapp:

  * ändrar `lang`
  * uppdaterar URL med `history.pushState` eller `replaceState`
* [ ] Språkval påverkar endast UI-text, inte dokumentinnehåll

> Språk är inställning → hör hemma i URL.

---

## Steg 7 – Inga placeholder-texter i editorerna

### Beslut

* Editorerna ska vara tomma från början
* Inget ”Hello world”
* Inget pedagogiskt förifyllt innehåll

### Att göra

* [ ] `defaultData.html = ""`
* [ ] `defaultData.css = ""`
* [ ] Ingen placeholder-text i `<textarea>`
* [ ] Exempel laddas endast via ”Examples”

> Tom ruta är mer ärlig än låtsasinnehåll.

---

## Medvetna beslut (inte brister)

* URL får vara lång
* URL får vara ful
* Historiken får bli stor
* Reload får rensa undo
* Browserns begränsningar accepteras fullt ut

---

## Definition of done

* [ ] Ingen localStorage
* [ ] Ingen egen undo/redo-logik
* [ ] Browsern hanterar historik
* [ ] Språk styrs via URL
* [ ] Tomma editorer vid start

**”Dokumentet och inställningarna ligger i URL:n, och webbläsaren sköter tiden.”**

```
```
