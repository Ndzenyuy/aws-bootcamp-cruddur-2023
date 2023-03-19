# Week 1 — App Containerization
In this week, we containerized our applications in two images: frontend and backend. These two were built indipendently but communicate with each other.

## Adding a backend docker file
Under the backend-flask directory, we created a docker file with the following contents:
```
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

```
## Building the backend docker image
In order to build a the docker image, we did
```
docker build -t  backend-flask ./backend-flask
```
## Running the docker image
Before running the image, it is preliminary to set FRONTEND_URL and BACKEND_URL to "*" and run them in the background(optional)
```
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker container run --rm -p 4567:4567 -d backend-flask
```
## Building the frontend image
We have to first install npm before building the container
```
cd frontend-react-js
npm i
```
Next we creat a docker file(Dockerfile) under the frontend-react-js folder with the following contents
```
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
**Build the container**
```
docker build -t frontend-react-js ./frontend-react-js
```
**Run the container**
```
docker run -p 3000:3000 -d frontend-react-js
```

## Running multiple containers
At the root of our project, we creat a docker-compose.yml file and input the following
```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

## Adding DynamoDB local and Postgres
We enter the following line in our docker compose file

**Postgres**
```
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

**DynamoDB local**
```
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```
Then install postgres client into gitpod
```
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```
## Testing the application with both frontend and backend running
We right click on the docker-compose.yml, then select compose up, after unlocking the frontend and backend ports, we open the link to the frontend container and we get: 
![Running Cruddur App](https://github.com/Ndzenyuy/aws-bootcamp-cruddur-2023/blob/main/images/w1-running-the-app.png)
