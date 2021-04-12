# imjs-help
Tips and tricks for users of imjs, the InterMine client-side query package. You should bookmark the imjs API documentation:

http://intermine.org/imjs/

as well as read the README on the imjs repo:

https://github.com/intermine/imjs

## Data Model
The InterMine data model is a Java-based hierarchical data structure. I will try to keep the mines' current data model updated and displayed in the README for each mine.

InterMine data classes (e.g. **Gene**) have three types of class entities:
- **attribute** a simple scalar, typically String, Integer, or Double.
- **reference** a single reference to another class
- **collection** a collection of references to another class

imjs queries are run against the actual data model class names and attributes, references, and collections. These names differ in some cases from the names shown on the mine, which may be aliases for display purposes. For example, `primaryIdentifier` is often called "LIS Identifier" on the mine.

## Example: Gene
Here is the **Gene** data model; **Gene** extends **SequenceFeature**, which is any feature that is located on a chromosome. One does not list the entities for **SequenceFeature** in the **Gene** definition. These are *extra* entities added to the **Gene** class.
```xml
<class name="Gene" extends="SequenceFeature" is-interface="true" term="http://purl.obolibrary.org/obo/SO:0000704">
        <attribute name="briefDescription" type="java.lang.String" term="http://semanticscience.org/resource/SIO_000136"/>
        <attribute name="geneFamilyScoreMeaning" type="java.lang.String"/>
        <attribute name="description" type="java.lang.String" term="http://semanticscience.org/resource/SIO_000136"/>
        <attribute name="geneFamilyScore" type="java.lang.Double"/>
        <reference name="upstreamIntergenicRegion" referenced-type="IntergenicRegion"/>
        <reference name="downstreamIntergenicRegion" referenced-type="IntergenicRegion"/>
        <reference name="panGeneSet" referenced-type="PanGeneSet" reverse-reference="genes"/>
        <reference name="geneFamily" referenced-type="GeneFamily" reverse-reference="genes"/>
        <collection name="flankingRegions" referenced-type="GeneFlankingRegion" reverse-reference="gene"/>
        <collection name="transcripts" referenced-type="Transcript" reverse-reference="gene"/>
        <collection name="introns" referenced-type="Intron" reverse-reference="genes"/>
        <collection name="proteins" referenced-type="Protein" reverse-reference="genes"/>
        <collection name="alleles" referenced-type="Allele" reverse-reference="gene"/>
        <collection name="CDSs" referenced-type="CDS" reverse-reference="gene"/>
        <collection name="proteinDomains" referenced-type="ProteinDomain"/>
        <collection name="exons" referenced-type="Exon" reverse-reference="gene"/>
        <collection name="regulatoryRegions" referenced-type="RegulatoryRegion" reverse-reference="gene"/>
        <collection name="pathways" referenced-type="Pathway" reverse-reference="genes"/>
        <collection name="UTRs" referenced-type="UTR" reverse-reference="gene"/>
</class>
```
So next we look at **SequenceFeature**, which, in turn, extends **BioEntity**:
```xml
<class name="SequenceFeature" extends="BioEntity" is-interface="true" term="http://purl.obolibrary.org/obo/SO:0000110">
        <attribute name="score" type="java.lang.Double" term="http://edamontology.org/data_1772"/>
        <attribute name="scoreType" type="java.lang.String" term="http://ncicb.nci.nih.gov/xml/owl/EVS/Thesaurus.owl#C25284"/>
        <attribute name="length" type="java.lang.Integer" term="http://semanticscience.org/resource/SIO_000041"/>
        <attribute name="assemblyVersion" type="java.lang.String"/>
        <attribute name="annotationVersion" type="java.lang.String"/>
        <reference name="sequenceOntologyTerm" referenced-type="SOTerm"/>
        <reference name="supercontigLocation" referenced-type="Location"/>
        <reference name="chromosomeLocation" referenced-type="Location"/>
        <reference name="supercontig" referenced-type="Supercontig"/>
        <reference name="sequence" referenced-type="Sequence"/>
        <reference name="chromosome" referenced-type="Chromosome"/>
        <collection name="overlappingFeatures" referenced-type="SequenceFeature"/>
        <collection name="childFeatures" referenced-type="SequenceFeature"/>
</class>
```
And then **BioEntity**, which extends **Annotatable**:
```xml
<class name="BioEntity" extends="Annotatable" is-interface="true">
        <attribute name="symbol" type="java.lang.String" term="http://semanticscience.org/resource/SIO_000105"/>
        <attribute name="name" type="java.lang.String" term="http://edamontology.org/data_2099"/>
        <attribute name="secondaryIdentifier" type="java.lang.String" term="http://semanticscience.org/resource/SIO_000675"/>
        <reference name="organism" referenced-type="Organism"/>
        <reference name="strain" referenced-type="Strain"/>
        <collection name="locatedFeatures" referenced-type="Location" reverse-reference="locatedOn"/>
        <collection name="locations" referenced-type="Location" reverse-reference="feature"/>
        <collection name="synonyms" referenced-type="Synonym" reverse-reference="subject"/>
        <collection name="dataSets" referenced-type="DataSet" reverse-reference="bioEntities"/>
        <collection name="crossReferences" referenced-type="CrossReference" reverse-reference="subject"/>
</class>
```
And, finally, **Annotatable** (which implictly extends **InterMineObject**, the mother of all data classes):
```xml
<class name="Annotatable" is-interface="true">
        <attribute name="primaryIdentifier" type="java.lang.String" term="http://semanticscience.org/resource/SIO_000675"/>
        <collection name="ontologyAnnotations" referenced-type="OntologyAnnotation" reverse-reference="subject"/>
        <collection name="publications" referenced-type="Publication" reverse-reference="entities"/>
</class>
```
The **Gene** class contains *all* of these attributes, references, and collections, from `primaryIdentifier` from **Annotatable** all the way up to the Gene-specific attribute `geneFamilyScoreMeaning`.

## Prototyping an imjs query
The easiest way to generate an imjs query is to run the query with the Query Builder on either the legacy mine or the BlueGenes mine, and then, on the results page (which is displayed with the imtables Javascript package) select "Generate JavaScript code" in the selector in the upper right (which defaults to "Generate Python code"):

![generate-javascript-code](https://user-images.githubusercontent.com/5657219/113608202-7bbf3c80-9607-11eb-931f-b972f3a0cb2a.jpg)

This tool will generate the imjs query that you need to make the same query from your web app, along with the imtables call which you probably do not want. Here is the imjs for the above query:

![Screenshot_2021-04-05 SoyMine Query results page](https://user-images.githubusercontent.com/5657219/113608500-e5d7e180-9607-11eb-8fa0-cd6ca9b0d21c.png)

You can save that to your desktop or just cut and paste it into your app.
