---
title: Wiki.js
description: Cheatsheet for the formatting
published: 1
date: 2021-06-21T18:20:35.646Z
tags: organisation
editor: markdown
dateCreated: 2021-06-21T18:00:17.506Z
---

# Headers

## Tab {.tabset}

### Example

Headers shown bellow

### code

\# Header 1  
\## Header 2  
\### Header 3  
\#### Header 4  
\##### Header 5  
\###### Header 6

# Header 1

## Header 2

### Header 3

#### Header 4

##### Header 5

###### Header 6

## Links

## Tab {.tabset}

### Example

Links can be [anywhere](/wikijs "This ironically goes nowhere") in the text

- [Links can be formatted *as a list of blocks*](/wikijs "Alt text")
- [- by adding <kbd>-</kbd> to each line *and `{.links-list}` at the end*](/wikijs "Alt text")
- [And including the description *[surrounded by \*asterix\*]*](/wikijs "Alt text")
{.links-list}

### Code

Links can be \[anywhere](/path/to/page "Description").

```md
- [Links can be formatted *as a list of blocks*](/wikijs "Alt text")
- [- by adding <kbd>-</kbd> to each line *and `{.links-list}` at the end*](/wikijs "Alt text")
- [And including the description *[surrounded by \*asterix\*]*](/wikijs "Alt text")
{.links-list}
```

## Images

## Tab {.tabset}

### Example

You can insert an image like a link ![rnd](/rnd.png "Logo" =x50)

### Code

You can insert an image like a link \!\[Image title](/path/to/image "Alt text" =100x50).

> You can include only one sides sizing to maintain aspect ratio, e.g. `100x` or `x50`  
Or use HTMLs `<img>` tag for complete control
{.is-info}

## Text Styling

## Tab {.tabset}

### Bold

This text is **Bold**.
> This text is \*\*Bold**.

### Italic

This text is *Italic*.
> This text is \*Italic*.

### Strikethrough

This text has a ~~Strikethrough~~.
> This text has a \~\~Strikethrough~~.

### Subscript

This text is~Subscript~.
> This text is\~Subscript~.

### Superscript

This text is ^Superscript^.
> This text is\^Superscript^.

## Lists

## Tab {.tabset}

### Unordered

- Unordered
- list
- items
- with
  - indented
    - items

> \- Use  
\- a  
\- hyphen  
\- and  
..\- two  
....\- spaces

### Ordered

1. Ordered
1. list
1. items
   1. with
   1. indented
      1. items

> `1.` Use  
`1.` just  
`1.` '1.'  
`1.` and  
...`1.` three  
......`1.` spaces

## Code and keys

## Tab {.tabset}

### Inline code

Line containing `some code` within.

> Line containing \`some code\` within.

### Keys

Line containing shortcut such as <kbd>CTLR</kbd> <kbd>ALT</kbd> <kbd>DEL</kbd>.

>Line containing shortcut such as \<kbd>CTLR\</kbd> \<kbd>ALT\</kbd> \<kbd>DEL\</kbd>.

### Code blocks

```txt
Block  
of  
code
```

>\```txt  
Block  
of  
code  
\```

## lines

## Tab {.tabset}

### Example

---

### Code

> --- produces a line

## Tables

## Tab {.tabset}

### Example

| Tables        | Are           |   created |
| ------------- | :-----------: | --------: |
| with pipes    | to seperate   |   columns |
| hyphens       | after the     | title row |
| And colons    | to determine  | alignment |

### Code

| Tables        | Are          |   created |  
| ------------- \| :-----------: | --------: |  <!--FIXME: Stop this showing as emoji's on the wiki-->
| with pipes    | to seperate   |   columns |  
| hyphens       | after the     | title row |  
| And colons    | to determine  | alignment |
