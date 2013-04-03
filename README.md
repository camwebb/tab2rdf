tab2rdf
=======

**A simple template-based converter for data in relational tables**

This application read a comprehensive publishing template in a
`conf.ini` file and then text, delimited files representing a set of
interlinked tables with primary and foreign keys, as exported from a
relational database or a multisheet spreadsheet.

Install
-------

The program is a standalone `awk` script.  It was developed with
`gawk` v.4 but should work with the ubiquitous `gawk` v.3.  Just make
sure the first line of the script points to your `gawk` executable (do
`$ which gawk` id you’re not sure where it is).  You should also make
sure the `tab2rdf` file is executable (`chmod u+x tab2rdf`).

Usage
-----

Place table files and `conf.ini` in a directory, and simply type:

      $ ./tab2rdf eg/

replacing `eg/` with your data directory. The turtle serialization of
the RDF is output to standard out.  There is extensive format checking
to help get the input files in the right state.

As you are testing your model, I recommend visualizing a _small_
dataset with:

      $ tab2rdf eg > eg/out.ttl
      $ rapper -i turtle -o dot eg/out.ttl > eg/out.dot
      $ dot -T jpg eg/out.dot > eg/out.jpg

Using Redland tools and GraphViz.

Data file format
----------------

Each data table should:

 * be delimited with `|` (pipe) characters
 * have a single header line
 * have a primary key column that is _either_ a URI (beginning with
   http://), _or_ a string with unique values that are _URL safe_
   (i.e., regex = `[A-Za-z0-9_-]+`); the primary key field does not
   need to be the first
 * have all values of any foreign key fields be present in the primary
   key field of another table
 * be named `Table.csv` where _Table_ is the table name, without any
   spaces in it

`conf.ini` file format
----------------------

See the [example](eg/conf.ini) file.  Format follows standard `.ini`
file syntax (a header in square brackets, then lines of the format
`key = "value"`, then a blank line before the next section. There
should be one `[_general]` section, and one section for each table,
named `[Table]` where _Table_ is the table name, without the `.csv`.

The `[_general]` section must contain one `_prefix` pair, and
optionally one `_unique` pair.  The value of `_unique` can be used to
add a unique part of resource URI names that may reside at a common
triplestore address.  Additionally, if any of the objects already have
external URIs, these can be substituted in by specifying
`_sub:Table:key` to have a value of `oldURI|newURI`. See the example
file for clarification.

Each `[Table]` section should have the following special pairs:

 * `_class` : the Semantic class of the table objects
 * `_GUID` :  the formation of the GUID for each data row
 * `_pk` : the field name of the primary key
 * `_fk:xxx` : (optional) an indication that field `xxx` contains
   foreign keys. The value of the pair is the Table name, to the
   primary keys of which the foreign keys point

The remaining lines in a `[Table]` section define the conversion from
field cells to RDF triples. Each _value_ of the pair:

 * is enclosed in `"` (double quotes). Any quotes inside the value
   should be escaped with a backslash.
 * Should represent valid RDF in the turtle serialization, _after_
   substitutions.
 * May contain any number of separate triples, and may contain blank
   nodes, using the `:s :p [ :p2 :o2 ; ] .` notation.
 * uses field names in `{}` (braces) to indicate that data from a cell
   in the table should be substituted in (e.g., `{genus}`).
 * can contain `{@}` to indicate the primary subject URI of a row’s
   data, and `{~}` to indicate the _unique value_ be inserted.

