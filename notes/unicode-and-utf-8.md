# Memo: is UTF-8 still the right encoding to specify?

*Non-normative. Status: checked 2026-08-07 against current practice; decision 14 unchanged. Records the external evidence for "Unicode is the character set; UTF-8 is the encoding" so a later session does not re-open the question from scratch, and separates the part that is settled — the encoding — from the part that is not: which Unicode version the open riders depend on. Definition material for the decision 14 section.*

## The question

Decision 14 fixes UTF-8 as the encoding of a Markleft document. UTF-8 is a 1993 design. A standard being written in 2026 should be able to say why it is not specifying something newer, and whether it is specifying something that will still be right in twenty years.

## Finding: UTF-8 is the terminal answer, not an interim one

Four independent lines of evidence, none of which depends on the others:

- **The WHATWG Encoding Standard requires UTF-8 for new formats, exclusively.** Its encoding table exists only so browsers can decode legacy content, and it is closed to additions. A format defined today that specifies anything else is non-conforming by construction.

- **UTF-8's definition is frozen.** RFC 3629 restricted it permanently to U+0000–U+10FFFF and removed the five- and six-byte forms of the original design. The encoding cannot grow new cases; there is nothing left in it to revise. IETF policy (BCP 18) makes it the default for new protocols.

- **Deployment is effectively total.** Roughly 99% of web pages are served as UTF-8, which puts it past the point where a competitor could accumulate enough tooling to matter.

- **There is no successor and no candidate.** The historical succession was UTF-1 to UTF-8, and it stopped. No standards body has a replacement under consideration.

"Most modern" turns out to be the wrong axis. UTF-8's age is the argument in its favour: a specification wants to name a target that has stopped moving, for the same reason the SI defines the metre by a constant rather than by the best current apparatus.

## Why not the alternatives

**UTF-16** reintroduces every problem UTF-8 removes. It has endianness, so it needs a byte-order mark or out-of-band agreement. Surrogate pairs mean a code unit is not a code point, so the naive index is wrong in exactly the cases that matter. It is not ASCII-compatible, so no existing byte-oriented tool works on it unmodified. And it loses self-synchronization: given a byte in the middle of a UTF-8 stream you can find the character boundary locally, which is what makes damaged or partial input recoverable. UTF-16's footprint is internal string APIs — Java, JavaScript, Win32 — where it was inherited from UCS-2 rather than chosen, and none of those is an interchange format.

**UTF-32** costs four bytes per character to buy a fixed-width property that is a fiction. Combining marks and grapheme clusters mean one code point was never one user-perceived character, so the indexing problem UTF-32 claims to solve is still there one level up. Nothing that matters gets easier, and every file quadruples.

Neither is a serious candidate for interchange, and interchange is the whole of what a document format does.

## The real durability risk is the Unicode version, not the encoding

UTF-8 is frozen; Unicode is not. Each release adds code points and revises derived property tables. So the future-proofing question does not attach to the encoding at all — it attaches to any rule in the specification that consults a Unicode table. That reframing is the useful output of this check.

Decision 14's **content is never normalized** clause already immunizes most of the language: verbatim text needs no tables, no case mapping, and no version. Two of decision 14's open riders do consult tables, and both are one-meaning-property questions across *time* rather than across parsers:

- **Anchor and label matching.** If matching case-folds, whether two labels are the same input can change between Unicode releases. A document that had one parse in 2026 would have a different one later, under a specification nobody edited.

- **What "column" means for decision 6.** Content-column alignment defined in terms of display width depends on East Asian Width, which is a versioned property with entries that have moved between releases.

**Recommendation for the definition — adopted:** `definition.md` §4.1 now defines both version-independently, exactly as recommended here — code-point identity for label matching, code-point count for columns — held as Appendix D items 3 and 4 pending confirmation, rather than pinning a minimum Unicode version. A pinned version is a dependency that ages, and it puts the specification on someone else's release schedule; the argument is the one in `notes/normative-hierarchy.md`, that a definition written to be independent of its realizations outlives all of them. The cost is that a language with wide characters aligns by count rather than by apparent width, which is worth stating plainly in the definition rather than burying.

The other two riders — byte-order mark handling and what a conforming parser does with invalid UTF-8 — are unaffected by version drift. They are one-time choices about a frozen encoding; this memo did not settle them, and `definition.md` §4.1 now answers both (a leading U+FEFF is not document text; invalid UTF-8 is rejected, not repaired), held as Appendix D items 1 and 2 pending confirmation.

## Sources

- [WHATWG Encoding Standard](https://encoding.spec.whatwg.org/)
- [W3C Internationalization: Character encodings](https://www.w3.org/International/docs/encoding/)
- [RFC 3629 — UTF-8, a transformation format of ISO 10646](https://www.rfc-editor.org/rfc/rfc3629)
- [UTF-8 — Wikipedia](https://en.wikipedia.org/wiki/UTF-8), for the deployment figure and the UTF-1 history
