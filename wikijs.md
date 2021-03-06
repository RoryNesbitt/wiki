---
title: Wiki.js
description: Cheatsheet for the formatting
published: 1
date: 2021-06-21T20:45:17.964Z
tags: organisation
editor: markdown
dateCreated: 2021-06-21T18:00:17.506Z
---

## Headers

## Tab {.tabset}

### Example

Headers Range in size with 1 being the biggest down to 6

### code

```md
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

```txt
You can insert an image like a link ![Image title](/path/to/image "Alt text" =100x50).
```

> You can include only one sides sizing to maintain aspect ratio, e.g. `100x` or `x50`, use a percentage of the availible area or use HTMLs `<img>` tag for complete control
{.is-info}

## Text Styling

## Tab {.tabset}

### Bold

This text is **Bold**.

```txt
This text is **Bold**.
```

### Italic

This text is *Italic*.

```txt
This text is *Italic*.
```

### Strikethrough

This text has a ~~Strikethrough~~.

```txt
This text has a ~~Strikethrough~~.
```

### Subscript

This text is~Subscript~.

```txt
This text is~Subscript~.
```

### Superscript

This text is ^Superscript^.

```txt
This text is^Superscript^.
```

## Lists

## Tab {.tabset}

### Unordered

- Unordered
- list
- items
- with
  - indented
    - items

```txt
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

```txt
`1.` Use  
`1.` just  
`1.` '1.'  
`1.` and  
   `1.` three  
      `1.` spaces
```

### Tasks

- [ ] Task
- [ ] lists
  - [ ] are
    - [x] like
- [x] ToDos

```txt
- [ ] Task
- [ ] lists
  - [ ] are
    - [x] like
- [x] ToDos
```

> Task lists are not native to Markdown but will work on both Wiki.js and Github
{.is-info}

## Code and keys

## Tab {.tabset}

### Inline code

Line containing `some code` within.

```txt
Line containing `some code` within.
```

### Keys

Line containing shortcut such as <kbd>CTLR</kbd>, <kbd>ALT</kbd> + <kbd>DEL</kbd>.

```txt
Line containing shortcut such as <kbd>CTLR</kbd>, <kbd>ALT</kbd> + <kbd>DEL</kbd>.
```

### Code blocks

```txt
Block  
of  
code
```

````txt
```txt  
Block  
of  
code  
```
````

> The txt after the first <kbd>```</kbd> indicates the formatting that the code should be displayed with.
{.is-info}

## lines

## Tab {.tabset}

### Example

Here is a line
___
In between text

### Code

```txt
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

```txt
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

```txt
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

````txt
## Content Tabs

## Tabs {.tabset}

### Example

Each of these example/code sections is a content tab

### code
```
Code goes here
```
````

> The `Tabs {.tabset}` header will not show  
Header size can vary but tab titles must be one size smaller than `Tabs {.tabset}`
{.is-info}

## Diagrams

## Tabs {.tabset}

### Mermaid

```mermaid
sequenceDiagram
    Alice ->> Bob: Hello Bob, how are you?
    Bob-->>John: How about you John?
    Bob--x Alice: I am good thanks!
    Bob-x John: I am good thanks!
    Note right of John: Bob thinks a long<br/>long time, so long<br/>that the text does<br/>not fit on a row.

    Bob-->Alice: Checking with John...
    Alice->John: Yes... John, how are you?
```

### Mermaid Code

````txt
```mermaid
sequenceDiagram
    Alice ->> Bob: Hello Bob, how are you?
    Bob-->>John: How about you John?
    Bob--x Alice: I am good thanks!
    Bob-x John: I am good thanks!
    Note right of John: Bob thinks a long<br/>long time, so long<br/>that the text does<br/>not fit on a row.

    Bob-->Alice: Checking with John...
    Alice->John: Yes... John, how are you?
```
````

### Plantuml

```plantuml
Bob->Alice : hello
```

### Plantuml Code

````txt
```plantuml
Bob->Alice : hello
```
````
