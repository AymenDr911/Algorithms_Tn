# Compiler Regression Test Matrix

**Reference:** `initial_test.html`, blob `427b1eac91bd094322074d77e203c898ca1e9987`.
**Status:** test plan derived from source inspection. Expected messages/scores must be captured from the original page before using these as automated golden tests.

## A. Structural parser and indentation

| ID | Input fragment / setup | Expected behavior |
|---|---|---|
| STR-01 | `Algorithme Exemple\nDébut\nFin` | No missing Début/Fin or unmatched-block fault; main header accepted. |
| STR-02 | Omit `Algorithme Exemple` | Conformity fault: first non-empty line must be `ALGORITHME Nom`. |
| STR-03 | Omit `Début` or `Fin` | Missing structural keyword is reported. |
| STR-04 | `Si x = 1 Alors\nFin Si` with no open `Si` | Parser reports a closer without matching opener. |
| STR-05 | Open `Si`, close with `Fin Pour` | Parser reports the mismatched/unclosed inner block. |
| STR-06 | `Pour i de 1 à n Faire\nFin Pour` | Recognized as a loop block; indentation follows nesting depth. |
| STR-07 | Open `Répéter`, close with `Jusqu’à condition` | Repetition block closes. |
| STR-08 | `Sinon` outside `Si`/`Selon` | Parser reports `Sinon` outside a valid block. |
| STR-09 | Omit `Alors` from `Si` | Conformity fault requiring `Alors` (or accepted next-line form). |
| STR-10 | Omit `Faire` from `Tant que` or `Pour` | Conformity fault requiring `Faire` (or accepted next-line form). |
| STR-11 | Nested `Si` inside `Pour`, both correctly closed | No structural mismatch; indentation depth is nested and returns to parent. |
| STR-12 | Indent a code line by 2 spaces with `IN=4` | Indentation fault is reported by `BAD`. |
| STR-13 | Run reindent on nested control flow and comments | Structure preserved; each line receives parser-derived indentation. |
| STR-14 | Empty algorithm editor | “Écris d’abord son algorithme”; conformity score is null, not 0/20. |

## B. Syntax and reserved conventions

| ID | Input fragment | Expected behavior |
|---|---|---|
| SYN-01 | `x = 3` as assignment | Fault: use `←` or `<-`. |
| SYN-02 | `x := 3` | Fault: `:=` is not the assignment operator. |
| SYN-03 | `Si x == 3 Alors` | Fault: equality is `=`, not `==`. |
| SYN-04 | `x <> 3` or `x != 3` | Fault: not-equal is `≠` or `#`. |
| SYN-05 | `x % 2` | Fault: use `mod`. |
| SYN-06 | Use `and`, `or`, `not`, `&&`, or `||` | Fault: use `Et`, `Ou`, `Non`. |
| SYN-07 | Use `print`, `input`, `range`, or `def` in pseudocode | Fault: Python keyword/function in algorithm. |
| SYN-08 | `Afficher(x)` / `Saisir(x)` | Fault unless this name is a declared module; use `Écrire`/`Lire`. |
| SYN-09 | `sqrt(x)`, `pow(x,2)`, or `x ** 2` | Fault: use prescribed `racinecarré(x)` / multiplication form. |
| SYN-10 | `racinecarré(x)` assigned to an `Entier` | Fault: result is real. |
| SYN-11 | `Pour i de 1 à n` | Check original array-index warning and preserve exact wording/severity. |
| SYN-12 | Call a predefined function with wrong arity | Fault; verify all arities against `FN`: racinecarre(1), abs(1), ent(1), arrondi(1), alea(2), ord(1), chr(1), long(1), pos(2), convch(1), estnum(1), valeur(1), sous_chaine(3), effacer(3), majus(1). |

## C. Declarations, types, and visibility

| ID | Setup / input | Expected behavior |
|---|---|---|
| DEC-01 | TDO name `nom_2` | Identifier accepted. |
| DEC-02 | TDO name `nom élève` or accented identifier | Fault: identifier must start with a Latin letter and contain only letters/digits/underscore. |
| DEC-03 | Use undeclared variable `total` | Fault: undeclared in visible TDO/parameters. |
| DEC-04 | Declare variable globally and use it in a module | Variable is visible from the module. |
| DEC-05 | Use another module's local-only variable | Fault: not in caller's visible scope. |
| DEC-06 | Declare parameter `x` again in module-local TDO | Fault: parameter is already declared. |
| DEC-07 | Declare a TDNT alias and use it as a variable type | Alias is resolved according to original type normalization. |
| DEC-08 | Index a declared array `T[i]` | Accepted if declaration is an array. |
| DEC-09 | Index a scalar `n[i]` | Fault: scalar is not an array. |
| DEC-10 | Assign an entire array `T ← U` | Fault: assign element by element. |
| DEC-11 | Assign to a string element `S[i] ← c` | Fault: string cannot be modified character-by-character. |
| DEC-12 | `Lire(T)` for array `T` | Warning: read elements individually. |
| DEC-13 | Display an entire array | Warning: display elements individually. |
| DEC-14 | TDO declares module `Calcul` as function but workspace is procedure | Fault: declared kind and workspace kind mismatch. |
| DEC-15 | Declared module has no workspace | Warning on the main program. |

## D. Functions and procedures

| ID | Setup / input | Expected behavior |
|---|---|---|
| MOD-01 | `Fonction Pair(n: Entier): Booléen` | Header parses; result type is simple and recognized. |
| MOD-02 | Function has no result type | Fault. |
| MOD-03 | Procedure declares `: Entier` | Fault: procedure has no result type. |
| MOD-04 | Parameters separated by semicolons | Fault: use commas. |
| MOD-05 | Parameter missing its own type | Fault. |
| MOD-06 | Function has no `Retourner` | Fault. |
| MOD-07 | Last non-empty statement before `Fin` is not `Retourner résultat` | Fault: return must be immediately before `Fin`. |
| MOD-08 | More than one `Retourner` | Warning. |
| MOD-09 | `Retourner` in main program or procedure | Fault. |
| MOD-10 | Function called as a standalone statement | Fault: function result must be used. |
| MOD-11 | Procedure call used as an expression | Fault. |
| MOD-12 | Wrong argument count/type | Fault. |
| MOD-13 | `@x` receives a literal/expression | Fault: by-reference parameter needs a variable or array cell. |
| MOD-14 | Correct arguments but incorrect order | Fault when source can establish mismatch. |
| MOD-15 | Module call uses different casing | Warning; exact header spelling is recommended. |
| MOD-16 | Valid module is never called | Warning. |
| MOD-17 | Python module definition missing or has different parameter count | Python/report warning from `PYB`. |
| MOD-18 | Python function lacks `return`; Python procedure returns a value | Corresponding `PYB` warning. |

## E. Score and report completeness

| ID | Setup | Expected behavior |
|---|---|---|
| SCO-01 | Zero `CF` faults | Conformity = 20/20. |
| SCO-02 | One `CF` fault | Conformity = 18/20. |
| SCO-03 | Ten or more `CF` faults | Score floor is 0/20, never negative. |
| SCO-04 | Add only a warning (`p:0`) | Conformity score unchanged. |
| SCO-05 | Structural parser issue but no added `CF` fault | Message is shown; verify score follows the original implementation's separate scoring path. |
| SCO-06 | Completeness checks: all five satisfied | 100%. |
| SCO-07 | Completeness checks: two of five satisfied | 40%. |
| SCO-08 | Algorithm has at most three non-empty lines and no indentation | The original completeness indentation criterion can still pass. |
| SCO-09 | Objects declared but some have no type | Completeness warning and criterion must reflect `ty`. |
| SCO-10 | Python code missing / NumPy import missing / lists detected | Report warning behavior retained; not conflated with conformity score. |

## F. Golden-test capture procedure

For each test case:
1. Run the input through the original HTML in a controlled browser environment.
2. Record the exact messages, severity, line numbers, conformity score, and report completeness.
3. Save the input, expected output, and source revision as a fixture.
4. Run the same fixture against the migrated engine.
5. Require exact expected severity and score; normalize only cosmetic whitespace if explicitly documented.

Do not infer semantic correctness for a free-form algorithm. Without a supplied exercise or teacher-defined rubric, the compiler can verify syntax, structure, declarations, and conventions, but cannot truthfully grade whether an arbitrary algorithm solves a particular problem.
