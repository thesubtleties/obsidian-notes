## Definition 📝
A Trie (pronounced "try" or "tree") is a tree-like data structure used to store a dynamic set of strings. Unlike a binary search tree, nodes in a trie don't store complete keys but rather parts of keys (typically individual characters). The path from the root to a node represents a specific prefix, and all descendants of that node share this prefix.

Tries are also known as **prefix trees** or **digital trees** because they excel at prefix-based operations.

## Complexity Analysis ⏱️
| Operation       | Time Complexity | Space Complexity |
|-----------------|-----------------|------------------|
| Insert          | O(m)            | O(m)             |
| Search          | O(m)            | O(1)             |
| Delete          | O(m)            | O(1)             |
| Prefix Search   | O(p) + O(n)     | O(1)             |

Where:
- m = length of the key/word
- p = length of the prefix
- n = number of words with the given prefix

## Core Operations 🔍
- **insert(word)**: Add a new word to the trie
- **search(word)**: Check if a word exists in the trie
- **startsWith(prefix)**: Check if any word starts with the given prefix
- **delete(word)**: Remove a word from the trie
- **getAllWords()**: Retrieve all words stored in the trie
- **getWordsWithPrefix(prefix)**: Get all words that start with a given prefix

## Basic Implementation 💻

```python
class TrieNode:
    def __init__(self):
        self.children = {}  # Dictionary to store child nodes
        self.is_end_of_word = False  # Flag to mark the end of a word

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        """Insert a word into the trie"""
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True
    
    def search(self, word):
        """Return True if the word is in the trie"""
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end_of_word
    
    def starts_with(self, prefix):
        """Return True if there is any word in the trie that starts with the given prefix"""
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
```

## Prefix Matching Implementation 🔍

```python
def get_words_with_prefix(self, prefix):
    """Return all words that start with the given prefix"""
    result = []
    node = self.root
    
    # Navigate to the node representing the prefix
    for char in prefix:
        if char not in node.children:
            return result  # Prefix not found
        node = node.children[char]
    
    # Use DFS to collect all words starting from this node
    self._dfs_with_prefix(node, prefix, result)
    return result

def _dfs_with_prefix(self, node, current_prefix, result):
    """Helper method to perform DFS and collect words"""
    if node.is_end_of_word:
        result.append(current_prefix)
    
    for char, child_node in node.children.items():
        self._dfs_with_prefix(child_node, current_prefix + char, result)
```

## Word Dictionary Implementation 📚

```python
class WordDictionary:
    def __init__(self):
        self.root = TrieNode()
    
    def add_word(self, word):
        """Add a word to the dictionary"""
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True
    
    def search(self, word):
        """
        Search for a word in the dictionary.
        Supports '.' as a wildcard that matches any character.
        """
        def dfs(node, index):
            if index == len(word):
                return node.is_end_of_word
            
            char = word[index]
            if char == '.':
                # Wildcard: try all possible characters
                for child in node.children.values():
                    if dfs(child, index + 1):
                        return True
                return False
            else:
                # Regular character
                if char not in node.children:
                    return False
                return dfs(node.children[char], index + 1)
        
        return dfs(self.root, 0)
```

## Autocomplete Implementation 🔮

```python
class AutocompleteSystem:
    def __init__(self, sentences, times):
        self.root = TrieNode()
        self.keyword = ""
        
        # Initialize with sentences and their frequencies
        for i, sentence in enumerate(sentences):
            self._add_sentence(sentence, times[i])
    
    def _add_sentence(self, sentence, count):
        node = self.root
        for char in sentence:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True
        node.count = node.count + count if hasattr(node, 'count') else count
        node.sentence = sentence
    
    def input(self, c):
        if c == '#':
            # End of input, add the sentence to the system
            self._add_sentence(self.keyword, 1)
            self.keyword = ""
            return []
        
        self.keyword += c
        node = self.root
        
        # Navigate to the node representing the current input
        for char in self.keyword:
            if char not in node.children:
                return []  # No matches
            node = node.children[char]
        
        # Collect all possible completions
        suggestions = []
        self._find_all_suggestions(node, suggestions)
        
        # Sort by frequency (count) and then lexicographically
        suggestions.sort(key=lambda x: (-x[1], x[0]))
        
        # Return top 3 suggestions
        return [item[0] for item in suggestions[:3]]
    
    def _find_all_suggestions(self, node, suggestions):
        if node.is_end_of_word:
            suggestions.append((node.sentence, node.count))
        
        for child_node in node.children.values():
            self._find_all_suggestions(child_node, suggestions)
```

## Visualization 📊
```
         root
        /  |  \
       a   b   c
      /    |    \
     p    a      a
    /     |       \
   p     t         t
  /       \         \
 l         h         s
 |         |
 e         e
(apple)   (bathe)   (cats)
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Very efficient for prefix-based operations
- Fast lookups (O(m) where m is the length of the key)
- Supports ordered traversal of keys
- Perfect for autocomplete and spell-checking features
- Can represent a large number of strings compactly when they share prefixes

### Disadvantages ❌
- Higher memory usage compared to other data structures (especially for sparse tries)
- Not as efficient for single string lookups compared to hash tables
- Deletion can be complex to implement correctly
- Not suitable for non-string data without modification

## Real-world Applications 🌐
- Autocomplete and predictive text
- Spell checkers
- IP routing (CIDR)
- T9 predictive text on phones
- Search engines and typeahead features
- Dictionary implementations
- Genome analysis in bioinformatics
- Natural language processing

## Further Reading 📚
- Books:
  - "Advanced Data Structures" by Peter Brass
  - "Algorithm Design Manual" by Steven Skiena
- Online Resources:
  - [Visualgo Trie Visualization](https://visualgo.net/en/trie)
  - [GeeksforGeeks Trie Tutorial](https://www.geeksforgeeks.org/trie-insert-and-search/)
  - [Princeton Algorithms Course - Tries](https://algs4.cs.princeton.edu/52trie/)
- Academic Papers:
  - "Efficient String Matching: An Aid to Bibliographic Search" by Aho and Corasick
  - "Fast Pattern Matching in Strings" by Knuth, Morris, and Pratt

---