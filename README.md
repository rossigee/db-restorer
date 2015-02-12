Staging Database Restorer
=========================

A simple script I use to help me download and deploy database backups to my various working environments.

It is designed to be run as part of automated background scripts, such as those that would refresh the test data in a staging database on a regular basis for the users of a QA/testing environment.


How it works
------------

It can be configured by setting various environment variables. For example:

```
$ export DB_SNAPSHOT_URL=s3://backups-bucket/some/dir/backups.sql.gz.gpg
$ export AWSCLI="aws --profile oneofmany"
$ db-restorer
```

Also, it can be given a configuration file to source/override specific configuration settings. For example...

$ db-restorer -c /etc/stagingdb.conf

If not set in the configuration file, they can be set in the environment, either manually or via configuration of a CI pipeline.

An example configuration file:

```
DB_SNAPSHOT_URL=s3://backups-bucket/some/dir/backups.sql.gz.gpg
GPG_SNAPSHOT_PASSPHRASE="yoursecretpassphrase"
DB_NAME=yourdbname

NOTIFY_STARTED="/usr/local/bin/staging-db-notify -m \"Database is now being restored from a recent snapshot...\""
NOTIFY_FAILED="/usr/local/bin/staging-db-notify -m \"Database restore failed. Please check logs.\""
NOTIFY_FINISHED="/usr/local/bin/staging-db-notify -m \"Database restore complete.\""

```

The main variables are:

DB_SNAPSHOT_URL - Where to download the latest snapshot from.

It uses the 'aws' command line tool to obtain SQL dumps from S3 storage, if the snapshot URL begins with a 's3://'. The authentication credentials for the AWS CLI can be configured in the ~/.aws/config file for the running user, as explained further in the AWS CLI docs.

If the snapshot URL begins with a 'http://' or 'https://', then the cURL command line tool is used to retrieve the snapshot.

If the retrieved/decompressed file has a '.gpg' extension, it uses GnuPG to decrypt it, using the following environment/configuration variables:

GPG_SNAPSHOT_PASSPHRASE - The symetric key/passphrase used to encrypt the database snapshot.

If the resulting file has a '.sql' or '.sql.gz' extension, the contents ar using the mysql client. The hostname and authentication credentials for the connection are configured in the '~/.my.cnf' file for the running user. The following environment/configuration variables are used:

DB_NAME - database name to restore the database to.


Success/Error handling and notification
---------------------------------------

As the actual mysql restore starts to run, an optional hook script can be run. We use this to send an announcement to our developers chat channel to let them know the database is currently being restored. If the restore fails or succeeds, other optional hook scripts can also be run. If these variables are not set, no additional action is taken.

NOTIFY_STARTED - Hook script to run when starting the mysql restore.
NOTIFY_FAILED - Hook script to run if mysql restore fails.
NOTIFY_FINISHED - Hook script to run if mysql restore succeeds.


Testing
-------

To avoid repetitively downloading the same test dumps while testing, there is a '-d' flag. This bypasses the download and proceeds using the file provided in it's place. For example:

```
$ db-restorer -c /etc/stagingdb.conf -d /var/tmp/alreadygotit.sql.gz.gpg
```


