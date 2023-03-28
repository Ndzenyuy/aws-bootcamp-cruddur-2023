# Week 2 — Distributed Tracing
In this week, we had to configure our app to send metrics to honeycomb in order to carry out tracing. Tracing is very important in the life of an application so that the user experience can be monitored so as to improve the app when it get slow or bugs due to the many users signing and using our app

## Honeycomb
The instructions below are necessary for creating new datasets in honeycomb We add the following to our requirements.txt file
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
Then in the app.py file, we add the following:
```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
```
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```
```
# Initialize automatic instrumentation with Flask
# these goes under the app = Flask(__name__)

FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```
We add the following Environment variables to backend-flask in docker compose 
```
OTEL_SERVICE_NAME: 'backend-flask'
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

Then the api key is to be gotten from honeycomb and set in gitpod

```
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```
### Capturing metrics for home activities
We edit the home_activities.py file in order to configure data to be sent to honeycomb, first the name of the trace is set through
```
from opentelemetry import trace

tracer = trace.get_tracer("home.activities")
```
Just below def_run(), we insert the code; 
```
with tracer.start_as_current_span("home-activities-mock-data"):
```
and just above the return, we insert
```
span.set_attribute("app.result_length", len(results))
```
These will sent home activities metrics to honeycomb, we can use heatmap and p90 to monitor the duration and the speed of each activity

We launch the app by running docker compose up, then we watch [honeycomb](https://ui.honeycomb.io) for the sent metrics:
![Distributed tracing captured in honeycomb](https://github.com/Ndzenyuy/aws-bootcamp-cruddur-2023/blob/main/images/w3%20monitoring.png)

## Amazon x-ray
Next we cequally onfigured XRAY to monitor metrics in the backend of the Cruddur, in the steps aheah, we'll see how xray was configured

Add the following to requirements.txt file
```
aws-xray-sdk
```
Then install requirements for the xray dependencies to be installed
```
pip install -r requirements.txt
```
Next we add the following to app.py
```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```
### Setting up AWS XRAY resources
We create a new file under aws/json/xray.json and populate it with
```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```
on the cli, we run the following 
```
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\") {fault OR error}"
``` 

```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
### We add the deamon service to docker-compose.yml
```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```
We also need to add two environment variables to docker-compose file
```
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

![xray up and running]()
and finaly we did Cloudwatch logs

## Cloudwatch logs
We add the following to requirements.txt
```
watchtower
```
On the cli, we run
```
pip install -r requirements.txt
```

