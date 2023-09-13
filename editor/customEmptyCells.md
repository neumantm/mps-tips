# Custom empty cell

In some cases it is desired to customize the empty cell of a reference or child cell but still retain the behavior of the cell to completely be replaced by the first character the user types.

To customize the empty cell of such a cell, usually the editor DSL provides a field labeled `/empty cell`. For some cell types this must first be enabled by setting `customize empty cell` in the Inspector to `true`.

However, when you just enter any constant here, the behavior of cell is not the same as the default empty cell.
To achieve the desired behavior of completely being replaced by the first character, use the `text*` aka `nullText` property of the constant cell instead of the `text` property (see bottom of Inspector).

When looking deeper into why and how this works, the actual behavior depends on whether the text is set via [`EditorCell_Label.setText(String)`][setText] or via [`EditorCell_Label.setDefaultText(String)`][setDefaultText] with any non-empty string being set via `setText` hiding the "default" or "null" text.
The `CellModel_Constant` basically just calls `setText` with the value of `text` and `setDefaultText` with the value of `text*` aka `nullText` (this is done in the generator).

[setText]: https://github.com/JetBrains/MPS/blob/7ed0ccb451197c7dc33d9395dbe0dfa74ca96786/editor/editor-runtime/source/jetbrains/mps/nodeEditor/cells/EditorCell_Label.java#L265
[setDefaultText]: https://github.com/JetBrains/MPS/blob/7ed0ccb451197c7dc33d9395dbe0dfa74ca96786/editor/editor-runtime/source/jetbrains/mps/nodeEditor/cells/EditorCell_Label.java#L271

### Error Cell

The error cell (both variants described in [Error Cells](./errorCells.md)) also uses this technique to achieve the same behavior.
However, it decides to which method to give its text based on its `myEditable` flag in its [`synchornizeViewWithModel` method][ErrorCellsynchronizeViewWithModel].

[ErrorCellsynchronizeViewWithModel]: https://github.com/JetBrains/MPS/blob/50a1d67b34f3510097c49cd7f6c590460ed25771/editor/editor-runtime/source/jetbrains/mps/nodeEditor/cells/EditorCell_Error.java#L65

## Setting styles to actually match default empty cell

To actually look exactly like the default empty cell, I've found that the following styles must be set:

```
EmptyCell {
  font-style : bold
}
```
with `EmptyCell` from `BaseLanguageStyle` from `jetbrains.mps.baseLangauge.editor`
