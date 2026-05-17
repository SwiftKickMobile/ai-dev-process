# UI Map Demo

A complete UI Map shown in both representations: YAML source (the source of truth) and the Mermaid render derived from it. For feedback on the format and the render.

Examples use **LumenNotes**, skai's shared fictional theme. The schema and conventions are project-agnostic.

## YAML

```yaml
version: 1
app: LumenNotes

modal_styles: [sheet, full_screen, card, popover]

domains:
  app:                                    # collapsed (domain == root scene)
    child:
      - library                           # cross-domain ref
      - login:
          nav:
            - welcome:
                todo: true
                note: "Onboarding flow not yet built"

  library:                                # collapsed
    tab:
      - notes_tab:
          nav:
            - note                        # cross-domain ref
      - folders_tab: {}
      - trash_tab: {}
    modal:
      - profile                           # cross-domain ref
      - search                            # cross-domain ref (reused)
    notes:
      - at: modal
        text: "Both surfaced from the library toolbar"

  note:                                   # collapsed
    composite:
      - editor                            # cross-domain ref
    modal:
      - tag_picker:
          modal_style: popover
      - attachment_viewer:
          modal_style: full_screen
      - share_sheet:
          modal_style: sheet
      - search                            # cross-domain ref (reused, pointer here)

  other:                                  # not collapsed (no common root)
    scenes:
      - profile:
          modal_style: sheet
          nav:
            - web                         # ref to common
      - search:
          modal_style: full_screen
          primary_parent: library         # canonical visual position
      - editor: {}

common:
  - web:
      implements: [privacy_policy, terms_of_service]

todos:
  - scope: folders_tab
    note: "Subfolder navigation planned for v2"
```

## Mermaid render

```mermaid
---
config:
  flowchart:
    diagramPadding: 20
    padding: 12
    wrappingWidth: 320
    subGraphTitleMargin:
      top: 0
      bottom: 0
    nodeSpacing: 30
    rankSpacing: 40
---
flowchart TD
    %% Domains
    subgraph domain_legend[" "]
        direction LR
        leg_app(["App"])
        leg_library(["Library"])
        leg_note(["Note"])
        leg_other(["Other"])
        leg_app ~~~ leg_library ~~~ leg_note ~~~ leg_other
    end

    %% Canonical scenes
    app(["App"])
    login(["Login"])
    welcome(["Welcome"])
    library(["Library"])
    notes_tab(["Notes Tab"])
    folders_tab(["Folders Tab"])
    trash_tab(["Trash Tab"])
    note(["Note"])
    tag_picker(["Tag Picker"])
    attachment_viewer(["Attachment Viewer"])
    share_sheet(["Share Sheet"])
    profile(["Profile"])
    search(["Search"])
    editor(["Editor"])
    web(["Web"])

    domain_legend ~~~ app

    app -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Child</span>"| app_child
    subgraph app_child[" "]
        direction LR
        library
        login
    end

    login -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Nav</span>"| login_nav
    subgraph login_nav[" "]
        welcome
    end

    library -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Modal</span>"| library_modal
    subgraph library_modal[" "]
        direction LR
        profile
        search
    end

    library -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Tab</span>"| library_tab
    subgraph library_tab[" "]
        direction LR
        notes_tab
        folders_tab
        trash_tab
    end

    notes_tab -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Nav</span>"| notes_tab_nav
    subgraph notes_tab_nav[" "]
        note
    end

    note -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Modal</span>"| note_modal
    subgraph note_modal[" "]
        direction LR
        tag_picker
        attachment_viewer
        share_sheet
        search_at_note(["Search"])
    end

    note -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Composite</span>"| note_composite
    subgraph note_composite[" "]
        editor
    end

    profile -->|"<span style='background:white;border:1px solid black;padding:4px 8px'>Nav</span>"| profile_nav
    subgraph profile_nav[" "]
        web
    end

    impl_web[/"Implements: Privacy Policy, Terms Of Service"/]
    impl_web -.- web

    note_welcome@{ shape: tag-rect, label: "Onboarding flow not yet built" }
    note_welcome -.- welcome
    note_library@{ shape: tag-rect, label: "Both surfaced from the library toolbar" }
    note_library -.- library_modal

    mstyle_library[/"<div style='text-align:left'>Profile <b>sheet</b>, Search <b>full_screen</b></div>"/]
    mstyle_library -.- library_modal
    mstyle_note[/"<div style='text-align:left'>Tag Picker <b>popover</b>, Attachment Viewer <b>full_screen</b>, Share Sheet <b>sheet</b>, Search <b>full_screen</b></div>"/]
    mstyle_note -.- note_modal

    classDef app_domain fill:#8AD1FA,stroke:#8AD1FA,color:#231f20,filter:none
    class leg_app,app,login app_domain
    classDef library_domain fill:#EFD74E,stroke:#EFD74E,color:#231f20,filter:none
    class leg_library,library,notes_tab,folders_tab,trash_tab library_domain
    classDef note_domain fill:#FE9CA8,stroke:#FE9CA8,color:#231f20,filter:none
    class leg_note,note,tag_picker,attachment_viewer,share_sheet note_domain
    classDef other_domain fill:#78DB85,stroke:#78DB85,color:#231f20,filter:none
    class leg_other,profile,search,editor,web other_domain
    classDef app_todo fill:#8AD1FA4C,stroke:#8AD1FA,stroke-width:2px,color:#231f20,filter:none
    class welcome app_todo
    classDef pointer fill:#bdbdbd,stroke:#bdbdbd,color:#231f20,filter:none
    class search_at_note pointer
    classDef mutex fill:#ffffff,stroke:#999,stroke-dasharray:10 5,filter:none
    class app_child,login_nav,library_modal,library_tab,notes_tab_nav,note_modal,profile_nav mutex
    classDef composite fill:#f0f0f0,stroke:#f0f0f0,stroke-width:0px,filter:none
    class note_composite composite
    classDef legendBox fill:#F5F5F5,stroke:#B5B5B5,color:#231f20,filter:none
    class domain_legend legendBox
    class impl_web other_domain
    classDef note fill:#fff7d6,stroke:#d4c171,color:#231f20,filter:none
    class note_welcome,note_library note
    classDef modalStyle fill:#ffffff,stroke:#999,stroke-dasharray:10 5,color:#231f20,filter:none
    class mstyle_library,mstyle_note modalStyle
    classDef smallFont font-size:13px
    class impl_web,note_welcome,note_library,mstyle_library,mstyle_note,leg_app,leg_library,leg_note,leg_other smallFont
```

## Notes on what's where

- **Wrappers per route.** Every route container in the YAML (`nav:`, `modal:`, `composite:`, `tab:`, `child:`) becomes a wrapper subgraph in the render. Even single-target routes get a wrapper. The route-kind label sits on the incoming arrow.
- **Canonical placement via `primary_parent`.** `Search` is YAML-organized under `other:` but its `primary_parent: library` puts the canonical instance in the `library_modal` wrapper. The reference from `note.modal` renders as a gray pointer (`Search_at_note`).
- **Gray fill marks non-canonical references.** Pointer nodes use the same shape as canonical scenes, distinguished by fill color only.
- **Root scenes are not wrapped.** `App` is the root: no incoming route, no enclosing wrapper. Matches the FigJam convention.
- **`common:` scenes render at their consumer.** `Web` is YAML-organized under `common:` (domain-agnostic), but the render places its canonical instance inside its only consumer's Nav wrapper (`profile_nav`), since it has a single inbound route. If Web were referenced from multiple consumers, a `primary_parent` would pick the visual home.
- **Scene fill encodes domain.** Each domain maps to a palette color (in YAML declaration order). The Domain Legend block at the top maps domain to color.
- **Mutex vs. composite wrappers are visually distinct.** Mutex wrappers use a dashed border with white fill; composite wrappers use light-gray fill with no border.
- **`Implements:` callouts.** Right-leaning parallelograms attached to the abstract scene via a dashed connector, with single-line comma-separated implementations. Fill matches the attached scene's domain color.
- **Modal-style callouts.** The project declares a `modal_styles` vocabulary; each scene reached by a modal route tags itself with a `modal_style`. For every modal wrapper, the render emits a callout — a parallelogram styled like the mutex wrapper (white fill, dashed gray border) — listing each destination and its bolded style on a single comma-separated line. The callout annotates the wrapper, not any one scene.
- **TODO scenes and notes.** `Welcome` is flagged `todo: true` in YAML. A TODO scene keeps its domain identity but signals the unimplemented state: full-opacity domain color as a 2px stroke, with the domain color at 0.3 alpha as the fill. `Welcome` also has a `note:` rendering as a pale-yellow sticky-note callout (a `tag-rect` shape). The two are independent — a scene can have either, both, or neither.
- **Container-attached notes.** A `notes:` entry on a scene can use `at: <route-kind>` to attach to a route container (wrapper) rather than the scene itself.
- **Auto-layout caveat.** Mermaid lays out the graph automatically; positions won't match the FigJam exactly. Topology, grouping, and edge labels match; relative positions within a domain are the renderer's choice.
