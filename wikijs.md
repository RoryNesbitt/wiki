---
title: WikiJS
description: Reminders for how to format the markdown
published: 1
date: 2021-06-21T00:25:03.429Z
tags: organisation
editor: markdown
dateCreated: 2021-06-20T23:40:46.623Z
---

## Headers

# H1

## H2

### H3

#### H4

##### H5

###### H6

> `# H1`
> `## H2`
> `### H3`
> `#### H4`
> `##### H5`
> `###### H6`

## Links and images

Links can be [anywhere](# "This ironically goes nowhere") in the text with 
> \[Link text](/path/to/page "Description").

or can insert an image with ![rnd.png](/rnd.png "Logo" =x50)
> \!\[Image title](/path/to/image "Alt text" =100x50).

> You can include only one sides sizing to maintain aspect ratio, e.g. `100x` or `x50`
Or use HTMLs `<img>` tag for complete control
{.is-info}


- [Links can be formatted](# "Alt text")  
- [as a list of blocks](# "Alt text")
- [- by adding <kbd>-</kbd> to each line](# "Alt text")  
- [and `{.links-list}` at the end](# "Alt text")
{.links-list}

## Text Styling

This text is **Bold**.

> This text is \*\*Bold**.

This text is *Italic*.

> This text is \*Italic*.

This text has a ~~Strikethrough~~.

> This text has a \~\~Strikethrough~~.

This text is~Subscript~.

> This text is\~Subscript~.

This text is ^Superscript^.

> This text is\^Superscript^.

## Lists

- Unordered
- list
- items

> \- Unordered
\- list
\- items

1. Ordered
1. list
1. items


> `1. All`
`1. numbers`
`1. are`
`1. one`


## Code and keys

Line containing `some code` within.

> Line containing \`some code\` within.

Line containing shortcut such as <kbd>CTLR</kbd> <kbd>ALT</kbd> <kbd>DEL</kbd>.

>Line containing shortcut such as \<kbd>CTLR\</kbd> \<kbd>ALT\</kbd> \<kbd>DEL\</kbd>.

```
Block
of
code
```


>\```
Block
of
code
\```

---

> --- produces a line

## Tables

| Tables        | Are           |   created |
| ------------- |:-------------:| ---------:|
| with pipes    | to seperate   |   columns |
| hyphens       | after the     | title row |
| And colons    | to determine  | alignment |


>| Tables        | Are          |   created |
| ------------- \|:-------------:| ---------:|
| with pipes    | to seperate   |   columns |
| hyphens       | after the     | title row |
| And colons    | to determine  | alignment |

## Blockquotes

> This is a 
blockquote

> \> It is made with
a <kbd>></kbd> before the first line

> It can become an info blockquote with `{.is-info}`
{.is-info}

> a success blockquote with `{.is-success}`
{.is-success}

> a warning blockquote with `{.is-warning}`
{.is-warning}

> or an error blockquote with `{.is-danger}`
{.is-danger}
