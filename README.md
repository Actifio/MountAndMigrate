# MountAndMigrate
**Actifio Mount and Migrate CLI examples and further details**

In Actifio 10c, there is a new feature called Mount and Migrate (M&M) that can be used for customers to help move an application (Database, Filesystems, etc) to an alternate physical storage location, rather than be mounted from an Actifio appliance virtually. This M&M feature helps customers greatly via multiple use cases, such as:
*1. Enhancing application recovery times for traditional restore requests, as Database(s) can be mounted (instantly) and put back into use quickly, while the movement of files can be done after the DB is back online, and a final switch over to the modified database at a point in time in the future. So very low RTO, and greatly assists customers with very large Database(s) in the multi-terabyte size.
*2. For helping with Database migrations to a new server in the same or remote location (such as a public cloud).
*3. When databases are mounted and used for a longer period than originally planned, and a team would like to move the database from an Actifio virtualised mount, to become a full physical copy, then Mount & Migrate is an awesome new innovative feature.

Examples provided are for MS SQL, however more examples can be added later for Linux Databases, and filesystems.

Mount and Migrate contains three separate functions to achieve the migration of files, from the Actifio mounted filesystem(s). These steps are:

1. Mount the filesystem or Database(s) to a target host. (This is the regular App Aware mount operation in Actifio).
2. Configure a schedule for restoring updates of the mounted filesystem & associated files to a target directory & filename(s).
3. When ready to switch from a virtual mounted copy, to the restored physical copy to make the final switch, which incurs a brief outage, but this is a completley automated switch and takes a short period of time to complete. The previously mounted volumes are also unmounted, leaving behind only a local copy of the SQL database(s)

Below is a CLI example of the above three step process. Please note you must be using the Actifio CLI and have sufficient rights to run the below commands, which is assumed to be configured already.

- [x] **Step 1 - Mount a Database backup image to SQL host DEMO-SQL-3 as new Database with the name DBMount2**

```udstask mountimage -host demo-sql-3 -image Image_22647206 -restoreoption "provisioningoptions=<provisioning-options><sqlinstance>DEMO-SQL-3</sqlinstance><dbname>DBMount2</dbname><recover>true</recover><userlogins>false</userlogins></provisioning-options>,reprotect=false"```

* -host equals the name of the host you want to mount the image to
* -image equals the image name of the point in time capture point from the source you want to access
* -restoreoption is a comma delimited list of restore options where each restore option is a name=value pair
* \<sqlinstance> is the name of the SQL instance you want to mount the database into e.g DEMO-SQL-3
* \<dbname> is the name of the SQL database you want created, e.g. DBMount2
* \<recover> is whether to bring the SQL database online, or leave it in recovery mode. e.g. true recovers the DB completely
* \<userlogins> is whether to recover the SQL user database. .e.g. false - Please note the user database needs to be backed up for this to be set to true.
* reprotect = false - refers to the ability to reprotect the virtual database with an existing Policy. If set to true then more syntax is required to bind the virtual database to a new policy and profile.

- [x] **Step 2 - Setup the migration schedule for regular restores pre-migration and get the first image ready**

```udstask migrateimage -image Image_22767594 -action config -restorelocation "usesourcelocation=false,targets=<files><file><name>smalldb.mdf</name><source>C:\SQLData</source><target>C:\DBCLI</target></file><file><name>smalldbl_log.ldf</name><source>C:\SQLData</source><target>C:\DBCLI</target></file></files>" -restoreoption "provisioningoptions=<provisioning-options><copythreadcount>10</copythreadcount></provisioning-options>" -frequency 1```

* -image equals the image name of the mounted image, not the original source image.
* -action config sets up the initial schedule of the migration, by copying the database files from the mount to the desired location on the target host. This process is then repeated on that schedule, until the finalize action is initiated.
* -restorlocation is a comma delimited list of restore options where each restore option is a name=value pair
* usesourcelocation = false - set to true, if you want to recover DB to same folder and file location as source server
* targets= xml syntax to define the target file names and folder location for the SQL database files (MDF, NDF and LDF). These can be separate drives if required.
* \<name> equals the filename
* \<source> equals the directory location where source files are located
* \<target> equals the location where you want the files to be located on the target host
* -restoreoption is a comma delimited list of restore options where each restore option is a name=value pair
* provisioningoptions = allows modifying of additional options and can be skipped if you do not wish to edit the default copythreadcount (4).
* \<copythreadcount> allows the modification of the copythreadcount (1-20) where this will increase the parallel copy threads to use per disk volume
* frequency is how often to update the flat files on the local host with changes from the mounted database. The values here are 1-24hrs. With 24hrs being the default.


- [x] **Step 3 - Migrate the Database files to the local server, and switch from the mounted copy to the restored files on the local server**

```udstask migrateimage -image Image_22767594 -action finalize```

* -image equals the image name of the mounted image, not the original source image.
* -action finalize runs the final backup and recovery task, and also switches the database from the mounted volumes, to the locally restored files, and unmounts the original disk image.

After the finalize step is executed, the new database should be re-protected if it is now migrated to a new target location.

#Supporting Videos
1. Demonstration of Mount and Migrate via AGM - 
2. Demonstration of Mount and Migrate via CLI - 


