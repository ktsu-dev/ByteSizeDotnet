<!-- Generated by SpecStory -->

# Add section on value vs reference semantics (2025-06-08 12:22:49)

_**User**_

in the quick reference, add a section quite near the start explaining the diofference between value and reference semantics

---

_**Assistant**_



---

I'll help you add a section about value and reference semantics to the quick reference. First, let me explore the codebase to find the quick reference document.

---

<details>
            <summary>Listed current directory • **6** results</summary>
        
| Name |
|-------|
| 📁 `curriculum` |
| 📁 `.specstory` |
| 📄 `.cursorignore` |
| 📁 `.git` |
| 📁 `.cursor` |
| 📄 `.cursorindexingignore` |

</details>

---

Let me check if there are any markdown files or documentation in the root or curriculum directories:

---

<details>
            <summary>Searched codebase "quick reference" • **1** results</summary>
        
| File |
|------|
| `undefined` |

</details>

---

Great! I found the quick reference file. Let me read its contents to understand the current structure:

---

Read file: curriculum/quick-reference.md

---

Let me read the rest of the file to see the complete structure:

---

Read file: curriculum/quick-reference.md

---

Perfect! I can see the structure of the quick reference guide. I notice there's already a section called "🏗️ Type System Quick Decisions" that mentions value and reference semantics briefly in the context of when to use `struct` vs `class`. However, it would be valuable to add a dedicated section that explains these concepts more thoroughly near the start of the document.

I'll add a new section after the naming conventions and before the type system section, as this is foundational knowledge that will help readers understand the subsequent sections better.

---



---

Perfect! I've added a comprehensive section on "Value vs Reference Semantics" to the quick reference guide. The new section:

1. **Location**: Positioned strategically after the naming conventions and before the type system decisions, making it foundational knowledge
2. **Content**: Explains both value and reference semantics with:
   - Clear definitions and characteristics
   - Practical code examples showing the behavior differences
   - Memory and performance implications
   - Mutability considerations
   - Null safety aspects

3. **Structure**: 
   - Uses clear headings and bullet points for easy scanning
   - Includes runnable code examples with expected outputs
   - Highlights key differences with practical implications
   - Notes the special case of `string` being immutable despite being a reference type

This section will help readers understand the fundamental concepts before diving into the more specific guidance about when to use `struct` vs `class` in the following section. The practical examples demonstrate the real-world implications of these semantic differences, which is crucial for making good design decisions in C#.

---

_**User**_

the quick reference should explain stack vs heap allocation before reference and value semantics

---

_**Assistant**_



---



---



---



---

