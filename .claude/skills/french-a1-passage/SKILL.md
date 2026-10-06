# French A1 Daily Passage Generator

> **CI version** (runs on GitHub Actions). The working directory is the repository root; all paths below are relative to it.
> Bash is not available. Do NOT run git — the workflow commits and pushes.

## Usage

```
/french-a1-passage [YYYY-MM-DD]
```

- **Date**: use the date given in the prompt (the workflow passes today's date in Europe/London time).

## Purpose

Run once every morning to get a short DELF A1-level reading passage and a complete study set.

## DELF A1 Level Criteria

- **Vocabulary**: A1 level (greetings, numbers, family, daily objects, colors, simple everyday actions — never use B1+ vocabulary)
- **Grammar**: présent de l'indicatif (être, avoir, aller, faire, regular -er/-ir/-re verbs), articles (définis, indéfinis, partitifs), negation (ne...pas), basic questions (est-ce que, qu'est-ce que, quel/quelle), adjective agreement, basic prepositions (à, de, dans, sur, avec). Avoid subjunctive, conditional, and past tenses beyond passé composé for a single completed action if needed.
- **Length**: 100–150 words
- **Structure**: 3–4 short paragraphs
- **Sentence complexity**: Short, simple sentences. Subordinate clauses limited to *parce que*, *quand*, and *que* (after verbs like *penser*, *savoir*)

## Regional Variety: Standard French (Paris)

Use standard French throughout. Specific guidance:

- Standard vocabulary: *voiture*, *appartement*, *portable* (phone), *boulangerie*, *café*
- Use *tu* for informal address, *vous* for formal or plural
- Settings should reflect everyday French life: cafés, boulangeries, marchés, école, famille, vacances
- Cultural references from France (Paris, Lyon, the south of France, etc.)

## Theme Pool

Select randomly from the following categories (use a different theme each time):

1. **Daily life**: Shopping, cooking, home, neighbors
2. **Travel & tourism**: Café visits, asking for directions, visiting a city
3. **Work & school**: School life, simple work situations
4. **Health & sports**: Sports, doctor visits, healthy habits
5. **Culture & society**: Festivals, traditions, simple cultural events
6. **Media & entertainment**: Movies, music, simple social activities
7. **Relationships**: Family, friends, meeting someone new

## History Management (Avoiding Repetition)

Use a `history.json` file in the project directory to track past outputs.

### Reading on Execution

Before generating a passage, check whether `history.json` exists.

- **If the file exists**: Read it and review past theme categories, subtopics, titles, and vocabulary.
- **If the file does not exist**: Treat it as an empty state and proceed as a first run.

### Theme Selection Logic

1. Check the categories of the most recent 7 entries in `history.json`
2. If all 7 categories appear in the last 7 entries, reset the rotation
3. Prioritize categories that have not appeared yet
4. Even within the same category, avoid repeating subtopics

### Vocabulary Overlap Check

A `vocab-used.json` at the project root maintains a flat sorted array of every vocabulary word ever used.

**On execution:**
1. Read `vocab-used.json` (treat as `[]` if not exists).
2. When selecting the 6–10 key vocabulary items for Part 3, only pick words not in `vocab-used.json`.
3. Allow at most **1 repeat** if unavoidable.

**After generating:**
4. Append new vocabulary words, re-sort alphabetically (case-insensitive), deduplicate, and save.
5. Save `vocab-used.json` (the workflow commits it).

### history.json Format

```json
{
  "entries": [
    {
      "date": "2026-06-11",
      "category": "Daily life",
      "subtopic": "At the bakery",
      "title": "La boulangerie du quartier",
      "vocab": ["boulangerie", "croissant", "pain", "acheter", "bonjour"]
    }
  ]
}
```

File path: `history.json` (repository root)

## Output Format

Write everything to `passages/YYYY-MM-DD/` (relative to the repository root), outputting two files:
- `passages/YYYY-MM-DD/YYYY-MM-DD.md` — Markdown format
- `passages/YYYY-MM-DD/YYYY-MM-DD.html` — HTML format (styled, self-contained)

Do NOT output the passage content to the CLI — only write to the files.

### Part 1: Passage

```
📖 Passage du jour — [Theme category]
Titre : [Title]

[Passage body (French only)]
```

### Part 2: English Translation

A natural English translation of the full passage body.

In the HTML output, Part 1 and Part 2 must be rendered **side by side** using a two-column flexbox layout (`.passage-columns`). French on the left, English on the right. On mobile (max-width: 700px), they stack vertically. Use `<div class="passage-col-label">` labels instead of headings.

Do **NOT** include a text-to-speech (TTS) button or any related JavaScript.

```
🇬🇧 English Translation

[English translation paragraph by paragraph]
```

### Part 3: Vocabulary List (6–10 words)

```
📝 Vocabulaire clé

| Mot | IPA | Définition (FR) | English | Japanese | Exemple du texte |
|-----|-----|-----------------|---------|----------|-------------------|
| ... | ... | ...             | ...     | ...      | ...               |
```

Include both English and Japanese in the vocabulary table (English for the "English" column, Japanese for the "Japanese" column).

**IPA column**: Give the standard French IPA transcription in slashes, e.g. `/sak a do/` for *sac à dos*, `/ʒɑ̃til/` for *gentille*. Use liaison-aware transcription when the word is naturally said with a following article/preposition in the example sentence (otherwise transcribe the word in isolation). This column is required for every vocabulary row — do not leave it blank.

> **問題と解答は作らない。** 出力は Part 1〜3（本文・訳・語彙）だけにする。Markdown にも HTML にも、読解問題・解答・解説のセクションを入れない。

## Execution Steps

1. **Determine target date**: Use the date given in the prompt.
2. **Check for existing output**: If `passages/YYYY-MM-DD/YYYY-MM-DD.md` already exists, stop and output:
   `⚠️ passages/YYYY-MM-DD/ already exists. To regenerate, delete the folder first.`
3. Read `history.json` (treat as empty if not exists).
4. Select a non-overlapping theme category and subtopic.
5. Generate an A1-level French passage.
6. Write a natural Japanese translation.
7. Extract 6–10 key vocabulary words. Cross-reference `vocab-used.json`. Replace overlaps until at most 1 remains.
8. Append to `history.json` and save.
9. Create `passages/YYYY-MM-DD/` directory.
10. Write `.md` and `.html` files.
11. Update `index.html` — prepend a new `<li>` at the top of `<ul class="list">`:
    ```html
    <li data-date="YYYY-MM-DD" data-category="CATEGORY">
      <a href="passages/YYYY-MM-DD/YYYY-MM-DD.html">
        <span class="title-text">TITLE</span>
        <span class="tag">CATEGORY</span>
        <span class="date">YYYY-MM-DD</span>
      </a>
    </li>
    ```
    Use `&amp;` for `&` in `data-category` and `<span class="tag">`.
12. Copy HTML to `today.html` at the repo root.
13. Do NOT run git. The workflow commits and pushes.
14. Output confirmation:
    `✅ Saved to passages/YYYY-MM-DD/ — [Title]`

## HTML Styling

Use the same visual style as the DELE B1 passages (Georgia serif, warm tan border `#d4a96a`, max-width 1100px). Left column uses `.passage-box` (gold border), right column uses `.translation-box` (blue border, background `#f0f4ff`).

### IPA hover tooltips in the passage text

In the HTML output only (not the Markdown), wrap each vocabulary word where it appears in the **French passage column** (`.passage-box`) in a `<span class="ipa-word" title="/ipa/">word</span>`, using the same IPA transcription as the vocabulary table. This lets the reader hover over the word directly in context to see its pronunciation via the browser's native tooltip — no JavaScript needed. Wrap only the first occurrence of each vocabulary word per passage (don't wrap the same word twice), and match the word's actual inflected form as it appears in the text (e.g. wrap `gentille`, not `gentil`, if that's what the sentence uses).

Add this CSS to the `<style>` block:

```css
.ipa-word {
  border-bottom: 1px dotted #a08050;
  cursor: help;
}
```

Do not wrap words in the English translation column or anywhere outside `.passage-box`.

## Quality Checks

- Passage is 100–150 words
- No vocabulary or grammar above A1
- Theme differs from recent 7 entries (checked via `history.json`)
- `history.json` and `vocab-used.json` are updated and saved
- Every vocabulary row has an IPA transcription, and each vocab word is wrapped with a hover tooltip (`.ipa-word`) at its first occurrence in the HTML passage text
