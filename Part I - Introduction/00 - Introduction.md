# Chapter 0 : Introduction

As a software engineer, it is hard to keep up with all existing field since technology grows fast, faster
than you learn. Back at the time when i was a student, i would never imagine having free SSL certificate.
Neural network in machine learning wasn't the reference in research as well as in the industry. In fact
machine learning wasn't either. Developping short pieces of software and plug them together in order to
make a complex microservice oriented architecture, using container, the cloud. That was only 5 years ago.

You get specialized in some field(s), draw your career path, and as long as you progress through it, some
concepts get lost in road. Low level programming is one of them, and when you hear about programming a
fancy gaming console emulator, this sounds mystical to you. Well that was the case for me. But what is great
about mystical stuff, is that working on it is really, really fun. It turns out that you probably already
have all the required knowledge to understand how an emulator works, the only thing is that you never
used that knowledge in a concrete project before.

## Yet another gameboy emulator

The idea here is to develop your own gameboy emulator, which is a great project to improve your
programming skills. First of all, the background is quite enjoyable, programming an emulator for the
first time as said earlier is really fun. But most of all it will help you to grow your understanding
in how a computer works (or at least it will help you remember how it does). Plus, emulation requires
good understanding of lot of fundamental concept such as algorithmic and datastructure, time and space
complexity, low level programming, system design, and more. Since emulation field requires performance
and well written code to keep up, it will sharp your software engineering skills.

Another good thing is the choosen emulated platform : the gameboy. As we will see in the next chapter,
this target is at the same time small enough to suceed in achieving it, and big enough to walk through
many skills.

## Audience

This books is a perfect match for intermediate software engineer whom want to assess themselves, as well
as improving their existing skills. We assume here that you are familiar with at least one programming
language, and not familliar with low level programming (but it won't be a problem if you are). And since
*Java* programming language is used here, it would be better if you have oriented object knowledge.

## Let's start !

But before starting the job, we need to define which tools we will use to realize it.

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

\newpage
