# Using LangVis in the Console

The LangVis plugin can be used in the console to control what concepts are rendered.
This is useful when you want to render concepts from multiple related languages.

To do this:

- Open the MPS console
- Import Model via Root (Ctrl-R): `PlantUMLRenderer`
- Import Model via Root (Ctrl-R): `AbstractConceptDeclaration`
- Execute the following statement sequence:
  ```java
  {
    PlantUMLRenderer r = new  PlantUMLRenderer(false, true); // showTraitsAsAnnotations = false, collectInheritors = true
    r.renderPlantUMLSource(
        new hashset<node<AbstractConceptDeclaration>>(copy: #instances(AbstractConceptDeclaration)), // a set of concepts to render; here: all concpets in the project
        true, true, true, true, false, // boolean flags collectStructureDown, collectHiearchyUp, renderCardinalities, renderRoleNames, flattenNamespaces (correspond to the checkboxes in the plugin panel)
        System.getProperty("user.home") + "/mps-metamodel-all.plantuml" // output file path
    );
  }
  ```
