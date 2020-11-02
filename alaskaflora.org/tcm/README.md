# tcm: a web app for managing taxon concepts and their interrelationships

_The problem_: taxonomic names are incomplete keys for referencing the
circumscription of a group of organisms, because the same name may be
used differently by different taxonomists. The complete key is _name +
accordingTo_, known as a taxon concept, or taxonomic name
usage. Despite this serious issue for communication in biodiversity
science, few databases handle taxon concepts.

_A solution_: the `tcm` web app enables:

 1. The defining of taxonomic concepts (“Taxon Concept” tab), linking
    a taxonomic name (“Name” tab) with a publication (“Publication”
    tab).
 2. A way to record their interrelationships (“TC Relationships” tab):
    taxon concepts represent sets of organisms and can thus be related
    using set relationship terms: _is congruent with_, _includes_,
    _overlaps with_, _intersects_ and _is disjunct from_.
 3. Visualization of taxon concept relationships (“Graph” tab).
 4. Export of taxon concept definitions and relationships as RDF.

## Installation

Prerequisites:

 * GNU Awk (`gawk`)
 * A CGI-enabled web server (e.g., `apache`)
 * The MySQL/MariaDB command line client (`mysql`)
 * The Graphviz command line client (`dot`)

### 1. Script

The web app is a single Awk script (GNU Awk flavor), called
`tcm`. (You can rename it to anything else - just make sure to adjust
the `.htaccess` file (see below), and change the config variable
`APPNAME` in the script.)

 1. Make sure the first (hashbang) line of the script points to the
    installed location of `gawk`. You can test that the script is
    running OK, independent of web server access, by i) making sure
    the script is executable (`chmod u+x tcm`), and ii) typing `./tcm` -
    you should see an HTML output.
 2. Create a world-writable directory `tmp/` for the graphviz image
    outputs (created by web server user `http` or `apache`): `mkdir
    tmp && chmod a+rw tmp`

### 2. MySQL database

 1. Log into the MySQL (or MariaDB) server. Create a database with 
    `CREATE DATABASE tcm CHARSET 'utf8' COLLATE 'utf8_bin';` 
 2. Create the database structure by executing SQL commands in `tcm.sql`.
    E.g., `mysql -u <user> -p<password> -h <host> tcm < tcm.sql`
 3. Configure the database access credentials in `pw.awk`. Copy
    `pw_template.awk` to `pw.awk` and enter the hostname, username,
    password and the name of the database (`tcm` by default, but just
    adjust this file to match the actual DB name).

### 3. .htaccess

The `.htaccess` file performs local configutation of the apache web
server. Adapt for use with other web servers (`nginx`, etc).

Create a file called `.htaccess` in this directory, with contents:

    Options +ExecCGI -Indexes
    <Files "tcm">
      SetHandler cgi-script
    </Files>
    DirectoryIndex tcm
    <FilesMatch "^\.pw$">
      Deny from all
    </FilesMatch>
    <FilesMatch "^pw\.awk$">
      Deny from all
    </FilesMatch>

If you wish to password-protect web access to this directory, add
these lines:

    AuthType Basic
    AuthName "Password Protected"
    AuthUserFile /full/path/to/this/directory/.pw
    Require valid-user

Then create the file `.pw` with `htpasswd -c .pw <username>` with your
chosen password.  Different users can be created with different
passwords.

For the cookie-based filtering by genus:

    Session On
    SessionEnv On
    SessionCookieName session path=/
    SessionHeader X-Replace-Session

And make sure that the `session_module` and `session_cookie_module`
are loaded (in `httpd.conf`).