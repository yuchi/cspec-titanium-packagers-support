# CSPEC 4 - Titanium Packagers Support

This [CSPEC][] is for adding official packaging abilities on Titanium. More precisely its aim is to define how [npm][]-like semantics will work on Titanium.

[CSPEC]: https://github.com/appcelerator/cspec
[npm]: https://www.npmjs.com

## Audience

The primary audiences for this specification are both final Titanium developers and authors of packages.

## Goals

The *raison d'être* of this specification is to define

1. how final users will be able to access the vast repository of already built packages available on *npm*;
2. how package authors will be able to use the strong dependency management of *npm* in their work;
3. what the experience of coming from npm-compatible environments to Titanium will be enhanced.

The *rationale* behind it is simple:

1. the *npm Registry* is full of useful packages that both final developers and package authors would like to use;
2. we really need some better nested dependencies management in Titanium;
3. it will make the transition to Titanium easier and smoother;
4. we will get an overall higher quality platform by simply ‘enlarging it’.

## Scope

> This section is not formally defined by the CSPEC template, but required for the discussion to be meaningful and focused.

This document **wants to** talk about
- the new philosophy to be embraced;
- the technical challenges and available solutions;
- the resulting ergonomics in terms of
  - package authoring,
  - cross-environment package authoring and consuming,
  - cross-platform package authoring and consuming,
  - and final product development.

This document will **defer to other CSPECs and discussions** regarding:
- the porting of the Node.js environment;
- the ethics-related issues concerning *npm*, packages immutability and *npm Inc.*;
- the challenges of closed-source free/pay-walled distribution of packages;
- the deep implications and collisions between other (native?) colliding package managers;
- the implications this has regarding the packaging and distribution of Hyperloop modules.

## Terminology

> This section is not formally defined by the CSPEC template, but required for the discussion to be meaningful and focused.

- **modules** are akin to *compilation units* and are the required actor in a *composable architecture*, where the codebase is split in logical sections.

  In JavaScript-land they can in fact `import` or `require()` other modules and `export` some values.

- **packages** are a *pack of resources* most probably created for distribution. They usually include one or more modules.

  In the scope of *npm* they can also define both **run-time dependencies** and **build-time dependencies** as well as other dependencies this packages “works well with”, **peer dependencies**.

Currently the term *Titanium Modules* encompassed both the meaning of *modules* and *packages*, mostly because a Titanium Module is in fact a single module with eventual resources (such as assets).

## Description

The ability for developers to share libraries and packaged functionalities has been tackled by almost every environment. Most *Package Management* solutions usually define (or require) four different things:

1. A list of semantics that the system will adhere too.
2. A ‘protocol’ or ‘contract’ for the targeted system to work with, usually a file-system structure.
3. A registry, either centralized or distributed, that hosts the list of packages available for download.
4. A tool or a toolchain that helps in enforcing both the semantics and the protocol, as well as helping accessing the registry.

In the case of JavaScript the most successful solution is *npm* (which is by number the most successful package manager of all languages and environments). We can define those four requirements for *npm* as follows:

#### Npm and Node.js semantics

By using a very small trick provided by the Node.js *module resolution algorithm*—that is the ability of *node_modules* folders to be nested—*npm* allows package authors to define how their package depends on other packages with modest freedom. In fact you are guaranteed that the version (or version range) of the package you depend on will always be satisfied even if 🅐 the consumer depends on another version or 🅑 the consumer depends on a dependency which depends on another version.

Also, having Node.js also great ergonomics for writing scripts and command-lines, *npm* gives authors a way to define a list of dependencies that is required not for the package to work correctly, but for the package to be *developed* correctly for example providing pre-processing or ‘linting’ functionalities. Those are in fact called **development dependencies**.

#### The Node.js’ contract, the module resolution algorithm

As stated before, all of this works because the resolution algorithm in Node.js is caller-dependent.

That means that in the next Fig. 1 the result from calling `require('underscore')` inside *MyApp/index.js* is different from that same call inside *MyApp/node_modules/backbone/index.js*.

>     • MyApp/
>       ├─ index.js   >────────────────── require('underscore')
>       ├─ package.json                            ═════╤════
>       └─ node_modules/                                │
>          ├─ backbone/                                 │
>          │  ├─ index.js   >── require('underscore')   │
>          │  ├─ package.json            ═════╤════     │
>          │  └─ node_modules/                │         │
>          │     └─ underscore/ ╌╌─┰──────────╯         │
>          │        ├─ index.js  <━┛                    │
>          │        └─ package.json                     │
>          └─ underscore/ ╌╌─┰──────────────────────────╯
>             ├─ index.js  <━┛
>             └─ package.json
>
> Fig. 1 — A sample Node.js directory structure with the resolution visualized.

## Proposal

The proposal would be to ....

## Timeline

The goal of this proposal is to ...

## Status

DATE - Initial Draft

## Legal Stuff

This proposal is non-binding and may not be implemented, may be implemented partially, differently or not at all. Any intellectual property developed as part of this proposal is owned and Copyright &copy; 2015 by Appcelerator, Inc. All Rights Reserved.

For more information about what a CSPEC is, please visit the [Community Specifications & Proposals Repo](https://github.com/appcelerator/cspec).
