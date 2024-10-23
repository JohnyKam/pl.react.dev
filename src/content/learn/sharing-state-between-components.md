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

#### Controlled and uncontrolled components {/*controlled-and-uncontrolled-components*/}

It is common to call a component with some local state "uncontrolled". For example, the original `Panel` component with an `isActive` state variable is uncontrolled because its parent cannot influence whether the panel is active or not.

In contrast, you might say a component is "controlled" when the important information in it is driven by props rather than its own local state. This lets the parent component fully specify its behavior. The final `Panel` component with the `isActive` prop is controlled by the `Accordion` component.

Uncontrolled components are easier to use within their parents because they require less configuration. But they're less flexible when you want to coordinate them together. Controlled components are maximally flexible, but they require the parent components to fully configure them with props.

In practice, "controlled" and "uncontrolled" aren't strict technical terms--each component usually has some mix of both local state and props. However, this is a useful way to talk about how components are designed and what capabilities they offer.

When writing a component, consider which information in it should be controlled (via props), and which information should be uncontrolled (via state). But you can always change your mind and refactor later.

</DeepDive>

## A single source of truth for each state {/*a-single-source-of-truth-for-each-state*/}

In a React application, many components will have their own state. Some state may "live" close to the leaf components (components at the bottom of the tree) like inputs. Other state may "live" closer to the top of the app. For example, even client-side routing libraries are usually implemented by storing the current route in the React state, and passing it down by props!

**For each unique piece of state, you will choose the component that "owns" it.** This principle is also known as having a ["single source of truth".](https://en.wikipedia.org/wiki/Single_source_of_truth) It doesn't mean that all state lives in one place--but that for _each_ piece of state, there is a _specific_ component that holds that piece of information. Instead of duplicating shared state between components, *lift it up* to their common shared parent, and *pass it down* to the children that need it.

Your app will change as you work on it. It is common that you will move state down or back up while you're still figuring out where each piece of the state "lives". This is all part of the process!

To see what this feels like in practice with a few more components, read [Myślenie reactowe](/learn/thinking-in-react).

<Recap>

* When you want to coordinate two components, move their state to their common parent.
* Then pass the information down through props from their common parent.
* Finally, pass the event handlers down so that the children can change the parent's state.
* It's useful to consider components as "controlled" (driven by props) or "uncontrolled" (driven by state).

</Recap>

<Challenges>

#### Synced inputs {/*synced-inputs*/}

These two inputs are independent. Make them stay in sync: editing one input should update the other input with the same text, and vice versa. 

<Hint>

You'll need to lift their state up into the parent component.

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  return (
    <>
      <Input label="First input" />
      <Input label="Second input" />
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

Move the `text` state variable into the parent component along with the `handleChange` handler. Then pass them down as props to both of the `Input` components. This will keep them in sync.

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
        label="First input"
        value={text}
        onChange={handleChange}
      />
      <Input
        label="Second input"
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

#### Filtering a list {/*filtering-a-list*/}

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
