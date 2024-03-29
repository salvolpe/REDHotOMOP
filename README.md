# REDHotOMOP
A tool that will allow researchers to leverage the benefits of two leading standards, HL7 FHIR and the OMOP CDM, to improve the quality of observational research in REDCap and integrate findings with the EHR.

There have been recent efforts to combine each of these resources: REDCap and OMOP (at institutional levels); REDCap and FHIR ([Redmatch](https://github.com/aehrc/redmatch)); and [OMOPonFHIR](https://github.com/omoponfhir/omoponfhir-main) and more recently, there has been an announcement of an [official partnership](https://www.ohdsi.org/ohdsi-hl7-collaboration/) between HL7 and the OHDSI network who maintains OMOP. To our knowledge, however, there is no work that leverages all three resources into one system. As a result, we are developing a new system called **REDHot OMOP**

Click [here](https://github.com/salvolpe/REDHotOMOP/blob/main/REDHot_OMOP_Abstract_2021.pdf) to read the recent conference abstract documenting the status of this project in greater detail (as of October 2021).

# Dependencies
All listed hyperlinks navigate to the intended release, which are also included above as submodules

REDCap v10.2.3
* External Modules:
  * FHIR Ontology Autocomplete Module v0.2 (accessible through local REDCap's *External Modules* section)

[OMOPonFHIR v1.2.0](https://github.com/omoponfhir/omoponfhir-main/releases/tag/v1.2.0)
* OMOPv5 or v6 instance (REDHotOMOP was implemented with a [dummy PostgresQL OMOPv5 server](https://github.com/omoponfhir/omopv5fhir-pgsql/) from GA Tech)
* This requires [Docker](https://docs.docker.com/get-docker/) installed on your machine to instantiate and maintain the servers

[Redmatch v1.2.1](https://github.com/aehrc/redmatch/releases/tag/1.2.1)
* mvn - Recommended that you install through [homebrew](https://brew.sh) with `brew install mvn`

# System Architecture
![REDHot_OMOP_Prototype](https://user-images.githubusercontent.com/37944330/140053395-a8e087f2-a6c2-4cdd-82a8-cf7333a52bb7.png)

# Setup
*How do you set up your environment correctly to use REDHotOMOP?*

## REDCap
If you do not already have REDCap, you will need to contact a member of your institution to secure access to the REDCap system. Protocols and availability differs from place to place. Make sure that your machine meets the [technical requirements](https://projectredcap.org/software/requirements/). There are two different installation guides you can examine based on your OS:
* [Windows](https://community.projectredcap.org/storage/attachments/6318-20180417redcapinstallation.docx) - Requires REDCap account login
* Mac/Linux/Unix
  * You need to run REDCap on a web server, e.g. localhost. If you take the local route, you will need to set up Apache and MySQL on your machine configured as shown [here](https://www.maketecheasier.com/setup-local-web-server-all-platforms/) (easiest on Linux and Unix)
    * When running on localhost, make sure that your MySQL section of the `database.php` file in `/redcap` looks as follows:
    ```
    $hostname 	= '127.0.0.1'; //We cannot use localhost here because the MySQL database connects using TCP for redcap
    $db 		= 'redcap';
    $username 	= 'redcap_user';
    $password 	= 'your_redcap_password';
    ```
  * In your prefered IDE, e.g. MySQL Workbench, you will want to run the following code:
  ```
  CREATE DATABASE IF NOT EXISTS `redcap`;
  CREATE USER 'redcap_user'@'your_host_here' IDENTIFIED WITH mysql_native_password BY 'your_redcap_password';
  GRANT SELECT, INSERT, UPDATE, DELETE ON `redcap`.* TO 'redcap_user'@'your_host_here';
  ```
  * Subsequently, you will have to run a large REDCap Installation SQL code block, which will take 5-10 minutes to execute, depending on your machine.
    * N.B. The code is not included here as this may change from version to version of REDCap. When you attempt to load `http://<your_host>:PORT####/routeIfNeededTo/redcap/install.php`, it will provide you with the additional code and information to complete installation.
* After installation, you can navigate to the Control Center and then in the side-menu under Technical / Developer Tools, select *External Modules*

## OMOPonFHIR
* *Prior to running OMOPonFHIR*, you will need to have an OMOP instance. If you do not have one yet, or want to connect to a test server, you can use [this OMOPv5 pgSQL server](https://github.com/omoponfhir/omopv5fhir-pgsql/) from GA Tech. (This also requires Docker)
  * If you are taking this route, be mindful that the SQL server defaults to port 5432. So you should change the port number when you perform the run command through docker to 5438:5432. This will avoid issues with keycloak (a service within Redmatch) accidently finding your SQL server and being unable to access its REDCap project handling service
* Follow the steps in the README for OMOPonFHIR for installation with the following considerations/adjustments:
  * OMOPonFHIR defaults to READ-ONLY, but you will need to write to this server. Navigate to `omoponfhir-main/omoponfhir-r4-server/src/main/webapp/WEB-INF/web.xml` where you will need to change `readOnly` to `False` at line 95.
  * If you use the `omopv5fhir-pgsql` server, you need to set the environment (ENV) variables in the `omoponfhir-main` Dockerfile as follows:
  ```
  ENV JDBC_URL=jdbc:postgresql://smart-postgres:5432/postgres?currentSchema=omop_v5 JDBC_USERNAME=omop_v5 JDBC_PASSWORD=i3lworks AUTH_BASIC=your_username:your_secret
  ```
  * To better understand the variables, particularly when working with the `omopv5fhir-pgsql` server, here is a breakdown of the variables in the Dockerfile:
    * JDBC_URL=jdbc:[dbclient]://[container-name]:[port-number]/[dbclient-command-form]?currentSchema=[POSTGRES_DB]
    * For currentSchema, JDBC_USERNAME, JDBC_PASSWORD look in the Dockerfile for `omopv5fhir-pgsql`
      - currentSchema, listed after POSTGRES_DB
      - JDBC_USERNAME, listed after POSTGRES_USER
      - JDBC_PASSWORD, listed after POSTGRES_PASSWORD
  * Both OMOPonFHIR and Redmatch use port 8080 as their default, and it is easier to adjust the port in OMOPonFHIR, so when running `docker run` send after the `-p` flag send it to `80:8080`
  * It is strongly recommended that you build and run rather than use dockercompose. If you try dockercompose and find a 404 error when trying to load the HAPI FHIR server, even if your database and omoponfhir containers are connected, do the following:
    ```
    docker network create network_name
    docker network connect network_name your_database_container_name
    docker network connect network_name your_omoponfhir_container_name
    sudo docker build -t omoponfhir . 
    sudo docker run --name omoponfhir --network=omop -p 8080:8080 -d omoponfhir:latest
    ```
    * This route only works when your database is in a Docker container. If that is not the case, you may not come across this issue.

### (TBD) - How to load ConceptMaps into OMOPonFHIR

## Redmatch (TBD)
* Redmatch relies on a Docker image to install, including the usage of docker-compose. 
* After running `mvn clean verify` as specified in the README, be sure to do the following:
  * If `docker-compose up -d` gives you an error similar to the one below, be sure to append `sudo` before `docker-compose`
    ```
     => ERROR [internal] load metadata for docker.io/jboss/keycloak:latest                                                                                                              0.2s
     ------
     > [internal] load metadata for docker.io/jboss/keycloak:latest:
     > ------
     > failed to solve with frontend dockerfile.v0: failed to create LLB definition: rpc error: code = Unknown desc = open /Users/sal/.docker/.token_seed: permission denied
     > ERROR: Service 'keycloak' failed to build
    ```
* In order to access the Redmatch interface, you will need an account. To do so, after the Docker containers are running, navigate to http://localhost:10001/auth/admin/master/console/#/realms/Aehrc/users and add a new user.
  * After you have a user, you can login at http://localhost:8888
* Redmatch and OMOPonFHIR both use 8080 as a default, so be sure to change the ports environment variable in the docker-compose.yml for Redmatch under keyclock to 8090:8080.
* If you are running REDCap on your local machine, in order to access it from within the Redmatch docker instance you need to replace `localhost` in the API URL from REDCap with `host.docker.internal`
# Installation
* (TBD) Clone this repository and install with the External Modules directions from REDCap
