# Creative Kids Project Format

The Creative Kids project format is heavily based upon the [kodeklubben lesson format](https://github.com/arve0/codeclub_lesson_builder/blob/master/FORMAT.md) which is based upon the format used by [Code Club UK](https://github.com/codeclub/lesson_format).

Every project belongs under a `Project Type` directory. For example, a Scratch<sup>1</sup> project called `Dive` would exist under:
```
projects/scratch/Dive
```

Inside this directory, the project is described in a file called `contents.lr`. This is a [Lektor content file](https://www.getlektor.com/docs/content/). The file allows key:value pairs each separated by three dashes `---`.

## Project Metadata

The currently allowed metadata key:value pairs for a project are `name`, `author`, `level`, `license`, `translator`.  For example:
```
name: Dive
---
author: Rhys Moyne
---
level: one
---
license: ccbysa
---
translator: George Gently
---
```

## Project Instructions

Followed by the metadata, a project should have a field called `instructions`. This describes the project in Markdown syntax. Use {.section-name} syntax to indicate the different sections of a project. Valid sections are: `{.intro}, {.activity}, {.check}, {.challenge}, {.tip}, {.flag}, {.save}, {.try}`. To add Scratch blocks to a project use ```blocks followed by the Scratch code. For example:

```
---
instructions:

# Introduction {.intro}

Here we are including code:
```blocks
when FLAG clicked
move 10 steps
```

<sup>1 Scratch is developed by the Lifelong Kindergarten Group at the MIT Media Lab. See http://scratch.mit.edu.</sup>
