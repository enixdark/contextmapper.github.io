---
title: Generic Generator (Templating)
permalink: /docs/generic-freemarker-generator/
---

 * [Introduction](#introduction)
 * [Data Model](#data-model)
 * [User Guide](#user-guide) 

## Introduction
With the generic generator based on [Freemarker](https://freemarker.apache.org/) templates Context Mapper users are allowed to generate arbitrary text files from CML Context Maps.
By providing a corresponding [Freemarker](https://freemarker.apache.org/) template (_*.ftl_ file) one can generate for example Markdown, XML, or any other textual output formats.

### Simple Example
This simple example illustrates how the generator works. The following Freemarker template outputs the names of all Bounded Contexts on the Context Map:

```ftl
The Context Map '${contextMap.name}' contains the following Bounded Contexts:

<#list contextMap.boundedContexts as bc>
${bc.name}
</#list> 
```

Applied to our [insurance example](https://github.com/ContextMapper/context-mapper-examples/tree/master/src/main/cml/insurance-example) the generator produces
the following output:

```text
The Context Map 'InsuranceContextMap' contains the following Bounded Contexts:

CustomerManagementContext
CustomerSelfServiceContext
PrintingContext
PolicyManagementContext
RiskManagementContext
DebtCollection
```

## Data Model
In order to write our Freemarker templates you have to know our data model. In the following we provide an overview over this CML model.

<div class="alert alert-custom">
<strong>Please note:</strong> We do not document the tactic DDD parts within Bounded Contexts in all details. In case your template needs information on that level please use 
our <a target="_blank" href="https://www.javadoc.io/doc/org.contextmapper/context-mapper-dsl/latest/org/contextmapper/dsl/contextMappingDSL/package-summary.html">JavaDoc</a> documentation
to get the necessary information about the data model.
</div>

### Root Level
The root level of the model contains the following elements:

```text
(root)
  |
  +- contextMap                    Context Map object
  |
  +- boundedContexts               Bounded Context list
  |
  +- domains                       Domains list
  |
  +- imports                       Imports list
  |
  +- useCases                      Use cases list
  |
  +- timestamp                     Time stamp with current/generation time
  |
  +- filename                      name of CML file
```

The corresponding CML objects and their syntax can be found in our language reference: [Context Map](/docs/context-map/), [Bounded Context](/docs/bounded-context/), 
[Domains and Subdomains](/docs/subdomain/), [imports](/docs/imports/), [use cases](/docs/aggregate/#use-case-declaration).

### Context Map
The _contextMap_ root element of the model contains the following data: (the illustrated values are just examples)

```text
contextMap
  |
  +- name = "ExampleContextMap"
  |
  +- type = "SYSTEM_LANDSCAPE"
  |
  +- state = "TO_BE"
  |
  +- boundedContexts               List of Bounded Contexts that are added to the Context Map
  |
  +- relationships                 List of Context Map relationships
```

#### Relationships
If you use relationships you must respect the different types as illustrated in our language [meta model](/docs/language-model/). The following types of relationships exist:

 * _SymmetricRelationship_
   * _Partnership_
   * _SharedKernel_
 * _UpstreamDownstreamRelationship_
   * _CustomerSupplierRelationship_
   
Depending on the relationship type (symmetric or upstream-downstream), the objects have a different structure.

The structure of a **SymmetricRelationship** is as follows:

```text
relationship
  |
  +- name = "ExampleRelationship"  Optional relationship name (nullable)
  |
  +- implementationTechnology = "Java Library"
  |
  +- participant1                  Bounded Context object (first relationship participant)
  |
  +- participant2                  Bounded Context object (second relationship participant)
```

The structure of an **UpstreamDownstreamRelationship** on the other hand is as follows:

```text
relationship
  |
  +- name = "ExampleRelationship"  Optional relationship name (nullable)
  |
  +- implementationTechnology = "RESTful HTTP"
  |
  +- upstream                      Bounded Context object (upstream context)
  |
  +- downstream                    Bounded Context object (downstream context)
  |
  +- upstreamRoles                 List of upstream roles (OHS, PL)
  |   |
  |   +- (1st)
  |   |   |
  |   |   +- name = "OHS"
  |   |
  |   +- (2nd)
  |       |
  |       +- name = "PL"
  |
  +- downstreamRoles               List of downstream roles (ACL, CF)
  |   |
  |   +- (1st, 2nd)
  |       |
  |       +- name = "ACL"          Semantic only allows one role here (ACL or CF)
  |
  +- upstreamExposedAggregates     List of Aggregate objects
  |
  +- downstreamGovernanceRights
      |
      +- name = "INFLUENCER"
```

#### Relationship Type Checking
To respect the different structures when processing the relationship list we provide a method that allows you to check the type of the relationship:

```ftl
<#if instanceOf(relationship, SymmetricRelationship)>
...
</#if>
```

```ftl
<#if instanceOf(relationship, UpstreamDownstreamRelationship)>
...
</#if>
```

You can also check which kind of relationship concretely it is:

```ftl
<#if instanceOf(relationship, Partnership)>
...
</#if>
```

```ftl
<#if instanceOf(relationship, CustomerSupplierRelationship)>
...
</#if>
```

### Bounded Context
The _boundedContexts_ root element of the model contains a list of _BoundedContext_ objects. The _BoundedContext_ object has the following structure:

```text
boundedContext
  |
  +- name = "CustomerManagement"
  |
  +- implementedDomainParts        List of implemented domain parts
  |   |
  |   +- (1st)                     (example: first element is a _Domain_)
  |   |   |
  |   |   +- name = "Insurance"
  |   |   |
  |   |   +- domainVisionStatement = "Insurance application vision statement ..."
  |   |   |
  |   |   +- subdomains            List of subdomains
  |   |
  |   +- (2nd)                     (example: second element is a _Subdomain_)
  |       |
  |       +- name = "CustomerManagement"
  |       |
  |       +- domainVisionStatement = "Customer management vision statement ..."
  |       |
  |       +- type
  |       |   |
  |       |   +- name = "CORE_DOMAIN"
  |       |
  |       +- entities              List of Entity objects
  |
  +- realizedBoundedContexts       List of Bounded Context objects
  |
  +- refinedBoundedContext         Bounded Context object
  |
  +- domainVisionStatement = "This Bounded Contexts goal is ..."
  |
  +- type
  |   |
  |   +- name = "FEATURE"
  |
  +- responsibilities              List of responsibilities (strings)
  |   |
  |   +- (1st)
  |   |   |
  |   |   +- "Managing customers"
  |   |
  |   +- (2nd)
  |       |
  |       +- "Managing addresses"
  |
  +- implementationTechnology = "Spring Boot"
  |
  +- knowledgeLevel
  |   |
  |   +- name = "META"
  |
  +- modules                       List of module objects (Sculptor)
  |   |
  |   +- (1st)
  |       |
  |       +- name = "ExampleModule"
  |       |
  |       +- aggregates            List of Aggregate objects
  |
  +- aggregates                    List of Aggregate objects
```

**Note:** The attribute _implementedDomainParts_ of the _BoundedContext_ returns a list of _DomainParts_. A _DomainPart_ can either be a _Domain_ or a _Subdomain_. Both have a _name_
and a _domainVisionStatement_. As long as you only use these two attributes you don't have to check for the type. However, if you want to access the _type_ attribute of a _Subdomain_
you have to check if the _DomainPart_ is a _Subdomain_ first:

```ftl
<#if instanceOf(domainPart, Subdomain)>
...
</#if>
```

### Aggregate
Bounded Contexts contain a set of Aggregates which have the following structure (rough):

```text
aggregate
  |
  +- name = "Customers"
  |
  +- responsibilities              List of responsibilities (strings)
  |   |
  |   +- (1st)
  |       |
  |       +- "Managing customers"
  |
  +- useCases                      List of use case objects
  |
  +- owner                         Bounded Context object
  |
  +- knowledgeLevel
  |   |
  |   +- name = "META"
  |
  +- likelihoodForChange
  |   |
  |   +- name = "NORMAL"
  |
  +- services                      List of Service objects (Sculptor)
  |
  +- resources                     List of Resource objects (Sculptor)
  |
  +- consumers                     List of Consumer objects (Sculptor)
  |
  +- domainObjects                 List of SimpleDomainObject objects (Sculptor)
```

We do not document the structures below Aggregates (Sculptor) at this point. To use them, please consult the 
[JavaDoc](https://www.javadoc.io/doc/org.contextmapper/context-mapper-dsl/latest/org/contextmapper/dsl/contextMappingDSL/Aggregate.html) documentation of the Aggregate.

**Note**: The _domainObjects_ list contains all domain objects such as Entities, Value Objects, etc. In total Sculptor defines the following type hierarchy:

 * SimpleDomainObject
   * BasicType
   * DataTransferObject
   * DomainObject
     * Entity
     * Event
       * CommandEvent
       * DomainEvent
     * ValueObject
   * Enum
   * Trait
   
To respect the different structures of these types you can use our _instanceOf_ method again:

```ftl
<#if instanceOf(domainObject, Entity)>
...
</#if>
```

As already mentioned you find the structure of all those domain objects in our 
[JavaDoc](https://www.javadoc.io/doc/org.contextmapper/context-mapper-dsl/latest/org/contextmapper/dsl/contextMappingDSL/Aggregate.html) documentation.

### Domains
The _domains_ root element of the model contains a list of _DomainPart_ objects which can either be _Domains_ or _Subdomains_. Use our _instanceOf_ method to check of which type
a _DomainPart_ is:

```ftl
<#if instanceOf(domainPart, Subdomain)>
...
</#if>
```

The structure of a _Domain_ object is the following:

```text
domain
  |
  +- name = "Insurance"
  |
  +- domainVisionStatement = "Insurance domain vision statement ..."
  |
  +- subdomains                    List of Subdomain objects
```

The structure of a _Subdomain_ object is as follows:

```text
subdomain
  |
  +- name = "Insurance"
  |
  +- domainVisionStatement = "Insurance domain vision statement ..."
  |
  +- type
  |   |
  |   +- name = "CORE_DOMAIN"
  |
  +- entities                      List of Entity objects
```

### Imports
CML models can import other files via imports. The mechanism is explained [here](/docs/imports/). Imports can be accessed on the root level of the model (_imports_) and have 
the following structure:

```text
import
  |
  +- importURI = "./other-file.cml"
```

### Use Cases
The _useCases_ root element of the model contains a list of _UseCase_ objects which have the following structure:

```text
usecase
  |
  +- name = "UpdateCustomer"
  |
  +- isLatencyCritical = false
  |
  +- nanoentitiesRead              List of nanoentities that are read by this use case (strings)
  |   |
  |   +- (1st)
  |   |   |
  |   |   +- "Customer.firstName"
  |   |
  |   +- (2nd)
  |       |
  |       +- "Customer.lastName"
  |
  +- nanoentitiesWritten           List of nanoentities that are written by this use case (strings)
      |
      +- (1st)
      |   |
      |   +- "Customer.firstName"
      |
      +- (2nd)
          |
          +- "Customer.lastName"
```

### Additional Root Attributes
The following additional attributes are currently available on the root level of the model:

 * _timestamp:_ generation time stamp (for example _26.02.2020 17:20:40 CET_)
 * _filename:_ name of the CML file (for example _ExampleModel.cml_)

## User Guide
To use the generic generator based on [Freemarker](https://freemarker.apache.org/) you need two files within your Eclipse workspace:

 * The input CML model (*.cml)
 * The [Freemarker](https://freemarker.apache.org/) template (*.ftl)
 
 With a right-click on the CML file or within the CML editor you can start the generator. You find it in the _Context Mapper_ context menu:
 
 <a href="/img/generic-generator-context-menu.png">![Generic Textual Generator Context Menu](/img/generic-generator-context-menu.png)</a>
 
 A dialog allows you to select a [Freemarker](https://freemarker.apache.org/) template (*.ftl file) which must be located somewhere in your workspace:
 
 <a href="/img/generic-generator-dialog.png">![Generic Textual Generator Dialog](/img/generic-generator-dialog.png)</a>
 
 By finishing the dialog you generate the required file:
 
 <a href="/img/generic-generator-result.png">![Generic Textual Generator Dialog](/img/generic-generator-result.png)</a>