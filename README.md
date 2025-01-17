String2Vocabulary
=================

[![](https://jitpack.io/v/DOREMUS-ANR/string2vocabulary.svg)](https://jitpack.io/#DOREMUS-ANR/string2vocabulary)

Look for literals in an RDF graph and substitute them with URIs from controlled vocabularies.
Built with [Gradle](https://gradle.org/) and [Apache Jena](https://jena.apache.org).

It uses the vocabulary filenames for grouping them in **families**. For example, `city-italy.ttl` and `city-france.ttl` are part of the family `city`.

## Input

The library needs in input:
- a **folder** containing vocabularies
- for a full graph replacement, a **configuration in csv** that declares the property to match, the relative vocabulary family, an if it should eventually check for the singular version of the label. Example:

```csv
http://data.doremus.org/ontology#U2_foresees_use_of_medium_of_performance,mop,singular
http://data.doremus.org/ontology#U11_has_key,key,
```

Given as input:
```turtle
ns:myMusicWork mus:U11_has_key [
          a mus:M4_Key ;
          rdfs:label "Ré majeur"@fr ] ;
   mus:U2_foresees_use_of_medium_of_performance "mezzosoprano" .
```

... this produces as output:

```turtle
ns:myMusicWork mus:U11_has_key <http://data.doremus.org/vocabulary/key/d> ;
   mus:U2_foresees_use_of_medium_of_performance  <http://data.doremus.org/vocabulary/iaml/mop/vms> .
```

## Features

- Vocabulary syntax supported:
  - SKOS
  - MODS
- Support for families of vocabularies
- Replace literals that match the given label
- Replace objects that have a `rdfs:label` or `ecrm:P1_is_identified_by` which match the given label
- _Strict mode_: match both label and language
- Normalise the labels by removing punctuation, decoding to ASCII, using lowercase
- Search also for the singular version of the word with [Stanford CoreNLP](https://github.com/stanfordnlp/CoreNLP)
- Support for [RDF Dataset](https://www.w3.org/TR/sparql11-query/#rdfDataset):
  - replace content at the *default graph* level
  - replace content at a given *named graph* level
- Supported textual syntax for RDF (serialization):
  - [RDF Turtle](https://www.w3.org/TR/turtle/) (.ttl)
  - [TriG](https://www.w3.org/TR/trig/) (.trig)

Dependencies:
* Build tool: [Gradle](https://gradle.org/) 7+
* See the `dependencies` section in the [build.gradle](build.gradle) file for project dependencies.

## Usage

### As a module

1. Add it as dependency. E.g. in `build.gradle`:

  ```
  dependencies {
     compile 'com.github.DOREMUS-ANR:string2vocabulary:0.7'
  }
  ```

2. Import and init in your Java class

  ```java
  import org.doremus.string2vocabulary.VocabularyManager;

  // ...

  // print full logs
  VocabularyManager.setVerbose(true);

  // set the folder where to find vocabuaries
  VocabularyManager.setVocabularyFolder("/location/to/vocabularyFolder");
  // set the folder where to find the config csv
  VocabularyManager.init("/location/to/property2family.csv");
  // set the language to be used for singularising the words
  VocabularyManager.setLang("fr");
  ```

3. Use it :)

```java
// Search for a term in a given family
// this performs a normal full search and one in strict mode
VocabularyManager.searchInCategory("violin", "en", "mop");
// --> http://www.mimo-db.eu/InstrumentsKeywords/3573

// or
// Search for a term in a given vocabulary
VocabularyManager.getVocabulary("mop-iaml").findConcept("violin", false);
// --> http://data.doremus.org/vocabulary/iaml/mop/svl
// strict mode
VocabularyManager.getVocabulary("mop-iaml").findConcept("violin@it", true);
// --> null

// or
// Get the URI by code (what is written after the namespace)
VocabularyManager.getVocabulary("key").getConcept("dm");
// --> http://data.doremus.org/vocabulary/key/dm

// or
// Full graph replacement
// search and substitute in the whole Jena Model
// (following the csv configuration)
VocabularyManager.string2uri(model)
```

See the [test](src/test) folder for another example of usage.

### Command Line

Run the library from CLI with `gradle run`:
```shell
# Canonical form
gradle run -Pmap="/location/to/property2family.csv" \
  -Pinput="/location/to/input.ttl" \
  -Pvocabularies="/location/to/vocabularyFolder"
```

Available CLI parameters:

| param | example | comment |
| ----- | ------- | ------- |
| map   | `/location/to/property2family.csv` | A table with mapping property-vocabulary |
| vocabularies   | `/location/to/vocabularyFolder` | Folder containing the vocabularies in turtle format |
| input   | `/location/to/input.ttl` | The input file (Turtle or TriG syntax) |
| output _(Optional)_   | `/location/to/output.ttl` | The output turtle file. Default: `<inputPath/inputName>_output.<inputFileExt>` |
| lang  _(Optional)_  | `fr` | Language to be used for singularising the words. Default: `en`. |
| graph _(Optional)_ | `http://example.org/graph/object/` | The [named graph](https://en.wikipedia.org/wiki/Named_graph) to process. Default: `` (i.e. the default graph) |

Default `gradle run` behavior rely on *project properties* set in the [gradle.properties](gradle.properties) file.
See the following links for details about *properties* in Gradle:
* [Passing Command Line Arguments in Gradle](https://www.baeldung.com/gradle-command-line-arguments)
* [Gradle project properties best practices](https://tomgregory.com/gradle-project-properties-best-practices/)
* [Configuring Gradle with "gradle.properties" ](https://dev.to/jmfayard/configuring-gradle-with-gradle-properties-211k)

CLI examples with provided test files:

```shell
# Example: Turtle syntax
gradle run -Pmap="src/test/resources/property2family.csv" \
  -Pinput="src/test/resources/input.ttl" \
  -Pvocabularies="src/test/resources/vocabulary"

# Example: TriG syntax, replace at the default graph level
gradle run -Pmap="src/test/resources/property2family.csv" \
  -Pinput="src/test/resources/input.trig" \
  -Poutput="src/test/resources/output.trig" \
  -Pvocabularies="src/test/resources/vocabulary"

# Example: TriG syntax, replace at the default graph level (alternative)
gradle run -Pmap="src/test/resources/property2family.csv" \
  -Pinput="src/test/resources/input.trig" \
  -Poutput="src/test/resources/output.trig" \
  -Pvocabularies="src/test/resources/vocabulary" \
  -Pgraph=""

# Example: TriG syntax, replace at a given named graph level
gradle run -Pmap="src/test/resources/property2family.csv" \
  -Pinput="src/test/resources/input.trig" \
  -Poutput="src/test/resources/output.trig" \
  -Pvocabularies="src/test/resources/vocabulary" \
  -Pgraph="http://example.org/graph/object/"
```

### Documentation

Generating local code documentation:

```shell
javadoc -d doc/ ./org/doremus/string2vocabulary/VocabularyManager.java
```

References:

* https://www.tutorialspoint.com/java/java_documentation.htm

## Contribute

In the general case, please
* *fork and create merge request* OR
* *raise an issue* into the project's space.
