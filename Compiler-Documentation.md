
* * * * *
CT4PWD Block Compiler --- README
==============================

**Index**
---------

1.  [Overview](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#overview)

2.  [Libraries Used](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#libraries-used)

3.  [Block Types (Lexer Output)](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#block-types-lexer-output)

4.  [Lexical Analysis --- `lex.py`](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#lexical-analysis-lexpy)

5.  [Parsing --- `parse.py`](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#parsing-parsepy)

6.  [Evaluation --- `eval.py`](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#evaluation-evalpy)

7.  [Maze Navigation Module](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#maze-navigation-module)

8.  [`main.py` --- Module Routing](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#mainpy--module-routing)

9.  [System Diagrams](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#system-diagrams)

10. [Extending the Compiler](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#extending-the-compiler)

11. [Examples (End-to-End)](https://chatgpt.com/c/6926e03b-bb04-8321-9aea-0fdee994e83d#examples-end-to-end)

* * * * *

1\. Overview
============

The **CT4PWD Block Compiler** is a **vision-based block programming system**.\
Users arrange **QR-based physical blocks** (`if`, `loop`, `colour`, `direction`, etc.).\
The system captures them using a camera and compiles them into executable logic.

### **System Purpose**

To teach programming concepts visually to students on the autism spectrum.

### **Target Learners**

Students who benefit from **concrete, visual programming concepts**.

### **Supported Activities**

-   Conditional logic

-   Pattern creation (colour sequences)

-   Loops

-   Maze navigation

### **High-Level Architecture**

```
Image
  ↓
lex.py   (QR detection → tokens)
  ↓
parse.py (build logical structure)
  ↓
eval.py  (run logic)
  ↓
main.py  (choose module & output)

```

* * * * *

2\. Libraries Used
==================

| Library | Purpose |
| --- | --- |
| **pyzbar** | QR code detection |
| **OpenCV (cv2)** | Image processing & cropping |
| **pytesseract** | Reading loop count text |
| **re** | Regex-based text normalization |

```
from pyzbar.pyzbar import decode
import cv2
import pytesseract
import re

```

Everything else is implemented using **pure Python**.

* * * * *

3\. Block Types (Lexer Output)
==============================

These are the `type` fields emitted by `lex.py`:

| Block Type | Meaning | Example |
| --- | --- | --- |
| `control` | if / elseif / else | `"if"` |
| `condition` | raining, sunny, red, etc. | `"raining"` |
| `action` | stop, go, umbrella | `"stop"` |
| `colour` | red, green, blue | `"blue"` |
| `loop` | loop count block | `loop x3` |
| `maze` | maze mode trigger | `"maze"` |
| `direction` | up/down/left/right | `"right"` |
| `label` | fallback text | any unmatched |

* * * * *

4\. Lexical Analysis --- `lex.py`
===============================

`lex.py` converts raw QR codes into structured token blocks.

### **4.1 QR Detection & Sorting**

```
qr_codes = decode(image)
qr_codes.sort(key=lambda q: q.rect.left)

```

Ensures **left-to-right** program order.

### **4.2 Text Normalization**

Examples:

-   `"Else If"` → `"elseif"`

-   `" GREEN!! "` → `"green"`

### **4.3 Block Classification**

Uses lookup sets:

-   `CONTROL_SET`

-   `CONDITION_SET`

-   `ACTION_SET`

-   `COLOUR_WORDS`

-   `DIRECTION_SET`

### **4.4 Loop Number Detection**

A crop to the right of the QR block is passed to Tesseract:

```
loop_count = int(pytesseract.image_to_string(crop))

```

### **4.5 Lexer Output Format**

```
{
  "blocks": [...],
  "loop_count": 3,
  "anchor_x": 528
}

```

* * * * *

5\. Parsing --- `parse.py`
========================

Creates structured representation from block tokens.

### **Parser Output Example**

```
{
  "colours": [],
  "loop_count": 1,
  "conditions": [],
  "sequence": [...]
}

```

### **5.1 Condition Parsing**

Handles `if`, `elseif`, `else` sequences.

Example:

```
if raining stop

```

Parsed as:

```
{ "if": "raining", "action": "stop" }

```

### **5.2 Colour Pattern Parsing**

Append colours directly:

```
colours.append(v)
sequence.append(v)

```

### **5.3 Loop Expansion**

If loop block exists:

```
["red", "green"] × 3
→ ["red","green","red","green","red","green"]

```

### **5.4 Maze Mode Parsing**

Maze and direction blocks passed raw:

```
sequence.append(b)

```

* * * * *

6\. Evaluation --- `eval.py`
==========================

Chooses logic based on parsed structure.

* * * * *

**6.1 Condition Logic Module**
------------------------------

### Steps:

1.  Validate logic (e.g., `red → stop` required)

2.  Validate order (`if → elseif → else`)

3.  Construct flattened output:

```
if red stop elseif green go else umbrella

```

* * * * *

**6.2 Pattern / Loop Module**
-----------------------------

For sequences without condition blocks:

Example:

```
Colours = ["red", "blue"]
Loop = 3

```

Output:

```
red blue red blue red blue

```

* * * * *

7\. Maze Navigation Module
==========================

Triggered by block:

```
{ "type": "maze", "value": "maze" }

```

### **7.1 Maze Input**

User enters maze with `S` (start), `E` (end), `0` (path), `1` (wall):

```
S 0 1
0 0 0
1 0 E

```

### **7.2 Movement Simulation**

Commands like:

```
right, down, down, right

```

Tracks:

-   `trail`

-   `directions_traversed`

-   `point_of_failure`

-   `direction_of_collision`

### **7.3 Failure Example**

```
{
  "result": "❌ Hit a wall at step 3",
  "trail": [[0,0],[0,1],[1,1]],
  "directions_traversed": ["right","down"],
  "point_of_failure": [1,2],
  "direction_of_collision": "right"
}

```

* * * * *

8\. `main.py` --- Module Routing
==============================

### Module Detection Rules

| Condition | Module |
| --- | --- |
| starts with `maze` | Maze module |
| contains condition blocks | Condition evaluator |
| else | Pattern / loop module |

* * * * *

9\. System Diagrams
===================

### **9.1 Compiler Pipeline**

```
Image
  ↓
lex.py  →  blocks, loop count
  ↓
parse.py → structured sequence
  ↓
eval.py → output execution
  ↓
main.py → routing + printing

```

### **9.2 Maze Execution Flow**

```
maze block detected
        ↓
main.py asks user for maze
        ↓
simulate_maze()
        ↓
success/failure + trail
        ↓
main.py prints result

```

* * * * *

10\. Extending the Compiler
===========================

Steps to add new features:

### **Step 1 --- Lexing**

Add new keywords to sets:

```
NEW_SET = {...}

```

### **Step 2 --- Parsing**

Add new handler or transformation logic.

### **Step 3 --- Evaluation**

Add a new evaluator function inside `eval.py`.

### **Step 4 --- Routing**

Modify `main.py` to detect and call the new module.

The compiler is designed to be **easily extensible**.

* * * * *

11\. Examples (End-to-End)
==========================

**11.1 Condition Example**
--------------------------

Blocks:

```
if → red → stop → else → umbrella

```

Output:

```
if red stop else umbrella

```

**11.2 Loop Example**
---------------------

Blocks:

```
loop 3 red blue

```

Output:

```
red blue red blue red blue

```

**11.3 Maze Example**
---------------------

Blocks:

```
maze right down right right down

```

Maze:

```
S 0 0 1
1 0 0 0
1 1 0 E

```

Output:

```
{
  "result": "Level cleared",
  "trail": [[0,0],[0,1],[1,1],[1,2],[1,3],[2,3]],
  "directions_traversed": ["right","down","right"],
  "point_of_failure": null
}

```

**11.4 Block Pattern Examples**
-------------------------------

-   Maze module

-   Loop module

-   If-Else module

* * * * *

### **Developed by:**

**Aditya Gupta (2022A7PS0090G)**

