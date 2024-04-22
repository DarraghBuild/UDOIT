# Heroku Deploy

**Warning:** There is a known issue with hosting UDOIT on Heroku.  Courses with large amounts of content can cause UDOIT to scan for longer than Heroku allows, resulting in a failed scan appears to never complete.  We do not recommend using Heroku for hosting your production instance of UDOIT at this time.  The instructions below are left here for reference.

## Installation Instructions
Installing UDOIT on Heroku requires some technical knowledge. We recommend that you have an IT staff member perform the steps below.

### Step 1: Setting up Heroku
Click the Build.io button and follow the instructions below:

<a href="https://heroku.com/deploy?template=https://github.com/ucfopen/UDOIT/tree/main" title="Deploy to Heroku"><img src="https://www.herokucdn.com/deploy/button.svg" alt="Deploy to Heroku" title="Deploy to Heroku Button"></a>

1. Create an account (if you don't have one already).
2. Give the app a name.
3. Set `APP_ENV` to `prod` for the production version of UDOIT. Please note: The `dev` environment is not available on Heroku.
4. Set `APP_LMS` to the name of your LMS.
   * `canvas` if you are using the Canvas LMS.
   * `d2l` if you are using the D2l Brightspace LMS.
5. Fill out the `BASE_URL` field with `https://yourapp.herokuapp.com`. (Replace 'yourapp' with the name you gave in step 1.2.)
6. Fill out the `JWK_BASE_URL` field with the URL to your LMS. The default value works for instructure hosted instances of Canvas, but will need to be modified if your JWK configuration is hosted at a different domain than `iss`.
7. Click the Deploy button and wait for the process to complete.

The above deploy uses the Heroku Postgres Mini plan by default. Please check `https://elements.heroku.com/addons/heroku-postgresql` for Heroku Postgresql plan details. You can upgrade Postgresql plan inside Heroku UI later.

### Step 2: Database Migration and Setup
Next we need to set up the database and insert our institution in to the appropriate table.
1. Click "Manage App" or go to `https://dashboard.heroku.com/apps/yourapp`. (Replace 'yourapp' with the name you gave in Step 1.2.)
2. In the top-right corner, click "More", then click "Run Console".
3. In the `heroku run` field, type `bash` and click **Run**.
4. Run the database migrations by running the following command in the Console:
```bash
php bin/console doctrine:migrations:migrate
```
5. If `APP_ENV` is set to `prod` you will need to generate the doctrine proxy classes by running the following command in the Console:
```bash
php bin/console cache:warmup --env=prod
```
6. Set up developer keys according to the instructions in INSTALL_\<LMS\>.md.
7. Access the Postgres database by running the following command in the Console:
```sql
psql <the DATABASE_URL located in your config vars under the Settings tab>
```
6. Insert your institution in to the institution table as described in INSTALL_\<LMS\>.md.  It may be something like:
```sql
INSERT INTO institution (id, title, lms_domain, lms_id, lms_account_id, created, status, vanity_url, metadata, api_client_id, api_client_secret) VALUES (0, 'Canvas', 'myinstitution.instructure.com', 'canvas', '1', '2021-10-21', true, 'vanity.example.com', '{"lang":"en"}', '123456', 'abcdefghijklmnopqrstuvwxyz');
```
(DO NOT copy that command and use it as-is!)

### Step 3: Finish
Install the app following the instructions described in INSTALL_\<LMS\>.md.

### (Optional) Step 4: Clone UDOIT and Push to Heroku
If you make changes to the source code of UDOIT, and you'd like to deploy those changes to your Heroku instance, follow these instructions.

1. Follow the instructions under the ['Source Code' section of INSTALL.md](https://github.com/ucfopen/UDOIT/blob/main/INSTALL.md#source-code).
2. Install and log in to the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).
3. Next, from within your UDOIT's root folder on your machine, connect your local repository to Heroku:
```
heroku git:remote -a your_app_name
```
4. Finally, run the following to upload the source to Heroku:
```
git push heroku main:master -f
```

## Manual Deployment
### Step 1: Create a new Heroku app
### Step 2: Add Buildpacks
1. Go to 'Settings' --> 'Buildpacks' --> 'Add Buildpack'.
2. Add 'php' and 'nodejs'.
### Step 3: Set up Config Vars
1. Go to 'Settings' --> 'Config Vars' --> 'Reveal Config Vars'.
2. Add the following:
  * APP_ENV = prod
  * APP_LMS = \<LMS\>
  * BASE_URL = \<Heroku app URL\>
### Step 4: Connect Database
If you intend to use your own databse, set a DATABASE_URL config var to your database's url. To use Heroku Postgresql:
  * Go to 'Resources' --> 'Add-ons' --> 'Find more add-ons'.
  * Find 'Heroku Postgres' and then install.

Continue with the steps outlined in the previous section.
