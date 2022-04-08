---
id: strict-mode
title: Tryb rygorystyczny
permalink: docs/strict-mode.html
---

`StrictMode` jest narzędziem podkreślającym potencjalne problemy w aplikacji. Tak samo jak `Fragment`, `StrictMode` nie renderuje żadnego widocznego UI. Służy natomiast do aktywacji dodatkowych sprawdzeń i ostrzeżeń dla swoich potomków.

> Uwaga:
>
> Sprawdzenia dla trybu rygorystycznego są uruchamiane wyłącznie w trybie deweloperskim; _nie mają one wpływu na build produkcyjny_.

Możesz uruchomić tryb rygorystyczny dla jakiejkolwiek części swojej aplikacji. Dla przykładu:
`embed:strict-mode/enabling-strict-mode.js`

W powyższym przykładzie sprawdzenia trybu rygorystycznego *nie* zostaną uruchomione dla komponentów `Header` i `Footer`. Zadziałają one natomiast dla `ComponentOne` i `ComponentTwo` oraz wszystkich ich potomków.

`StrictMode` aktualnie pomaga w:
* [Identyfikacji komponentów używających niebezpiecznych metod cyklu życia komponentu](#identifying-unsafe-lifecycles)
* [Ostrzeganiu o użyciu przestarzałego API tekstowych referencji](#warning-about-legacy-string-ref-api-usage)
* [Ostrzeganiu o użyciu przestarzałego findDOMNode](#warning-about-deprecated-finddomnode-usage)
* [Wykrywaniu nieoczekiwanych efektów ubocznych](#detecting-unexpected-side-effects)
* [Wykrywaniu użycia przestrzałego API kontekstów](#detecting-legacy-context-api)
* [Wykrywaniu niebezpiecznych efektów](#detecting-unsafe-effects)

Dodatkowe funkcjonalności zostaną dodane w przyszłych wydaniach Reacta.

### Identyfikacja niebezpiecznych metod cyklu życia komponentu {#identifying-unsafe-lifecycles}

Tak jak zostało to wytłumaczone [w tym poście](/blog/2018/03/27/update-on-async-rendering.html), niektóre stare metody cyklu życia komponentu nie są bezpiecznie dla asynchronicznych aplikacji reactowych. Jednakże jeżeli twoja aplikacja używa bibliotek firm trzecich, upewnienie się, że nie używa ona niebezpiecznych metod cyklu życia komponentu może być trudne. Na szczęście, tryb rygorystyczny może z tym pomóc!

Gdy tryb rygorystyczny jest włączony, React tworzy listę wszystkich komponentów klasowych, które używają niebezpiecznych metod cyklu życia, i loguje komunikaty ostrzegawcze z informacją o tych komponentach, jak na przykładzie:

![](../images/blog/strict-mode-unsafe-lifecycles-warning.png)

Rozwiązanie _teraz_ problemów zidentyfikowanych przez tryb rygorystyczny ułatwi użycie współbieżnego renderowania w przyszłych wydaniach Reacta.

### Ostrzeganiu o użyciu przestarzałego API tekstowych referencji {#warning-about-legacy-string-ref-api-usage}

Poprzednio React umożliwiał zarządzanie referencjami na dwa sposoby: przestarzałe referencje tekstowe i funkcje zwrotne. Pomimo że referencja tekstowa była z tych dwóch wygodniejsza, miała [kilka wad](https://github.com/facebook/react/issues/1373), dlatego też naszą oficjalną rekomendacją było [używanie formy funkcji zwrotnej](/docs/refs-and-the-dom.html#legacy-api-string-refs).

React 16.3 wprowadził trzecią opcję, która oferuję wygodę jak w przypadku referencji tekstowej, bez żadnych wad:
`embed:16-3-release-blog-post/create-ref-example.js`

Z uwagi na fakt, iż referencje obiektowe zostały dodane głównie jako zamiennik za referencje tekstowe, tryb rygorystyczny obecnie ostrzega w przypadku użyciu referencji tekstowych.

> **Uwaga:**
>
> Referencje w formie funkcji zwrotnej nadal będą wspierane wraz z nowym interfejsem `createRef`.
>
> Nie musisz zamieniać referencji w postaci funkcji zwrotnej w swoich komponentach. Są one nieco bardziej elastyczne, więc pozostaną jako zaawansowana funkcjonalność.

[Dowiedz się więcej o nowym API `createRef` tutaj.](/docs/refs-and-the-dom.html)

### Ostrzeganie o użyciu przestarzałego findDOMNode {#warning-about-deprecated-finddomnode-usage}

React wspierał `findDOMNode`, aby umożliwić wyszukanie węzła powiązanego z daną instancją klasy w drzewie DOM. Zwykle jest to zbędna funkcjonalność, gdyż możesz [podczepić referencję bezpośrednio do węzła DOM](/docs/refs-and-the-dom.html#creating-refs).

`findDOMNode` mógł również być używany na komponencie klasy, jednak powodowało to zakłócenie poziomów abstrakcji poprzez umożliwienie rodzicowi, aby wymagał wyrenderowania się pewnego potomka. Tworzyło to ryzyko podczas tzw. "refactoringu", ponieważ nie można było zmienić implementacji komponentu, z węzłów którego mógłby korzystać rodzic. `findDOMNode` zawsze zwraca pierwsze dziecko, ale w przypadku użycia fragmentów jest możliwe, że komponent wyrenderuje wiele węzłów DOM. `findDOMNode` pozwala na jednorazowy odczyt, zwracając wynik tylko na żądanie. Jeżeli komponent potomny wyrenderuje inny węzeł, nie ma sposobu na obsłużenie tej zmiany. Innymi słowy `findDOMNode` działa wyłącznie gdy komponenty zwracają zawsze pojedynczy węzeł DOM, który nigdy się nie zmienia.

Zamiast tego do komponentu możesz jawnie przekazać referencję, którą następnie przekażesz do drzewa DOM przy użyciu [przekierowania referencji](/docs/forwarding-refs.html#forwarding-refs-to-dom-components).

Możesz również dodać do swojego komponentu opakowujący węzeł DOM i bezpośrednio do niego dowiązać referencję.

```javascript{4,7}
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.wrapper = React.createRef();
  }
  render() {
    return <div ref={this.wrapper}>{this.props.children}</div>;
  }
}
```

> Uwaga:
>
> W CSS możesz użyć atrybutu [`display: contents`](https://developer.mozilla.org/en-US/docs/Web/CSS/display#display_contents), jeżeli nie chcesz, aby węzeł nie był częścią szablonu.

### Wykrywanie nieoczekiwanych efektów ubocznych {#detecting-unexpected-side-effects}

Zasadniczo, React działa w dwóch fazach:
* Faza **renderowania** określa, jakie zmiany należy zaaplikować w np. drzewie DOM. Podczas tej fazy React wywołuje metodę `render` i porównuje jej wynik z poprzednim.
* Faza **aktualizacji** następuje wtedy, gdy React aplikuje zmiany. (W przypadku React DOM następuje to, gdy React dodaje, aktualizuje i usuwa węzły DOM.) Podczas tej fazy React wywołuje również metody cyklu życia komponentu tj. `componentDidMount` czy `componentDidUpdate`.

Faza aktualizacji jest zazwyczaj bardzo szybka, jednak renderowanie może być powolne. Z tego powodu nadchodzący tryb współbieżny (który nie jest jeszcze domyślnie włączony), rozbija pracę związaną z renderowaniem na części, zatrzymując i wznawiając pracę, aby uniknąć blokowania przeglądarki. To oznacza, że React może wywołać metody cyklu życia w fazie renderowania więcej niż raz przed aktualizacją lub może je wywołać nawet bez aktualizacji (z powodu błędu lub przerwania o wyższym priorytecie).

Cykl życia fazy renderowania odnosi się do poniższych metod z komponentu klasowego:
* `constructor`
* `componentWillMount` (or `UNSAFE_componentWillMount`)
* `componentWillReceiveProps` (or `UNSAFE_componentWillReceiveProps`)
* `componentWillUpdate` (or `UNSAFE_componentWillUpdate`)
* `getDerivedStateFromProps`
* `shouldComponentUpdate`
* `render`
* funkcje aktualizujące dla `setState` (pierwszy argument)

Ponieważ powyższe metody mogą być wywołane więcej niż raz, ważne jest, aby nie zawierały efektów ubocznych (ang. *side-effects*). Zignorowanie tej zasady może prowadzić do różnych problemów, włączając w to wycieki pamięci i niepoprawny stan aplikacji. Niestety, może być ciężko wykryć problemy tego typu, ponieważ są one często [niedeterministyczne](https://pl.wikipedia.org/wiki/Algorytm_deterministyczny).

Tryb rygorystyczny nie zapewni automatycznego wykrywania efektów ubocznych, ale może pomóc w ich zauważeniu poprzez sprawienie, by były trochę bardziej deterministyczne. Dzieje się to za sprawą celowego podwójnego wywoływania poniższych funkcji:

* Metod `constructor`, `render` oraz `shouldComponentUpdate` komponentu klasowego
* Metody statycznej `getDerivedStateFromProps` komponentu klasowego
* Ciała komponentu funkcyjnego
* Funkcji aktualizującej stanu (pierwszy argument `setState`)
* Funkcji przekazywanych do `useState`, `useMemo` oraz `useReducer`

> Uwaga:
>
> Ma to zastosowanie tylko w trybie deweloperskim. _Metody życia komponentu nie będą podwójnie wywoływane w trybie produkcyjnym._

Dla przykładu, rozważ poniższy kod:
`embed:strict-mode/side-effects-in-constructor.js`

Na pierwszy rzut oka kod nie wydaje się problematyczny. Ale jeżeli `SharedApplicationState.recordEvent` nie jest [idempotentna](https://pl.wikipedia.org/wiki/Idempotentno%C5%9B%C4%87#Informatyka), to stworzenie wielu instancji tego komponentu może prowadzić do niepoprawnego stanu aplikacji. Ten subtelny błąd może się nie ukazać podczas dewelopmentu lub może występować sporadycznie i zostać przeoczony.

Przez celowe podwójne wywołanie metod, takich jak konstruktor komponentu, tryb rygorystyczny sprawia, że wzorce tego typu są łatwiejsze do zauważenia.

> Uwaga:
>
> Począwszy od wersji 17, React automatycznie modyfikuje metody takie jak `console.log()`, aby uciszyć logi przy powtórnym wywołaniu funkcji odpowiedzialnych za cykl życia komponentów. Jednak w niektórych przypadkach może to prowadzić do niepożądanych zachowań, [na co można zaradzić w ten sposób](https://github.com/facebook/react/issues/20090#issuecomment-715927125).
>
> Od wersji React 18, React nie ucisza żadnych logów. Jeśli jednak korzystasz z React DevTools, logi z drugiego wywołania będą nieco bardziej wyszarzone. React DevTools posiada również opcję (domyślnie wyłączoną), która pozwala całkowicie je uciszyć.

### Wykrywanie przestarzałego API kontekstów {#detecting-legacy-context-api}

Przestarzałe API kontekstów jest podatne na błędy i może zostać usunięte w przyszłych wersjach. Nadal będzie działać dla wszystkich wydań 16.x, ale w trybie rygorystycznym będzie w nich wyświetlany poniższy komunikat ostrzegawczy:

![](../images/blog/warn-legacy-context-in-strict-mode.png)

Przeczytaj [dokumentację nowego API kontekstów](/docs/context.html), aby dowiedzieć się, jak zmigrować do nowej wersji.


### Ensuring reusable state {#ensuring-reusable-state}

In the future, we’d like to add a feature that allows React to add and remove sections of the UI while preserving state. For example, when a user tabs away from a screen and back, React should be able to immediately show the previous screen. To do this, React support remounting trees using the same component state used before unmounting.

This feature will give React better performance out-of-the-box, but requires components to be resilient to effects being mounted and destroyed multiple times. Most effects will work without any changes, but some effects do not properly clean up subscriptions in the destroy callback, or implicitly assume they are only mounted or destroyed once.

To help surface these issues, React 18 introduces a new development-only check to Strict Mode. This new check will automatically unmount and remount every component, whenever a component mounts for the first time, restoring the previous state on the second mount.

To demonstrate the development behavior you'll see in Strict Mode with this feature, consider what happens when React mounts a new component. Without this change, when a component mounts, React creates the effects:

```
* React mounts the component.
  * Layout effects are created.
  * Effects are created.
```

With Strict Mode starting in React 18, whenever a component mounts in development, React will simulate immediately unmounting and remounting the component:

```
* React mounts the component.
    * Layout effects are created.
    * Effect effects are created.
* React simulates effects being destroyed on a mounted component.
    * Layout effects are destroyed.
    * Effects are destroyed.
* React simulates effects being re-created on a mounted component.
    * Layout effects are created
    * Effect setup code runs
```

On the second mount, React will restore the state from the first mount. This feature simulates user behavior such as a user tabbing away from a screen and back, ensuring that code will properly handle state restoration.

When the component unmounts, effects are destroyed as normal:

```
* React unmounts the component.
  * Layout effects are destroyed.
  * Effect effects are destroyed.
```

> Note:
>
> This only applies to development mode, _production behavior is unchanged_.

For help supporting common issues, see:
  - [How to support Reusable State in Effects](https://github.com/reactwg/react-18/discussions/18)
