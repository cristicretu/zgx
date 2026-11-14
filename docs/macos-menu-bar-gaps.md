# macOS menu bar: what an app can't ask for yet

Found while making Anara's native app's menu bar a Mac app's (anaralabs/anara, `apps/desktop-native/ui/menus.luau`).
This branch fixes the first item; the rest are what the app still can't do through `ui.command`.

## 1. A key-only command's key redrew the whole bar (fixed here)

`NativeMenu::update_states` compared every menu's shape, the key-only menu (`zgx:keys`) included. That menu never
shows, but a command whose `key` is a function (one key per focused pane, an Escape bound only while a popover is
open) turns its accelerator on and off as the app moves, so every navigation reinstalled the main menu and AppKit
redrew the bar: a flash across it. `shape` now leaves the key-only menu out and keeps only what it decides about
the bar (whether zgx's own find and zoom items join Edit and View). Test: `menu_services`, "the key-only menu's
keys keep the bar".

## 2. Titles that follow the app

`ui.command`'s `title` is read once. `update_states` already changes titles in place, so a state or function
`title` (as `key` and `enabled` take) would give Show Sidebar / Hide Sidebar, Pin / Unpin and a Window menu that
lists the open tabs by name (fixed slots, disabled when empty) without a rebuild.

## 3. Zoom keys reach a focused element first

AppKit offers ⌘= ⌘- ⌘0 to the menu bar before the window, so with View's zoom items present a focused PDF viewer
never sees them; the app had to choose between its viewer's zoom keys and the window's zoom items. Proposal: the
zoom items' action dispatches the key to the focused node first (as `zoom::after_dispatch` does for a key nothing
took) and zooms the window only if nothing took it.

## 4. Edit's text submenus

Spelling and Grammar, Substitutions, Transformations and Speech are AppKit items whose actions (`checkSpelling:`,
`toggleAutomaticQuoteSubstitution:`, `uppercaseWord:`, `startSpeaking:`…) go to an `NSTextView` first responder.
zgx's fields aren't one, so the items would stay disabled. zgx would answer those selectors from its focused
field (it already runs the platform spell checker for its text menu) and put the submenus in Edit.

## 5. AppKit's automatic items, checked live

Start Dictation, Emoji & Symbols and AutoFill (Edit), Enter Full Screen (View), and the window tiling items
(Fill, Center, Move & Resize, in Window) are added by AppKit to menus it recognizes. Whether each appears and
works with zgx's `NSTextInputClient` and windows needs a foreground app; if any is missing, zgx can add it
(`orderFrontCharacterPalette:`, `startDictation:`, `toggleFullScreen:`).

## 6. Its own titles in the app's language

zgx's items (Find…, Zoom In, Actual Size, Minimize, Zoom, Bring All to Front, Services, Hide, Quit) are English.
A table of them the app fills from its catalog, or the bundle's localizations, would show them in the app's language.
