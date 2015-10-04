---
layout: spec
title: Specification - Fiscal Data Package
version: 0.3.0-alpha2
updated: 22 September 2015
created: 14 March 2014
author:
 - Tryggvi Björgvinsson (Open Knowledge)
 - Rufus Pollock (Open Knowledge)
 - Paul Walsh (Open Knowledge)
summary: Fiscal Data Package is a lightweight and user-oriented format for publishing and consuming fiscal data. Fiscal data packages are made of simple and universal components. They can be produced from ordinary spreadsheet software and used in any environment.
---

<div class="alert alert-info" markdown="block">
NOTE: This is a draft specification and still under development. If you have comments
or suggestions please file them in the [issue tracker][issues]. If you have
explicit changes please fork the [git repo][repo] and submit a pull request.
</div>

[issues]: https://github.com/openspending/budget-data-package/issues
[repo]: https://github.com/openspending/budget-data-package/issues

# Table of Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Changelog

- `0.3.0-alpha2`: rename Budget Data Package to Fiscal Data Package
- `0.3.0-alpha`: very substantial rework of spec to use "mapping" approach between physical and logical model. Core framework, based on Tabular Data Package, is unchanged.
- [`0.2.0`](./0.2/): large numbers of changes and clarifications for particular fields but no substantive change to the overall spec
- [`0.1.0`](./0.1/): first complete version of the specification

# Overview

Data on government budgets and spending is becoming available in unprecedented quantities. The practice of publishing budget information as machine-readable and openly licensed data is spreading rapidly and will become increasingly standard.

Fiscal Data Package is an open specification for quantitative fiscal data, especially data generated during the planning and execution of budgets. This includes both data on expenditures and revenues, and also covers both aggregated data and highly granular data recording individual transactions.

The specification is both simple and easy for publishers to use and, at the same time, sufficiently rich and structured to be useful and processable - especially machine processable - by consumers. In particular, Fiscal Data Packages are:

* Made from lightweight and easily made components (CSV data, JSON metadata)
* Structured according to a simple open standard
* Self-documented with metadata
* Including sufficient information to allow for automated and standardized processing and analysis

Fiscal Data Package specifies the *form* for fiscal data and offers a standardized framework for the *content*. By giving a common *form* to budget data, Fiscal Data Package frees data users from the artificial obstacles created by the lack of a standard structure. By clarifying the *content* of budget data and recommending standard information that fiscal data should contain, Fiscal Data Package helps make data releases more comparable and useful. The Fiscal Data Package specification provides support for:

* A simple and standard way to provide rich metadata about fiscal information - where it came from, who produced it, how it is licensed, what time period it covers etc
* Mapping the raw "physical" model, as represented by columns in the data files, to a standardized "logical" model based around basic fiscal concepts: amounts spent, suppliers, administrative and functional classifications etc
* Progressive enhancement of data via a range of *recommended*, but not *required* metadata, in order to establish clear path for data providers to enhance data quality, and to address new use cases going forward.


# Background

This proposal assumes some familiarity with fiscal data - e.g. budgets, spending etc - as this is the data we are structuring and describing.

Often, this data takes the form of rows in a spreadsheet or database with each row describing some kind of expenditure or receipt of money. The data can get considerably more complex but keep this simple model in mind for what follows.

```
+--------+------+------------+------------+
| Amount | Year | Department | Spend Type |
+--------+------+------------+------------+
| 1500   | 2014 | Education  |   Capital  |
+--------+------+------------+------------+
| ....
+-----
```

This proposal also builds one and reuses the [Data Package][dp] specifications. These are a family of simple, lightweight formats for publishing data. If you are unfamiliar with these, more information can be found in the Appendix.

# The Standard

A Fiscal Data Package has a simple structure:

* Data: the data MUST be stored in CSV files.
* Descriptor: there must a descriptor in the form of a single `datapackage.json` file. This file describes both the data and the "package" as a whole (e.g. who created it, its license etc).

Fiscal Data Package builds on the existing [Data Package][dp] specifications. In particular, a Fiscal Data Package is a [Tabular Data Package][tdp] which is, in turn, a Data Package. We will spell out key implications of this as we proceed.

<!--
Here's an overview diagram that not only illustrates the basic setup but also the relation with the Data Package specifications:

![Basic diagram of Fiscal Data Package](img/open-spending-data-package.svg){: .center-block}

Basic overview of the Fiscal Data Package
{: style="text-align: center"}
-->

## Data Packages on Disk

Here are some examples of what an Fiscal Data Package looks like on disk. Usually, the datapackage.json and data files are bundled together, and collectively referred to as "the data package".

A simple example of an Fiscal Data Package:

```
datapackage.json
# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv
```

A more complex example, with additional files:

```
datapackage.json
README.md           # optional

# data files - can be data/ subdirectory or just in the base directory, must be CSV
data/my-financial-data.csv

# directory for storing original data and other 'archival' material
# (optional)
archive/my-original-data.xls

# scripts used in preparing the data package
# (optional)
scripts/scrape-and-clean-the-data.py
```

And, an example of a data package with normalized data could be:

```
datapackage.json
README.md

# data files, but only the first one actually contains the spend data. It may contain references (foreign keys) to the other files.
data/my-financial-data.csv # actually contains spend data
data/my-list-of-entities-receiving-money.csv # data that augmented the spend data
data/my-list-of-projects-the-money-is-associated-with.csv # additional augmenting data
```

## The Data

The data in your Data Package MUST:

* Be in CSV format.
* Have well-structured CSVs- no blank rows, columns etc. [Tabular Data Package][tdp] speels this out in detail.

*Note: you can store other data files in your data package - for example, you may want to archive the original xls or data files you used. However, we do not consider these data for the purposes of this specification.*

## The Descriptor - `datapackage.json`

A Fiscal Data Package MUST contain a `datapackage.json` - it is the central file in an Fiscal Data Package.

The `datapackage.json` contains information in three key areas:

* Package Metadata - title, author etc
* Resources - describing data files
* Mapping - mapping the source data to a "Logical" model

We will detail each in turn.

## General Package Metadata

This follows [Data Pacakge][dp] (DP). In particular, the following properties `MUST` be on the top-level descriptor:

* `name` (DP): a url-compatible short name ("slug") for the package
* `title` (DP): a human readable title for the package

The [Data Package specification][dp] list various additional potential metadata properties including information on licensing, package authors and much more. Here we focus on detailing key properties, especially those which are specific to this specification:

The following properties `SHOULD` be on the top-level descriptor:

* `license` (DP): specifies the license for the data in this package.
* `countryCode`: a valid 2 -digit ISO country code (ISO 3166-1 alpha-2) ,or, an array of valid ISO codes (if this relates to multiple countries). This field is for listing the country of countries associated to this data.  For example, if this the budget for country then you would put that country's ISO code.

The following properties `MAY` be present:

* `granularity`: a keyword that represents the type of spend data, being one of "aggregated" or "transactional".
* `direction`: A keyword that represents the *direction* of the spend, being one of "expenditure" or "revenue".
* `status`: A keyword that represents the status of the data, being one of "proposed", "approved", "adjusted", or "executed".
* `fiscalPeriod`: the fiscal period of the dataset, represented in the ISO 8601 time interval convention, that is two ISO dates separated by a solidus (/), e.g. 1982-04-22/1983-04-21

In addition to the properties described above, the descriptor `MAY` contain any number of additional properties that are not declared in the specification.


## Resources

The Data Package MUST have a `resources` property. 

The definition and behaviour of the `resources` property is described in detail in the [Data Package][dp-resources] and [Tabular Data Package][tdp] specifications.

The two key points we emphasize here from the [Tabular Data Package specification][tdp] are:

* Each data file MUST have an entry in the `resources` array
* That entry in the resources array MUST have a [JSON Table Schema][jts] schema describing the data file

## Mapping

The Fiscal Data Package MUST provide a `mapping` property. `mapping` MUST be a hash.

The `mapping` hash provides a way to link the "physical" model - the data in CSV files - to a more general, conceptual, "logical" model for fiscal information.

<img src="https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/pub?w=710&h=357" alt="" />

*Diagram illustrating how the mapping connects the "physical" model (raw CSV files) to the "logical", conceptual, model. The conceptual model is heavily oriented around OLAP.  ([Source on Gdocs](https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/edit))*
{: style="text-align: center"}

### Logical Model

The logical model has some key concepts:

* Amount (money): fundamentally fiscal information is usually amount amounts of money.
  * Key subconcept are things like: currency, units of account vs nominal (i.e. deflated or purchasing power parity values vs nominal values)
* Date / Time: most financial transactions have a date or time associated
* Description(s): fiscal information frequently has some kind of description or summary
* Entities who spend or receive monies: entities, whether individuals or organizations, are the spenders or receivers of money and so often.
  * Payor: the entity expending money
  * Payee: the entity receiving money
* Classifications (taxonomies): for example, that a given transaction relates to Healthcare, or is a capital vs non-capital expenditure.
* Project / Programs: expenditure is often linked to a specific project or program

The actual description implemenation utilises [OLAP][olap] terminology (and ideas). Key aspects for our purpose are:

* Numerical *measures*: these will usually be the monetary amounts in the spending data
* Dimensions: dimensions cover all items othe than the measure
  * In OLAP attributes is also used for dimensions that are "single-valued" - for example, a description field.

From an OLAP perspective many of these dimensions may not split out in actual separate tables but map to attributes on the fact table if they are very simple (e.g. a given classification may just be a single field).

### Details

The `mapping` is a hash. It MUST contain a `measures` property and it MUST contain a `dimensions` property. Both `measures` and `dimensions` MUST be arrays:

```
  {
    "measures": [
      {
        "name": "measure-1",
        "source": "...",
        "..."
      },
      {
        "name": "measure-2",
        "source": "...",
        "..."
      }
    ],
    "dimensions": [
      {
        "name": "dimension-1",
        "source": "...",
        "..."
      },
      {
        "name": "dimension-2",
        "source": "...",
        "..."
      }
    ]
  }
```

**Describing sources**: one common feature that we will need repeatedly is to indicate that the data for a given part of the logical model comes from a given field/column in a CSV file. Our common pattern for this is:

```
# full representation, using an object and the source property
"property-on-logical-model": {
  "source": "name-of-field-on-the-resource",
  # Optional - if not present it implicitly defaults to first resource in resources list
  "resource": "name-of-resource"

  ...
}
```

### Measures

Measures are numerical and usually correspond to finanical amounts in the source data. Each measure is represented by a hash in the `measures` array. The hash structure is like the following:

```
{
  "name": "measure-name",
  "source": "amount",
  "currency": "USD",
  "factor": 1
}
```

Properties:

* `name`: (`MUST`) The measure name in the logical model
* `currency`: (`MUST`) Any valid ISO 4217 currency code.
* `factor`: (`MAY`) A factor by which to multiple the raw monetary values to get the real monetary amount, eg `1000`. Defaults to `1`.

### Dimensions

Each dimension is represented by a hash in the `dimensions` array. The hash has the structure:

```
{
  "name": "dimension-name",
  # dimensionType is optional
  # it can be used to indicate this is a standard types e.g. entity, classification, program etc
  "dimensionType": "...",
  "fields": {
    "field-1": ...,
    "field-2": ...
  }
  other properties ...
}
```

Properties:

* `name`: (`MUST`) The dimension name in the logical model

Each `field` is a property on the dimension - think of it as column on that dimension in a database. At a minimum it must have "source" information - i.e. where the data comes from for that property (see "Describing Sources" above):

```
"field-1": {
  "source": "abc"
}
```

A dimension MUST have a `primaryKey` property. The property MUST be either an array of strings corresponding to the `fields` hash properties or a single string corresponding to one of the `fields` hash properties. The value of `primaryKey` indicates the primary key or primary keys for the dimension. An example:

```
"dimension-name": {
  "fields": {
    "field-1": {
      "source": "..."
    },
    "field-2": {
      "source": "..."
    }
  },
  "primaryKey": ["field-1"]
}
```

Dimension Types:

* `datetime`
* `entity`
* `classification`
* `project`

#### Common Dimensions

We illustrate here some common dimensions.

**`date`**

```
{
  "name": "date",
  # note the list of fields is for illustration - you can have any fields you like
  "fields": {
    "year": "source field name"
  }
}
```

**`description`**

```
{
  "name": "description",
  # note the list of fields is for illustration - you can have any fields you like
  "fields": {
    "description": "description source field name"
  }
}
```

**`payer`**

```
# note the list of fields is for illustration - you can have any fields you like
{
  "name": "payer",
  "fields": {
    "id": {
      "source": "entity_id"
    },
    "title": {
      "source": "entity_name"
    },
    "description": {
      "source": "entities/about"
    }
  }
  ...
}
```

**`payee`**

```
{
  "name": "payee",
  # as per payer ...
}
```

**`classification`**

Classifications do not have any standardized name, If the classification is a well-known and standardized one, then it is conventional to use the name of that classification e.g. for COFOG the dimension would be called `cofog`.

```
# this is a made-up name for the dimension - you could call your dimension anything
{
  "name": "function",
  "dimensionType": "classification",
  "fields": {
    "id": {
      "source": "budget_tree_id"
    }
    "title": {
      "source": "budget_tree_label"
    }
  }
}
```

## Examples

### Minimal example

Here is the most basic example of an Fiscal Data Package. The example describes a package that only has transactional data, with only required fields.

```
# budget.csv
{% include minimal/budget.csv %}

# datapackage.json
{% include minimal/datapackage.json %}
```

### Example with entities (denormalized)

Here is an example with information on the payer and payee, denormalized.

```
# budget.csv
{% include entities-denormalized/budget.csv %}

# datapackage.json
{% include entities-denormalized/datapackage.json %}
```

### Example with entities (normalized)

Here is the same example as previous, but with the entity data normalized. That means this is also an example of an Fiscal Data Package with a resource that is not a spend resource.

```
# budget.csv
{% include entities-normalized/budget.csv %}

# entities.csv
{% include entities-normalized/entities.csv %}

# datapackage.json
{% include entities-normalized/datapackage.json %}
```

# Acknowledgements

Thanks to Vitor Baptista, Sarah Bird, Samidh Chakrabarti, Pierre Chrzanowski, Andrew Clarke, Velichka Dimitrova, Friedrich Lindenberg, James McKinney, Rufus Pollock, Paolo de Renzio, Martin Tisne and Paul Walsh.

----

# Appendix

## Budget data

A budget is over a year-long process of planning, execution, and oversight of a government's expenditures and revenues. At multiple stages in the process, *quantitative data* is generated, data which specifies the sums of money spent or collected by the government. This data can represent either plans/projections or actual transactions.

In a typical budget process, a government authority, e.g. the executive arm, will put together a **proposed** budget and submit that for approval, e.g. by the country's legislative arm. The approval process might involve making changes to the proposal before the **approved** version is accepted. As time goes by there is a possibility that some projects, institutions etc. will require more money to fulfill their task so adjustments need to be made to the approved budget. The **adjustment** is then approved by the original budget entity, e.g. the legislative arm. This usually requires reasoning for why the original budget was not sufficient. The **executed** budget is the actual money spent or collected which can then be compared to the approved and adjusted plan.

By recognizing the following distinctions between data types, Fiscal Data Package is expressive enough to cover the data generated at every stage:

* Data can represent either *expenditures* or *revenues*.
* Data can be either *aggregated* or *transactional*. An item of aggregated data represents a whole category of spending (e.g. spending on primary education). An item of transactional data represents a single transaction at some specific point in time.
* Data can come from any stage in the budget cycle (proposal, approval, adjustment, execution). This includes three different types of planned / projected budget items (proposal, approval, adjustment) and one representing actual completed transactions (execution).

### Budget hierarchy and categorizations

Budget data has various degrees of hierarchy, depending on the perspective. From a functional perspective it can use a functional classification. The functional classification can be set up as a few levels (a hierarchy). An economical classification is not compatible with the functional hierarchy and has a different hierarchy. Another possible hierarchy would be a program project hierarchy where many projects are a part of a program.

All of these hierarchies give a picture of how the budget line fits into the bigger picture, but none of them can give the whole picture. Budget data usually only includes general classification categories or the top few hierarchies. For example a project can usually be broken down into tasks, but budget data usually would not go into so much detail. It might not even be divided into projects.

Categorizing and organizing the data is more about describing it from the bigger perspective than breaking it down into detailed components and the Fiscal Data Package specification tries to take that into account by including top level hierarchies and generalised classification systems but there is still a possibility to go into details by supplying a good description of every row in the budget data.



[dp]: http://dataprotocols.org/data-packages/
[tdp]: http://dataprotocols.org/tabular-data-package/
[bdp]: https://github.com/openspending/budget-data-package
[bdp-resources]: https://github.com/openspending/budget-data-package/blob/master/specification.md#budget-specific-metadata
[dp-profiles]: https://github.com/dataprotocols/dataprotocols/issues/87
[dp-resources]: http://dataprotocols.org/data-packages/#resource-information
[fd]: http://data.okfn.org
[mapping]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[views]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[jts]: http://dataprotocols.org/json-table-schema/
[current-model]: http://docs.openspending.org/en/latest/model/design.html
[cofog]: http://unstats.un.org/unsd/cr/registry/regcst.asp?Cl=4
[imf-budget]: http://www.imf.org/external/pubs/ft/tnm/2009/tnm0906.pdf
[olap]: https://en.wikipedia.org/wiki/Online_analytical_processing

