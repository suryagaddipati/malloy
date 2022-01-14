# Fields

Fields constitute all kinds of data in Malloy. They
can represent dimensional attributes sourced directly from
tables in a database, constant values to be used in later analyis, computed metrics derived from other fields, or even nested structures created from aggregating subqueries.

## Defining Fields

Fields are defined in explores are reusable and are `dimension`, `measures` or `query`.  When these are used in a query, these fields are `project`, `group_by`, `aggregate` or `nest`.   The definitions are the same way whether part of an
explore or a query stage. In either case, they are defined using the `is` keyword.

**In an explore**

```malloy
explore users is table('malloy-data.ecomm.users'){
  dimension: age_in_dog_years is age * 7
}
```

**In a query stage**

```malloy
query: users->{
  group_by: age_in_dog_years is age * 7
}
```

The right hand side of this kind of definition can be any
field expression. See the [Expressions](expressions.md)
section for more information.

Named queries (see [below](#queries)) can also be defined as
part of an explore or query stage. When a named query is defined in a query stage, it is known as a "nested query" or an "aggregating
subquery." See the [Nesting](nesting.md) section for a
detailed discussion of nested queries.

```malloy
query: flights->{
  group_by: carrier
  nest: by_month is {
    group_by: departure_month is dep_time.month
    aggregate: flight_count is count()
  }
}
```

## Field Names

Field names must start with a letter or underscore, and can only contain letters, numbers, and underscores. Field names which conflict with keywords must be enclosed in backticks, e.g. `` `year` is dep_time.year``.

## Kinds of Fields

Malloy includes three different _kinds_ of fields: _dimensions_, _measures_, and _queries_.

### Dimensions

Dimensions are fields representing scalar values. All fields
inherited directly from a table are dimensions.

Dimensions are defined using expressions that contain no
aggregate functions.

```malloy
expore users is table('malloy-data.ecomm.users'){
  dimension: full_name is concat(first_name, ' ', last_name)
}
```

Dimensions may be used in both `reduce` and `project`
queries.

```malloy
-- Show the top 10 full names by number of occurrances
query: users->{
  top: 10
  group_by: full_name
  aggregate: occurances is count()
}

-- Show 10 users' full names
query: users->{
  project full_name
  limit: 10     // top and limit are synonyms
```

### Measures

Measures are fields representing aggregated data over
multiple records.

Measures may not be used in `project` queries. However, any measures that appear in a `reduce` query stage are "dimensionalized" as part of the query, and are therefore usable as dimensions in subsequent stages.

```malloy
query: flights->{
  group_by: carrier
  aggregate: flight_count is count()
} -> {
  project: flight_count
}
```

### Queries

Queries represent a pipelined data transformation including a source and one or more transformation stages. When queries are defined as part of an explore or query stage,
their source is implicit.

```malloy
explore flights is table('malloy-data.faa.flights'){
  query: by_carrier is {
    group_by: carrier
    aggregate: flight_count is count()
  }
}
```

A named query's pipeline can always begin with another named query.

```malloy
explore flights is table('malloy-data.faa.flights'){
  ...
  query: top_carriers is by_carrier->{
    project: carrier
    limit: 5
  }
);
```

<!-- TODO this does not seem to work in a query stage, but it does work in an explore or model -->

See the [Nesting](nesting.md) section for more details about named queries.