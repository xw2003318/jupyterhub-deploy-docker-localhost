# Background
This code base to deploy Jupyter Notebooks using JupyterHub is based on the open source reference implementation from https://github.com/jupyterhub/jupyterhub-deploy-docker.

It also uses Docker (https://www.docker.com/) containers to manage the three pieces of software needed to run this set up:
1. JupyterHub - takes care of authentication and notebook spawning
2. Jupyter Notebooks - notebook environment with 2 kernels: Python and R
3. PostgreSQL - database backend to store notebook user data

# Installation Guide

## Install Docker and Docker Compose

* Install Docker: https://docs.docker.com/install/
* Install Docker Compose: https://docs.docker.com/compose/install/

## Create secrets directory
Type the following on your Linux command line:
> `$ mkdir secrets`
> `$ pwd`

You will copy the result of `pwd` later to your `letsencrypt-certs.sh` file.

## Create `.env` and `userlist` files
There are two template files for `.env` and `userlist`, `.env-template` and `userlist-template`. Rename these files by removing the `-template` suffix. These are the only two files you need to update.

## Create the `drive.jupyterlab-settings` file in `singleuser` folder
Rename `drive.jupyterlab-settings-template` to `drive.jupyterlab-settings`. The file `drive.jupyterlab-settings-template` is under the `singleuser` folder.

## Obtain Domain name
If you intend to use JupyterHub on your laptop for localhost use, there is no need to obtain a domain name and you can skip this step. If JuypterHub will be used with a domain name, obtain domain name (see "Using a Domain Name" below).

## Create SSL Certificate (two options)
### Self-signed certificate
Using a self-signed certificate is useful for testing or limited use (localhost).
> `$ chmod +x create-certs.sh`

> `$ ./create-certs.sh`

### Using a Domain Name
If you want to use your own domain name, obtain one from a domain name registrar. You will use this domain name to replace instances of "mydomain.com" in configuration files and letsencrypt bash script below.

### Obtain "Lets Encrypt" SSL Certificate
If using JupyterHub with a domain name, open the `letsencrypt-certs.sh` bash script with a text editor (e.g., nano) and edit the first few lines (lines 3, 4 and 5):

* For the line below, replace "mydomain.com" with your domain name, "e.g., example.com", that you obtained from a domain name registrar:
> `export JH_FQDN="mydomain.com"`
* For the line below, replace email address with your email address
> `export JH_EMAIL="myname@mydomain.com"`
* For the line below, replace "/path/to" with full path (result of `pwd` earlier)
> `export JH_SECRETS="/path/to/secrets"`

> `$ chmod +x letsencrypt-certs.sh`
> `$ ./letsencrypt-certs.sh`

## JupyterHub Authentication

This build of JupyterHub has three options for Authentication. Go to about Line 18 of the `.env` file and set the environment variable `JUPYTERHUB_AUTHENTICATOR` to the selected option.
* tmp_authenticator
* dummy_authenticator (default)
* github_authenticator (thru OAuth, requires obtaining GitHub credentials, see below)

Possible scenarios for GitHub authentication (if you choose 'github_authenticator'):
1. Default GitHub settings in .env - no need to change or update any .env settings
2. Set up your own GitHub account and OAuth:

   2.1 Set up GitHub account

   2.2 Set up OAuth application (see below "Obtain GitHub Credentials")

## Obtain your GitHub Account Credentials
If you will be using GitHub Oauth to authenticate users to JupyterHub, you need to sign up for a GitHub Account:
1. Go to https://www.github.com and create an account if you do not have one yet.
2. Remember your GitHub user name. You will use this for #3 below.
3. Open the file `userlist` with your text editor and add your GitHub user name below "jovyan admin" as below:
> `<github user name> admin`

## Obtain GitHub OAuth Credentials
* Log in to GitHub
* Go to Developer Settings (https://github.com/settings/developers) - create new Oauth App
* Record the following information:
  * GitHub Client ID
  * GitHub Client Secret
  * GitHub Callback URL: This should be of the form https://"mydomain.com"/hub/oauth_callback if with a domain name (remember to replace "mydomain.com" with your domain name, as obtained from the step above, "Using a Domain Name".)
* Copy-paste each of these to right `.env` section (about Line 23):

> `GITHUB_CLIENT_ID=<github client id>`

> `GITHUB_CLIENT_SECRET=<github client secret>`

> `OAUTH_CALLBACK_URL=https://mydomain.com/hub/oauth_callback`

If using localhost, replace "mydomain.com" in OAUTH_CALLBACK with "localhost" (i.e., "https://localhost/hub/oauth_callback").

## Generate Postgres password
This JupyterHub deployment uses the PostgreSQL database as a backend (instead of sqlite).
* Create the postgres password by typing the Linux command below:
> `$ openssl rand -hex 32`
* Copy the result of the command to the right `.env` section (about Line 74) by replacing the `geeks@localhost` entry or current value with the cryptic, "hex" value:
> `POSTGRES_PASSWORD=geeks@localhost`

> `JPY_PSQL_PASSWORD=geeks@localhost`

## Create Docker networks and volumes
Type the following Linux commands on the command line:
> `$ make network`

> `$ make volumes`

## Build the Notebook Server Docker Image
Type the following command on the command line:
> `$ make notebook_image`

This command creates the Jupyter Notebook Docker image that will be "spawned" by JupyterHub.

## Build the PostgreSQL 9.5 and JupyterHub Docker images
Type the following command on the command line:
> `$ ./buildhub.sh`

This script does two things:
* Obtains the internal Docker network IP of the JupyterHub container.
* Builds the final JupyterHub and Postgres DB images

## Launch JupyterHub and Browse Your Brand New Notebook Server
* To troubleshoot potential issues during first launch, use the following command:
> `$ ./starthub.sh`

This script launches both JupyterHub and Postgres DB backend in the background and launches log monitoring.

Launch JupyterHub on your browser:
* If localhost, go to https://localhost in your browser. If using a domain name, go to https://mydomain.com.
* Sign in to GitHub using your account

Due to unconfigured Google Drive Integration, you will see something like this error message pop up after launching Jupyterlab. Just click "OK". You can configure Google Drive integration later.
![Google Drive Error](./docs/GoogleDriveAPI-Error.png)

Please see next section for instructions on how to configure Google Drive to work with your JupyterLab set up.

# Jupyterlab Google Drive Integration

Please follow the instructions here:
https://github.com/jupyterlab/jupyterlab-google-drive/blob/master/docs/advanced.md

These instructions will help you obtain your Google Client ID as in the figure below.
![Google Drive Error](./docs/client-id-secret.png)

After you obtain Google API credentials, rename the file `drive.jupyterlab-settings-template` by removing the `-template` suffix, then update the file by filling upthe empty string "" in "clientId":"" with your Google Client ID (copy-paste from the developer console as above) . It's the long string with cryptic characters that end in `.apps.googleusercontent.com`.

# Notes for Windows users

Windows 10 users should do the following:
1. To run the bash scripts in a bash shell on Windows 10, install the Linux Subsystem on Windows 10 here:
https://docs.microsoft.com/en-us/windows/wsl/install-win10
2. Use "Edge Channel" Docker version for Windows 10: https://download.docker.com/win/edge/Docker%20for%20Windows%20Installer.exe
3. Check "Expose daemon on tcp://localhost:2375 without TLS as follows:
![Windows 10 without TLS](./docs/25342.LINE.jpg)


# Summary
In summary, there are a few steps to get started with Jupyter Notebooks:

1. Decide whether to run as `localhost` or with domain name. (Best to try out `localhost` first.) If running as `localhost`, run `create-certs.sh` script. If using a domain name, obtain domain name first, then modify and run `letsencrypt-certs.sh` script.
2. Configure the `.env` and `userlist` files accordingly.
3. Run `buildhub.sh` script.
4. Run `starthub.sh` script.

# Upgrading from JupyterHub 0.7* to 0.8*
Delete the old `jupyterhub_cookie_secret` file:
> `$ sudo rm /var/lib/docker/volumes/jupyterhub-data/_data/jupyterhub_cookie_secret`

# JupyterHub Logs / Launch Issues
## Logs: Old base64 cookie-secret detected in /data/jupyterhub_cookie_secret.
* While jupyterhub is running, type the following commands:
> `$ docker exec -it jupyterhub /bin/bash`
* This brings you to the jupyterhub bash prompt. Type the following command to regenerate a new cookie secret:
> `# openssl rand -hex 32 > "/data/jupyterhub_cookie_secret"`
## Browser: 403 : Forbidden
* Add your GitHub username to the `userlist` file as described above.

## JupyterHub Logs: socket.gaierror: [Errno -2] Name or service not known
* If you see this error in the logs it means the JUPYTERHUB_SERVICE_HOST_IP is misconfigured.

## Windows user
* There may be slight differences in how Chrome or Firefox behaves compared to installations on Linux or Mac (YMMV).
