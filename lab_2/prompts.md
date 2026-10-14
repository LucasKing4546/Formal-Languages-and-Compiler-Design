# Lab 2 – Prompts and responses

## Prompt 1

> create an md file which will include the prompts i give you and the responses you will give, please check the if my BNF is valid and if my 2 programs are correct

## Response 1

Files checked: `Data processing.txt` (BNF), `program_1.txt`, `program_2.txt`.

**Verdict:** the BNF is almost valid: it has 3 notation errors and 1 real design gap (block bodies have no end). Both programs can be derived from the grammar, but program 2 relies on indentation, which the grammar does not describe.

### BNF: errors to fix

1. **Line 12, unquoted comma.** In `"getIndex" "(" <id>, <id> ")"` the `,` is a terminal and must be quoted:

   ```
   "getIndex" "(" <id> "," <id> ")"
   ```

2. **Line 31, `"""` is not a well-formed terminal.** Quote the double quote with single quotes:

   ```
   <string> ::= '"' <string_content> '"'
   ```

3. **Line 19, `…` is not BNF.** Strict BNF needs all 52 letters listed. Many courses accept the ellipsis as shorthand, so check what yours expects.

4. **Lines 48–50, block bodies never end.** `<if>`, `<for>` and `<while>` end in `<statements>`, which has no terminator, so every statement after an `if` can belong to its body. The grammar is ambiguous here, and indentation is not part of it. Simplest fix is a closing keyword:

   ```
   <if>    ::= "if" <condition> ":" <statements> "end"
   <for>   ::= "for" <id> "in" <id> ":" <statements> "end"
   <while> ::= "while" <condition> ":" <statements> "end"
   ```

   The alternative is to keep indentation and state that the lexer produces `INDENT` / `DEDENT` tokens, then use them in these three rules.

### BNF: ambiguities and redundancies

These do not make the grammar invalid, but they are worth cleaning up.

- **Lines 7 and 15, `""` alternatives.** `<identifier_rest>` and `<digits_opt>` each allow the empty string next to non-empty alternatives, so `1` or `a` has more than one derivation. Dropping `""` and reusing `<digits>` is enough:

  ```
  <identifier>      ::= <letter> | <letter> <identifier_rest>
  <identifier_rest> ::= <letter> | <digit> | <letter> <identifier_rest> | <digit> <identifier_rest>
  <number>          ::= "0" | <nonzero_digit> | <nonzero_digit> <digits>
  ```

- **`getIndex` is defined twice.** It is in `<function>` (one argument) and also has its own two-argument alternative on line 12. Remove it from `<function>`.
- **`isEmpty` is defined twice.** It is in `<function>` (so it is a `<term>`) and also a special case in `<comparison>`. Keep one.
- **`<id> ::= <identifier>`** is a pure alias; one of the two names is enough.
- **Keywords are valid identifiers.** `in`, `out`, `sort`, `asc`, `and` and the rest all match `<identifier>`. Add a note that they are reserved words.
- **Whitespace and newlines are not defined.** The grammar mixes character-level rules (`<letter>`, `<digit>`) with token-level rules and never says what separates tokens or statements. A one-line note that tokens are separated by whitespace covers it.

### BNF: limitations

These are legal, just restrictive. Change them only if you want the features.

- `<value_expr>` has no precedence and is right-recursive, so `a - b - c` parses as `a - (b - c)` and `*` does not bind tighter than `+`. There are no parentheses.
- `<comparison>` compares `<term>`s only, so `a + 1 > 3` is not allowed.
- There are no negative numbers and no empty string (`<string_content>` needs at least one character).
- `<append_arg>`, `<insert>` and `<array>` accept only literal values, so `grades.append(x)` and `[a, b]` are not allowed.
- `remove` and `insert` take a literal `<number>` as index, never a variable.

### Program 1

Syntactically correct: every line derives from the grammar.

| Line | Rule |
|---|---|
| `array = []` | `<assign>` → `<value_expr>` → `<term>` → `<array>` |
| `load array "input.csv"` | `<load>` |
| `sort array desc` | `<sort>` |
| `remove array 0` | `<remove>` |
| `reverse array` | `<reverse>` |
| `filter array x => x > 10 and x <= 100` | `<filter>` with `<comparison> <logical_op> <condition>` |
| `distinct array` | `<distinct>` |
| `out array` | `<output>` |
| `save array "out.csv"` | `<save>` |

One runtime note: `remove array 0` fails on an empty array if `input.csv` has no rows.

### Program 2

Every line derives from the grammar, with one caveat.

- **Caveat: the indented blocks (lines 8–11).** By the grammar as written, the `if` body does not stop at `grades.append(0)`; it can absorb the `while` and everything after it. The program is only unambiguous once fix 4 above is applied. With the `end` version it becomes:

  ```
  if isEmpty(grades):
      grades.append(0)
  end
  while size(grades) > 3:
      remove grades 3
  end
  ```

- The remaining lines are fine: `in bonus`, `map grades g => g + bonus`, `filter ...`, `copy grades top`, `insert top 0 10`, `avg = sum(top) / size(top)`, `out max(top) - min(top)`, `out count(top, 10)`, `save top "top3.csv"`.

Two logic notes, not syntax errors:

- After the `while`, `grades` has at most 3 elements, but `insert top 0 10` then makes `top` up to 4 elements, and the file is named `top3.csv`. It also adds a 10 that may not be in the data, so `max(top)` is always 10 and `count(top, 10)` is always at least 1.
- The grammar does not say whether `/` is integer or float division, so the value of `avg` is undefined by the spec.
