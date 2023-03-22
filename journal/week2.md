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
We launch the app by running docker compose up, then we watch [honeycomb](https://ui.honeycomb.io) for the sent metrics:
![Distributed tracing captured in honeycomb](https://github.com/Ndzenyuy/aws-bootcamp-cruddur-2023/blob/main/images/w3%20monitoring.png)

