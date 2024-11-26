---
title: Współdzielenie stanu między komponentami
---

<Intro>

Zdarzają się sytuacje, kiedy stan dwóch komponentów musi zawsze zmieniać się w tym samym czasie. W tym celu należy usunąć wspólny stan z obydwu komponentów, następnie dodać go do ich wspólnego przodka, a na koniec przekazać go do komponentów poprzez właściwości (*ang.* props). Taki zabieg określa się mianem "wynoszenia stanu w górę" i jest jedną z najczęstszych czynności wykonywanych podczas tworzenia aplikacji reactowych.

</Intro>

<YouWillLearn>

- Jak "wynieść stan do góry", aby współdzielić go między komponentami
- Co to jest komponent kontrolowany i komponent niekontrolowany

</YouWillLearn>

## Wynoszenia stanu w górę {/*lifting-state-up-by-example*/}

W poniższym przykładzie komponent przodka `Accordion` renderuje dwa komponenty `Panel`:

* `Accordion`
  - `Panel`
  - `Panel`

Każdy komponent `Panel` posiada stan `isActive`, który przechowuje wartość logiczną. Stan `isActive` określa czy zawartość komponentu jest widoczna

Naciśnij przycisk "Pokaż" dla obu paneli:

<Sandpack>

```js
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Pokaż
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="O mieście">
        Jego populacja wynosi ponad 2 miliony, co sprawia, że Almaty jest największym miastem w Kazachstanie. W latach 1929-1997 był stolicą kraju.
      </Panel>
      <Panel title="Etymologia">
        Nazwa pochodzi od <span lang="kk-KZ">алма</span>, kazachskiego słowa oznaczającego "jabłko", i najczęściej tłumaczona jest jako "pełne jabłek". Region otaczający Almaty podobno jest domem pradawnych odmian jabłek, a dziko tu rosnąca <i lang="la">Malus sieversii</i> uważana jest za przodka współczesnej jabłoni domowej.
      </Panel>
    </>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Zauważ, że naciśnięcie przycisku na jednym panelu nie ma wpływu na drugi panel – są one niezależne.

<DiagramGroup>

<Diagram name="sharing_state_child" height={367} width={477} alt="Diagram przedstawiający drzewo trzech komponentów, jednego komponentu  przodka oznaczonego jako Accordion i dwóch komponentów potomnych oznaczonych jako Panel. Oba komponenty Panel zawierają stan isActive z wartością false.">

Początkowo stan `isActive` każdego `Panelu` jest równy `false`, więc oba są zwinięte

</Diagram>

<Diagram name="sharing_state_child_clicked" height={367} width={480} alt="Ten sam diagram co poprzedni. W pierwszym komponencie potomnym Panel jest wyróżniony, wskazując na kliknięcie i zmianę stanu isActive na wartość true. W drugim Panelu w dalszym ciągu stan isActive ma wartość false." >

Kliknięcie dowolnego przycisku `Panelu` spowoduje jedynie aktualizację stanu `isActive` tego `Panelu`

</Diagram>

</DiagramGroup>

**Ale teraz powiedzmy, że chcesz to zmienić tak, aby w danym momencie rozwinięty był tylko jeden panel.** W taki sposób, że rozwinięcie jednego panelu powinno zwinąć drugi. Jak to zrobić?

Aby kontrolować te dwa panele, musisz „wynieść ich stan” do komponentu przodka w trzech krokach:

1. **Usuń** stan z komponentów potomnych.
2. **Przekaż** dane z wspólnego komponentu przodka.
3. **Dodaj** stan do wspólnego przodka i przekaż go razem z procedura obsługi zdarzeń.

Pozwoli to komponentowi `Accordion` na sterowanie komponentami `Panel` i rozwinięcie tylko jednego z nich w danym momencie.

### Krok 1: Usuń stan z komponentów potomnych {/*step-1-remove-state-from-the-child-components*/}

Przekażesz kontrolę nad stanem `isActive` komponentu `Panel` do komponentu przodka. W zamian przodek przekaże właściwość `isActive` do `Panelu`. Zacznij od **usunięcia  tej linii** z komponentu `Panel`:

```js
const [isActive, setIsActive] = useState(false);
```

Następnie dodaj `isActive` do właściwości `Panelu`:

```js
function Panel({ title, children, isActive }) {
```

W ten sposób komponent przodka `Panelu` *kontroluje* właściwości `isActive` poprzez [przekazywanie wartości do komponentu.](/learn/passing-props-to-a-component) Natomiast komponent `Panel` *nie ma kontroli* nad wartością `isActive`--teraz zależy to od komponentu przodka!

### Krok 2: Przekaż dane ze wspólnego przodka {/*step-2-pass-hardcoded-data-from-the-common-parent*/}

Aby wynieść stan w górę, musisz znaleźć najbliższy wspólny komponent przodka *obu* komponentów potomnych, które chcesz skoordynować:

* `Accordion` *(najbliższy wspólny przodek)*
  - `Panel`
  - `Panel`

W tym przykładzie jest to komponent `Accordion`. Ponieważ znajduje się on powyżej obu paneli i może kontrolować ich właściwości, stanie się "źródłem prawdy" dla tego, który panel jest aktualnie aktywny. Zmień komponent `Accordion` tak aby przekazywał zakodowaną na stałe właściwość `isActive` (na przykład `true`) do obu paneli:

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="O mieście" isActive={true}>
        Jego populacja wynosi ponad 2 miliony, co sprawia, że Almaty jest największym miastem w Kazachstanie. W latach 1929-1997 był stolicą kraju.
      </Panel>
      <Panel title="Etymologia" isActive={true}>
        Nazwa pochodzi od <span lang="kk-KZ">алма</span>, kazachskiego słowa oznaczającego "jabłko", i najczęściej tłumaczona jest jako "pełne jabłek". Region otaczający Almaty podobno jest domem pradawnych odmian jabłek, a dziko tu rosnąca <i lang="la">Malus sieversii</i> uważana jest za przodka współczesnej jabłoni domowej.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Pokaż
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Pozmieniaj wartość właściwości `isActive` w komponencie `Accordion` i zobacz jaki jest efekt tych zmian.

### Krok 3: Dodanie stanu komponentu do wspólnego przodka {/*step-3-add-state-to-the-common-parent*/}

Wynoszenie stanu w górę często zmienia sposób w jaki będzie zarządzany stan komponentów.

Założenie jest takie, że tylko jeden panel powinien być aktywny w danym momencie. Oznacza to, że wspólny komponent nadrzędny `Accordion` musi śledzić *który* panel jest aktywny. Zamiast wartości `boolean` można użyć liczby, która odpowiada wartości indeksu aktywnego `Panelu` dla zmiennej stanu:

```js
const [activeIndex, setActiveIndex] = useState(0);
```

Gdy `activeIndex` wynosi `0`, pierwszy panel jest aktywny, a gdy wynosi `1`, aktywny jest drugi panel.

Naciśnięcie przycisku "Pokaż" w dowolnym `Panelu` musi zmienić aktywny indeks w `Accordion`. `Panel` nie może bezpośrednio ustawić stanu `activeIndex`, ponieważ jest on zdefiniowany wewnątrz `Accordion`. Komponent `Accordion` musi *wyraźnie zezwolić* komponentowi `Panel` na zmianę swojego stanu poprzez [przekazanie procedury obsługi zdarzeń jako właściwości](/learn/responding-to-events#passing-event-handlers-as-props):

```js
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```

Do przycisk `<button>` wewnątrz komponentu `Panel` należy również dodać właściwość `onShow`, która przekaże obsługę zdarzenia kliknięcia:

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="O mieście"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        Jego populacja wynosi ponad 2 miliony, co sprawia, że Almaty jest największym miastem w Kazachstanie. W latach 1929-1997 był stolicą kraju.
      </Panel>
      <Panel
        title="Etymologia"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        Nazwa pochodzi od <span lang="kk-KZ">алма</span>, kazachskiego słowa oznaczającego "jabłko", i najczęściej tłumaczona jest jako "pełne jabłek". Region otaczający Almaty podobno jest domem pradawnych odmian jabłek, a dziko tu rosnąca <i lang="la">Malus sieversii</i> uważana jest za przodka współczesnej jabłoni domowej.
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Pokaż
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

To kończy proces wynoszenia stanu w górę! Przeniesienie stanu do wspólnego przodka pozwoliło koordynowanie stanem dwóch paneli. Użycie indeksu zamiast dwóch flag "jest pokazany" zapewniło, że w danym momencie aktywny jest tylko jeden panel. Zas przekazanie procedury obsługi zdarzeń do komponentu potomnego pozwoliło mu na zmianę stanu przodka.

<DiagramGroup>

<Diagram name="sharing_state_parent" height={385} width={487} alt="Diagram przedstawiający drzewo trzech komponentów, jednego komponentu przodka oznaczonego jako Accordion i dwóch komponentów potomnych oznaczonych jako Panel. Accordion zawiera stan activeIndex równy zero, który jest przekazywana jako właściwość isActive o wartości true do pierwszego Panelu, oraz jako właściwość isActive o wartości false do drugiego Panelu." >

Początkowo `activeIndex` komponentu `Accordion` wynosi `0`, więc w pierwszym `Panel` właściwość `isActive = true`

</Diagram>

<Diagram name="sharing_state_parent_clicked" height={385} width={521} alt="Ten sam diagram co poprzedni, z wyróżnioną wartością stanu activeIndex komponentu przodka Accordion wskazującą na kliknięcie i zmianę wartości na jeden. Przepływ do obu komponentów potomnych Panel jest również wyróżniony, a wartość właściwości isActive przekazywana do każdego dziecka jest ustawiona na przeciwną: false dla pierwszego Panelu i true dla drugiego." >

Kiedy stan `activeIndex` komponentu `Accordion` zmieni się na `1`, wówczas w drugim `Panel` właściwość `isActive = true`

</Diagram>

</DiagramGroup>

<DeepDive>

#### Komponenty kontrolowane i niekontrolowane {/*controlled-and-uncontrolled-components*/}

Często mówi się, że komponent z lokalnym stanem jest "niekontrolowany". Na przykład, oryginalny komponent `Panel` z zmienną stanu `isActive` jest niekontrolowany, ponieważ jego przodek nie może wpływać na to, czy panel jest aktywny, czy nie.

Z drugiej strony, można powiedzieć, że komponent jest "kontrolowany", gdy ważne informacje w nim są sterowane przez właściwości zamiast jego własnego lokalnego stanu. Pozwala to komponentowi przodka w pełni określić jego zachowanie. Ostatecznie komponent `Panel` z właściwością `isActive` jest kontrolowany przez komponent `Accordion`.

Komponenty niekontrolowane są łatwiejsze w użyciu przez ich przodka, ponieważ wymagają mniej konfiguracji. Jednak są mniej elastyczne, gdy chcesz je ze sobą powiązać. Komponenty kontrolowane są maksymalnie elastyczne, ale wymagają, aby komponenty przodka w pełni je konfigurowały za pomocą przekazywanych właściwości.

W praktyce "kontrolowane" i "niekontrolowane" nie są ścisłymi terminami technicznymi, każdy komponent zazwyczaj ma pewną mieszankę zarówno lokalnego stanu, jak i przekazywanych właściwości. Jednakże, jest to użyteczny sposób określanie tego, jak komponenty są zaprojektowane i jakie możliwości oferują.

Budując komponent, zastanów się, które informacje w nim powinny być kontrolowane (przez przekazanie właściwości), a które informacje powinny być niekontrolowane (przez użycie stanu). Oczywiście zawsze możesz zmienić zdanie i przerobić komponent.

</DeepDive>

## A single source of truth for each state {/*a-single-source-of-truth-for-each-state*/}

W aplikacji React wiele komponentów będzie miało swój własny stan. Niektóre komponenty, mające stan mogą być komponentami znajdującymi się na dole struktury (końcu hierarchii). Niektóre komponenty są sterowane przez stan przechowywany w komponencie na szczycie struktury. Dla przykładu, biblioteki routingu i nawigacji przechowują bieżącą ścieżkę w swoim stanie i przekazują jej wartość innym komponentom poprzez właściwości.

**Musisz wybrać, który komponent będzie "posiadał" stan,** zgodnie z zasadą znaną jako ["jedno źródło prawdy."](https://en.wikipedia.org/wiki/Single_source_of_truth) Nie oznacza to, że wszystkie stany muszą być przechowywane w jednym głównym komponencie – chodzi o to, aby _konkretny_ komponent przechowywał informację o _danym_ stanie. Unikaj duplikowania stanu w różnych komponentach. Zamiast tego, *wynieś stan do góry* do najbliższego wspólnego przodka i *przekaż go w dół* do komponentów potomnych, które go potrzebują.

Twoja aplikacja w trakcie swojego rozwoju będzie się zmieniać. Często może dochodzić do tego, że będziesz przenosić stan w dół lub z powrotem w górę, szukając odpowiedniego komponentu, który ma dany stan przechowywać. Jest to częścią procesu budowy aplikacji! 

Aby zobaczyć, jak to wygląda w praktyce z kilkoma dodatkowymi komponentami, przeczytaj [Myślenie w React](/learn/thinking-in-react).

<Recap>

* Kiedy chcesz kontrolować dwa komponenty, przenieś ich stan do wspólnego przodka.
* Następnie przekaż informacje poprzez właściwości od wspólnego przodka.
* Na koniec przekaż procedury obsługi zdarzeń, aby komponenty potomków mogły zmieniać stan przodka.
* Warto rozważyć komponenty jako "kontrolowane" (sterowane przez właściwości) lub "niekontrolowane" (sterowane przez stan).

</Recap>

<Challenges>

#### Zsynchronizowane pola wejściowe {/*synced-inputs*/}

Te dwa pola wejściowe są niezależne. Spraw, aby były zsynchronizowane: edytowanie jednego pola powinno aktualizować drugie pole tym samym tekstem i odwrotnie.

<Hint>

Musisz wynieść ich stan do komponentu przodka.

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  return (
    <>
      <Input label="Pierwsze pole" />
      <Input label="Drugie pole" />
    </>
  );
}

function Input({ label }) {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <label>
      {label}
      {' '}
      <input
        value={text}
        onChange={handleChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

<Solution>

Przenieś zmienną stanu `text` do komponentu przodka wraz z procedurą obsługi `handleChange`. Następnie przekaż je jako właściwości do obu komponentów `Input`. To pozwoli na ich synchronizację.

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <Input
        label="Pierwsze pole"
        value={text}
        onChange={handleChange}
      />
      <Input
        label="Drugie pole"
        value={text}
        onChange={handleChange}
      />
    </>
  );
}

function Input({ label, value, onChange }) {
  return (
    <label>
      {label}
      {' '}
      <input
        value={value}
        onChange={onChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

</Solution>

#### Filtrowanie listy {/*filtering-a-list*/}

In this example, the `SearchBar` has its own `query` state that controls the text input. Its parent `FilterableList` component displays a `List` of items, but it doesn't take the search query into account.

Use the `filterItems(foods, query)` function to filter the list according to the search query. To test your changes, verify that typing "s" into the input filters down the list to "Sushi", "Shish kebab", and "Dim sum".

Note that `filterItems` is already implemented and imported so you don't need to write it yourself!

<Hint>

You will want to remove the `query` state and the `handleChange` handler from the `SearchBar`, and move them to the `FilterableList`. Then pass them down to `SearchBar` as `query` and `onChange` props.

</Hint>

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  return (
    <>
      <SearchBar />
      <hr />
      <List items={foods} />
    </>
  );
}

function SearchBar() {
  const [query, setQuery] = useState('');

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <label>
      Search:{' '}
      <input
        value={query}
        onChange={handleChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody>
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: 'Sushi',
  description: 'Sushi is a traditional Japanese dish of prepared vinegared rice'
}, {
  id: 1,
  name: 'Dal',
  description: 'The most common way of preparing dal is in the form of a soup to which onions, tomatoes and various spices may be added'
}, {
  id: 2,
  name: 'Pierogi',
  description: 'Pierogi are filled dumplings made by wrapping unleavened dough around a savoury or sweet filling and cooking in boiling water'
}, {
  id: 3,
  name: 'Shish kebab',
  description: 'Shish kebab is a popular meal of skewered and grilled cubes of meat.'
}, {
  id: 4,
  name: 'Dim sum',
  description: 'Dim sum is a large range of small dishes that Cantonese people traditionally enjoy in restaurants for breakfast and lunch'
}];
```

</Sandpack>

<Solution>

Lift the `query` state up into the `FilterableList` component. Call `filterItems(foods, query)` to get the filtered list and pass it down to the `List`. Now changing the query input is reflected in the list:

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  const [query, setQuery] = useState('');
  const results = filterItems(foods, query);

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <>
      <SearchBar
        query={query}
        onChange={handleChange}
      />
      <hr />
      <List items={results} />
    </>
  );
}

function SearchBar({ query, onChange }) {
  return (
    <label>
      Search:{' '}
      <input
        value={query}
        onChange={onChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody> 
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: 'Sushi',
  description: 'Sushi is a traditional Japanese dish of prepared vinegared rice'
}, {
  id: 1,
  name: 'Dal',
  description: 'The most common way of preparing dal is in the form of a soup to which onions, tomatoes and various spices may be added'
}, {
  id: 2,
  name: 'Pierogi',
  description: 'Pierogi are filled dumplings made by wrapping unleavened dough around a savoury or sweet filling and cooking in boiling water'
}, {
  id: 3,
  name: 'Shish kebab',
  description: 'Shish kebab is a popular meal of skewered and grilled cubes of meat.'
}, {
  id: 4,
  name: 'Dim sum',
  description: 'Dim sum is a large range of small dishes that Cantonese people traditionally enjoy in restaurants for breakfast and lunch'
}];
```

</Sandpack>

</Solution>

</Challenges>
