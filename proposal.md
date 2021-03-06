ergo proposal
================

# The problem

> What problem do you want to solve? Why is it a problem? Who does it
> affect? What will solving the problem enable? This section should
> include a brief summary of existing work, such as R packages that may
> be relevant. If you are proposing a change to R itself, you must
> include a letter of support from a member of R core.

R is an amazing interpreted language, giving a flexible and agile
foundation for data science. It is however sometimes slow. This is
typically not a problem these days with the ubiquity of `Rcpp`, making
it possible and relatively easy to connect R and a compiled language,
namely C++.

[Go](https://golang.org) is an open source programming language that
makes it easy to build simple, reliable and efficient software. It is
sometimes said to be the language C++ should have been, in particular if
it did not carry a strong commitment to backwards compatibility to C and
a taste for complexity.

`Go` is a beautiful and simple language. Its standard library is one of
the most impressive for a programming language. It comes with
concurrency built in, which makes it trivial to write concurrent code,
that includes (but is not limited to) running code in parallel.

The static site generator [hugo](https://gohugo.io) and the
containerization plaform [docker](https://www.docker.com/) are examples
of systems that are built on go.

There currently is no solution to easily connect R and go, i.e. invoke
go code from R, and this is what this project is about. We want R
packages to be able to leverage existing or original go code. We
admittedly do not have specific use cases in mind, but at the same time
it would have been impossible to imagine the importance of Rcpp when it
was first developped.

Having go as an alternative high performance language will open
interesting avenues for R package developpment.
[cgo](https://golang.org/cmd/cgo/) gives us a C api we can use together
with the R c api. [Prior work](https://purrple.cat/tags/go/) from Romain
established that this a viable pursuit.

# The plan

> How are you going to solve the problem? Include the concrete actions
> you will take and an estimated timeline. What are likely failure modes
> and how will you recover from them? The Team: Who will work on the
> project. Briefly describe all participants, and the skills they will
> bring to the project.

The plan is to develop `ergo` as a *development time dependency* that
facilitates the generation of code to interface R and go via their
respective C apis. From the point of view of the user of `ergo`, the
workflow will be: - write code in idiomatic go - call that code from R

The role of `ergo` is to hide the C layer entirely, so that users can
focus on writing go and R code. In a way, this is similar to the feature
of [Rcpp
attributes](https://cran.r-project.org/web/packages/Rcpp/vignettes/Rcpp-attributes.pdf)
but it goes further in that event though attributes generate code, but
that code relies as Rcpp as a runtime dependency. We’d like to avoid
that for a variety of reasons that are outside the scope of this
document. Once `ergo` has generated the code, the target package is
autonomous.

The step zero will be to extend [prior
work](https://purrple.cat/tags/go/) to validate the possibility of
passing back and forth basic R objects (vectors, lists). Basic
functionality includes ways to hand R vectors as go slices, and create R
vectors from within go, which is essentially two sides of the same coin.
Later on, we might rely on the new ALTREP features of R to implement
synergy between go arrays and slices (i.e. things allocated and managed
by go) and R vectors.

Once we have enough confidence, we can move on to the first step.
Automatic generation of boiler plate code to connect all basic R vector
types and their associated scalar types. The
[parser](https://golang.org/pkg/go/parser/) package for go gives us an
abstract syntax tree of go code, we’ll use that together with a set of
conventions to drive the code generation. Rcpp uses the decoration (aka
attribute) `[[Rcpp::export]]` to identify which functions to expose to
the R side. Go has this (confusing at first, but so simple and powerful)
convention that functions that start with capital letters are public and
the rest are private to the package. We might expand on that in the
context of exposing go functions to R.

At that stage, the project will need community adoption. The second step
will involve promotion and development of use cases that demonstrate the
use of `ergo`, this will without doubt reveal needs we had not planned
for, things like how I use this particular go package, how do I connect
that particular R structure with go, …

A dedicated blogdown/hugo powered site, will be use throughout the
various phases of the project, perhaps with separate sections to isolate
the technical issues and feedback related to the development of `ergo`
itself, from use case material, perhaps featuring invited posts from the
community.

Initial feedback and casual twitter probing revealed that the R
community seems to like the idea of an R and Go integration. We need to
keep the community excited and on board. Lack of feedback from the
community would have to be identified as a failure mode for the project.

Go is currently not one of the languages supported by R, which might
create friction down the line. In early phases of development, we can
get away with having instructions about the tools needed to use `ergo`
and the tools needed to use packages that include code generated by
`ergo`. But ultimately a package with go code should be as easy to
install as any other R package. The third step of the plan will consider
the distribution of such packages, can we use CRAN ? If not, what else ?
Do we need code inside the base R distribution, i.e. something similar
to `R CMD javareconf` to help mitigate these issues.

## Team

[Romain François](https://twitter.com/romain_francois/) is co-author of
Rcpp that now has more than 1000 downstream dependencies, the experience
of having written Rcpp is extremely valuable in the pursuit of this
project. Romain pioneered integration of Go into R in a series of blog
post in the 2017 summer. He has more than 12 years professional of
experience with R and a string commitment to open source development.

[Kirill Müller](https://twitter.com/krlmlr) applied the techniques
developed by Romain for successful completion of the “Joint profiling of
R and native code” project, and successfully completed two rounds of the
“DBI” series of projects. He has more than 20 years of software
engineering experience, and has been an user of and contributor to the R
ecosystem since 2012.

## Commitment

Romain will take the lead on the project and spend a minimum of one day
per week on this project (typically friday).

Kirill …

# Project Milestones

> Outline the milestones for development and how much funding will be
> required for each stage ( as payments will be ties to project
> milestone completion ). Each milestone should specify the work to be
> done and the expected outcomes, providing enough detail for the ISC to
> understand the scope of the project work.

## Case study. $8k

Continue [Romain’s](https://purrple.cat/tags/go/) exploratory work to
evaluate the possibility of calling go functions taking a diverse set of
R objects as inputs: vectors of basic types, and associated scalars.

At that stage, this only involves manual production of the code that
`ergo` would end up generate in the long term. Going through several
cases will give us a feel of the type of code to generate later.

This is also when we can decide on a set of conventions for things like
where to put go code within the standard R package directory structure.

## First iteration. $30k + expenses

This is the main lump of technical work on this project. We go from the
code we manually generated in the case study and abstract that into code
that is generated automatically based on the signature of the go
function.

This is when `ergo` starts to materialise as an R package, a development
time R package. Other examples of development time packages are
`roxygen2`, `usethis` and `devtools`.

The end user of a package that contains code generated by `ergo` does
not need to have `ergo` installed.

## Second iteration. $15k + expenses

In this second iteration, we’ll react to the community feedback, apply
lessons learned, and tighten the distribution of packages with code
generated by `ergo`.

## Third iteration. $15k + expenses

More integration of community feedback. Perhaps working with Rstudio
towards IDE support.

# How Can the ISC help

> Please describe how you think the ISC can help. If you are looking for
> a cash grant include a detailed itemised budget and spending plan. We
> expect that most of the budget will be allocated for people, but we
> will consider funding travel, equipment and services, such as cloud
> computing resources with good justification. If you are seeking to
> start an ISC working group, then please describe the goals of the
> group and provide the name of the individual who will be committed to
> leading and managing the group’s activities. Also describe how you
> think the ISC can we help promote your project.

## Funding

The main resource we need is funding for our time. With the milestones
described above, we can serenely spend the time we want to commit to
this project.

## First level support from Google

We understand Go is a language that gets a lot of attention from Google.
…

## Travel

This probably will require promotion in various meetups and conferences
(useR\! … ) for which we’d like to request funding.

In addition to the pace we would commit to, it make make sense for both
of us to work together on the same site for a few days.

# Dissemination

> How will you ensure that your work is available to the widest number
> of people? Please specify the open source or creative commons
> license(s) will you use, how you will host your code so that others
> can contribute, and how you will publicise your work. We encourage you
> to plan content to be shared quarterly on the R Consortium blog.
