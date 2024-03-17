---
title: Metaspec
description: A specification for SUAVE specifications
date: 2024-02-15
editors:
    - name: Louis Thibault
      email: louis@flashbots.net
      github: lthibault
      affiliation:
          name: Flashbots
          url: https://flashbots.net/
maturity: stable
---

# Metaspec

![draft](https://img.shields.io/badge/status-draft-yellow.svg?style=flat-square)

Specifications have two competing goals.  On the one hand, they should be easy for humans to write using natural language, and on the other, they should be precise and unambiguous.  The SUAVE Metaspec is a collection of standards for writing SUAVE specifications that aims to strike a balance between these two opposing goals, such that specs can be written in prose while nonetheless supporting sophisticated testing, analysis and rendering pipelines.  This document specifies how SUAVE specifications should be written, and defines an explicit lifecycle for their creation, review, release and maintenance.

## Structure

SUAVE specifications are Markdown documents written in natural language, with the goal of being easily understandable by humans.  However, to ensure precision and avoid ambiguity, the following guidelines must be followed. 

### Frontmatter

A spec must being with frontmatter. Frontmatter is a preamble to the document placed right at the start, delimited with --- and containing YAML data. The frontmatter for the present spec looks like this:

```
---
title: Metaspec
description: A specification for SUAVE specifications
date: 2024-02-15
editors:
    - name: Louis Thibault
      email: louis@flashbots.net
      github: lthibault
      affiliation:
          name: Flashbots
          url: https://flashbots.net/
maturity: stable
---
```

The frontmatter MUST contain a title field, which is the title of the specification. In addition, it MUST include a description field with the description of the specification, as well as a date field containing the date of creation and a maturity field (defined below).  The editors fields is RECOMMENDED to facilitate discussions and corrections.

The maturity field indicates the document's stability. This list is subject to revision, but the maturity levels currently supported are:

- ![draft](https://img.shields.io/badge/status-draft-yellow.svg?style=flat-square)
- ![stable](https://img.shields.io/badge/status-stable-brightgreen.svg?style=flat-square)
- ![deprecated](https://img.shields.io/badge/status-deprecated-red.svg?style=flat-square)

The date field is a YYYY-MM-DD specification of the last dated change to the spec.

### Title and Sections

A spec MUST have exactly one title, which is to say an h1 heading.  The title MUST appear at the top of the Markdown document.

Sections in a spec are nested by using various heading depths.  For the avoidance of doubt, "section" SHALL refer to any hn>1 heading, to distinguish it from the title.  Content between the title and the first section of a document is considered to be the abstract for the document.  Rendering pipelines SHOULD incorporated the abstract into the header content.

The first line of the abstract MUST contain a status badge indicating maturity.

### Writing Testable Specifications

Specifications MUST make use of [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) keywords such as MUST and MUST NOT in order to express conformance expectations as testable assertions.  No markup is required, but as per the RFC 2119 specification, these keywords MUST be written in capital letters to distinguish them from informal use.  Use of RFC 2119 is important because the Metaspec explicitly aims to support [automated conformance tests](https://www.w3.org/TR/test-methodology/#dfn-testable-assertion).

Where possible, it is RECOMMENDED to exclusively use `SHALL`, `MUST`, `SHOULD`, `MAY` and their negations because optional behaviors tend to produce complex specifications with non-obvious edge cases.  This is a guideline and not a hard rule, and specifications should favor accuracy and generality over dogma.

>Specification authors are highly encouraged to familiarize themselves with the W3C's [A Method for Writing Testable Conformance Requirements](https://www.w3.org/TR/test-methodology/).
