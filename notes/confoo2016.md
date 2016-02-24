---
layout: default
title: Notes From Confoo 2016 
---

These are some notes I took for myself at Confoo this year, in order
of the order of the talks. They may not make sense to anyone except
myself.

# Day 1

## 12 reasons your API sucks
- Aim for < 5 min "time to hello world"
- Include link to documentation in error responses, to save
  users from googling

## Rasmus's Talk on PHP7
 - the room was ridiculously small for this talk
 - PHP 7 has much lower memory requirements, not just raw speed
   (arrays are copy-on-write from the opcode cache now)
 - JIT is coming in 7.1 but wasn't ready for 7.0
 - Use Rasmus's phan tool for static analysis. Type errors are still a 
   runtime exception since PHP7 is optionally typed, variable types are
   not necessarily known and it can't always be a parser error.
   (it's not being built in to the default php linter because optional
   typing makes it too complicated)
 - There's a PHP extension to access the AST in PHP7, it'll
   probably be in core in PHP 7.1

## Introduction to Spark
 - Spark seems useful for Map/Reduce algorithms, but I'm not
   sure if I have much need for my day to day work.

## PHPUNIT 5, PHP 7, and Beyond
 - Moving to semantic versioning made is easy to drastically
   shorten release cycle (was ~1.5 years, now features are released
   every 8 weeks)
 - Projects should make sure they document their official 
   release/support schedule
 - PHP7 is a major improvement over 5

 ## You Don't Need Javascript For That!
  - "label for" attribute gives focus to the input element, it isn't just
    for screen readers.
  - datalist allows autocomplete like functionality without javascript
  - there are :valid :invalid etc pseudo-selectors for styling forms with
    HTML5 elements
  - there is a pattern=regex attribute for HTML5 elements
  - flexbox is very powerful for layout, need to look into more.
  - CSS3 animations are very powerful for design and layout without
    any javascript

