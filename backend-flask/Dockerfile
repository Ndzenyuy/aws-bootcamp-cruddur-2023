FROM python:3.10-slim-buster AS build

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

#EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

FROM alpine:latest
EXPOSE ${PORT}
RUN apk --no-cache add ca-certificates
WORKDIR /backend-flask
COPY --from=build /backend-flask /backend-flask/multi
CMD ["./multi"]
