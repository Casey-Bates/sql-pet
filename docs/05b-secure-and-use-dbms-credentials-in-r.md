# Securing and using your dbms credentials (05b)

> This chapter demonstrates how to:
>
>  * Keep necessary credentials secret while being available to R when it executes.
>  * Interact with PostgreSQL using your secret dbms credentials

Connecting to a dbms can be very frustrating at first.  In many organizations, simply **getting** access credentials takes time and may involve jumping through multiple hoops.  

In addition, a dbms is terse or deliberately inscrutable when your credetials are incorrect.  That's a security strategy, not a limitation of your understanding or your software.  When R can't log you on to a dbms, you will have no information as to what went wrong.

The following packages are used in this chapter:

```r
library(tidyverse)
library(DBI)
library(RPostgres)
require(knitr)
library(sqlpetr)
```
## Set up the sql-pet docker container

### Verify that Docker is running

Check that docker is up and running:


```r
sp_check_that_docker_is_up()
```

```
## [1] "Docker is up but running no containers"
```
### Start the docker container:

Start the sql-pet docker container:

```r
sp_docker_start("sql-pet")
```

## Storing your dbms credentials

In previous chapters the connection string for connecting to the dbms has used default credentials specified in play text as follows:

  `user= 'postgres', password = 'postgres'`

When we call `sp_get_postgres_connection` below we'll use environment variables that R obtains from reading the *.Renviron* file when R starts up.  This approach has two benefits: that file is not uploaded to GitHub. R looks for it in your default directory every time it loads.  To see whether you have already created that file, use the R Studio Files tab to look at your **home directory**:

![](screenshots/locate-renviron-file.png)

That file should contain lines that **look like** the example below. Although in this example it contains the PostgreSQL <b>default values</b> for the username and password, they are obviously not secret.  But this approach demonstrates where you should put secrets that R needs while not risking accidental uploaded to GitHub or some other public location..

Open your `.Renviron` file with this command:

>
> `file.edit("~/.Renviron")`
>

Or you can execute [define_postgresql_params.R](define_postgresql_params.R) to create the file or you could copy / paste the following into your **.Renviron** file:
```
DEFAULT_POSTGRES_PASSWORD=postgres
DEFAULT_POSTGRES_USER_NAME=postgres
```
Once that file is created, restart R, and after that R reads it every time it comes up. 

### Connect with Postgres using the Sys.getenv function

Connect to the postgrSQL using the `sp_get_postgres_connection` function:

```r
con <- sp_get_postgres_connection(user = Sys.getenv("DEFAULT_POSTGRES_USER_NAME"),
                         password = Sys.getenv("DEFAULT_POSTGRES_PASSWORD"),
                         dbname = "dvdrental",
                         seconds_to_test = 10)
```
Once the connection object has been created, you can list all of the tables in the database:

```r
dbListTables(con)
```

```
##  [1] "actor_info"                 "customer_list"             
##  [3] "film_list"                  "nicer_but_slower_film_list"
##  [5] "sales_by_film_category"     "staff"                     
##  [7] "sales_by_store"             "staff_list"                
##  [9] "category"                   "film_category"             
## [11] "country"                    "actor"                     
## [13] "language"                   "inventory"                 
## [15] "payment"                    "rental"                    
## [17] "city"                       "store"                     
## [19] "film"                       "address"                   
## [21] "film_actor"                 "customer"
```
<!--
The following needs to be expanded and explained -- it doesn't stand on its own as written.
### Alternative: put the database password in an environment file

The goal is to put the password in an untracked file that will **not** be committed in your source code repository. Your code can reference the name of the variable, but the value of that variable will not appear in open text in your source code.

We have chosen to call the file `dev_environment.csv` in the current working directory where you are executing this script. That file name appears in the `.gitignore` file, so that you will not accidentally commit it. We are going to create that file now.

You will be prompted for the database password. By default, a PostgreSQL database defines a database user named `postgres`, whose password is `postgres`. If you have changed the password or created a new user with a different password, then enter those new values when prompted. Otherwise, enter `postgres` and `postgres` at the two prompts.

In an interactive environment, you could execute a snippet of code that prompts the user for their username and password with the following snippet (which isn't run in the book):

Your password is still in plain text in the file, `dev_environment.csv`, so you should protect that file from exposure. However, you do not need to worry about committing that file accidentally to your git repository, because the name of the file appears in the `.gitignore` file.

For security, we use values from the `environment_variables` data.frame, rather than keeping the `username` and `password` in plain text in a source file.
-->

## Clean up

Afterwards, always disconnect from the dbms:

```r
dbDisconnect(con)
```
Tell Docker to stop the `cattle` container:

```r
sp_docker_stop("sql-pet")
```