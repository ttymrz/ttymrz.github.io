---
title: "kramdown markdown"
date: 1970-01-01 00:00+00:00
tags:
- markdown
---

## kramdownのmarkdownチート

[kramdown Quick Reference](https://kramdown.gettalong.org/quickref.html)
からコピペ。

## Block-level Elements - Main Structural Elements

### Paragraphs

The first paragraph.

Another paragraph

This is a paragraph  
which contains a hard line break.

### Headers


First level header
==================

Second level header
-------------------


# H1 header

## H2 header

### H3 header

#### H4 header

##### H5 header

###### H6 header


### Blockquotes

> A sample blockquote.
>
> >Nested blockquotes are
> >also possible.
>
> ## Headers work too
> This is the outer quote again.

> This is a blockquote
continued on this
and this line.

But this is a separate paragraph.

### Code Blocks

This is a sample code block.

    Continued here.

~~~~~~
This is also a code block.
~~~
Ending lines must have at least as
many tildes as the starting line.
~~~~~~~~~~~~


~~~ ruby
def what?
  42
end
~~~

### Horizontal Rules


* * *

---

  _  _  _  _

---------------


### Lists

1. This is a list item
2. And another item
2. And the third one
   with additional text


* A list item
with additional text

1.  This is a list item

    > with a blockquote

    # And a header

2.  Followed by another item

---

1. Item one
   1. sub item one
   2. sub item two
   3. sub item three
2. Item two

This is a paragraph.
1. This is NOT a list.

1. This is a list!


* Item one
+ Item two
- Item three

### Definition Lists

term
: definition
: another definition

another term
and another term
: and a definition for the term

term

: definition
: definition

This *is* a term

: This will be a para

  > a blockquote

  # A header

### Tables

| A simple | table |
| with multiple | lines|


| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|----
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=====
| Foot1   | Foot2   | Foot3
{: rules="groups"}

### Block Attributes

> A nice blockquote
{: title="Blockquote title"}

> A nice blockquote
{: .class1 .class2}

> A nice blockquote
{: #with-an-id}

{:refdef: .c1 #id .c2 title="title"}
paragraph
{: refdef}

{:refdef: .c1 #id .c2 title="title"}
paragraph
{: refdef .c3 title="t" #para}

### Extensions

This is a paragraph
{::comment}
This is a comment which is
completely ignored.
{:/comment}
... paragraph continues here.

Extensions can also be used
inline {::nomarkdown}**see**{:/}!

{::options auto_ids="false" /}

# Header without id

## Span-Level Elements - Text Modifiers

### Emphasis

This is *emphasized*,
_this_ too!


This is **strong**,
__this__ too!

This w**ork**s as expected!

### Links and Images

A [link](http://kramdown.gettalong.org)
to the kramdown homepage.


A [link](http://kramdown.gettalong.org "hp")
to the homepage.

A [link][kramdown hp]
to the homepage.

[kramdown hp]: http://kramdown.gettalong.org "hp"

A link to the [kramdown hp].

[kramdown hp]: http://kramdown.gettalong.org "hp"

An image: ![gras](img/image.jpg)

### Inline Code

Use `Kramdown::Document.new(text).to_html`
to convert the `text` in kramdown
syntax to HTML.

Use backticks to markup code,
e.g. `` `code` ``.

### Footnotes

This is a text with a
footnote[^1].

[^1]: And here is the definition.


This is a text with a
footnote[^2].

[^2]:
    And here is the definition.

    > With a quote!

### Abbreviations

This is an HTML
example.

*[HTML]: Hyper Text Markup Languag

### HTML Elements

This is <span style="color: red">written in
red</span>.

### Inline Attributes

This is *red*{: style="color: red"}.

