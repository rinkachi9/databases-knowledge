# Guideline

This document describes how to approach the Databases Knowledge repository: what belongs in it, how notes are organized, how concepts must be explained and what quality bar every note has to meet. It applies to people and to AI agents alike.

## Table of Contents

1. [Purpose and audience](#1-purpose-and-audience)
2. [Core principles](#2-core-principles)
3. [Repository structure and naming](#3-repository-structure-and-naming)
4. [Anatomy of a note](#4-anatomy-of-a-note)
5. [How to explain a concept](#5-how-to-explain-a-concept)
6. [Examples, code and diagrams](#6-examples-code-and-diagrams)
7. [Writing style](#7-writing-style)
8. [Sources, references and verification](#8-sources-references-and-verification)
9. [Rules specific to this repository](#9-rules-specific-to-this-repository)
10. [Quality checklist](#10-quality-checklist)
11. [Maintenance and versioning](#11-maintenance-and-versioning)
12. [Working with AI agents](#12-working-with-ai-agents)

## 1. Purpose and audience

The repository is a long term reference for databases: data models, database design, query languages, query processing, transactions, distributed data management, integration, administration and security, and the concrete database engines that implement them. Most of it is *technology agnostic*: a note explains a concept that holds across engines, and a concrete engine appears only as an example. A separate area holds the engine specific material (PostgreSQL, MongoDB, Redis and others). It is written for a backend engineer, data engineer, database administrator or software architect who wants expert level understanding. It has two jobs:

- **Teach.** Someone with general technical background but no experience in the specific topic should be able to read a note and come away with a correct mental model.
- **Recall.** Someone who already knows the topic should be able to find the exact detail they need quickly, without wading through introductions.

To serve both, every note is layered. It starts with a short plain language summary, then builds up to precise detail. A reader can stop at any layer and still have learned something true.

The primary audience is the future version of the author. Write as if explaining to a smart colleague who was not in the room when you learned the topic. Do not assume they remember the context, the tools you used or the problem you were solving at the time.

## 2. Core principles

**Depth over breadth.** A note that explains one idea thoroughly is worth more than five that name-drop ten. Cover a topic completely before moving on: what it is, why it exists, how it works, when to use it, when not to use it, and how it fails.

**Explain the why, not only the what.** Facts without reasons are trivia. For every rule, tool or technique, state the problem it solves. A reader who understands the problem can reconstruct the solution and can tell when it does not apply.

*Example.* "Use parameterized queries" is a rule. "Parameterized queries keep data and code separate, so input can never change the structure of the statement" is understanding.

**Precision.** Use the correct technical term, define it on first use and keep using it consistently. Do not swap synonyms for variety, because a reader may assume two words mean two things. Quantify when you can: "about 10 ms on a warm cache" beats "fast".

**Show, then generalize.** Introduce an abstraction with a concrete case first, or immediately after. People learn patterns from instances.

**Be honest about uncertainty.** If something is unverified, simplified or version dependent, say so in the note. A clearly marked gap is better than a confident error.

**One idea, one place.** Each concept has a single home note. Other notes link to it. Duplicated explanations drift apart and become contradictory.

**Make it verifiable.** Claims link to sources. Examples can be run or reproduced. Numbers say where they came from.

**Show it as well as say it.** Notes are meant to be attractive to read and quick to grasp. A structure, a process, a model or a set of numbers is drawn, not only described. Every note has at least one diagram, schema, model or chart that carries real information (see [section 6.2](#62-diagrams-models-and-visuals)).

## 3. Repository structure and naming

### 3.1 Layout

The structure follows the *Data management systems* branch of the [ACM Computing Classification System (CCS) 2012](https://dl.acm.org/ccs), the classification that the ACM uses to index computer science literature, with the order of the parts of Silberschatz, Korth and Sudarshan, *Database System Concepts* (7th edition, 2019) where the CCS gives no order. Each CCS concept of that branch is an *area* (a top level directory such as `transactions-and-concurrency/`). Inside an area, a *subarea* groups notes that clearly belong together (such as `database-design-and-models/nosql/`). Four areas are not CCS concepts of that branch: `foundations/` holds what every other area assumes (what a database is, SQL versus NoSQL, choosing a database), `database-security/` maps the CCS concept *Database and storage security* from the security branch, `engines/` holds the material that is specific to one database engine, and `case-studies/` describes real systems. The list of areas is in the [README](README.md#areas). Checked against the CCS 2012 tree, 2026-11.

```text
ToC.md                  # full tree of every file in the repository
_sources/               # original exports that notes are migrated from (not committed)
librarian-os.json       # configuration for Librarian OS: project name, entry points, rules (see 3.5)
_assets/
    <area>/             # images, diagram sources, media and other files used by notes
<area>/
    README.md           # entry point: what the area covers, reading order, links to notes
    README.md.json      # its sidecar
    <topic>.md          # a single note
    <topic>.md.json     # its sidecar: node header and relations (see 3.5)
    <subarea>/          # optional, groups related notes
        README.md
        <topic>.md
        <topic>/        # optional, when a topic has its own scripts or files
            README.md   # the note itself
            scripts/    # scripts that belong to this topic only
```

Guidance:

- **Prefer flat over deep.** Area and subarea are the normal depth. Add a subarea when an area has more than about twelve files or when a group of notes clearly belongs together.
- **One note, one main idea.** If a note needs a table of contents longer than about ten entries, it is probably several notes.
- **Concepts are technology agnostic, engines have their own area.** A note in a concept area explains the idea so that it holds for any engine and uses engines only as labeled examples. What is true for one engine only (configuration, internals, tools, version history, license) belongs to `engines/<engine>/`. For example, replication in general lives in `distributed-databases/replication.md`, and streaming versus logical replication in PostgreSQL lives in `engines/postgresql/`.
- **One engine, one home.** Every engine that has more than a short description gets a subarea `engines/<engine>/` with a `README.md` and an overview note. An engine that is only named as an example stays in the table of representative engines of its data model note until it gets its own subarea.
- **Every area and subarea has a `README.md`.** It is the entry point and lists its notes in a recommended reading order with a one line description each. Areas and topics that are planned but not written yet are listed under "Planned topics".
- **Media goes to `_assets/<area>/`, scripts stay with their topic.** Images, diagram sources, downloadable files and media go to `_assets/<area>/`. A script lives next to the material it belongs to: when a topic gets scripts, the note becomes a directory `<topic>/` with `README.md` (the note) and a `scripts/` directory. There is no global `scripts/` directory.
- **`librarian-os.json` lists the entry points.** The area directories are the entry points of the knowledge base. Adding an area means adding it to the `entryPoints` list there, in the same change. Content outside the entry points (this guideline, the README, `_sources/`, `_assets/`) is not shown as notes by Librarian OS.
- **Keep `ToC.md` complete.** See [section 3.4](#34-table-of-contents).

*Analogy.* Think of a library. Areas are sections, area READMEs are the catalog cards for a section, and notes are the books. A book that tries to cover an entire section belongs on several shelves.

### 3.1.1 Placement of a note

Classify a note by the main question it answers and place it in the area whose CCS concept covers that question:

| Area directory | CCS 2012 concept | Primary home for |
| --- | --- | --- |
| `foundations/` | (none) | What databases and DBMSs are, SQL versus NoSQL, choosing a database, the landscape of engines. |
| `database-design-and-models/` | Database design and models | The relational model, keys, relationships, constraints, normalization, denormalization, entity relationship modeling, and the NoSQL data models (key-value, document, wide-column, graph, time series, vector). |
| `storage-and-indexing/` | Data structures (data access methods, data layout) | Pages and files, row and column storage, B-trees, LSM trees, hash and inverted indexes, buffer management, table partitioning. |
| `query-languages/` | Query languages | SQL (statement classes, joins, procedural extensions such as stored procedures and triggers) and the query languages of non relational engines. |
| `query-processing-and-optimization/` | DBMS engines: query processing, query optimization | Execution plans, the optimizer, statistics, join algorithms and SQL tuning. |
| `transactions-and-concurrency/` | DBMS engines: transaction processing | Transactions, ACID, isolation levels, locking, MVCC, deadlocks, logging and recovery. |
| `distributed-databases/` | DBMS engines: parallel and distributed DBMSs | Cluster layout, consistent hashing, replication, sharding, scaling strategies, consistency models, consensus. |
| `information-integration/` | Information integration | Federation, data virtualization, ETL and ELT, change data capture. |
| `database-administration/` | Database administration | Backup and recovery, monitoring, performance evaluation, schema migrations, high availability operations. |
| `database-middleware/` | Middleware for databases | Connection pooling, database proxies, object relational mapping. |
| `database-security/` | Security and privacy: database and storage security | Access control, injection, encryption, auditing. |
| `engines/` | (none) | Everything specific to one engine. |
| `case-studies/` | (none) | Descriptions of real systems that apply the concepts above. |

For a topic that fits several areas, choose the home by its central mechanism and learning goal, and link to it from the other places. Do not copy the explanation into several directories. When a note moves, keep its sidecar `node.id` so that relations stay valid, and update READMEs, relative links and `ToC.md`.

### 3.2 Naming

| Item | Convention | Example |
| --- | --- | --- |
| Directories | lowercase, words separated by hyphens | `security-and-identity/` |
| Note files | lowercase, hyphens, descriptive noun phrase, `.md` | `consistent-hashing.md` |
| Images and media | note name, then a short description, stored in `_assets/<area>/` | `_assets/distributed-databases/consistent-hashing-ring-ownership.png` |
| Diagram sources | same base name as the rendered image, source format as extension | `_assets/distributed-databases/consistent-hashing-ring-ownership.drawio` |
| Scripts | lowercase, hyphens, verb phrase, stored in the `scripts/` directory of their topic | `engines/postgresql/vacuum/scripts/report-transaction-id-age.sql` |
| Entry points | always `README.md` | `<area>/<subarea>/README.md` |
| Table of contents | always `ToC.md` in the repository root | `ToC.md` |
| Sidecar | the full name of the Markdown file plus `.json` | `sharding.md.json`, `README.md.json` |

Names describe the content, not its status or origin. Avoid `notes.md`, `misc.md`, `new-draft-2.md` or dates in file names. Do not use spaces or uppercase letters, so that links and shell commands behave the same on every system.

### 3.3 Links

- Use relative links between notes, for example `[B-tree indexes](../storage-and-indexing/b-tree-indexes.md)`.
- Link to a heading when you point at a specific part: `[isolation levels](transactions-and-acid.md#isolation-levels)`.
- Link text describes the target ("the section on isolation levels"), never "click here".
- When a prerequisite is needed, link it in the note's front section instead of re-explaining it.

### 3.4 Table of contents

`ToC.md` in the repository root is the complete map of the repository. It is an indented tree with one entry for every file: root documents, area and subarea `README.md` files, notes, assets and scripts. Each entry is a relative link followed by a short description. A sidecar (`.md.json`) is not listed on its own: the entry of its Markdown file covers it. 

- **Every change is reflected in `ToC.md`.** Adding, moving, renaming or deleting a note, an asset or a script updates `ToC.md` in the same change. Run `npm run toc` in Librarian OS with this repository as the argument to regenerate it: entries come from the files, the note titles and summaries, and the "Planned topics" lists of the READMEs, so a change of title or summary updates the description there.
- **Planned areas are marked.** An area or subarea that has only a `README.md` is marked `(planned)`. Planned topics are listed without a link.
- **Verify before finishing.** Compare `ToC.md` with the output of `git ls-files --cached --others --exclude-standard` and check that every link resolves.
- **`ToC.md` is not a replacement for area READMEs.** The area README keeps the recommended reading order and the context. `ToC.md` keeps the complete list.

### 3.5 Sidecar metadata

Every Markdown file inside an entry point (a directory listed in `librarian-os.json`) has a *sidecar*: a JSON file with the same name plus `.json`, stored next to it (`sharding.md` and `sharding.md.json`, `README.md` and `README.md.json`). The Markdown file stays the single source of truth for the text. The sidecar holds only what Markdown cannot express well: the identity of the note as a node in a graph, typed relations to other notes, structured sources, a list of key terms and practice questions. The Librarian OS reader (the repository is configured for it by [`librarian-os.json`](librarian-os.json)) uses the sidecars to draw mind maps, show backlinks, search and build practice decks. AI tools use them to generate questions without parsing prose.

*Analogy.* The note is a book on the shelf, and the sidecar is its catalog card. The card does not repeat the book. It says what the book is, what it needs to be read after and what it connects to. The limit of the analogy is that a catalog card is written once, while a sidecar must be kept in step with the note when the note changes.

**Example** (`distributed-databases/sharding.md.json`, shortened):

```json
{
  "node": {
    "id": "distributed-databases.sharding",
    "kind": "note",
    "status": "draft",
    "level": "intermediate",
    "tags": ["distributed-databases", "partitioning", "scaling"],
    "aliases": ["horizontal partitioning", "partitioning"]
  },
  "links": [
    { "to": "distributed-databases.consistent-hashing", "rel": "prerequisite" },
    { "to": "distributed-databases.replication", "rel": "related", "note": "Each shard is usually replicated" }
  ],
  "sources": [
    {
      "key": "kleppmann-ddia",
      "type": "book",
      "authors": ["Kleppmann, M."],
      "year": 2017,
      "publisher": "O'Reilly Media",
      "title": "Designing Data-Intensive Applications",
      "url": "https://dataintensive.net/",
      "verified": false,
      "checked": "2026-11"
    }
  ],
  "concepts": [
    { "term": "Shard key", "anchor": "#shard-key", "definition": "The attribute whose value decides which shard stores a row or document." }
  ],
  "map": { "group": "scaling" }
}
```

**Fields**

| Field | Required | Meaning |
| --- | --- | --- |
| `$schema` | no | Path or URL of the JSON Schema of Librarian OS (`schema/note.schema.json`), so that editors can validate and autocomplete. |
| `node.id` | yes | Stable identifier, unique in the repository: lowercase segments of letters, digits and hyphens joined by dots. Links use ids, not paths, so moving a file does not break them. |
| `node.kind` | yes | `note` (a topic note), or `index` (an area or subarea README). Documents at the repository root, such as this one, are outside the entry points and have no sidecar. |
| `node.status` | no | `draft` or `reviewed`. Default is `draft`. |
| `node.level` | no | `intro`, `intermediate` or `advanced`. |
| `node.tags`, `node.aliases` | no | Free tags for filtering, and alternative names used by search. |
| `links` | no | Typed relations from this note to others (table below). |
| `sources` | no | Structured bibliography: `key`, `type`, `authors`, `year`, `title`, `url`, `verified`, `checked`. The human readable list stays in "Further reading". |
| `concepts` | no | Key terms with an `anchor` to the heading that explains them and a one sentence `definition`. |
| `questions` | no | Practice questions with `id`, `type` (`recall`, `explain`, `apply`, `compare`), `q`, `a`, optional `hint`, `anchor`, `origin` (`author` or `ai`) and `reviewed`. |
| `map` | no | Hints for mind maps, such as `group`. |

**Relation types**

| `rel` | Meaning | Shown on the target as |
| --- | --- | --- |
| `prerequisite` | Read the target first. | Required by |
| `extends` | Builds on or deepens the target. | Extended by |
| `related` | Neighbors in the same topic, no order. | Related |
| `contrasts` | An alternative or opposing view. | Contrasts |
| `example-of` | A concrete case of the target. | Examples |
| `applies` | Applies the target idea in a specific setting. | Applied in |
| `part-of` | A part of a larger topic note. | Parts |

**Rules**

- **Follow `INSTRUCTIONS.md` of Librarian OS.** Whenever you create or change a note, its sidecar, a README or `librarian-os.json`, follow `INSTRUCTIONS.md` in the Librarian OS repository for everything that concerns compatibility with the reader: the configuration standard, the sidecar standard, entry points and the steps to adapt a repository. If this guideline and `INSTRUCTIONS.md` disagree on a Librarian OS matter, `INSTRUCTIONS.md` wins for the technical format, and this guideline wins for content and style.
- **Derived, not stored.** The title comes from the `#` heading and the area and subarea come from the path. The hierarchy edges of the graph come from the directory tree. Storing them again would let them drift.
- **Relations point forward only.** Backlinks are computed. Do not write the inverse relation in the target.
- **Link only to existing ids.** A planned topic without a sidecar cannot be a target. `npm run check` fails on a broken id.
- **No duplicated prose.** A definition in `concepts` is one sentence. The explanation belongs in the note.
- **Practice questions.** The "Practice" section of the note stays the authoritative place for questions the author writes. The `questions` list is the deck for the reader and for generated questions, so do not copy the same question into both. Questions with `origin: "ai"` start with `reviewed: false` until the author accepts them.
- **Format.** UTF-8, 2 space indent, one trailing newline, no comments, English only. The hyphen rule of [section 7.2](#72-punctuation-hyphens-only) applies to every string.
- **Validation.** `npm run check` in Librarian OS (with this repository as the argument) validates every sidecar against the schema and checks ids, relation targets, titles, links and `ToC.md`. `npm run scaffold` creates missing sidecars.

## 4. Anatomy of a note

Every note follows the same skeleton, so a reader always knows where to look. Sections that do not apply may be dropped, but their order stays. Notes have no front matter: a note starts with its title.

```markdown
# <Title>

## Summary
Two to four sentences in plain language. What is this, and why should the reader care?

## Prerequisites
Links to notes the reader should know first. State the assumed knowledge.

## Core concepts
Precise definitions, the mechanism, the vocabulary. Build from simple to complex.
Include at least one visual: a diagram, a model or a chart (section 6.2).

## Worked example
A concrete, reproducible example that walks through the idea end to end.

## Trade offs and when to use it
Benefits, costs, alternatives, and the situations where this is the wrong choice.

## Common mistakes
Frequent misunderstandings and failure modes, with the fix for each.

## Practice
Questions or small exercises, with answers or hints kept at the bottom.

## Further reading
Annotated links to primary sources, one sentence on why each is worth reading.
```

Notes on the skeleton:

- **No metadata block.** The directory path tells the area and subarea, the title is the first line, and open gaps are marked with `TODO:` lines.
- **Visuals sit where they help.** Put the diagram right after the paragraph that introduces the idea, with one sentence saying what to look at. Charts of the numbers in a worked example go directly after its table.
- **Summary first.** A reader who only reads the summary must not come away with a wrong idea. Avoid teasers such as "we will see how this works".
- **Motivation before mechanism.** People remember mechanisms better when they know what they were invented to fix.
- **Practice is part of the note.** Retrieval practice is what turns reading into knowledge. Even three good questions are valuable.

### 4.1 Migrated notes

Notes that were migrated from an older source (an export in `_sources/`) keep their original content and are brought up to the standard over time.

- Sections of the skeleton that the original did not have contain a single line that starts with `TODO:`. Replace it with real content, or delete the section if it does not apply.
- Search for open gaps with `grep -rn "^TODO:" --include="*.md" .`. A note without `TODO:` lines has met the skeleton, but still has to be checked against sources (see [section 8](#8-sources-references-and-verification)).
- Content that was migrated has not been verified against sources.
- A migrated note does not mention where it came from. Do not add "Source material" sections or sentences such as "from the export" or "the original notes say". The only provenance a note carries is its sources list (see [section 8](#8-sources-references-and-verification)).
- The dash and emoji rules of [section 7](#7-writing-style) apply to migrated content as well. Dashes were replaced by hyphens and emoji by words during migration.

## 5. How to explain a concept

Every concept that gets its own section or note is explained with the same five moves. This is the core of the standard.

1. **Definition.** One or two precise sentences. Say what the thing is and what distinguishes it from things that look similar.
2. **Intuition through analogy.** A comparison to something familiar. State where the analogy holds and where it breaks.
3. **Mechanism.** How it actually works, step by step, with the correct terminology.
4. **Concrete example.** A small, realistic case that uses real values and can be reproduced.
5. **Boundaries.** When to use it, when not to, what commonly goes wrong.

### 5.1 Writing good analogies

An analogy is a bridge to a known idea, not a decoration.

- **Map the parts explicitly.** "The index is the book's back-of-book index, the page number is the row location."
- **Choose a familiar source.** Everyday objects and systems work best: libraries, kitchens, traffic, post offices, buildings.
- **Say where it breaks.** Every analogy misleads somewhere. Naming the limit prevents the reader from carrying a false belief forward.
- **Use one analogy per idea.** Stacking three metaphors confuses more than it clarifies.
- **Never let the analogy replace the mechanism.** It opens the door, and the precise explanation must still follow.

### 5.2 Writing good examples

- **Minimal but real.** Small enough to hold in your head, realistic enough to feel like something you would meet in practice.
- **Complete.** The reader can run it or reproduce it without guessing at missing pieces.
- **Show expected output.** Say what happens, including the failure case, so the reader can check their understanding.
- **Progress in difficulty.** Start with the simplest case, then add one complication at a time.
- **Contrast.** Show the wrong way next to the right way, and explain why the first fails.

### 5.3 Depth checklist for a concept

Before a concept is considered explained, you should be able to answer:

- What problem does it solve, and what happened before it existed?
- What are its precise definition and its key properties?
- How does it work internally, at least at one level below the surface?
- What are the alternatives, and how do they differ?
- What does it cost (time, complexity, money, risk)?
- How does it fail, and how would you notice?
- What is the most common misconception about it?

### 5.4 A full example of the standard

The following shows the five moves applied to one concept in this repository.

**Concept: Defense in depth**

*Definition.* Defense in depth is the practice of stacking several independent security controls, so that the failure of one control does not lead to a full compromise.

*Analogy.* Think of a medieval castle. An attacker must cross a moat, climb a wall, get past guards and then break into the keep. Each layer is imperfect, but together they make the attack slow, noisy and expensive. The important detail is that the layers are independent: a drawbridge that fails open should not also disable the wall.

*Example.*

A web application stores customer data in a database. The layers might be:

1. A web application firewall filters obviously malicious requests before they reach the application.
2. The application validates input and uses parameterized queries.
3. The database account used by the application has `SELECT` and `INSERT` on two tables and nothing else.
4. The database is only reachable from the application subnet.
5. Data at rest is encrypted, so a stolen disk is not a stolen dataset.

If an attacker finds a SQL injection in layer 2, layers 3 to 5 still limit what they can read, change or exfiltrate.

*Pitfall.* Treating the layers as a checklist. Five controls that all depend on the same admin password are one control, not five.

Notice the pattern: the definition is precise, the analogy explains the idea in familiar terms, the example is concrete and reproducible, and the pitfall warns about a mistake that real practitioners make.

## 6. Examples, code and diagrams

### 6.1 Code

- Use fenced code blocks and always declare the language (` ```python `, ` ```sql `, ` ```bash `).
- Examples must be **tested**. If you cannot run it, mark it clearly as untested.
- State the **versions** of languages, tools and libraries involved.
- Keep examples **self contained**: imports, setup and sample data included.
- **Comment the why**, not the what. `# retry because the API rate limits at 10 req/s` helps. `# increment i` does not.
- Do not paste large listings. Show the relevant part and link to the full file if needed.
- Output is shown in its own block, labeled as output.

### 6.2 Diagrams, models and visuals

The library should be pleasant to look at and easy to scan. A reader should be able to take in the shape of a topic from its visuals before reading a line of prose.

**What every note has**

- **At least one visual** in "Core concepts" or "Worked example". Notes with several concepts usually need two or three: one overview of the whole topic and one for the central mechanism.
- **An overview for each area and subarea `README.md`** where it lists more than three notes: a `mindmap` or a `flowchart` of the notes and their order.
- **A chart for any table of numbers** in a worked example, when the numbers are compared or change over time.

**Choosing the form**

| What you show | Use |
| --- | --- |
| A process, a cycle or a decision | `flowchart` (left to right for steps, top to bottom for hierarchies) |
| The parts of a topic or a taxonomy | `mindmap` |
| Messages between actors over time | `sequenceDiagram` |
| Phases, stages or a plan over time | `timeline` |
| Two dimensions, such as effort and impact | `quadrantChart` |
| Numbers compared or trending | `xychart-beta` (bar or line) |
| States and transitions | `stateDiagram-v2` |
| Structure with exact geometry, such as a hexagon model or an annotated figure | hand-drawn SVG in `_assets/<area>/` |

**Rules**

- **Prefer text based diagrams.** Use a fenced `mermaid` block so the diagram is diffable and versioned with the note. Use a fenced `text` block for ASCII diagrams such as trees and data layouts. Mermaid renders on GitHub and in Librarian OS.
- **Keep diagrams small.** Split a diagram when it grows past about 15 nodes. Node labels are short noun phrases or verbs, not sentences.
- **A diagram earns its place** by showing structure, flow over time or relationships that prose cannot. Do not add one for decoration, and do not draw a box around a single idea.
- **Introduce it.** Every visual has a sentence before or after it saying what to look at. The same information is available in the text, so a reader who cannot see the image loses nothing.
- **Only facts from the note.** A visual never adds a claim that the text does not make or source. Invented numbers in a chart follow the same rule as invented numbers in a table: the note says they are examples.
- **Mermaid syntax pitfalls.** Quote labels that contain parentheses, colons or special characters. Avoid em dashes and en dashes in labels, as everywhere. Check that the diagram renders in Librarian OS before finishing.
- **Images and SVG.** Put the file in `_assets/<area>/` named after the note and the subject, keep the editable source in the same directory (`.drawio`, `.excalidraw`, or the `.svg` itself), and add descriptive alt text. A hand-drawn SVG has a `<title>` and `<desc>`, a light background of its own so that it reads in dark themes, and no text smaller than 11 px.
- **Other people's graphics.** Images taken from other people (vendor infographics, slides) are allowed only for personal study. Credit the author in the alt text or caption and prefer redrawing them as Mermaid or SVG when the note is reviewed.

*Example.* The distributed databases area uses a `mindmap` for the map of the area, a `flowchart` for the choice of a replication mode, a `sequenceDiagram` for the acknowledgments of synchronous and asynchronous replication and an `xychart-beta` for the share of keys that move when a node is added to a hash ring.

### 6.2.1 Scripts, files and media

- **Scripts** live in a `scripts/` directory next to the topic they belong to (see [section 3.1](#31-layout)). The note shows the relevant part and links to the full script. A script states its language and tool versions in a header comment, is self contained and says how to run it.
- **Downloadable files** (checklists, configuration templates, exported rules, small datasets) go to `_assets/<area>/`. Only files that are safe to publish belong here: no secrets, no real client or personal data, no live malware. Samples, captures and dumps are excluded by `.gitignore` on purpose.
- **Large media** (video, large captures) is linked from a public source instead of being committed. If a large file is essential, discuss it first because it stays in the git history forever.
- **Every asset and script** appears in `ToC.md`.

### 6.3 Tables

Use tables for comparisons across the same dimensions (features, costs, trade offs). Do not use tables for prose that happens to have two columns. Keep cells short.

### 6.4 Data

Use synthetic or public data only. If a dataset is needed, either generate it in the example or link to its public source. Never include personal, confidential or client data.

## 7. Writing style

The writing must sound like a knowledgeable person explaining something to a colleague. It should be plain, direct and specific.

### 7.1 Language

- **Everything is written in English.** This includes notes, headings, comments in code, commit messages, file names and the repository documents (README, GUIDELINE, ToC, AGENTS, CLAUDE). Source material in another language is translated into English when it is migrated.
- Use one variant of English consistently (American spelling is the default here).
- Prefer short sentences. If a sentence needs two readings, split it.
- Use the active voice and concrete subjects. "The scheduler drops the task" beats "the task is dropped".
- Prefer plain words over inflated ones. Write "use", not "utilize". Write "help", not "facilitate".
- Define acronyms on first use in each note, then use the acronym.

### 7.2 Punctuation: hyphens only

Use ordinary hyphens (`-`) and nothing longer. **Do not use em dashes or en dashes** in any file in this repository. They are a recognizable marker of machine generated text and they make prose harder to read.

Instead, rewrite the sentence:

| Instead of | Write |
| --- | --- |
| The cache, which is fast, is not durable | The cache is fast, but it is not durable |
| Two options: fast, or safe | There are two options: fast or safe |
| It works, in theory | It works in theory |

Guidance:

- Use commas, periods, colons and parentheses for pauses and asides.
- Use a hyphen for compound modifiers ("read-only replica", "low-level API") and for number ranges written with digits.
- If a sentence seems to need a dash, it is usually two sentences, or it needs a comma.

### 7.3 Avoiding machine sounding language

Text produced by language models has recognizable habits. Remove them, whether you wrote the text or an assistant did.

- No filler openers: "It is important to note that", "In today's fast-paced world", "Let's dive in", "In conclusion".
- No vague intensifiers or hype: "powerful", "seamless", "robust", "game changing", "cutting edge", "leverage", "delve", "landscape", "tapestry", "crucial" without a reason.
- No empty triads and rhetorical patterns ("fast, scalable, and reliable"), or the "It's not just X, it's Y" construction.
- No summary sentence that only repeats the paragraph.
- No decorative emojis and no exclamation marks.
- No headings that are questions or slogans. Use plain descriptive headings.
- Do not over-format. Bullet lists are for parallel items, not for every thought. Bold is for a term being defined or for a rare warning.
- Be specific. Replace "many companies use this" with a named example, or with nothing.

**Before:**

> It is important to note that database indexes are a powerful tool that can seamlessly boost your query performance, making your applications faster and more scalable!

**After:**

> An index lets the database find rows without reading the whole table. A lookup by customer ID on a table with 5 million rows drops from a full scan to a few page reads. The cost is slower writes, because every insert must update the index.

### 7.4 Formatting conventions

- One top level heading (`#`) per file, matching the title. Then `##` and `###`. Do not skip levels.
- Headings use sentence case: "Query performance", not "Query Performance".
- Use backticks for code, commands, file names, and identifiers. Use *italics* for the first mention of a new term when it is being defined.
- Wrap nothing by hand. Write one paragraph per line, so diffs stay readable.
- End every file with a single newline.
- Numbers: use digits for measurements and counts above nine, write out smaller numbers in prose, and always include units.

## 8. Sources, references and verification

- **Prefer primary sources.** Specifications, official documentation, original papers and source code beat blog posts and videos. Use secondary sources to find primary ones.
- **Sources are listed in "Further reading".** That section is the sources list of the note, and every note keeps it filled in: title, author or organization, year and a link. Structured detail goes into `sources` in the sidecar.
- **No "Source material" sections.** Notes and READMEs do not describe which export, file or earlier note their content came from. The sources list names only real references (papers, standards, official documentation, books).
- **Cite what you use.** Put links in the "Further reading" section, or inline for a specific claim. Include enough to find the source if the link dies: title, author or organization, year.
- **Write in your own words.** Summaries are paraphrased. Quote only when the exact wording matters, keep quotes short, and mark them clearly.
- **Record the date.** For anything that changes over time (versions, prices, limits, recommendations), write the version and the date it was checked in the text, for example "Checked against NIST CSF 2.0, 2026-09".
- **Separate fact, interpretation and opinion.** Mark opinions as opinions ("In my experience", "I prefer this because").
- **Check claims that sound too good.** Reproduce numbers when you can. Look for a second source for anything surprising.

Typical primary sources here are the SQL standard (ISO/IEC 9075), the official documentation and release notes of each engine (PostgreSQL, MySQL, SQLite, MongoDB, Redis, Apache Cassandra, Neo4j, Elasticsearch), the original research papers (Codd 1970 on the relational model, Gray and Reuter on transactions, Dynamo, Bigtable, Spanner, Karger et al. on consistent hashing), standard textbooks (Silberschatz, Korth and Sudarshan, *Database System Concepts*; Kleppmann, *Designing Data-Intensive Applications*; Petrov, *Database Internals*), the engineering blogs of the companies in a case study, and standards for security (OWASP, CWE, NIST). For security notes see also [section 9.1](#91-security-notes).

## 9. Rules specific to this repository

Database theory changes slowly, but engines change every year: new major versions, changed defaults, new features, renamed products and changed licenses. The rules below keep the notes true over time.

- **Name the engine and version.** A statement about engine behavior (a default, a limit, a feature, a planner decision) names the engine and the major version it was checked against, with the date, for example "Checked against PostgreSQL 17, 2026-11". A concept note that gives engine examples does the same for each example.
- **Separate the standard from the dialect.** SQL examples say whether they use standard SQL (ISO/IEC 9075) or a dialect, and which one. Prefer standard syntax in concept notes.
- **Date licenses and product names.** Licenses of database engines change (Redis, Elasticsearch, MongoDB and CockroachDB all changed theirs). A license statement carries the date it was checked. Former product names go into `node.aliases`.
- **Mark deprecated features.** When a note describes something that is deprecated or removed in a current version, say so and name the replacement.
- **No real data or identifiers.** Use synthetic data, placeholders such as `<PASSWORD>` and `example.com`, and never commit connection strings, credentials, dumps or real customer data.
- **SQL and commands are tested or marked.** A query or command is either run against the named engine and version (for example in a throwaway Docker container) with its real output shown, or marked as untested.
- **One home per concept, one home per engine.** See [section 3.1](#31-layout). A note that touches many concepts links to their home notes instead of repeating them.
- **Case studies describe real systems.** Notes in `case-studies/` describe the problem, the chosen design, the trade offs and the technologies used, in my own words, and link to the concept and engine notes and to the original source.
- **Drifting topics.** If a note drifts into another area, keep the part that serves its own area and link to the home note of the rest.

### 9.1 Security notes

These rules apply to every note under `database-security/` and to the security sections of other notes.

- **Authorized use only.** An offensive technique (for example a SQL injection) is described only in the context of a lab, a CTF, a penetration test with written permission, or defensive research. State this context explicitly in the note.
- **Defense first.** Each attack description ends with detection and mitigation.
- **No live secrets.** Use placeholders such as `<API_KEY>` and reserved documentation addresses (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`, `example.com`).
- **Reference identifiers.** When a note discusses a known weakness, link the CVE, CWE or MITRE ATT&CK technique ID so the reader can verify the claim.

## 10. Quality checklist

Run this list before marking a note as `reviewed`.

**Content**

- [ ] The summary is correct on its own and written in plain language.
- [ ] Every new term is defined at its first use.
- [ ] The problem the topic solves is stated before the mechanism.
- [ ] There is at least one worked example that can be reproduced.
- [ ] At least one analogy is used, and its limit is stated.
- [ ] Trade offs, alternatives and failure modes are covered.
- [ ] Common mistakes are listed with their fixes.
- [ ] Versions and dates are recorded where relevant.

**Accuracy**

- [ ] Claims are backed by sources, and the sources are linked.
- [ ] Code was run, and the output shown is real.
- [ ] Numbers are reproducible or have a stated origin.

**Style**

- [ ] The note is entirely in English.
- [ ] No em dashes or en dashes anywhere (search for them before committing).
- [ ] No filler phrases, hype words or decorative emojis.
- [ ] Headings are descriptive and in sentence case.
- [ ] Links are relative and work.

**Visuals**

- [ ] The note has at least one diagram, model or chart, and it renders in Librarian OS.
- [ ] Each visual is introduced by a sentence, and shows only facts from the note.
- [ ] Charts match the numbers in the table they illustrate.
- [ ] Images have alt text, and SVG files have a title, a description and their own background.

**Hygiene**

- [ ] No secrets, personal data or confidential material.
- [ ] File and directory names follow the naming rules.
- [ ] The note is linked from its area or subarea `README.md`.
- [ ] `ToC.md` reflects every added, moved, renamed or deleted file, and its links resolve.
- [ ] The note has a sidecar (`.md.json`), it follows `INSTRUCTIONS.md` of Librarian OS, and `npm run check` in Librarian OS passes.

A quick way to check the dash rule from the repository root:

```bash
grep -rnP "\x{2013}|\x{2014}" --include="*.md" --include="*.json" --include="*.ts" --include="*.scss" --exclude-dir=node_modules .
```

`npm run check` repeats this test for all text files it reads.

## 11. Maintenance and versioning

- **Small, focused commits.** One note or one logical change per commit. Explain the reason in the message.
- **Commit message format.** A short imperative subject in English, for example `Add note on B-tree indexes`. Add a body when the reason is not obvious.
- **Branching.** Use short lived branches for larger additions. Small edits can go directly to the main branch.
- **Review cycle.** New notes contain `TODO:` lines until the skeleton is filled. After that, check the content against sources and record what was checked in the text.
- **Refresh.** Revisit notes on fast moving topics regularly, update the recorded check date and fix what changed. Delete or rewrite content that is no longer true instead of leaving it with a warning.
- **Refactor structure early.** When a note grows too large, split it. When two notes overlap, merge them or extract the shared part into its own note and link to it.
- **Keep the index honest.** Every new note is added to its area or subarea `README.md` and to `ToC.md` in the same commit. Every move, rename or deletion is reflected in both as well.

## 12. Working with AI agents

AI assistants are welcome collaborators. The rules are the same as for a human contributor, and the human author remains responsible for correctness.

- **Read this file first.** `AGENTS.md` and `CLAUDE.md` point here. This GUIDELINE is the only place where conventions are defined.
- **Do not invent facts.** If a detail is uncertain, say so in the note or leave it out. Never fabricate citations, version numbers, benchmarks or quotes.
- **Verify before writing.** Use primary sources for claims, and record them.
- **Follow the style rules strictly.** In particular the ban on em dashes and en dashes, and the ban on filler language (see [section 7](#7-writing-style)).
- **Make it visual.** Add diagrams, models and charts as described in [section 6.2](#62-diagrams-models-and-visuals). A note with only prose and tables is not finished.
- **No "Source material" sections.** Never mention `_sources/` exports in a note. Fill in the sources list instead.
- **Stay in scope.** Do not add content unrelated to the request. Do not restructure the repository without being asked.
- **Prefer editing to creating.** Extend an existing note when the topic already has a home. Create a new file only when the topic deserves its own note.
- **Follow `INSTRUCTIONS.md`.** For every Librarian OS adaptation (configuration, sidecars, entry points, checks) follow `INSTRUCTIONS.md` of Librarian OS, and read it before creating the first note of a session.
- **Keep sidecars in step.** A new or renamed note gets a sidecar (`npm run scaffold`), and relations are added only to ids that exist. Run `npm run check` in Librarian OS before finishing.
- **Update `ToC.md` with every change.** Any file added, moved, renamed or deleted must be reflected in `ToC.md` before the task is finished (see [section 3.4](#34-table-of-contents)).
- **Ask when a decision is unclear.** Structure, scope and naming decisions belong to the author.
- **Never commit secrets or personal data,** and never run destructive git operations without explicit approval.
- **Show your work.** When adding a note, summarize what was written, which sources were used and what remains unverified.
