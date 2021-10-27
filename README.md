# Ilmomasiina

Ilmomasiina is the event registration system originally created by Athene, forked by Tietokilta and currently under heavy development for our new site. Once finished, it will be available for all organizations to use, along with migration tools from the Athene-made version.

The latest development version is in the `dev` branch. **Please note that the code is currently in alpha phase.** It likely has major bugs, and easy migrations will not be provided between versions until we reach beta.

## Contributing

Progress and planning is tracked in GitHub issues. Please see and update the [project board](https://github.com/Tietokilta/ilmomasiina/projects/1) for ongoing work.

All help is appreciated. Please contact @PurkkaKoodari or another member of Tietokilta's Digitoimikunta if you wish to actively help with development &ndash; there are still major changes to be done that may conflict with yours.

## Documentation

See docs folder. Very incomplete, contributions welcome.

## Requirements

- Node.js 14
- npm 6
- MySQL or MariaDB (latest, older supported versions unknown)

Containerization for these is in progress.

<!--
## Using Docker container
In project root directory
```bash
docker-compose up
```
This should build and run the environment so that it is accesible at [localhost:3000](http://localhost:3000). You will need to create an `.env` file in project root (see [ENV.md](ENV.md)).

### Create fake data
Use `docker exec ilmomasiina_backend_1 npm run create-fake-data` to create some data to dockerized Ilmomasiina. The server does not like an empty database, so this is a really good idea to do when first starting the server.
-->
## MySQL Setup
<!--Only follow this if you don't use the Docker container.-->

### Mac
1. Install `mysql` (8.x) with Homebrew (https://gist.github.com/nrollr/3f57fc15ded7dddddcc4e82fe137b58e)
2. Start the mysql service with `brew services start mysql`
3. Open the mysql terminal with `mysql -u root`
4. In the mysql terminal, create a new user e.g. `CREATE USER 'juuso'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`
5. Fix permissions (this is probably too permissive, but it works): `GRANT ALL PRIVILEGES ON *.* TO 'sampo'@'localhost' WITH GRANT OPTION;`
6. Type `exit` to exit the mysql terminal, and sign in with your new user e.g. `mysql -u juuso -p password`
7. Create the `ilmomasiina` database with `CREATE DATABASE ilmomasiina;`

### Ubuntu
1. Install mysql with `sudo apt install mysql-server`
2. Service should start automatically
3. Same as with Mac, but use `sudo mysql -u root`
4. Follow Mac instructions
5. Fix permissions (this is probably too permissive, but it works): `GRANT ALL PRIVILEGES ON *.* TO 'sampo'@'localhost' WITH GRANT OPTION;`
6. Exit with `exit` and sign in with your new user e. g. `mysql -u juuso -p` (don't use `mysql -u juuso -p password`)
7. Follow Mac instructions step 6

## Getting started
If you are using the Docker container, only follow step 1 as rest are automatically executed.

1. Create an `.env` file at the root of the project. For the contents of the .env file, check [ENV.MD](./ENV.MD)
2. `npm install`
3. **(IMPORTANT)** `npx lerna bootstrap`
4. `npm start`

**Optional**: You can create mockup data for development by running `npm run create-fake-data`. During development, database can be resetted with `npm run reset-db`.

### Troubleshooting (Ubuntu)
If `npm start` gives error `Error: You must provide a 'secret' in your authentication configuration`, it probably means that the `.env` file is not loaded correctly. A quick fix for this is to run `source .env`.

## Mailgun setup

Mailgun provides 10 000 free messages per month which is suitable for small projects. With minor changes sending mail could be also done via Sendgrid. Using your own mail server gets you labelled as spam pretty fast.

Add mailgun credentials to .env configuration.

### Create first admin user

Add this line to .env configuration.

```
ADMIN_REGISTRATION_ALLOWED=true
```

Create new user with POST request.

```
curl 'http://localhost:3000/api/users' -H 'Content-Type: application/json' --data-binary '{ "email": "ville@athene.fi", "password": "password" }'
```

**Important**: Disallow admin user creation by removing the line.

By default, only logged in admin users can create new admin users (via `/admin`).

## Production

**Important**: Ilmomasiina is currently on alpha stage. Use it with your own risk.

Example of `.htaccess` config:

```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteRule ^ilmo$ ilmo/ [NC,R=301,L]
    RewriteRule ^ilmo/$ http://0.0.0.0:2011/ [P,L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ilmo/(.*)$ http://0.0.0.0:2011/$1 [P,L]
</IfModule>
```

Example of relevant lines in .env file

```
EMAIL_BASE_URL=https://athene.fi
PATH_PREFIX=/ilmo
PORT=2011
```

With some hosting providers (such as Otax) you might need to request the access to the port.
Running production version within pm2 is recommended

### Updating production

```
git pull otax/master
npm run compile
pm2 restart prod-server
```
