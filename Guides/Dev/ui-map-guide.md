# UI Map Guide

A UI Map is a YAML document that defines every scene in an app and the routing relationships between them. The map is the single source of truth for the app's UI architecture; implementation should mirror it directly. A render script produces a visual diagram from the YAML for human review.

## Contents

- [YAML format](#yaml-format)
- [SwiftUI](#swiftui)
- [Jetpack Compose](#jetpack-compose)
- [Figma Jam Diagram (deprecated)](#figma-jam-diagram-deprecated)

## YAML format

A UI Map is a YAML document with a fixed schema. The schema is formally defined in [`ui-map.schema.json`](ui-map.schema.json); this section is the conceptual reference.

### Top-level structure

```yaml
version: 1
app: <name>

modal_styles:              # optional
  - <style>

domains:
  <domain-id>:
    ...

common:                    # optional
  - <scene>

todos:                     # optional
  - scope: <scene-id>
    note: "..."
```

- `version`: integer, currently always `1`.
- `app`: human-readable app name.
- `modal_styles`: optional project vocabulary of valid modal presentation styles (see [Modal styles](#modal-styles)). Required only if any scene declares a `modal_style`.
- `domains`: top-level grouping. Each key is a domain ID; each value is either a *collapsed* or *non-collapsed* domain (see below).
- `common`: optional list of domain-agnostic scenes (e.g. a reusable Web component) shared across the app.
- `todos`: optional list of cross-cutting TODOs not tied to a single scene.

### Domains

A domain is a top-level grouping of related scenes. Code organization typically mirrors the domain layout.

**Collapsed**: domain ID matches its single root scene. The domain entry carries scene-level fields directly:

```yaml
app:                       # the App scene IS the domain root
  child:
    - login: {}
    - main                 # cross-domain ref
```

**Non-collapsed**: no single root scene. Contains a `scenes:` list of top-level entries:

```yaml
assessment:                # no scene named "assessment" exists
  scenes:
    - question_set: {}
    - media_capture: {}
```

Rule for collapsing: if domain ID == root scene ID, collapse.

### Scenes

Scenes appear inside route containers or directly under a non-collapsed domain's `scenes:` list. Each scene appears in two forms:

- **Reference** — bare string: `- foo`
- **Definition** — single-key mapping: `- foo: { ...body... }`

A scene is *defined* exactly once (its canonical home). All other appearances are references by ID.

Scene body fields (all optional):

| Field | Meaning |
|---|---|
| `nav`, `modal`, `composite`, `tab`, `child` | Route containers — lists of destinations |
| `modal_style` | Presentation style when reached by a modal route (one of the project's `modal_styles`) |
| `implements` | Variant identifiers for an abstract scene |
| `platform` | Platform scoping (`android`, `ios`, …) |
| `todo` | Boolean — scene is unimplemented |
| `note` | Single free-text annotation |
| `notes` | Multiple annotations (with optional `at:` for container attachment) |
| `primary_parent` | Visual canonical parent (for reused scenes with multiple inbound routes) |

A leaf scene with no body is written as `foo: {}`.

### Route containers

Each route kind is a list of destinations. The list represents a mutually-exclusive group (for nav/modal/tab/child) or the set of children for composite.

```yaml
appointment_details:
  nav:
    - question_set         # cross-domain ref
  composite:
    - thumbnail_slider     # cross-domain ref
  modal:
    - media_capture        # cross-domain ref
    - appointment_submitted: {}   # in-domain definition
    - record_warning: {}
```

Route kinds:

| Kind | Meaning |
|---|---|
| `nav` | Push onto a navigation stack |
| `modal` | Present modally (presentation style chosen at implementation time) |
| `composite` | Embed as a subview |
| `tab` | Embed as a tab in a tab container |
| `child` | Replace primary content (mutually exclusive root-view selection) |

### Modal styles

A `modal` route says a scene is presented modally but not *how*. Many projects have a fixed vocabulary of modal presentations (sheet, full-screen, card, popover, …). The UI Map can capture that vocabulary and tag each modal destination with its style.

Declare the project vocabulary once at the top level:

```yaml
modal_styles: [sheet, full_screen, card, popover]
```

Then tag a scene with `modal_style` wherever it is *defined* (its canonical home). The value must be one of the declared `modal_styles`:

```yaml
appointment_details:
  modal:
    - media_capture                # cross-domain ref — style lives at its definition
    - appointment_submitted:
        modal_style: card
    - record_warning:
        modal_style: card

media_capture:                     # canonical home
  modal_style: full_screen
```

`modal_style` belongs to the scene, not the route — a scene presented modally from several parents carries one style. It is informational: it does not change routing semantics, only how the destination is presented. In the render, each modal wrapper gets a callout listing its destinations and their styles.

Both fields are optional. Omit `modal_styles` entirely if the project doesn't track modal presentation; a scene with no `modal_style` simply isn't listed in the callout.

### Cross-domain references

Scenes can be referenced across domain boundaries by ID. References use the scene ID alone; the scene's body lives at its canonical home elsewhere in the document.

```yaml
domains:
  pipeline:
    tab:
      - appointments_stage:
          nav:
            - appointment_details   # cross-domain ref

  appointment_details:              # canonical home
    nav: [...]
```

Scene IDs are globally unique across the whole document.

### Reused scenes and `primary_parent`

A scene referenced from multiple places renders with one canonical visual instance and pointers everywhere else. When there are multiple inbound references, `primary_parent` picks the visual home:

```yaml
thumbnail_slider:
  primary_parent: appointment_details   # canonical instance renders here
  modal:
    - gallery: {}
```

With a single inbound reference, `primary_parent` is unnecessary — the only parent is implicitly primary.

### Annotations

Four annotation types, all optional and independent:

**`implements`** — Concrete variants of an abstract scene:

```yaml
web:
  implements: [privacy_policy, terms_of_service]
```

**`todo`** — Boolean flag: scene is not yet implemented:

```yaml
verify:
  todo: true
```

**`note` / `notes`** — Free-text annotations. Single shorthand or list of items:

```yaml
verify:
  todo: true
  note: "iOS implementation missing"

library:
  notes:
    - "Subfolder support planned for v2"
    - at: modal
      text: "Both surfaced from the library toolbar"
```

A note item can be:

- A bare string (attaches to the scene)
- An object `{ text: "...", at: <route-kind> }` (attaches to a route container of that kind instead of the scene)

> Note: the field is `at:` (not `on:`) because YAML 1.1 — which PyYAML uses — treats bare `on` as the boolean `true`. `at:` avoids the trap.

**`platform`** — Platform scoping:

```yaml
demo_mode_loading:
  platform: android
```

### Common scenes

Scenes that are domain-agnostic — typically reusable UI primitives used across features — live under top-level `common:` rather than inside a domain.

```yaml
common:
  - web:
      implements: [privacy_policy, terms_of_service]
```

References to common scenes from inside domains work the same as cross-domain references: by ID.

In code, common scenes belong to no feature module. They live in the shared UI module under its own top-level `Scenes/` folder — one folder per scene, following the same per-scene layout as any other scene.

### Top-level `todos:`

For TODOs that don't naturally attach to a single scene (e.g., architectural concerns, cross-cutting refactors), use the top-level `todos:` list:

```yaml
todos:
  - scope: folders_tab
    note: "Subfolder navigation planned for v2"
  - note: "Migrate analytics tracking to new framework"
```

`scope` is optional. For scene-specific TODOs, prefer the scene-level `todo: true` flag with an accompanying `note:`.

### Schema reference

The formal contract is in [`ui-map.schema.json`](ui-map.schema.json) (JSON Schema, draft 2020-12). Validate a UI Map with:

```
pipx run check-jsonschema --schemafile ui-map.schema.json <your-map>.yaml
```

The schema catches structural errors (missing required fields, invalid route kinds, malformed notes, etc.). Semantic errors (dangling cross-references, duplicate canonical homes) are validated by the render script.

## SwiftUI

Each scene's routing is implemented locally: the view model owns the outgoing route state as `@Published` properties, and the view applies SwiftUI modifiers driven by those properties. There is no central router.

### Scene file layout

For a scene named `Foo`:

```
Foo/
  FooView.swift          // struct FooView: View
  FooViewModel.swift     // route enums at the top, then @Observable class FooViewModel
  Views/                 // optional — Foo's own subviews
```

All of the scene's route enums live at the top of `FooViewModel.swift`, above the view-model class. A non-routing scene with no state to manage may omit `FooViewModel.swift` entirely — the view stands alone.

A scene's folder may contain a `Views/` subfolder for the scene's own subviews — small presentational views split out to keep `FooView.swift` uncluttered. `Views/` holds subviews only: never a scene (every scene gets its own folder), and never another scene's subviews.

Folder nesting reflects the diagram's parentage. Reused scenes pick one canonical parent folder; other parents import and instantiate them.

Commonly-accessed scenes may be moved higher in the folder hierarchy as a convenience, provided all of the scene's files remain together in a single, dedicated folder.

### Route enums

One enum per route kind. The enum is named `<Scene><Kind>Route` and each case maps to a destination scene by name. Cases carry whatever associated values the destination needs at construction. Modal-style enums are `Identifiable, Hashable` with a stable string `id`:

```swift
enum DashboardNavRoute: Identifiable, Hashable {
    case detail(itemID: String)

    var id: String {
        switch self {
        case .detail: "detail"
        }
    }
}
```

### Nav routing

```swift
enum DashboardNavRoute: Identifiable, Hashable {
    case detail
    var id: String { "detail" }
}

@Observable
class DashboardViewModel {
    var navRoute: DashboardNavRoute?
}

struct DashboardView: View {
    @State private var viewModel = DashboardViewModel()

    var body: some View {
        @Bindable var viewModel = viewModel
        NavigationStack {
            content
                .navigationDestination(item: $viewModel.navRoute) { route in
                    switch route {
                    case .detail: DetailView()
                    }
                }
        }
    }
}
```

The view model sets `navRoute` to navigate; SwiftUI clears it when the user pops or swipes back.

### Modal routing

A `Modal` connector in the diagram doesn't specify *how* the modal is presented — that's an implementation choice. SwiftUI has five distinct modal modifiers, each backed by its own enum and property:

| Style | Enum suffix | Property | Modifier |
|---|---|---|---|
| Sheet | `SheetRoute` | `sheetRoute` | `.sheet(item:)` |
| Full-screen cover | `CoverRoute` | `coverRoute` | `.fullScreenCover(item:)` |
| Custom modal | `ModalRoute` | `modalRoute` | `.modal(item:)` |
| SwiftMessages | `MessageRoute` | `messageRoute` | `.swiftMessage(message:...)` or `.modal(item:)` |
| Popover | `PopoverRoute` | `popoverRoute` | `.popover(item:)` |

A single scene can expose several at once — one enum and one `@Published` per presentation style:

```swift
@Observable
class DashboardViewModel {
    var navRoute: DashboardNavRoute?
    var sheetRoute: DashboardSheetRoute?
    var coverRoute: DashboardCoverRoute?
    var messageRoute: DashboardMessageRoute?
}

struct DashboardView: View {
    @State private var viewModel = DashboardViewModel()

    var body: some View {
        @Bindable var viewModel = viewModel
        content
            .navigationDestination(item: $viewModel.navRoute) { route in /* ... */ }
            .sheet(item: $viewModel.sheetRoute) { route in /* ... */ }
            .fullScreenCover(item: $viewModel.coverRoute) { route in /* ... */ }
            .modal(item: $viewModel.messageRoute) { route in /* ... */ }
    }
}
```

The choice of presentation style is informal and not encoded in the diagram.

### Child routing

Child routes are mutually-exclusive root views. The enum is non-optional and has a default case.

```swift
enum AppChildRoute {
    case login
    case main
}

@Observable
class AppViewModel {
    var childRoute: AppChildRoute = .login
}

struct AppView: View {
    @State private var viewModel = AppViewModel()

    var body: some View {
        switch viewModel.childRoute {
        case .login: LoginView()
        case .main: MainView()
        }
    }
}
```

### Tab routing

```swift
enum MainTabRoute: Hashable {
    case home
    case library
    case settings
}

@Observable
class MainViewModel {
    var currentTabRoute: MainTabRoute = .home
}
```

The view binds `currentTabRoute` to whatever tab container is in use (`TabView`, a custom material tab bar, etc.).

### Composite scenes

A composite parent has no route enum for its composite children — they're embedded directly in the parent's body.

```swift
struct DetailView: View {
    var body: some View {
        ScrollView {
            HeaderView(...)
            PlayerView(...)
            // ...
        }
    }
}
```

The composite wrapper in the diagram is a grouping; no Swift artifact corresponds to it.

### Reused scenes

A reused scene is a `View` struct instantiated from multiple parents. It may itself be a full scene with its own routes — e.g. a `PlayerView` used as a composite child in several parents, with its own modal route to a fullscreen presentation.

### Implements

An `Implements: A, B, C` annotation means a single Swift type renders multiple variants, parameterized at construction. The variant is typically carried as an associated value on the route case that targets it:

```swift
enum ProfileNavRoute: Identifiable, Hashable {
    case web(Resource)

    var id: String {
        switch self {
        case .web: "web"
        }
    }
}
```

`WebView(resource:)` renders the page for the given `Resource` (e.g. `.privacyPolicy`, `.termsAndConditions`). One view, one view model, multiple instances.

### State ownership

Routes live on the view model. The view never owns or mutates route state. Mutations come from:

- View-model methods called by the view, e.g. `viewModel.detailTapped()` sets `navRoute = .detail`.
- External publishers observed in the view model's `init`, e.g. an auth service publisher flipping `childRoute` between `.login` and `.main`.
- SwiftUI itself: any modifier bound with `item:` (Nav, Sheet, Cover, Modal, Message, Popover) clears the route back to `nil` when the user dismisses the destination (back button, swipe-back, sheet drag-down, etc.).

The view model can also set or clear any route programmatically at any time — assign `nil` to dismiss, or assign a new value to transition to a different destination.

## Jetpack Compose

Each scene's routing is implemented locally: the view model emits one-shot navigation effects, and the screen subscribes and dispatches each effect to a Compose Navigation `NavController` scoped to that scene. There is no central router.

### Scene file layout

For a scene named `Foo`:

```
foo/
  FooScreen.kt          // @Composable fun FooScreen(viewModel: FooViewModel = hiltViewModel())
  FooViewModel.kt       // route enums at the top, then @HiltViewModel class FooViewModel : ViewModel()
  FooViewState.kt       // data class for observable scene state
  FooViewEvent.kt       // sealed interface — UI inputs to the view model
  FooViewEffect.kt      // sealed interface — one-shot outputs (navigation, toasts, etc.)
  views/                // optional — Foo's own subviews
```

All of the scene's route enums live at the top of `FooViewModel.kt`, above the view-model class. A non-routing scene with no state to manage may omit the view model, view state, event, and effect files entirely — the composable stands alone.

A scene's package may contain a `views/` subpackage for the scene's own subviews — small composables split out to keep `FooScreen.kt` uncluttered. `views/` holds subviews only: never a scene (every scene gets its own package), and never another scene's subviews.

Package nesting reflects the diagram's parentage. Reused scenes pick one canonical parent package; other parents call the composable directly.

Commonly-accessed scenes may be moved higher in the package hierarchy as a convenience, provided all of the scene's files remain together in a single, dedicated package.

### Route enums

Sealed classes implementing a `Route` marker interface, with `@Serializable` cases for type-safe Compose Navigation. Cases carry whatever associated values the destination needs as `data class` fields.

```kotlin
sealed class DashboardNavRoute : Route {
    @Serializable data object Detail : DashboardNavRoute()
    @Serializable data class Item(val id: String) : DashboardNavRoute()
}
```

### Scene state, events, and effects

Every scene exposes three sealed types alongside its view model:

- `FooViewState` — a `data class` of observable state; exposed as `StateFlow<FooViewState>`.
- `FooViewEvent` — a `sealed interface` of UI inputs; the screen calls `viewModel.processEvent(event)`.
- `FooViewEffect` — a `sealed interface` of one-shot outputs (navigation, toast, etc.); exposed as `Flow<FooViewEffect>` from a `Channel`.

```kotlin
@HiltViewModel
class DashboardViewModel @Inject constructor() : ViewModel() {

    private val _viewState = MutableStateFlow(DashboardViewState())
    val viewState: StateFlow<DashboardViewState> = _viewState.asStateFlow()

    private val _viewEffects = Channel<DashboardViewEffect>(Channel.BUFFERED)
    val viewEffects: Flow<DashboardViewEffect> = _viewEffects.receiveAsFlow()

    fun processEvent(event: DashboardViewEvent) { /* … */ }
}
```

### Nav routing

The view model emits a `Navigate(route)` effect; the screen dispatches it to the local `NavController`.

```kotlin
sealed class DashboardNavRoute : Route {
    @Serializable data object Detail : DashboardNavRoute()
}

data class Navigate(val route: DashboardNavRoute) : DashboardViewEffect

@Composable
fun DashboardScreen(viewModel: DashboardViewModel = hiltViewModel()) {
    val viewState by viewModel.viewState.collectAsStateWithLifecycle()
    val navController = rememberNavController()

    LaunchedEffect(Unit) {
        viewModel.viewEffects.collect { effect ->
            when (effect) {
                is Navigate -> navController.navigatePush(effect.route)
            }
        }
    }

    SlideNavHost(navController = navController) {
        composable<NavStart> { DashboardContent(viewState, viewModel::processEvent) }
        composable<DashboardNavRoute.Detail> { DetailScreen() }
    }
}
```

### Modal routing

One `ModalRoute` enum per scene. The presentation style (dialog, bottom sheet, full-screen composable) is chosen at the `NavHost` declaration site, not in the route enum:

```kotlin
sealed class DashboardModalRoute : Route {
    @Serializable data object Profile : DashboardModalRoute()
    @Serializable data class MediaCapture(val id: String) : DashboardModalRoute()
    @Serializable data object RecordWarning : DashboardModalRoute()
}

data class NavigateModal(val route: DashboardModalRoute) : DashboardViewEffect
data object CloseModal : DashboardViewEffect
```

```kotlin
val modalNavController = rememberNavController(rememberBottomSheetNavigator())

LaunchedEffect(Unit) {
    viewModel.viewEffects.collect { effect ->
        when (effect) {
            is NavigateModal -> modalNavController.navigatePush(effect.route)
            is CloseModal    -> modalNavController.popBackStack()
        }
    }
}

ModalBottomSheetNavHost(navController = modalNavController) {
    bottomSheet<DashboardModalRoute.Profile>            { ProfileScreen() }
    composable<DashboardModalRoute.MediaCapture>        { MediaCaptureScreen() }
    dialog<DashboardModalRoute.RecordWarning>           { RecordWarningDialog() }
}
```

### Child routing

Child routes replace what the scene renders. The start destination lives on the view state; transitions use `navigateReplace` instead of `navigatePush`.

```kotlin
sealed class AppChildRoute : Route {
    @Serializable data object Login : AppChildRoute()
    @Serializable data object Main : AppChildRoute()
}

data class AppViewState(val startDestination: AppChildRoute = AppChildRoute.Login)

data class Navigate(val route: AppChildRoute) : AppViewEffect
```

```kotlin
LaunchedEffect(Unit) {
    viewModel.viewEffects.collect { effect ->
        when (effect) {
            is Navigate -> navController.navigateReplace(effect.route)
        }
    }
}

NavHost(navController = navController, startDestination = viewState.startDestination) {
    composable<AppChildRoute.Login> { LoginScreen() }
    composable<AppChildRoute.Main>  { MainScreen() }
}
```

### Tab routing

The view state carries `currentTabRoute`. The screen uses a pager and tab row, syncing the pager's position to the route via events.

```kotlin
sealed class MainTabRoute : Route {
    @Serializable data object Home    : MainTabRoute()
    @Serializable data object Library : MainTabRoute()
    @Serializable data object Settings: MainTabRoute()
}

data class MainViewState(val currentTabRoute: MainTabRoute = MainTabRoute.Home)
```

### Composite scenes

A composite parent has no route enum for its composite children — they're called directly as composables in the parent's body.

```kotlin
@Composable
fun DetailScreen(viewModel: DetailViewModel = hiltViewModel()) {
    Column {
        HeaderView(/* … */)
        PlayerView(/* … */)
    }
}
```

The composite wrapper in the diagram is a grouping; no Kotlin artifact corresponds to it.

### Reused scenes

A reused scene is a `@Composable` function called from multiple parents. It may itself be a full scene with its own routes — e.g. a `PlayerScreen` used as a composite child in several parents, with its own modal route to a fullscreen variant.

### Implements

An `Implements: A, B, C` annotation means a single composable renders multiple variants, parameterized at call site. The variant is typically carried as a field on the route case that targets it:

```kotlin
sealed class ProfileNavRoute : Route {
    @Serializable data class Web(val resource: Resource) : ProfileNavRoute()
}
```

`WebScreen(resource = …)` renders the page for the given `Resource` (e.g. `PrivacyPolicy`, `TermsAndConditions`). One composable, one view model, multiple instances.

### State ownership

Routes are owned by the local `NavController`, not the view model. The view model expresses *intent* to route via one-shot effects (`Navigate`, `NavigateModal`, `CloseModal`); the screen dispatches each effect to the appropriate `NavController`. The screen never reads the current route from the view model — Compose Navigation is the source of truth.

Effects come from:

- `processEvent` handlers called by the screen, e.g. a click handler that sends `Navigate(...)` to the effect channel.
- External flows observed in the view model's `init`, e.g. an auth flow that sends `Navigate(AppChildRoute.Login)` when the user logs out.
- Compose Navigation itself handles dismissal: system back press, drag-to-dismiss, and outside-tap pop the back stack automatically. The view model can pop programmatically via `CloseModal`.

## Figma Jam Diagram (deprecated)

Each app has its own UI Map in FigJam. The diagram uses a fixed vocabulary of shapes, colors, and connector labels.

### Scenes

A scene is a discrete unit of UI that owns its own state and routes. A scene is usually a full screen, but it can also be a subview that owns routes of its own (e.g. a reusable player that can present its own modal).

A scene is drawn as a **rounded rectangle** labeled with the scene name, with two shape exceptions for reused and TODO scenes (see below). Color is used redundantly with shape, both as a visual hint for human readers.

- **Rounded rectangle, solid blue fill** — a scene that handles routing (has one or more outgoing routes).
- **Rounded rectangle, light blue fill** — a scene with no outgoing routes.
- **Hexagon, gray fill** — a scene defined elsewhere in the map and reused here. Hexagons are pointers; the canonical definition is wherever the scene is drawn as a rounded rectangle.
- **Octagon, pink fill** — a scene whose routing is a TODO (future scene, incomplete mapping, or pending routing refactor).

### Routes

A route is a **connector** between two scenes, labeled with the route kind:

| Label | Meaning |
|---|---|
| `Nav` | Parent pushes the child onto a navigation stack. |
| `Modal` | Parent presents the child modally. The diagram does not specify the modal presentation style; see the SwiftUI section. |
| `Child` | Parent embeds the child as its primary content. Switching child routes replaces what the parent renders. |
| `Tab` | Parent embeds the child as one tab in a tab container. |
| `Composite` | Parent embeds the child as a subview. The child is part of the parent screen, not a separate destination. |

### Wrappers

A wrapper drawn around one or more scenes is itself meaningful, not just visual grouping. Shape distinguishes the two wrapper kinds; stroke style is used redundantly as a visual hint.

- **Rounded rectangle, dashed stroke** — a set of *mutually-exclusive routes*. A connector terminating on the wrapper routes to exactly one scene inside at a time.
- **Manual-input shape, solid stroke** — a *composite parent*. The wrapper itself is never the target of a route; its children compose a single screen.

### Annotations

A right-leaning parallelogram attached to a scene carries one of:

- `Implements: A, B, C` — the scene is a single class with multiple concrete instances. Each name is an instance, distinguished by a construction-time parameter. For example, a `Web` scene annotated `Implements: Privacy Policy, Terms and Conditions` is one reusable web component; "Privacy Policy" and "Terms and Conditions" are two concrete instances distinguished by URL.
- `<Platform> only` (e.g. `Android only`) — the scene exists only on the named platform.
- Free text — a designer or developer note.

Sticky notes mark TODOs and known divergences between the map and the current implementation.
