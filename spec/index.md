---
layout: spec
title: Specification - Fiscal Data Package
version: 0.3.0-alpha5
updated: 4 November 2015
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

- `0.3.0-alpha5`: variety of improvements and corrections including #35, #37 etc
- `0.3.0-alpha4`: reintroduce a lot of the content of data recommendations from v0.2
- `0.3.0-alpha3`: rework mapping structure in various ways
- `0.3.0-alpha2`: rename Budget Data Package to Fiscal Data Package
- `0.3.0-alpha`: very substantial rework of spec to use "mapping" approach between physical and logical model. Core framework, based on Tabular Data Package, is unchanged.
- [`0.2.0`](./0.2/): large numbers of changes and clarifications for particular fields but no substantive change to the overall spec
- [`0.1.0`](./0.1/): first complete version of the specification

# Overview

Data on government budgets and spending is becoming available in unprecedented quantities. The practice of publishing budget information as machine-readable and openly licensed data is spreading rapidly and will become increasingly standard.

Fiscal Data Package is an open specification for quantitative fiscal data, especially data generated during the planning and execution of budgets. It supports both data on expenditures and revenues, and also supports publishing both highly aggregated and highly granular data, for example individual transactions.

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

This proposal also builds on and reuses the [Data Package][dp] specifications. These are a family of simple, lightweight formats for publishing data. If you are unfamiliar with these, more information can be found in the Appendix.

# Form and Structure

A Fiscal Data Package has a simple structure:

* Data: the data `MUST` be stored in CSV files.
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

The data in your Data Package `MUST`:

* Be in CSV format.
* Have well-structured CSVs- no blank rows, columns etc. [Tabular Data Package][tdp] spells this out in detail.

*Note: you can store other data files in your data package - for example, you may want to archive the original xls or data files you used. However, we do not consider these data for the purposes of this specification.*

## The Descriptor - `datapackage.json`

A Fiscal Data Package `MUST` contain a `datapackage.json` - it is the central file in an Fiscal Data Package.

The `datapackage.json` contains information in three key areas:

* Package Metadata - title, author etc
* Resources - describing data files
* Mapping - mapping the source data to a "Logical" model

We will detail each in turn.

## General Package Metadata

This follows [Data Package][dp] (DP). In particular, the following properties `MUST` be on the top-level descriptor:

* `name` (DP): a url-compatible short name ("slug") for the package
* `title` (DP): a human readable title for the package

The [Data Package specification][dp] lists various additional potential metadata properties including information on licensing, package authors and much more. Here we focus on detailing key properties, especially those which are specific to this specification:

The following properties `SHOULD` be on the top-level descriptor:

* `license` (DP): specifies the license for the data in this package.
* `countryCode`: a valid 2-digit ISO country code (ISO 3166-1 alpha-2), or, an array of valid ISO codes (if this relates to multiple countries). This field is for listing the country of countries associated to this data.  For example, if this the budget for country then you would put that country's ISO code.

The following properties `MAY` be present:

* `granularity`: a keyword that represents the type of spend data, being one of "aggregated" or "transactional".
* `fiscalPeriod`: the fiscal period of the dataset, represented as a hash with two properties (`start` and `end`) whose values are ISO 8601 dates.

```
{
  "start": "1982-04-22",
  "end": "1983-04-21"
}
```

In addition to the properties described above, the descriptor `MAY` contain any number of additional properties that are not declared in the specification.


## Resources

The Data Package `MUST` have a `resources` property. 

The definition and behaviour of the `resources` property is described in detail in the [Data Package][dp-resources] and [Tabular Data Package][tdp] specifications.

The two key points we emphasize here from the [Tabular Data Package specification][tdp] are:

* Each data file `MUST` have an entry in the `resources` array
* That entry in the resources array `MUST` have a [JSON Table Schema][jts] schema describing the data file

## Mapping

The Fiscal Data Package `MUST` provide a `mapping` property. `mapping` `MUST` be a hash.

The `mapping` hash provides a way to link the "physical" model - the data in CSV files - to a more general, conceptual, "logical" model for fiscal information.

<img src="https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/pub?w=710&h=357" alt="" />

*Diagram illustrating how the mapping connects the "physical" model (raw CSV files) to the "logical", conceptual, model. The conceptual model is heavily oriented around OLAP.  ([Source on Gdocs](https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/edit))*
{: style="text-align: center"}

### Logical Model

The logical model has some key concepts:

* Amount (money): fundamentally fiscal information relates to amounts of money.
  * Key subconcept are things like: currency, units of account vs nominal (i.e. deflated or purchasing power parity values vs nominal values)
* Date / Time: most financial transactions have a date or time associated
* Description(s): fiscal information frequently has some kind of description or summary
* Entities who spend or receive monies: entities, whether individuals or organizations, are the spenders or receivers of money.
  * Payor: the entity expending money
  * Payee: the entity receiving money
* Classifications (taxonomies): for example, that a given transaction relates to Healthcare, or is a capital vs non-capital expenditure.
* Project / Programs: expenditure is often linked to a specific project or program

The actual description implementation utilises [OLAP][olap] terminology (and ideas). Key aspects for our purpose are:

* Numerical *measures*: these will be the monetary amounts in the spending data
* Dimensions: dimensions cover all items other than the measure
  * In OLAP attributes is also used for dimensions that are "single-valued" - for example, a description field.

From an OLAP perspective many of these dimensions may not split out in actual separate tables but map to attributes on the fact table if they are very simple (e.g. a given classification may just be a single field).

### Details

The `mapping` is a hash. It `MUST` contain a `measures` property and it `MUST` contain a `dimensions` property. Both `measures` and `dimensions` `MUST` be arrays:

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
        "fields": [
          "...",
        ],
        "..."
      },
      {
        "name": "dimension-2",
        "fields": [
          "...",
        ],
        "..."
      }
    ]
  }
```

**Describing sources**: the logical model will repeatedly need to indicate that the data for a given part of the model comes from a given field/column in a CSV file. This is done with a `source` property.

A full representation of the logical model property is a hash that `MUST` contain `source` and `MAY` contain `resource`. `source` declares the name of the field on the resource. `resource` declares the resource where the source field is. If `resource` is not included, it defaults to the first resource in the resource array:

```
# full representation, using an object and the source property
"property-on-logical-model": {
  "source": "name-of-field-on-the-resource",
  "resource": "name-of-resource"
  ...
}
```

### Measures

Measures are numerical and correspond to financial amounts in the source data. Each measure is represented by a hash in the `measures` array. The hash structure is like the following:

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
* `source`: (`MUST`) Field name of source field
* `currency`: (`MUST`) Any valid ISO 4217 currency code.
* `factor`: (`MAY`) A factor by which to multiple the raw monetary values to get the real monetary amount, eg `1000`. Defaults to `1`.
* `resource`: (`MAY`) Resource containing the source field. Defaults to the first resource in the resources list.
* `direction`: (`MAY`) A keyword that represents the *direction* of the spend, being one of "expenditure" or "revenue".
* `phase`: (`MAY`) the phase of the budget that the values in this measure relate to. It MUST be one of "proposed", "approved", "adjusted", or "executed".

### Dimensions

Each dimension is represented by a hash in the `dimensions` array. The hash has the structure:

```
{
  "name": "dimension-name",
  # dimensionType is optional
  # it can be used to indicate this is a standard types e.g. entity, classification, project etc
  "dimensionType": "...",
  "fields": [
    {
      "name": "field-1",
      "source": "...",
      "resource": "..."
    },
    {
      "name": "field-2",
      "source": "..."
    }
  ],
  "primaryKey": ["field-1"],
  other properties ...
}
```

Properties:

* `name`: (`MUST`) The dimension name in the logical model
* `fields`: (`MUST`) An array of field objects that make up the dimension. Each `field` is an entry in the array - think of it as column on that dimension in a database. At a minimum it must have "source" information - i.e. where the data comes from for that property (see "Describing Sources" above). A `field` MUST have a `name` attribute and `source` information.
* `primaryKey`: (`MUST`) Either an array of strings corresponding to the `fields` hash properties or a single string corresponding to one of the `fields` hash properties. The value of `primaryKey` indicates the primary key or primary keys for the dimension.
* `dimensionType`: (`MAY`) Describes what kind of a dimension it is. `dimensionType` is a string that `MUST` be one of the following:
  * `datetime`
  * `entity`
  * `classification`
  * `project`
  * `fact`
  * `location`
  * `other`

#### Common Dimensions

We illustrate here some common dimensions.

**`date`**

```
{
  "name": "date",
  "dimensionType": "datetime",
  # note the list of fields is for illustration - you can have any fields you like
  "fields": [
    {
      "name": "year",
      "source": "source field name"
    }
  ]
}
```

**`description`**

```
{
  "name": "description",
  # note the list of fields is for illustration - you can have any fields you like
  "fields": [
    {
      "name": "description",
      "source": "description source field name"
    } 
  ]
}
```

Note, that it might be more common to have description and other fields clustered on a "fact" dimension:

```
{
  "name": "fact",
  "dimensionType": "fact",
  # note the list of fields is for illustration - you can have any fields you like
  "fields": [
    {
      "name": "description",
      "source": "description source field name"
    },
    {
      "title": "title",
      "source": "title source field name"
    }
  ]
}
```


**`payer`**

```
# note the list of fields is for illustration - you can have any fields you like
{
  "name": "payer",
  "dimensionType": "entity",
  "fields": [
    {
      "name": "id",
      "source": "entity_id"
    },
    {
      "name": "title",
      "source": "entity_name"
    },
    {
      "name": "description",
      "source": "about"
    }
  ]
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

Classifications do not have any standardized `name`, If the classification is a well-known and standardized one, then it is conventional to use the name of that classification e.g. for COFOG the dimension would be called `cofog`.

```
# this is a made-up name for the dimension - you could call your dimension anything
{
  "name": "function",
  "dimensionType": "classification",
  "fields": [
    {
      "name": "id",
      "source": "budget_tree_id"
    },
    {
      "name": "title",
      "source": "budget_tree_label"
    }
  ]
}
```

## Examples

{% assign sorted_pages = site.pages | sort:"order" %}
{% for page in sorted_pages %}
  {% if page.category == 'example' %}
  * [{{ page.title }}]({{ page.url | remove: 'index.html' }}) 
  {% endif %}
{% endfor %}

# Content

This section provides a standard framework for the "content" of Fiscal Data Packages. The previous section has been about the form both for the data (e.g. that it is CSV) and for the metadata (the information in the datapackage.json). This section is about the "content", that is the kind of actual data a Package contains. In particular, it sets out guideliness for what information, exactly, is present. For example, that government budget information is classified according to a standard classification codesheet like the [UN's COFOG][cofog].

[cofog]: http://data.okfn.org/data/core/cofog/

Content requirements will necessarily vary across the different types of fiscal data. For example, the data describing high level budgets may be different from that describing day to day expenditures, and expenditure information may be different from revenue. Thus, our framework will distinguish different types of fiscal data.

We also emphasize that what we provide is a framework rather than a strict standard. That is, we provide recommendations on what information should be provided rather than strict requirements. These recommendations are also categorised into "quality" levels. Each level requires more information be provided.

Finally, our recommendations place requirements on the "logical" model not the physical model. Of course, the logical model data is sourced from the physical model so requirements on the logical model ultimately place requirements on the physical model. However, by defining our requirements on the logical model we keep the flexibility in naming and structure of the raw, source data - for example, whilst a classification dimension with COFOG data should be named `cofog` and have a field called `code` your source CSV could have that COFOG data in a column called "COFOG-Code" or "Classification" or any other name.


## Required data (all categories)

All datasets MUST have at least one measure. Essentially this is requiring each dataset have at least one field / column which corresponds to an "amount" (of money).

## Recommended data (all categories)

The following fields SHOULD be included wherever possible:

* `id`: A globally unique identifier for the budget item. This `id` field will usually be located on the default `fact` dimension.

## Special Dimensions

### Classifications

It is common for fiscal data to be classified in various ways. A classification is a labelling of a given item with a reference to standardized codesheet.

Classifications will be represented in the mapping as a dimension. Each classification dimension `MUST` have a `code` field whose value will correspond to the classification code in the official codesheet. Sometimes classifications can change and we recommend utilizing a `version` field if there is a need to indicate the version of a classification.

Whenever we have a code field in a classification dimension, the licit values for that field consists of the numerical codes from the appropriate codesheet, with hierarchical levels separated by periods. `1.1.4.1.3` is a licit value for `gfsmRevenue`, for example, corresponding to the code for "Turnover and other general taxes on goods and services".

Classifications are of different types. The type of the classification `MAY` be indicated using the `classificationType` attribute on the dimension. Values are:

* `functional`
* `administrative`
* `economic`

It is common for classifications to be hierarchical and have different levels. If this is present in your data and you wish to record it in the mapping, We recommend adopting the following structure using the keyword `level` with level 1 being the highest, most aggregate level:

```
{
  "name": "your-classification",
  "fields": [
    {
      # this will be the precise code
      "name": "code",
      "source": "..."
    },
    {
      # you can name the levels however you want, this is just an example
      "name": "level1"
      "level": 1,
    }
    {
      "name": "level2"
      "level": 2,
    }
  ]
}
```

#### COFOG (Classifications of Functions of Government)

This classification uses the United Nations [Classification of the Functions of Government][cofog]. Here is the simplest example as a dimension:

```
{
  "name": "cofog",
  "fields": [
    {
      "name": "code",
      "source": "..."
    }
  ]
}
```

#### IMF GSFM

GSFM is the [IMF Government Finance Statistics Manual (2001)][gfsm2001]. For expenditure classification we use Table 6.1. For revenue use Table 5.1.

[gfsm2001]: http://www.imf.org/external/pubs/ft/gfs/manual/

#### Chart of Accounts

It is common for both revenue and expenditure that there is some general "economic" classification for the item using the publisher's chart of accounts. Relevant fields for this dimension:

* `code`  The internal code identifier for the economic classification.
* `title`:  Human-readable name of the economic classification of the budget item (i.e. the type of expenditure, e.g. purchases of goods, personnel expenses, etc.), drawn from the publisher's chart of accounts.

### Programs and Projects

Expenditures are frequently associated with a program or project. Often these terms are used interchangeably. There is a rough distinction:

* Program: A program is a set of goal-oriented activities, such as projects, that has been reified by the government and made the responsibility of some ministry. A program can, e.g. be a government commitment to reducing unemployment.
* Project: A project is an indivisible activity with a dedicated budget and fixed schedule. A project can be a part of a bigger program and can include multiple smaller tasks. A project in an unemployment reduction program can e.g. be increased education opportunities for adults.

In terms of representation as a dimension the structure is common:

```
{
  "name": "program" or "project" or "your-chosen-name",
  "dimensionType": "project",
  "fields": [
    {
      # Name of the government program or project underwriting the budget item.
      "name": "title"
      "source": ...
    },
    {
      # The internal code identifier for the government program or project
      "name": "id"
      "source": ...
    }
  ]
}
```

### Entities

#### Administrators

* `title`: The title or name of the government entity legally responsible for spending the budgeted amount.
* `id`: The internal code for the administrative entity.
* `location`: Reference to a `location` dimension listing geographical region where administrative entity is located.
* `locationCode`: code for geographical region where administrative entity is located.

#### Accounts

Whilst strictly not an entity, the concept of an "account" from which money is spent or into which money is deposited is common and closely resembles an "entity" in terms of functionality within fiscal analysis (e.g. tracing the movement of money between different pots). Recommended fields on an account dimension are:

* `title`: The fund into which the revenue item will be deposited. (This refers to a named revenue stream.)
* `id`: The internal code identifier for the fund.

### Location

There is a frequent desire to label items with location, usually by attaching geographic codes for a region or area. This allows the spending or revenue to  be analysed by region or area. This geographic information can be introduced directly by classifying the item with a code, or, more frequently indirectly by associating a geographic code to e.g. an entity. For example, by labelling a supplier with their location one can then associate a spend with that supplier as spending in that location.

We RECOMMEND using a `location` dimension though fields may also be applied directly onto another object (e.g. an entity). Here are fields that `MAY` be applied either directly to an item or to an entity or other object associated to an item.

* `code`: The internal or local geographicCode id based for the geographical region
* `title`: Name or title of the geographical region targeted by the budget item
* `codeList`: the geo codelist from which the geocode is drawn
* `ocdid`: The [Open Civic Data Division Identifier](http://docs.opencivicdata.org/en/latest/proposals/0002.html), if it exists, for the geographical region

Note when applying these as fields directly on an object we suggest prefixing each value with `geo` so you would have `geoCode` rather than `code` etc.

----

## Aggregated expenditure data

Aggregated expenditure data (financialStatement `expenditure`, granularity `aggregated`) describes planned or executed government expenditures. These planned expenditures are disaggregated to at least the *functional category* level, and they can optionally be disaggregated up to the level of individual projects.

Aggregated data is in many cases the proposed, approved or adjusted budget (but can also be an aggregated version of actual expenditure). For this reason there are fields in aggregated data which are not applicable to transactional data, and vice versa.

| Dimension | Type | Quality | Description|
| ----- | -------- | ------- | ---------- |
| cofog | `classification` | 2 | The COFOG functional classification for the budget item. |
| gsfm  | `classification` | 2 | The GFSM 2001 economic classification for the budget item. |
| chart-of-accounts | `classification` | 2 | Human-readable name of the (non-COFOG) functional classification of the budget item (i.e. the socioeconomic objective or policy goal of the spending; e.g. "secondary education"), drawn from the publisher's chart of account. |
| administrator | `entity` | 2 | The name of the government entity legally responsible for spending the budgeted amount. |
| account | `entity` | 3 | The fund from which the budget item will be drawn. (This refers to a named revenue stream.) |
| program | `project` | 3 |  Name of the government program underwriting the budget item. A program is a set of goal-oriented activities, such as projects, that has been reified by the government and made the responsibility of some ministry. A program can, e.g. be a government commitment to reducing unemployment. |
| procurer | `entity` | (3) | The government entity acting as the procurer for the transaction, if different from the institution controlling the project. |

## Aggregated revenue data

Aggregated revenue data describes projected or actual government revenues, disaggregated to the *economic category* level.

Aggregated data is in many cases the proposed, approved or adjusted budget (but can also be an aggregated version of actual revenue). For this reason there are fields in aggregated data which are not applicable to transactional data, and vice versa.

| Dimension | Type | Quality | Description|
| ----- | ---- | ---------- |
| chart-of-accounts | `classification` | (2) | Name of the economic classification of the revenue item, drawn from the publisher's chart of account. |
| gsfm | `classification` | 2 | The GFSM economic classification of revenues for the revenue item. |
| account | `entity` | 3 | The fund into which the revenue item will be deposited. (This refers to a named revenue stream.) |
| recipient | `entity` | 2 | The recipient (if any) targetted by the revenue item. |
| source | `location` | (3) | Geographical region from which the revenue item originates. |

## Transactional expenditure data

Transactional expenditure data (direction `expenditure`, granularity `transactional`) describes government expenditures at the level of individual transactions, exchanges of funds taking place at a specific time between two entities. 

| Dimension | Type | Quality | Description|
| --------- | ---- | ------- | ---------- |
| administrator | `entity` | 2 | The government entity responsible for spending the amount. |
| date | `datetime` | 1 | The date on which the transaction took place. |
| supplier | `entity` | 2 | The recipient of the expenditure. |
| contract | `project` | 3 | The contract associated with the transaction. |
| budgetLineItem | `fact` | (3) | The unique ID of budget line item (value of id column for budget line) authorizing the expenditure. The budget line can either come from an approved or adjusted budget, depending on if the transaction takes place after the related budget item has been adjusted or not. |
| invoiceID | `fact` | (3) | The invoice number given by the vendor or supplier. |
| procurer | `entity` | (3) | The government entity acting as procurer for the transaction, if different from the institution controlling the project. |


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

Budget data has various degrees of hierarchy, depending on the perspective. From a functional perspective it can use a functional classification. The functional classification can be set up as a few levels (a hierarchy). An economical classification is not compatible with the functional hierarchy and has a different hierarchy. Another possible hierarchy would be a program/project hierarchy where many projects are a part of a program.

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

