- Feature Name: data_frames
- Start Date: 2020-08-05
- RFC PR: [nushell/rfcs#0003](https://github.com/nushell/rfcs/pull/0000)
- Nushell Issue: [nushell/nushell#0000](https://github.com/nushell/nushell/issues/0000)

# Summary

[summary]: #summary

This RFC merges the Row and Table Value types into a single new value type: Frame. Data frames take inspiration from data processing systems like R and Pandas. Data frames will play the fundamental role of modelling data in Nu and will have enough descriptive power to describe all forms of structure, including streaming tables, lists, and objects.

# Motivation

[motivation]: #motivation

The current system has a few unexpected limitations:

- The top-level rows represent a table of rows, but it's unclear how to represent a top-level list of strings vs a stream of strings.
- A similar ambiguity exists between an "object" (a data structure denoted by key/value pairs) and a table of one row
- Inner-tables are modelled differently than top-level tables, leading to confusion
- There is no way to currently represent a matrix
- As rows are streamed instead of tables, it's not possible to predict how to display this information. This is currently mitigated by buffering some number of rows and treating them as one "table"
- Likewise, since rows are streamed instead of tables, it's unclear what the user should expect if they request a column that is not present, as this column may appear in the following row instead.
- When table data is serialized, there is a large amount of duplication, as columns are repeated with each row sent.
- Additionally, there is currently no way to represent rows using a row literal. We propose a frame literal that will fill this role.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Data frame representation:

```rust
struct DataFrame {
    headers: Option<Vec<String>>,
    rows: Vec<Vec<Value>>,
    partial_frame_id: Option<Uuid>,
    is_object: bool,
}
```

## Tables

A self-contained table would look like this:

```rust
let frame = DataFrame {
    headers: Some(vec!["name", "age"]),
    rows: vec![
        vec![Value::from("Bob"), Value::from(30)],
        vec![Value::from("Sally"), Value::from(43)]
    ],
    partial_frame_id: None,
    is_object: false,
};
```

The above code could be created using this Nu syntax:

```sh
[name: [Bob, Sally], age: [30, 43]]
```

## Lists and matrices

A self-contained list would look like this:

```rust
let list = DataFrame {
    headers: None,
    rows: vec![
        vec![Value::from(1), Value::from(2), Value::from(3)]
    ],
    partial_frame_id: None,
    is_object: false,
}
```

The above code could be created using this Nu syntax:

```sh
[1, 2, 3]
```

## Objects (aka hash tables)

A self-contained object would look like this:

```rust
let obj = DataFrame {
    headers: Some(vec!["name", "level"]),
    rows: vec![
        vec![Value::from("Thomas"), Value::from(12)]
    ],
    partial_frame_id: None,
    is_object: true,
}
```

**Note:** we use the boolean in the table rather than enumeration because all processing on the frame remains uniform regardless of if the frame is a single row with headers vs an object. This simplifies algorithms to only have to work with the data directly, and we can later represent this data and/or serialize this data in a way that maintains the user's model.

The above code could be created using this Nu syntax:

```sh
[name: Thomas, level: 12]
```

## Streaming

An important part of Nu is being able to work with potentially endless streams of data. As is often the case when working with external commands, there's no guarantee that the output will terminate.

We need to be able to represent the results of processing a stream of unprocessed data as a stream of data frames.

To be able to output a data frame as a stream, we need to know two key elements: that the data frame is incomplete as-is, and a unique identifier that allows us to later stream additional data for this frame.

To accomplish this streaming, we also introduce an `EndFrame`. Frame and EndFrame work together to allow a frame to be streamed as a multi-part frame, ending once the corresponding EndFrame has been read.

As an example, let's say we were processing some content and wanted to output the first row and later the second row of this table:

| tag  | length |
| ---- | ------ |
| head | 1024   |
| body | 8192   |

To do this, we would send the two separate frames, both marked as partial frames.

```rust
let frame_id = Uuid::new_v4();
output.send(UntaggedValue::DataFrame {
    headers: Some(vec!["tag", "length"]),
    length: vec![
        vec![ Value::from("head"), Value::from(1024)]
    ],
    partial_frame_id: Some(frame_id),
    is_object_false,
}.into_value());

// ... time passes

output.send(UntaggedValue::DataFrame {
    headers: Some(vec!["tag", "length"]),
    length: vec![
        vec![ Value::from("body"), Value::from(8192)]
    ],
    partial_frame_id: Some(frame_id),
    is_object_false,
}.into_value());

output.send(UntaggedValue::EndFrame(frame_id).into_value());
```

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

Much of the implementation is part of the explanation in the previous section. In this section, we'll explore more of the impact of making this change.

`UntaggedValue` will have `Table` and `Row` replaced with `DataFrame` (possibly just called `Frame` for brevity) and `EndFrame`.

All commands that operate on Row today will need to be updated to work with the data frame instead.

Rather than operating on a single row, commands will need to be updated to handle a frame at a time. Here, the mapping should largely be the same, though an additional inner loop to process over the rows will be necessary. This processing can be done serial or in parallel and may be done synchronously or asynchronously.

Commands that worked over inner tables should be able to migrate to data frames, as there is a strong overlap in functionality.

Commands that filter will work similarly as before, and may opt to output streams which are flattened by the output stream. This allows them to optionally return no frame if no rows in the frame met the requirements of the filter.

# Drawbacks

[drawbacks]: #drawbacks

Some drawbacks come to mind:

- This is a large, non-trivial amount of work. Getting this landed, updating the commands to use the new model, and thoroughly testing will take time.
- This will break most, if not all, third-party plugins

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

## Everything is a frame

One alternative is to require everything to live inside of a frame. There are some advantages here: this is seemingly more uniform, but at the risk of overloading the data frame concept.

# Prior art

[prior-art]: #prior-art

Other data processing systems and languages have a data frame concept. The R language and the `pandas` library for Python use it as a model for working with data in tabular format.

## R data frame

```r
Live Demo

# Create the data frame.
emp.data <- data.frame(
   emp_id = c (1:5),
   emp_name = c("Rick","Dan","Michelle","Ryan","Gary"),
   salary = c(623.3,515.2,611.0,729.0,843.25),

   start_date = as.Date(c("2012-01-01", "2013-09-23", "2014-11-15", "2014-05-11",
      "2015-03-27")),
   stringsAsFactors = FALSE
)
# Print the data frame.
print(emp.data)
```

which outputs:

```
 emp_id    emp_name     salary     start_date
1     1     Rick        623.30     2012-01-01
2     2     Dan         515.20     2013-09-23
3     3     Michelle    611.00     2014-11-15
4     4     Ryan        729.00     2014-05-11
5     5     Gary        843.25     2015-03-27
```

## Pandas data frame

Below is an example of the pandas data frame:

```python
>>> d = {'col1': [1, 2], 'col2': [3, 4]}
>>> df = pd.DataFrame(data=d)
>>> df
   col1  col2
0     1     3
1     2     4
```

# Unresolved questions

[unresolved-questions]: #unresolved-questions

- Are there syntactic ambiguities with the proposed syntax? This will require that we support parsing data frames, which includes colons and commas at the end of bare words.
- How do we want to handle partial inner data frames? That is, a data frame that is inside of another data frame.
- How do we handle non-data frames in between data frames? Do all partial data frames have to stream out until complete?

# Future possibilities

[future-possibilities]: #future-possibilities

We would like to be able to extend Data Frames further to be able to handle sending snapshots of data at the current time. This allows us to stream updates to existing tables, allowing viewers to animate as data is updated.

We may also elect to add type information to the columns, so that we can maintain a more rigorous internal representation.

Frames also allow us to store values in an unboxed way if we can ensure all the values in a column match, and that this holds for all columns in the frame.

Commands that collect a stream into a list could potentially have the optional to merge together all partial data frames into self-contained data frames for further processing.
