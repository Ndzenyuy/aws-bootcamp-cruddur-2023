# Week 1 — App Containerization
In this week, we containerized our applications in two images: frontend and backend. These two were built indipendently but communicate with each other.

##Adding a backend docker file
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
