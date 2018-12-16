| FIP | Title                                                                          | Status | Category | Author                             | Created    |
| ----- | ---------------------------------------------------------------------------- | ------ | -------- | ---------------------------------- | ---------- |
| X     | Standard, Contributions, and Process Guidelines for the Factom&reg; Protocol | Draft  | Meta     | Devon Katz \<<devonk@dbgrow.com>\> | 20181124   |



# FIP-X

[FIP-X](/) is the initial standard that describes the core framework,
conventions, contribution guidelines, and ecosystem of FIP standards.


# What Is An FIP?

FIP is a formal proposal and process for accepting and integrating new
standards into the Factom&reg; Protocol standard collection.


# Why Are They Important?

FIP provides an open forum for the community to collaborate on and accept new
standards into the Factom&reg; Protocol. The Factom&reg; Protocol is completely open source, and we rely
on our community and standing parties to keep us pointed in the right direction.


# Contributing

Create your submission in a markdown file following the guidelines defined in
[FIP-X](x.md) and the [FIP-Template](template.md)

To submit your standard, fork this repo and then submit a pull request to have
it reviewed and included for evaluation by the community.


# Proposal Categories


## Core

Standards that define the basic building blocks, technology, and conventions
that govern Factom&reg; Protocol standards


## Apps &amp; libraries

Standards that define application level functionality on top of the Factom&reg; Protocol


## Interface

Standards that define API architecture, conventions, and specs


## Meta

Standards about FIP standards, processes, etc.


# Proposal Statuses


## Work In Progress (WIP)

The standard's information is sufficient to be reviewed by the community. This
is essentially an "Intent to submit" a FIP standard. The community member(s)
submit a formatted pull request containing the preliminary version of the
proposed standard.

- If reviewed and denied, the proposal may be revised and improved unless it is
  explicitly rejected
- If reviewed and approved it is assigned a FIP number and other metadata.
  The proposal moves on to drafting.


## Draft

The standard is in the progress of being revised. Follow up pull requests will
be accepted to revise the standard until it is ready to go through the last
call process (explained below).


## Last Call

The token standard is open to final evaluation by the community and general
public.

- If the standard requires further changes, it reverts to drafting again.
- If the proposal is approved and it is a Core, or Interface FIP
  standard then it will move onto accepted.
- If the proposal is approved and it is an information standard then it will
  move onto final.

As FIP is young, currently no maximum timeline is set for this status.
  

## Accepted

The go ahead for development is given and the standard is finalized.
Implementation in core, clients and APIs can occur. When the community reaches
consensus on the implementation's success, the status will change to final.


## Final

The proposal has been finalized and accepted by the community. The proposal is
adorned with the final official prefix **Factom-Protocol**, replacing the former **FIP**
prefix. Errata may be formally submitted following this stage if required.


# Proposal Workflow


# FIP Editors

For each new FIP that comes in, an editor does the following:

- Read the FIP to check if it is ready: sound and complete. The ideas must
  make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the FIP for language (spelling, grammar, sentence structure, etc.),
  markup (Github flavored Markdown), code style

If the FIP isn't ready, the editor will send it back to the author for
revision, with specific instructions.

Once the FIP is ready for the repository, the FIP editor will:

- Assign an FIP number (generally the PR number or, if preferred by the
  author, the Issue # if there was discussion in the Issues section of this
repository about this FIP)
- Merge the corresponding pull request
- Send a message back to the FIP author with the next step.

Many FIPs are written and maintained by developers with write access to the codebases
of the Factom&reg; Protocol or 3rd party applications. The FIP editors monitor FIP changes, and correct any
structure, grammar, spelling, or markup mistakes they see.




# Proposal Structure

A [FIP Template](template) is supplied to begin writing your standard.


## Header

An informational header containing metadata about the FIP being submitted,
like so:

| FIP | Title         | Status | Category | Author                               | Created   |
| ----- | ------------- | ------ | -------- | ------------------------------------ | --------- |
| N     | Standard Name | Status | Category | Author Name \<<author@example.com>\> | 7-23-2018 |

Once accepted as a draft an editor will assign an official FIP number.


## Summary

"If you can't explain it simply, you don't understand it well enough." Provide
a simplified and layman-accessible explanation of the FIP.


## Motivation

Clearly explain why the existing protocol specification is inadequate to
address the problem that the FIP solves. FIP submissions without sufficient
motivation may be rejected outright. What motivated the design and why were
particular design decisions made?


## Specification

The technical specification should describe the syntax and semantics of any new
feature. The specification should be detailed enough to allow competing,
interoperable implementations for any of the current Factom&reg; Protocol.


## Implementation

The implementations must be completed before any FIP is given status "Final",
but it need not be completed before the FIP is given status "Accepted". While there is merit
to the approach of reaching consensus on the specification and rationale before
writing code, the principle of "rough consensus and running code" is still
useful when it comes to resolving many discussions of standard details.


## Copyright

The standard must have a copyright section that waives rights according to CC0. Please note that this is only applicable for the proposal itself:

```
Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
```


# Inheritance & Dependencies

Factom&reg; Protocol standards can extend and inherit functionality and rules from others. The
`extends` \<FIP #> Keyword and Tag denotes feature inheritance on a feature
by feature basis. The author of the FIP must explain how and what is being
inherited, and if there are any changes being made. It is also possible to depend on another FIP. The
`depends-on` \<FIP #> Keyword and Tag denotes dependency on a feature
by feature basis.


# Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
