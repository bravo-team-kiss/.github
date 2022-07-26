# Background

In order to perform its mission the U.S. Air Force's [2d Weather Group](https://www.557weatherwing.af.mil/Units/2d-Weather-Group/) requires regular/recurring access to weather observations from a wide variety of sources. The plethora of weather sensors deployed and managed by a variety of public, private, and defense organizations presents both an opportunity and challenge. In the era of digital, connected systems there are more observational data sources available every day, but these sources are not consistent in how data is retreived nor how it is formatted.

One of those challenging source of data comes from a large number of observation systems deployed by the U.S. Space Force's [45th Weather Squadron](https://www.patrick.spaceforce.mil/About-Us/Weather/) in support of their primary mission to “exploit the weather to assure safe access to air and space.”

# Plan

Not wanting to lose sight of the core requirement of _rapidly_ making additional data streams readily avaible to the _prodigious_ weather modeling capabilities of the 2d Weather Group, our team agreed to resist temptation to become distracted by attempts to build unasked-for predictive or visualization applications. Instead we would remain laser focused on enabling the flow of data, and identifying a format for the data that would present the least possible challenge to the 2d Weather Group when importing into existing systems.

# Our Team

Our team was comprised of a diverse selection of U.S. Space Force personnel in a wide range of roles, and one private contractor. The software development experience level on our team varied widely from a number of members who had completed [Supra Coder](https://supracoders.us/) programs, to a Ph.D. data analyst, and an software developer with 15+ years of experience.

The biggest highlights of our team dynamic were:

 * Regular, cohesive, communication.
 * High level of enthusiasm and commitment.
 * Mutual support through coaching and a willingness to help _wherever_ help was needed.
 * Consistently staying focussed on shared objectives.

# Challenges

## Deciphering Data Formats

The data files made available for this problem were not designed with machine readability in mind and were instead formated as plain text for human readability. Despite being human readable, the data within these files was also largely indecipherable to individuals without domain expertise.

Our team was very fortunate to have a team member with data analyst experience in the weather domain specifically as well as the support of an excellent subject metter expert from the 2d Weather Group.

Eventually a document was provided with details about the information within these files and their formatting.

## Connectivity Limitations

The BRAVO Hackathon organizers did an incredible job providing a secure computing environment in which to operate while handing sensitive data. However the necessary restrictions placed on such an environment presents significant challenges to the modern software developer accustomed to pulling public software libraries, source code, container image, and documentation.

Creating an architecture and developer-operations (DevOps) infrastructure that would allow for rapid developer iteration in this environment represented a significant portion of the engineering effort in our teams final product, and could have use cases well beyond this specific application.

## Diverse Skillsets  

Because the members of the team had a diverse set of skills with which to contribute, it was necessary to come up with a solution that would benefit from the various efforts with which or team members were most comfortable. Specifically the solution needed to support a variety if programming languages and runtime environments, as well as being capable for integrating tools and libraries developers are generally accustomed to using.

# What We Built

At a high level the system we implemented consisted of 1) a central database, 2) a number if indenpendent microservices to parse different data formats and insert data into that database, and 3) a web UI to allow users to download the desired data. Underlying these components was a DevOps infrastructure that enabled developers to rapidly iterate on individual components of the system in spite of connectivity limitations.

### Conceptual Architecture

![conceptual architecture diagram](https://github.com/bravo-team-kiss/.github/raw/main/Conceptual.png)

### Infrastructure

![infrastructre diagram](https://github.com/bravo-team-kiss/.github/raw/main/Infrastructure.png)

## Central Data Storage

From the nature of the iregular time series data being analyzed it was immediately clear that a columnar time series data would be appropriate. Influx DB seems particularly appropriate because 1) it has client libraries for numerous programming environments as well as a trivial REST API when a library is not available, 2) Its differentiation between "tags" which describe the unique attributes of a particular sensor, and "fields" which capture the measured values from the sensor, was particularly well suited for the data, 3) it includes a powerful and performant query language for data transformation and retrieval, 4) it is specifically designed to handle sparse time series (such a records of events like lightning strikes) and a dynamic set of "fields" per measurement, and 5) it is open source and free to use.

## Ingest Microservices

Each ingest service followed a very simple pattern: they were implemented as a simple command line application, that would be executed for an input file, and would parse that file and input the resulting data into InfluxDB. In adition to having the path of the input file specified as a command line argument, a number of stardard environment variables wer set to configure the destination for the data.

Each microservice was containerized separately via a Dockerfile so that it could be invoked in different environments and isolated from the host system. This enabled each developer to choose whatever programming environment the preferred, and ensured that dependencies would be present when deployed the offline network environment.

## REST API & Web UI

Our REST API and Web UI were fairly simple and meant for demonstration purposes only. The backend was implemeneted using Node.js with Express, and the frontend with React.js. They both implemented the ability to list data source types, produce data exports in JSON format.

The JSON format we used was based on discovery sessions with personnel from the 2d Weather Group and designed to maximize machine readability.

## DevOps Platform

In order to facilitate rapid developer iteration we created a virtual machine definition taking an “Infrastructure as Code” approach using [Packer](#TODO-link-too-hashicorp-packer). By using this approach it was possible to ensure repeatability of the platform configuration, and deploy small tweaks without having to rebuild the entire virtual machine.

The virtual machine hosted primary services in Docker, which included InfluxDB and a Docker image registry. The image registry played a critical part in enabling rabid developer iteration because it was not necessary to rebuild the platofm for minor code changes, instead we simply transfered container images to the secure network and pushed them to the image registry.

Parser images, once pushed to the image registry, were leveraged by the backend server to run data parsing jobs within Docker using the latest image. This ensured that the backend was decoupled from the individual parser implementations.

# The Future

What we build was just a proof of concept for a more advanced ETL system and data store for collecting and diseminating large volumes of disparate sensor data (be it weather data or other sensor information).

## Automation

Our proof of concept required data files to be manually uploaded and downloaded. Ideally data files would be automatically parsed as soon as they are available. This could be done using a number of standard ETL or workflow automation tools capable of executing jobs in response to events such as file uploads or message publishing.

Additionally exports of data could be scheduled for regular recurrence, dispatching data to a desired endpoint such as a HTTP REST API, sftp server, or cloud storage bucket.

## Configurable Exports

Currently the only configurability for data exports is sensor type and time range. Additional configuration could include sample/window size, aggregate functions, data transformations, or the ability to join multiple timeseries.

## Realtime Import

The system as it exists is designed to process data in batches mad available on a regular basis. It may be useful, however, to enable the importing of data in real time via a high performance message queueing and processing system such as Apache Kafka.

## Mesh Networked Sensor Array

In the scenario we were working on there was a discrete set of existing sensors that were not being utilized for modelling due to the difficulty of importing the data, and our initial approach solved that immediate issue. However our approach could also enable solutions such as deploying a grid arrangement of low cost mesh networked weather observation stations that would be even better for high resolution modeling purposes.

## Security

The REST API and Web UI currently have no Authentication or Authorization. It may be necessary to restrict access to certain data sources, and it will certainly be necessary to control who is able to modify data export configurations.

# Credits

* [Patrick Hughlett](https://github.com/phughlett) - Man With the Plan / Parser Developer (Node.js)
* [Andy Wagers, Ph.D.](https://www.linkedin.com/in/andrew-wagers/) - Data Wrangler / Subject Matter Expert / Parser Developer (Python)
* [Aileen Ocampo](https://github.com/aileenocampo) - Parser Developer (Java) / Frontend Developer (React.js)
* [Alexander Parris](https://github.com/manwithaplan-pepsiman) - Backend Developer (Node.js)
* [Dustin Stringer](https://www.linkedin.com/in/dustin-stringer-4a666480/) - Parser Developer (Node.js) 
* [Roemello McCoy](https://www.linkedin.com/in/roemello-mccoy-9a183110b/) - Parser Developer (Python)
* [Paul Wheeler](https://www.linkedin.com/in/paulwh/) - Software Architect / DevOps Engineer

## Thanks

 * [Mike Lanciano](https://www.linkedin.com/in/mikelanciano/) - For pitching in with containerization efforts while also working on his own project.
