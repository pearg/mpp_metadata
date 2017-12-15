# ATERM Notes

Notes from 2017-12-15 meeting

## Asset Structure

```
asset
  metadata
    system
      non-user created, eg. creation date date

    user
      user defnined xml

  content
    one content file per asset
```

----------

# ATERM Java Client

Using the ATERM java client:
 - [Research Platform documentation](https://wiki.cloud.unimelb.edu.au/resplat/doku.php?id=data_management:mediaflux:howto:login_with_aterm)
 - need to check 'allow untrusted'

--------

## ATERM Commands

Has tab completion for services (e.g. `asset.<tab>`) but not for paths.

Has help documentation with the `help` command (e.g. `help asset.create`)

Arguments begin with `:` (e.g. `asset.create :id 123`).

`^P`: cycle through last matching command instead of `^R`

Navigate namespace with: `cd`, `ls`, `pwd`


### CREATE

```
asset.create
```

Create empty asset and return the new asset ID number. By default, creates in the root direction.

```
> asset.create :namespace /projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29
    :id "35415655"
> ls
    :namespace -path "/projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29"
        :asset -id "35415655" -version "1" "__asset_id__35415655"
```

By default, assets are given names like `__asset_id__35415655`. Can specify the name with the `:name` argument:

```
asset.create :namespace /projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29 :name my_test
```


### GET

```
asset.get :id <id-number>
```

Return information stored in asset.

```
> asset.get :id 35415656
    :asset -id "35415656" -version "1" -vid "58732988"
        :type "content/unknown"
        :namespace "/projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29"
        :path "/projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29/my_test"
```

For attributes, specify with `-<attr-name>`.

```
asset.get :id -version "1" 35415656
```

Information stored as XML and displayed in a compressed view.

```
# Instead of:
<asset id="35415656" version="1" vid="58732988>
    <type>content/unknown</type>
    ...
</asset>

# ATERM displays info as:
    :asset -id "35415656" -version "1" -vid "58732988"
        :type "content/unknown"
    ...
```

Each element has its own line and its children are indented one level down. Elements begin with `:`.  
Any attributes an element has are on the same line, represented by a `-<attr-name> <attr-value>`. (e.g. `id "1234"`)  
The element value is the last value on the line.

```
:element <attributes> value
```

### DESTROY

```
asset.destroy
```

Hard destroy assets. Warning: this is permanent.

To hide the asset instead of deleting it:

```
asset.soft.destroy
```


### Document definitions

Print out all types of documents:

```
asset.doc.type.list
```

Note that mediaflux doc defs begin with `mf-`.

Show document info with:

```
> asset.doc.type.describe :type mf-note
    :type -name "mf-note" -id "6" -version "1" -for "asset"
        :label "Note"
        :description "Abitrary text note."
        :generated-by "user"
        :access
            :administer "false"
            :create "true"
            :modify "true"
            :publish "true"
        :creator -id "0"
            :domain "system"
            :user "manager"
        :ctime -tz "Australia/Melbourne" -gmt-offset "10.0" -dst "true" -time "1411360673232" "22-Sep-2014 14:37:53"
        :definition
            :element -name "note" -type "string" -max-occurs "1"
                :description "Contents of the note."
                :restriction -base "string"
                    :lines "2147483647"
```

### Create asset with a document type

```
asset.create :meta < :mf-note < :note "Hello world" > >

# Note: Need to prefix document name and attributes with `:`
# The `:meta` parameter is xml structured where `<` and `>` indicate going down/up the xml tree
```

### Edit assets


Remove metadata document `mf-note`:

```
asset.set :id 123 :meta -action remove < :mf-note >
```

Add metadata document `mf-note`:

```
asset.set :id 123 :meta < mf-note < note "Test" > >

# Can also specify the action
asset.set :id 123 :meta -action [add|merge|remove|replace] <document>

# Allow invalid metadata using the `:allow-invalid-meta` option
asset.set :allow-invalid-meta ...

# Validate metadata
asset.meta.validate :id 123
```

### QUERY

[Query Snytax Documentation](https://mediaflux.vicnode.org.au:8443/mflux/portal.mfjp?module=help&shelf=development/queries&document=Asset%20Query%20Language%20Syntax)

Note: use single quotes `'` for defining strings for comparisons in expressions.

Note: `asset.query` returns 100 by default

```
asset.query :where <expression>
asset.query :where <expression> and <expression>
asset.query :where <expression> :where <expression>
```

Metadata is indexed.

If multiple expressions are used, separated by `and`, each expression is
evaluated, then the intersection of the results of both queries is returned.
Both expressions are run on the indexed metadata.

If using multiple expressions with multiple `:where` arguments, the first
expression is evaluated, then the second expression is run on the return of the
first expression. The second expression is run on metadata which is un-indexed
and therefore may be slower than usual. Usually filter the namespace first,
since `asset.query` searches everything.

```
asset.query :where id=123
asset.query :where name='test'
asset.query :where name='test' and id=35009187
asset.query :where namespace>=/projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29 :where mf-note has value
```

Can perform an action on each return of the query, similar to using xargs on find.

```
asset.query :where name='test' :action get-meta
```

Looking within metadata fields with xpath:

```
asset.query :where namespace=/projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29 :where xpath(mf-note/note)='Hello world'
asset.query :where namespace=/projects/proj-marsupial_genomics-1128.4.19/Test_2017-08-29 :where xpath(mf-note/note) contains literal('test')
```

Querying time:

```
asset.query :where ctime >= 'TODAY-3DAY'
asset.query :where atime >= '01-DEC-2017 00:00:00'
```
