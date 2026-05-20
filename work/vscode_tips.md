---
marp: true
theme: lorca
size: 16:9
---

# VSCODE TIPS

---

## ␛ should hide *auto suggestions* without entering *vim normal mode*

---

```json
{
  "key": "escape",
  "command": "hideSuggestWidget",
  "when": "suggestWidgetVisible && textInputFocus && suggestWidgetHasFocusedSuggestion"
},
{
  "key": "ctrl+[",
  "command": "hideSuggestWidget",
  "when": "suggestWidgetVisible && textInputFocus && suggestWidgetHasFocusedSuggestion"
}
```

---

## Extension [tabout](https://marketplace.visualstudio.com/items?itemName=albert.TabOut) conflicts with snippets and *next edit suggestions*

---

make `tabout` has the lowest priority.

```json
{
  "key": "tab",
  "command": "-tabout",
},
{
  "key": "tab",
  "command": "tabout",
  "when": "editorTextFocus && !inSnippetMode &&
   !inlineSuggestionVisible && !suggestWidgetVisible &&
    !hasSnippetCompletions && !hasOtherSuggestions &&
     !editorTabMovesFocus && !tabShouldJumpToInlineEdit &&
      !tabShouldAcceptInlineEdit && !vim.active"
},
```

---

## How to trigger parameter hints and switch between overloads?

---

Use `⌘+⇧+⎵` to trigger parameter hints.
Use `⌥+↑`/`⌥+↓` to switch between overloads.

---