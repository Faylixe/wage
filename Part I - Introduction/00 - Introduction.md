# Chapter 0 : Introduction

TODO : Introduce the project itself.

## Let's start !

### Programming language

Choosing a programming language for developping an emulator is not harmless. Indeed, lots of aspects are
involved in this choice :

- _Performance_, that sounds obvious, but emulating a game console requires high level performances. Of
course, the gameboy hardware specifications aren't that big, versus modern computer resources. We can
totally assume that we can exploit such hardware resources gap, but it won't work for other target so
we won't.
- _Data typing_, since you will structure data as the original hardware will (or at least try to do it),
you need a fine control over primitive types used. Bit endianess, signed, unsigned, memory footprint,
all those aspects will impact your code.
- _Confort_, this point comes last but is maybe the most important one. As you are supposed to not be
familiar with low level programming, you don't want to focus on language aspect, but rather on low
level concept. Picking a language in which you are really confortable, will make you develop
quicker, and doesn't get you discouraged when implementing any features. On the other hand, programming
an emulator turns out to be an outstanding experience that will more than everything improve your coding
skill in the language you will choose. So weigh pros and cons about improving a target language experience
versus time it will take you to do the job, and motivation upkeep.

This list is non-exhaustive as other items could be considered, such as the available tooling or existing
implementation that can be used as reference. Final choice was arrested with **Java** (version 8).

### Tooling

#### Unit testing

TODO : JUnit5 introduction.

#### Continuous integration

TODO : CircleCI introduction.