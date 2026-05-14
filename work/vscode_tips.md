# VSCODE TIPS

## Extension `tabout` conflicts with *auto suggestions*

Using `ESCAPE` to hide suggestion widget will come normal mode when *vim* extension is enabled. So another **rarely used** keystroke would be a better choice to hide *suggestion widget*, here we choose `'`;

add following keybinding:

```json
{
"key"    : "'",
"command": "hideSuggestWidget",
"when"   : "suggestWidgetHasFocusedSuggestion && suggestWidgetVisible && textInputFocus",
}
```

or, if you type quick enough, you may *tabout* before suggestion widget pop up.
