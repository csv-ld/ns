# CSV-LD: Spreadsheet-based Linked Data

In this talk, I will introduce the idea and practice of Linked Data, and I will propose a way to produce Linked Data from the comfort of spreadsheet-based data entry.

## Linked Data is a formal way to identify context within data

Linked Data is a formal way to identify context within data.

By formal, I mean that a machine processing the data can find, access, and interpret the context. A typical README file is not formal, in that while a human can read it and identify the section that lists terms and their expected data types, for instance, a machine will not be able to find this without some guidance in the form of a custom program that can parse the bespoke structure of the README file.

By context, I mean the concepts and relationships that are needed to understand both the data’s meaning and the data’s formatting.

By *within* data, I mean within the same delivered artifact as the data itself. Often, the documentation for a data object is not directly linked or provided in-line within the data object. Even if this context is delivered alongside the data object, for example as a README file in the same shared directory, there is a risk that the data object will be lifted and forwarded along without its sibling.

Sir Tim Berners-Lee’s note on Linked Data gives a succinct description of its principal expectations. First, use uniform resource identifiers, URIs, as names for things. Second, use HTTP URIs so that people can resolve these names with Web browsers. Third, when someone requests such a URI, provide a formal response using the Web standard resource description framework, or RDF. Finally, your Linked Data should link to other people’s Linked Data when appropriate, to provide even more context, just like how our web documents link to other people’s web documents to provide additional context.

## A JSON-LD document is a JSON document that includes its context

There are many standard serializations for Linked Data. One of them is JSON-LD, where the LD stands for Linked Data.

Using Javascript Object Notation, or JSON, to serialize data is quite popular. A JSON-LD document is a JSON document that includes its context.

Here, I show an example JSON-LD document. This is also shown at json-ld.org. The document makes statements about a resource identified by the value of the @id key. That value is a uniform resource identifier, or URI, under a different namespace, at the DBpedia.org domain, with different governance than json-ld.org. And that’s okay.

The JSON-LD document has another special key, @context, that links to its context to help a human and their machine agents to understand the meaning and formatting of this JSON document.

If we resolve the @context link, we get a map of vocabulary terms to URIs that in turn have more information. Our processor might already know about XML Schema Datatypes and doesn’t need to follow that link to know what to expect for date formatting. We see that the spouse field is to be interpreted not as a general string value, but as a URI, given the @type directive in the context.

JSON-LD is a great way to ensure that a JSON dataset’s context travels with the data, whether served via a document database like MongoDB, or via a web API that returns requests for data as JSON responses.

## CSV on the Web (CSVW) helps you build sidecars for spreadsheets

But we’re not here for JSON – we’re here for spreadsheets! How can I power-up my spreadsheet to be Linked Data?

CSV on the Web, or CSVW, is a standard that helps you build sidecars for spreadsheets.

The term “sidecar” is used for a functional addition. A motorcycle sidecar can carry things and people. A Kubernetes sidecar container has access to the namespace and storage volumes of its pod’s main container, and so supports auxiliary work. Unstructured documentation, for example a typical README file, is not a functional sidecar.

The World Wide Web Consortium’s “CSV on the Web” working group published seven documents, including a note on 25 identified use cases, and and a primer on effective use of its recommendations in practice. For example, when you’re serving a csv file like mydata.csv, you can serve a JSON-LD sidecar by adding -metadata.json to the name, and you use the CSVW vocabulary to provide extra information about your data.

## Linking and Packaging are complementary

Before I elaborate on spreadsheet-based Linked Data, I want to highlight the relationship between linking and packaging.

Linking and Packaging are complementary techniques. With Linked Data, your entry point is the data, which links out to context and other data. With Data Packages, your entry point is the package, which links in to the contained data and its metadata.

A great example of data packaging is the Frictionless Data family of specifications. Just like a Dockerfile can package up a filesystem and provide an entry point to its resources, a data package yields a container that facilitates white-glove service to the data within.

When it comes to linking, links can be embedded within the data, like what I showed with the @context field of JSON-LD, or they can be not embedded within the data, as is the case with CSVW sidecars or link headers.

## Barcode labels for columns

What could a standard for embedded links do for spreadsheets?

Just like you can print out barcode labels to stick onto sample carriers in a laboratory, you could mint barcode labels for your columns.

Instead of, or in addition to, a column header label that may or may not be understood by a data consumer, a barcode label would let someone unambiguously fetch the meaning and format of a column.

## “Go to definition” for columns

Many code editors have a “go to definition” or “go to declaration” function -- you can control-click on a symbol in your code, and the editor will take you to where that symbol is defined, even if the definition is in another module or package you have imported.

If we use HTTP URIs for our barcodes, then we can use the existing infrastructure of the Web to read them! Here I show a basic example of a CSV-LD file . The formatVersion directive says what version of the CSV-LD format should be assumed for this file. The vocabBase directive points to a “vocabulary base”, which is like a “data base”, but for vocabulary. A tool reading this file can help resolve each column label in the header to a definition in the vocab-base.

I’ll now delve a little deeper into CSV-LD, including more on the formatVersion and vocabBase directives.

## It is still a delimiter-separated-values file

First, a CSV-LD file is still a CSV file. Whatever comes before formatVersion on the first line is inferred to be the comment prefix, and whatever comes after formatVersion and before the H in the HTTP URI is inferred to be the delimiter.

The CSV on the Web vocabulary includes terms for dialect descriptions. CSVW identifies the octothorpe as the default comment prefix and the comma as the default delimiter, and that’s what I use in my examples.

## “What is going on here?” “Follow your nose.”

The goals of the formatVersion directive on the first line are two-fold. The first goal is to help someone new to the format learn more. “Follow your nose” is a widely celebrated Linked Data pattern. The idea is that people as well as their software agents will sniff out links and follow them when it makes sense for them to do so. Hence, the second goal: help someone’s software agents know what to expect in the rest of the file.

## Reuse header rows with a ”vocab-base”

The next line is a directive that identifies a vocabulary base for column labels. If a column label is not given as a URI, then the vocab base can be queried for the label.

There are two cases here. Either the column label is the last part of the corresponding URI, with the vocab base URI as the prefix, or there is a full URI in the vocab base that has been annotated with the column label as a human-readable label for the URI. In this way, a data producer can retain preferred column labels, even if they contain spaces or otherwise would not form part of a valid URI.

## Reuse vocabularies, succinctly

As an alternative to maintaining all column vocabulary terms in one vocab base, you can directly reuse vocabularies maintained by others. Here, I enable succinct linking to terms hosted by the example.org organization, which I trust. I link to the “atom” entry in its namespace for terms from the International Union of Pure and Applied Chemistry. I do this because doing so can signal to a CSV-LD processor that all values for this column must be standard symbols for atomic elements.

If you like another organization’s vocabulary, but you don’t trust their governance, you can always fork their terms into your vocab base or to a prefix namespace you govern, maintaining links upstream to the source terminology.

## Include other metadata via RDF statements

Apart from directive statements, you can also include RDF statements in the header. Resource description framework statements are in the form of triples, with a subject, predicate, and object. The subject of each triple is the current sheet. The predicate is always a URI, and in this example I am saying that CSVW metadata for this sheet is available at the URI given as the object of the triple.

The header also accommodates comment lines, for which the comment prefix is followed directly by a space character.

Finally, there is one additional directive in CSV-LD beyond formatVersion, vocabBase, and prefix, and that is the id directive, which allows you to assign a URI to identify the sheet and thus the subject of any RDF statements in the header.

## Vocabulary terms are resolvable HTTP URIs

From the FAIR Guiding Principles for scientific data management and stewardship, where FAIR stands for findable, accessible, interoperable, and reusable, one of the elements of interoperability is that data and metadata vocabularies themselves follow FAIR principles. In this case, the vocabulary terms need to be resolvable so people can look them up. There are different ways to do this. One way is to host a static site with an HTML document for a human-readable version of the vocabulary, and to have the HTML document include a link in its header to an RDF document that is related as an alternative, machine-actionable representation of the vocabulary. RDF Site Summary feeds, that is, RSS feeds, are also communicated via such HTML header links.

## Near term: CSV-LD to validate and to collect

In the near term, I hope that CSV-LD proves useful for validating and collecting data from individual spreadsheets.

Columns can be validated independently of each other. Beyond that, a statement in the header may indicate to a processor that each row in the sheet should validate as a certain kind of entity, meaning certain columns are required to be present and valid, and other optional columns, if present, will also be validated and collected.

Furthermore, there are standard affordances in RDF ontologies such as property hierarchies that could help to make the use of vocabularies flexible and to layer on new meaning over time via compatible additive changes rather than breaking changes.

## Someday: CSV-LD to unify and to discover

Eventually, I hope that the basic column and entity validation facilities of CSV-LD processors will allow you to unify known spreadsheets under your governance via column URIs, and ultimately to discover related spreadsheets hosted elsewhere by allocating computation budgets to your software agents to follow their noses for more Linked Data.

## After this talk, come find me

After this talk, come find me in the csv,conf Slack. Also, if you raise issues in the CSV-LD namespaces github repository, I will see them! Or, feel free to email me.

If you are a materials scientist or know one, I am involved with a data dictionaries working group as part of the materials research data alliance. Come join the discussion on matsci.org.

If you are a microbiome researcher or know one, I am also involved with a microbiome data collaborative, and you can see what we’ve been cooking up as a pilot project.

Finally, the Recurse Center is a great venue for a professional mini-sabbatical, and if you are a Recurser, I am one too, and I hang out in the Zulip chat a lot.
