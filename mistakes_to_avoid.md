## 1. STRUCTURE & ORDERING

### 1.1 Definition Order

- Check if the DSL requires a specific order (e.g., metadata first, types before entities, imports before declarations)
- Some DSLs require all content in one file with **no imports allowed**
- Never violate the grammar's required order for top-level elements

### 1.2 Top-Level Requirements

- Models may need to start with specific elements (e.g., `connector` blocks, header metadata in `[...]`)
- **No orphan declarations**: Never start code with content blocks if the grammar expects metadata/configuration first
- Check if the grammar allows starting directly with business logic or requires scaffolding

### 1.3 Completeness

- Every construct must be **fully completed** before starting the next one
- Never leave blocks incomplete, empty, or truncated unless the grammar explicitly allows it
- **Priority**: Fewer complete items > many incomplete items
- Before emitting a closing delimiter (`}`, `]`, `endif`, `end`), verify all required inner sections exist

### 1.4 Mandatory Fields

- **When defining an element, any field or parameter without `?` (optional marker) in the grammar must always be
  generated**
- Example: If grammar shows `deployment=Deployment` (no `?`), the `deployment` field is mandatory
- Example: If grammar shows `('config_server' '=' config_server=FQN)?`, the `config_server` field is optional
- Even if a field seems logically redundant or obvious, if the grammar requires it, include it

---

## 2. WHITESPACE

**Always respect the grammar's exact whitespace requirements:**

- **Spaces before symbols**: Type parameters often need space before `<`: `algorithm <Type>` NOT `algorithm<Type>`
- **Spaces around operators**: Check if `=`, `:=`, `<-` require surrounding spaces (e.g., `x = y` vs `x=y`)
- **Spaces after delimiters**: Commas typically need space after: `[a, b, c]` NOT `[a,b,c]`
- **No spaces in compounds**: `@pre` NOT `@ pre`, `endif` NOT `end if` (unless grammar uses two words)
- **Context matters**: The same symbol may have different spacing rules in different positions

When in doubt, check the grammar or reference examples.

---

## 3. DELIMITERS & PAIRING

### 3.1 Block Delimiters

Different DSLs use different block syntax patterns. **Never assume - always check the grammar**:

**Pattern A: Braces**

```
block_name {
    field = value
}
```

**Pattern B: Colons (often with indentation for readability)**

```
block_name:
    field = value
    nested_block:
        sub_field = value
```

**Pattern C: Keywords**

```
if condition then
    action
endif
```

**Pattern D: Other delimiters**

- Some DSLs may use `begin...end`, `do...done`, `[...]`, or other paired keywords
- Always verify the exact syntax in the grammar

### 3.2 Match All Delimiters

Every opening must have exactly one closing:

- `(` with `)`
- `[` with `]`
- `{` with `}`
- `<` with `>`
- `if` with `endif` (or `fi`, `end if` depending on DSL)
- `let` with `in`
- `typedef` with `end` (or `]` depending on DSL)

### 3.3 Nested Structures

- Each nested structure requires its **own** closing delimiter
- Count your nesting levels and ensure each has a matching closer
- Example: Nested `if` statements each need their own `endif`:

```
  if condition1 then
      if condition2 then
          action
      endif    // closes condition2
  endif        // closes condition1
```

- Common mistake: Using one `endif` for multiple `if` statements
- Before closing any block, verify all inner blocks are already closed
- Work inside-out: close the innermost structure first, then work outward

---

## 4. IDENTIFIERS & NAMING

### 4.1 Valid Characters

- **TextX `ID` rule**: By default, identifiers must start with a letter or underscore, followed by letters, digits, or
  underscores
- **No dots**: `my.variable` is NOT a valid `ID` (it's two IDs separated by a dot)
- **No spaces**: `my variable` is NOT a valid `ID` (it's two separate IDs)
- **No hyphens**: Use underscores for word separation: `my_variable` ✅, `my-variable` ❌
- **IDs in brackets**: If the grammar requires `[id]` syntax, the same `ID` rules apply (e.g., `[under_50]` ✅,
  `[50_150]` ❌)

### 4.2 Quoted vs Unquoted Identifiers

- **Unquoted identifiers (`ID`)**: Keywords, enum values, variable names, field references
    - Example: `condition = congruent`, `type = StandardScaler`
- **Quoted strings (`STRING`)**: Human-readable text, file paths, literal content
    - Example: `name = "My Component"`, `path = "/data/file.txt"`
- **Never quote identifiers that should be keywords or enum values**:
    - ❌ `condition = "congruent"` (if `congruent` is a keyword)
    - ✅ `condition = congruent`
- **Check the grammar** to determine if a context expects `ID` or `STRING`

### 4.3 Reserved Words

- Never use category names as identifiers
- Don't use structural keywords as content

---

## 5. TYPE REFERENCES & ENUMS

### 5.1 Use Concrete Type Names, Not Abstract Categories

- If the grammar defines specific type literals (enum values), use them exactly
- Never use placeholder or category names that don't exist in the grammar
- Example: If grammar has `StandardScaler | MinMaxScaler`, use those exact names, not `<scaling_method>`

### 5.2 Exact Matches Required

- Type names must match the grammar exactly (case-sensitive, character-for-character)
- Check the grammar for the complete list of valid type names
- Do not create your own type names or assume the grammar will accept variations

### 5.3 Avoid Redundant Type Information

- Don't add `name` or `type` parameters that duplicate information already specified by the type itself
- If the construct's type is already declared, don't repeat it in parameters

---

## 6. LITERALS & DATA TYPES

### 6.1 Type Constraints and Tokenization

- **Type restrictions**: Constraints often accept only specific types (e.g., INT or STRING, but not floats)
- **Decimal points vs range operators**: Set{10.00..10000.00} may fail (decimal conflicts with ..)
    - Use integers: Set{10..10000}
- **Hyphens as minus vs separator**: 50-150 in content may tokenize as subtraction
    - Rephrase in strings: "Between 50 and 150 miles"
    - In identifiers: NOT allowed (use underscores)
- **General rule**: Avoid patterns in literal content that resemble operators or structural syntax

### 6.2 Collections and Separators

- **List separators**: Check grammar - commas `[a, b, c]`, spaces `a b c`, semicolons `a; b; c`, or newlines
- **List delimiters**: Some DSLs use `[...]`, others use no brackets at all
- **Verify bracket support**: Not all DSLs support `[]` syntax

### 6.3 NEVER ASSUME PYTHON-STYLE DATA STRUCTURES

- **DO NOT** use Python tuples `(a, b)` unless grammar explicitly supports them
- **DO NOT** use Python dicts `{key: value}` unless that's the DSL syntax
- **DO NOT** use Python lists `[a, b, c]` unless brackets are in the grammar
- **ALWAYS** use DSL-native data structures and syntax as defined in the grammar

---

## 7. OPERATORS

### 7.1 Context-Specific Operators

Don't assume operators from other languages work the same way:

- **Composition**: `+`, `|`, `>>`, `->` (check grammar)
- **Assignment**: `=`, `:=`, `<-`, `:` (check grammar)
- **Access**: `.`, `->`, `::`
- **Always verify each operator in the grammar** - syntax varies widely between DSLs

---

## 8. INSTANTIATION & COMPOSITION

### 8.1 Check Instantiation Style

DSLs vary widely in how elements are created:

**Pattern A: Named declarations first, then references**

```
preprocessor <StandardScaler> scaler
algorithm <ARFClassifier> model
```

**Pattern B: Inline constructors**

```
pipeline = Scaler() | Model()
```

**Pattern C: Key-value composition (no constructors)**

```
composer:
    scaler = StandardScaler
    model = ARFClassifier
```

**Note on Pattern C and parameter assignment:**
Even within declarative key-value style, parameter assignment syntax varies:

- Space: `shape rectangle`
- Equals: `shape = rectangle`
- Colon: `shape: rectangle`

**Always check the grammar for the exact syntax expected.**

### 8.2 No Raw Constructors in Declarative DSLs

- If the DSL is **declarative**, it likely doesn't support function-call syntax
- Use **named parameter assignment** instead:

```
# ❌ Wrong (function-style)
composer = Pipeline(StandardScaler(), ARFClassifier())

# ✅ Correct (declarative key-value)
composer:
    scaler = StandardScaler
    model = ARFClassifier
```

### 8.3 Chaining and Composition

- Check if the DSL supports **direct chaining** (`a | b | c`)
- If not, wrap in a composer/pipeline construct:

```
# ❌ Wrong (if chaining not supported)
preprocessing = scaler | normalizer

# ✅ Correct
preprocessing:
    pipeline:
        scaler = StandardScaler
        normalizer = Normalizer
```

### 8.4 Standalone Definitions

- Some DSLs require you to **define elements as standalone named objects** before referencing them elsewhere
- Don't inline everything - check if the grammar expects separate declarations

---

## 9. CONDITIONS & CONTROL FLOW

### 9.1 Complete Condition Syntax

- Bare keywords may not be valid: `congruent` alone is incomplete
- Must form complete expressions: `condition = congruent`

### 9.2 Value References

- Remove quotes from condition values if they should be identifiers
- Check if boolean/enum values are keywords or strings

---

## CRITICAL REMINDERS

1. **THE GRAMMAR IS YOUR SOURCE OF TRUTH** - Always consult it before making assumptions
2. **When in doubt, check examples** - Reference implementations are authoritative
3. **Prioritize correctness over brevity** - A verbose but valid model is better than a concise but broken one
4. **Build incrementally** - Validate small pieces before combining into complex structures
5. **Never mix syntax from different languages** - Respect the DSL's native constructs and conventions

