## Bank SPMS

[![Build Status](https://travis-ci.org/nkorobkov/lfmsh_bank.svg?branch=master)](https://travis-ci.org/nkorobkov/lfmsh_bank)

**Bank** is a web application that hosts individual accounts of more than 130 users with different permissions.
It allows the users to issue and transfer money, living within the virtual economic system of the
[Summer Physical and Mathematical School](https://ipfran.ru/training/summer-school).

A more detailed description of the project, architecture and technologies used can be found at the
[Bank Project Description Page](https://nkorobkov.github.io/projects/bank)

## Installation and running

Before any installation, you'll need to clone the repo and install the requirements.
You may want to use a [virtual environment](https://docs.python.org/3/tutorial/venv.html) for this process. Here are example steps to do that: 
```bash
> python3 -m venv venv
> source venv/bin/activate
> pip3 install -r requirements.txt
```

Run `cp .env.template .env` to create .env file with configurations. Update secret values accordingly (especially if you are planning to deploy).

### Local startup instructions

- to deploy test server locally we can only work with django code, so `cd django-app`
- make migrations with:
```bash
  ./manage.py makemigrations
  ./manage.py makemigrations bank
```
- apply migrations: `./manage.py migrate`
- create a superuser for admin actions: `./manage.py createsuperuser`
- populate static tables: `./manage.py add_static_data`
- prepare users' info and place it in `meta_files/users_data` You may use example files for testing `cp meta_files/users_data/student_example.csv meta_files/users_data/student.csv & cp meta_files/users_data/staff_example.csv meta_files/users_data/staff.csv`.
- add users to the DB with `./manage.py add_users`
This command will print credentials for all users from student and staff groups.  
It will also create one user with extra privileges called `bank_manager`.
- collect static files from different apps to a single folder: `./manage.py collectstatic`
- run the test server:  `./manage.py runserver`
-  go to <http://127.0.0.1:8000/bank> and enter some user's credentials from the console output to test the app. Bank interface is in Russian language, so feel free to use chrome integrated translation function, if you don't read Russian. 

### Local testing

```bash
cd django-app
python ./manage.py test
```

### Building with docker

Build an image for local hosting with:
`docker build -t django-bank:0.01  -f ./docker/django-app/Dockerfile .`
and then run it with:
`docker run -d -p 1234:8000 django-bank:0.01`

You can check that the bank is working at <http://127.0.0.1:1234/bank/>.

This approach requires you to make migrations and populate tables in advance, before constructing an image.
Alternatively, if image is already constructed, you can log in into running container with  
`docker exec -it <YOUR_CONTAINER_ID> /bin/bash` and exec all the commands needed to populate database from local startup section above.
All the changes in db would be lost after container restart unless you commit them to image.

### Docker Compose
Alternatively, there is a possibility to launch lfmsh bank in production-like mode with a real server and db.

`docker-compose.yml` file contains configuration for deployment of following schema:

![](https://nkorobkov.github.io/assets/bank/deployment-pic.png)


Run `docker-compose up` from root of the project (where docker-compose.yml is located) to see the application start up with postgres db, nginx server
and django app, all in separate containers.
All postgres data would be stored in `docker/postgres/volumes` and would persist upon the container restart.

After the first startup, you  will still need to populate tables manually. Log in to uwsgi container:
docker exec -it lfmsh_bank_uwsgi_1 /bin/bash
`docker exec -it lfmsh_bank_uwsgi_1 /bin/bash`
and run migrations and static data operations from local startup section above.

In the current configuration, static data is not stored on GitHub. To download it, you'll still need to collectstatic manually.
Run `./django-app/manage.py collectstatic` from the host machine, and the nginx container will gain access to this data automatically.

After these two operations (static collection and db population), bank would be avaliable at http://localhost/bank
You'll be able to toggle bank on and off with single `docker-compose up/down` command.


You may need to manually bump permissions on socker after compose startup `chmod 666 docker.sock`

### History

This is the second version of Bank application. Version 1 was developed in 2016 and is hosted in another repo [here](https://github.com/insolia/lfmsh_bank).
