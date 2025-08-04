---
title: 🔪 Slicing Strings in Python
author: hcoco1
date: 2025-08-04 16:10:00 +0800
categories: [Programming, Python]
tags: [python, slicing, joining]
render_with_liquid: false
pin: false
description: Slicing a string is just like taking a slice out of a homemade apple pie. When it comes to strings, slices can be as small or as big as you want. If you want to put two or more of those slices back together, you join them together end-to-end—in Python speak, you “concatenate” them—to make one bigger, single string.

---

**Slicing** means extracting parts of a string using indexes.

### 🧠 Syntax:

```python
string[start:stop:step]
```

| Part    | Meaning                                  |
| ------- | ---------------------------------------- |
| `start` | Index to begin the slice (inclusive)     |
| `stop`  | Index to end the slice (exclusive)       |
| `step`  | How many characters to skip (default: 1) |

------

### 🧪 Example:

```python
text = "Python"

print(text[0:2])    # 'Py' → from index 0 up to (but not including) 2
print(text[2:])     # 'thon' → from index 2 to the end
print(text[:3])     # 'Pyt' → from start to index 3
print(text[::2])    # 'Pto' → every 2nd character
print(text[::-1])   # 'nohtyP' → reversed string
```

------

## 🔗 Joining Strings in Python

**Joining** means combining a list (or any iterable) of strings into one string.

### 🧠 Syntax:

```python
'separator'.join(iterable)
```

- `'separator'` is what you want between each piece (like a space, comma, dash, etc.)
- `iterable` is usually a list of strings

------

### 🧪 Example:

```python
words = ["Python", "is", "fun"]

sentence = " ".join(words)
print(sentence)  # 'Python is fun'

csv_line = ",".join(words)
print(csv_line)  # 'Python,is,fun'
```

------

### 🔁 Slicing + Joining Example

```python
word = "developer"

# Slice out every other letter
sliced = word[::2]  # 'deelpr'

# Join with dashes
joined = "-".join(sliced)
print(joined)  # 'd-e-e-l-p-r'
```

------

## 🧠 Summary Table

| Operation    | Syntax Example         | Result                 |
| ------------ | ---------------------- | ---------------------- |
| Slice        | `word[1:4]`            | characters from 1 to 3 |
| Reverse      | `word[::-1]`           | reverses the string    |
| Join (space) | `" ".join(["a", "b"])` | `'a b'`                |
| Join (dash)  | `"-".join(["a", "b"])` | `'a-b'`                |

------

