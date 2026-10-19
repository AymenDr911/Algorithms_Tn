# Compiler Rules Specification — Algorithms_Tn

**Source of truth:** `initial_test.html` at blob `427b1eac91bd094322074d77e203c898ca1e9987`.
**Purpose:** preserve the existing deterministic French-language algorithm compiler while removing the fixed exercise/quiz flow. This document records observed behavior; it is not a claim that every legacy check is bug-free.
**Runtime constraint:** compiler and assessment must work without an AI API or network request.

## 1. Product behavior to preserve

- A student can start writing immediately; no predefined exercise, pasted statement, or upload is required.
- Keep French educational terminology and the existing visual design direction, with responsive mobile/tablet/desktop layout.
- The main program and each function/procedure have separate workspaces.
- The compiler reports line-specific faults and warnings. A warning is not a conformity fault.
- Keep the conformity score out of 20 separate from report completeness as a percentage.
- Keep Python editing/execution and detailed report as distinct capabilities. Do not claim Python execution is available until the local runtime is integrated and regression-tested.
- Persisting reports and teacher access are SaaS features; they must not alter compiler semantics.

## 2. Algorithm text and parser conventions

Observed in `ana(txt)`, `cmp(id)`, `ckRun(id, fx)`, `reind(t)`:

- Empty lines are allowed. Lines beginning with `/` are treated as comments by the structural parser; the conformity checker also strips `//...` comments.
- Assignment-like lines are recognized by the structural parser using `<-`, `←`, or `:=`, so these lines do not open control blocks.
- Recognized block openers: `Début`, `Si`, `Pour`, `Tant que`, `Répéter`, `Selon`.
- Recognized closers: `Fin` (closes `Début`), `Fin Si`, `Fin Pour`, `Fin Tant que`, `Fin Selon`, and `Jusqu…` (closes `Répéter`).
- `Sinon` is valid structurally only inside `Si` or `Selon`; `Alors` and `Faire` are treated as continuation/header lines for indentation.
- The parser reports a closer without a matching opener and a still-open block at end of input. It also reports when a closer crosses an inner unclosed block.
- Reindentation is based on parser depth and the configured indentation unit `IN`; preserve this behavior only after testing nested blocks, comments, empty lines, and mixed closers.
- Main program's first non-empty line must be `Algorithme Nom` (identifier starts with a letter; remaining characters may be letters, digits, or underscore).
- Both main program and module must contain `Début` and `Fin`.
- One module per module workspace. A module workspace begins with `Fonction` or `Procédure`; `Algorithme` is reserved for the main program.

## 3. Identifier, declaration, and visibility rules

- TDO identifiers must match Latin letter followed by letters/digits/underscore; no accents or spaces.
- Variables and objects must be declared in a visible TDO: global TDO of the main program, local TDO of a module, or the module's parameters. A module's effective arguments must be visible in the caller's context.
- The original checker validates unknown names and suggests a similarly named module where a close match is found.
- A parameter must not be redeclared in a module's local TDO.
- Module names must be unique, and must not collide with reserved words or predefined functions.
- Main-program global TDO declarations identify function/procedure names; these declarations must match actual module workspaces and their kind.
- The source supports new type declarations in TDNT, including resolving aliases when identifying array types. Preserve the original type normalization/alias behavior; validate the complete TDNT grammar from the source before reimplementation.
- Supported simple type vocabulary in diagnostics: `Entier`, `Réel`, `Booléen`, `Caractère`, `Chaîne` / `Chaîne de caractères`. The source also handles arrays and user-defined types.
- Do not permit a string to be assigned character-by-character. Arrays are accessed by index; whole-array assignment is rejected. Direct `Lire(tableau)` and printing a whole array produce warnings recommending element-by-element access.

## 4. Function and procedure header rules

- Header starts with `Fonction Nom(...)` or `Procédure Nom(...)`.
- Parameters are comma-separated, not semicolon-separated; each parameter has its own type and follows `[@]nom: Type`. The optional `@` denotes a by-reference parameter.
- Missing parentheses may be accepted with a warning, recommending `Nom()`; malformed or unclosed parentheses are faults.
- A function must declare a single simple result type after `): Type`; a procedure must not declare a result type.
- Parameter types must be known simple types or declared TDNT types.
- A function must have `Retourner Résultat` immediately before `Fin`; more than one `Retourner` is a warning. A function return expression is checked against its declared type where inference is possible.
- `Retourner` is invalid in the main program and in a procedure.
- Calls are checked for module existence, exact name casing (warning), argument count, argument types, argument order, and by-reference arguments being variables/array cells rather than expressions.
- A function call used as a standalone instruction is a fault; its result must be assigned/used. A procedure call is an instruction and must not be used as a value.
- Unused modules trigger a warning.

## 5. Operators and control-structure conventions

- Assignment is `←` (or `<-`); `:=` and assignment written with `=` are faults.
- Equality uses `=`, not `==`.
- Not-equal uses `≠` or `#`, not `!=` or `<>`.
- Remainder uses `mod`, not `%`.
- Logical operators are `Et`, `Ou`, `Non`, not `and`, `or`, `not`, `&&`, `||`, or `!`.
- Python keywords/functions (for example `print`, `input`, `len`, `range`, `def`, `if`, `for`, `while`, `return`, `import`, `append`) are rejected inside algorithm text.
- Use `Écrire(...)`, `Écrire_nl(...)`, and `Lire(...)`; alternate terms such as `Afficher` or `Saisir` are rejected unless they are a declared module name.
- `Si` / `Sinon Si` requires `Alors`; `Tant que` requires `Faire`; `Pour` follows `Pour Compteur de Début à Fin [Pas = valeur] Faire`; `Jusqu’à` requires a condition.
- The `Selon` selector must be of type `Entier` or `Caractère`.
- `racinecarré(x)` is the prescribed square-root form; `sqrt`, `pow`, `math`, and `**` are rejected. A result assigned from `racinecarré` must not be stored in an integer variable.
- Calls to known predefined functions are checked for expected argument counts. The exact predefined-function table is defined in the original source and must be ported unchanged (do not infer or replace it).
- Parentheses are checked for balance. Expression type inference handles basic literals, variables, indexed arrays/strings, comparisons/logical operators, and basic arithmetic; inference may be inconclusive for complex expressions.

## 6. Faults, warnings, and scores

- `CF(i)` returns entries marked either fault (`p: 1`) or warning (`p: 0`).
- Main/module conformity score: `max(0, 20 - 2 × number of conformity faults)`. Warnings do not reduce the score.
- Structural parser faults from `ana` are added to the displayed message list by `cmp`, but the score calculation is based on the count of `CF` fault entries. This distinction must be preserved unless a deliberate, tested policy change is approved.
- Empty editor returns a message and no score.
- Report completeness is a separate heuristic in `an(i)`: it checks presence of Début/Fin, at least one control structure, at least one object declared with types, Python code present, and indentation present (or a short algorithm of three non-empty lines or fewer). The percentage is the count of satisfied checks divided by five, rounded to an integer.
- Report analysis also warns about Python lists instead of NumPy arrays, missing NumPy import, objects without types, absent control structures, missing Python, and other checks from `PYB(i)`. These are report warnings, not the conformity score.
- The original source's conformity score is labelled for “conventions 2SI et 2024”. Do not relabel this as semantic correctness of an arbitrary exercise.

## 7. Python and report behavior in the original HTML

- Original Python execution uses Skulpt in the browser, with execution/yield limits and a custom limited NumPy compatibility layer. It is not full CPython/NumPy. Preserve offline/no-AI execution, but test its supported subset and clearly communicate limitations.
- Original report includes student identity/contact fields, time, work for both fixed exercises, completeness, line/indentation metrics, loops/conditions/declarations/new types, conformity, warnings, algorithm text, conformity findings, Python source, and latest execution state.
- Fixed exercise statements and quiz-specific scoring are out of scope for the new free-writing workspace and must be removed. Generic rule validation remains.
- The legacy `deposit()` references `window.claude.use('db')` and `window.claude.use('user')`; this is not a dependable standard SaaS backend. Replace it with authenticated server-side persistence and explicit student/teacher authorization. Never expose a service-role secret in browser code.

## 8. Regression-test requirements before claiming parity

Create positive and negative tests for every category above, including:
1. Correct minimal main algorithm; missing `Algorithme`, `Début`, or `Fin`.
2. Every block opener/closer, nested blocks, invalid closer, unmatched block, `Sinon`, `Alors`, `Faire`, indentation/reindentation.
3. Assignment and forbidden operator variants; Python keywords accidentally used in pseudocode.
4. Valid/invalid identifiers, undeclared variables, global/local visibility, parameters and duplicate declarations.
5. TDNT aliases, array indexing, string indexing, whole-array operations, and simple expression types.
6. Valid/invalid function/procedure headers, result types, return placement, parameter direction, argument count/type/order, case mismatch, and unused modules.
7. Each predefined function's accepted arity and representative invalid arities.
8. Conformity score boundary (0 faults = 20; 1 fault = 18; floor at 0) and warnings not reducing score.
9. Report-completeness percentage independently of conformity.
10. Python supported-subset behavior, syntax/runtime errors, execution limits, and output capture.

**Release gate:** do not claim “same reliability” until the migrated compiler passes this suite against expected outcomes captured from the original HTML. First inspect possible legacy defects, record them, and change only with explicit tests.
