# Film! - Movie Schedule and Ticket Booking

## Project Build
To build the project, run:

```sh
npm run build
```

or

```sh
yarn build
```

## Project Launch
To start the project, run:

```sh
npm run start
```

or

```sh
yarn start
```

---

## Project Description
The **"Film!"** project implements a typical ticket booking service, specifically for a cinema. Users can browse the movie schedule, select a session, and book tickets. The project is built using **TypeScript** and follows the **SPA (Single Page Application)** approach, utilizing an **API** to fetch movie and session data.

### Key Features:
- Users can book tickets for only **one session at a time**.
- Movie sessions are **updated daily** on the server.
- When using a **mock API**, booked tickets remain **reserved for 24 hours**, displaying occupied seats.
- The **cart contents are stored in localStorage** until the booking is completed.
- After a **successful booking**, contact details are saved in `localStorage` and **auto-filled** in the booking form for future use.

---

## User Interface Overview
The interface consists of three main steps:

1. **Browsing the movie schedule** (`MainScreen`)
2. **Selecting a session and seats** (`SelectSessionScreen`, `SelectPlaceScreen`)
3. **Placing an order** (`BasketScreen`, `OrderScreen`, `SuccessScreen`)

Since the project uses a **unified modal window system**, its common logic and structure are encapsulated in an **abstract class `ModalScreen`**. All modal windows inherit from it and override the necessary methods for specific functionality.

---

## Project Structure
```
.  
├── src/  
│   ├── common.blocks/       [Component styles]  
│   ├── components/          [Implementation]  
│   │   ├── base/            [Base code]  
│   │   ├── model/           [Data models and API]  
│   │   ├── view/            [Views]  
│   │   │   ├── common/      [Shared components]  
│   │   │   ├── partial/     [Domain-specific components]  
│   │   │   ├── screen/      [Top-level screens]  
│   │   ├── controller/      [Controllers]  
│   ├── pages/  
│   │   ├── index.html       [Main page and component templates]  
│   ├── types/               [Type definitions]  
│   │   ├── global.d.ts      [Global types and environment extensions]  
│   │   ├── settings.ts      [Settings types]  
│   │   ├── html.ts          [HTML-related types]  
│   ├── utils/  
│   │   ├── constants.ts     [Project settings]  
│   │   ├── html.ts          [DOM utility functions]  
├── api.yaml                 [API specification]  
```

---

## Project Architecture (MVC)
The project follows an **MVC (Model-View-Controller)** structure:

### Model (`AppState.ts`)
A single **application state model** (`src/components/model/AppState.ts`) contains all data logic and actions. Data changes occur through model methods, which notify observers via the `onChange(changes: AppStateChanges)` method, ensuring decoupled communication between components.

A wrapper, `src/components/model/AppStateEmitter.ts`, connects the model to the event system.

### Controller
Controllers act as **event handlers** for user actions and update the model state through its methods. The model instance is passed to controllers, which are then injected into top-level views (screens).

### View
Top-level screens listen for updates in `AppStateEmitter` and re-render accordingly. Screens encapsulate implementation details and receive only event handlers and necessary data.

#### Interaction Flow:
```ts
const api = new Api(); // API initialization
const app = new ModelEmitter(api); // Model and event system initialization

const screen = new Screen( // Screen initialization
    new Controller( // Controller initialization
        app.model // Passing the model to the controller
    )
);

app.on('change:value', () => {
    screen.value = app.model.value;
});

// Screen.onClick -> Controller.onClick -> Model.value -> Screen.value
```
This connects all application components efficiently.

---

## View Components
### Categories:
- **Common Components (`common`)** – Independent UI components not tied to project-specific logic.
- **Partial Components (`partial`)** – Components that implement domain-specific project logic.
- **Screen Components (`screen`)** – High-level components representing entire screens.

Common (`common`) and partial (`partial`) components are **independently typed**, avoid global settings, and can be **reused across projects**. Screens (`screen`) rely on global settings for initialization and data transfer between nested views.

#### **Component Example:**
```ts
class Component extends View<DataType, SettingsType> {
    constructor(public element: HTMLElement, protected readonly settings: Settings) {
        super(element, settings);
        // Avoid overriding the constructor in child components!
    }

    protected init() {
        // Lifecycle method for component initialization
        // Attach event listeners here
    }

    set value(value: number) {
        // Set the "value" field in the UI
    }

    render() {
        // Render the component
        return this.element;
    }
}
```
---

## Model
The `AppState` class represents the project's **data model**, handling all data logic. It follows an **Observer pattern**, notifying subscribers of changes via `onChange(changes: AppStateChanges)`.

#### **Basic Model Structure:**
```ts
enum ModelChanges {
    value = 'change:value'
}

interface ModelSettings {
    onChange(changes: ModelChanges): void;
}

class Model {
    constructor(
        protected api: Api, // API for data management
        protected settings: ModelSettings // Settings and event handlers
    ) {
        // Model initialization
    }

    public changeValue(value: number) {
        // Modify data
        this.onChange(ModelChanges.value);
    }
}
```

---

## Controller
Controllers handle user interactions and update the model state through its methods.

#### **Example Controller:**
```ts
class Controller {
    constructor(
        protected model: Model // Model instance
    ) {
        // Controller initialization
    }

    public onClick = () => { // Preserve `this` context
        this.model.changeValue(1);
    }
}
```
Controllers primarily manage event handling and decision-making, while models handle data dependencies. This separation ensures flexibility.

---

## Final Thoughts
- The **Model** encapsulates **business logic** and **data management**.
- **Controllers** act as **intermediaries** between the Model and the View.
- **Views** manage **UI rendering** and **user interactions**.
- The project adheres to **modular development principles**, ensuring **scalability and maintainability**.

This approach allows for **clear separation of concerns**, making it easier to expand and refactor the project in the future. 🚀
