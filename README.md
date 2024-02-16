[![License](https://img.shields.io/badge/license-Apache%202.0-blue)](https://github.com/manwestc/FOGES/blob/main/LICENSE)
[![DOI](https://zenodo.org/badge/758401776.svg)](https://zenodo.org/doi/10.5281/zenodo.10669668)

# FOGES

_From Cloud and Fog Computing to Federated-Fog Computing: A Comparative Analysis of Computational Resources in Real-Time IoT Applications based on Semantic Interoperability_

## Steps
The used mapping for these scenarios is also available at this [auroral repository](https://auroralh2020.github.io/auroral-ontology-contexts/foggy/mapping.ftl). Even though every scenario explores a different configuration, the base data flux goes like this:

1. Sensors collects data.
2. Sensor data is sent to the HELIO-REST service.
3. Data is transformed to a JSON-LD format.  
4. JSON-LD sensor data is inserted into the GraphDB instance.

## Step 1: Data creation 

Data is collected from sensors into an array where each object has following format:

``` json
{
    "parameter": "temperature",
    "timestamp": 100023123123,
    "value": 23
}
```

This array is sent to the HELIO web service available in every scenario.

## Step 2: HELIO-REST service

The HELIO-REST web service receives the data through the parameter `sensor_data` and iterates on each sensor reading. Before converting data into JSON-LD format, a couple of json objects are being used to map the `parameter` value in sensors. This `parameter` value is replaced to the corresponding property defined in the SAREF Ontology. Full context is available also in the [Auroral repository](https://auroralh2020.github.io/auroral-ontology-contexts/foggy/context.json). The same process is used to add the corresponding unit of measure from the UnitsOfMeasure Ontology.

```json
{
    "@context": {
        "core": "https://auroral.iot.linkeddata.es/def/core#",
        "adapters": "https://auroral.iot.linkeddata.es/def/adapters#",
        "saref": "https://saref.etsi.org/core#",
        "om": "http://www.ontology-of-units-of-measure.org/page/om-2#",
        "id": "core:hasId",
        "serial": "core:serialNumber",
        "version": "core:versionNumber",
        "measures": {
            "@id": "saref:makesMeasurement",
            "@type": "@id",
            "@context": {
                "id": "saref:hasId",
                "timestamp": "saref:hasTimeStamp",
                "value": "saref:hasValue",
                "relates": {
                    "@id": "saref:relatesToProperty",
                    "@type": "@vocab"
                },
                "measuredIn":{
                    "@id":"saref:isMeasuredIn",
                    "@type": "@vocab"
                }
            }
        },
        "device": "core:Device",
        "humiditySensor": "adapters:HumiditySensor",
        "sensor": "adapters:Sensor",
        "thermometer": "adapters:Thermometer",
        "airQualitySensor": "adapters:AirQualitySensor",
        "weatherSensor": "adapters:WeatherSensor",
        "relativeHumidity": "adapters:RelativeHumidity",
        "atmosphoricPressure": "adapters:AtmosphoricPressure",
        "ambientTemperature": "adapters:AmbientTemperature",
        "noiseLevel": "adapters:NoiseLevel",
        "illuminance": "adapters:Illuminance",
        "cO2Concentration": "adapters:CO2Concentration",
        "uVIndex": "adapters:UVIndex",
        "pM10Concentration": "adapters:PM10Concentration",
        "pM2.5Concentration": "adapters:PM2.5Concentration",        
        "o3Concentration": "adapters:O3Concentration",
        "relativeHumidityUnit": "om:Percentage",
        "hectopascal": "om:hectopascal",
        "degreeCelcius": "om:degreeCelcius",
        "lux":"om:lux",
        "partsPerMillion": "om:PartsPerMillion",
        "wattPerSquareMetre": "om:wattPerSquareMetre",
        "metrePerSecond": "om:PrefixedMetrePerSecond-Time",
        "decibel": "core:Decibel"
    }
}
```

## Step 3: JSON-LD transformation

Using the context built in the previous step, now it is possible to generate JSON-LD sensor data according to the SAREF Ontology. In the end, this data is represented in JSON-LD format as the following.

```json
{
    "@context":"https://auroralh2020.github.io/auroral-ontology-contexts/foggy/context.json",
    "measures":[
        {
          "@type": "saref:Measurement",
          "id": "urn:uuid:upm:sensor:temperature:1690193198",
          "timestamp": "2023-07-24T10:06:38+02:00",
          "value": 21,
          "relates": "ambientTemperature",
          "measuredIn": "degreeCelcius"
        }
    ]
}
```

## Step 4: GraphDB insertion

Now that there is data in a JSON-LD format, it is possible to insert it into the graph through a POST request to the GraphDB instance. 

```
<#assign body="INSERT DATA {   GRAPH <https://ts.fogger3.linkeddata.es/abced>  { " +rdf+ " } }">
<#assign graph="https://ts.fogger3.linkeddata.es/repositories/data-cloud/statements">
<#assign dataset>
{
            "method": "POST",
            "url": "[=graph]",
            "body": "[=body?js_string]",
            "headers" : {  "Content-Type" : "application/sparql-update" }
}
</#assign>
```

Finally, some time values are sent back in order to measure reading, transformation and insertion times.


## The docker configuration file
The file `docker.yml` contains the base Docker configuration file used in this experiment. Depending on the role of each **service** some services are disabled. 