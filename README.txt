POSITIONAL INDEX FOR HINDI (IR Assignment A1 - PostIndex, M2026)

Name    : Sachin Kumar
Roll No : S20240010206


1. OVERVIEW

A positional inverted index for the complete Hindi Wikipedia.

    Documents      176,041 articles
    Tokens         51.6 million
    Vocabulary     1,054,110 unique terms

For every term the index stores its document frequency (df), collection
frequency (cf), and the positions where it occurs in each document. The
vocabulary is kept in main memory (18 MB). The postings lists are stored
on disk (145 MB) and read only when a query needs them.

Written in pure Python 3 (3.8 or newer). No libraries to install.


2. DATA AND INDEX LINKS

    Hindi Wikipedia dump (Nov 2026):
    https://dumps.wikimedia.org/hiwiki/latest/hiwiki-latest-pages-articles.xml.bz2

    Full built index (287 MB, split into 7 parts):
    https://github.com/sachiniiits/IR-assignment/tree/main/built_index

    Source code:
    https://github.com/sachiniiits/IR-assignment

To get the full built index, run these three commands:

    git clone https://github.com/sachiniiits/IR-assignment.git
    cd IR-assignment/built_index
    python join_index.py

This joins the parts, checks them, and extracts the folder
S20240010206-hiwiki-index.


3. HOW TO RUN

Extract S20240010206-src.zip (or clone the repository), then open a
terminal in its src folder. Run one command per line. On Linux or macOS,
use python3 instead of python.

Quick test with the bundled sample (about 10 seconds):

    python -m indicindex build ../sample_data/hiwiki_sample.jsonl.gz -o ../index_sample
    python -m indicindex term ../index_sample हिमालय
    python -m indicindex verify ../index_sample

Build the full index from the Wikipedia dump (2 to 8 minutes):

    python -m indicindex build hiwiki-latest-pages-articles.xml.bz2 -o ../index_hiwiki

Queries (IDX is the index folder, e.g. ../index_sample):

    python -m indicindex term IDX भारत              df, cf and positions of a word
    python -m indicindex phrase IDX "भारत की राजधानी"  exact phrase
    python -m indicindex near IDX गंगा यमुना -w 3    words within 3 positions
    python -m indicindex wildcard IDX "हिमाल*"       words starting with हिमाल
    python -m indicindex search IDX "ताजमहल आगरा"     ranked search (BM25)
    python -m indicindex search IDX "~bharat"        Roman letters -> भारत
    python -m indicindex repl IDX                    interactive search

Other commands:

    python -m indicindex testcases IDX -o ../output  generate the test-case output
    python -m indicindex stats IDX                   Zipf/Heaps statistics
    python -m indicindex crawl URL -o pages.jsonl    crawl Hindi web pages

Unit tests (from the top folder, not src):

    python -m unittest discover -s tests


4. MODULES (src/indicindex)

    textproc.py      Hindi text processing: Unicode normalisation (nukta,
                     chandrabindu, native digits) and tokenisation
    wikitext.py      removes Wikipedia markup (templates, links, tables)
    corpus.py        reads Wikipedia dumps, FIRE/TREC files, JSONL, text
    crawler.py       crawls Hindi web pages (robots.txt, language filter,
                     duplicate detection)
    codec.py         VByte compression of gaps
    spimi.py         builds partial indexes in parallel (SPIMI) and merges
                     them on disk
    index_format.py  on-disk postings with skip pointers, in-memory
                     front-coded vocabulary
    builder.py       runs the whole build
    search.py        term, phrase, proximity, wildcard and ranked queries
    translit.py      converts Roman-letter queries to Devanagari
    stats.py         Zipf/Heaps statistics and compression report
    verify.py        checks the index against the original text
    testcases.py     generates the test-case output
    cli.py           command line interface


5. INDEX FILES

    lexicon.bin/.blk   vocabulary with df, cf and postings offset (in RAM)
    postings.bin       positional postings lists (on disk)
    docs.tsv           document id, title, URL, length
    docstore.bin/.idx  compressed document text (for snippets)
    meta.json          settings and collection statistics

Each postings list holds df, cf and skip pointers, then for every document
the gap from the previous document id, the term frequency and the gaps
between positions. All numbers are VByte-compressed.


6. OUTPUT AND TEST CASES

output.txt contains:

    Part A  10 rare terms with their full positional index, written as
            term [df, cf] -> < docID: pos, pos; docID: pos >.
            Each position is shown in its sentence and checked against
            the original text.

    Part B  5 test cases, all passing:
            (1) rare-term lookup   (2) phrase query   (3) proximity query
            (4) Roman-letter query (5) Unicode normalisation

The output folder inside S20240010206-src.zip also has the statistics
report, a JSON copy of the rare-term index, and the build and verification
logs.


7. RESULTS

    Build time             about 2.5 minutes (4 cores)
    Postings on disk       145 MB (3.1x smaller than raw integers)
    Vocabulary in RAM      18 MB (a plain Python dict would need 284 MB)
    Query time             a few milliseconds for most queries
    Verification           0 errors
