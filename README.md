# Cassie is Autodesk BIM 360 Field: a SaaS platform for field management

[![Codeship](https://codeship.com/projects/74905cc0-b38a-0132-673b-521937a52203/status?branch=develop)](https://codeship.com/projects/70217/)
[![Code Climate](https://codeclimate.com/repos/53ace56a6956807d7b0014f9/badges/9007836ed323e1f1d52c/gpa.png)](https://codeclimate.com/repos/53ace56a6956807d7b0014f9/feed)

## Setup
0. Start from a brand new Mac

  install rvm 
   ```
    \curl -sSL https://get.rvm.io | bash
   ```

  install brew 
   ```
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```
   
   you must install mysql@5.7.21
   
  start my sql once, or there won't be '/tmp/mysql.sock'
   ```
   mysql.server start
   ```

1. Map `local.autodesk.com` to localhost by adding the following line to your `/etc/hosts` file (needed for Oxygen authentication):

   ```
   127.0.0.1 local.autodesk.com
   ```
1. Set up artifactory credentials by adding this line to your `.bashrc` (ask team about credentials):
   ```
   export BUNDLE_ARTIFACTORY__ADSKENGINEER__NET="<un>:<pw>"
   ```
1. Run `./bin/setup`.  
You can find solution for some errors that may happen in this step on [Installation Wiki](https://github.com/Vela/Cassie/wiki/Installation)
1. Configure MySQL's `sql_mode`:
   1. Open `/usr/local/Cellar/mysql/5.6.21/my.cnf` in an editor or create a new file `~/.my.cnf`. Note that your version of MySQL may be different.
   2. Ensure that you have the following configuration:

      ```
      [mysqld]
      character-set-server=utf8
      sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES,PIPES_AS_CONCAT
      socket          = /tmp/mysql.sock
      ```
1. Start redis with `redis-server` or run `brew info redis` for more information. **`bundle exec rake db:create db:migrate db:test:prepare` in the step of running `./bin/setup` will fail unless redis server is running.**
1. Start MySQL with `mysql.server start` or run `brew info mysql` for info on setting it up to start automatically. Your root password should be *blank*. **`bundle exec rake db:create db:migrate db:test:prepare` in the step of running `./bin/setup` will fail unless mysql server is running.**
1. Before running `foreman` to start the application, make sure passenger is compiled by starting it alone with `passenger start`.
1. Run the application with `foreman start`. This will start Passenger, Solr, and Resque. In new MacOS with M1 chip, you may try `exec rails server`, if pervious didn't work.
1. Access the app at the host we added in step 1—`local.autodesk.com:3000`.

### Running the specs

Use `rake` to run the specs. This will create a clean test database (via `rake db:test:prepare`) and then run rspec and teaspoon. For more options, see [Running Tests](//github.com/Vela/Cassie/wiki/Running-Tests).

### Jasper

If you need to setup reporting server, follow the guide in [Gemini](https://github.com/Vela/Gemini) repository.

### HQ
If you need to integrate with HQ, follow instructions on the wiki page [Running Service Product, HQ UI and HQ API locally]
(https://wiki.autodesk.com/pages/viewpage.action?title=Running+Service+Product%2C+HQ+UI+and+HQ+API+locally&spaceKey=PDCA).

### Docker

To run Cassie in a Docker container, follow the steps below:

1. On your host machine, update `/etc/hosts` to include an alias for `local.autodesk.com`:

```
127.0.0.1	localhost local.autodesk.com
```

2. Copy the `env.sample` file to `.env`:

```
cp env.sample .env
```

3. Build and run the container:

```
docker-compose build
docker-compose up
```

4. Navigate to the [Login](http://local.autodesk.com:3000/auth/developer) page, and log in with `velasystems@gmail.com`.

Additional information about Docker may be found in the [README.DOCKER](./docker/images/app/README.DOCKER.md) file.

## Bypassing Oxygen Authentication for Development mode

Visiting any page on the site without a session will trigger a redirect to Oxygen for authentication.

To bypass this (in development mode only):

1. Visit `/auth/developer`.
1. Enter the email of the user you'd like to sign in as.
1. Click 'Sign In'.

*If you still can't login, you may need to clear your cookies.*

## Deploying

To deploy, use the following commands:

  ```
  BRANCH=release/1517_cockroach bundle exec cap qa deploy
  BRANCH=release/1518_scorpion bundle exec cap staging deploy:reports
  ```

* To deploy the develop branch, omit the BRANCH env variable
* To see more detail during the deploy, add the LOG_LEVEL environment variable set to debug (eg. `LOG_LEVEL=debug bundle exec cap qa deploy`)
* To have it post updates to [slack](https://autodesk.slack.com/messages/C1P2R0ZMW/), add 2 [vars](https://autodesk.slack.com/archives/C1P322VK3/p1508256676000845)
 to your env 
  ```
  SLACKISTRANO_TOKEN=<token value>
  SLACKISTRANO_WEBHOOK=<webhook value>
  ```
 
## Time zones

Cassie relies heavily on mysql named time zones. The setup script should
have installed all the required named time zones from your system's zoneinfo
folder. However, this can incorrectly name 'UTC' as 'etc/UTC'. If you find
certain specs failing (e.g. spec/lib/digest_maker_spec) you can use the
following command to create the 'UTC' named time zone correctly:

```bash
mysql_tzinfo_to_sql /usr/share/zoneinfo/UTC UTC | mysql -u root mysql
```

## Conecting to QA server
https://github.com/Vela/Cassie/wiki/SSH-into-QA-and-open-a-rails-console

## Development Boxes Map

* dev (dev.bim360field.autodesk.com):
  * web00.ng.b3f.dev00
  * job00.ng.b3f.dev00
  * reports00.ng.b3f.dev00

* qa (qa.bim360field.autodesk.com):
  * web00.ng.b3f.qa
  * web01.ng.b3f.qa
  * job00.ng.b3f.qa
  * reports00.ng.b3f.qa
  * reports01.ng.b3f.qa

* staging (staging.bim360field.autodesk.com):
  * web00.ng.b3f.stg
  * job00.ng.b3f.stg
  * redissolr00.ng.b3f.stg
  * reports00.ng.b3f
  * reports01.ng.b3f
  * reports-ui0.ng.b3f
  * reports-ui1.ng.b3f

## About

BIM 360 Field offers a large amount of fieldfunctionality:

* Punchlists/Worklists
* QAQC Checklists
* Safety Checklists
* Commissioning Checklists
* Equipment Management (including model mappings)
* Closing and Viewing Workflow
* Document Libraries
* Transactional Reporting (emailed and printed)
* BIM integration using navis models - both equipment records and geometry
* Task management, passing work processes through multiple parties

Additionally, BIM 360 Field offers several ways to input data and export data:

* VAPI - BIM 360 Field's RESTful API
* Import/export of issues, areas, companies, checklists, users
* iPad app - BIM 360 Field Mobile (runs disconnected)

## CSE
In order to get user email and profile changing in time, field listens the CSE Channel to get the changing information
Start the task:
*rake cse:identity_pull_start
more detail: https://git.autodesk.com/cloudplatform-compute/cse-ruby-sdk

# Mobile development

* Use velasystems@gmail.com user
* set user's `oxygen_uid = 'E9QRK4VWQ7DN'`

# Compile Java code about calling Runreport to run adhoc reports
Check if you have installed ANT. 
```
ant -version
```
if not install it
```
brew install ant
```
Set JAVA_HOME in your env file, such as .bash_profile.

Then goto "Cassie/java/workspaces/VelaReports"
run ant, if you are lucky that should be all set.
```
ant
```
