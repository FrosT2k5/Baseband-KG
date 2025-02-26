# Tools Needed:
- CodeQL CLI: https://github.com/github/codeql-cli-binaries/releases
- Jadx: https://github.com/skylot/jadx/releases
- CodeQL VSCode Extension for writing and executing codeql in vscode
- Queries: https://github.com/FrosT2k5/midgard_driver_codeql
# Building CodeQL Database
Command for creating a database from code:
`codeql database create database_path/ -s=./ --language=java --build-mode=none -j 12 --overwrite`
change source code (-s), language, database_path and build mode as per requirements

For C-like languages, it's important to provide build command:
`codeql database create database_path/ -s=./ --language=cpp --command="make" -j 12 --overwrite`
This will compile the code with make and then build the database from source code and compiled objects

# Querying CodeQL Database for CWEs:
To run language specific [CWE](https://codeql.github.com/codeql-query-help/codeql-cwe-coverage/) Queries in CodeQL, we need to download the codeql language pack: 
`codeql pack download codeql/java-all`
change language as required

This will download the CodeQL CWE Queries maintained by CodeQL.
We can run these against any database: 
`codeql database analyze --format=csv --output=out.csv -j 12 database_path/`
This will run all the queries for the language in the database (defined while creating database) and store the result in a csv file (out.csv)

# Running Custom CodeQL Queries
We can write custom queries and execute them on previously created CodeQL Databases.

First, we need to create a CodeQL Workspace which is just a folder with a file "qlpack.yml"
example qlpack.yml:
```
name: kernel-queries
version: 1.0.0
dependencies:
    codeql/cpp-queries: "*"
```
we can list out the dependencies for language queries if needed in this file

Finally, write the query code into a \*.ql file and execute it:
`codeql query run query.ql -d=database_path/ --threads=12 -o output.bqrs`
codeql writes the queries into [bqrs](https://codeql.github.com/docs/codeql-overview/codeql-glossary/#bqrs-file) format by default

We can convert the bqrs file to csv file to interpret the queries result as follows:
`codeql bqrs decode --format=csv -o result.csv output.bqrs`

This will save the query results into `result.csv`  file which can be viewed and processed.
CodeQL can also decode the result into json file format.

# Scripts for automating codeql 

There are few bash scripts that I've written (in the codeql repo) that automate querying and creating database
Repo in: [[Mali Midgard Driver Compilation]]

Scripts:
Few scripts I've written to make database creation and querying easier
- [apk/extract_db.sh](https://github.com/FrosT2k5/midgard_driver_codeql/blob/master/apk/extract_db.sh)
	This scripts decompiles APK file using JADX into java code and creates a new database from it. And then executes common Java CWE Queries and then stores it's results into csv file
- [qlqueries/run_all_queries.sh](https://github.com/FrosT2k5/midgard_driver_codeql/blob/master/qlqueries/run_all_queries.sh)
	This script executes all the codeql queries in it's current folder on the database specified in $database environment variable and store it's results into outputs/ folder in csv file