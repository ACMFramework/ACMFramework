# ACM Framework
The Artefact Consistency Management (ACM) Framework is a PhD research prototype / proof-of-concept tool written in Java. The aim of ACM is to support managing the consistency of heterogeneous software artefacts (the current implementation caters for Java source code, UML diagrams (class, sequence and use case), JUnit test and software architecture (module and conceptual view) artefacts), specifically:

1. The extraction of fine-grained information from original artefacts
2. The transformation of heterogeneous schemas to a property graph representation, where artefact elements are represented by nodes, and trace links connecting them are modelled by edges of the graph.
3. Incorporating a machine learning approach to semi-automatically set up trace links between artefact elements. Artefact elements and their links are stored in a graph database (Neo4j).
4. The detection and identification of changes to artefacts.
5. Graph-based change impact analysis to identify other elements potentially impacted by the modification
6. Rule-based consistency checking of potentially impacted artefact elements
7. Providing suggestions on resolving (potential) inconsistencies.

## Usage
The current implementation does not provide a user interface. The user can invoke the required functionality in the main method of the Startup class (framework.general package). 

Framework setup (manageSetup() method) extracts artefacts from the specified repository, transforms them to .graphml files through XSLT transformation and imports them to the specified graph database. The repository and database paths are configured in the config.properties file. The .graphml files are stored in the local file system in the framework directory and its subdirectories. 

Pre-requisites:

1. Neo4j (version 2.1.3)
2. Mercurial (and local and remote repository paths)
3. Java (version 7)
4. Original artefacts are in local Mercurial repository
5. Artefacts can be exported to an XML-based representation (The src2ml tool (http://www.srcml.org/) can be used to extract Java, C, C# and C++ source code files. Other artefacts may require manual export.)
6. The "ACM" folder and the following subfolders ("ArchitectureConceptual", "ArchitectureModuleView",
"Requirement", "SourceCode", "UMLClass", "UMLSequence", "UMLUseCase", and "UnitTests") are setup in the local file system. When using src2ml for artefact data extraction, the required executables should be copied in the "SourceCode" and "UnitTests" subfolders.
7. Additional XSLT files for new artefact types are supplied.

The manageConsistency() method is called following framework setup and establishing trace links and after changes to any of the artefacts in the repository have been made. At the end of the consistency management process, the framework provides suggestions on resolving (potential) inconsistencies.

## Trace link creation
The framework provides an approach to automatic trace link creation between heterogeneous software artefacts using
machine learning techniques. The functionality can be invoked by calling the manageTraceLinkCreation() method in the Startup class mentioned above. 

The framework provides a data set, obtained from six open source systems, containing 1100 data instances representing trace links between UML use case, UML sequence diagram, UML class diagram, Java source code, JUnit test case, module view and conceptual architecture diagram type software artefacts. The data set provides a foundation for conducting traceability experiments with heterogeneous artefacts using machine learning.

Features description
********************
The following features have been created: NameSimilarity, IsSourceContainer, IsTargetContainer, AbstractionLevelSeparation IsSourceRequirement, IsSourceDiagram, IsSourceArchitecture, IsSourceSourceCode, IsSourceUnitTest, IsTargetRequirement, IsTargetDiagram, IsTargetArchitecture, IsTargetSourceCode, IsTargetUnitTest

Example: 1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1

This data instance can denote the following case: Source is a Java class called "Maze" and target is a UML class called "Maze". The NameSimilarity feature shows that there is a 100% match between the names of the source and the target. Both source and target are container types, hence they are both given value 1 for the IsSourceContainer and IsTargetConatiner features. They are at different abstraction levels, and that is expressed by the value 1. The source is not a requirement, not a diagram, not an architecture, not a unit test but it is a source code type and hence IsSourceSourceCode is given the value 1. The target is not a requirement, not an architecture, not a source code element, not a unit test but it is a diagram element, hence IsTargetDiagram is given the value 1. Since the two elements are related by a trace link, the related flag is set to 1.

Features detailed:
1. Name similarity based on Levensthein distance - score is between 1.0 and 0.0 where 1.0 denotes a 100% match

2. IsSourceContainer value that can take the following values: 1 - it is a container element (that can contain other elements, such as a class can contain methods), 0 - it is a non-container element (such as a method or a field)

3. IsTargetContainer - same check for the target

4. AbstractionLevelSeparation - how far the source and target are from each other in their abstraction levels. Each abstraction level is given a value, e.g. high abstraction level is 0, medium is 1, low is 2. Requirements are high abstraction level, diagrams are medium, source code and unit tests are low abstraction level. Both source and target are assigned this number. The result is the absolute value of the difference between the source and the target.

5. IsSourceRequirement - if the source is a requirement type element, it is assigned the value 1, otherwise 0.

6. IsSourceDiagram - if the source is a UML diagram type element, it is assigned the value 1, otherwise 0.

7. IsSourceArchitecture - if the source is an architecture element, it is assigned the value 1, otherwise 0.

8. IsSourceSourceCode - if the source is a source code type element, it is assigned the value 1, otherwise 0.

9. IsSourceUnitTest - if the source is a unit test type element, it is assigned the value 1, otherwise 0.

10. The same features are applied for the targets

11. RELATED - where 0 denotes non-related (no trace link), 1 denotes related (a trace link exists between source and target)

Data
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Data extracted from original artefacts is stored in .graphml files. Based on the contents of the .graphml files, feature data is generated and is stored in the TrainingData.csv file. The TrainingDataNominal.arff and TrainingDataNumeric.arff files were generated based on TrainingData.csv. The csv files can be found in the data folder.

Data is extraced from the following systems:
1. MazeSolver (https://code.google.com/p/maze-solver/)
Data was exctracted from Java source code and a UML diagram.

2. JGAP (http://jgap.sourceforge.net/)
Data was exracted from Java source code and jUnit tests.

3. Neo4j (https://github.com/neo4j/neo4j)
Data was extracted from Java source code, JUnit tests and a high level module view architecture.

4. Java Binary Parser (https://github.com/raydac/java-binary-block-parser)
Data was extracted from Java source code, JUnit tests and a UML use case diagram.

5. MyRobotLab (https://github.com/MyRobotLab/myrobotlab)
Data was extracted from Java source code, JUnit tests and a sequence diagram.

6. Titan (https://github.com/thinkaurelius/titan)
Data was extracted from Java source code and a high level conceptual view architecture.

Related data instances:
Relationships were manually set up between artefact elements based on human judgement.
Examples:
- A Java class is related to a UML class that has the same methods and name
- A jUnit test case is related to a Java method which is called in its assert statement

Non-related data instances:
Data instances containing pairs that are not related through a trace link were automatically generated by associating elements of a .graphml file with elements of another .graphml file. Human judgement was also required to make sure that artefact elements within the selected files are not actually linked.

Usage
********************

To reproduce the experiments the user supplies their test data in the following form:


		<Relations>
		  <Relation id="1_Inter">
		    <SourceNode>ut0C:\Users\I\Dropbox\TestCasesGraphml\Impl\FixedBinaryGeneTest.graphml
		    </SourceNode>
		    <TargetNode>sc0C:\Users\I\Dropbox\SourceCodeGraphml\Impl\FixedBinaryGene.graphml</TargetNode>
	   </Relation>
	  </Relations>

## Similar Projects
1. EMFTrace (https://sourceforge.net/projects/emftrace/)
2. CLIME (http://cs.brown.edu/~spr/research/envclime.html)
3. ArchTrace (https://github.com/gems-uff/archtrace)

