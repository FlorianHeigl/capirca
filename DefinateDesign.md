# Table of Contents #


# Definate Design Doc #

Status: Final<br>
Author: Martin Suess<br>
Created: Dec 6th, 2012<br>
Last Updated: Dec 6th, 2012<br>

<h2>Objective</h2>

Definate defines a configuration language and a framework that allows the generation of network definitions for Capirca based on various sources. The generated output can be used in Capirca policies and network definitions.<br>
<br>
<h2>Goals</h2>

<ul><li>Provide a framework to generate lists of network definitions that can be used by the Capirca framework for network policy generation.<br>
</li><li>Provide a modular framework to allow easy integration of new sources for network addresses.<br>
</li><li>Provide a modular filtering mechanism that provides functionality to customize results on multiple levels and allows easy extension.</li></ul>

<h2>Background</h2>

Currently, Capirca does not contain any automation around generating network assets for use in other network definitions or policies. This means that network definitions have to be either maintained manually or an own solution has to be developed to generate definitions. In a huge network environment where most of the network access control lists (network ACLs) are generated automatically, being able to maintain lists of assets is crucial and does not scale if done manually.<br>
<br>
<h2>Overview</h2>

Definate is a generic framework capable of generating ACL definitions, with definitions specified in a configuration file independent of the code to generate them.<br>
<br>
A flexible and intuitive configuration language allows that.  The configuration directs a configured generator to consult an external data source to obtain IPs or networks that comprise a configured definition, then a set of filters will transform the output (the definition list) before it is written to a file or output elsewhere. This includes tasks like resolving host names to generate comments, sorting the entries, and formatting the definition as a human-readable string. Filters can run on mutliple levels (global, file, definition) and can be run before and/or after that level is passed.<br>
<br>
In a first step, Definate will include a DNS generator as an example generator and depending on needs, other generators can easily be added later.<br>
<br>
<br>
<h2>Details</h2>

The high-level program flow is as follows:<br>
<br>
<ul><li>Read configuration file<br>
</li><li>Run specified global level pre-filters<br>
</li><li>Generate files<br>
<ul><li>Run specified file level pre-filters<br>
</li><li>Generate definitions per file<br>
<ul><li>Run specified definition level pre-filters<br>
</li><li>Run actual generator<br>
</li><li>Run specified definition level post-filters<br>
</li></ul></li><li>Run specified file level post-filters<br>
</li></ul></li><li>Run specified global level post-filters</li></ul>

<h3>Generators</h3>

A key role play the generators which are all inherited from the base generator class to provide an equal interface. A generator factory creates generator instances ad hoc when needed by Definate based on the name specified in the configuration file. Each generator has to implement the "GenerateDefinition" function that returns a list of tuples of ipaddr.IPNetwork and string comment.<br>
<br>
<h3>Filters</h3>

Definate has filters that are also inherited from a base filter class to provide an equal interface. A filter factory creates filter instances ad hoc when needed by Definate based on the name specified in the configuration file and the level the generator is at.<br>
<br>
As mentioned earlier, there is a pair of filters (pre- and post-filter) on each level:<br>
<ul><li>Global: Filters that are run as the first and the last thing that Definate does. An example could be checking things out of a versioning system that need to be present for the generation and afterwards checking the changed files in again. Another possibility is to automatically run Capirca to make sure the network policies are up to date after running Definate.<br>
</li><li>File: Filters that are run before and after generating the file content. Examples can be writing the file content to an actual file, printing the outputor even copying the file to a remote system.<br>
</li><li>Definition: Filters that are run before and after generating a single definition. This can for example be used to align output nicely, enhance comments or similar.</li></ul>

<h3>Configuration Language</h3>

The configuration language is explained in detail in the "DefinateConfiguration" page.