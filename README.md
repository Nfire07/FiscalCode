# Layout.vue — Documentazione props

Componente Vue generico per layout flex e grid.
Genera inline style calcolati a partire dalle props,
senza classi CSS esterne.

---

## Struttura HTML

### `tag`
- **Tipo:** `String`
- **Default:** `"div"`

Tag HTML dell'elemento radice. Qualsiasi tag semantico valido: `section`, `main`, `article`, `nav`, `header`, ecc.

---

## Modalità display

### `type`
- **Tipo:** `String`
- **Default:** `"flex"`

Prop principale che controlla la modalità del layout.
Viene analizzata da `parseTemplateType()` e può assumere i seguenti valori:

| Valore | Risultato |
|---|---|
| `"flex"` | `display: flex` |
| `"row"` | `display: flex` + `flex-direction: row` |
| `"col"` / `"column"` / `"stack"` | `display: flex` + `flex-direction: column` |
| `"wrap"` | `display: flex` + `flex-wrap: wrap` |
| `"grid"` | `display: grid` senza colonne predefinite |
| `"3"` (numero intero) | `display: grid` + `grid-template-columns: repeat(3, 1fr)` |
| `"1fr 2fr"` | `display: grid` + `grid-template-columns: 1fr 2fr` |
| `"1fr 200px"` | `display: grid` + colonne miste fr + unità assolute |
| `"repeat(3, minmax(...))"` | `display: grid` + template CSS diretto |

### `inline`
- **Tipo:** `Boolean`
- **Default:** `false`

Se `true`, usa `inline-flex` o `inline-grid` al posto di `flex` / `grid`.

---

## Grid — colonne, righe, aree

### `columns`
- **Tipo:** `String | Number`
- **Default:** `null`

Sovrascrive la definizione delle colonne grid derivata da `type`.
- Numero intero → `repeat(N, 1fr)`
- Stringa → usata direttamente come valore di `grid-template-columns`

### `rows`
- **Tipo:** `String | Number`
- **Default:** `null`

Definisce `grid-template-rows`.
- Numero intero → `repeat(N, 1fr)`
- Stringa → valore CSS diretto

### `areas`
- **Tipo:** `String | Array`
- **Default:** `null`

Named grid areas per `grid-template-areas`.
- Array di stringhe → ogni elemento diventa una riga racchiusa tra virgolette
- Stringa → usata direttamente come valore CSS

**Esempio con array:**
```vue
:areas="['header header', 'sidebar main', 'footer footer']"
```

### `autoFill`
- **Tipo:** `Boolean`
- **Default:** `false`

Genera `repeat(auto-fill, minmax(minWidth, 1fr))`.
Richiede che la prop `minWidth` sia impostata.

### `autoFit`
- **Tipo:** `Boolean`
- **Default:** `false`

Genera `repeat(auto-fit, minmax(minWidth, 1fr))`.
Richiede che la prop `minWidth` sia impostata.
A differenza di `autoFill`, le colonne vuote vengono collassate.

### `minWidth`
- **Tipo:** `String`
- **Default:** `null`

Larghezza minima delle colonne per `autoFill` e `autoFit`.
Esempi: `"200px"`, `"12rem"`, `"30%"`.

---

## Flex — direzione e wrap

### `direction`
- **Tipo:** `String`
- **Default:** `null`

Sovrascrive `flex-direction` indipendentemente dal valore di `type`.
Valori accettati: `"row"`, `"col"`, `"column"`, `"stack"` (tutti → `column`),
o qualsiasi valore CSS valido per `flex-direction`.

### `wrap`
- **Tipo:** `Boolean`
- **Default:** `true`

Controlla `flex-wrap` in modalità flex.
- `true` → `flex-wrap: wrap`
- `false` → `flex-wrap: nowrap`

Ignorato in modalità grid.

---

## Spaziatura

### `gap`
- **Tipo:** `String | Number`
- **Default:** `"1rem"`

Gap uniforme tra gli elementi (sia in flex che in grid).
- Numero → convertito in `px`
- Stringa → usata come valore CSS diretto

Ignorato se `rowGap` o `columnGap` sono impostati.

### `rowGap`
- **Tipo:** `String | Number`
- **Default:** `null`

Gap verticale tra le righe. Ha precedenza su `gap` per l'asse verticale.

### `columnGap`
- **Tipo:** `String | Number`
- **Default:** `null`

Gap orizzontale tra le colonne. Ha precedenza su `gap` per l'asse orizzontale.

### `padding`
- **Tipo:** `String`
- **Default:** `null`

Padding interno dell'elemento. Accetta qualsiasi valore CSS valido per `padding`.

### `margin`
- **Tipo:** `String`
- **Default:** `null`

Margin esterno dell'elemento. Applicato solo se il valore è diverso da `"0"`.

> ⚠️ Nel sorgente è presente un typo: `defautl` al posto di `default`.
> Il valore di fallback non viene applicato correttamente.
> Correggere in `default: "0px"`.

---

## Dimensioni e allineamento

### `align`
- **Tipo:** `String`
- **Default:** `null`

Imposta `align-items`. Valori CSS validi: `center`, `flex-start`, `flex-end`, `stretch`, `baseline`.

### `justify`
- **Tipo:** `String`
- **Default:** `null`

Imposta `justify-content`. Valori CSS validi: `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly`.

### `width`
- **Tipo:** `String`
- **Default:** `null`

Imposta `width`. Qualsiasi valore CSS valido.

### `height`
- **Tipo:** `String`
- **Default:** `null`

Imposta `height`. Qualsiasi valore CSS valido.

### `maxWidth`
- **Tipo:** `String`
- **Default:** `null`

Imposta `max-width`. Se `center` è `true` e questa prop non è definita,
viene forzata automaticamente a `100%`.

### `center`
- **Tipo:** `Boolean`
- **Default:** `false`

Centra l'elemento orizzontalmente impostando `margin-inline: auto`.
Se `maxWidth` non è definito, imposta automaticamente `max-width: 100%`
per evitare che l'elemento si espanda oltre il contenitore.

---

## Comandi di installazione

```bash
# Crea progetto Vue 3 con Vite (setup consigliato da zero)
npm create vue@latest my-project
cd my-project
npm install
npm run dev

# Solo Vue 3
npm install vue@^3

# Vite + plugin Vue (per progetti esistenti)
npm install --save-dev vite @vitejs/plugin-vue

# Font DM Sans usato dal componente
npm install @fontsource/dm-sans
```

```css
/* globals.css — variabili CSS richieste dal componente */
:root {
  --font-family: 'DM Sans', sans-serif;
  --foreground: #1a1a1a;
}
```

```js
// main.js — import del font se si usa @fontsource
import '@fontsource/dm-sans'
```

# DynamicForm.vue — Documentazione

Componente Vue che genera un form completo a partire
da uno schema dichiarativo passato via prop `fields`.
Gestisce validazione, stato interno, visibilità password,
file upload e tag input senza configurazione aggiuntiva.
Dipende da **PrimeVue 4** per i componenti UI.

---

## Props

### `fields`
- **Tipo:** `Array`
- **Required:** sì

Array di oggetti FieldSchema che definisce tutti i campi
del form. Ogni oggetto deve avere almeno `name`, `type`
e `label`. Vedi la sezione "Schema dei campi" per i dettagli.

### `submitText`
- **Tipo:** `String`
- **Default:** `"Submit"`

Testo del pulsante di submit.

### `loading`
- **Tipo:** `Boolean`
- **Default:** `false`

Se `true`, disabilita il pulsante di submit e mostra
lo stato di caricamento (spinner PrimeVue).
Utile per gestire richieste async dal componente padre.

---

## Emits

### `submit-form`

Emesso quando il form viene sottomesso e supera
la validazione. Il payload è una copia shallow di
`formData` con i valori di tutti i campi.

```js
// Nel componente padre
<DynamicForm
  :fields="schema"
  @submit-form="handleSubmit"
/>

function handleSubmit(data) {
  // data = { nome: 'Mario', email: 'mario@example.com', ... }
}
```

---

## Schema dei campi — Props comuni

Ogni oggetto in `fields` può avere queste proprietà condivise:

| Prop | Tipo | Obbligatoria | Descrizione |
|---|---|---|---|
| `name` | `String` | sì | Chiave univoca del campo. Usata come chiave in `formData`. |
| `type` | `String` | sì | Tipo del campo. Vedi elenco tipi supportati. |
| `label` | `String` | sì | Etichetta visibile sopra il campo. |
| `required` | `Boolean` | no | Se `true`, il campo è obbligatorio e viene validato. |
| `fullWidth` | `Boolean` | no | Forza il campo a occupare tutta la larghezza del grid. |
| `placeholder` | `String` | no | Testo placeholder dell'input. |
| `hint` | `String` | no | Testo secondario affiancato alla label (allineato a destra). |
| `icon` | `String` | no | Nome icona Material Icons Round mostrata a sinistra dell'input. |
| `defaultValue` | `Any` | no | Valore iniziale del campo. Se omesso viene usato `null`, `false` o `[]` a seconda del tipo. |
| `errorMessage` | `String` | no | Messaggio di errore personalizzato per la validazione `required`. |
| `validate` | `Function` | no | Funzione di validazione custom: `(value, formData) => string \| ''`. Ritorna stringa vuota se valido. |
| `pattern` | `String` | no | RegExp come stringa per validare il valore. |
| `patternMessage` | `String` | no | Messaggio mostrato se `pattern` non corrisponde. Default: `"Invalid format"`. |
| `minLength` | `Number` | no | Lunghezza minima del valore stringa. |

---

## Tipi di campo supportati

### `text` · `email` · `number`

Input testuale standard. Renderizza `InputText` di PrimeVue.

| Prop aggiuntiva | Tipo | Descrizione |
|---|---|---|
| `min` | `Number` | Valore minimo (solo `number`). |
| `max` | `Number` | Valore massimo (solo `number`). |
| `step` | `Number` | Step incremento (solo `number`). |

- `email` valida automaticamente il formato con regex.
- `number` valida `min` e `max` al blur.

---

### `password`

Input password con icona lucchetto e toggle visibilità.
Renderizza `InputText` con tipo dinamico `text` / `password`.
Lo stato di visibilità è gestito internamente per campo.

---

### `textarea`

Area di testo multiriga. Renderizza `Textarea` di PrimeVue.

| Prop aggiuntiva | Tipo | Default | Descrizione |
|---|---|---|---|
| `rows` | `Number` | `4` | Numero di righe iniziali. |
| `maxLength` | `Number` | — | Lunghezza massima. Mostra un contatore `n / maxLength`. |
| `autoResize` | `Boolean` | `true` | Se `true`, l'altezza si adatta al contenuto. |

---

### `select`

Menu a tendina a selezione singola. Renderizza `Select` di PrimeVue.

| Prop aggiuntiva | Tipo | Descrizione |
|---|---|---|
| `options` | `Array` | Array di `{ label, value }`. |

---

### `multiselect`

Menu a tendina a selezione multipla. Renderizza `MultiSelect` di PrimeVue.

| Prop aggiuntiva | Tipo | Default | Descrizione |
|---|---|---|---|
| `options` | `Array` | — | Array di `{ label, value }`. |
| `maxSelectedLabels` | `Number` | `3` | Numero massimo di label mostrate prima del collasso. |

---

### `radio`

Gruppo di pulsanti radio con stile a pillola selezionabile.
Supporta icone Material per ogni opzione.

| Prop aggiuntiva | Tipo | Descrizione |
|---|---|---|
| `options` | `Array` | Array di `{ label, value, icon? }`. |

---

### `range`

Slider numerico. Renderizza `Slider` di PrimeVue.
Mostra il valore corrente centrato e min/max ai lati.

| Prop aggiuntiva | Tipo | Default | Descrizione |
|---|---|---|---|
| `min` | `Number` | `0` | Valore minimo. |
| `max` | `Number` | `100` | Valore massimo. |
| `step` | `Number` | `1` | Incremento per ogni passo. |
| `labels` | `Array` | — | Etichette testuali sotto lo slider, distribuite uniformemente. |

---

### `date`

Selettore data con calendario popup. Renderizza `DatePicker`
di PrimeVue con formato `dd/mm/yyyy`.

### `time` · `datetime-local` · `month` · `week`

Input temporali nativi HTML. Renderizzano `InputText`
con il tipo corrispondente.

---

### `color`

Color picker. Renderizza `ColorPicker` di PrimeVue.
Il valore è una stringa hex senza cancelletto (es. `"ff5733"`).
Il valore iniziale di default è `"ffffff"`.

---

### `file`

File input stilizzato con label cliccabile e icona.

| Prop aggiuntiva | Tipo | Descrizione |
|---|---|---|
| `accept` | `String` | Tipi MIME accettati. Es. `"image/*"`, `".pdf,.doc"`. |
| `multiple` | `Boolean` | Se `true`, permette selezione multipla. |

- Selezione singola → `formData[name]` è un oggetto `File`.
- Selezione multipla → `formData[name]` è un `Array<File>`.

---

### `checkbox`

Checkbox binaria. Renderizza `Checkbox` di PrimeVue.
Il valore è `true` / `false`. Default: `false`.
La label è mostrata a destra della checkbox, non sopra.

---

### `toggle`

Toggle switch. Renderizza `ToggleSwitch` di PrimeVue.
Comportamento identico a `checkbox`. Default: `false`.
La label è mostrata a destra del toggle, non sopra.

---

### `tags`

Input per aggiungere tag liberi. Premi `Enter` o `,`
per confermare un tag. Ogni tag è rimovibile.
Il valore è un `Array<string>`. Default: `[]`.

---

### `divider`

Separatore visivo orizzontale con linea e label opzionale
centrata. Non genera alcun valore in `formData`.
La prop `label` è opzionale.

> `divider` non viene validato durante il submit.

---

## Validazione

La validazione avviene in due momenti:
- **Al blur** di ogni campo (`validateField`)
- **Al submit** del form su tutti i campi contemporaneamente

L'ordine dei controlli per ogni campo è:

1. `required` — valore null, stringa vuota o array vuoto
2. Formato `email` — regex standard
3. Range `min` / `max` per campi `number`
4. `minLength` — lunghezza minima stringa
5. `pattern` — regex personalizzata
6. `validate` — funzione custom

Il primo errore trovato blocca la catena e viene mostrato
sotto il campo con animazione `slide-error`.

Il submit viene bloccato se almeno un campo non supera
la validazione. L'evento `submit-form` viene emesso solo
in caso di successo.

---

## Dipendenze richieste

```bash
# PrimeVue 4 e tema
npm install primevue@^4

# Font e icone Material Icons Round (via Google Fonts — già nel template)
# oppure in locale:
npm install material-icons

# Font DM Sans (già nel CSS via Google Fonts)
# oppure in locale:
npm install @fontsource/dm-sans
```

```js
// main.js
import { createApp } from 'vue'
import PrimeVue from 'primevue/config'
import Aura from '@primevue/themes/aura'
import '@fontsource/dm-sans'
import App from './App.vue'

const app = createApp(App)
app.use(PrimeVue, {
  theme: { preset: Aura }
})
app.mount('#app')
```

---

## Variabili CSS richieste

Il componente usa variabili CSS che devono essere
definite nel contesto padre o globalmente:

```css
:root {
  --background: #1a1a2e;   /* sfondo del form */
  --foreground: #e2e4ff;   /* testo e bordi */
  --primary: #6c63ff;      /* colore accent */
  --error: #ff6b6b;        /* colore errori */
}
```

---

## Esempio di schema completo

```js
const formSchema = [
  {
    name: 'nome',
    type: 'text',
    label: 'Nome',
    required: true,
    icon: 'person',
    placeholder: 'Mario',
  },
  {
    name: 'email',
    type: 'email',
    label: 'Email',
    required: true,
    icon: 'mail',
    placeholder: 'mario@example.com',
  },
  {
    name: 'ruolo',
    type: 'select',
    label: 'Ruolo',
    options: [
      { label: 'Frontend', value: 'fe' },
      { label: 'Backend',  value: 'be' },
    ],
  },
  {
    name: 'note',
    type: 'textarea',
    label: 'Note',
    fullWidth: true,
    maxLength: 300,
    rows: 3,
  },
  {
    name: 'privacy',
    type: 'checkbox',
    label: 'Accetto la privacy policy',
    required: true,
    errorMessage: 'Devi accettare la privacy policy',
  },
]
```