# oai-ore

A Clojure library used to generate Open Archives Initiative Object Reuse and Exchange files for the CyVerse Data Commons
repository. The primary purpose of this library is to make it easier to generate OAI-ORE files from a set of URIs and
attribute metdata stored in the CyVerse Data Store.

## Usage

All examples assume that these commands have been executed in the REPL:

``` clojure
(require '[org.cyverse.oai-ore :as ore])
(require '[clojure.data.xml :refer :all])
(def agg-uri "http://foo.org")
(def arch-uri "http://foo.org/bar.xml")
(def file-uris ["http://foo.org/bar1.txt" "http://foo.org/bar2.txt"])
```

The suggested way to build an OAI-ORE file is to use the `build-ore` function. The result of this function is an
instance of `org.cyverse.oai-ore.Ore`, which can then be converted to RDF/XML and serialized. The simplest such file is
an empty archive:

``` clojure
(def empty-ore (ore/build-ore agg-uri arch-uri []))
```

Once you have the `org.cyverse.oai-ore.Ore` instance, you can convert it to RDF/XML by calling its `to-rdf` method:

``` clojure
(def rdf (ore/to-rdf empty-ore))
```

The result of calling this method is an instance of `clojure.data.xml.Element`, which can then be serialized using any
serialization method available in `org.clojure/data.xml`. For example, you can use the following commands to emit
pretty-printed RDF/XML:

``` clojure
(print (indent-str rdf))
```

An empty OAI-ORE file is good for an example, but not very useful. It's possible to add aggregated entites by including
URIs in the third argument to `build-ore`:

``` clojure
(def populated-ore (ore/build-ore agg-uri arch-uri file-uris))
```

This library also supports several of the metadata attributes that are used when a data set is published to the CyVerse
Data Commons repository. The following attributes are currently supported:

| CyVerse Attribute     | ORE Element      |
| --------------------- | ---------------- |
| datacite.title        | dc:title         |
| datacite.publisher    | dc:publisher     |
| datacite.creator      | dc:creator       |
| datacite.resourcetype | dc:type          |
| contributorName       | dc:contributor   |
| Subject               | dc:subject       |
| Rights                | dc:rights        |
| Description           | dc:description   |
| Identifier            | dc:identifier    |
| geoLocationBox        | dcterms:Box      |
| geoLocationPlace      | dcterms:Location |
| geoLocationPoint      | dcterms:Point    |

Any attribute that is associated with the data set that is not in this list is ignored. Similarly, any attribute that is
in the list but either contains an empty value or is not associated with the data set is ignored:

``` clojure
(def attr-ore (ore/build-ore agg-uri arch-uri file-uris [{:attr "datacite.title" :value "The Title"}
                                                         {:attr "datacite.creator" :value "The Creator"}
                                                         {:attr "ignored.attribute" :value "Who Cares?"}
                                                         {:attr "Subject" :value ""}]))
```

Serializing the RDF using `(print (indent-str (ore/to-rdf attr-ore)))` should produce the following output:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<rdf:RDF xmlns:dc="http://purl.org/dc/elements/1.1/"
         xmlns:owl="http://www.w3.org/2002/07/owl#"
         xmlns:foaf="http://xmlns.com/foaf/0.1/"
         xmlns:dcterms="http://purl.org/dc/terms/"
         xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
         xmlns:ore="http://www.openarchives.org/ore/terms/"
         xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#">
  <rdf:Description rdf:about="http://foo.org">
    <rdf:type rdf:resource="http://www.openarchives.org/ore/terms/Aggregation"/>
    <ore:aggregates rdf:resource="http://foo.org/bar1.txt"/>
    <ore:aggregates rdf:resource="http://foo.org/bar2.txt"/>
    <dc:title>The Title</dc:title>
    <dc:creator>The Creator</dc:creator>
    <dc:subject/>
  </rdf:Description>
  <rdf:Description rdf:about="http://foo.org/bar.xml">
    <rdf:type rdf:resource="http://www.openarchives.org/ore/terms/ResourceMap"/>
    <ore:describes rdf:resource="http://foo.org"/>
  </rdf:Description>
  <rdf:Description rdf:about="http://foo.org/bar1.txt"/>
  <rdf:Description rdf:about="http://foo.org/bar2.txt"/>
</rdf:RDF>
```

Note: the namespace declarations have been reformatted for readability.

## License

http://www.cyverse.org/license
