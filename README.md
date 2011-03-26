#Database Refactoring Workshop

Course materials for the Liquibase Workshop.

##Requirements

* MySQL 5.x (server and client)
* JDK 1.5+
* [Gradle 0.9+](http://gradle.org/downloads.html)
* OR
	* VMware Player on Windows
	* VMware Fusion on Mac
	* Provided VM image

**PLEASE NOTE**: if you don't have MySQL installed or are not comfortable administering it, you will probably benefit from installing the provided VMware player and using the VM image. If you do have MySQL installed and can create users and databases, it will probably be easier for you to use your host OS and skip VMware.

##Setup

1. Copy Workshop Files
	* Create a working directory in the location of your choosing
	* Copy files from thumb drive
	
1. Verify installation of MySQL
	* From command line, run "mysql" command
	* Must be able to log in as admin (to create users and databases)
	* You may optionally use the GUI management tool of your choice
	
1. Create Test databases
	1. mysql -u root -p
		* Enter root password when prompted
	1. CREATE DATABASE liquibase\_workshop;
	1. GRANT ALL PRIVILEGES ON liquibase_workshop.* TO liquibaseuser@localhost IDENTIFIED BY 'password';
	1. exit
	1. \. create\_schema.sql
	
1. Install Gradle
    * Use the gradlew script to install via the "Gradle wrapper"
        - or
	* Unzip Gradle ZIP file to a directory of your choosing
	* Add a GRADLE\_HOME environment variable, pointing to this directory
	* Add $GRADLE\_HOME/bin to your PATH
	* Test by running "gradle --version" from the command line
	
1. Create a database.properties file in your project directory
	> url: jdbc:mysql://localhost/liquibase\_workshop
	
	> username: liquibaseuser
	
	> password: password
	
	> change.log.file: changelog.xml

1. Be sure to keep a browser window open to the excellent [Liquibase docs](http://www.liquibase.org/manual/home)



##Database Refactorings 

1. Rename Table
	* Rename the inv table to invoice
	* Rename the lineitem table to line\_item
	* Rename the lidetail table to line\_item\_detail
1. Rename columns in the invoice table
	* Rename invid to id
	* Rename invnumber to invoice\_number
	* Rename datetimecreated to date\_created
1. Combine two columns using data transformation
	* invoice.udtime and invoice.uddate should be combined into invoice.date\_updated
	* First, create a new column called last\_updated with type of DATETIME
	* In the same changeset, populate that new column with an UPDATE query that merges the udtime and uddate values
		* HINT: timestamp(udtime, uddate) 
	* Discuss whether you should drop the two source columns in this refactoring.
		* What does it imply if you drop them?
		* What would you have to do if you kept them around?
1. Create tables
	* contact\_ball\_of\_mud is too ambitious of a table (or insufficiently coherent). Let's begin splitting it up.
		* The **contact** table should contain name fields, gender, email address, street address, birthday, occupation, and national ID
		* The **security\_info** table should contain password and mother's maiden name
		* The **credit\_card** table should contain credit card type, number, expiration and CVV
		* Choice of data type for each column is left as an exercise for the student.
	* Don't run this refactoring yet!
1. Tagging and rolling back
	* Tag the database, then run the table rename refactoring written in the previous step
		* gradle tag -Dtag=&lt;tagname&gt;
	* Now roll back to continue development on the refactoring
		* gradle rollback -Dtag=&lt;tagname&gt;
1. Finish refactoring of contact\_ball\_of\_mud
	* Write data transformation code to populate the three tables from their source
	* Remember that security\_info and credit\_card should have foreign keys to contact. Be sure to add these constraints with the appropriate refactorings
1. Add a column
	* Add a full\_name column to contact
	* Write data transformation SQL to populate it with the three existing name fields combined
		* HINT: CONCAT_WS()
	* Don't drop of the source name columns.
1. Create a trigger
	* Create a directory called src/triggers under your project
	* Create a file called contact\_insert.sql in src/triggers
		* Write trigger logic to keep full\_name up to date with the fields for first name, middle initial, and last name every time a new record is inserted
	* Create a file called contact\_update.sql in src/triggers
		* Same logic as the insert trigger
	* Write changeSets that use the sqlFile refactoring to install these triggers.
		* Remember the runOnChange attribute.
		* HINT: be sure the changeSet is idempotent!
1. Introduce lookup table
	* The invoice table has a field for payment terms, which should be normalized, not stored as strings values in the table
	* Create a table called payment\_terms with an auto\-incrementing id and a varchar(50) field called terms
	* Write a series of insertData refactorings to populate this table with all of the possible values of payment terms you found by inspecting the invoice table
	* Add a column to the invoice table called payment\_terms\_id
	* Add data transformation code to set payment\_terms\_id to refer to the appropriate rows of the payment\_terms table
	* Test your work by manually executing a join on invoice and payment\_terms
	* Add a foreign key constraint to payment\_terms\_id
		* The details of this constraint are left as an exercise for the student
	* Note that this step may best be implemented as more than one changeSet. Use your judgment.
1. Introduce surrogate key
	* Add an auto\-incrementing column to line\_item called id.
	* Add a primary key constraint on id.
1. Remap foreign keys
	* Add an integer column to line\_item\_detail called line\_item\_id
	* Write an UPDATE query to set its values to line\_item.id



