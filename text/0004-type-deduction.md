- Feature Name: type_deduction
- Start Date: 2020-10-13
- RFC PR: [nushell/rfcs#0004](https://github.com/nushell/rfcs/pull/4)
- Nushell Issue: [nushell/nushell#0000](https://github.com/nushell/nushell/issues/0000)

# Summary

[summary]: #summary

The purpose of this RFC is to explore how type deduction can be implemented for nushell. Type deduction is helpful for e.G. aliases with typed variables, static analysis...

# Motivation

[motivation]: #motivation

Currently no module for the sake of type deduction is implemented. Groundwork for such a module has been layed out in alias.rs. However, the code in alias.rs doesn't handle all cases.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Given a set of variables and a block of code, the purpose of the type deduction module (TDM) is to infer all possible types a variable can have so that the block of code is still valid. In nushell a "type" is represented by a SyntaxShape variant.
The TDM infers by looking in which kind of expression (following: containing expression) a variable is used. Then it checks which expressions (following: sub expressions) are allowed inside the containing expression at the position of the variable and maps them to their corresponding SyntaxShape variant.
For most cases it is sufficient to look at the nushell grammar, to figure out the allowed sub-expressions. E.G. `echo 1..$var` $var is of type Int as only Int is allowed in a range expression.
Special cases are listed below.

## As a commands mandatory positional argument | As a argument to flag 
```shell Example positional argument
ls | where $filter
$filter -> SyntaxShape::Math
```
```shell Example named argument
cal --full-year $year
$year -> SyntaxShape::Int
```

The shape of a variable used as a positional argument can be infered from the command signature.
If the signature of the command is not available, no inference will be done. At the time of writing, no signature will be available for external commands.

## As a commands optional positional argument
Note: At the time of writing this rfcs, optional positional arguments are parsed in order. Meaning: No optional positional argument can be left out. Trickier deductions might be needed, if the parser would handle optional arguments differently.
```shell One optional variable 
git log signature: git log [<options>] [<revision range>] [[--] <path>...]
git log $arg ./src/
```
One can infer that $arg has to be a revision range, as there is no other possibility.

```shell Multiple variables
cmd signature: cmd [<FilePath>] [<FileSize>] [<Block>] [<Int>...]
cmd $a1 $a2
```
One can infer that $a1 is FilePath and $a2 is FileSize

## As a part of a column in a table
Example
```shell
echo [[names, $ranking_col_name]; [$best_shell, 1] [fish, 2] [zsh, $zsh_ranking] [bash, 4]]
$ranking_col_name -> SyntaxShape::String
$best_shell -> SyntaxShape::Any
$zsh_ranking -> SyntaxShape::Any
```
The types within a column are heterogeneous. Therefore one can't deduce the type of a variable in a column. For variables in table headers, String seems as the most applicable SyntaxShape.

## As part of a binary expression
A distinction has to be made by the operator in use (and depending on the operator additionaly the side on which the variable appears).

### Operator && || (Logical Operators)
```shell Example
ls | where $it.name == LICENSE || $catch_all
$catch_all -> SyntaxShape::Boolean
```
The variable can be of any type that can be coerced to a boolean value.

### Operator In NotIn
#### Variable on right side
```shell Example
ls | where name in $values
$values -> SyntaxShape::Table
```
### Operator In NotIn
#### Variable on left side
```shell Example
ls | where $value in [...]
$value -> All SyntaxShapes present in the Table ([...])
```

### Operator Plus Minus
```shell Example 1
ls | where size < 1kb + $offset
$offset -> SyntaxShape::Unit
```
```shell Example 2
ls | where 1.5 < 1 + $var
$var -> SyntaxShape::Number, SyntaxShape::Int
```
The shape of the variable is Unit if other side of the binary expression is Unit, Number or Int otherwise.

### Operator Multiply Divide
```shell Example
ls | where size < 1.5 * $size
$size -> SyntaxShape::Unit
```
In general: the variable can be one of Int, Number, Unit.
The variable can't be Unit if the result expression has a undefined unit type (e.G. Unit * Unit or Int / Unit gives an undefined unit type for every unit nu currently implements).
The variable must be of Unit type, if the result type of the binary expression is used as a Unit type and the not variable side of the binary is a Number or Unit (see example above).

### Operator Contains NotContains
The variable must be of type String.


## Correct Inference for dependencies
Extra care has to be taken when deducing in binary expressions, with an Operator (like Plus, Minus, Multiply, Divide, In (var on lhs), NotIn (var on lhs)) where the variable type depends on the type of the other side.

```shell Variables on both sides
ls | where size < $a * $b; kill $a
$a -> SyntaxShape::Int
$b -> SyntaxShape::Unit
```
```shell Variable on one side, math expression with variables on other
ls | where size < $a * ($b * $c); kill $b $c
$a -> SyntaxShape::Unit
$b -> SyntaxShape::Int
$c -> SyntaxShape::Int
```

At the time of traversing the AST, any deduction for any variable may not be present, and if present may not be complete. Therefore constellations in which the deduction of one variable depends on the deduction of another one (we will call these constellations 'dependencies'), have to be postponed.

As soon as the AST is traversed, the dependencies have to be resolved.
Please note: One might think of a dependency as a edge in a directed graph, with variables as nodes. Every Node with no outgoing edge is completly deduced. Every Node with an edge towards such a completly deduced node can then be inferred. Nodes with cycles (e.G. `ls | where size < $a * $b` $a depends on $b, $b depends on $a) can't be deduced completly.
It is in question what to return here best dependend on the operator in use. One might solves this elegantly by returning the dependency and let the caller handle it (if needed). 
(Note: One might also think about this not as a graph problem, but as an CSP.)

## Merging of deductions.
Any variable can occur multiple times within the block. In each position the variable can have different possible Types.
```shell Example
config | where size < 1kb * $a; kill $a
$a -> SyntaxShape::Int
```
In the first position, $a might be SyntaxShape::Int or SyntaxShape::Number. In the second it is of SyntaxShape::Int.
The result set of possible types for a variable can be computed as the set intersection between the already deduced types and the new deduced types. If the intersection is empty, it means that the variable has usages with different and non compatible types. An error is returned.

```rust
fn checked_insert(existing_deductions, new_deductions) -> Result<set_of_deductions, ShellError>{
    if existing_deductions == None return new_deductions
    else return set_intersection(existing_deductions, new_deductions)?
}

existing_deductions = checked_insert(existing_deductions, new_deductions);
```

Special cases arise if one (or both) sets contains SyntaxShape::Any. SyntaxShape::Any is a placeholder for any possible type. The above pseudo code has to be changed to: 
```rust
fn checked_insert(existing_deductions, new_deductions) -> Result<set_of_deductions, ShellError>{
    if existing_deductions == None return new_deductions
    else match (has_any(existing_deductions), has_any(new_deductions)){
        (true, true) -> set_union_including_any(existing_deductions, new_deductions)
        (true, false) -> set_union_excluding_any(new_deductions, set_intersection(existing_deductions, new_deductions))
        (false, true) -> set_union_excluding_any(existing_deductions, set_intersection(existing_deductions, new_deductions))
        (false, false) -> set_intersection(existing_deductions, new_deductions)?
    }
}

existing_deductions = checked_insert(existing_deductions, new_deductions);
```

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

A start may be: https://github.com/nushell/nushell/pull/2685

# Drawbacks

[drawbacks]: #drawbacks

None I can think of.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

# Prior art

[prior-art]: #prior-art

# Unresolved questions

[unresolved-questions]: #unresolved-questions

## Deducing of result type of column paths
Currently it is not possible to infer the shape of column paths pointing to a table given by a prior command in the pipeline. 
Input and Output types of each command would be needed to support this deduction.
```shell $size nor $name are deducable
ls | where size == $size | get name | where $it == $name
```

It is yet not clear how to best integrate this into nushell. Some thoughts can be found at: https://github.com/nushell/nushell/pull/2486#issuecomment-687704131.

# Future possibilities

[future-possibilities]: #future-possibilities
