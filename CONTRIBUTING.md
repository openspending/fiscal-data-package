# How to contribute

Fiscal Data Package is a lightweight, descriptive data model that can be used to describe and compare a wide variety of fiscal data. It acts as a backbone to connect many of the applications in the OpenSpending technology stack, but is also an open specification that aims to support the variety of ways individuals work with fiscal data.  Therefore, we encourage feedback and contributions from all types of fiscal data producers, intermediaries (e.g. journalists and researchers), as well as consumers.

Rufus Pollock of [Open Knowledge](https://okfn.org/) (@rgrp) is currently overseeing the development of the specification. 
## Getting Started

* Sign up for a [GitHub account](https://github.com/signup/free) if you don't already have one
* Learn the [basics](https://help.github.com/articles/creating-an-issue/) of [submitting an issue](https://github.com/openspending/fiscal-data-package/issues/new) on GitHub
* Familiarize yourself with the [Data Package](http://dataprotocols.org/data-packages/index.html) and [Tabular Data Package](http://dataprotocols.org/tabular-data-package/index.html) specifications, upon which Fiscal Data Package is based

## Issues Types

Upon receipt of a new issue, the standards team (@danfowler) will tag the issue as one of:

* Discussion
* Meta

The default state for a new issue is `Discussion`.  This is especially so if the issue relates to more than one specific aspect of the standard.  The `Meta` tag is reserved for issues "about" the standards process (e.g. this contribution document).  This initial assessment of the issue should happen within **3 working days**, though more likely within **1 working day**.

## Issue States

While an issue is in the Discussion state, interested parties are invited to provide feedback on the merits of the issue and how to properly address it.  We aim for broad consensus on resolution, but in the interests of rapid iteration, we intend to keep issues open for discussion no longer than **10 working days**.  If a clear resolution has been broadly agreed within this time period, a new issue specifically outlining the proposed change will be created by a member of the standards team and tagged as one of the following:

* Proposal (breaking change): for major changes that would break existing implementations
* Proposal (enhancement): for minor, non-breaking changes 
* Proposal (trivial): for minor formatting, grammar, or other clarification of existing text

This issue is itself presented for discussion for no longer than **5 working days**.  Note: a “trivial” Proposal issue may be raised without any previous discussion.

Discussion may also lead to an acknowledgement that some further work must be done to aid resolution.  This will lead to a Discussion issue tagged with one of:

* Example Needed
* Clarification Needed
* Research Needed

An additional **5 working days** for resolution is given to discussions tagged as such.

If no clear resolution has been achieved in the time allotted to the discussion, it will be tagged as one of:

* Wontfix
* Duplicate
* Invalid

And closed with justification provided.

## Proposals

Once a concrete proposal has been submitted, a [Pull Request](https://help.github.com/articles/using-pull-requests/) directly addressing that proposal can be submitted.  A Pull Request is a concrete set of changes to the text that ready to merge into the existing text of the specification (https://github.com/openspending/fiscal-data-package/blob/master/spec/index.md)


## Versioning

Starting at version 0.3.0., when the core of the standard is more stable, we will likely move to [Semantic Versioning 2.0.0](http://semver.org/).

Given a version number MAJOR.MINOR.PATCH, increment the:

* MAJOR version when you make incompatible API changes,
* MINOR version when you add functionality in a backwards-compatible manner, and
* PATCH version when you make backwards-compatible bug fixes.

# Additional Resources

* [General GitHub documentation](https://help.github.com/)
* [GitHub pull request documentation](https://help.github.com/send-pull-requests/)
* #openspending IRC channel on freenode.org ([Archive](https://botbot.me/freenode/openspending/))
* [OpenSpending Discussion List](https://discuss.okfn.org/c/openspending)


