
## Rules for Creating New Notes vs. Expanding Existing

### Create a NEW note when:
1. **The concept stands alone** - It can be understood and referenced independently (e.g., "Factory Pattern", "Async Context Managers")
2. **You'll link to it multiple times** - If 3+ other notes would reference it
3. **It exceeds 300 words** - The topic becomes substantial enough to warrant separation[3]
4. **It represents a distinct pattern or practice** - Each pattern/paradigm gets its own note
5. **It's a concrete example** - All code examples should be separate notes linked from concept notes

### Expand an EXISTING note when:
1. **Adding clarifying details** - Minor elaborations on existing points
2. **It's a sub-variant** - Small variations can be subsections (e.g., "Variations" section in pattern notes)
3. **The total would stay under 500 words** - Keep notes atomic but not fragmented
4. **It's tightly coupled** - The information only makes sense in the context of the parent topic
5. **It's a single example** - One illustrative code snippet can stay inline; multiple examples should be separate

## Topic Splitting Guidelines

### The Atomic Note Principle
Each note should cover **one coherent idea** that you can explain in a single sitting. Apply this hierarchy:[4]

**Level 1: Concept** → Gets its own note
- Example: "Dependency Injection"

**Level 2: Implementation** → Separate note or detailed section
- Example: "Dependency Injection in FastAPI"

**Level 3: Variation** → Subsection within implementation note
- Example: "Constructor Injection vs. Property Injection" (subsection)

**Level 4: Example** → Separate example note
- Example: "Example - DI with FastAPI Repository Pattern"

### The "Can I Explain This in One Breath?" Test
If you can't summarize the note's main point in one sentence, it probably covers multiple concepts and should be split.[1]

### Progressive Detail Approach
1. **Start broad**: Create a Map of Content (MOC) note like "Python Async Programming MOC"
2. **Link to concepts**: Individual notes for "Async/Await", "Event Loop", "Asyncio", "Async Context Managers"
3. **Add examples**: Separate example notes showing real implementations
4. **Cross-link patterns**: Connect to related architecture patterns like "CQRS with Async"

## Recommended Workflow

1. **Start with MOCs** - Create index pages for each major folder (Python Core MOC, Design Patterns MOC, etc.)
2. **Write concepts first** - Document the "what" and "why" before implementation details
3. **Add examples separately** - Keep code examples in dedicated notes to avoid bloat
4. **Link bidirectionally** - Use `[[]]` links liberally; Obsidian's graph view will show connections[2]
5. **Use tags for cross-cutting concerns** - Tags like `#async`, `#performance`, `#testing` help find notes across categories
6. **Review and refactor** - Periodically split notes that grow beyond 500 words or merge orphaned fragments

This structure balances organization with flexibility, allowing you to build a comprehensive knowledge base that grows naturally with your learning.[3][1]
