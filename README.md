# MVVM vs MVI Architecture

This document explains the key differences between **MVVM (Model–View–ViewModel)** and **MVI (Model–View–Intent)**, especially in the context of Android and Jetpack Compose.

---

## 📌 MVVM (Model–View–ViewModel)

### 🔹 Overview
- **Data Flow**: Two-way (View ⇄ ViewModel).
- **State**: Can have **multiple observable states** (e.g., LiveData, StateFlow).
- **User Interaction**: View directly calls methods on the ViewModel.
- **ViewModel Role**: Exposes observable data and handles business logic.

### 🖼️ Sample Scenario (Login Screen)
- The View has two text fields (username, password).
- As the user types, the View updates the ViewModel with methods like `onUsernameChanged` and `onPasswordChanged`.
- The ViewModel exposes two observable states: `username` and `password`.
- The View observes these states and updates UI accordingly.
- On "Login" button click, the View calls `viewModel.login()`.

### ✅ Advantages
- Simpler to implement.
- Less boilerplate.
- Well-suited for **small to medium apps**.

### ❌ Drawbacks
- Risk of **state inconsistency** (scattered LiveData/StateFlow).
- Harder to trace state changes in complex apps.

---

## 📌 MVI (Model–View–Intent)

### 🔹 Overview
- **Data Flow**: Strict **one-way flow**  
  (View → Intent → ViewModel → State → View).
- **State**: **Single immutable state object** representing the entire screen.
- **User Interaction**: View emits **Intents (events)** to the ViewModel.
- **ViewModel Role**: Works like a **reducer** → old state + intent = new state.

### 🖼️ Sample Scenario (Login Screen)
- The View emits **Intents** like `UsernameChanged("Sai")`, `PasswordChanged("1234")`, and `LoginClicked`.
- The ViewModel takes the current state and the intent, and produces a new **UiState** with updated fields.
- The View only observes **one immutable state** object:

  •	Once login completes, the state updates again.

✅ Advantages
•	Single source of truth (all UI comes from one state).
•	Predictable, easy to debug, test, and replay flows.
•	Great for large/complex apps with many interactions.

❌ Drawbacks
•	More boilerplate.
•	Can feel heavy for simple apps.



🔄 Data Flow Diagrams

📌 MVVM → Two-Way Data Flow

┌───────────┐      updates       ┌────────────┐
│   View    │ ─────────────────▶ │ ViewModel  │
└───────────┘                    └────────────┘
▲                                  │
│                                  ▼
observes state                     exposes data
│                                  │
└──────────────────────────────────┘

	•	The View calls functions on the ViewModel (e.g., when a button is clicked).
	•	The ViewModel updates observable state (LiveData/StateFlow).
	•	The View observes that state and updates the UI.
	•	Both directions are possible → “two-way”.
📌 MVI → One-Way Data Flow
┌───────────┐   sends Intents   ┌────────────┐
│   View    │ ─────────────────▶│ ViewModel  │
└───────────┘                   └────────────┘
                                     │
                             reduces into new
                                   State
                                     │
                                     ▼
┌─────────────────────────────────────────────────┐
│                     State                       │
└─────────────────────────────────────────────────┘
                                     │
                                     ▼
┌───────────┐  renders from State ┌────────────┐
│   View    │ ◀────────────────── │  (loop)    │
└───────────┘                     └────────────┘

	The View emits Intents (events).
	•	The ViewModel processes them and produces a new immutable State.
	•	The View re-renders from that State.
	•	Flow is always one-way and predictable.


## 🔑 Key Differences

| Aspect              | MVVM                              | MVI                                     |
|---------------------|----------------------------------|-----------------------------------------|
| **Data flow**       | Two-way                          | One-way only                            |
| **State handling**  | Multiple observable states       | Single state object                     |
| **UI updates**      | Auto updates via observers       | Full state replacement                  |
| **Complexity**      | Easier, lightweight              | More structured, heavier                |
| **Best for**        | Small/medium apps                | Large/complex apps                      |


	•	MVVM
	•	Two-way communication (View ⇄ ViewModel).
	•	Multiple states (LiveData/StateFlow).
	•	Easier to implement.
	•	Risk of inconsistent state.
	•	MVI
	•	One-way communication (View → Intent → ViewModel → State → View).
	•	Single immutable state.
	•	Predictable & testable.
	•	More boilerplate.

📂 MVVM Folder Structure

MVVM usually organizes code by feature and separate
app/
└── src/
└── main/
└── java/com/example/app/
└── features/
└── login/
├── model/
│   ├── LoginRequest.kt
│   └── User.kt
│
├── view/
│   ├── LoginActivity.kt      // or LoginScreen.kt in Compose
│   └── LoginFragment.kt
│
├── viewmodel/
│   └── LoginViewModel.kt
│
└── repository/
└── LoginRepository.kt
🔹 Notes for MVVM:
•	model/ → Data classes, network responses, database entities.
•	view/ → UI layer (Activity, Fragment, Compose screen).
•	viewmodel/ → Exposes LiveData/StateFlow, handles UI logic.
•	repository/ → Data access (network + database).

👉 In MVVM, each View talks directly to a ViewModel and observes its data.

📂 MVI Folder Structure

MVI organizes by feature but adds Intents, State, and Reducers.

app/
└── src/
└── main/
└── java/com/example/app/
└── features/
└── login/
├── intent/
│   └── LoginIntent.kt        // all user actions
│
├── state/
│   └── LoginState.kt         // immutable UI state
│
├── view/
│   ├── LoginScreen.kt        // Compose screen or Fragment
│   └── LoginUi.kt            // UI rendering components
│
├── viewmodel/
│   └── LoginViewModel.kt     // processes intents → updates state
│
└── repository/
└── LoginRepository.kt

🔹 Notes for MVI:
•	intent/ → All possible user actions (LoginClicked, UsernameChanged, etc.).
•	state/ → Single immutable state object (LoginState).
•	view/ → UI layer, just renders from state.
•	viewmodel/ → Acts as a reducer: takes Intent + OldState → NewState.
•	repository/ → Data access (network + database).

👉 In MVI, the View never directly updates the state — it only emits Intents, and the ViewModel produces a new State.

⸻
## 🔑 Key Differences

| Aspect              | MVVM                              | MVI                                     |
|---------------------|----------------------------------|-----------------------------------------|
| **Architecture**    | Model, View, ViewModel           | Intent, State, Reducer(ViewModel), View |
| **State handling**  | Multiple LiveData/StateFlow      | Single immutable State object            |
| **UI updates**      | Views observe many variables     | Views render from one state              |
| **Folder structure**| Simple (model/view/vm)           | Extra layers (intent/, state/)           |


📖 Notes: Why ViewModel is Reducer in MVI
1.	Definition of Reducer
•	A Reducer is a pure function that takes:
•	The old state
•	An intent/event
•	Produces a new state
•	Formula:newState = reducer(oldState, intent)
      2.	Role of ViewModel in MVI
      •	In MVI, the ViewModel is responsible for:
      •	Receiving Intents from the View.
      •	Accessing the current State.
      •	Reducing them into a new State.
      •	This makes the ViewModel act as a Reducer.
      3.	Immutable State Principle
      •	MVI uses immutable states.
      •	Instead of mutating old state, the ViewModel always creates a new instance of the state (new reference).
      •	This ensures predictability and consistency.
      4.	One-Way Data Flow
      •	Intent → ViewModel (Reducer) → New State → View.
      •	No back-and-forth updates.
      •	Makes debugging, logging, and state replay possible.
      5.	Difference from MVVM
      •	MVVM ViewModel: Updates multiple observable fields (LiveData, StateFlow).
      •	MVI ViewModel: Acts as a Reducer → outputs a single immutable state object every time.
      6.	Benefits of Reducer Pattern in ViewModel
      •	✅ Single source of truth → All UI driven by one state object.
      •	✅ Traceable state history → Can log/replay all states for debugging.
      •	✅ Predictable UI → No partial or inconsistent updates.
      •	✅ Easy testing → Reducer logic is testable like pure functions.
