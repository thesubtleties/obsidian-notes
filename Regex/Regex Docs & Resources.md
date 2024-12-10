## Official Documentation
### Language-Specific Regex
- [Python re module](https://docs.python.org/3/library/re.html)
- [JavaScript RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
- [Java Pattern](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)

## Interactive Learning & Testing
- [Regex101](https://regex101.com/) - Interactive testing with explanation
- [RegExr](https://regexr.com/) - Learning and testing tool
- [Debuggex](https://www.debuggex.com/) - Visual regex debugging
- [RegexCrossword](https://regexcrossword.com/) - Learn through puzzles

## Video Resources
- [Regular Expressions (Regex) Tutorial](https://www.youtube.com/watch?v=sa-TUpSx1JA) - Corey Schafer
- [Learn Regular Expressions In 20 Minutes](https://www.youtube.com/watch?v=rhzKDrUiJVk) - Web Dev Simplified
- [Regex Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9g6m_6Sld9Q4jzqdqHd2HiD) - The Net Ninja

## Basic Patterns
```regex
# Character Classes
\d - digit [0-9]
\w - word character [a-zA-Z0-9_]
\s - whitespace
. - any character except newline

# Quantifiers
* - zero or more
+ - one or more
? - zero or one
{n} - exactly n times
{n,} - n or more times
{n,m} - between n and m times

# Position
^ - start of line
$ - end of line
\b - word boundary

# Groups and Ranges
[abc] - any of a, b, or c
[^abc] - not a, b, or c
[a-z] - any letter a through z
(abc) - capture group
(?:abc) - non-capturing group
```

## Common Use Cases
### Email Validation
```regex
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$
```

### URL Matching
```regex
https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)
```

### Phone Numbers
```regex
# US Phone Numbers
^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$
```

## Advanced Concepts
### Lookaheads & Lookbehinds
```regex
(?=...) - Positive lookahead
(?!...) - Negative lookahead
(?<=...) - Positive lookbehind
(?<!...) - Negative lookbehind
```

### Flags/Modifiers
```regex
g - Global search
i - Case-insensitive
m - Multiline
s - Dot matches newline
u - Unicode
y - Sticky search
```

## Language-Specific Examples
### Python
```python
import re

# Search
pattern = r'\b\w+@\w+\.\w+\b'
text = "Contact us at: info@example.com"
match = re.search(pattern, text)

# Replace
new_text = re.sub(r'old', 'new', text)

# Find all matches
matches = re.findall(pattern, text)
```

### JavaScript
```javascript
// Test
const pattern = /\b\w+@\w+\.\w+\b/;
pattern.test("info@example.com");

// Match
"info@example.com".match(/\b\w+@\w+\.\w+\b/);

// Replace
"old text".replace(/old/, "new");
```

## Tools & Resources
### Online Tools
- [RegexBuddy](https://www.regexbuddy.com/)
- [Regex Generator](https://regex-generator.olafneumann.org/)
- [iHateRegex](https://ihateregex.io/) - Common regex patterns
- [Regex Tester](https://www.regextester.com/)

### Cheat Sheets
- [Regular-Expressions.info](https://www.regular-expressions.info/refquick.html)
- [RegexOne](https://regexone.com/)
- [RexEgg](https://www.rexegg.com/)

## Best Practices
1. Use Readable Patterns
```regex
# Bad
\w+@\w+\.\w+

# Better
[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}
```

2. Use Named Groups
```regex
(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})
```

3. Comment Complex Regex
```regex
(?x)                    # Enable comments
(\d{4})                # Year
-                      # Separator
(0[1-9]|1[0-2])       # Month
-                      # Separator
(0[1-9]|[12]\d|3[01]) # Day
```

## Performance Considerations
- Avoid excessive backtracking
- Use atomic groups where possible
- Be specific with character classes
- Avoid nested quantifiers
- Use non-capturing groups when capture isn't needed

## Common Patterns Library
```regex
# Dates
YYYY-MM-DD: ^\d{4}-(?:0[1-9]|1[0-2])-(?:0[1-9]|[12]\d|3[01])$

# Time
HH:MM: ^(?:[01]\d|2[0-3]):[0-5]\d$

# Credit Cards
Visa: ^4[0-9]{12}(?:[0-9]{3})?$
MasterCard: ^5[1-5][0-9]{14}$

# Passwords
# At least 8 chars, 1 uppercase, 1 lowercase, 1 number
^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{8,}$
```

## Testing Strategies
1. Test Edge Cases
2. Test Invalid Inputs
3. Test Performance with Large Inputs
4. Test Unicode Support
5. Test Multiline Input

Remember to:
- Keep patterns simple and readable
- Test thoroughly
- Document complex patterns
- Consider performance implications
- Use appropriate flags
- Validate user input
- Handle edge cases
- Use proper escaping
- Consider Unicode support