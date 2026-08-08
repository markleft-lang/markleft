# The Definition of Markleft

*Crown copyright, Canada — 2026. Licensed CC BY 4.0.*

## Status of this document

This is a working draft, written during Phase 0 of the project. The language it describes is settled in substance: every rule stated here comes from a recorded decision. Nothing is frozen. The formal grammar in Appendix B is not written, the conformance suite does not exist, and no implementation has shipped — so no conformance claim is possible yet, and any rule here may still change. Appendix D lists, precisely, every point this document leaves open.

The document is named *definition* rather than *specification* or *standard* for a reason worth stating once. Metrology defines the metre in a single sentence and then checks every physical realization against that definition; the most rigorous document in the history of programming languages is *The Definition of Standard ML*. Both senses apply here. The name also says what this document is **not** — a realization. It was called `charter.md` until 2026-08-07; in IETF and W3C practice a charter is a working group's governance document and never the technical text, so the old name pointed at the wrong thing. If sections 10 to 12 ever split into a separate document, that document is legitimately the charter.

## How to read this document

**Normative sections** state what the language is and what an implementation must do: section 3 (invariants), section 4 (the language), section 9 (conformance), section 12 (versioning), Appendix A (deltas), and Appendix B (the grammar, when it exists).

**Informative sections** explain, justify, and position: sections 1, 2, 5, 6, 7, 8, 11, and Appendices C and D. Section 10 states a legal position rather than a technical requirement. Where an informative section appears to state a rule, section 4 governs.

**Requirement words.** *Must* states a requirement on a conforming implementation. *Must not* states a prohibition. *May* states a permission — an implementation is free to do the thing, and free not to. Nothing in this document uses *should*: a rule that only recommends is a rule the language cannot rely on.

**Terms.** A **document** is the complete text an author writes. A **parser** turns a document into a tree. A **renderer** turns that tree into something a person looks at — a web page, a printed page, a screen reader's speech. A **realization** is any of these: an independent implementation of this definition. A **validator** reports problems a parser is not allowed to fail on. A **formatter** rewrites a document into one canonical form. A **migrator** converts Markdown into Markleft. The **conformance suite** is the executable set of examples every realization is checked against.

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

It defines the language. Specifically, it states the five invariants that every part of the language must satisfy (section 3), the language itself (section 4), the reasoning behind each binding decision including the alternatives that were rejected (section 5), what conformance means and how it is checked (section 9), and the project's licensing and governance position (sections 10 and 11).

### 1.4 What is in scope, and what is not

Markleft defines what a document **is**. It does not define what a renderer **does** with it. The language guarantees that a document has one unambiguous meaning; how that meaning is typeset, coloured, or spoken aloud is left free, and always was. This line is drawn deliberately and appears throughout section 4 — most visibly around mathematics, where the language guarantees that content is carried through untouched and says nothing whatsoever about how it is typeset.

## 2. Non-goals

Three things Markleft is not. Each rules out a different misreading, and each is enforced by a rule rather than by intention.

### 2.1 Not a toolchain, and not a document-preparation system

There is no templating, no executable code, no scripting, no page or layout awareness, and no output-format directive. Languages that have those things — Quarto, Typst, Quarkdown, and LaTeX before them — are a different and legitimate product. They are also, all of them, languages in which the first five minutes are expensive, and adoption is decided in the first five minutes.

### 2.2 Not a richer Markdown for its own sake — and not an austerity project either

Markleft adds things. Mathematics is part of the core, decorators are new, pipe tables are in the language rather than in a platform's post-processing, and several constructs are adopted from djot. What every addition has to clear is a single bar: **it must remove an ambiguity or fill a genuine gap, and it must cost nothing in plain-text clarity.** A construct that makes ordinary unmarked prose read worse is declined regardless of merit, because prose-safety is invariant 1 and richness is not an invariant at all.

The budget for all of this is the five-minute property. Most of the binding decisions in section 5 remove something; a few add; the total still fits on one reference card. So "Markleft could also do X" is never an argument by itself. The argument has to be that X is worth what it costs every reader who never uses it.

### 2.3 Nothing renders that is not in the source

This is invariant 5, stated here as a promise rather than as a rule. There is no table-of-contents directive, no automatic section numbering, no generated index, no transclusion of text, and no variable substitution. **A reader can always read the entire document in plain text, with no compiler, viewer, or extension.**

The important thing about this non-goal is that it does not remove capability; it relocates it. Tables of contents, citation formatting, cross-file snippet synchronization, and figure numbering are all genuinely useful, and all of them are better done by a tool that writes plain text **into** the source at authoring time than by a renderer that produces text at display time. The tool's output is then ordinary text: a reader without the tool still sees it, a diff still shows it, and the format has gained nothing it must carry forever.

That last clause is the strategic point. Because capability accumulates in tools rather than in the language, the first five minutes never get worse however large the ecosystem grows. Most small languages start small and then grow. This one has a structural reason not to.

## 3. The invariants

Five invariants. They are constitutional: a proposed feature that violates one is rejected regardless of merit, and no rule elsewhere in this document may contradict them.

### 3.1 Invariant 1 — Prose-safety

Natural-language prose renders verbatim. A paragraph means what it says unless it contains a delimiter in a structurally meaningful position, and the set of such positions is small enough to state on one page.

The word doing the work is *by construction*. Money amounts, `snake_case` identifiers, arithmetic like `5 * 3 = 15`, shell commands, file globs, and lines that happen to begin with a hash or a hyphen are safe because no rule in the language claims those characters in those positions — not because a heuristic guessed correctly. Prose-safety is testable, and it is tested: the conformance suite includes a corpus of prose written specifically to attack it.

### 3.2 Invariant 2 — The five-minute property

The complete core fits on one reference card, and **no rule has an "unless" clause.** A rule that needs an exception is a wrong rule, and the exception is evidence that something upstream was designed badly.

This is the invariant that most often does the rejecting, because almost every attractive addition arrives with a small exception attached.

### 3.3 Invariant 3 — The one-meaning property

Every input has exactly one parse. Not one parse per implementation, not one parse per platform: one parse.

This requires the definition to be executable rather than interpretable. Prose can be read two ways; a grammar and a suite of worked examples cannot. Two authors may write the same document two ways — that is style. One document parsing two ways is a defect.

### 3.4 Invariant 4 — The linear-time property

A single pass, no backtracking, and no pathological inputs. Which block a line opens or continues is decided from that line and the containers already open — never from a later line. Parsing time is linear in the length of the document.

This is not only a performance claim. Prefix-decidability is what makes the language explainable: a rule that depends on a later line cannot be stated as a rule about the line in front of you.

### 3.5 Invariant 5 — The cardinal rule

**You can always read the entire document in plain text, with no compiler, viewer, or extension. Nothing renders that the source does not already say.**

The rule is enabling, not restrictive. It is the minimum that lets capability live in tooling and be built cleanly: a tool that writes into a document needs to know that what it wrote is what the reader will see, and this rule is that guarantee. Features are relocated, not forbidden.

The operative detail — where exactly the line falls between presenting written content and manufacturing new content — is decision 15 in section 5.

### 3.6 Five invariants, six guarantees

The project's public material lists **six guarantees**: prose-safety, "verbatim means verbatim", five minutes, one meaning, linear time, and "read it anywhere". The counts differ on purpose and the relationship is this: the invariants are internal constitutional tests used to reject features; the guarantees are the user-facing promises that follow from them.

Five of the six correspond one-to-one with the five invariants. The sixth, **verbatim means verbatim** — a backslash before any character yields that character, line endings included, with no exception list, and every Unicode code point is text — is not an invariant. It is a promise derived from decisions 3 and 14. It is stated publicly because it is what an author most wants to be told, and it is not stated as an invariant because it is a consequence rather than a test.

## 4. The language

This section is normative. It states the whole language in prose. The formal grammar in Appendix B, when it is written, pins the exact forms; where the grammar and this prose disagree, the disagreement is a defect to be triaged under the precedence rule in section 9.

### 4.1 Characters, encoding, and lines

A Markleft **document** is a finite sequence of Unicode code points.

A document is stored and exchanged as **UTF-8**. Decoding happens before parsing and is a separate step: the parser sees code points, never bytes. A byte sequence that is not well-formed UTF-8 is not a Markleft document, and a tool handed one reports an encoding error rather than guessing at a repair. *(This clause closes an open rider; see Appendix D.1.)*

Every code point is text. No part of the language privileges an ASCII subset. The full Unicode range is available in prose, in headings, in link text, in verbatim content, and in decorator tokens alike, and every code point carries the same guarantees as any other.

**Content is never normalized.** A conforming parser applies no Unicode normalization form, changes no case, and substitutes no character. What the author wrote is what the document contains. This is why there is no smart punctuation in the language: quotation marks and dashes are what the author typed.

It follows that **two identifiers or labels match when they are the same sequence of code points, and not otherwise.** There is no case-folding and no normalization. This keeps matching independent of the Unicode version: a rule that folded case would let two labels stop matching, or start matching, on a Unicode release that nobody in this project made. *(This clause closes an open rider; see Appendix D.3.)*

If a document begins with U+FEFF, that code point is not part of the document text. Anywhere else, U+FEFF is ordinary text. *(This clause closes an open rider; see Appendix D.2.)*

A **line ending** is a line feed, a carriage return, or a carriage return followed by a line feed. A **line** is a sequence of code points up to the next line ending or the end of the document; the last line need not end with a line ending.

**Tabs never determine structure.** Where the language measures structure in columns, a tab is not a substitute for spaces. A tab in a structural position is well-formed input and produces a validator warning with an automatic fix, never a parse failure.

**Trailing whitespace at the end of a line is never significant.** A line and the same line with spaces appended are the same input.

Where this document measures a **column**, it counts code points from the start of the line, beginning at column 1. Display width is not used. This means text containing wide characters aligns by count rather than by apparent width, which is stated plainly here rather than discovered later. The reason is durability: display width is a versioned Unicode property whose values have moved between releases, and a rule that consults it would let a document's parse change over time under a definition nobody edited.

### 4.2 How a document is read

Parsing has two stages. First the **block structure** is determined, line by line. Then **inline content** is parsed inside those blocks that contain text.

Block structure is decided from each line's own prefix together with the containers already open. No rule looks ahead to a later line, no rule backtracks, and parsing takes time linear in the length of the document.

**The uniform fallback.** A line that does not match the opening form of any block is paragraph text. Inline content works the same way: a sequence of characters that does not match a construct's form is text. This single fallback is what makes prose safe, and it is why so few rules in this document need to say anything about prose at all — prose is simply everything that is left.

**The parser is total.** Every document that decodes as UTF-8 has exactly one parse, and there are no parse errors. A fence that is never closed closes at the end of its container. Emphasis that is never closed is an asterisk. A table row that does not match the table form is a paragraph. Nothing an author can type causes a parser to refuse the document.

Judgements that need more than the text in front of the parser therefore belong to the **validator**, and they are warnings, never failures. There are seven of them today:

- a decorator word the validator does not recognize;
- an anchor identifier used more than once in a document;
- a link to a fragment in the same document that no anchor defines;
- an image with no alternative text;
- a tab in a structural position;
- a decorator that only restates what the bare construct already means;
- a verbatim block closed only by the end of its container, naming any unmatched backtick run it passed on the way.

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

Number signs are the only heading marker. There are no underlined (setext) headings: besides being a second spelling of one thing, an underlined heading is a heading only because of the *next* line, which invariant 4 forbids.

There is no closing sequence. In `## Title ##` the text is `Title ##`. CommonMark's rule for when a trailing run closes a heading and when it does not is the clearest available example of the "unless" clause invariant 2 forbids.

There is no leading indentation: the run sits in the first column, after any container prefix such as a block-quote marker or list-item indentation. CommonMark allowed up to three spaces in order to keep headings clear of indented code blocks, and decision 5 removed indented code blocks, so the allowance now buys nothing and costs a boundary case.

There is no empty heading. A line holding only a number sign is a paragraph whose text is a number sign.

Six levels, because HTML has six, DocBook and LaTeX agree, and CommonMark caps there too. A document that needs a seventh level needs splitting.

Note what this rule does for prose without any special case: `#hashtag` and `#5` are text at the start of a line because neither has the required space. The one genuinely ambiguous spelling, `# 5 is my favourite`, is written `\# 5 is my favourite`, using the ordinary escape from section 4.4.2.

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

A block quote is introduced by a `>` at the start of a line, optionally followed by one space, and contains blocks. It is inherited from CommonMark unchanged.

#### 4.3.5 Lists

Lists are the most rule-heavy part of any Markdown-like language, and the place where implementations most often disagree. Markleft replaces the archaeology with arithmetic.

**There is one marker of each kind.** A bullet item is `-`, one space, then content. An ordered item is one or more digits, `.`, one space, then content. `*`, `+`, and `1)` are not list markers: a line beginning with one of them is text, or a thematic break where it matches that form.

**Content-column alignment.** An item's **content column** is the column at which its content begins — the marker, one space, then content. Every continuation line of that item is indented to exactly that column.

**No lazy continuation.** A line that is not indented to the content column is not part of the item. Markdown's tolerance for under-indented continuation lines is the single largest source of disagreement between implementations, and it is removed.

**Adjacent items are one list.** A list ends only where a block that is not a list item begins — a paragraph, a heading, a fence, a table, or a thematic break. **Blank lines never split a list:** one makes it loose, and two do nothing further. Markdown's trick of switching markers to separate two touching lists does not exist here, because there is no second marker to switch to.

````
- The first item.
  Its second line is indented to the content column.

- The second item.

10. An ordered item.
    Its content column is column 5.
````

**Ordered lists renumber.** A list begins at its first marker's value and its items render sequentially from there, whatever numerals are written. `1.` `1.` `1.` renders as 1, 2, 3.

This is worth defending explicitly, because it looks at first like generated content and invariant 5 forbids that. It is not. A rendered ordinal is not text taken from somewhere else; it is a presentation of the item's **position**, and position is fully stated by the source. A reader looking at three items in plain text already knows there are three, in order — they can count. Nothing is relocated, as a table of contents relocates heading text, and nothing is imported, as an image imports a file. What makes the behaviour necessary rather than merely permissible is that hand-maintained numerals are wrong in practice: they break silently on every edit, and the failure is invisible until someone reads the rendered output.

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

**A rule row is recognized only as a table's first or second row.** Anywhere else, a row of dashes is content. That restriction and the three-hyphen minimum exist for the same reason: a cell holding a single dash is a common way to write "not applicable", and without them it would be read as structure and its content would disappear. A cell whose content is genuinely three or more dashes, in one of those two positions, is written `\---`.

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

**A table is decidable from its own first line**, like every other block. Markdown's table needs to see the next line before it knows whether the current one is a header or a paragraph, which is the property section 4.3.2 removed setext headings for. Opening on the pipe removes it here too, rather than granting tables an exception.

**A block cell is a container**, in the same sense as a list item or a block quote: inside it, lines belong to the cell. Note the one difference, stated rather than left to discovery — a list item marks every line it owns by indentation, where a block cell marks only its first and relies on the terminator. Both are coherent, and the reason a cell does not use indentation is that the `|` already does the work indentation does in a list.

#### 4.3.7 Thematic break and link reference definitions

Both are inherited from CommonMark unchanged: a thematic break is a line of three or more `-`, `_`, or `*`; a link reference definition binds a label to a destination for use by reference links (section 4.4.6). See Appendix D.6 for one open detail.

Note that a thematic break no longer competes with anything. Setext headings are gone, so `---` under a line of text is unambiguously a break rather than an underline; and `*` is no longer a bullet marker, so `***` is a break rather than a list item whose content is `**`. Both conflicts were removed by subtraction elsewhere, not by a precedence rule here.

### 4.4 Inline content

#### 4.4.1 Text

Anything that does not match a construct below.

#### 4.4.2 Escapes

**A backslash before any character yields that character.** There is no exception list, and *any* means any.

````
\*not emphasis*     yields   *not emphasis*
\\                  yields   \
\a                  yields   a
\{#not-an-anchor}   yields   {#not-an-anchor}
````

This is the most uniform rule in the language and also the one that most often surprises someone arriving from Markdown, where a backslash before a letter is a literal backslash. Here it is not. A backslash that is the last code point of a document is a literal backslash, because there is no character after it to yield.

#### 4.4.3 Hard line break

A backslash immediately before a line ending is a hard line break. It is the only one.

Markdown's other spelling — two or more spaces at the end of a line — is removed. Invisible syntax cannot satisfy prose-safety or the one-meaning property: a reader cannot see it, a diff barely shows it, editors and commit hooks strip it on save, and the widespread habit of typing two spaces after a full stop turns ordinary prose into structure whenever such a sentence happens to end a line. Nobody can rely on a mark they cannot see.

#### 4.4.4 Emphasis

`*text*` is emphasis. `**text**` is strong emphasis. Underscore is not emphasis syntax anywhere, which is what makes `snake_case` identifiers safe in running prose with no escape.

The flanking rule is two conditions: an opening delimiter is immediately followed by a non-whitespace character, and a closing delimiter is immediately preceded by one. CommonMark needs seventeen interlocking rules and a delimiter stack here; those seventeen rules are the price of keeping underscore.

For emphasis inside a word, use the bracketed form `{*text*}`, adopted from djot. The braces make the boundaries explicit, so no flanking rule is consulted at all.

#### 4.4.5 Verbatim spans

A verbatim span opens with a run of one or more backticks and closes with a run of the same length. Runs are maximal, in the sense given in section 4.3.3. Its content is verbatim: no escape is processed and no construct is recognized inside it.

The counting rule is the same sentence as at block size, and for the same reason: no escape reaches inside, so **choose a length that does not occur as a run in the content**. That is what "one or more" is for. Inline the choice may go down as well as up, a single-backtick span being free to hold a doubled run.

A decorator list (section 4.5) may follow the closing run immediately, in braces.

````
`x = y`             an unlabelled verbatim span
``a `b` c``         a doubled run, so the content may hold a single one
`x = y`{math}       a verbatim span labelled math
`x`{#snippet-3}     a verbatim span carrying an anchor
````

**A bare verbatim span, and a bare fence, mean verbatim plain text.** That is their definition, not a fallback from a missing label.

#### 4.4.6 Links and images

Links are unchanged from CommonMark: `[text](destination)`, optionally with a title, plus the reference forms that use a link reference definition elsewhere in the document.

Cross-references need no new syntax, and that is the point. `[text](#anchor)`, `[text](other.md#anchor)`, and `[text](https://example.org/page#anchor)` are one construct — a link whose destination is a URL carrying a fragment. Internal and external references behave consistently because URL semantics already unified them.

Images are also unchanged from CommonMark: `![alt](src)`. Alternative text is optional and empty alternative text is legal; missing alternative text is a validator warning, never an error.

The relationship between the two is worth stating, because it makes the exclamation mark memorable rather than arbitrary: **`[…]` links to the target; `![…]` shows it instead.** The mark is a presentation switch, not a different construct.

#### 4.4.7 Anchors

`{#identifier}` is an **anchor**. It is valid anywhere inline content is valid, and it marks a **point** in the document rather than decorating any particular construct. There is no attachment rule, so there is no question of what an anchor attaches to, no exclusion for paragraphs, and no adjacency subtlety. Mid-sentence anchoring is free.

Whitespace on either side of an anchor is optional and insignificant.

The rule that places an anchor well is one sentence: **an anchor marks where the reader arrives.** A link scrolls its target to the top of the reader's view, so an anchor at the end of a long paragraph delivers the reader past the thing they were sent to read.

````
{#five-minute}The complete core fits on one reference card.

## The five-minute property {#five-minute}
````

Leading for anything longer than a line, trailing on a heading. Neither is a special case; both follow from asking where the reader should be looking.

Two authors anchoring the same paragraph at different points is a difference in style, not an ambiguity: each document has one parse.

Identifiers are opaque. `fig-3` means no more to the language than `banana` does, which is what lets tools build figure numbering, section cross-references, and citation keys on naming conventions without the language arbitrating between them. Non-normative guidance for tool authors is in Appendix C.

Uniqueness is a validator check, not a parse rule. A repeated identifier is well-formed syntax and a broken anchor, and a document-wide constraint cannot be checked without abandoning the local, single-pass discipline invariant 4 buys.

#### 4.4.8 Brace groups, disambiguated

Three constructs use braces. One rule separates them, it is positional, and it needs no lookahead:

- A brace group **immediately following a verbatim closing run** is a decorator list.
- A brace group beginning with `#` elsewhere is an anchor.
- A brace group beginning with `*` is bracketed emphasis.
- Any other brace group is ordinary text.

The cost of this is real and worth naming: `{#` is structural in prose, so template snippets such as `{#if}` and `{#each}` need `\{` when written outside a verbatim span. The uniform escape covers it and no new rule is required, but it is a papercut for one audience.

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

**The braces belong to the list, not to any token.** They are its delimiter, and they appear only where one is needed: inline, where the sentence continues after the list, and not on a fence, where the rest of the line is the list and nothing follows it. So the same `#identifier` token is written bare on a fence, bare inside an inline list, and braced on its own in running prose — where the braces are the standalone anchor's own delimiter (section 4.4.7), not the anchor's punctuation. Braces on a fence's list would nest inside the inline one, `` `x`{rust {#id}} ``, which is why the fence form carries none.

**At most one bare word**, because two format words would claim two content types for one box. There is no meaning to combine them into and no resolution order to appeal to, so a second word buys a race rather than a capability.

**At most one anchor**, matching HTML, where an element carries a single identifier.

**No word is ever reserved.** This document defines the grammar of a decorator list and the meaning of the `#` sigil. It does not define what any bare word means. A label is opaque, is carried into the tree exactly as written, and **cannot affect the parse**. The grammar is therefore fixed independently of any word list — no future word can quietly become a directive, because words have no parse-level power by construction, and there is no registry to maintain, version, or argue about.

Meanings live outside this document. That `math` means TeX and `rust` means Rust is convention, recorded in the non-normative Appendix C and in a validator's list of recognized words, both revisable without touching the language. An unrecognized word parses as plain verbatim content and earns a validator warning, never a parse error. Vendors are advised to prefix `x-` against collisions; nothing enforces it, because it is a convention rather than syntax.

There is consequently **no `text` label**. It is not forbidden — forbidding a word would reserve it — but it is never blessed either: nothing here assigns it a meaning, a validator reports it as redundant, and the canonical formatter removes it.

A labelled span and an unlabelled span are **different nodes**. They may render identically; the tree records which the author wrote.

**Token character set.** A decorator token is a run of Unicode characters excluding whitespace (which separates tokens), control characters (nothing invisible is ever syntax), `{`, `}`, `(`, `)` (which would close a brace group or a link destination), and the backtick (which would put an unpaired run inside a construct delimited by counting runs). Nothing else is excluded: letters in any script, digits, `:`, `-`, `.`, and `_` are all writable. The list is stated by exclusion, and it has five entries, because a list of five exclusions can be audited where an inclusion list is a standing argument.

The backtick exclusion earns its place twice. On a fence the decorator list runs to the end of the line, so without it ```` ```rust``` ```` would be an opening fence whose label is `` rust``` `` — a line that reads as a closed empty fence and is not one. Inline, a backtick inside a decorator would be a candidate opener for the next span in the paragraph, making the pairing of later runs depend on text inside braces. Nothing is lost: no format word and no identifier wants a backtick, and CommonMark excludes it from a backtick fence's info string for the same reason.

What such a line is *instead* is a question this exclusion raises and does not answer — whether a malformed decorator list makes the whole line paragraph text, under the uniform fallback of section 4.2, or leaves a well-formed fence carrying a validator error. It is the general case and it applies equally to two bare words in one list. See Appendix D.11.

**The first character selects the shape:** `#` makes the token an anchor, anything else makes it the format word. Sigils are significant only in first position, so `#` inside a token is an ordinary character.

**Legibility.** `` `x=y`{math #eq-euler} `` is the ceiling of this grammar rather than its normal use, and a decorator can grow longer than the content it decorates. The grammar permits it anyway, because forbidding the full form inline only would be an "unless" clause. House style carries that cost instead: anchors belong on fences, where a block has room for them, and an inline decorator normally carries the format word alone. What keeps even the ceiling inside invariant 1 is the postfix order — the content reads first, and the decoration is a suffix on a span the reader has already finished.

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

### 4.7 What the language does not contain

Stated explicitly, so that no reader has to infer an absence:

- no raw passthrough of any kind — no raw block, no raw span, no format escape hatch, and no HTML;
- no generated content — no table-of-contents directive, no automatic numbering, no index, no textual transclusion, no variable substitution;
- no underlined (setext) headings, and no heading closing sequences;
- no indented code blocks;
- no lazy list continuation, no alternative list markers, and no alphabetic or Roman numbering;
- no trailing-space hard breaks;
- no underscore emphasis;
- no smart punctuation;
- no classes, no key-value attributes, and no column width hints;
- no reserved words.

Because there is no raw passthrough, **a Markleft document is inert by construction.** It cannot execute anything and cannot embed anything, ever. No Markdown flavour can make that claim.

### 4.8 File names

Exactly two extensions: **`.markleft`**, the long form, canonical in documentation; and **`.lf`**, the short form. A language with one meaning per input should not have four names for its files. `.lft` is held in reserve as a fallback and is not a live option.

No media type is registered. That is an open item, listed in Appendix D.

## 5. The binding decisions

This section is informative. It records what was decided and why, including the alternatives that were rejected, so that a reader can evaluate the language rather than merely learn it, and so that a later contributor does not relitigate a settled question from scratch.

**The numbering is stable and citable.** Decisions are referred to by number throughout this project — in this document, in the deltas appendix, in commit messages, and in design memos. Numbers are never reused and never renumbered.

### Decision 1 — The dollar sign is never syntax; mathematics is decorated verbatim content

**Rule:** a bare dollar sign is always literal text. Display mathematics is a fence labelled `math`; inline mathematics is a verbatim span decorated `{math}`. Section 4.6 states the three guarantees this buys.

**Why:** section 1.2. Dollar-delimited mathematics was never in any Markdown specification, and every implementation of it is a heuristic operating outside the parser. The goal here was never to specify a mathematics engine. It was unequivocalness — to end the dollar mess — and the language achieves that by guaranteeing what the source unambiguously *is* while saying nothing about what a renderer does with it.

**Rejected — `\(…\)` for inline mathematics.** This was the original design and it is withdrawn, for two reasons. It cannot coexist with decision 3, which yields the character after any backslash with no exception list, so `\(` is simply a literal parenthesis and those delimiters cannot exist. The deeper failure is in the content rather than the delimiters: inline TeX is dense with backslashes, so `\(\frac{a}{b}\)` loses `\f` to an escape and offers no way to tell a closing `\)` from an escaped parenthesis. **Inline mathematics has to be verbatim**, which is why the fenced form was never in trouble and why the inline form belongs in the same machinery.

**Rejected — carving `\(` and `\)` out of decision 3.** It costs the "no exception list" guarantee *and* does not fix the content problem, so it buys nothing.

### Decision 2 — Underscore is not syntax

**Rule:** emphasis is `*em*` and `**strong**` only. Underscore is an ordinary character. Intra-word emphasis uses the bracketed form `{*text*}`.

**Why:** `snake_case` is ordinary technical prose, and CommonMark's answer to it is seventeen interlocking emphasis rules and a delimiter stack — a construct that cannot be expressed as a context-free grammar. Removing underscore collapses the flanking rules to the two conditions in section 4.4.4, and makes the identifier safe without an escape.

### Decision 3 — Uniform escaping

**Rule:** a backslash before any character yields that character. No exception list, line endings included.

**Why:** an exception list is an "unless" clause, and invariant 2 forbids those. Uniformity here also gives the language its one visible hard line break (decision 13) for free.

**The cost, stated honestly:** this is the most dangerous entry in the migration guide, because a backslash that used to survive as a literal character now consumes the character after it. That is a silent change in output rather than an error, so a migrator has to report every occurrence.

### Decision 4 — ATX headings only, in one closed form

**Rule:** section 4.3.2.

**Why:** one sentence with a uniform fallback disposes of every case that Markdown legislates separately — seven number signs, hashtags, numbered items, the lone number sign, indentation, and closing runs — with no exception clause and no new error class. Underlined headings additionally violate invariant 4, because an underlined heading is a heading only on account of the *next* line.

**Rejected — keeping the optional closing run.** CommonMark's rule for when a trailing run of number signs closes a heading and when it does not (`# foo #` closes, `# foo #bar` does not) is the clearest available specimen of the clause invariant 2 exists to forbid.

*This decision, with 5 and 6, carries the highest muscle-memory risk in the language. See section 5's closing note.*

### Decision 5 — Fenced verbatim blocks only

**Rule:** indented code blocks are removed. Four leading spaces are four leading spaces.

**Why:** indented code is invisible in the source, interacts badly with list indentation, and forces every list rule to reason about whether an indented line is content or code. Removing it simplifies decision 6 substantially, and it retires CommonMark's three-space heading allowance as a side effect (decision 4).

### Decision 6 — Lists with arithmetic, not archaeology

**Rule:** section 4.3.5 — one bullet marker, one ordered marker, content-column alignment, no lazy continuation, adjacent items form one list, and ordered lists renumber.

**Why:** list indentation is where Markdown implementations disagree most, and nearly all of that disagreement traces to lazy continuation and to the interaction with indented code. Both are gone.

**Why one marker of each kind.** `*` and `+` existed as bullet markers for a single purpose: switching marker was how Markdown separated two touching lists. Remove them and that trick has nothing left to do, so the rule that governed it — "one marker per list" — leaves the language rather than being reworded. A rule is deleted from the reference card, not rewritten.

What that buys is measurable rather than asserted. **`+` becomes free everywhere**, having no other job in the grammar, which puts it beside the dollar sign and the underscore; it matters in ordinary prose because a diff pasted outside a fence carries a `+` on every added line. **`*` loses one of its three jobs**, and with it the precedence question between a thematic break and a bullet item whose content is `**`. **`1)` frees nothing** — a closing parenthesis still ends a link destination — and goes purely for symmetry: one bullet marker and one ordered marker state the whole of list syntax in two lines.

This is the same subtraction as decisions 4, 5, and 13. Setext headings, closing runs, indented code, and trailing-space breaks were each a second spelling of one construct. Three spellings of a bullet was the largest instance still standing.

**Rejected — alphabetic and Roman markers** (`a.`, `A.`, `i.`). They fail the addition bar in section 2.2 on its second half, because they cost plain-text clarity. `A. Smith argues that…` and `J. R. R. Tolkien wrote…` are initials and author names, ordinary in citations and bibliographies, so letters put a frequent collision at the start of a line — a form that occurs naturally in writing, promoted to structure, which is the dollar-sign failure in another costume.

They also arrive with three questions, one of which needs an "unless": `i.` is both the ninth letter and Roman numeral one, so admitting letters forces a choice that surprises somebody either way; nothing says what follows `z.`, since `aa.` is a numbering scheme rather than a marker; and with one marker per list gone, no rule says whether `a.` followed by `B.` is one list or two.

The deciding objection is a fourth. Under renumbering, a marker is a *presentation of position*, so choosing between `1.`, `a.`, and `A.` is choosing a numbering **style** — a list-style property written into the source, which is the nearest thing this language would have to a class, and decision 9 removed classes for being the only construct whose entire purpose was to let rendering vary. The want behind the proposal is real, since legal and academic outlines do letter their sub-items, and it gets the answer everything else got: nesting already distinguishes levels, and an author who needs `(a)` writes it as text, which the plain-text reader sees exactly as the rendered reader does.

**On renumbering:** the argument that this is not generated content is given in full in section 4.3.5. The point that decides it is that manufacturing content means *relocating* or *importing* it, and deriving a presentation of the position an item already has does neither. That distinction is useful beyond lists: it is the same line that admits images and excludes file includes.

### Decision 7 — No raw passthrough, at all

**Rule:** Markleft has no way to emit content in a target format. There is no raw block, no raw span, and no format escape hatch.

**Why:** invariant 5. An `iframe`, a `script`, or an `object` element renders content that is not in the source — transclusion under another name. Any document containing one falsifies "you can read the whole document in plain text" by exactly the mechanism the cardinal rule exists to exclude.

**The hole cannot be patched by rule.** A line break element adds nothing and an inline frame adds everything, and separating them requires knowing HTML semantics — an unbounded blocklist and an "unless" clause at once.

**What removal buys:** a document that is inert by construction; no lock-in to a target format, where a raw HTML block had put HTML *inside* a language claiming independence from any serialization; and a simpler decorator grammar, since the format-escape token shape leaves with it.

**What it costs, stated rather than hidden:** an escape hatch is what lets a small language *refuse* feature requests. "Markleft cannot do X" is answerable with "drop to a raw block" only while one exists. Without it, every gap presses directly on the core, which is the force that has broken the five-minute property everywhere else. Collapsible sections on GitHub are the concrete casualty.

**Rejected — keeping the raw block as a loudly-marked exception**, where the cardinal rule holds *except* inside a fence that warns the reader the plain text is not the whole story. Honest, but still an "unless" clause on the cardinal rule, and the rule is worth more intact.

### Decision 8 — Pipe tables in the core, strictly

**Rule:** section 4.3.6.

**Why:** tables are the one construct that every Markdown platform added and none specified compatibly. Putting them in the language with an exact grammar removes a whole class of platform divergence. The strictness is the point: a table that does not match is prose, not a guess.

**The compatibility budget is different here, and it is worth saying so.** Pipe tables are not in CommonMark — they are a GitHub extension — so decision 12 does not protect them. Keeping the familiar spelling is a choice made for authors' hands, not an obligation, and that is what left room to fix the three things GFM tables cannot do.

**Why the pipe opens the table.** It removes the last lookahead in the language. Under the GFM form a header row is indistinguishable from a paragraph until the rule row arrives on the following line, which made a table the only block whose type was not decidable from its own first line — precisely the property decision 4 removed setext headings for. The alternative on the table was to narrow invariant 4 to a uniform one-line confirmation window, which would have cost decision 4 one of its two arguments. Opening on the pipe removes the conflict at its source, so the invariant stands unchanged. It also makes the leading pipe mandatory by construction, which closes a prose-safety hole: with the leading pipe optional, `a | b` in ordinary prose is a valid header row.

**Why block cells.** Not line length, though that is the visible symptom. A table row *is* a line, so soft wrapping destroys the column alignment that was the format's whole reason for existing, hard wrapping is not available because the row cannot be broken, and changing one word in one cell shows the entire row as changed. This is the fourth application of one principle in this project — after unwrapped prose, `1.` numerals, and named rather than numbered anchors: **keep the unit of the source close to the unit of the edit.** A Markdown table is the only one of the four where the author has no good spelling available at all.

**Rejected — width hints in the rule row** (`|70:---|`). Not on the grounds first offered, three of which were wrong and are withdrawn in the decision record: a proportion *can* be shown in plain text, because the canonical formatter already chooses source column widths; a proportion is medium-independent in a way that page-layout measures are not; and a stated proportion is in fact more consistent across renderers than automatic sizing, not less. What decides it is that a width is a hand-maintained number that goes stale silently as content grows — the same shape rejected for list numerals and numbered anchors — and that a number in the source would actively work against whatever sizing a renderer or formatter later does well. Alignment survives the same test on a real distinction: it is closed, three-valued, and describes the content, where width is open, continuous, and describes only the display, and it is the continuity that invites the endless tuning that decision 9 removed classes to avoid. Full analysis, and what would reopen it, in `notes/table-width-hints.md`.

**Rejected — a closing rule row**, and **a blank line**, and **two blank lines**, as the table's terminator. A closing rule row strains when inline and block cells mix in one table and carries the load of remembering to write one. A blank line cannot work, since a blank line is what separates the paragraphs a block cell exists to hold. Two blank lines would work and contradicts decision 6, where blank lines never split a list.

**The cost, stated:** a list item marks every line it owns, and a block cell does not. Two solutions to one problem is a real charge against invariant 2, accepted because the pipe already does for a cell what indentation does for a list item, and because a table is a distinct editing mode in an author's mental model however it is spelled.

### Decision 9 — Decorators: one vocabulary, two positions, no reserved words

**Rule:** section 4.5.

**Why:** the language needs a way to say "this verbatim content is Rust" and "the reader arrives here", and it needs exactly one such way rather than one per construct. Two token shapes, each at most one, is the smallest grammar that does both jobs. That no word is reserved is the stronger half of the decision: it means no future word can become a directive by accident, because words have no parse-level power at all.

**Rejected — `.class`.** Classes were in the design and were removed. Four counts against them. They fail the design test in decision 15: a platform that sanitizes CSS strips the class attribute, so the mark is consumed and nothing replaces it, which is exactly the information destruction that decision forbids. They pay for nothing under the addition bar in section 2.2: no ambiguity removed, and the only gap filled is "Markdown has no styling", which is the richer-Markdown trap. They balloon the source, so the cost falls on invariant 1. And they invite mixing styling with content — the thing many people came to Markdown to escape. Being freed from styling decisions is a feature, not a deprivation, and Markdown's lack of styling is part of why its text pastes anywhere.

**Rejected — key-value attributes.** Arbitrary attributes only ever existed to feed arbitrary HTML, so decisions 7 and 9 removed each other's reason to exist.

**Rejected — a `text` label.** Blessing a second spelling of "verbatim plain text" costs the one-way-to-do-it discipline for nothing, and naming the label invites peppering prose with monospace for emphasis.

### Decision 10 — Tabs are not structural

**Rule:** spaces determine structure. A tab in a structural position is a validator warning with an automatic fix.

**Why:** a tab's width is a display setting, so structure that depends on it is structure that depends on the reader's editor. The same family as decision 13: whitespace does not carry meaning here.

### Decision 11 — Deltas from djot

**Rule:** no smart punctuation in the core; Markdown's muscle memory is kept wherever Markdown was not broken — links, block quotes, and backtick verbatim.

**Why:** djot (MacFarlane, 2022) is the nearest prior art and most of this language's syntax decisions are djot-vetted. Where we differ, we differ deliberately. Smart punctuation modifies what the author wrote, which decision 14's no-normalization clause forbids for the same reason.

**Challenged and kept — backtick verbatim.** The backtick is genuinely hostile to non-US keyboards: it is a dead key on QWERTZ, AZERTY, Nordic, and Spanish layouts, it is visually close to the apostrophe and the acute accent, and mobile keyboards bury it. The proposed replacement was the pipe. The pipe fails twice before compatibility is even considered: it is already the table delimiter, and it breaks prose-safety, since `|x| < 5`, `P(A|B)`, and shell pipelines are ordinary prose.

No substitute exists, and the reason is structural and worth keeping: **the backtick's inaccessibility and its prose-safety are the same property.** It is hard to reach because almost nobody types it in ordinary writing, and almost nobody typing it in ordinary writing is exactly what qualifies it as a delimiter. Any character reachable enough to fix the complaint is common enough to re-create the collision. The concern is answered with tooling instead — editor snippets, a line on the reference card about non-US layouts, and a validator diagnostic that catches a decorator following something that is not a verbatim span and asks whether an apostrophe or an acute accent was typed for a backtick.

Reopening this requires a candidate character, not an argument.

### Decision 12 — Compatibility with CommonMark

**Rule:** match CommonMark byte-for-byte wherever that is unambiguous and harmless. Diverge only to remove an ambiguity or to protect prose. Every divergence is documented in Appendix A, which doubles as the migration guide.

**Why:** familiarity is most of what Markdown won on, and a divergence that buys nothing is a divergence that costs adoption for free. This rule is also the reason Appendix A is short: each entry must justify its own existence.

### Decision 13 — Hard breaks are explicit; trailing whitespace is never significant

**Rule:** section 4.4.3.

**Why:** invisible syntax cannot satisfy invariant 1 or invariant 3. The deciding argument is not aesthetic: typing two spaces after a full stop is a live habit forty years after typewriters, so an ordinary sentence that happens to end a line silently becomes structure. Everything else compounds it — editors and commit hooks strip end-of-line whitespace to keep diffs clean, so the syntax is not merely invisible but actively unstable.

### Decision 14 — Unicode is the character set; UTF-8 is the encoding

**Rule:** section 4.1.

**Why UTF-8, in a specification written in 2026:** its age is the argument in its favour, and four independent lines of evidence agree. The WHATWG Encoding Standard requires it for new formats exclusively, and its encoding table is closed to additions. RFC 3629 froze UTF-8's definition permanently, so it cannot grow new cases. Deployment is roughly 99% of the web. And there is no successor and no candidate: the historical succession was UTF-1 to UTF-8, and it stopped.

**Rejected — UTF-16**, which reintroduces every problem UTF-8 removes: endianness, surrogate pairs that break naive indexing, no ASCII compatibility, and loss of self-synchronization. Its footprint is internal string APIs, none of which is an interchange format. **Rejected — UTF-32**, which costs four bytes per character to buy a fixed-width property that is a fiction, since combining marks mean one code point was never one user-perceived character.

**The durability risk is the Unicode version, not the encoding.** UTF-8 is frozen; Unicode releases annually. So the risk attaches only to rules that consult a Unicode table, and "content is never normalized" already immunizes the bulk of the language. The two rules that would have consulted a table — how anchors and labels match, and what "column" means — are defined version-independently in section 4.1, by code-point identity and code-point count. A pinned Unicode version is a dependency that ages, and it puts this definition on somebody else's release schedule.

### Decision 15 — No generated content

**Rule:** everything a reader will see is written in the source, in the order it appears there. A renderer may style, link, and lay out what is written; it may never supply text that is not.

This is invariant 5's operative detail. The rest of this entry is the part a reader needs in order to apply it.

**The boundary, precisely.** The rule is *not* that the source must look like the rendered form: `**bold**` is not bold, and a TeX fraction is not a typeset fraction. It is that the source must be **semantically equivalent to the rendered form, up to format.** Rendering may change how written content appears; it may never create content.

So the test for any future construct is one question: **is this changing the presentation of written content, or manufacturing content?** Presentation instructions stay admissible in principle — a decorator is one. Content-manufacturing constructs are closed permanently.

Reference links pass the test: the visible text is written where it appears, and only the destination lives elsewhere in the same document, a destination being metadata rather than content. A table of contents fails, because its entries appear where nobody wrote them.

**The back half: rendering is lossy on marks and lossless on content.** The front half says rendering adds nothing. The converse is that rendering *removes*: `` `E=mc^2`{math} `` becomes a typeset equation and the word `math` is gone from the page. Nothing is lost by that. The statement "this is an equation" was not deleted but **re-expressed as typography**, exactly as `**bold**` surrenders its asterisks to boldness. A decorator is the plain-text stand-in for a visual distinction the plain-text reader cannot receive, which is why these are two halves of one rule rather than two rules.

The constraint on this direction is narrow and absolute: **only marks may be consumed, and every unit of content survives.** A renderer that dropped the content a mark identified would break the rule as surely as one that invented content.

**The design test that follows:** every format word must name something a conforming renderer visibly distinguishes. A format whose rendered output is indistinguishable from undecorated content destroys information on render, and is refused. Anchors are exempt by kind rather than by exception: an anchor is navigation, metadata in the same category as a link destination, and metadata carries no content to lose.

**Worked consequence — citations and bibliographies.** Both generate text, so both are excluded from the language, and the capability moves to authoring time rather than being lost. A preprocessor or editor extension reads a bibliography file and writes the citation text and the reference list *into* the source, where they become plain text like everything else: `As [Smith (2020)](#ref-smith2020) showed…` above a references section whose entries carry `{#ref-smith2020}` anchors. The anchor is the citation key, serving as the reader's navigation target and the tool's stable identifier at once.

This is better than the alternative on the rule's own terms, because the plain-text reader sees "Smith (2020)" rather than a citation key. **The cost is restyling:** changing citation style is a flag in a citation processor and a full re-run with a large diff here. Immaterial for a document with one venue; a genuine loss for a paper shopped across journals. The trade is diffable-and-readable always, against cheap-restyling sometimes, and this project takes the first consistently.

**Worked consequence — file includes stay forbidden.** Could an image-like construct include a text file in place? No, and the rule decides it with no amendment, which is the test of an invariant rather than a carve-out. An include inserts text the source does not contain, so a plain-text reader has not read the document. The legitimate want behind it — keeping a code sample in sync with the file it came from — gets the same answer tables of contents got: a tool writes the snippet into the source. An include directive hides drift; an inlined snippet shows it in the diff.

### Decision 16 — Images are core, and the line is textual

**Rule:** `![alt](src)`, unchanged from CommonMark. Alternative text is optional; empty alternative text is legal and missing alternative text is a validator warning.

**Why an image is legal where raw HTML is not:** decision 15 governs the document's *text*, and an image contributes none. A reader of the plain source has read every word of the document.

**The bend, stated honestly.** What that reader lacks — what the picture shows — is real content, and no alternative text recovers it, because an image is worth a thousand words and an alt attribute is sized for a screen reader. So the cardinal rule *is* bent here, deliberately and in one bounded place, for two reasons that do not generalize: images are essential to clarity in most real documents, and unlike text there is no way to carry one in plain text at all. **A rule bent knowingly, at a named place, with the reason written down, is not the same thing as a rule with a hole in it** — which is why this does not license the next request.

An inline frame imports arbitrary text that becomes part of what the reader reads. An image is bounded and inert, occupies one non-textual slot, runs no scripts, and cannot embed recursively. That is the whole distinction, and it is why "no transclusion" in decision 15 means no *textual* transclusion.

**Rejected — `` `path`{image} ``,** despite fitting the decorator notation exactly. A decorator must mean the same thing in both positions, and an `image` fence is meaningless: a word that works only inline is a special case wearing the notation. Worse, a decorator labels content that is *present*, where a path references content that is *absent*. And it has nowhere to put alternative text.

**Rejected — `[alt](src){image}`,** the strongest alternative, which fails on cost rather than on principle. Principle favours it, since `![]()` is the only inline construct whose type marker precedes its content. But decision 12 forbids diverging from an unambiguous, harmless CommonMark construct for legibility alone; it would open a third decorator position, growing the rule rather than shrinking it; and it would make nearly every real document invalid CommonMark, since almost every project README carries badges or screenshots.

### Decision 17 — Anchors are positional; references are ordinary links

**Rule:** section 4.4.7.

**Why positional:** making an anchor mark a point rather than decorate a construct removes an entire class of question — what it attaches to, whether paragraphs are excluded, what happens when two constructs are adjacent — without adding anything.

**Markleft's contribution to cross-referencing is the removal of a heuristic, not the addition of a feature.** Platforms derive anchors by slugifying heading text, and every platform slugifies differently, so the same link resolves on one and dangles on another. That is the founding exhibit in navigation form. Explicit anchors delete the guess, and cost no new construct at all.

**Rejected — a bare reference sigil,** in every spelling considered. Two independent failures. It would have to **generate its own text**: rendering a bare reference means emitting either the useless raw identifier or the target's title or number, which is text pulled from elsewhere and inserted where nobody wrote it — decision 15, arriving through a smaller door than a table of contents but through the same door. And a bare `#identifier` is prose-unsafe, since `#general` and `#nowplaying` are ordinary technical prose; it would also make a hash-word text at the start of a line and syntax in the middle of one. As a consequence no digit-guard rule is needed anywhere: with references written as links, `#1` in prose is never syntax.

**A closed gap worth noting.** Decision 16 left open whether decorators needed a third position so that `## Heading {#custom-id}` could be written. This decision closes it without a new position: an anchor is valid anywhere inline content is, and a heading's text is inline content, so `## The five-minute property {#five-minute}` already works.

**The validator stays a single-document tool.** Duplicate identifiers and dangling same-document fragments are lint. A destination carrying a path or a host is a URL, and Markleft does not validate URLs — an external link can fail and is not checked either, so a cross-document fragment gets the same treatment. This keeps the tooling from needing to know about a set of documents, which would be its first multi-file commitment.

### A note on risk

Decisions 4, 5, and 6 — one heading form, fenced blocks only, strict lists — carry the highest risk in the language, and the risk is not correctness but habit. They break what a fluent Markdown author's fingers already do. Every one of them is defensible on the invariants, and none of them has yet been tested against a large body of real documents. That evidence will arrive from the migrator's change report in Phase 3, which is later than anyone would like, and it is recorded here so that when it arrives it is read as evidence rather than as complaint.

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

- **CommonMark** — the specification-as-tests methodology, and the byte-compatibility baseline in decision 12.
- **djot** — the linear-time architecture, uniform escaping, attributes (reduced here to decorators), and bracketed emphasis. **Not** its raw-content design: decision 7 removes passthrough entirely.
- **MyST and reStructuredText** — the directive concept, disciplined here into a closed extension point, and never the content-generating directive that decision 15 forbids.
- **Markdoc** — proof of demand for schema validation and documents-as-data.
- **Typst** — the standard to meet for error messages, and the Rust-plus-playground approach to credibility.
- **AsciiDoc and the Eclipse Foundation** — the technology compatibility kit concept, which is what the conformance suite is, and a cautionary timeline.

## 7. The name

Markup went up. Markdown went down. Markleft moved sideways.

Four readings, all of which describe the whole language rather than one feature:

1. **The compass.** An orthogonal move: the same size as Markdown, different rigour.
2. **What is left.** The language is what *remains* after fifteen years of curation. Most of the binding decisions in section 5 remove something.
3. **We left.** A departure from the informal era. The fourth direction stays unclaimed, left to whoever comes next to do it "right".
4. **The left margin.** The natural state of plain text — prose-safety, subliminally.

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

**When the suite and this document disagree, this document governs and the test is presumed to be the defect.** But the disagreement must be triaged, because sometimes the test is the evidence that the document is wrong. Exactly one of three things is true:

1. **The test is wrong.** It asserts something this document does not say. Fix the test. This is the common case and needs no ceremony.
2. **This document means the right thing but says it unclearly.** The test did its job: it found an ambiguity, which under invariant 3 is a defect whether or not any implementation has tripped over it. Fix the prose or the grammar so that the intended reading is the only reading. Behaviour does not change; the text that determines it does.
3. **This document is clear, and wrong.** The design is defective. That is a revision of the specification, and it goes through the change process in section 12 — never through a quiet edit, and never by letting the suite silently carry corrected behaviour on its own.

"The document governs" settles *where a change must land*, not *who was right*. It is a routing rule, not a claim of infallibility. The kilogram was redefined because realizations outran the artefact; a discrepant comparison is sometimes a finding about the definition.

**What the rule forbids is exactly one thing: a behaviour that exists in the suite and nowhere in this document.** That is how a standard quietly relocates into its tests.

### 9.4 The conformance mark

The project reserves the possibility of a lightly governed "Markleft-conformant" mark, held by the project's maintainers and by no institution. This is a reservation, not a promise; nothing about it is decided.

## 10. Copyright, licensing, and non-endorsement

**Copyright.** Crown copyright, Canada — 2026. An author is a Canadian federal public servant, so the Copyright Act vests copyright in the Crown by operation of law.

**This is not sponsorship.** No department, agency, or institution owns, funds, backs, endorses, reviews, supervises, or vouches for Markleft. It is an independent project maintained by its authors. A copyright notice arising from employment law says nothing about who supports a project, and nothing in this document or any other should be read as implying that it does.

**Licensing.** This definition and the conformance suite are licensed **CC BY 4.0**. Copy them, quote them, adapt them, implement them, and sell what you build on them. Attribution is the only condition. All code produced by the project is licensed **MIT**.

CC BY was chosen over the share-alike variant deliberately: share-alike would force anyone quoting substantial parts into their own documentation to license that documentation alike, which is friction a specification meant to be reimplemented must not impose.

## 11. Governance

Markleft is maintained by its authors and governed by no institution.

The process today is exactly this: one maintainer decides, and writes the decision down with its rationale and its rejected alternatives. That record is public, and section 5 of this document is its distillation. Describing anything more elaborate would be describing a committee that does not exist.

The trigger for something more is specific: **more than one decision-maker.** At that point the decision record becomes a request-for-comments process, the binding decisions in section 5 become numbered proposals retroactively, and contributor-facing process — a contributing guide, a code of conduct — is published alongside it. Standing that up sooner would produce ceremony and no contributors, which is a failure this project has studied in others.

## 12. Versioning and change process

**This document is versioned; the conformance suite follows it.** Each release of the suite records the revision of this document that it exercises, so any checkout of the suite states unambiguously which version of the language it is testing.

**The specification is never tagged ahead of the suite that exercises it.** A standard whose suite silently lags it is precisely the drift this project exists to prevent.

**A change to the language and its conformance examples land together.** They are opened together and merged together across the two repositories; neither lands alone. This is weaker than atomicity and will occasionally be violated by accident, which is why continuous integration checks it.

**No behaviour may exist only in the suite** (section 9.3). A reviewer of a new conformance example should ask which clause of this document determines it, and treat "none" as a finding rather than a formality.

**Every change is recorded before it is made.** A challenge to a settled decision is written down with its argument and its outcome, whether it succeeds or fails, so that a later contributor finds the analysis instead of repeating it.

## Appendix A — Deltas from Markdown

*Normative.*

The complete list of every point where Markleft departs from CommonMark is maintained in `deltas.md` and incorporated here by reference. It doubles as the migration guide.

Two entries deserve advance warning. **Uniform escaping (decision 3)** is the most dangerous change for anyone porting documents, because a backslash that used to survive as a literal character now consumes the character after it — a silent change in output rather than a visible failure. **The removal of trailing-space hard breaks (decision 13)** is the second, because the old syntax is invisible, so a reader cannot audit that conversion by eye and has to trust the migrator's report.

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

Items 1 to 4 are answered in section 4 for the first time; each closes a rider that the decision record left open, and each needs the answer confirmed before it is treated as settled. Items 5 to 11 are not answered at all.

1. **Invalid UTF-8.** Section 4.1 states that a byte sequence which is not well-formed UTF-8 is not a Markleft document. The alternatives were replacement with U+FFFD and passing bytes through; both silently alter content, which decision 3 and invariant 1 argue against.

2. **Leading byte-order mark.** Section 4.1 states that a leading U+FEFF is not part of the document text. The alternative is to treat it as content, which is the purer reading but lets an editor's byte-order mark silently break the first heading of a file.

3. **Anchor and label matching.** Section 4.1 defines matching as code-point identity, with no case-folding and no normalization. This is the recommendation of the encoding memo and is adopted here.

4. **The meaning of "column".** Section 4.1 defines it as a count of code points rather than display width, so that no rule consults a versioned Unicode property. Adopted from the same memo, and the cost — wide characters aligning by count rather than by apparent width — is stated in section 4.1 rather than buried.

5. **Tilde fences.** CommonMark admits `~~~` as an alternative fence. Section 4.3.3 describes backtick fences only. Whether tilde fences are inherited or removed is undecided; removing them would be a new entry in Appendix A, and the one-way-to-do-it discipline argues for removal.

6. **The thematic break spelling.** It is inherited unchanged, which admits a run of underscores. Decision 2 removes underscore as *emphasis* syntax only, so there is no conflict, but the interaction should be confirmed rather than assumed.

7. **Lazy continuation in block quotes.** Decision 6 removes it from lists. Block quotes are inherited from CommonMark, which permits it. Consistency argues for removing it there too; nothing has decided.

8. **The loose-and-tight list rule.** Decision 6 requires one explicit rule and does not state it. It belongs in Appendix B.

9. **Verbatim-span whitespace.** CommonMark strips one leading and one trailing space from a verbatim span when both are present. That is a rule with an exception clause, and whether it is inherited is undecided.

10. **Media type.** No media type is registered for either file extension.

11. **A malformed decorator list.** Section 4.5 states the form of a decorator list without saying what a line that fails it becomes. Two answers are available and both are defensible: the uniform fallback of section 4.2 makes the whole line paragraph text, which is what CommonMark does with ```` ```rust``` ```` and keeps the grammar honest; or the fence opens anyway and the malformed list is a validator error, which keeps a document's block structure from collapsing over a typo in a label. The cases are a token containing a backtick, two bare words, and two anchors. Raised by the backtick exclusion decided 2026-08-08; it applies to all three.

Formatter behaviour — the order of tokens inside a decorator, whether ordered-list numerals are normalized to all-`1.` or written in sequence, and whether a redundant label is removed — is deliberately not in this list. Those are questions about a tool, not about the language, and they are settled when the formatter is built.
