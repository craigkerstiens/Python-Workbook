Heroku for Python Workbook
==========================

This workbook walks you through the steps of building Python apps that can run on Heroku.

Tip: You can find the latest version of this workbook, including all the source code, online at:  
[https://github.com/craigkerstiens/python-workbook](https://github.com/craigkerstiens/python-workbook)


Prerequisites
-------------

Before you get started, you need to set up your development environment, as specified in the Prerequisites section of each tutorial.


Terminology
-----------

We will start with some terminology that is helpful to understand.

PaaS - "Platform as a Service" is a general term for managed environments that run apps.

Heroku - A PaaS provider.

heroku - (Lower case "h") The command-line client for managing apps on Heroku.

git - A popular distributed version control system that is used to deploy apps to Heroku.

Heroku Add-on - The primary way Heroku can be extended. Add-ons are exposed as services that can be used from any app on Heroku.

Tutorial #1: Building a Web App
-------------------------------

In this tutorial, you will create a web app and deploy it to Heroku. You will use a Flask create the app. You'll first run the app locally, and then deploy it to Heroku using git.

#### Prerequisites

* Create an account on [heroku.com](https://api.heroku.com/signup)
* Install the `heroku` command-line client (Appendix A)
* Installing git and Setting up an SSH Key (Appendix B)
* Ensure your Python environment is setup
  * [Getting Setup with Python](http://www.craigkerstiens.com/2011/10/27/gettingsetupwithpython/)
  * [Installing Python Packages](http://www.craigkerstiens.com/2011/11/01/installingpythonpackages/)


### Step 1: Create a Web App

1. Create and load your virtualenv: 

    virtualenv --no-site-packages venv 
    source venv/bin/activate


2. Create your application in app.py:

    import os
	from flask import Flask
	app = Flask(__name__)

	@app.route("/")
	def hello():
	    return "Hello from Python!"

	if __name__ == "__main__":
	    port = int(os.environ.get("PORT", 5000))
	    app.run(host='0.0.0.0', port=port)

### Step 2: Test the App Locally
	
1. Run your application locally:

   python app.py
	
2. You should be able to navigate in your browser to ['http://localhost:5000'](http://localhost:5000/) to view your hello world application. You'll notice for Flask the one unique portion is to attempt to read the port variable if it exists, this is to enable Heroku to know which port to listen to. 

3. Press `CTRL-C` to stop the process.

You are now ready to deploy this simple Python/Flask web app to Heroku.

### Step 4: Deploy the Web App to Heroku

1. In the project directory, create a new file named Procfile containing:

        web: python app.py

    Note: The file names, directories, and code are case sensitive. The Procfile file name must begin with an uppercase "P" character.

    Caution: Some text editors on Windows, such as Notepad, automatically append a .txt file extension to saved files. If that happens, you must remove the file extension.

    `Procfile` is a mechanism for declaring what commands are started when your dynos are run on the Heroku platform.  In this case, we want Heroku to run the webapp startup script for web dynos.

2. Initialize a local git repository, add the files to it, and commit them:

        git init
        git add .
        git commit -m "initial commit for helloheroku"

    Note: On Windows, you can ignore the following message when running the “git add .” command:

        warning : LF will be replaced by CRLF in .gitignore

    The commit operation has output similar to the following:

        [master (root-commit) b914eee] initial commit
        7 files changed, 165 insertions(+), 0 deletions(-)
        create mode 100644 .gitignore
        create mode 100644 Procfile
        create mode 100644 app.py

3. Create a new app provisioning stack on Heroku by using the `heroku` command-line client:

        heroku create --stack cedar

    Note: You must use the "cedar" stack when creating this new app because it’s the only Heroku stack that supports Python.

    The output looks similar to the following:

        Creating empty-winter-343... done, stack is cedar
        http://empty-winter-343.herokuapp.com/ | git@heroku.com:empty-winter-343.git
        Git remote heroku added

    Note: `empty-winter-343` is a randomly generated temporary name for the app. You can rename the app with any unique and valid name using the `heroku apps:rename` command.

    The create command outputs the web URL and git URL for this app. Since you had already created a git repository for this app, the heroku client automatically added the heroku remote repository information to the git configuration.

4. Deploy the app to Heroku:

        git push heroku master

    This command instructs `git` to push the app to the master branch on the heroku remote repository. This automatically triggers a Maven build on Heroku. When the build finishes, the output ends with something like the following:

        ----->Discovering process types
        Procfile declares types -> web
        -----> Compiled slug size is 17.0MB
        -----> Launching... done, v6
        http://empty-winter-343.herokuapp.com deployed to Heroku
        To git@heroku.com:empty-winter-343.git
        + 3bcf805...a72152c master -> master (forced update)

5. Verify that the output contains the message:

        Procfile declares types -> web

    If it doesn't, confirm that the `Procfile` is named correctly with no file extension and that it contains:

        web: sh target/bin/webapp

    If you fix `Procfile`, deploy the changes to Heroku:

        git add Procfile
        git commit -m "fixed Procfile"
        git push heroku master
        heroku scale web=1

6. Open the app in your browser using the generated app URL or by running:

        heroku open

    You should see `hello, world` on the web page.


### Step 5: Scale the App on Heroku

By default, the app runs on one dyno. To add more dynos, use the `heroku scale` command.

1. Scale the app to two dynos:

        heroku scale web=2

2. See a list of your processes:

        heroku ps

    Tip: This command is very useful as a troubleshooting tool. For example, if your web app is not accessible, use `heroku ps` to ensure that a web process is running. If it’s not running, use `heroku scale web=1` to start the web app and use the heroku logs command to determine why there was a problem.

3. Scale back to one web dyno:

        heroku scale web=1

### Step 6: View App Logs on Heroku

You can see everything that your app outputs to the console (STDOUT and STDERR) by running the heroku logs command.

1. To see the logs, run:

        heroku logs

2. To see log messages as they happen, use the "tail" mode:

        heroku logs -t

3. Press `CTRL-C` to stop seeing a tail of the logs.

### Step 6: Roll Back a Release on Heroku

Whenever you deploy code, change a config variable, or add or remove an add-on resource, Heroku creates a new release and restarts your app. You will learn more about add-ons in Tutorial #4: Using a Heroku Add-on.

You can list the history of releases, and use rollbacks to revert to prior releases to back out of bad deployments or config changes.  This enables you to quickly revert to a known working state instead of creating a quick fix that might have other unforeseen effects.

1. To use the releases feature, install the `releases:basic` add-on.

        heroku addons:add releases:basic

    Note: If the output indicates that your app already has the add-on, you can ignore the message.

2. To try it out, change an environment variable for your app on Heroku:

        heroku config:add MYVAR=42

3. Now review your list of releases on Heroku:

        heroku releases

    You'll see a list of recent releases, including version number and the date of the release.

4. Roll back to the release before the MYVAR environment variable was set:

        heroku rollback

5. Verify that the MYVAR environment variable is no longer set:

        heroku config

#### Summary

In this tutorial, you created a web app and deployed it to Heroku. You learned how to push apps to Heroku using `git` and how the `Procfile` declares what commands are started when dynos are run. You also learned how to list and scale the number of dynos, view logs, and roll back releases.


Tutorial #2: Connecting to a Database
-------------------------------------

This tutorial extends the first tutorial by configuring your app to connect to a database. You will first test with a local PostgreSQL database and then use a shared PostgreSQL database on Heroku.

You could use JPA / Hibernate or other ORM frameworks to connect your Java app to the database, but in this tutorial we keep things very simple and use plain JDBC.

#### Prerequisites

* Complete Tutorial #1: Building a Web App
* Install and configure PostgreSQL (Appendix D)

### Step 1: Configure the Web App to Use PostgreSQL

1. Navigate to the base directory of the app you created in Tutorial #1.

2. Update the Maven dependencies to include the PostgreSQL JDBC driver by adding the following dependency inside the `<dependencies>` tag in the `pom.xml` file:

        <dependency>
            <groupId>postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>9.0-801.jdbc4</version>
        </dependency>

### Step 2: Create a Java DAO Class

Create a Data Access Object to read from and write to the database.

In the `src/main/java/foo` directory, create a new file named `TickDAO.java` with the following contents:

    package foo;
    
    import java.sql.*;
    
    public class TickDAO {
    
        private static String dbUrl;
    
        static {
            dbUrl = System.getenv("DATABASE_URL");
            dbUrl = dbUrl.replaceAll("postgres://(.*):(.*)@(.*)", "jdbc:postgresql://$3?user=$1&password=$2");
        }
     
        public int getTickCount() throws SQLException {
            return getTickcountFromDb();
        }
    
        public static int getScalarValue(String sql) throws SQLException {
            Connection dbConn = null;
            try {
                dbConn = DriverManager.getConnection(dbUrl);
                Statement stmt = dbConn.createStatement();
                ResultSet rs = stmt.executeQuery(sql);
                rs.next();
                System.out.println("read from database");
                return rs.getInt(1);
            } finally {
                if (dbConn != null) dbConn.close();
            }
        }
    
        private static void dbUpdate(String sql) throws SQLException {
            Connection dbConn = null;
            try {
                dbConn = DriverManager.getConnection(dbUrl);
                Statement stmt = dbConn.createStatement();
                stmt.executeUpdate(sql);
            } finally {
                if (dbConn != null) dbConn.close();
            }
        }
    
        private int getTickcountFromDb() throws SQLException {
            return getScalarValue("SELECT count(*) FROM ticks");
        }
    
        public static void createTable() throws SQLException {
            System.out.println("Creating ticks table.");
            dbUpdate("DROP TABLE IF EXISTS ticks");
            dbUpdate("CREATE TABLE ticks (tick timestamp)");
        }
    
        public void insertTick() throws SQLException {
            dbUpdate("INSERT INTO ticks VALUES (now())");
        }
    }

Heroku uses environment variables for app configuration. The `DATABASE_URL` environment variable is automatically set for each dyno on Heroku. `TickDAO` expects a `DATABASE_URL` value in the following format:

    postgres://[username]:[password]@[server]/[database-name]

`TickDAO` reads the `DATABASE_URL` environment variable and transforms it into the correct format for JDBC. When you run the app locally, you can set `DATABASE_URL` to point to your local database. You will do this in a later step.

### Step 3: Create a JSP

Create a JSP file named `ticks.jsp` in the `src/main/webapp` directory. The file contents are:

    <%@ page import="foo.*"%>
    <html>
    <%
    TickDAO tickDAO = new TickDAO();
    tickDAO.insertTick();
    %>
    <%=tickDAO.getTickCount()%> Ticks
    </html>

The `ticks.jsp` file uses the TickDAO to insert a new "tick" (row) into the database and then displays the number of ticks that have been inserted into the database.

Note: The `insertTick()` and `getTickCount()` calls are not wrapped in a database transaction so in this simple example it is possible that a user will not get the correct value if another user's "tick" is inserted between those calls.  In a real-world app, database transactions should be used for these types of situations.

### Step 4: Create a Java Schema Creation Class

1. In the `src/main/java/foo` directory, create a Java class named `SchemaCreator.java` that creates the database schema. The file contents are:

        package foo;
        import java.sql.SQLException;
        public class SchemaCreator {
            public static void main(String[] args) throws SQLException {
                TickDAO tickDAO = new TickDAO();
                TickDAO.createTable();
            }
        }

2. Navigate to the base directory of your app.

3. Update the `pom.xml` file to create a startup script for the `SchemaCreator` class by adding the following `<program>` block in the appassembler-maven-plugin's `<programs>` section:

        <program>
            <mainClass>foo.SchemaCreator</mainClass>
            <name>schemaCreator</name>
        </program>

### Step 5: Test Database Access Locally

We will test on a local PostgreSQL database before deploying to Heroku.

1. Set the `DATABASE_URL` environment variable to point to your local PostgreSQL database so you can test locally before running on Heroku. We assume that you've set up PostgreSQL as defined in the Appendix D, with a database called `helloheroku` and a user `foo` with password `foo`.

    On Mac or Linux:

        export DATABASE_URL=postgres://foo:foo@localhost/helloheroku

    On Windows:

        set DATABASE_URL=postgres://foo:foo@localhost/helloheroku

2. Compile and install the app into the local Maven repository so that the webapp script can find it:

        mvn package

3. Create the database schema locally by running:

    On Mac or Linux:

        sh target/bin/schemaCreator

    On Windows:

        target\bin\schemaCreator.bat

4. Start the web app by running:

    On Mac or Linux:

        sh target/bin/webapp

    On Windows:

        target\bin\webapp.bat

    You should see output similar to the following:

        2011-07-20 18:14:07.342:INFO::jetty-7.4.4.v20110707
        2011-07-20 18:14:07.509:INFO::started o.e.j.w.WebAppContext{/,file:/home/jamesw/projects/java-workbook/tutorial-2/src/main/webapp/}
        2011-07-20 18:14:07.567:INFO::Started SelectChannelConnector@0.0.0.0:8080 STARTING

5. Verify that `ticks.jsp` works locally by navigating to [http://localhost:8080/ticks.jsp](http://localhost:8080/ticks.jsp) in your browser.

    You should see 1 Ticks on the web page. The tick count should increment by one every time you reload the page.

6. Press `CTRL-C` to stop the web app.

### Step 6: Deploy the Web App to Heroku

1. Add the changes to git and commit:

        git add .
        git commit -m "added new DAO and JSP. Updated pom."

2. Deploy the new version of the app to Heroku:

        git push heroku master

3. We need to set up the database schema on the Heroku shared database. To do that, we will run the `schemaCreator` command just like we did locally, but this time on Heroku using the `heroku run` command:

        heroku run "sh target/bin/schemaCreator"

4. Navigate to `ticks.jsp` in your browser using the app's Heroku URL. The URL is similar to the following, but contains your unique app name instead of `empty-winter-343`:  
    [http://empty-winter-343.herokuapp.com/ticks.jsp](http://empty-winter-343.herokuapp.com/ticks.jsp)

    You should see a web page that displays the number of ticks in the database. The tick count should increment by one every time you reload the page.

#### Summary

In this tutorial, you extended the app created in Tutorial #1 to connect to a database. The standard approach for configuring apps in Heroku is to use environment variables. You used the `DATABASE_URL` environment variable to set up the database connection.  You also learned how to use `heroku run` to execute one-off processes.




Next Steps
----------

If you've completed this workbook, you now have a few Python apps deployed and running on Heroku. You also know how to scale your apps and extend them with Heroku add-ons.

To continue exploring:

* Visit [http://devcenter.heroku.com/](http://devcenter.heroku.com/) to learn more about what you can do on the Heroku platform.
* Visit [http://addons.heroku.com/](http://addons.heroku.com/) to learn about the add-ons that enable you to easily extend your app by using other cloud services.


Appendix A: Installing the heroku Command-Line Client
-----------------------------------------------------

The heroku command-line client wraps the Heroku RESTful APIs and enables you to manage your apps on Heroku.

The client is written in Ruby. To install it, you first need to install Ruby, the Ruby package manager (RubyGems), and finally the client itself.

1. To install Ruby:

    * On Mac—Mac Snow Leopard and Lion ship with Ruby.

    * On Ubuntu Linux, run:

        sudo apt-get install ruby

    * On Windows—Use the RubyInstaller from:  
    [http://rubyinstaller.org/](http://rubyinstaller.org/)

2. To install RubyGems:

    a. Download the latest zip file from:  
    [http://rubygems.org/](http://rubygems.org/)

    b. Uncompress the downloaded file and change directory to the expanded folder.

    c. Run the setup script in a terminal or command prompt:

    * On Mac or Windows:

        ruby setup.rb

    * On Linux:

        sudo ruby setup.rb
        sudo ln -s /usr/bin/gem1.8 /usr/bin/gem

3. To install the `heroku` client, run the following in a terminal or command prompt:

    * On Mac or Windows:

        gem install heroku

    * On Linux:

        sudo gem install heroku

4. Verify your installation by running the following command:

        heroku version

5. Confirm that you see output similar to the following:

        heroku-gem/2.3.6


Appendix B: Installing git and Setting up an SSH Key
----------------------------------------------------

The `git` tool is used to upload your app to Heroku.  It is a native app so follow the installation instructions for your operating system.

To upload your app to Heroku, you will need to create an SSH key (if you don't already have one) and associate it with your Heroku account.

1. Download the distribution of git for your operating system at:  
    [http://git-scm.com/download](http://git-scm.com/download)

2. Install `git` and create an SSH key by following the instructions in the link for your operating system. You only have to install `git` and create an SSH key. You don’t need a GitHub account to complete this workbook.

    * On Mac: [http://help.github.com/mac-set-up-git/](http://help.github.com/mac-set-up-git/)
    * On Windows: [http://help.github.com/win-set-up-git/](http://help.github.com/win-set-up-git/)  
    Note: The instructions in this workbook assume that the directory containing git.cmd or git.exe is included in your PATH environment variable.
    * On Linux: [http://help.github.com/linux-set-up-git/](http://help.github.com/linux-set-up-git/)

3. If you haven't already created an account on [http://heroku.com](http://heroku.com), create one now.
4. Log in to Heroku using a terminal or command line:

    heroku auth:login

5. Associate your SSH key with your Heroku account:

    heroku keys:add


Appendix C: Installing PostgreSQL
---------------------------------

PostgreSQL is an open source object-relational database system.

1. Download and install the PostgreSQL database by following instructions for your operating system:  
    [http://www.postgresql.org/download/](http://www.postgresql.org/download/)

    Note: During installation, write down the installation directory that you choose as you’ll have to navigate to the bin sub-directory later to create a database user.

2. Start the PostgreSQL server (if the installer didn't do so already).

3. Create a new user with superuser privileges, username `foo`, and password `foo`:

    * On Linux:

        sudo -u postgres createuser -P foo

    * On Mac:

        createuser -P foo

    * On Windows:

        a. Using the terminal or command line, navigate to the bin sub-directory of your installation directory.

        b. Run the following command:

            createuser -U postgres -P foo

        c. Enter `foo` when you are prompted to enter the password for the new role and to confirm the password.

        d. Enter `y` to make the new role a superuser.

        e. At the final password prompt, enter the password for the postgres user.

4. Create a new database named helloheroku:

    a. Run the following command:

        createdb -U foo -W -h localhost helloheroku

    b. Enter `foo` when prompted for a password.

5. Test the connection to the database:

        psql -U foo -W -h localhost helloheroku

6. Verify that you see output similar to the following:

        Password for user foo:
        psql (9.0.4)
        helloheroku=#

7. Type \q to exit the psql command line.


Appendix D: Installing Redis
----------------------------

Redis is an open source, advanced key-value store.

1. Install the Redis Server.

    * On Ubuntu Linux:

        sudo apt-get install redis-server

    The server starts by default.

    * On Mac: Follow the Redis installation instructions at:  
    [http://redis.io/download](http://redis.io/download)

    * On Windows: Download and install the Redis Server for Windows from:  
    [https://github.com/dmajkic/redis/downloads](https://github.com/dmajkic/redis/downloads)

    After uncompressing the files, start the server by executing:

        redis-server.exe

2. Verify it works by running redis-cli. You should see a command line but the prompt differs depending on your operating system:

        redis 127.0.0.1:6379>


