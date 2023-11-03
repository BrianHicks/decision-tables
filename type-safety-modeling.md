---
title: Modeling Decision Tables
---

**Hi!** This is a literate [Alloy](https://alloytools.org) file.
If you have Alloy installed, you can load this up directly and check my work.

Start with types.
For the purposes of this app, we're only using concrete types, so no need to model type parameters or anything.

```alloy
sig Type {}

sig Value {
  type: one Type,
}
```

Next, we have some columns and tables.
These have rules governing how values interact (see `README.md`), but we're only interested in type safety for this model so we'll constrain ourselves along those lines:

```alloy
one sig Table {
  inputs: one Type,
  outputs: one Type,
  rules: Value one-> Value,
  wantToCollapse: set Value,
}

-- get the set of values on the left-hand side of the rules
fun lhs[r: Value -> Value]: Value {
  ~r[Value]
}

-- get the set of values on the right-hand side of the rules
fun rhs[r: Value -> Value]: Value {
  r[Value]
}
```

To avoid having to deal with ordering, we'll just say that inputs and outputs are exactly one column (really just one `Type`.)
In the real application this is a list of named columns, but the types have to sync the types in the same way.
If you like, just pretend that you see list of `Column` or a tuple of `Value` wherever you see single values like that in `Table`.

Right away, Alloy shows that we can get a situation where the `Value`s in `rules` don't match the types in `inputs` and `outputs`.
Let's acknowledge we'll have to take care of that with brute-force.
Axioms ahoy:

```alloy
fact "left-hand side of the rules match the inputs" {
  all t: Table | t.rules.lhs.type = t.inputs
}

fact "right-hand side of the rules match the outputs" {
  all t: Table | t.rules.rhs.type = t.outputs
}
```

But now Alloy shows another issue: the values that we want to collapse (again, see `README.md`) might not appear in `rules`.
We're going to state something slightly stronger than we need here since we'll need this exact rule in the app:

```alloy
fact "all collapses appear in the left-hand side of rules" {
  all t: Table | all v: t.wantToCollapse | v in t.rules.lhs
}
```

After this, I can't find anything in the model that looks like it shouldn't be possible.

So for our purposes, we should be OK if we can ensure:

1. The types on the LHS of every rule match the inputs (in order, which again wasn't modeled)
2. The types on the RHS of every rule match the outputs
3. The types in collapses match the types in the inputs
4. There are no duplicate rules in the LHS of the rules (that's the `one` in `Value one-> Value` above.)
