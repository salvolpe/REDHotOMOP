# REDHotOMOP
A tool that will allow researchers to leverage the benefits of two leading standards, HL7 FHIR and the OMOP CDM, to improve the quality of observational research in REDCap and integrate findings with the EHR.

There have been recent efforts to combine each of these resources: REDCap and OMOP (at institutional levels); REDCap and FHIR ([Redmatch](https://github.com/aehrc/redmatch)); and [OMOPonFHIR](https://github.com/omoponfhir/omoponfhir-main) and more recently, there has been an announcement of an [official partnership](https://www.ohdsi.org/ohdsi-hl7-collaboration/) between HL7 and the OHDSI network who maintains OMOP. To our knowledge, however, there is no work that leverages all three resources into one system. As a result, we are developing a new system called REDHot OMOP

# Dependencies
All listed hyperlinks navigate to the intended release, which are also included above as submodules
REDCap v10.2.3
* External Modules:
  * FHIR Ontology Autocomplete Module v0.2

OMOPonFHIR v1.2.0
* OMOPv5 instance

Redmatch v1.2.1
