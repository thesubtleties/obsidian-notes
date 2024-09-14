#### [[Email Regex Example|Example - Email Address]]
## Basic Matchers

- `.`   - Any character except newline
- `\d`  - Digit (0-9)
- `\D`  - Not a digit (0-9)
- `\w`  - Word character (a-z, A-Z, 0-9, _)
- `\W`  - Not a word character
- `\s`  - Whitespace (space, tab, newline)
- `\S`  - Not whitespace

Note: These are like wildcard characters. For example, `\d` will match any single digit, while `\D` will match anything that's not a digit.

## Quantifiers

- `*`     - 0 or more
- `+`     - 1 or more
- `?`     - 0 or 1
- `{3}`   - Exactly 3
- `{3,}`  - 3 or more
- `{3,5}` - 3, 4, or 5

Note: These specify how many times something should appear. For instance, `a*` means "zero or more a's", while `a+` means "one or more a's".

## Positioning

- `^`     - Start of string
- `$`     - End of string
- `\b`    - Word boundary
- `\B`    - Not a word boundary

Note: These help you specify where in the text you want to match. `^hello` will match "hello" only at the start of a line.

## Character Classes

- `[abc]`   - A single character of: a, b, or c
- `[^abc]`  - Any single character except: a, b, or c
- `[a-z]`   - Any single character in the range a-z
- `[a-zA-Z]`- Any single character in the range a-z or A-Z

Note: These allow you to specify sets of characters. `[aeiou]` will match any single vowel.

## Special Characters

- `\`    - Escapes special characters
- `|`    - Alternation (or)
- `()`   - Grouping
- `\1`   - Backreference to group #1

Note: These have special meanings. For example, `cat|dog` means "match either cat or dog".

## Flags

- `g`    - Global search
- `i`    - Case-insensitive search
- `m`    - Multi-line search

Note: These modify how the search is performed. For instance, `i` makes the search ignore case, so `a` will match both "a" and "A".

## Common Patterns

- Email: `^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$`
- URL: `https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)`
- Date (YYYY-MM-DD): `^\d{4}-\d{2}-\d{2}$`
- Strong Password: `^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$`

Note: These are complex patterns for common use cases. The email pattern, for example, looks for something@something.something.

## Examples

1. Match a phone number:
   ```regex
   ^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$
   ```

2. Match an IP address:
   ```regex
   ^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$
   ```

3. Match a hex color code:
   ```regex
   ^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$
   ```

Note: These are real-world examples. The phone number pattern, for instance, allows for various formats like (123) 456-7890 or 123.456.7890.

## Lookahead and Lookbehind

- `(?=...)` - Positive lookahead
- `(?!...)` - Negative lookahead
- `(?<=...)`- Positive lookbehind
- `(?<!...)`- Negative lookbehind

Note: These are advanced features that let you match based on what comes before or after, without including it in the match. They're like peeking ahead or behind.

## JavaScript RegEx Methods

- `test()`: Returns true if the pattern matches, false otherwise
- `exec()`: Returns an array of match information or null
- `match()`: Returns an array of matches or null
- `replace()`: Replaces matches with a replacement string
- `search()`: Returns the index of the match, or -1 if not found

Note: These are ways to use regex in JavaScript. For example, `test()` is a simple way to check if a pattern exists in a string.