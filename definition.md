# The Definition of Markleft

*Crown copyright, Canada — 2026. Licensed CC BY 4.0.*

## Status of this document

This is a working draft, written during Phase 0 of the project. The language it describes is settled in substance: every rule stated here comes from a recorded decision. Nothing is frozen. The formal grammar in Appendix B is not written, the conformance suite does not exist, and no implementation has shipped — so no conformance claim is possible yet, and any rule here may still change. Appendix D lists, precisely, every point this document leaves open.

The document is named *definition* rather than *specification* or *standard* for a reason worth stating once. Metrology defines the metre in a single sentence and then checks every physical realization against that definition; the most rigorous document in the history of programming languages is *The Definition of Standard ML*. Both senses apply here. The name also says what this document is **not** — a realization. It was called `charter.md` until 2026-08-07; in IETF and W3C practice a charter is a working group's governance document and never the technical text, so the old name pointed at the wrong thing. If sections 10 to 12 ever split into a separate document, that document is legitimately the charter.

## How to read this document

**Normative sections** state what the language is and what an implementation must do: section 3 (invariants), section 4 (the language), section 9 (conformance), section 12 (versioning), Appendix A (deltas), and Appendix B (the grammar, when it exists).

**Informative sections** explain, justify, and position: sections 1, 2, 5, 6, 7, 8, 11, and Appendices C and D. Section 10 states a legal position rather than a technical requirement. Where an informative section appears to state a rule, section 4 governs.

**Requirement words.** *Must* states a requirement on a conforming implementation. *Must not* states the same requirement negatively. *May* marks a free choice — an implementation is free to do the thing, and free not to. No normative section of this document uses *should*: a rule that only recommends is a rule the language cannot rely on.

**Terms.** A **document** is the complete text an author writes. A **parser** turns a document into a tree. A **renderer** turns that tree into something a person looks at — a web page, a printed page, a screen reader's speech. A **realization** is any of these: an independent implementation of this definition. A **validator** reports problems a parser never fails on. A **formatter** rewrites a document into one canonical form. A **migrator** converts Markdown into Markleft. The **conformance suite** is the executable set of examples every realization is checked against.

Examples in this document are shown in fenced blocks. Where an example shows what a construct means, the meaning is stated in words beside it; this document never asserts a particular HTML output, because the language is not defined in terms of HTML.

## 1. Purpose and scope

Markleft is a language for writing documents in plain text. It is the size of Markdown, it looks like Markdown, and it is meant to be learned in one sitting. What it adds is certainty: a formal grammar, an executable conformance suite, and exactly one parse for every input.

### 1.1 The problem

Markdown has no single definition. Gruber's 2004 description was informal and incomplete; CommonMark standardized what had been built on top of it; GitHub Flavored Markdown extends CommonMark; and every platform then adds its own layer of processing outside the parser. The result is that the same source produces different documents depending on where it is read. That is not a matter of taste in rendering. It is the text itself changing meaning between one reader and the next.

### 1.2 The founding exhibit

This project began as an investigation of one character. No Markdown specification — not Gruber 2004, not CommonMark, not the GFM specification — defines the dollar sign as mathematical notation. Dollar-delimited mathematics is entirely a renderer-level addition, bolted on outside the parser: GitHub added a MathJax pass in 2022, Pandoc has an extension with elaborate heuristics, and KaTeX ships with single-dollar delimiters disabled by default precisely because they are unsafe.

While that investigation was being written up, the paragraph explaining the problem was itself destroyed by exactly the mechanism it described. Two unrelated dollar signs in ordinary prose were paired by a chat interface's mathematics pass, the text between them was reinterpreted as mathematics, its spacing collapsed, and code spans gave no protection because the mathematics pass ran after the Markdown parser had finished.

That is the whole problem in one incident: **the same source, two different documents, and no way to say which one was correct.** In metrology this is a traceability failure — a measurement that cannot be traced to a definition is not a measurement. In a document format it means an author cannot know what a reader will see.

### 1.3 What this document does

It defines the language. Specifically, it states the five invariants that every part of the language must satisfy (section 3), the language itself (section 4), the reasoning behind each binding decision including the alternatives it set aside (section 5), what conformance means and how it is checked (section 9), and the project's licensing and governance position (sections 10 and 11).

### 1.4 What is in scope, and what is not

Markleft defines what a document **is**. It does not define what a renderer **does** with it. The language guarantees that a document has one unambiguous meaning; how that meaning is typeset, coloured, or spoken aloud is left free, and always was. This line is drawn deliberately and appears throughout section 4 — most visibly around mathematics, where the language guarantees that content is carried through untouched and says nothing whatsoever about how it is typeset.

## 2. Non-goals

Four things Markleft is not. Each rules out a different misreading, and each is held by a rule rather than by intention.

### 2.1 Not a toolchain, and not a document-preparation system

There is no templating, no executable code, no scripting, no page or layout awareness, and no output-format directive. Languages that have those things — Quarto, Typst, Quarkdown, and LaTeX before them — are a different and legitimate product. They are also, all of them, languages in which the first five minutes are expensive, and adoption is decided in the first five minutes.

### 2.2 Not a richer Markdown for its own sake — and not an austerity project either

Markleft adds things. Mathematics is part of the core, decorators are new, pipe tables are in the language rather than in a platform's post-processing, and several constructs are adopted from djot. What every addition has to clear is a single bar: **it removes an ambiguity or fills a genuine gap, and it costs nothing in plain-text clarity.** A construct that makes ordinary unmarked prose read worse does not clear it, however much it offers elsewhere, because prose-safety is invariant 1 and richness is not an invariant at all.

The budget for all of this is the five-minute property. Most of the binding decisions in section 5 remove something; a few add; the total still fits on one reference card. So "Markleft could also do X" is never an argument by itself. The argument has to be that X is worth what it costs every reader who never uses it.

### 2.3 Nothing renders that is not in the source

This is invariant 5, stated here as a promise rather than as a rule. There is no table-of-contents directive, no automatic section numbering, no generated index, no transclusion of text, and no variable substitution. **A reader can always read the entire document in plain text, with no compiler, viewer, or extension.**

The important thing about this non-goal is that it does not remove capability; it relocates it. Tables of contents, citation formatting, cross-file snippet synchronization, and figure numbering are all genuinely useful, and all of them are better done by a tool that writes plain text **into** the source at authoring time than by a renderer that produces text at display time. The tool's output is then ordinary text: a reader without the tool still sees it, a diff still shows it, and the format has gained nothing it must carry forever.

That last clause is the strategic point. Because capability accumulates in tools rather than in the language, the first five minutes never get worse however large the ecosystem grows. Most small languages start small and then grow. This one has a structural reason not to.

### 2.4 Not a Markdown dialect, and not compatible with one

**A Markleft document is not a Markdown document.** The two languages share a block layer — headings, lists, block quotes, fences, tables — and every inline construct differs: links, images, emphasis, anchors, and escapes. A Markdown file will not parse as intended here, and a Markleft file will not render as intended there.

That is a deliberate position rather than an accident of design, and the argument for it is Markdown's own history. Near-compatibility is what produced flavour drift: when every implementation is *almost* the same as every other, a writer cannot answer whether their document survives a move between platforms, and the differences stay invisible until they bite. That is section 1.2's failure at ecosystem scale. **A clean break makes the question answerable.** A platform can support Markdown and Markleft side by side, as two languages with two file extensions; what stops paying is the temptation to stretch one implementation across both, which is how a dialect is born.

**The spirit is inherited; the syntax is not.** What Markleft takes from Markdown is the thing Markdown got right — that a source file should read as prose, and that the marks should be few and quiet. Section 6 records the inheritances specifically, and decisions 11, 12, and 20 record where the line falls and why.

## 3. The invariants

Five invariants. They are constitutional in the ordinary sense: a proposal that breaks one cannot land as it stands, and changing that means amending the invariant itself — in the open, with the argument written down. No rule elsewhere in this document contradicts them.

### 3.1 Invariant 1 — Prose-safety

Natural-language prose renders verbatim. A paragraph means what it says unless it contains a delimiter in a structurally meaningful position, and the set of such positions is small enough to state on one page.

The word doing the work is *by construction*. Money amounts, `snake_case` identifiers, arithmetic like `5 * 3 = 15`, shell commands, file globs, and lines that happen to begin with a hash or a hyphen are safe because no rule in the language claims those characters in those positions — not because a heuristic guessed correctly. Prose-safety is testable, and it is tested: the conformance suite includes a corpus of prose written specifically to attack it.

### 3.2 Invariant 2 — The five-minute property

The complete core fits on one reference card, and **no rule has an "unless" clause.** A rule that needs an exception is a wrong rule, and the exception is evidence that something upstream was designed badly.

This is the invariant most additions fail against, because almost every attractive addition arrives with a small exception attached.

### 3.3 Invariant 3 — The one-meaning property

Every input has exactly one parse. Not one parse per implementation, not one parse per platform: one parse.

This is why the definition is executable rather than interpretable. Prose can be read two ways; a grammar and a suite of worked examples cannot. Two authors may write the same document two ways — that is style. One document parsing two ways is a defect.

### 3.4 Invariant 4 — The linear-time property

A single pass, no backtracking, and no pathological inputs. Which block a line opens or continues is decided from that line and the containers already open — never from a later line. Parsing time is linear in the length of the document.

This is not only a performance claim. Prefix-decidability is what makes the language explainable: a rule that depends on a later line cannot be stated as a rule about the line in front of you.

### 3.5 Invariant 5 — The cardinal rule

**You can always read the entire document in plain text, with no compiler, viewer, or extension. Nothing renders that the source does not already say.**

The rule is enabling, not restrictive. It is the minimum that lets capability live in tooling and be built cleanly: a tool that writes into a document needs to know that what it wrote is what the reader will see, and this rule is that guarantee. Features are relocated, not forbidden.

The operative detail — where exactly the line falls between presenting written content and manufacturing new content — is decision 15 in section 5.

### 3.6 Five invariants, six guarantees

The project's public material lists **six guarantees**: prose-safety, "verbatim means verbatim", five minutes, one meaning, linear time, and "read it anywhere". The counts differ on purpose and the relationship is this: the invariants are the internal constitutional tests a proposal must hold; the guarantees are the user-facing promises that follow from them.

Five of the six correspond one-to-one with the five invariants. The sixth, **verbatim means verbatim** — content inside a verbatim span or fence reaches the tree exactly as written, TeX backslashes included; a delimiter appearing in content is answered by lengthening the delimiter, with no second mechanism; and every Unicode code point is text — is not an invariant. It is a promise derived from decisions 5, 14, and 20. It is stated publicly because it is what an author most wants to be told, and it is not stated as an invariant because it is a consequence rather than a test.

## 4. The language

This section is normative. It states the whole language in prose. The formal grammar in Appendix B, when it is written, pins the exact forms; where the grammar and this prose disagree, the disagreement is a defect to be triaged under the precedence rule in section 9.

### 4.1 Characters, encoding, and lines

A Markleft **document** is a finite sequence of Unicode code points.

A document is stored and exchanged as **UTF-8**. Decoding happens before parsing and is a separate step: the parser sees code points, never bytes. A byte sequence that is not well-formed UTF-8 is not a Markleft document, and a tool handed one reports an encoding error rather than guessing at a repair. *(This clause closes an open rider; see Appendix D.1.)*

Every code point is text. No part of the language privileges an ASCII subset. The full Unicode range is available in prose, in headings, in link text, in verbatim content, and in decorator tokens alike, and every code point carries the same guarantees as any other.

**Content is never normalized.** A conforming parser applies no Unicode normalization form, changes no case, and substitutes no character. What the author wrote is what the document contains. This is why there is no smart punctuation in the language: quotation marks and dashes are what the author typed.

It follows that **two identifiers or labels match when they are the same sequence of code points, and not otherwise.** There is no case-folding and no normalization. This keeps matching independent of the Unicode version: a rule that folded case would let two labels stop matching, or start matching, on a Unicode release that nobody in this project made. *(This clause closes an open rider; see Appendix D.3.)*

If a document begins with U+FEFF, that code point is not part of the document text. Anywhere else, U+FEFF is ordinary text. *(This clause closes an open rider; see Appendix D.2.)*

A **line ending** is a line feed, a carriage return, or a carriage return followed by a line feed. A **line** is a sequence of code points up to the next line ending or the end of the document; the last line need not end with a line ending. A **blank line** is a line holding no code points, or only spaces.

**Tabs never determine structure.** Where the language measures structure in columns, a tab is not a substitute for spaces. A tab in a structural position is well-formed input and produces a validator warning with an automatic fix, never a parse failure.

**Trailing whitespace at the end of a line is never significant.** A line and the same line with spaces appended are the same input.

Where this document measures a **column**, it counts code points from the start of the line, beginning at column 1. Display width is not used. This means text containing wide characters aligns by count rather than by apparent width, which is stated plainly here rather than discovered later. The reason is durability: display width is a versioned Unicode property whose values have moved between releases, and a rule that consults it would let a document's parse change over time under a definition nobody edited.

### 4.2 How a document is read

Parsing has two stages. First the **block structure** is determined, line by line. Then **inline content** is parsed inside those blocks that contain text.

Block structure is decided from each line's own prefix together with the containers already open. No rule looks ahead to a later line, no rule backtracks, and parsing takes time linear in the length of the document.

**The uniform fallback.** A line that does not match the opening form of any block is paragraph text. Inline content works the same way: a sequence of characters that does not match a construct's form is text. This single fallback is what makes prose safe, and it is why so few rules in this document need to say anything about prose at all — prose is simply everything that is left.

**The parser is total.** Every document that decodes as UTF-8 has exactly one parse, and there are no parse errors. A fence that is never closed closes at the end of its container. Emphasis that is never closed is an asterisk. A table row that does not match the table form is a paragraph. Nothing an author can type causes a parser to refuse the document.

Judgements that need more than the text in front of the parser therefore belong to the **validator**, and they are warnings, never failures. There are ten of them today:

- a decorator word the validator does not recognize;
- an anchor identifier used more than once in a document;
- a link to a fragment in the same document that no anchor defines;
- an image with no alternative text;
- a tab in a structural position;
- a decorator that only restates what the bare construct already means;
- a verbatim block closed only by the end of its container, naming any unmatched backtick run it passed on the way;
- a superscript or subscript whose brace group is empty;
- a decorator-shaped brace group following anything other than a backtick run, which usually means an apostrophe or an acute accent was typed for a backtick;
- a sigil followed directly by a bare alphanumeric run — the TeX-habit spelling `E=mc^2`, left as text by the uniform fallback.

This split is deliberate and stable: the parser is local and total, the validator carries every judgement, and the two never trade places.

### 4.3 Blocks

#### 4.3.1 Paragraph

A paragraph is a sequence of lines containing inline content. It is what a line becomes when it matches nothing else. Paragraphs are separated by blank lines.

#### 4.3.2 Heading

**A heading is a run of one to six number signs in the first column, then one space, then non-empty text. A line that does not match that form is text.**

That is the entire rule, and every case falls out of it rather than being legislated separately.

````
# Title                    a level-1 heading
###### Deepest             a level-6 heading
####### Seven              text — seven number signs match no heading
#hashtag                   text — no space after the run
#5                         text — no space after the run
 # Indented                text — the run is not in the first column
#                          text — no heading text
## Title ##                a level-2 heading whose text is "Title ##"
````

Number signs are the only heading marker. There are no underlined (setext) headings: besides being a second spelling of one thing, an underlined heading is a heading only because of the *next* line, which invariant 4 rules out.

There is no closing sequence. In `## Title ##` the text is `Title ##`. CommonMark's rule for when a trailing run closes a heading and when it does not is the clearest available example of the "unless" clause invariant 2 leaves no room for.

There is no leading indentation: the run sits in the first column, after any container prefix such as a block-quote marker or list-item indentation. CommonMark admitted up to three spaces in order to keep headings clear of indented code blocks, and decision 5 removed indented code blocks, so the allowance now buys nothing and costs a boundary case.

There is no empty heading. A line holding only a number sign is a paragraph whose text is a number sign.

Six levels, because HTML has six, DocBook and LaTeX agree, and CommonMark caps there too. A document that needs a seventh level needs splitting.

Note what this rule does for prose without any special case: `#hashtag` and `#5` are text at the start of a line because neither has the required space. The one genuinely ambiguous spelling, `# 5 is my favourite`, is written `\{#} 5 is my favourite`, using the ordinary escape from section 4.4.2.

#### 4.3.3 Verbatim block

A **backtick run** is a maximal sequence of backticks: the characters immediately before and after it are not backticks. Every rule in this document that counts backticks counts a run in this sense, at both sizes.

A verbatim block opens with a run of three or more backticks at the start of a line, after any container prefix, optionally followed by a decorator list (section 4.5). It closes with a run of **the same length**, alone on a line. An unclosed block closes at the end of its container.

Everything between the fences is **verbatim**: no escape is processed, no inline construct is recognized, and no decorator is read. The content reaches the tree exactly as written.

**Why a run rather than a fixed delimiter.** Because no escape is processed inside, decision 3's universal backslash — the answer to every other "how do I write the delimiter literally" question in this language — is switched off in the one construct whose delimiter an author most often needs to write. The run length is therefore the entire escape mechanism, and it is the only one there is. One rule states it at both sizes: **choose a length that does not occur as a run in the content.** At this size the content is lines, so only a run alone on a line can close the block.

**The minimum of three is a separate rule with a separate reason**, and it is not about content at all. A block's type is decidable from its own first line (section 4.2), so if one backtick could open a fence, a paragraph beginning `` `x = y` is the formula `` would open a verbatim block instead of being prose. Three is the smallest run that does not begin an ordinary sentence, and it is CommonMark's number.

**Closing on an exact match is a delta from CommonMark** (Appendix A), which also closes on a longer run. Exact matching gives the language one counting rule instead of two, and it lets a document show a longer run inside a shorter fence — which a specification about fences needs. The cost is a real failure mode and is stated rather than hidden: a block opened with three backticks and closed with four is not closed, and runs to the end of its container. That is well-formed input with one parse, so it is the validator that reports it.

````
```
Everything here is literal. \n stays \n. *stars* stay stars.
```

```math
\frac{\partial u}{\partial t} = \alpha \nabla^2 u
```

```rust #example-3
let total: u32 = items.iter().map(|i| i.count).sum();
```
````

There are no indented code blocks. Four leading spaces are four leading spaces.

#### 4.3.4 Block quote

A block quote is introduced by a `>` at the start of a line, optionally followed by one space, and contains blocks. Its form is inherited from CommonMark; whether its lazy continuation is inherited too is open (Appendix D.7), and the no-lazy-continuation discipline of section 4.3.5 argues that it is not.

#### 4.3.5 Lists

Lists are the most rule-heavy part of any Markdown-like language, and the place where implementations most often disagree. Markleft replaces the archaeology with arithmetic.

**There is one marker of each kind.** A bullet item is `-`, one space, then content. An ordered item is one or more digits, `.`, one space, then content. `*`, `+`, and `1)` are not list markers: a line beginning with one of them is text.

**Content-column alignment.** An item's **content column** is the column at which its content begins — the marker, one space, then content. Every continuation line of that item is indented to exactly that column.

**No lazy continuation.** A line that is not indented to the content column is not part of the item. Markdown's tolerance for under-indented continuation lines is the single largest source of disagreement between implementations, and it is removed.

**Adjacent items are one list.** A list ends only where a block that is not a list item begins — a paragraph, a heading, a fence, or a table. **Blank lines never split a list:** one makes it loose, and two do nothing further. Markdown's trick of switching markers to separate two touching lists does not exist here, because there is no second marker to switch to.

````
- The first item.
  Its second line is indented to the content column.

- The second item.

10. An ordered item.
    Its content column is column 5.
````

**Ordered lists renumber.** A list begins at its first marker's value and its items render sequentially from there, whatever numerals are written. `1.` `1.` `1.` renders as 1, 2, 3.

This is worth defending explicitly, because it looks at first like generated content, which invariant 5 leaves out. It is not. A rendered ordinal is not text taken from somewhere else; it is a presentation of the item's **position**, and position is fully stated by the source. A reader looking at three items in plain text already knows there are three, in order — they can count. Nothing is relocated, as a table of contents relocates heading text, and nothing is imported, as an image imports a file. What makes the behaviour necessary rather than merely available is that hand-maintained numerals are wrong in practice: they break silently on every edit, and the failure is invisible until someone reads the rendered output.

#### 4.3.6 Table

The whole of a table's structure is carried by one character. **Any line beginning with `|` opens a table** — a vertical mark begins a vertical arrangement — and every rule below follows from that.

````
| Construct | Position | Verbatim |
|-----------|----------|----------|
| fence     | block    | yes      |
| span      | inline   | yes      |
````

**Cells are separated by `|`.** A line beginning with `|` and carrying content holds inline cells, one per pipe, which is how Markdown has always written them. Every structure line begins and ends with `|`; the closing pipe terminates the line and opens no cell.

**Rows form by counting cells** up to the table's column count, which is set by the first row — a rule row or an ordinary one. A single-column table needs no rule row.

**The rule row fixes alignment.** Each of its cells is a run of three or more `-`, optionally carrying `:` at one end or both.

````
| Construct | Position | Verbatim |
|:----------|:--------:|---------:|
| fence     | block    | yes      |
| span      | inline   | yes      |
````

A leading colon aligns the column left, a trailing colon right, both centre it. A bare run states no preference, which is not the same as stating left: the two may render identically, and the tree records which the author wrote — the same rule as a labelled and an unlabelled verbatim span in section 4.5.

**The rule row's position says whether there is a header.** First line: no header. Second row: the row above it was the header. Absent: neither, and alignment is whatever the renderer does by default.

**A rule row is recognized only as a table's first or second row.** Anywhere else, a row of dashes is content. That narrowing and the three-hyphen minimum exist for the same reason: a cell holding a single dash is a common way to write "not applicable", and without them it would be read as structure and its content would disappear. A cell whose content is genuinely three or more dashes, in one of those two positions, is written `\{---}` — the ordinary escape of section 4.4.2, which keeps the cell from matching the rule-row form.

````
|-----------|----------|
| fence     | block    |
| span      | inline   |
````

**A `|` alone on a line opens a block cell**, whose content is every following line up to the next line beginning with `|`. A block cell holds blocks — paragraphs, lists, fences — so a cell may run to any length without its source running to any width.

````
| Invariant    | What it promises |
|--------------|------------------|
| Prose-safety |
|
Natural-language prose renders verbatim.

Money, `snake_case`, and shell snippets are safe by construction, never by heuristic.
|

This paragraph is outside the table.
````

**A table ends at a blank line immediately following a line that begins with `|`**, or at the end of its container. An ordinary inline table therefore ends on its own, its last row line beginning with `|` already. A table whose last cell is a block cell ends with **the closing mark — a `|` alone on a line, before a blank line — which opens no cell.** Write an empty cell inline, as `| |`.

The form is exact. A block that does not match it is not a table and, under the uniform fallback, is a paragraph. There are no heuristics for ragged rows, no guessing at intended column counts, and no recovery: a table is either well-formed or it is prose.

Two properties are worth naming, because they are why the construct is shaped this way.

**A table is decidable from its own first line**, like every other block. Markdown's table needs to see the next line before it knows whether the current one is a header or a paragraph, which is the property section 4.3.2 removed setext headings for. Opening on the pipe removes it here too, rather than carving tables an exception.

**A block cell is a container**, in the same sense as a list item or a block quote: inside it, lines belong to the cell. Note the one difference, stated rather than left to discovery — a list item marks every line it owns by indentation, where a block cell marks only its first and relies on the terminator. Both are coherent, and the reason a cell does not use indentation is that the `|` already does the work indentation does in a list.

#### 4.3.7 What does not open a block

Six characters open a block, and the list is closed: `` ` ``, `#`, `-`, a digit, `>`, and `|`. Any other line is a paragraph. Two constructs a reader may look for are absent.

**There is no thematic break.** A line of three or more `-`, `_`, or `*` reserves nothing; it is paragraph text, or a bullet item where it takes that form (`- - -`). The construct is removed because it never carried a plain-text meaning: a run of hyphens says that something changes and never says what, where a heading says which. Its rendering was a second problem — a horizontal rule is a style choice, and it collides with the rule most house styles already draw under a level-2 heading. **Where a break was wanted, write the heading it stood in for, or a second document.** Decision 19.

**There is no link reference definition.** A line beginning with `[` reserves nothing. Reference links are absent (section 4.4.6), so there is nothing for a definition to define, and a link's target is written where the link is.

### 4.4 Inline content

**One rule governs this section: a sigil carries meaning only when the very next code point is `{`.**

Every construct below is an instance of it, with one stated extension: the link and the image, when they carry their own text, put `[` where the brace would be and the brace group after it (section 4.4.6). Eight sigil characters share the shape, every construct opens on a sigil run and the single code point after it, and each of those characters is ordinary text everywhere else in a line.

| Opener | Construct | Written |
|---|---|---|
| `*{` | emphasis | `*{em}` |
| `**{` | strong emphasis | `**{strong}` |
| `***{` | both | `***{both}` |
| `^{` | superscript | `10^{23}` |
| `_{` | subscript | `H_{2}O` |
| `#{` | anchor | `#{tessier-2026}` |
| `@{` | link | `@{https://example.org}` |
| `@[` | link carrying its own text | `@[Tessier et al.]{#tessier-2026}` |
| `!{` | image | `!{assets/kangaroo.png}` |
| `![` | image carrying alternative text | `![a kangaroo in Cairns]{assets/kangaroo.png}` |
| `\{` | escape range | `a \{\|} b` |
| `` `{ `` | decorator list | `` `x=y`{math} `` |

What follows from the rule, and what makes it worth stating before any construct:

**Prose-safety is by construction here, with nothing left over.** No condition is consulted about what surrounds a character, so no reader has to know one. `5 * 3 = 15`, `snake_case`, `2^32`, `[sic]`, `C:\Users\frederic`, `#hashtag`, `@handle`, `{"key": 1}`, and `f(x)` are all text, and none of them needs an escape.

**Every construct is decidable from its sigil run and one more code point.** A left-to-right scan reaches a sigil before anything else, and no two constructs share an opener, so no rule in this section looks ahead, backtracks, or consults a table.

**Every mark sits to the left of its own content.** Blocks mark the left of a line, inlines mark the left of a span, and a backslash before a line ending marks the left of the line ending. The one place the arrow reverses is the decorator list, which follows what it annotates; section 4.5 says why.

The one exception to the shape is the verbatim span, delimited on *both* sides because its content is not interpreted and so cannot be balance-counted (section 4.4.5). The bracket forms are not an exception but the stated extension above: what a reader sees goes in front, and what it means goes in the braces (section 4.4.6).

#### 4.4.1 Text

Anything that does not match a construct below.

#### 4.4.2 Escape ranges

**`\{…}` makes its content literal.** No construct is recognized inside it. A backslash followed by anything other than `{` is an ordinary backslash.

````
a \{|} b               yields   a | b
\{*{not emphasis}}     yields   *{not emphasis}
C:\Users\frederic      yields   C:\Users\frederic — unchanged, no escape needed
\{}                    yields   nothing
````

The range is what an escape needs to be in this language, for two reasons that arrive together.

**The escape is visible.** `a \{|} b` says *I meant this pipe*, where a lone `\|` in a paragraph reads as a typo. The braces delimit what is protected, so a search-and-replace that empties one leaves `\{}` behind rather than a stray backslash.

**Almost nothing needs escaping.** Under the rule in section 4.4, ordinary writing produces no operative sequences at all, so a heavier escape is reached for far less often than a lighter one was. The characters that still want one are the six that open blocks, and only in first position: `\{-} 5 degrees below`, `\{#} 5 is my favourite`, `\{|}x| < 5`.

A single backslash is *not* an escape. This is a deliberate departure from CommonMark, which escapes ASCII punctuation, and from an earlier form of this language, which escaped every character with no exception list. That form is what made `C:\Users\frederic` render as `C:Usersfrederic` — text altered with nothing on the page to show it, which is precisely what invariant 1 exists to prevent. Decision 3.

#### 4.4.3 Hard line break

A backslash immediately before a line ending is a hard line break. It is the only one, and it is the backslash's only operand that is not a brace group.

That is not an exception to section 4.4's rule so much as its last case: the sigil sits immediately to the left of the line ending it marks, exactly as it sits to the left of a brace group. A line ending is the one code point that cannot appear inside an inline range, so it cannot be reached any other way.

Markdown's other spelling — two or more spaces at the end of a line — is removed. Invisible syntax cannot satisfy prose-safety or the one-meaning property: a reader cannot see it, a diff barely shows it, editors and commit hooks strip it on save, and the widespread habit of typing two spaces after a full stop turns ordinary prose into structure whenever such a sentence happens to end a line. Nobody can rely on a mark they cannot see.

To write a literal backslash at the end of a line, use an escape range: `the drive root is C:\{\}`.

#### 4.4.4 Emphasis

`*{text}` is emphasis. `**{text}` is strong emphasis. `***{text}` is both.

Runs of one, two, and three asterisks are defined. A run of four or more is not a construct, so the run and the brace that follows it are text — the same shape as the six-level ceiling on headings.

There is **no paired form** and **no flanking condition anywhere in the language**. That is the point of the construct rather than a limitation of it: a flanking rule decides whether an asterisk is a mark by looking at what surrounds it, which is prose-safety by heuristic where invariant 1 promises prose-safety by construction. `5 * 3 = 15` is safe here because the form makes it safe, and `2*3*4` — which CommonMark emphasizes — is safe for the same reason.

Underscore is not emphasis syntax anywhere, which is what makes `snake_case` identifiers safe with no escape. Its one structural use is the subscript of section 4.4.8.

Emphasis inside a word needs no separate construct, because the braces already delimit: `un*{frigging}believable`. CommonMark needs seventeen interlocking rules and a delimiter stack for this section; `*{a **{b} c}` is unambiguous by balance alone.

#### 4.4.5 Verbatim spans

A verbatim span opens with a run of one or more backticks and closes with a run of the same length. Runs are maximal, in the sense given in section 4.3.3. Its content is verbatim: no escape is processed and no construct is recognized inside it.

The counting rule is the same sentence as at block size, and for the same reason: no escape reaches inside, so **choose a length that does not occur as a run in the content**. That is what "one or more" is for. Inline the choice may go down as well as up, a single-backtick span being free to hold a doubled run.

A decorator list (section 4.5) may follow the closing run immediately, in braces.

````
`x = y`             an unlabelled verbatim span
``a `b` c``         a doubled run, so the content may hold a single one
`a ``b`` c`         a single run, so the content may hold a doubled one
`x = y`{math}       a verbatim span labelled math
`x`{#snippet-3}     a verbatim span carrying an anchor
````

**A bare verbatim span, and a bare fence, mean verbatim plain text.** That is their definition, not a fallback from a missing label.

**This is the one construct delimited on both sides rather than marked on the left, and its own semantics require it.** Verbatim content is not interpreted, so a closing delimiter cannot be found by balancing — doing so would mean reading the content. Matching run lengths is the only escape that works without looking inside. That is why a verbatim span keeps the paired form and no other construct does.

#### 4.4.6 Links and images

A **link** is `@{target}`, or `@[text]{target}` when it carries its own text. A bare `@{target}` renders the target as its own text.

An **image** is `!{source}`, or `![alt]{source}` when it carries alternative text. Alternative text is optional and empty alternative text is well-formed; missing alternative text is a validator warning, never an error.

````
@{https://example.org}
@[the founding exhibit]{#founding-exhibit}
@[Tessier et al.]{papers/tessier-2026.markleft#abstract}
![the Alexandra Bridge at dusk]{assets/bridge-dusk.jpg}
````

The relationship between the two is worth stating, because it makes the exclamation mark memorable rather than arbitrary: **`@[…]` links to the target; `![…]` shows it instead.** The mark is a presentation switch, not a different construct, and the two forms differ by exactly one character.

**The target is literal.** It is never parsed as inline content, so a target holding `*` or `_` needs no escape.

**Cross-references need no new syntax, and that is the point.** `@[text]{#anchor}`, `@[text]{other.md#anchor}`, and `@[text]{https://example.org/page#anchor}` are one construct — a link whose target is a URL carrying a fragment. Internal and external references behave consistently because URL semantics already unified them.

**There are no reference links and no autolinks.** A target used repeatedly is written repeatedly; that is a repetition, not a missing feature, and the editor and the canonical formatter are where it is answered. What their absence buys is the removal of the only construct in the language whose meaning depended on text arbitrarily far away: under CommonMark's shortcut form, `[sic]`, `[1]`, and `[citation needed]` become links if a definition with a matching label exists anywhere in the same document, including one added later by someone who never saw the paragraph they changed. `@{https://example.org}` covers the autolink case exactly, so `<` carries no meaning either and `a < b` and `Vec<T>` are text.

**Two delimited parts, in reading order.** `@[text]{target}` puts what a reader sees in brackets and what it means in braces — the same order as `` `x=y`{math} ``, where the span is what you see and the label is what it means. These two forms are the only place a sigil's next code point is `[` rather than `{` — the stated extension of section 4.4's rule.

#### 4.4.7 Anchors

`#{identifier}` is an **anchor**. It is valid anywhere inline content is valid, and it marks a **point** in the document rather than decorating any particular construct. There is no attachment rule, so there is no question of what an anchor attaches to, no exclusion for paragraphs, and no adjacency subtlety. Mid-sentence anchoring is free.

Whitespace on either side of an anchor is optional and insignificant.

The rule that places an anchor well is one sentence: **an anchor marks where the reader arrives.** A link scrolls its target to the top of the reader's view, so an anchor at the end of a long paragraph delivers the reader past the thing they were sent to read.

````
#{five-minute}The complete core fits on one reference card.

## The five-minute property #{five-minute}
````

Leading for anything longer than a line, trailing on a heading. Neither is a special case; both follow from asking where the reader needs to be looking.

Two authors anchoring the same paragraph at different points is a difference in style, not an ambiguity: each document has one parse.

Identifiers are opaque. `fig-3` means no more to the language than `banana` does, which is what lets tools build figure numbering, section cross-references, and citation keys on naming conventions without the language arbitrating between them. Non-normative guidance for tool authors is in Appendix C.

Uniqueness is a validator check, not a parse rule. A repeated identifier is well-formed syntax and a broken anchor, and a document-wide constraint cannot be checked without abandoning the local, single-pass discipline invariant 4 buys.

#### 4.4.8 Superscript and subscript

`^{text}` raises its content. `_{text}` lowers it.

````
H_{2}O and CO_{2}          E=mc^{2}          2^{32} bytes          the 1^{st} of May
````

**A sigil carries meaning only when the very next code point is `{`.** That single condition is the whole of the prose-safety story, and it is worth spelling out what stays text as a result, with no escape needed:

````
2^32    [^0-9]    ISO_8601    UTF_8    file_2.txt    snake_case    a^b
````

The braces are what make the construct safe, so there is no braceless form. A one-character spelling would have to guess where the raised text ends, and a guess that ends it after one digit turns `2^32` into 2³2 — output that is wrong rather than merely limited, with nothing in the source to show it.

**What the construct does is a baseline offset and a size reduction, and nothing else.** No italics, no mathematical spacing, no interpretation of what is inside. This is the definition rather than a limit imposed on it, and it is what separates the construct from mathematics: `E=mc^{2}` is body text with a raised 2, while `` `E=mc^2`{math} `` is typeset mathematics with italic variables. They do not render the same, so they are not two ways of writing one thing.

The practical effect is that each case sorts itself. A variable set upright looks wrong to whoever wrote it, so `x^{n}` pushes its author toward `math`, where variables belong. Digits, units, ordinals and chemical formulae look right upright and stay in prose — chemistry being the case where upright is not an approximation but the correct typesetting.

The boundary, in one sentence: **superscript and subscript position content and interpret nothing; anything that needs interpreting is `math`.**

The content is ordinary inline content, so it nests and `x^{*{n}}` emphasizes — the construct never italicizes, and an author still may. An unclosed `^{` is text, under the uniform fallback of section 4.2. `^{}` is well-formed, renders nothing, and is a validator warning.

This rule was written for two characters before it was the shape of the whole section. Nothing about it changed when the rest of the inline layer adopted it; what changed is that it stopped being a special case.

#### 4.4.9 Brace groups: nesting and lengthening

A brace group opens with a sigil, then a run of one or more `{`, and closes on a run of **exactly the opening length** of `}`. Runs are maximal, in the sense given in section 4.3.3.

**A brace with no sigil before it is text, everywhere, always.** `{"key": 1}` and `${HOME}` need no escape.

**The content comes in two flavours, and this is the only distinction that reaches a parser.** *Inline content* — emphasis, superscript, subscript — is parsed recursively, so a nested construct is consumed whole and its braces never reach the outer close. *Literal content* — escape ranges, link and image targets, anchors, decorator lists — is not parsed, so the close is found by counting brace depth. Both are single-pass with no backtracking, and neither changes what an author types.

**Single braces work whenever the content is balanced.** `\{a {b} c}` closes at the final `}` with no lengthening at all.

**Lengthening is the escape hatch for the case that fails.** `*{the } character}` closes early; `*{{the } character}}` does not, because a lone `}` is a run of one and cannot close a run of two.

That gives the language exactly one escape mechanism, and it is the same sentence at every size: **when a delimiter appears in your content, lengthen the delimiter.** Verbatim fences, verbatim spans, and brace groups, identically. There is no second mechanism, and the escape range of section 4.4.2 is not one — it is a construct that happens to hold literal text, not a way of escaping a character.

**The costs, named rather than smoothed over.** Two of the openers occur as idioms in code that a writer may quote outside a verbatim span: `#{` is string interpolation in Ruby and Elixir (`"Hello #{name}"`), and `*{` is shell brace expansion after a glob (`ls *{png,jpg}`). Both are answered by a verbatim span, which is where such text belongs; the rest of the openers have no idiom behind them at all.

### 4.5 Decorators

A **decorator list** attaches a small, fixed amount of information to a verbatim construct. It is written in two positions and means the same in both: on a fence, immediately after the opening backticks; inline, in braces immediately after the closing backtick run.

A decorator list is a space-separated sequence of tokens in exactly two shapes, **each at most one**:

- a **bare word** — the format label, a single token drawn from the character set given below (`rust`, `objective-c`, `x-mermaid`);
- **`#identifier`** — an anchor.

There is nothing else. No key-value pairs, no classes, no format sigils, no colons, no nesting. Order within the list is free; the canonical formatter normalizes it.

````
```math                `x=y`{math}                 labelled
```rust                `let x = 1`{rust}           labelled
```math #eq1           `x=y`{math #eq1}            label and anchor
```#snippet-3          `x`{#snippet-3}             anchor only
```                    `x`                         unlabelled verbatim
````

**The braces belong to the list, not to any token.** They are its delimiter, and they appear only where one is needed: inline, where the sentence continues after the list, and not on a fence, where the rest of the line is the list and nothing follows it. So the same `#identifier` token is written bare on a fence and bare inside an inline list, while a standalone anchor in running prose is `#{identifier}` — where the sigil comes first and the braces are that construct's own delimiter (section 4.4.7). Braces on a fence's list would nest inside the inline one, `` `x`{rust {#id}} ``, which is why the fence form carries none.

**At most one bare word**, because two format words would claim two content types for one box. There is no meaning to combine them into and no resolution order to appeal to, so a second word buys a race rather than a capability.

**At most one anchor**, matching HTML, where an element carries a single identifier.

**No word is ever reserved.** This document defines the grammar of a decorator list and the meaning of the `#` sigil. It does not define what any bare word means. A label is opaque, is carried into the tree exactly as written, and **cannot affect the parse**. The grammar is therefore fixed independently of any word list — no future word can quietly become a directive, because words have no parse-level power by construction, and there is no registry to maintain, version, or argue about.

Meanings live outside this document. That `math` means TeX and `rust` means Rust is convention, recorded in the non-normative Appendix C and in a validator's list of recognized words, both revisable without touching the language. An unrecognized word parses as plain verbatim content and earns a validator warning, never a parse error. Prefixing `x-` is the usual guard against collisions; nothing checks it, because it is a convention rather than syntax.

There is consequently **no `text` label**. Nothing rules it out — ruling a word out would reserve it — but nothing blesses it either: nothing here assigns it a meaning, a validator reports it as redundant, and the canonical formatter removes it.

A labelled span and an unlabelled span are **different nodes**. They may render identically; the tree records which the author wrote.

**Token character set.** A decorator token is a run of Unicode characters excluding whitespace (which separates tokens), control characters (nothing invisible is ever syntax), `{` and `}` (which would close the brace group), and the backtick (which would put an unpaired run inside a construct delimited by counting runs). Nothing else is excluded: letters in any script, digits, `(`, `)`, `:`, `-`, `.`, and `_` are all writable. The list is stated by exclusion, and it has four entries, because a short exclusion list can be audited where an inclusion list is a standing argument.

The backtick exclusion earns its place twice. On a fence the decorator list runs to the end of the line, so without it ```` ```rust``` ```` would be an opening fence whose label is `` rust``` `` — a line that reads as a closed empty fence and is not one. Inline, a backtick inside a decorator would be a candidate opener for the next span in the paragraph, making the pairing of later runs depend on text inside braces. Nothing is lost: no format word and no identifier wants a backtick, and CommonMark excludes it from a backtick fence's info string for the same reason.

What such a line is *instead* is a question this exclusion raises and does not answer — whether a malformed decorator list makes the whole line paragraph text, under the uniform fallback of section 4.2, or leaves a well-formed fence carrying a validator error. It is the general case and it applies equally to two bare words in one list. See Appendix D.11.

**The first character selects the shape:** `#` makes the token an anchor, anything else makes it the format word. Sigils are significant only in first position, so `#` inside a token is an ordinary character.

**This is the one place in the language where a mark follows what it marks.** A decorator annotates the span before it, so the arrow points backwards even though the brace group is still opened by the code point to its left — the closing backtick run. No other construct opens on a backtick, so nothing collides, and the rule needs no clause of its own. The asymmetry is deliberate: a fence already delimits, so there is nothing for a brace to scope, and a decorator is annotation rather than marking.

**Legibility.** `` `x=y`{math #eq-euler} `` is the ceiling of this grammar rather than its normal use, and a decorator can grow longer than the content it decorates. The grammar has room for it anyway, because closing off the full form inline only would be an "unless" clause. House style carries that cost instead: anchors belong on fences, where a block has room for them, and an inline decorator normally carries the format word alone. What keeps even the ceiling inside invariant 1 is the postfix order — the content reads first, and the decoration is a suffix on a span the reader has already finished.

### 4.6 Mathematics

Mathematics is part of the core. That claim is precise, and it is exactly three guarantees:

1. **The dollar sign is never mathematical syntax, anywhere.** A bare dollar is always literal text. Money amounts, shell variables, and regular-expression anchors are safe with no escape.

2. **A first-class construct exists at both sizes — fenced and inline — whose content is verbatim** and therefore cannot be reinterpreted by anything downstream. TeX backslashes survive intact.

3. **The label reaches the tree unchanged.**

````
A coffee costs $4.75 and a refill costs $1.50. Nothing here is mathematics.

The identity `e^{i\pi} + 1 = 0`{math} sits inline.

```math
\int_{0}^{\infty} e^{-x^2}\,dx = \frac{\sqrt{\pi}}{2}
```
````

Those three guarantees are exactly what the founding exhibit needed. What failed in section 1.2 was that dollars paired across a paragraph and that code spans did not protect their content — not that the word "mathematics" lacked a definition.

So `math` is a **conventional label, not a reserved word**, and this document says nothing about how mathematics is typeset. That is a smaller claim than "mathematics is core" usually implies, and it is the whole of what a *language* can honestly promise: a renderer still chooses the typesetting, as it always did, but it can no longer change what the source says.

**Superscript and subscript are not part of this.** Section 4.4.8 offers `^{…}` and `_{…}` for running prose, and they interpret nothing — a raised or lowered run of ordinary text, set in body type. A simple exponent therefore has two homes and they render differently, which is what keeps them distinct: `E=mc^{2}` is prose with a raised 2, and `` `E=mc^2`{math} `` is an equation. Anything with variables, operators, or structure belongs in the second.

### 4.7 What the language does not contain

Stated explicitly, so that no reader has to infer an absence:

- no raw passthrough of any kind — no raw block, no raw span, no format escape hatch, and no HTML;
- no generated content — no table-of-contents directive, no automatic numbering, no index, no textual transclusion, no variable substitution;
- no underlined (setext) headings, and no heading closing sequences;
- no indented code blocks;
- no lazy list continuation, no alternative list markers, and no alphabetic or Roman numbering;
- no trailing-space hard breaks;
- no thematic breaks;
- no underscore emphasis, no paired emphasis delimiters, and no flanking rules anywhere;
- no reference links, no link reference definitions, and no autolinks;
- no braceless superscript or subscript, and no braceless anchor, link, image, or escape;
- no single-character backslash escape;
- no smart punctuation;
- no classes, no key-value attributes, and no column width hints;
- no reserved words.

Because there is no raw passthrough, **a Markleft document is inert by construction.** It cannot execute anything and cannot embed anything, ever. No Markdown flavour can make that claim.

### 4.8 File names

Exactly two extensions: **`.markleft`**, the long form, canonical in documentation; and **`.lf`**, the short form. A language with one meaning per input does not need four names for its files. `.lft` is held in reserve as a fallback rather than offered as a live option.

No media type is registered. That is an open item, listed in Appendix D.

## 5. The binding decisions

This section is informative. It records what was decided and why, including the alternatives it set aside, so that a reader can evaluate the language rather than merely learn it, and so that a later contributor finds the analysis instead of rebuilding it.

**The numbering is stable and citable.** Decisions are referred to by number throughout this project — in this document, in the deltas appendix, in commit messages, and in design memos. Numbers are never reused and never renumbered.

### Decision 1 — The dollar sign is never syntax; mathematics is decorated verbatim content

**Rule:** a bare dollar sign is always literal text. Display mathematics is a fence labelled `math`; inline mathematics is a verbatim span decorated `{math}`. Section 4.6 states the three guarantees this buys.

**Why:** section 1.2. Dollar-delimited mathematics was never in any Markdown specification, and every implementation of it is a heuristic operating outside the parser. The goal here was never to specify a mathematics engine. It was unequivocalness — to end the dollar mess — and the language achieves that by guaranteeing what the source unambiguously *is* while saying nothing about what a renderer does with it.

**Not adopted — `\(…\)` for inline mathematics.** This was the original design and it is withdrawn, for two reasons. It cannot coexist with decision 3, which yields the character after any backslash with no exception list, so `\(` is simply a literal parenthesis and those delimiters cannot exist. The deeper failure is in the content rather than the delimiters: inline TeX is dense with backslashes, so `\(\frac{a}{b}\)` loses `\f` to an escape and offers no way to tell a closing `\)` from an escaped parenthesis. **Inline mathematics has to be verbatim**, which is why the fenced form was never in trouble and why the inline form belongs in the same machinery.

**Not adopted — carving `\(` and `\)` out of decision 3.** It costs the "no exception list" guarantee *and* does not fix the content problem, so it buys nothing.

### Decision 2 — Emphasis is braced, and underscore is not emphasis at all

**Rule:** `*{em}`, `**{strong}`, `***{both}`. There is no paired form and no flanking condition anywhere in the language.

**Why underscore went:** `snake_case` is ordinary technical prose, and CommonMark's answer to it is seventeen interlocking emphasis rules and a delimiter stack — a construct that cannot be expressed as a context-free grammar. Removing underscore makes the identifier safe with no escape.

**Why the paired form went, which is the larger of the two:** flanking rules decide whether an asterisk is a mark or a character by looking at what surrounds it. That is prose-safety by *heuristic*, and invariant 1 promises prose-safety by *construction*. Emphasis was the last construct in the language where those two came apart — `5 * 3 = 15` was safe by a rule rather than by its form, and `2*3*4` was not safe at all. The braced form closes the gap, and with it invariant 1 holds with no residue anywhere in the language.

**What comes with it, none of it the point but all of it real:** CommonMark's hardest algorithm disappears, since the delimiter stack exists only to resolve emphasis; `***{…}` falls out of the run rule instead of needing the nesting corner nobody predicts; and intra-word emphasis needs no separate construct, since the braces already delimit.

**The typing cost, counted:** `*em*` → `*{em}` is one character more, `**em**` → `**{em}` is free, and `***em***` → `***{em}` is one character fewer.

**Not adopted — keeping `*em*` alongside `*{em}`.** The paired form reads better in isolation and is what fingers know. Two spellings of one construct is what decisions 4, 5, 6, and 13 each removed, and this one would have kept the heuristic alive in order to serve the spelling that needs it.

**Scope.** Underscore keeps the one position decision 18 gave it, immediately before `{`, which leaves every identifier alone.

### Decision 3 — Escaping is a range, not a character

**Rule:** `\{…}` makes its content literal. A backslash followed by anything other than `{` is an ordinary backslash. The one operand that is not a brace group is a line ending, which is decision 13.

**Why:** the previous rule — a backslash before *any* character yields that character, with no exception list — read perfectly and was the only rule in the language that made ordinary writing worse than CommonMark makes it. CommonMark escapes ASCII punctuation only, so `\U` survives there; under uniform escaping `C:\Users\frederic` rendered as `C:Usersfrederic`, and `\n` and `\d+` in running prose went the same way. Text silently altered, with nothing on the page to show it, is the failure class invariant 1 exists to prevent, and this was the only place in the language that produced it.

**What the range buys beyond the repair.** It joins the backslash to every other sigil rather than leaving it as the one character that works differently. It is *visible* — `a \{|} b` says "I meant this pipe", where a lone `\|` reads as a typo. And the need for it collapsed: under the rule in section 4.4, ordinary writing produces no operative sequences at all, so the heavier escape is reached for far less often than the lighter one was.

**Literal braces need no backslash.** Content balances, so `\{a {b} c}` closes at the final `}`; genuinely unbalanced content lengthens the run, `\{{…}}`, which is the mechanism fences and verbatim spans already use.

### Decision 4 — ATX headings only, in one closed form

**Rule:** section 4.3.2.

**Why:** one sentence with a uniform fallback disposes of every case that Markdown legislates separately — seven number signs, hashtags, numbered items, the lone number sign, indentation, and closing runs — with no exception clause and no new error class. Underlined headings additionally violate invariant 4, because an underlined heading is a heading only on account of the *next* line.

**Not adopted — keeping the optional closing run.** CommonMark's rule for when a trailing run of number signs closes a heading and when it does not (`# foo #` closes, `# foo #bar` does not) is the clearest available specimen of the clause invariant 2 was written against.

*This decision, with 5 and 6, carries the highest muscle-memory risk in the language. See section 5's closing note.*

### Decision 5 — Fenced verbatim blocks only

**Rule:** indented code blocks are removed. Four leading spaces are four leading spaces.

**Why:** indented code is invisible in the source, interacts badly with list indentation, and forces every list rule to reason about whether an indented line is content or code. Removing it simplifies decision 6 substantially, and it retires CommonMark's three-space heading allowance as a side effect (decision 4).

### Decision 6 — Lists with arithmetic, not archaeology

**Rule:** section 4.3.5 — one bullet marker, one ordered marker, content-column alignment, no lazy continuation, adjacent items form one list, and ordered lists renumber.

**Why:** list indentation is where Markdown implementations disagree most, and nearly all of that disagreement traces to lazy continuation and to the interaction with indented code. Both are gone.

**Why one marker of each kind.** `*` and `+` existed as bullet markers for a single purpose: switching marker was how Markdown separated two touching lists. Remove them and that trick has nothing left to do, so the rule that governed it — "one marker per list" — leaves the language rather than being reworded. A rule is deleted from the reference card, not rewritten.

What that buys is measurable rather than asserted. **`+` becomes free everywhere**, having no other job in the grammar, which puts it beside the dollar sign; it matters in ordinary prose because a diff pasted outside a fence carries a `+` on every added line. **`*` loses one of its three jobs**, and with it the precedence question between a thematic break and a bullet item whose content is `**`. Decision 19 later removed the thematic break too, so `*` no longer claims a line's first column at all. **`1)` freed nothing at the time** — a closing parenthesis still ended a link destination — and went purely for symmetry: one bullet marker and one ordered marker state the whole of list syntax in two lines. Decision 20 later freed the parenthesis outright, so the symmetry argument was paid back with interest.

This is the same subtraction as decisions 4, 5, and 13. Setext headings, closing runs, indented code, and trailing-space breaks were each a second spelling of one construct. Three spellings of a bullet was the largest instance still standing.

**Not adopted — alphabetic and Roman markers** (`a.`, `A.`, `i.`). They fail the addition bar in section 2.2 on its second half, because they cost plain-text clarity. `A. Smith argues that…` and `J. R. R. Tolkien wrote…` are initials and author names, ordinary in citations and bibliographies, so letters put a frequent collision at the start of a line — a form that occurs naturally in writing, promoted to structure, which is the dollar-sign failure in another costume.

They also arrive with three questions, one of which needs an "unless": `i.` is both the ninth letter and Roman numeral one, so admitting letters forces a choice that surprises somebody either way; nothing says what follows `z.`, since `aa.` is a numbering scheme rather than a marker; and with one marker per list gone, no rule says whether `a.` followed by `B.` is one list or two.

The deciding objection is a fourth. Under renumbering, a marker is a *presentation of position*, so choosing between `1.`, `a.`, and `A.` is choosing a numbering **style** — a list-style property written into the source, which is the nearest thing this language would have to a class, and decision 9 removed classes for being the only construct whose entire purpose was to let rendering vary. The want behind the proposal is real, since legal and academic outlines do letter their sub-items, and it gets the answer everything else got: nesting already distinguishes levels, and an author who needs `(a)` writes it as text, which the plain-text reader sees exactly as the rendered reader does.

**On renumbering:** the argument that this is not generated content is given in full in section 4.3.5. The point that decides it is that manufacturing content means *relocating* or *importing* it, and deriving a presentation of the position an item already has does neither. That distinction is useful beyond lists: it is the same line that admits images and excludes file includes.

### Decision 7 — No raw passthrough, at all

**Rule:** Markleft has no way to emit content in a target format. There is no raw block, no raw span, and no format escape hatch.

**Why:** invariant 5. An `iframe`, a `script`, or an `object` element renders content that is not in the source — transclusion under another name. Any document containing one falsifies "you can read the whole document in plain text" by exactly the mechanism the cardinal rule exists to exclude.

**The hole cannot be patched by rule.** A line break element adds nothing and an inline frame adds everything, and separating them requires knowing HTML semantics — an unbounded blocklist and an "unless" clause at once.

**What removal buys:** a document that is inert by construction; no lock-in to a target format, where a raw HTML block had put HTML *inside* a language claiming independence from any serialization; and a simpler decorator grammar, since the format-escape token shape leaves with it.

**What it costs, stated rather than hidden:** an escape hatch is what lets a small language answer a feature request without growing. "Markleft cannot do X" is answerable with "drop to a raw block" only while one exists. Without it, every gap presses directly on the core, which is the force that has broken the five-minute property everywhere else. Collapsible sections on GitHub are the concrete casualty.

**Not adopted — keeping the raw block as a loudly-marked exception**, where the cardinal rule holds *except* inside a fence that warns the reader the plain text is not the whole story. Honest, but still an "unless" clause on the cardinal rule, and the rule is worth more intact.

### Decision 8 — Pipe tables in the core, strictly

**Rule:** section 4.3.6.

**Why:** tables are the one construct that every Markdown platform added and none specified compatibly. Putting them in the language with an exact grammar removes a whole class of platform divergence. The strictness is the point: a table that does not match is prose, not a guess.

**The compatibility budget is different here, and it is worth saying so.** Pipe tables are not in CommonMark — they are a GitHub extension — so decision 12 does not protect them. Keeping the familiar spelling is a choice made for authors' hands, not an obligation, and that is what left room to fix the three things GFM tables cannot do.

**Why the pipe opens the table.** It removes the block layer's last lookahead. Under the GFM form a header row is indistinguishable from a paragraph until the rule row arrives on the following line, which made a table the only block whose type was not decidable from its own first line — precisely the property decision 4 removed setext headings for. The alternative on the table was to narrow invariant 4 to a uniform one-line confirmation window, which would have cost decision 4 one of its two arguments. Opening on the pipe removes the conflict at its source, so the invariant stands unchanged. It also makes the leading pipe mandatory by construction, which closes a prose-safety hole: with the leading pipe optional, `a | b` in ordinary prose is a valid header row.

**Why block cells.** Not line length, though that is the visible symptom. A table row *is* a line, so soft wrapping destroys the column alignment that was the format's whole reason for existing, hard wrapping is not available because the row cannot be broken, and changing one word in one cell shows the entire row as changed. This is the fourth application of one principle in this project — after unwrapped prose, `1.` numerals, and named rather than numbered anchors: **keep the unit of the source close to the unit of the edit.** A Markdown table is the only one of the four where the author has no good spelling available at all.

**Not adopted — width hints in the rule row** (`|70:---|`). Not on the grounds first offered, three of which were wrong and are withdrawn in the decision record: a proportion *can* be shown in plain text, because the canonical formatter already chooses source column widths; a proportion is medium-independent in a way that page-layout measures are not; and a stated proportion is in fact more consistent across renderers than automatic sizing, not less. What decides it is that a width is a hand-maintained number that goes stale silently as content grows — the same shape set aside for list numerals and numbered anchors — and that a number in the source would actively work against whatever sizing a renderer or formatter later does well. Alignment survives the same test on a real distinction: it is closed, three-valued, and describes the content, where width is open, continuous, and describes only the display, and it is the continuity that invites the endless tuning that decision 9 removed classes to avoid. Full analysis, and what would reopen it, in `notes/table-width-hints.md`.

**Not adopted — a closing rule row**, and **a blank line**, and **two blank lines**, as the table's terminator. A closing rule row strains when inline and block cells mix in one table and carries the load of remembering to write one. A blank line cannot work, since a blank line is what separates the paragraphs a block cell exists to hold. Two blank lines would work and contradicts decision 6, where blank lines never split a list.

**The cost, stated:** a list item marks every line it owns, and a block cell does not. Two solutions to one problem is a real charge against invariant 2, accepted because the pipe already does for a cell what indentation does for a list item, and because a table is a distinct editing mode in an author's mental model however it is spelled.

### Decision 9 — Decorators: one vocabulary, two positions, no reserved words

**Rule:** section 4.5.

**Why:** the language needs a way to say "this verbatim content is Rust" and "the reader arrives here", and it needs exactly one such way rather than one per construct. Two token shapes, each at most one, is the smallest grammar that does both jobs. That no word is reserved is the stronger half of the decision: it means no future word can become a directive by accident, because words have no parse-level power at all.

**Not adopted — `.class`.** Classes were in the design and were removed. Four counts against them. They fail the design test in decision 15: a platform that sanitizes CSS strips the class attribute, so the mark is consumed and nothing replaces it, which is exactly the information destruction that decision rules out. They pay for nothing under the addition bar in section 2.2: no ambiguity removed, and the only gap filled is "Markdown has no styling", which is the richer-Markdown trap. They balloon the source, so the cost falls on invariant 1. And they invite mixing styling with content — the thing many people came to Markdown to escape. Being freed from styling decisions is a feature, not a deprivation, and Markdown's lack of styling is part of why its text pastes anywhere.

**Not adopted — key-value attributes.** Arbitrary attributes only ever existed to feed arbitrary HTML, so decisions 7 and 9 removed each other's reason to exist.

**Not adopted — a `text` label.** Blessing a second spelling of "verbatim plain text" costs the one-way-to-do-it discipline for nothing, and naming the label invites peppering prose with monospace for emphasis.

### Decision 10 — Tabs are not structural

**Rule:** spaces determine structure. A tab in a structural position is a validator warning with an automatic fix.

**Why:** a tab's width is a display setting, so structure that depends on it is structure that depends on the reader's editor. The same family as decision 13: whitespace does not carry meaning here.

### Decision 11 — Markdown's block layer is kept; its inline layer is replaced

**Rule:** no smart punctuation in the core. Markdown's muscle memory is kept wherever Markdown was not broken, and the line falls almost exactly on the block/inline boundary.

**Why:** djot (MacFarlane, 2022) is the nearest prior art and most of this language's syntax decisions are djot-vetted. Where this language differs, it differs deliberately. Smart punctuation modifies what the author wrote, which decision 14's no-normalization clause leaves out for the same reason.

**Where the line falls.** Headings, bullets, ordered items, block quotes, fences, and tables keep Markdown's shape — tightened by decisions 4, 5, 6, and 8, but recognizable, and a Markdown writer's fingers are right about all six. The inline layer is where CommonMark's ambiguity actually lives: flanking classification and the rule of 3 for emphasis, nested brackets and balanced parentheses for links, and a shortcut reference form that reaches across a whole document. Decision 20 replaces it.

**The clause was tested against emphasis and links, and they came out opposite ways.** The link is the construct this decision was written to protect, and decision 12 explains why it is not protected here. Emphasis was never protected, because it is arguably the most broken construct CommonMark has, and this decision exempts broken constructs by its own terms.

**Challenged and kept — backtick verbatim.** The backtick is genuinely hostile to non-US keyboards: it is a dead key on QWERTZ, AZERTY, Nordic, and Spanish layouts, it is visually close to the apostrophe and the acute accent, and mobile keyboards bury it. The proposed replacement was the pipe. The pipe fails twice before compatibility is even considered: it is already the table delimiter, and it breaks prose-safety, since `|x| < 5`, `P(A|B)`, and shell pipelines are ordinary prose.

No substitute exists, and the reason is structural and worth keeping: **the backtick's inaccessibility and its prose-safety are the same property.** It is hard to reach because almost nobody types it in ordinary writing, and almost nobody typing it in ordinary writing is exactly what qualifies it as a delimiter. Any character reachable enough to fix the complaint is common enough to re-create the collision. The concern is answered with tooling instead — editor snippets, a line on the reference card about non-US layouts, and a validator diagnostic that catches a decorator following something that is not a verbatim span and asks whether an apostrophe or an acute accent was typed for a backtick.

What would reopen this is a candidate character, not another argument.

### Decision 12 — Markleft is not Markdown-compatible, by construction and on purpose

**Rule:** a Markleft document containing a link or emphasis is not valid CommonMark and does not render as CommonMark. Appendix A maps the two languages onto each other; it does not promise that a document ports.

**Why this is not a reversal.** The rule this replaces said to match CommonMark *wherever that is unambiguous and harmless*, and the inline constructs decision 20 replaced are neither: emphasis needs a delimiter stack, links need balanced-parenthesis rules and nested-bracket rules, and the shortcut reference form reaches across a whole document. The clean break is what the compatibility principle licensed rather than a departure from it. The principle is spent, not abandoned.

**Why the break is taken deliberately rather than absorbed.** Near-compatibility is what produced Markdown's flavour drift. A writer cannot answer whether their document survives a move between platforms, because every implementation is *almost* the same as every other and the differences are invisible until they bite — the founding exhibit of section 1.2, at ecosystem scale. A clean break makes the question answerable. A vendor can support Markdown and Markleft side by side; what stops paying is the temptation to stretch one implementation across both.

**The file extensions are the instrument.** `.markleft` and `.lf` (section 4.8) are what keeps the two languages from blending, and they were decided before this rule needed them.

**What remains true of the block layer is an observation, not a constraint.** Headings, fences, lists, quotes, and tables are Markdown's because Markdown's were right, not because compatibility asked for them.

**The cost, stated plainly.** A Markdown user's fingers are wrong about links, images, emphasis, and escapes on the first day. A migrator is buildable and remains a Phase 3 deliverable, since the rewrites are mechanical; what it cannot promise is that its output means what the input meant on any particular platform, which was never true of any two Markdown implementations either.

### Decision 13 — Hard breaks are explicit; trailing whitespace is never significant

**Rule:** section 4.4.3.

**Why:** invisible syntax cannot satisfy invariant 1 or invariant 3. The deciding argument is not aesthetic: typing two spaces after a full stop is a live habit forty years after typewriters, so an ordinary sentence that happens to end a line silently becomes structure. Everything else compounds it — editors and commit hooks strip end-of-line whitespace to keep diffs clean, so the syntax is not merely invisible but actively unstable.

**Why the backslash form survived decision 20 unchanged**, when every other use of the backslash was rewritten. It is the most cardinal-rule-compliant construct in the language: the source line ends exactly where the output line ends, and nothing else here is isomorphic to its own rendering. Its one realistic collision *renders correctly* — a shell continuation quoted in unfenced prose, `gcc -o foo \`, becomes a hard break, which is the line break the author already wanted, and no other collision in the language has a benign failure mode. And it needs no new machinery to escape: `the drive root is C:\{\}`.

**Not adopted — `\\`, matching TeX.** A trailing `\\` is exactly as invisible as a trailing `\`, so it buys nothing on intentionality, and it puts UNC paths and escaped backslashes in quoted code into the blast radius. **Not adopted — `\n`**, which would reintroduce the exception list decision 3 was rewritten to remove.

### Decision 14 — Unicode is the character set; UTF-8 is the encoding

**Rule:** section 4.1.

**Why UTF-8, in a specification written in 2026:** its age is the argument in its favour, and four independent lines of evidence agree. The WHATWG Encoding Standard requires it for new formats exclusively, and its encoding table is closed to additions. RFC 3629 froze UTF-8's definition permanently, so it cannot grow new cases. Deployment is roughly 99% of the web. And there is no successor and no candidate: the historical succession was UTF-1 to UTF-8, and it stopped.

**Not adopted — UTF-16**, which reintroduces every problem UTF-8 removes: endianness, surrogate pairs that break naive indexing, no ASCII compatibility, and loss of self-synchronization. Its footprint is internal string APIs, none of which is an interchange format. **Not adopted — UTF-32**, which costs four bytes per character to buy a fixed-width property that is a fiction, since combining marks mean one code point was never one user-perceived character.

**The durability risk is the Unicode version, not the encoding.** UTF-8 is frozen; Unicode releases annually. So the risk attaches only to rules that consult a Unicode table, and "content is never normalized" already immunizes the bulk of the language. The two rules that would have consulted a table — how anchors and labels match, and what "column" means — are defined version-independently in section 4.1, by code-point identity and code-point count. A pinned Unicode version is a dependency that ages, and it puts this definition on somebody else's release schedule.

### Decision 15 — No generated content

**Rule:** everything a reader will see is written in the source, in the order it appears there. A renderer styles, links, and lays out what is written, and never supplies text that is not.

This is invariant 5's operative detail. The rest of this entry is the part a reader needs in order to apply it.

**The boundary, precisely.** The rule is *not* that the source must look like the rendered form: `**bold**` is not bold, and a TeX fraction is not a typeset fraction. It is that the source must be **semantically equivalent to the rendered form, up to format.** Rendering may change how written content appears; it may never create content.

So the test for any future construct is one question: **is this changing the presentation of written content, or manufacturing content?** Presentation instructions stay open in principle — a decorator is one. Content-manufacturing constructs are closed.

Reference links passed the test — the visible text is written where it appears, and a destination is metadata rather than content — and were removed anyway, by decisions 17 and 20, on prose-safety grounds: passing the generated-content test does not exempt a construct from the other invariants. A table of contents fails the test itself, because its entries appear where nobody wrote them.

**The back half: rendering is lossy on marks and lossless on content.** The front half says rendering adds nothing. The converse is that rendering *removes*: `` `E=mc^2`{math} `` becomes a typeset equation and the word `math` is gone from the page. Nothing is lost by that. The statement "this is an equation" was not deleted but **re-expressed as typography**, exactly as `**bold**` surrenders its asterisks to boldness. A decorator is the plain-text stand-in for a visual distinction the plain-text reader cannot receive, which is why these are two halves of one rule rather than two rules.

The constraint on this direction is narrow and absolute: **only marks are consumed, and every unit of content survives.** A renderer that dropped the content a mark identified would break the rule as surely as one that invented content.

**The design test that follows:** every format word names something a conforming renderer visibly distinguishes. A format whose rendered output is indistinguishable from undecorated content destroys information on render, and does not clear the test. Anchors are exempt by kind rather than by exception: an anchor is navigation, metadata in the same category as a link destination, and metadata carries no content to lose.

**Worked consequence — citations and bibliographies.** Both generate text, so both are excluded from the language, and the capability moves to authoring time rather than being lost. A preprocessor or editor extension reads a bibliography file and writes the citation text and the reference list *into* the source, where they become plain text like everything else: `As @[Smith (2020)]{#ref-smith2020} showed…` above a references section whose entries carry `#{ref-smith2020}` anchors. The anchor is the citation key, serving as the reader's navigation target and the tool's stable identifier at once.

This is better than the alternative on the rule's own terms, because the plain-text reader sees "Smith (2020)" rather than a citation key. **The cost is restyling:** changing citation style is a flag in a citation processor and a full re-run with a large diff here. Immaterial for a document with one venue; a genuine loss for a paper shopped across journals. The trade is diffable-and-readable always, against cheap-restyling sometimes, and this project takes the first consistently.

**Worked consequence — file includes stay out of the language.** Could an image-like construct include a text file in place? No, and the rule decides it with no amendment, which is the test of an invariant rather than a carve-out. An include inserts text the source does not contain, so a plain-text reader has not read the document. The legitimate want behind it — keeping a code sample in sync with the file it came from — gets the same answer tables of contents got: a tool writes the snippet into the source. An include directive hides drift; an inlined snippet shows it in the diff.

### Decision 16 — Images are core, and the line is textual

**Rule:** `![alt]{src}`, or `!{src}` with no alternative text. Alternative text is optional; empty alternative text is well-formed and missing alternative text is a validator warning.

**Why an image is legal where raw HTML is not:** decision 15 governs the document's *text*, and an image contributes none. A reader of the plain source has read every word of the document.

**The bend, stated honestly.** What that reader lacks — what the picture shows — is real content, and no alternative text recovers it, because an image is worth a thousand words and an alt attribute is sized for a screen reader. So the cardinal rule *is* bent here, deliberately and in one bounded place, for two reasons that do not generalize: images are essential to clarity in most real documents, and unlike text there is no way to carry one in plain text at all. **A rule bent knowingly, at a named place, with the reason written down, is not the same thing as a rule with a hole in it** — which is why the next case is read on its own terms rather than inheriting this one.

An inline frame imports arbitrary text that becomes part of what the reader reads. An image is bounded and inert, occupies one non-textual slot, runs no scripts, and cannot embed recursively. That is the whole distinction, and it is why "no transclusion" in decision 15 means no *textual* transclusion.

**Not adopted — `` `path`{image} ``,** despite fitting the decorator notation exactly. A decorator must mean the same thing in both positions, and an `image` fence is meaningless: a word that works only inline is a special case wearing the notation. Worse, a decorator labels content that is *present*, where a path references content that is *absent*. And it has nowhere to put alternative text.

**Not adopted — `[alt](src){image}`,** proposed as the strongest alternative when this decision was first written, on the principle that `![]()` was then the only inline construct whose type marker preceded its content. Decision 20 resolved that tension in the opposite direction and the proposal is moot twice over: every inline construct is now marked by a prefix, so the image is no longer the odd one out, and a decorator naming a construct rather than labelling content is the shape decision 15 leaves out — the same objection that sank `` `path`{image} ``.

**The mnemonic survived the form change and got cheaper.** `@[…]` links to the target, `![…]` shows it instead. The two constructs now differ by exactly one character, where before they differed by a bracket-and-parenthesis dance.

### Decision 17 — Anchors are positional; references are ordinary links

**Rule:** section 4.4.7.

**Why positional:** making an anchor mark a point rather than decorate a construct removes an entire class of question — what it attaches to, whether paragraphs are excluded, what happens when two constructs are adjacent — without adding anything.

**Markleft's contribution to cross-referencing is the removal of a heuristic, not the addition of a feature.** Platforms derive anchors by slugifying heading text, and every platform slugifies differently, so the same link resolves on one and dangles on another. That is the founding exhibit in navigation form. Explicit anchors delete the guess, and cost no new construct at all.

**Reference links, link reference definitions, and autolinks are removed**, the shape of the removal being decision 20's and the reason this decision's: the shortcut form was the only construct in either language whose meaning depended on text arbitrarily far away, so `[sic]` became a link whenever a definition with a matching label existed anywhere in the same document, and it failed silently when it fired. A target used twice is written twice — a repetition, answered by the editor and the canonical formatter, not by syntax. `@{target}` covers the autolink case exactly, which is what freed `<` (section 4.4.6).

**Not adopted — a bare reference sigil,** in every spelling considered. Two independent failures, and only one of them survives. It would have to **generate its own text**: rendering a bare reference means emitting either the useless raw identifier or the target's title or number, which is text pulled from elsewhere and inserted where nobody wrote it — decision 15, arriving through a smaller door than a table of contents but through the same door. That objection is untouched by anything since. The second was that a bare `#identifier` is prose-unsafe, since `#general` and `#nowplaying` are ordinary technical prose, and a bare hash-word would be text at the start of a line and syntax in the middle of one. **Decision 20 answered the second and left the first standing**, which is exactly why `@{#five-minute}` exists and a bare sigil does not: the brace makes the sigil prose-safe, and a link with no text renders its target rather than inventing one. No digit-guard rule is needed anywhere as a consequence: `#1` in prose is never syntax.

**A closed gap worth noting.** Decision 16 left open whether decorators needed a third position so that a heading could carry a custom identifier. This decision closes it without a new position: an anchor is valid anywhere inline content is, and a heading's text is inline content, so `## The five-minute property #{five-minute}` already works.

**The validator stays a single-document tool.** Duplicate identifiers and dangling same-document fragments are lint. A destination carrying a path or a host is a URL, and Markleft does not validate URLs — an external link can fail and is not checked either, so a cross-document fragment gets the same treatment. This keeps the tooling from needing to know about a set of documents, which would be its first multi-file commitment.

### Decision 18 — Superscript and subscript are core, braced, and typographic only

**Rule:** section 4.4.8.

**Why it is worth a construct at all:** a simple exponent or subscript is painful in Markdown, where the only route is raw HTML, which decision 7 removes outright — and it is current in exactly the scientific and technical writing this language is aimed at. It reads correctly in plain text with no compiler, which satisfies invariant 5 rather than merely surviving it: `H_{2}O` and `E=mc^{2}` are how people have written these in plain-text mail for decades. And it adds nothing to what unmarked prose has to survive, which is the bar in section 2.2 that any addition clears before its usefulness is weighed at all.

**Not adopted — the braceless form,** `^2` and `_2`, capped at a single digit, which is where the proposal started. It fails twice and either failure is enough. Underscore before a digit turns `UTF_8` into UTF₈, `ISO_8601` into ISO₈601, `SHA_256` into SHA₂56, and `file_2.txt` into file₂.txt; the decisive part is not the list but that **`H_2O` and `ISO_8601` have the same local shape** — letter, underscore, digit, alphanumeric — so no condition on the surrounding characters separates them the way a heading's required space separates `#` from `#hashtag`. That lands on `snake_case`, which is invariant 1's founding exhibit, and escaping identifiers would be the dollar-pairing failure of section 1.2 happening again in the character decision 2 was proud of freeing. Separately, a caret capped at one digit truncates: `2^32` becomes 2³2 and `2^64` becomes 2⁶4, silently, and multi-digit exponents are the common case in technical prose. **The cap was not a limit but a silent limit** — and the inversion is the part worth keeping, because a greedy digit run would have been *safer* than one digit. The cap was costing prose-safety while appearing to buy it.

**Why the sigil sits outside the brace** — the choice that turned out to be the language's organizing principle. `{^2}` was proposed first and looked cheaper, since the brace-first constructs of the day already made `{` plus a sigil one shape and a fourth would have joined them for nothing. It was set aside on **invariant 5**. `H_{2}O` is a plain-text reading convention that predates Markdown; `H{_2}O` makes a plain-text reader stop. The cardinal rule is a promise to the reader, so where the two arguments met, the reader's governed.

That was recorded at the time as consistency losing to legibility, deliberately and once. **It was neither once nor a loss.** Decision 20 turned the other constructs around to match this one rather than the reverse, so the sigil-outside form became the rule and the brace-first form left the language. What looked like a bounded exception was the correct shape arriving early — and the general lesson is that the reader's argument and the consistency argument were never opposed here: taking the reader's side found the more consistent design.

**Why this is not a second way to write mathematics.** The objection is answered on the output rather than in prose: `E=mc^{2}` renders as body text with a raised 2, and `` `E=mc^2`{math} `` renders as typeset mathematics with italic variables. They do not produce the same page, so they are not two spellings of one construct — which is what invariant 2 is concerned with, as setext and ATX headings were. The construct then sorts its own cases, since a variable set upright looks wrong to whoever wrote it. The boundary holding the remaining pressure is one sentence with no "unless", and it is in section 4.4.8.

**The price, stated where it can be seen.** `^` stops being a character with no meaning anywhere, and `_` gains a meaning inside a line. Both sit at the mildest severity, each in one position only. One new collision survives: raw TeX pasted into prose outside a fence, where both braced runs fire. Against that, underscore comes out **safer than CommonMark's**, where it is emphasis and identifiers italicize mid-word under the flanking rules — the language adds a construct and the character gets safer than the baseline.

**The accepted failure mode:** someone writes `E=mc^2` from TeX habit and gets a literal caret. That is visible on the page rather than silent, which is the class this language tolerates — the same class as decision 5's over-long closing fence — and it is a validator diagnostic, on a sigil followed by a bare alphanumeric run. **Decision 20 generalized that diagnostic along with the rule:** every sigil in the language now means nothing without a brace, so one check covers all of them and the validator's most useful message is the same sentence in every place it fires.

### Decision 19 — The thematic break is removed

**Rule:** section 4.3.7. A line of three or more `-`, `_`, or `*` reserves nothing.

**Why — the construct has no plain-text meaning to preserve.** This language's method is to ask what a mark tells someone reading the source with no renderer. A run of hyphens on its own line says that something changes, and never says what: a topic shift, a scene break, an aside ending, the document finishing. A heading says which. The construct therefore fails invariant 5 from the inside — not because a reader cannot see it, but because seeing it tells them nothing.

**Why — its rendering is a style choice the language cannot describe.** A horizontal rule is typography, and it collides with the renderer's own: most house styles already draw a thin rule beneath a level-2 heading, so a document that separates sections with breaks *and* headings gets two rules and an argument between them. Decision 15's back half asks that every mark name something a renderer visibly distinguishes; this one asks the renderer to draw something the document has not said.

**The replacement is what it was standing in for: section properly, or write a second document.** A break between two topics is a heading that was not written. That is the same move as decision 4 removing setext headings and decision 6 removing marker-switching — not a capability withdrawn, but a shape whose job another construct already does, more explicitly.

**What the removal returns.** `*` and `_` each get their line-start position back. Neither becomes free outright, both keeping inline work — but this was the first of the two changes that emptied that column of everything except block markers, decision 20 being the second. And the list rule loses its only exception: the clause by which a thematic break beat a bullet item on `- - -` was the sole precedence contest in the block layer, and an ordering exception is an "unless" clause in other clothing, so invariant 2 collects a dividend unrelated to rendering.

**What it costs, stated plainly.** `---` between two paragraphs is common, and it now renders as three literal hyphens. The failure is loud rather than silent, which is the class this language tolerates, and a migrator can find every instance — though it cannot decide whether the author meant a heading or a document split, so this one is reported rather than converted. `***` and `___` go with it.

**Not adopted — keeping it for compatibility alone.** The compatibility principle decision 12 retired matched CommonMark only where unambiguous *and harmless*, and this construct was neither: harmless fails on the renderer collision, and its meaning is under-specified in a way no conformance test could pin down. A delta needs an invariant behind it; this one has invariants 2 and 5 both.

### Decision 20 — Every inline construct opens with a sigil and a brace

**Rule:** section 4.4. A sigil carries meaning only when the very next code point is `{` — extended by `[` for the link and image forms that carry their own text.

**Why it is one decision and not ten.** That sentence was decision 18, written for `^` and `_` and treated as a bounded cost. It was not bounded: it was the shape of the whole inline layer, discovered on two characters. Applying it to the rest costs *less* than the two-character version did, because one rule on the reference card replaces eight constructs with a shape each.

**The case rests on three findings, not on consistency.**

**Decision 3 was less prose-safe than CommonMark.** Uniform escaping rendered `C:\Users\frederic` as `C:Usersfrederic` where CommonMark leaves it alone. Ordinary technical writing, silently altered, with nothing on the page to show it. That is the failure class invariant 1 exists to prevent, and it had been in the language for two days behind a rule that reads perfectly.

**Decision 11 never protected emphasis.** The muscle-memory clause is conditional on Markdown not being broken, and emphasis is arguably the most broken construct CommonMark has.

**Braced emphasis removes invariant 1's last heuristic.** Flanking rules are prose-safety by heuristic; invariant 1 promises prose-safety by construction. After this decision the invariant holds with no residue, and no other available change does that.

**What it buys beyond the invariants.** The **last inline lookahead closes** — decision 8 closed the block layer's, and the bare `[` of a Markdown link was the inline layer's, requiring a scan to `]` and a check for `(`, where `@[` decides on the second code point. **Two heuristic rescues become constructions**: `[sic]` rendered literally under CommonMark *because no `(` followed*, and a brace in JSON or a shell snippet was safe for want of a neighbour. **CommonMark's hardest algorithm evaporates.** **Search becomes exact**, since every construct opens on a sigil run and one deciding code point, so one short pattern finds every operative inline construct at any depth — the un-regular part is only the *extent*, never the *presence* — and the constructs holding opaque targets cannot nest at all, so a regular expression over them is complete rather than approximate. **Syntax highlighting can be correct**, where editor grammars mis-highlight CommonMark emphasis everywhere because flanking is not expressible in the pattern languages they use. And **a language model gets shorter instructions**: flanking and the rule of 3 to emit CommonMark, `sigil{content}` to emit Markleft — which lands on the adoption argument in section 1 and was not available to Markdown in 2004.

**The three classes, which are the reference card.** *Blocks* are decided by position in the line — the start opens one, the end breaks one. *Inlines* are decided by a sigil immediately followed by `{` — with `[` in its place when a link or image carries its own text. *Verbatim* is delimited by matching backtick runs. The third is an exception that explains itself in one sentence, given in section 4.4.5, and one sentence is the bar invariant 2 sets.

**The ledger.** One character is spent: `@` stops being free, at the mildest severity, in one position only. Six are earned — the backslash drops two severity bands, the asterisk one, brackets and braces one each, and parentheses and the angle bracket become free outright. Seventeen of the thirty-two ASCII punctuation characters now mean nothing anywhere in the language, `-` is the only character left in the worst-but-one band, and a line's first column holds exactly the six characters that open blocks.

**The costs, none of them hidden.** A **clean break with Markdown**, which is decision 12 and is chosen rather than absorbed. **New collisions**, small and named in section 4.4.9. **Keyboard load**, since braces are a modified keystroke on several common layouts and this puts every operative inline construct behind that — the counter is the one decision 11 already accepted, that a character's inaccessibility and its prose-safety are the same property, and the mitigation is tooling and partial by construction, because a phone keyboard and a web textarea have no extension.

**Not adopted — keeping `*em*` alongside `*{em}`;** decision 2 records why. **Not adopted — a depth cap on brace nesting**, which would make the language regular for a regular expression's benefit and cost invariant 2 an "unless" clause; approximation belongs in the tool, not the definition. **Not adopted — prohibiting nesting outright**, which would kill the emphasis inside a superscript that decision 18 explicitly blesses.

### A note on risk

Decisions 4, 5, and 6 — one heading form, fenced blocks only, strict lists — carry the highest habit risk in the *block* layer, and decision 20 carries more than all of them together in the inline layer. They break what a fluent Markdown author's fingers already do. Every one of them is defensible on the invariants, and none of them has yet been tested against a large body of real documents. That evidence will arrive from the migrator's change report in Phase 3, which is later than anyone would like, and it is recorded here so that when it arrives it is read as evidence rather than as complaint.

## 6. Prior art and inheritances

Markleft is not the first attempt to fix Markdown, and it borrows heavily. This section names what is borrowed and from whom, because attribution is more useful than novelty.

### 6.1 The landscape

| Project | Size | Formal specification | Prose-safe | Mathematics | Validator | Steward |
|---|---|---|---|---|---|---|
| CommonMark | small | tests, no grammar | no | out of scope | no | volunteers |
| djot | small | prose and reference code | partial | yes, dollar delimiters | no | personal, then org |
| MyST | large | superset definition | inherits CommonMark | yes | partial | Jupyter |
| Quarto | large | none — a toolchain | inherits Pandoc | yes | no | Posit |
| Markdoc | mid | tag schema only | inherits its parser | no | tag layer only | Stripe |
| MDX | large | none | no | no | no | community |
| Typst | large | documentation and implementation | not applicable — it is code | yes, dollar delimiters | compiler | company |
| AsciiDoc | large | in progress for years | no | via stem blocks | in progress | Eclipse |
| Markleft | small | grammar and normative suite | invariant 1 | core, collision-free | Phase 3 | independent |

Every cell of the last row exists somewhere else in the table. The conjunction exists nowhere, and that conjunction is the project.

### 6.2 What each attempt taught

**CommonMark** proved that specification-as-tests works: a rigorous prose specification plus several hundred executable examples aligned an entire ecosystem. What it could not fix was the syntax itself — emphasis needing seventeen interlocking rules and a delimiter stack, list-indentation archaeology, seven kinds of HTML block, no mathematics, and an extension mechanism that never shipped. MacFarlane's own retrospective concedes the unfixable parts. **The lesson: standardizing an accident preserves the accident.**

**GitHub Flavored Markdown** added tables, strikethrough, task lists, and autolinks. Mathematics is *not* in that specification; it is platform post-processing added in 2022. **That architecture is the founding exhibit of this project.**

**djot** is the closest prior art and the direct implementation of CommonMark's own retrospective: linear-time parsing, no backtracking, uniform escaping, attributes on any element, mathematics, tables. Markleft shares its design principles and takes a large share of its syntax. Where it differs: djot keeps underscore emphasis and smart punctuation, treats mathematics and money collisions as ordinary rather than headline concerns, has no formal grammar, and has no validation or migration story. Markleft's additions over djot are formalization, prose-safety as invariant 1, the dollar verdict, and the tooling.

**MyST** has the most principled extension mechanism in the Markdown family — directives and roles, inherited from reStructuredText — but grown as a superset of an ambiguous base, so it inherits every underlying pathology. **A clean extension mechanism on an ambiguous core does not produce a clean language.**

**Quarto** is a publishing system rather than a language: its markup is whatever Pandoc accepts, plus conventions. It is enormously successful, and it succeeded on outputs, not on syntax. **Toolchain gravity beats language design — which is a reason to be a language, not a reason to become a toolchain.**

**Markdoc** demonstrated real demand for schema validation and machine-readable markup, and got closer than anyone to "Markdown with a validator" — but its validation covers its own tag layer, not the Markdown underneath. **Nobody has offered validation for the base language itself.**

**MDX** embeds JavaScript in content. It is the anti-pattern this language defines itself against: maximal power, no prose-safety, and unparseable without a JavaScript engine. It is also proof that platform integration drives adoption regardless of language virtue.

**Typst** is a document-preparation system with a real scripting language, an incremental compiler, and error messages people praise by name. It chose dollar delimiters for mathematics and underscore for emphasis — the collisions removed here — and that is legitimate, because Typst documents are programs rather than prose-safe plain text. **Different design goals produce different verdicts on the same character; scope discipline is what makes both correct.**

**AsciiDoc** is semantically rich and has been formalized retroactively under Eclipse Foundation stewardship since around 2020, with the effort still in progress. **Even with excellent governance, formalizing a large language after the fact takes half a decade and counting. Formalize small, and formalize first.**

**reStructuredText** had a proper specification before it was fashionable and is steadily losing ground anyway, including inside Python. **Correctness without ergonomics loses.**

**Org-mode** is arguably the most capable plain-text format ever built, and it is permanently trapped in one editor. **Editor-coupled formats do not propagate.**

### 6.3 Explicit inheritances

- **CommonMark** — the specification-as-tests methodology, and the block layer that decision 11 keeps.
- **djot** — the linear-time architecture, attributes (reduced here to decorators), and the principle of one uniform escape, which survives as delimiter lengthening after decision 20 retired the backslash form. Its bracketed emphasis was the stepping stone to the braced form. **Not** its raw-content design: decision 7 removes passthrough entirely.
- **MyST and reStructuredText** — the directive concept, disciplined here into a closed extension point, and never the content-generating directive that decision 15 leaves out.
- **Markdoc** — proof of demand for schema validation and documents-as-data.
- **Typst** — the standard to meet for error messages, and the Rust-plus-playground approach to credibility.
- **AsciiDoc and the Eclipse Foundation** — the technology compatibility kit concept, which is what the conformance suite is, and a cautionary timeline.

## 7. The name

Markup went up. Markdown went down. Markleft moved sideways.

Four readings, all of which describe the whole language rather than one feature:

1. **The compass.** An orthogonal move: the same size as Markdown, different rigour.
2. **What is left.** The language is what *remains* after fifteen years of curation. Most of the binding decisions in section 5 remove something.
3. **We left.** A departure from the informal era. The fourth direction stays unclaimed, left to whoever comes next to do it "right".
4. **Marks on the left.** **Every construct's mark sits to the left of that construct's content.** Blocks mark the left of a line; inlines mark the left of a span; a backslash before a line ending marks the left of the line ending. The one place the arrow reverses is the decorator, which annotates what precedes it. This is a retronym and worth saying so: the name came first, and it turned out to describe the design.

**The tagline states two invariants at once.**

> **Everything that is not prose leaves a mark.**

Its content is the contrapositive — **no mark, therefore prose** — which is invariant 1 from the reader's side and the cardinal rule at the same time, without naming either. Note that the name is the verb: *leaves a mark* and *Markleft* are the same two words in different tenses.

**The copyleft lineage is structural rather than decorative.** Copyleft used copyright's own machinery to guarantee openness. Markleft uses formal-specification machinery to guarantee prose freedom: the formalism exists so that plain text stays plain.

**On the naming family.** Markdown-adjacent names sort onto two axes. *Suffix-keepers* vary the prefix and keep "-down" — Showdown, Kramdown, Quarkdown — and keeping the suffix signals *membership*: another flavour of Markdown. *Prefix-keepers* hold "mark-" and vary the direction: markup, markdown, Markleft. Keeping the prefix signals *succession*: the next move in the sequence rather than another dialect of the ambiguous thing. Markleft is deliberately on the succession axis.

**And the pitch is a question.** A stranger asks "what's left?" — and the answer is the language.

## 8. Definition, realization, and comparison

The structure of this project is borrowed from metrology as an intellectual model. It carries no authority and implies no institutional backing; it earns its place because it is the right model.

**One definition.** The metre is defined by a single sentence about the speed of light. That definition has outlived every piece of apparatus ever used to realize it, precisely because it was written to be independent of all of them. This document is meant to work the same way: it defines the language without reference to any parser, any output format, or any platform.

**Many independent realizations.** A parser, a formatter, a migrator, a syntax highlighter, an implementation in another language — none is privileged, including the project's own reference parser. A standard whose definition is its implementation has one realization and no way to detect that it has drifted.

**Kept honest by comparison.** In metrology, a *key comparison* is how independent realizations are checked against each other and against the definition. That is exactly what the conformance suite is: mandatory, exhaustive, and binding on any conformance claim — and subordinate to this document, because a key comparison is not a definition.

Three consequences follow, and they shape the whole project:

- Conformance examples assert against the specified tree, not against HTML. CommonMark's suite asserts HTML output, which binds a standard to a serialization unrelated to the language it defines. That is the main reason test suites age badly.
- The reference parser is a realization and is documented as one. It is not the tie-breaker.
- The specification is versioned and the suite follows it (section 12).

## 9. Conformance

### 9.1 What conformance means

A **conforming parser** accepts any well-formed UTF-8 document, produces exactly the tree this definition specifies for it, never fails on any input, and passes the conformance suite in its entirety.

A **conforming validator** may report any of the warnings listed in section 4.2. It must not reject a document that a conforming parser accepts.

A **conforming renderer** may consume marks and must not lose content (decision 15). Beyond that, this definition does not constrain rendering.

There is no partial conformance and there are no conformance levels. The language is one reference card long; a subset of it is not a useful thing to implement.

### 9.2 How a claim is checked

By running the conformance suite. It is published separately so that an implementer in any language can vendor it without cloning anything else, and it records the revision of this document that it exercises.

### 9.3 The precedence rule

**When the suite and this document disagree, this document governs and the test is presumed to be the defect.** But the disagreement is triaged, because sometimes the test is the evidence that the document is wrong. Exactly one of three things is true:

1. **The test is wrong.** It asserts something this document does not say. Fix the test. This is the common case and needs no ceremony.
2. **This document means the right thing but says it unclearly.** The test did its job: it found an ambiguity, which under invariant 3 is a defect whether or not any implementation has tripped over it. Fix the prose or the grammar so that the intended reading is the only reading. Behaviour does not change; the text that determines it does.
3. **This document is clear, and wrong.** The design is defective. That is a revision of the specification, and it goes through the change process in section 12 — never through a quiet edit, and never by letting the suite silently carry corrected behaviour on its own.

"The document governs" settles *where a change must land*, not *who was right*. It is a routing rule, not a claim of infallibility. The kilogram was redefined because realizations outran the artefact; a discrepant comparison is sometimes a finding about the definition.

**What the rule excludes is exactly one thing: a behaviour that exists in the suite and nowhere in this document.** That is how a standard quietly relocates into its tests.

### 9.4 The conformance mark

The project reserves the possibility of a lightly governed "Markleft-conformant" mark, held by the project's maintainers and by no institution. This is a reservation, not a promise; nothing about it is decided.

## 10. Copyright, licensing, and non-endorsement

**Copyright.** Crown copyright, Canada — 2026. An author is a Canadian federal public servant, so the Copyright Act vests copyright in the Crown by operation of law.

**This is not sponsorship.** No department, agency, or institution owns, funds, backs, endorses, reviews, supervises, or vouches for Markleft. It is an independent project maintained by its authors. A copyright notice arising from employment law says nothing about who supports a project, and nothing in this document or any other should be read as implying that it does.

**Licensing.** This definition and the conformance suite are licensed **CC BY 4.0**. Copy them, quote them, adapt them, implement them, and sell what you build on them. Attribution is the only condition. All code produced by the project is licensed **MIT**.

CC BY was chosen over the share-alike variant deliberately: share-alike would force anyone quoting substantial parts into their own documentation to license that documentation alike, which is friction a specification meant to be reimplemented is better without.

## 11. Governance

Markleft is maintained by its authors and governed by no institution.

The process today is exactly this: one maintainer decides, and writes the decision down with its rationale and the alternatives it set aside. That record is public precisely so that the next person can argue with it, and section 5 of this document is its distillation. Describing anything more elaborate would be describing a committee that does not exist.

The trigger for something more is specific: **more than one decision-maker.** At that point the decision record becomes a request-for-comments process, the binding decisions in section 5 become numbered proposals retroactively, and contributor-facing process — a contributing guide, a code of conduct — is published alongside it. Standing that up sooner would produce ceremony and no contributors, which is a failure this project has studied in others.

## 12. Versioning and change process

**This document is versioned; the conformance suite follows it.** Each release of the suite records the revision of this document that it exercises, so any checkout of the suite states unambiguously which version of the language it is testing.

**The specification is never tagged ahead of the suite that exercises it.** A standard whose suite silently lags it is precisely the drift this project exists to prevent.

*The rule binds from the first release of the suite onward, and saying so is not a loophole.* Version 0.1.0 of this document was tagged while the suite was still a skeleton, which looks like a violation and is not one: the rule protects against a suite that once matched a specification and quietly stopped, and before any suite exists there is nothing that can lag. What the suite records in the meantime is which revision it *targets*, which it does now. Stating the qualifier is better than leaving a reader to find an apparent contradiction between this clause and the tag history — and it is the only version of the rule that can survive Phase 0, where the definition necessarily comes first.

**A change to the language and its conformance examples land together.** They are opened together and merged together across the two repositories; neither lands alone. This is weaker than atomicity and will occasionally be violated by accident, which is why continuous integration checks it.

**No behaviour may exist only in the suite** (section 9.3). A reviewer of a new conformance example asks which clause of this document determines it, and treats "none" as a finding rather than a formality.

**Every change is recorded before it is made.** A challenge to a settled decision is written down with its argument and its outcome, whether it succeeds or fails, so that a later contributor finds the analysis instead of rebuilding it.

**Version numbers follow semantic versioning, and for a specification the public interface is the language** — what parses, and to what. That makes the bands decidable rather than a judgement call each time:

- **`0.x.0`** — the language changed. A decision landed that changes what parses or what it means.
- **`0.x.y`** — everything else. Prose, rationale, corrections, worked examples. The language is byte-identical.
- **`1.0.0`** — this document is normative and a realization passes the conformance suite.

The `0.0.x` series covers the scaffolding pass, when nothing was stable and nothing claimed to be. `0.1.0` is the first release of the language as described here.

## Appendix A — Deltas from Markdown

*Normative.*

The complete list of every point where Markleft departs from CommonMark is maintained in `deltas.md` and incorporated here by reference. Under decision 12 it is a **translation guide** rather than a migration guide: it maps the two languages onto each other for a reader who already knows Markdown, and it makes no promise that a document ports.

Three things deserve advance warning, and the first is the shape of the whole appendix. **Under decision 20, every inline construct differs from Markdown's** — links, images, emphasis, anchors, and escapes. A Markdown document is not a Markleft document, and a Markleft document is not Markdown; the two languages share a block layer and nothing below it. That is a break taken on purpose, and decision 12 gives the argument.

Beyond that, two entries are the ones a migrator has to report rather than silently convert. **The removal of trailing-space hard breaks (decision 13)** is invisible in the source, so a reader cannot audit that conversion by eye. **The removal of the thematic break (decision 19)** cannot be converted at all, because nothing in the source says whether the author meant a heading or a document split.

## Appendix B — Formal grammar

*Normative, when it exists.*

Not yet written; it is the first deliverable of Phase 1. It will contain a parsing-expression grammar for inline content and an explicit small-step algorithm for block structure, stated honestly where the block layer is not context-free — fence-length matching, for instance, is not.

The division of labour is deliberate: this document's prose is what a reader learns the language from, and the grammar is what an implementer builds from. Neither is a summary of the other, and where they disagree the disagreement is a defect, triaged under section 9.3.

## Appendix C — Conventional labels

*Non-normative, and deliberately outside this document's normative text.*

Not yet written. It will record what decorator words conventionally mean (`math` is TeX, `rust` is Rust), and what anchor-naming conventions tools have settled on. It is revisable without a revision of this definition, which is the entire reason it is a separate appendix: **no word is ever reserved**, so the meanings of words cannot live in the normative text.

Two pieces of guidance for tool authors are worth recording in advance.

**Prefer a colon for tool-managed anchors.** An author inventing an anchor by hand reaches for hyphens and almost never for a colon, so a colon works as a namespace separator precisely because people do not type it spontaneously. Tool-managed and hand-written anchors then occupy visibly different spaces, which makes a collision **unlikely rather than impossible** — nothing stops an author typing a colon, so this is a probability argument and a tool needing certainty must still check.

**Name an anchor; do not number it.** Inserting one figure renumbers every figure after it, so a number-bearing anchor means every later anchor changes *and* every link to it changes — a large diff for a one-figure edit. A name is stable under insertion. The displayed number is written into the source at authoring time and can be regenerated freely; the anchor it hangs on should never move.

That second piece of guidance is a specific case of a principle that decides three unrelated questions in this project — how prose is wrapped, how ordered lists are numbered, and how anchors are named: **do not encode a position in something that other things point at.** Positions shift; references do not want to.

## Appendix D — Open items

*Non-normative. Every point this document settles for the first time, and every point it leaves open, so that no gap has to be discovered by a reader.*

Items 1 to 4 are answered in section 4 for the first time; each closes a rider that the decision record left open, and each needs the answer confirmed before it is treated as settled. Items 5 to 14 are not answered at all, except item 6, which is closed and kept in place — numbers here are retained rather than reused, so a citation always resolves. Items 12 and 13 were raised by decision 20; item 14 was found during its review.

1. **Invalid UTF-8.** Section 4.1 states that a byte sequence which is not well-formed UTF-8 is not a Markleft document. The alternatives were replacement with U+FFFD and passing bytes through; both silently alter content, which decision 3 and invariant 1 argue against.

2. **Leading byte-order mark.** Section 4.1 states that a leading U+FEFF is not part of the document text. The alternative is to treat it as content, which is the purer reading but lets an editor's byte-order mark silently break the first heading of a file.

3. **Anchor and label matching.** Section 4.1 defines matching as code-point identity, with no case-folding and no normalization. This is the recommendation of the encoding memo and is adopted here.

4. **The meaning of "column".** Section 4.1 defines it as a count of code points rather than display width, so that no rule consults a versioned Unicode property. Adopted from the same memo, and the cost — wide characters aligning by count rather than by apparent width — is stated in section 4.1 rather than buried.

5. **Tilde fences.** CommonMark admits `~~~` as an alternative fence. As drafted, this document answers no: section 4.3.3 describes backtick fences only, and section 4.3.7 closes the block-opener list without `~` — which is also what keeps `~` a character with no meaning anywhere. The answer follows the one-way-to-do-it discipline and stays here until it is confirmed; confirming it adds the removal to Appendix A.

6. **The thematic break spelling — CLOSED 2026-08-08 by decision 19.** The item asked whether a run of underscores survived decision 2. The construct is removed entirely, so the question does not arise. The number is retained rather than reused, so that a note citing D.6 still resolves.

7. **Lazy continuation in block quotes.** Decision 6 removes it from lists. Block quotes are inherited from CommonMark, which permits it. Consistency argues for removing it there too; nothing has decided.

8. **The loose-and-tight list rule.** Decision 6 requires one explicit rule and does not state it. It belongs in Appendix B.

9. **Verbatim-span whitespace.** CommonMark strips one leading and one trailing space from a verbatim span when both are present. That is a rule with an exception clause, and whether it is inherited is undecided.

10. **Media type.** No media type is registered for either file extension. The open question is whether and when to pursue an IANA registration (`text/markleft` being the obvious candidate); nothing before 1.0.0 forces it.

11. **A malformed decorator list.** Section 4.5 states the form of a decorator list without saying what a line that fails it becomes. Two answers are available and both are defensible: the uniform fallback of section 4.2 makes the whole line paragraph text, which is what CommonMark does with ```` ```rust``` ```` and keeps the grammar honest; or the fence opens anyway and the malformed list is a validator error, which keeps a document's block structure from collapsing over a typo in a label. The cases are a token containing a backtick, two bare words, and two anchors. Raised by the backtick exclusion decided 2026-08-08; it applies to all three.

12. **Whether a brace group may span a line ending.** Section 4.4.9 defines how a brace group closes without saying whether the close may fall on a later line of the same paragraph. It matters for a long link target and for emphasis in a hard-wrapped document. The natural answer is that a paragraph is one logical unit of inline content, so a group closes anywhere within it — but nothing states that, and an implementer would have to guess. Raised by decision 20.

13. **Whether `**{…}` is worth having.** Strong emphasis inside a word is close to unattested, and symmetry with `*{…}` and `***{…}` is the only argument for it. Keeping it costs no rule, since it falls out of the run count; removing it would be the first arbitrary gap in a run sequence. Raised by decision 20 and deliberately not settled by it.

14. **Strikethrough and HTML entity references.** Neither is mentioned anywhere in this document, and both are absent by accident rather than by decision. Strikethrough is a GitHub extension, and under decision 20 a braced form would be `~{text}`, which would cost `~` its position as a character with no meaning anywhere. Entity references are in CommonMark, and they render a character the source does not contain, which is decision 15's front half through a very small door; the compatibility argument for keeping them ended with decision 12.

Formatter behaviour — the order of tokens inside a decorator, whether ordered-list numerals are normalized to all-`1.` or written in sequence, and whether a redundant label is removed — is deliberately not in this list. Those are questions about a tool, not about the language, and they are settled when the formatter is built.
