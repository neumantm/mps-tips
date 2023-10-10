# Typesystem

This file contains some remarks on the MPS typesystem.

## Node whose type is set by a rule in another node

Assume we have the following structure:

- Interface Concept `VariableDeclaration`
- Interface Concept `AssignmentLeftSide`
- Concept `VariableDeclarationAssignment` implements `AssignmentLeftSide`, `VariableDeclaration`
    - attribute(string) called `name`
- Concept `VariableAssignment` implements `AssignmentLeftSide`
    - reference to `VariableDeclaration` called `variable`
- Concpet `Assignment`
    - child(`AssignmentLeftSide`) called `leftSide`
    - child(`Expression`) called `rightSide`

Now we want to make sure the type of the left side is a supertype of the right side.
In the case of `VariableDeclarationAssignment`, its type should also be inferred from the right side.

Both can be achieved by a simple rule:

```
do {
    infer typeof(assignment.leftSide) :>=: typeof(assignment.rightSide);
}
```

After compiling this seems to work in the sandbox.

However, there is one crucial floor:
When trying to determine the type of a `VariableDeclarationAssignment` in isolation, the typesystem cannot infer its type because there is no typesystem rule attached to that concept.
According to [the mps docs on typesystem dependencies], the typesystem should automatically check the parent node if the type is not concrete.
However, in my experience this does not happen in the illustrated case (on MPS 2022.2).

To fix this we need to add a typesystem rule to `VariableDeclarationAssignment`, which forces the typesystem to evaluate the rule on `Assignment`.
This can be done with the following snippet:

```
do {
    typeof(assignmentLeftSide.parent);
}
```

This is also explaine in [the mps docs on typesystem dependencies]. 
However, when I first read it, I did not understand it correctly, hence me writing this chapter. 

A note on when the problem manifests:

- As mentioned above the problem only occurs when the type needs to be determined in isolation
    - In my experience that is the case when using the type (or a type which is based on that type) outside of the typesystem (e.g. for Scope Calculation)
- According to my observation the problem does not manifest in the editor
    - I assume in the editor the typesystem always evaluates the rules of all nodes visible within the current editor or something.
- However the problem does manifest when running a model check (which is done on a rebuild)
- According to [the mps docs on typesystem dependencies] it can also manifest in generators
- It can be forced to manifest by forcing the typesystem to calculate the type in isolation (see [Getting the type of a node](#getting-the-type-of-a-node))

[the mps docs on typesystem dependencies]: https://www.jetbrains.com/help/mps/typesystem.html#dependencies

## Getting the type of a node

To get the type of a node outside of inference rules, one can usually use the `type` operation (you need the language `jetbrains.mps.lang.typesystem`).
This is also explained in [the mps docs on using the typesystem].

However, as described in [Node whose type is set by a rule in another node](#node-whose-type-is-set-by-a-rule-in-another-node), 
it can be the case that the type of a node differs based on whether it was calculated in isolation (e.g. during model checks or in the generator) or in a typechcking session which checked many nodes (as the one of the editor).

To avoid risking to have this difference, it is possible to always force the typesystem to calculate the type in isolation:

```
node<> myType = TypecheckingFacade.getFromContext().computeIsolated({ => myNode.type; })
```

(You need the model `jetbrains.mps/typechecking@java_stub` and the language `jetbrains.mps.lang.typesystem`.)

[the mps docs on using the typesystem]: https://www.jetbrains.com/help/mps/using-typesystem.html#typeoperation