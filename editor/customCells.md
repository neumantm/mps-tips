# Custom Cells

The Editor DSL allows embedding custom cells via the `CellModel_custom` aka `$custom cell$` aka `custom` and even custom Swing Components via the `CellModel_JComponent` aka `$swing component$` aka `swing component`.
These are very powerful features, but they are not very well documented.
This page tries to collect some information about how to use them.

## Swing Component

After adding a swing component cell to the editor, you need to implement a component provider in the Inspector:
```
component provider: (node, editorContext)->JComponent {
  <no statements>
}
```
You need to, given the editor context and node, return the `JComponent` to be displayed in the cell.
To be able to instantiate any `JComponent`, you usually want to import the model `javax.swing@java_stub`.
Then you can use the `new` expression to instantiate any `JComponent` you want.

Adding a button that performs some action when clicked is explained pretty well by the [Series "Writing an importer"][series] by Sergej Koščejev on <https://specificlanguages.com/>, specifically [Post 6: Invoking from the Editor][Post6].

[series]: https://specificlanguages.com/posts/2022-01/06-writing-an-importer-introduction/
[Post6]: https://specificlanguages.com/posts/2022-01/18-writing-an-importer-invoking-from-editor/

## Custom Cell

After adding custom cell to the editor, you need to implement a cell provider in the Inspector:
```
cell provider (editorContext, node)->AbstractCellProvider {
  <no statements>
}
```
You need to, given the editor context and node, return an `AbstractCellProvider`, which produces cell instances to be displayed in the cell.
To be able to instantiate an `AbstractCellProvider`, you usually want to import the model `jetbrains.mps.nodeEditor@java_stub`.
Then you can instantiate an anonymous subclass of `AbstractCellProvider`:
```
(editorContext, node)->AbstractCellProvider {
  new AbstractCellProvider(node) {
    @Override
    public EditorCell createEditorCell(EditorContext p1)  {
      <no statements>
    }
  };
}
```
Alternatively you can also create a named subclass of `AbstractCellProvider` in an util model and instantiate that.

If you want to store the node as its actual node type (`node<MyConcept>`) within the Provider, you can also override the constructor with a typed argument, store in a typed class attribute and pass it down to the super constructor.

To be able to instantiate cells, you usually want to import the model `jetbrains.mps.nodeEditor.cells@java_stub`.
I've found that the cells `EditorCell_Constant` and `EditorCell_RefPresentation` are particularly useful.

It is also possible to set the style on those cells:

- First import model `jetbrains.mps.editor.runtime.style@java_stub`
- Then set the style on a given cell: e.g.
  ```
  cell.getStyle().set(StyleAttributes.INDENT_LAYOUT_INDENT, true);
  ```

### EditorCell_RefPresentation

Here is a Snippet on how to create a EditorCell_RefPresentation:

This snippet additionally needs these model imports:
- `jetbrains.mps.smodel.action@java_stub` for `IReferentPresentationProvider`
- `org.jetbrains.annotations@java_stub` for `@NotNull`
- `org.jetbrains.mps.openapi.model@java_stub` for `SNode`
- `jetbrains.mps.editor.runtime.style@java_stub` for `StyleAttributes`

```
EditorCell_Property refCell = EditorCell_RefPresentation.create(editorContext, sourceNode, targetNode, new IReferentPresentationProvider() {
  @NotNull()
  @Override
  public String getPresentation(@NotNull() SNode p1, @NotNull() SNode p2)  {
    p2.getPresentation();
  }
});
// No idea why these two are necessary and why the "ref" cell does not do this automatically
refCell.setReferenceCell(true);
refCell.getStyle().set(StyleAttributes.NAVIGATABLE_NODE, targetNode);
return refCell;
```
