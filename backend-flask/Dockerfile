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
WORKDIR /usr/local/bin/demo
COPY --from=build /backend-flask /usr/local/bin/demo 
CMD ["./demo"]