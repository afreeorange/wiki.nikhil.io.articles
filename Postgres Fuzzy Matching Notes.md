Cached [this excellent StackOverflow Answer](https://dba.stackexchange.com/a/10696) by [Erwin Brandsetter](https://dba.stackexchange.com/users/3684/erwin-brandstetter).

---

## Pattern matching operators

- [**`LIKE`**][1] (**`~~`**) is simple and fast but limited in its capabilities.
  [`ILIKE`][1] (**`~~*`**) the case insensitive variant.

- [**`~`**][2] (regular expression match) is powerful but more complex and may be slow for anything more than basic expressions.
  [`~*`][2] is the case insensitive variant.

- [**`SIMILAR TO`**][3] is just _pointless_. A peculiar blend of `LIKE` and regular expressions. I never use it. See below.

All of the above can use a trigram **index**. See below.
For left-anchored patterns, also a B-tree index using `COLLATE "C"`. Or with any other collation and the operator class `text_pattern_ops`. See below. Or even with an SP-GiST index.

Basics about [pattern matching in the manual][4].

### Related operators

- [**`^@`**][5] is "starts with" operator (for prefix matching), equivalent to the [`starts_with()`][5] function.
  Added with Postgres 11, can use an [SP-GiST index][6]. Since [Postgres 15][7] also a B-tree index using a "C" collation. See below.

- [**%**][8] is the "similarity" operator, provided by the additional module [`pg_trgm`][8]. See below.

- [**`@@`**][9] is the text search operator. See below.

## Your query

... is pretty much the optimum. Syntax won't get much shorter, query won't get much faster:

    SELECT name FROM spelers
    WHERE  name LIKE 'B%' OR name LIKE 'D%'
    ORDER  BY 1;

Or equivalent, using RegEx (slightly more expensive):

    ... WHERE name ~ '^B' OR name ~ '^D'

A bit shorter (can use the same index in modern Postgres):

    ... WHERE name LIKE ANY ('{B%,D%}')
    ... WHERE name ~ ANY ('{^B,^D}')

A regular expression with *branches* shortens the syntax some more:

    ... WHERE name ~ '^(B|D).*'

Or a *character class* (only for the case with a single character):

    ... WHERE name ~ '^[BD].*'

For bigger tables, index support improves performance by orders of magnitude.

In Postgres 11 or later the new `^@` is more convenient as we can use the unadorned prefix directly - and fast when supported with an SP-GiST index:

    ... WHERE name ^@ 'B' OR name ^@ 'D'

Or:

    ... WHERE name ^@ ANY ('{B,D}')

Since Postgres 15, the first variant can also use a B-tree index using `COLLATE "C"`.

*db<>fiddle [here](https://dbfiddle.uk/?rdbms=postgres_15&fiddle=172ba71b883c26f176488ab01a6be58d)*

## Index

If concerned with performance, create an index like this for bigger tables to support **left-anchored** search patterns (matching from the start of the string):

    CREATE INDEX spelers_name_special_idx ON spelers (name COLLATE "C");

Requires [per-column collation support added with Postgres 9.1][10].

See:

- https://dba.stackexchange.com/questions/291248/is-there-a-difference-between-text-pattern-ops-and-collate-c/291250#291250
- [B-tree index does not seem to be used?][11]

In DBs running with the "C" locale (not typical), a plain B-tree index does the job.

In older versions (or still today), you can use the [special operator class `text_pattern_ops`][12] for the same purpose:

    CREATE INDEX spelers_name_special_idx ON spelers (name text_pattern_ops);

`SIMILAR TO` or regular expressions with basic left-anchored expressions can use this index, too. **Very old** versions failed to use the index for branches `(B|D)`, character classes `[BD]`, or constructs with `LIKE ANY` or `~ ANY`.

**Or** use an expression index based on a truncated substring for columns with long strings, and repeat the same expression in queries. **Or** use the `^@` operator with a matching index.

### Trigram matching

Trigram matches or text search use special GIN or GiST indexes.

You can install the **additional module [`pg_trgm`][8]** to provide index support for any `LIKE` / `ILIKE` pattern (and simple regexp patterns with `~` / `~*`) using a GIN or GiST index.

Details, example and links:

 - https://dba.stackexchange.com/questions/2195/how-is-like-implemented/10856#10856

`pg_trgm` provides [additional operators][13] like:

- **`%`** - the "similarity" operator
- **`<%`** (commutator: `%>`) - the "word_similarity" operator in Postgres 9.6 or later
- **`<<%`** (commutator: `%>>`) - the "strict_word_similarity" operator in Postgres 11 or later

### Text search

Is a special type of pattern matching with separate infrastructure and index types. It uses dictionaries and stemming and is a great tool to find words in documents, especially for natural languages.

_Prefix matching_ is also supported:

 - https://dba.stackexchange.com/questions/157951/get-partial-match-from-gin-indexed-tsvector-column/157982#157982

As well as _phrase search_ since Postgres 9.6:

- https://dba.stackexchange.com/questions/204588/how-to-search-hyphenated-words-in-postgresql-full-text-search/204601#204601

Consider the [introduction in the manual][14] and the [overview of operators and functions][9].

### Additional tools for fuzzy string matching

The additional module [**fuzzystrmatch**][15] offers some more options, but performance is generally inferior to all of the above.

In particular, various implementations of the `levenshtein()` function may be instrumental.

### Why are regular expressions (`~`) always faster than `SIMILAR TO`?

`SIMILAR TO` expressions are rewritten into regular expressions internally. For every `SIMILAR TO` expression, there is *at least* one faster regular expression (saving the overhead of rewriting the expression). There is no performance gain in using `SIMILAR TO` **ever**.

Simple expressions that can make do with `LIKE` (`~~`) are faster with `LIKE` anyway.

`SIMILAR TO` is only supported in PostgreSQL because it ended up in early drafts of the SQL standard. They still haven't gotten rid of it. But there are plans to remove it and include regexp matches instead - or so I heard.

`EXPLAIN ANALYZE` reveals it. Just try with any table yourself!

    EXPLAIN ANALYZE SELECT * FROM spelers WHERE name SIMILAR TO 'B%';
Reveals:

    ...
    Seq Scan on spelers  (cost= ...
      Filter: (name ~ '^(?:B.*)$'::text)

`SIMILAR TO` has been rewritten with a regular expression (`~`).

[1]: https://www.postgresql.org/docs/current/functions-matching.html#FUNCTIONS-LIKE
[2]: https://www.postgresql.org/docs/current/functions-matching.html#FUNCTIONS-POSIX-REGEXP
[3]: https://www.postgresql.org/docs/current/functions-matching.html#FUNCTIONS-SIMILARTO-REGEXP
[4]: https://www.postgresql.org/docs/current/functions-matching.html
[5]: https://www.postgresql.org/docs/current/functions-string.html#FUNCTIONS-STRING-OTHER
[6]: https://www.postgresql.org/docs/current/spgist-builtin-opclasses.html#SPGIST-BUILTIN-OPCLASSES-TABLE
[7]: https://www.postgresql.org/docs/15/release-15.html#id-1.11.6.5.5.3.4
[8]: https://www.postgresql.org/docs/current/pgtrgm.html
[9]: https://www.postgresql.org/docs/current/functions-textsearch.html
[10]: https://www.postgresql.org/docs/9.1/release-9-1.html
[11]: https://stackoverflow.com/a/68385039/939860
[12]: https://www.postgresql.org/docs/current/indexes-opclass.html
[13]: https://www.postgresql.org/docs/current/pgtrgm.html#PGTRGM-OP-TABLE
[14]: https://www.postgresql.org/docs/current/textsearch.html
[15]: https://www.postgresql.org/docs/current/fuzzystrmatch.html
