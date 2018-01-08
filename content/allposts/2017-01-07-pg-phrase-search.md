---
title: Postgres Phrase Search
subtitle: tie fighter
date: '2017-01-07'
slug: pg-phrase-search
---

## Tie fighter operator

One of the new features in Postgres 9.6 is the ability to do phrase text
searching. There has always been full text search available in previous
versions, but if you wanted to look for specific phrases, that required more
of an involved query. But now with version 9.6, we are able to search for
phrases where the words are grouped together. Say if we had the text `My
weekly summary text`, previous versions could allow you to search for `weekly
summary` and find it, but then you might also find a text that had `weekly`
and `summary` separated by 100 words.

With 9.6, there is a 'tie fighter' operator `<->` that allows for specifying
the number of words in between. This allows for grouping of words, known as
phrases.

Test table looking like:

```
    postgres=# select * from test;
     id |             sample_text
    ----+-------------------------------------
      1 | test string number 1
      2 | test daily string number 2
      3 | test weekly string number 3
      4 | nothing to be worried about
      5 | something not to be me caring about
```

### 9.4

```
    postgres=# select * from test where sample_text::tsvector @@ 'test & number'::tsquery;
     id |         sample_text
    ----+-----------------------------
      1 | test string number 1
      2 | test daily string number 2
      3 | test weekly string number 3
```

This shows searching for text `test` and `number` within the table.

### 9.6

```
    postgres=# select * from test where sample_text @@ to_tsquery('test & number') ;
     id |         sample_text
    ----+-----------------------------
      1 | test string number 1
      2 | test daily string number 2
      3 | test weekly string number 3
    (3 rows)
    
    postgres=# select * from test where sample_text @@ phraseto_tsquery('test number');
     id | sample_text
    ----+-------------
    (0 rows)
    
    postgres=# select * from test where sample_text @@ to_tsquery('test <2> number') ;
     id |     sample_text
    ----+----------------------
      1 | test string number 1
    (1 row)
    
    postgres=# select * from test where sample_text @@ to_tsquery('test <3> number') ;
     id |         sample_text
    ----+-----------------------------
      2 | test daily string number 2
      3 | test weekly string number 3
    (2 rows)
```

Here you can see the tie fighter operator in use, being able to specify the
number of words between the two specified words.

That's a very quick and short summary of phrase searching for now, I haven't
been using it extensively yet but there is also `tsvector` editing functions
to fine tune editing features.