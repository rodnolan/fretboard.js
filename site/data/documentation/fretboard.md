# Documentation - Fretboard

## Installation

Use `npm`/`yarn`:

```bash
$ npm i @moonwave99/fretboard.js --save
```

## Usage

Given an existing DOM element:

```html
<figure id="fretboard"></figure>
```

`require` / `import` the library accordingly:

```javascript
const { Fretboard, CAGED } = require('@moonwave99/fretboard.js');
// OR
import { Fretboard, CAGED } from '@moonwave99/fretboard.js';
```

Then initialise a `Fretboard` instance with desired options:

```javascript
const fretboard = new Fretboard({
  el: '#fretboard',
  fretColor: 'blue',
  dotFill: 'red',
  ...
});
```
Call the render method with the information you want to display eventually:

```javascript
const box = CAGED({
  mode: 'major',
  root: 'C3',
  box: 'C'
});

fretboard.render(box);
```

In this case, the `CAGED` method returns an array of objects containing further musical information like the note name and the scale degree, but you can pass just an array of `{ string, fret }`:

```javascript
// this would render an open C chord
fretboard.render([
  {
    string: 5,
    fret: 3
  },
  {
    string: 4,
    fret: 2
  },
  {
    string: 2,
    fret: 1
  }
]);
```

## Configuration options

**Note**: even though the context should provide enough disambiguation, the word _string_ refers to both the instrument ones and the programming data type!

Parameter         | Type     | Default      | Description
------------------|----------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
el                | string   | '#fretboard' | Container element selector
stringCount       | string   | 6            | Number of instrument strings to display
stringWidth       | string   | 1            | String line stroke width
stringColor       | string   | 'black'      | String color
fretCount         | string   | 15           | Number of frets to display
fretWidth         | string   | 1            | Fret line stroke width
fretColor         | string   | 'black'      | Fret color
nutWidth          | string   | 7            | Nut stroke width
nutColor          | string   | 'black'      | Nut color
middleFretColor   | string   | 'red'        | Middle fret color
middleFretWidth   | string   | 3            | Middle fret stroke width
scaleFrets        | string   | true         | If `true`, spaces frets logarithmically, otherwise linear
topPadding        | string   | 20           | Top padding (relative to SVG container)
bottomPadding     | string   | 15           | Bottom padding
leftPadding       | string   | 20           | Left padding
rightPadding      | string   | 20           | Right padding
height            | string   | 150          | SVG element height
width             | string   | 960          | SVG element width
dotSize           | string   | 20           | Dot diameter
dotStrokeColor    | string   | 'black'      | Dot stroke color
dotStrokeWidth    | string   | 2            | Dot stroke width
dotTextSize       | string   | 12           | Dot text size
dotFill           | string   | 'white'      | Dot fill color
dotText           | Function | (dot) => ''  | Returns the text for given dot
disabledOpacity   | string   | 0.9          | Opacity level for disabled dots
showFretNumbers   | string   | true         | Show fret numbers if true
fretNumbersHeight | string   | 40           | Fret numbers container height
fretNumbersMargin | string   | 20           | Fret number container top margin
fretNumbersColor  | string   | '#00000099'  | Fret numbers color
font              | string   | 'Arial'      | Text font
crop              | boolean  | false        | If `true`, crops the rendering. Must be used in conjunction with `fretCount`, so set it to a value enough to contain your diagram (3/4 for a chord, 5/6 for a scale box for instance)
fretLeftPadding   | number   | 0            | Amount of empty frets to display before dots.

## Fretboard API

The `Fretboard` object has the following methods:

### render()

```typescript
render(positions: Position[]): Fretboard
```

Displays the passed positions on the fretboard. Returns the instance itself.

### style()

```typescript
style({
  filter = (): boolean => true,
  text,
  fontSize,
  fontFill,
  ...opts  
}: {
  filter?: (position: Position) => boolean;
  text?: (position: Position) => string;
  fontSize?: number;
  fontFill?: string;
  [key: string]: string | number | Function;  
}): Fretboard
```

Applies the passed properties to selected positions (via the `filter` function parameter). If no filter is provided, all positions are affected. Returns the instance itself. Example:

```typescript
const fretboard = new Fretboard();

// the box positions contain the note name and the interval from the root
const box = CAGED({
  mode: 'major',
  root: 'C3',
  box: 'C'
});

fretboard.render(box);
fretboard.style({
  // this gives us just the root notes
  filter: ({ interval }) => interval === '1P',
  // displays the note name
  text: ({ note }) => note, 
  // sets the value of the fill attribute
  fill: ({ interval }) => interval === '1P' ? 'red' : 'white' 
})
```

### renderChord()

```typescript
render(chord: string): Fretboard
```

Shorthand for rendering positions from a chord voicing string, e.g. `x32010` for a C Major in open position.

The string is mapped onto the fretboard starting from the bottom string, in this case:

- `x`: 6th string muted;
- `3`: 5th string 3rd fret;
- `2`: 4th string 2nd fret;
- `0`: 3rd string open;
- `1`: 2nd string 1st fret;
- `0`: 1st string open.

Examples:

```typescript
// renders an open C major
const fretboard = new Fretboard({
  fretCount: 3,
  showFretNumbers: false
});

fretboard.renderChord('x32010');

// renders the Hendrix chord, displaying only frets 6, 7, 8
const fretboard = new Fretboard({
  fretCount: 3,
  showFretNumbers: true,
  crop: true
});

fretboard.renderChord('x7678x');
```

**Note:** for frets above the 9th, the dash-splitted-notation should be used in order to prevent parsing ambiguity - for instance `10-x-10-10-8-x` for a `Dmadd11` chord.

### muteStrings()

```typescript
muteStrings({
  strings,
  width,
  strokeWidth,
  stroke
}: {
  strings: number[];
  width?: number;
  strokeWidth?: number;
  stroke?: string;
}): Fretboard
```

Marks passed strings with a cross, e.g. `fretboard.muteStrings({ strings: [1, 6]})`.

## Events

You can listen to `click` and `mousemove` events on a fretboard instance. The callback function will be invoked with the corresponding `Position` (string/fret number).

```typescript
fretboard.on(eventName, (position: Position) => void)
```

For example:

```typescript
// this renders a dot following the mouse coordinates
const fretboard = new Fretboard();
fretboard.render([]);
fretboard.on('mousemove', position => fretboard.render([position]));

// you can remove the eventListeners with
fretboard.removeEventListeners();
```

## FAQ

> Why don't you provide a more expressive API like .highlightMajorTriads()?

The aim of this library is to be as abstract as possible, and to make no assumptions besides the bare string/fret positioning. Since you can pass as many properties as you want to the position entries, you can provide full controlled and rich visualisations.