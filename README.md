# Django and React

- Django
- ReactJs
- PostgreSQL, for DB
- Redis, for Celery
- Sendgrid, for e-mail sending

This is a good starting point for modern Python/JavaScript web projects.

## Project bootstrap
- [ ] Make sure you have Python 3.8 installed
- [ ] Install Django with `pip install django`, to have the `django-admin` command available.
- [ ] Open the command line and go to the directory you want to start your project in.
- [ ] Start your project using:
    ```
    django-admin startproject theprojectname --extension py,yml,json --name Procfile,Dockerfile,README.md,.env.example,.gitignore,Makefile --template=https://github.com/vintasoftware/django-react-boilerplate/archive/boilerplate-release.zip
    ```
    Alternatively, you may start the project in the current directory by placing a `.` right after the project name, using the following command:
    ```
    django-admin startproject theprojectname . --extension py,yml,json --name Procfile,Dockerfile,README.md,.env.example,.gitignore,Makefile --template=https://github.com/vintasoftware/django-react-boilerplate/archive/boilerplate-release.zip
    ```
In the next steps, always remember to replace theprojectname with your project's name
- [ ] Above: don't forget the `--extension` and `--name` params!
- [ ] Change the first line of README to the name of the project
- [ ] Add an email address to the `ADMINS` settings variable in `{{project_name}}/backend/{{project_name}}/settings/base.py`
- [ ] Change the `SERVER_EMAIL` to the email address used to send e-mails in `{{project_name}}/backend/{{project_name}}/settings/production.py`
- [ ] Rename the folder `github` to `.github` with the command `mv github .github`

After completing ALL of the above, remove this `Project bootstrap` section from the project README. Then follow `Running` below.

## Running
### Tools
- Setup [editorconfig](http://editorconfig.org/), [prospector](https://prospector.landscape.io/en/master/) and [ESLint](http://eslint.org/) in the text editor you will use to develop.

### Setup
- Inside the `backend` folder, do the following:
  - Create a copy of `{{project_name}}/settings/local.py.example`:  
  `cp {{project_name}}/settings/local.py.example {{project_name}}/settings/local.py`
  - Create a copy of `.env.example`:
  `cp .env.example .env`

### If you are using Docker:
- Open the `/backend/.env` file on a text editor and uncomment the line `DATABASE_URL=postgres://{{project_name}}:password@db:5432/{{project_name}}`
- Open a new command line window and go to the project's directory
- Run the initial setup:
  `make docker_setup`
- Create the migrations for `users` app:  
  `make docker_makemigrations`
- Run the migrations:
  `make docker_migrate`
- Run the project:
  `make docker_up`
- Access `http://localhost:8000` on your browser and the project should be running there
  - When you run `make docker_up`, some containers are spinned up (frontend, backend, database, etc) and each one will be running on a different port
  - The container with the React app uses port 3000. However, if you try accessing it on your browser, the app won't appear there and you'll probably see a blank page with the "Cannot GET /" error
  - This happens because the container responsible for displaying the whole application is the Django app one (running on port 8000). The frontend container is responsible for providing a bundle with its assets for [django-webpack-loader](https://github.com/django-webpack/django-webpack-loader) to consume and render them on a Django template
- To access the logs for each service, run:
  `make docker_logs <service name>` (either `backend`, `frontend`, etc)
- To stop the project, run:
  `make docker_down`

#### Adding new dependencies
- Open a new command line window and go to the project's directory
- Update the dependencies management files by performing any number of the following steps:
  - To add a new **frontend** dependency, run `npm install <package name> --save`
    > The above command will update your `package.json`, but won't make the change effective inside the container yet
  - To add a new **backend** dependency, update `requirements.in` or `dev-requirements.in` with the newest requirements
- After updating the desired file(s), run `make docker_update_dependencies` to update the containers with the new dependencies
  > The above command will stop and re-build the containers in order to make the new dependencies effective

### If you are not using Docker:
#### Setup and run the frontend app
- Open a new command line window and go to the project's directory
- `npm install`
- `npm run start`
  - This is used to serve the frontend assets to be consumed by [django-webpack-loader](https://github.com/django-webpack/django-webpack-loader) and not to run the React application as usual, so don't worry if you try to check what's running on port 3000 and see an error on your browser

#### Setup the backend app
- Open the `/backend/.env` file on a text editor and do one of the following:
  - If you wish to use SQLite locally, uncomment the line `DATABASE_URL=sqlite:///backend/db.sqlite3`
  - If you wish to use PostgreSQL locally, uncomment and edit the line `DATABASE_URL=postgres://{{project_name}}:password@db:5432/{{project_name}}` in order to make it correctly point to your database URL
    - The url format is the following: `postgres://USER:PASSWORD@HOST:PORT/NAME`
  - If you wish to use another database engine locally, add a new `DATABASE_URL` setting for the database you wish to use
    - Please refer to [dj-database-url](https://github.com/jacobian/dj-database-url#url-schema) on how to configure `DATABASE_URL` for commonly used engines
- Open a new command line window and go to the project's directory
- Create a new virtualenv with either [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) or only virtualenv: `mkvirtualenv {{project_name}}` or `python -m venv {{project_name}}-venv`
  > If you're using Python's virtualenv (the latter option), make sure to create the environment with the suggested name, otherwise it will be added to version control.
- Make sure the virtualenv is activated `workon {{project_name}}` or `source {{project_name}}-venv/bin/activate`
- Run `make compile_install_requirements` to install the requirements
  > Please make sure you have already setup PostgreSQL on your environment before installing the requirements

  > In case you wish to use a Conda virtual environment, please remove the line `export PIP_REQUIRE_VIRTUALENV=true; \` from `Makefile`

#### Run the backend app
- With the virtualenv enabled, go to the `backend` directory
- Create the migrations for `users` app: 
  `python manage.py makemigrations`
- Run the migrations:
  `python manage.py migrate`
- Run the project:
  `python manage.py runserver`
- Open a browser and go to `http://localhost:8000` to see the project running

#### Setup Celery
- Open a command line window and go to the project's directory
- `workon {{project_name}}` or `source {{project_name}}-venv/bin/activate` depending on if you are using virtualenvwrapper or just virtualenv.
- `python manage.py celery`

#### Mailhog
- For development, we use Mailhog to test our e-mail workflows, since it allows us to inspect the messages to validate they're correctly built
  - Docker users already have it setup and running once they start the project
  - For non-Docker users, please have a look [here](https://github.com/mailhog/MailHog#installation) for instructions on how to setup Mailhog on specific environments
> The project expects Mailhog SMTP server to be running on port 1025, you may alter that by changing `EMAIL_PORT` on settings


### Testing
`make test`

Will run django tests using `--keepdb` and `--parallel`. You may pass a path to the desired test module in the make command. E.g.:

`make test someapp.tests.test_views`

### Adding new pypi libs
Add the libname to either `requirements.in` or `dev-requirements.in`, then either upgrade the libs with `make upgrade` or manually compile it and then,  install.
`pip-compile requirements.in > requirements.txt` or `make upgrade`
`pip install -r requirements.txt`


