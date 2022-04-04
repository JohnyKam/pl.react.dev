---
id: rendering-elements
title: Renderowanie elementów
permalink: docs/rendering-elements.html
redirect_from:
  - "docs/displaying-data.html"
prev: introducing-jsx.html
next: components-and-props.html
---

Elementy to najmniejsze bloki budujące reactowe aplikacje.

Element opisuje, co chcesz zobaczyć na ekranie:

```js
const element = <h1>Witaj, świecie!</h1>;
```

W przeciwieństwie do elementów drzewa DOM w przeglądarce, reactowe elementy są zwykłymi obiektami i mają niski koszt tworzenia. React DOM zajmuje się aktualizowaniem drzewa DOM tak, aby odpowiadało strukturze elementów reactowych.


>**Wskazówka:**
>
>Łatwo pomylić elementy z szerzej znaną koncepcją "komponentów". Komponenty przedstawimy w [kolejnym rozdziale](/docs/components-and-props.html). Elementy są tym, z czego komponenty "są zbudowane". Zachęcamy do przeczytania tego rozdziału przed przejściem dalej.

## Renderowanie elementu w drzewie DOM {#rendering-an-element-into-the-dom}

Powiedzmy, że gdzieś w twoim pliku HTML jest `<div>`:

```html
<div id="root"></div>
```

Ten węzeł drzewa DOM nazywamy "korzeniem", bo wszystko, co się w nim znajduje będzie zarządzane przez React DOM.

Aplikacje zbudowane przy pomocy samego Reacta zazwyczaj posiadają pojedynczy węzeł drzewa DOM. Jeśli natomiast integrujesz reactową aplikację z już istniejącą aplikacją, możesz mieć tyle odizolowanych "korzeni", ile chcesz.

<<<<<<< HEAD
Aby wyrenderować reactowy element w węźle drzewa DOM, przekaż oba do [`ReactDOM.render()`](/docs/react-dom.html#render):
=======
To render a React element, first pass the DOM element to [`ReactDOM.createRoot()`](/docs/react-dom-client.html#createroot), then pass the React element to `root.render()`:
>>>>>>> 707f22d25f5b343a2e5e063877f1fc97cb1f48a1

`embed:rendering-elements/render-an-element.js`

**[Przetestuj kod na CodePen](https://codepen.io/gaearon/pen/ZpvBNJ?editors=1010)**

Na stronie wyświetli się napis "Witaj, świecie!".

## Aktualizowanie wyrenderowanego elementu {#updating-the-rendered-element}

Reactowe elementy są [niezmienne](https://en.wikipedia.org/wiki/Immutable_object) (ang. *immutable*). Kiedy już stworzysz element, nie możesz zmienić jego komponentów potomnych ani właściwości. Element jest jak pojedyncza klatka z filmu: reprezentuje interfejs użytkownika w pewnym punkcie czasu.

<<<<<<< HEAD
Przy naszej dotychczasowej wiedzy, jedynym sposobem aktualizacji interfejsu użytkownika jest stworzenie nowego elementu i przekazanie go do [`ReactDOM.render()`](/docs/react-dom.html#render).
=======
With our knowledge so far, the only way to update the UI is to create a new element, and pass it to `root.render()`.
>>>>>>> 707f22d25f5b343a2e5e063877f1fc97cb1f48a1

Rozważ ten przykład tykającego zegara:

`embed:rendering-elements/update-rendered-element.js`

**[Przetestuj kod na CodePen](https://codepen.io/gaearon/pen/gwoJZk?editors=1010)**

<<<<<<< HEAD
Wywołuje on [`ReactDOM.render()`](/docs/react-dom.html#render) z wewnątrz funkcji zwrotnej [`setInterval()`](https://developer.mozilla.org/pl/docs/Web/API/Window/setInterval) co sekundę.
=======
It calls [`root.render()`](/docs/react-dom.html#render) every second from a [`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) callback.
>>>>>>> 707f22d25f5b343a2e5e063877f1fc97cb1f48a1

>**Wskazówka:**
>
<<<<<<< HEAD
>W praktyce większość reactowych aplikacji wywołuje [`ReactDOM.render()`](/docs/react-dom.html#render) tylko raz. W kolejnych rozdziałach dowiemy się, jak można taki kod wyizolować do [komponentów stanowych](/docs/state-and-lifecycle.html).
=======
>In practice, most React apps only call `root.render()` once. In the next sections we will learn how such code gets encapsulated into [stateful components](/docs/state-and-lifecycle.html).
>>>>>>> 707f22d25f5b343a2e5e063877f1fc97cb1f48a1
>
>Radzimy jednak nie pomijać żadnych tematów, ponieważ kolejne rozdziały oparte są o wiedzę z poprzednich.

## React uaktualnia tylko to, co potrzebne {#react-only-updates-whats-necessary}

React DOM porównuje element i jego potomków do poprzedniego oraz nakłada tylko te aktualizacje drzewa DOM, które konieczne są do doprowadzenia go do pożądanego stanu.

Możesz to sprawdzić przez zbadanie (ang. *inspect*) [ostatniego przykładu](https://codepen.io/gaearon/pen/gwoJZk?editors=1010) przy użyciu narzędzi deweloperskich:

![inspektor DOM pokazujący cząstkowe aktualizacje](../images/docs/granular-dom-updates.gif)

Mimo że przy każdym tyknięciu zegara tworzymy element opisujący cały interfejs użytkownika, tylko węzeł tekstowy, którego treść uległa zmianie, zostaje zaktualizowany przez React DOM.

Według naszego doświadczenia, myślenie o tym, jak powinien wyglądać interfejs użytkownika w danym momencie, a nie jak zmieniać go w czasie, eliminuje całą klasę błędów.
