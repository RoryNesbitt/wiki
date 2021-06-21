---
title: Wiki.js
description: Cheatsheet for the formatting
published: 1
date: 2021-06-21T19:02:39.367Z
tags: organisation
editor: markdown
dateCreated: 2021-06-21T18:00:17.506Z
---

## Headers

## Tab {.tabset}

### Example

Headers Range in size with 1 being the biggest down to 6

### code

```
# Header 1  
## Header 2  
### Header 3  
#### Header 4  
##### Header 5  
###### Header 6
```

## Links

## Tab {.tabset}

### Example

Links can be [anywhere](/wikijs "This ironically goes nowhere") in the text

- [Links can be formatted *as a list of blocks*](/wikijs "Alt text")
- [- by adding <kbd>-</kbd> to each line *and `{.links-list}` at the end*](/wikijs "Alt text")
- [And including the description *[surrounded by \*asterix\*]*](/wikijs "Alt text")
{.links-list}

### Code

```txt
Links can be [anywhere](/path/to/page "Description").

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

```
You can insert an image like a link ![Image title](/path/to/image "Alt text" =100x50).
```

> You can include only one sides sizing to maintain aspect ratio, e.g. `100x` or `x50`, use a percentage of the availible area or use HTMLs `<img>` tag for complete control
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

```
- Unordered
- list
- items
- with
  - indented
    - items
```

### Ordered

1. Ordered
1. list
1. items
   1. with
   1. indented
      1. items

```
`1.` Use  
`1.` just  
`1.` '1.'  
`1.` and  
   `1.` three  
      `1.` spaces
```

## Code and keys

## Tab {.tabset}

### Inline code

Line containing `some code` within.

> Line containing \`some code\` within.

### Keys

Line containing shortcut such as <kbd>CTLR</kbd>, <kbd>ALT</kbd> + <kbd>DEL</kbd>.

>Line containing shortcut such as \<kbd>CTLR\</kbd>, \<kbd>ALT\</kbd> + \<kbd>DEL\</kbd>.

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

> Note the txt affter the first ``` indicates the formatting that the code should be displayed with.
{.is-info}

## lines

## Tab {.tabset}

### Example

Here is a line
___
In between text

### Code

```
Here is a line
___
In between text
```

## Tables

## Tab {.tabset}

### Example

| Tables        | Are           |   created |
| ------------- | :-----------: | --------: |
| with pipes    | to seperate   |   columns |
| hyphens       | after the     | title row |
| And colons    | to determine  | alignment |

### Code

```
| Tables        | Are           |   created |  
| ------------- | :-----------: | --------: |
| with pipes    | to seperate   |   columns |  
| hyphens       | after the     | title row |  
| And colons    | to determine  | alignment |
```

## Blockquotes

## Tabs {.tabset}

### Example

> This is a  
blockquote

> It is made with  
a <kbd>></kbd> before the first line

> It can become an info blockquote with `{.is-info}`
{.is-info}

> a success blockquote with `{.is-success}`
{.is-success}

> a warning blockquote with `{.is-warning}`
{.is-warning}

> or an error blockquote with `{.is-danger}`
{.is-danger}

### Code

```
> This is a  
blockquote

> It is made with  
a > before the first line

> It can become an info blockquote with `{.is-info}`  
{.is-info}

> a success blockquote with `{.is-success}`  
{.is-success}

> a warning blockquote with `{.is-warning}`  
{.is-warning}

> or an error blockquote with `{.is-danger}`  
{.is-danger}
```

## Content Tabs

## Tabs {.tabset}

### Example

Each of these example/code sections is a content tab

#### Tabs {.tabset}

##### Example

They can be nested, but it's not a great idea.

##### code

### code

```
## Content Tabs

## Tabs {.tabset}

### Example

Each of these example/code sections is a content tab

### code
```

> The `Tabs {.tabset}` header will not show  
Header size can vary but tab titles must be one size smaller than `Tabs {.tabset}`
{.is-info}
