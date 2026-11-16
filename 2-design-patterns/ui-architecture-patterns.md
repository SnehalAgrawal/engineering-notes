# UI Architecture Patterns (MVC, MVP, MVVM)

### 1. Overview

MVC, MVP, and MVVM split a UI app into three parts:

* **Model**: data and business rules.
* **View**: what the user sees.
* **A middle part** (Controller, Presenter, or ViewModel) that connects them.

They differ in who talks to whom and how the View gets updated. They are ideas, not language features. Frameworks lean toward one:

* **React** is a view library. A custom hook that holds state and actions acts like a ViewModel, but there is no binding layer.
* **Angular** components (a class plus a template with data binding) are close to MVVM.
* **Vanilla JS** can do any of them by hand.

The examples all build the same counter: a button that adds one, and a label that shows the count.

### 2. Key Concepts

#### 2.1. MVC (Model-View-Controller)

* **Model**: holds the data and tells listeners when it changes.
* **View**: watches the Model and redraws.
* **Controller**: turns user input into calls on the Model.

Flow: User → Controller → Model → change event → View

```js
class CounterModel {
  constructor() { this.count = 0; this.listeners = []; }
  onChange(fn) { this.listeners.push(fn); }
  increment() {
    this.count++;
    this.listeners.forEach(fn => fn(this.count));
  }
}

class CounterView {
  constructor(model) {
    this.button = document.getElementById("increment");
    this.label = document.getElementById("label");
    model.onChange(count => (this.label.textContent = count)); // View watches Model
  }
  bindClick(handler) { this.button.addEventListener("click", handler); }
}

class CounterController {
  constructor(model, view) {
    view.bindClick(() => model.increment()); // Controller only handles input
  }
}
```

Server-side web MVC (Rails, ASP.NET MVC, Spring MVC) is a variation. The controller handles an HTTP request, works with the Model, and picks a template to render. Django uses the same split under the name MTV (model, template, view).

#### 2.2. MVP (Model-View-Presenter)

* **View**: passive. It forwards clicks and shows what it is told. It never reads the Model.
* **Presenter**: all display logic. It gets events from the View, updates the Model, and sends ready-to-show values back.

Flow: User → View → Presenter → Model, then Presenter → View

```js
class CounterPresenter {
  constructor(view, model) {
    this.view = view;   // any object with bindClick() and showCount()
    this.model = model;
    view.bindClick(() => {
      model.count++;
      this.refresh();
    });
    this.refresh();
  }

  refresh() { this.view.showCount(`Count: ${this.model.count}`); }
}
```

The Presenter only knows the View's small interface. Tests can pass a fake View and check the Presenter without a browser. Common in WinForms apps and in Android apps written before Jetpack's ViewModel.

#### 2.3. MVVM (Model-View-ViewModel)

* **ViewModel**: exposes state and actions shaped for the View. It has no reference to the View.
* **View**: binds to the ViewModel. When ViewModel state changes, the View updates by itself. Nobody calls `render()`.

Flow: View ⇄ ViewModel (data binding, often two-way) → Model

MVVM came from Microsoft's WPF. It is common with Android Jetpack ViewModel, SwiftUI, and Vue or Angular templates.

```jsx
function useCounterViewModel() {          // ViewModel: state and actions, no markup
  const [count, setCount] = useState(0);
  return { count, increment: () => setCount(c => c + 1) };
}

function CounterView() {                  // View: binds to the ViewModel
  const { count, increment } = useCounterViewModel();
  return <button onClick={increment}>{count}</button>;
}
```

#### 2.4. Comparison

| | MVC | MVP | MVVM |
| --- | --- | --- | --- |
| Who updates the View | View watches Model | Presenter calls View | Data binding |
| View knows the Model? | Yes | No | No, only the ViewModel |
| What is easy to test | Controller | Presenter, with a fake View | ViewModel, with no UI |
| Typical fit | Server-side web frameworks | Desktop, older Android | Frameworks with data binding |

#### 2.5. Related Architectures

* **Flux / Redux**: data flows one way through a single store. Flux: Action → Dispatcher → Store → View. Redux has no dispatcher: `dispatch(action)` → reducer (a pure function) → new state → UI.
* **Component-based**: the UI is a tree of components that each own their rendering and local state. The default in React, Vue, and Svelte.
* **Clean Architecture**: layers in rings (entities, use cases, adapters, frameworks). Dependencies point inward only, so core logic never imports UI or database code.
* **Hexagonal (ports and adapters)**: core logic defines ports (interfaces); UI, DB, and APIs plug in as adapters. Common in domain-driven design (DDD).
* **Reactive / signals**: the UI subscribes to streams or signals and redraws on change (RxJS, MobX, SolidJS).
* **State machines**: UI behavior as explicit states and transitions. Good for wizards, multi-step forms, and chatbots (for example XState).
* **Micro frontends**: a large frontend split into apps that separate teams build and deploy on their own (for example with Webpack Module Federation). It solves a team problem, not a code problem.

### 3. Real-World Usage

* **Server-rendered web apps**: MVC, with a controller per route and templates as Views.
* **Desktop and mobile apps**: MVVM with the platform's binding (WPF, Jetpack, SwiftUI).
* **Single-page apps**: components, with a store (Redux, Pinia) or hooks acting as ViewModels.
* **Complex flows**: state machines for checkout or onboarding steps.

### 4. Tradeoffs

* **MVC**: simple and familiar. Views tied to Models are hard to unit test, and controllers tend to grow too big.
* **MVP**: easy to test. Each screen needs a Presenter-View interface, which adds code.
* **MVVM**: no manual View updates. Binding is hidden, so "why did this redraw?" can be hard to debug.
* **Redux-style stores**: predictable and easy to inspect, but a lot of code for simple state.

### 5. When NOT to Use

* **Small widgets**: one component with local state is enough.
* **A global store for everything**: form inputs and other local UI state usually don't belong there.
* **Micro frontends for one team**: all the deployment cost, none of the team benefit.

### 6. Interview Focus

* **MVC vs. MVP vs. MVVM**: answer with who updates the View and whether the View knows the Model.
* **Why MVP/MVVM test more easily**: display logic lives in a class with no UI code.
* **"Is React MVC?"**: React is the View. Hooks, stores, or services provide the rest.

### 7. Common Mistakes

* **Fat Controller or ViewModel**: business rules end up in the middle layer instead of the Model.
* **View reading the Model in MVP**: loses the passive View and the easy testing.
* **Arguing about labels**: most frameworks mix these ideas. Explain the responsibilities instead.
