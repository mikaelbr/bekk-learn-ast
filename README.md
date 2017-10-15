# Exploration of AST

Learning resource for practise group meeting on abstract syntax trees.

## Resources

1. [Slides](http://mib.im/bekk-learn-ast/slides/#1)
2. [ASTExplorer](http://astexplorer.net/)
3. [JSCodeshift – API for traversing and transforming recast ASTs](https://github.com/facebook/jscodeshift)
4. [JS-Codemods – jscodeshift examples](https://github.com/cpojer/js-codemod/tree/master/transforms)
5. [VSCodemods  – VSCode extension for doing codemods (examples)](https://github.com/mikaelbr/vscodemod)


## Types

To know what types to transform, look at the [definitions](https://github.com/benjamn/ast-types/tree/master/def).

For instance, with type like:

```js
// The Parser API calls this ArrowExpression, but Esprima and all other
// actual parsers use ArrowFunctionExpression.
def("ArrowFunctionExpression")
  .bases("Function", "Expression")
  .build("params", "body", "expression")
  // The forced null value here is compatible with the overridden
  // definition of the "id" field in the Function interface.
  .field("id", null, defaults["null"])
  // Arrow function bodies are allowed to be expressions.
  .field("body", or(def("BlockStatement"), def("Expression")))
  // The current spec forbids arrow generators, so I have taken the
  // liberty of enforcing that. TODO Report this.
  .field("generator", false, defaults["false"]);
```

You can use jscodeshift like:

```js
// Construct a function: a => "Returned Statement"
j.arrowFunctionExpression([j.identifier("a")], j.literal("Returned Statement"));
```

## Traversing and replacing

You can traverse existing code with the jscodeshift API:

```js
function transformer(file, api) {
  const j = api.jscodeshift;

  return j(file.source)
    .find(j.Literal) // What node to look after
    .forEach(function (path) { // Do something with each result
      // Here we replace the current node in the AST with a new node (mutating the AST)
      path.replace(j.arrowFunctionExpression([j.identifier("a")], j.literal("Returned Statement")));
    })
    .toSource(); // Prints back to source code. Takes some printing options.
}
```

Types can also be used to check in if statements:

```js
// ...
.forEach(function (path) {
  if (!j.Expression.check(path.parentPath)) {
    return;
  }
  // ... do something.
})
```