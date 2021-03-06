[Next Lab](https://github.com/Bart-2020/rubix-tci-flogo/blob/master/lab_02/lab_02.md)

# LAB-01 Maak een Flogo app op het TIBCO Cloud™ Integration platform (iPaaS).

**Doel:** In dit eerste lab maken we een Flogo app op het TCI platform, om kennis te maken met de basics van Flogo.



Een Flogo App is te vatten in onderstaand - eenvoudig - model:

![](https://github.com/Bart-2020/rubix-tci-flogo/blob/master/lab_01/FlogoApp%20model.jpg)

Handig dit plaatje steeds in het achterhoofd te hebben.

Onze eerste **Flogo App** zal een eenvoudige RESTService worden. Afhankelijk van één input (path) parameter, antwoordt de service met een reply message (body). 
We realiseren deze service in drie stappen:

- We beginnen met de **Trigger**
  De trigger (BW-*resource*) en de flow zijn onafhankelijk van elkaar (zolang ze geen binding hebben). Je zou dus ook eerst met het bouwen van de flow kunnen beginnen.
  Onder andere de trigger output - en reply settings definiëren we hier.
- Vervolgens definiëren we de input - en output parameters van de **Flow**
  De binding van trigger en flow kan nu gemaakt worden.
- Als laatste stellen we de flow (BW-*process*) samen door het toevoegen van **Activities** (BW-*activities*), die gelinkt kunnen worden met **Branches** (BW-*transitions*).

Daarna zijn we klaar om te gaan testen. We voeren twee tests uit:

- We testen de flow (BW-*tester*)
  Met een test-input en we checken de flow output
- Ten slotte testen we de Flogo App (een RESTService) nadat we deze pushed (deployed) hebben.
  We sturen een test-trigger (service request) en checken de reply message.

Na elke stap stoppen we even en doen we een korte evaluatie en/of check.



Ga naar [TIBCO Cloud™](https://eu.account.cloud.tibco.com/manage/home) en aan de slag!

--------------------------------------------------



### Flogo App
Click op de *Integration* hyve en daarna door naar [Integration Apps](https://eu.integration.cloud.tibco.com/applications).
Rechtsboven zie je de blauwe *Create/Import* button.

Kies onder "Develop" voor *Flogo* en de blauwe *Create New App* button. *Create* je app door onder "Create new" een Trigger toe te voegen.

Zoals hieboven aangegeven, starten we met het maken en configureren van de Trigger.



---------------------------------------------

### Trigger [1/5]

We bouwen een RESTService, dus we hebben een HTTP connector nodig. Click op de + en blader eens door de Triggers catalog.

Kies voor een *Receive HTTP Message* trigger. *Create* deze Trigger (zonder swagger te uploaden - Configure Using API Specs = False):

Trigger Settings - Port:

```
8443
```

Je komt nu in de "Trigger View" en daar voegen we eerst een *+New Flow* toe, door 'm enkel een naam te geven. Daarna gaan we weer verder met het configureren van de *Receive HTTP Message* trigger.

Method:

```
GET
```

... en configureer path parameter "tekst" in het Resource Path. Gebruik {} om path parameters te configureren:

```
/groeten/{tekst}
```

Zonder JSON Schema of - voorbeeld bericht.

Kies na *Continue* ervoor de gedefinieerde Trigger output gelijk aan de Flow input definitie te maken met *Copy Schema*.

Ga verder met de Trigger configuratie (icoontje met oranje waarschuwingsdriehoekje - omdat de trigger --> flow mappings ontbreken)



Check de Path Parameter "tekst" bij Output Settings.

Nu mappen we de Trigger output op de Flow input. Ga door naar Map to Flow Inputs.

**{}**   pathParams   **a..**   tekst

```json
$trigger.pathParams.tekst
```

Definieer Reply Settings zonder Configure Response Codes.

We hoeven hier alleen een voorbeeld bericht te geven om het reply schema te definiëren.
Reply Data Schema:

```json
{
  "greetz-returned": "string"
}
```

Nu mappen we het Flow output schema op het Trigger reply schema. Map from Flow Outputs - to Trigger Reply:

**1..**   code

```json
$flow.code
```

**{}**   data   **a..**   greetz-returned

```json
$flow.data["greetz-returned"]
```



*Sync* synchroniseert de Trigger output en de - Reply met respectievelijk de Flow Input en - Output. De trigger schema's en flow schema's zijn zo gelijk en binded.



------

### Flow[2/5]

We maken de meest eenvoudige flow, met één activity en zonder branches.

###### input & output

We definiëren eerst de flow input & output; Omdat we met de Trigger begonnen zijn is dit - door *Sync* - al gebeurd. Check de JSON schema's:

###### Input

```json
{
    "type": "object",
    "title": "ReceiveHTTPMessage",
    "properties": {
        "pathParams": {
            "type": "object",
            "properties": {
                "tekst": {
                    "type": "string"
                }
            },
            "required": []
        },
        "headers": {
            "type": "object",
            "properties": {
                "Accept": {
                    "type": "string",
                    "visible": false
                },
                "Accept-Charset": {
                    "type": "string",
                    "visible": false
                },
                "Accept-Encoding": {
                    "type": "string",
                    "visible": false
                },
                "Content-Type": {
                    "type": "string",
                    "visible": false
                },
                "Content-Length": {
                    "type": "string",
                    "visible": false
                },
                "Connection": {
                    "type": "string",
                    "visible": false
                },
                "Cookie": {
                    "type": "string",
                    "visible": false
                },
                "Pragma": {
                    "type": "string",
                    "visible": false
                }
            },
            "required": []
        }
    }
}
```

###### Output 

```json
{
    "type": "object",
    "title": "Inputs",
    "properties": {
        "code": {
            "type": "integer",
            "required": false
        },
        "data": {
            "$schema": "http://json-schema.org/draft-04/schema#",
            "type": "object",
            "properties": {
                "greetz-returned": {
                    "type": "string"
                }
            }
        }
    },
    "required": []
}
```



-----------------

### Activity[3/5]

###### mapping

Je voegt een activity toe aan de flow door na *+* een activity uit een category te kiezen. Kies *Default* activity *Return*. Map de *Return* activity output op de flow output:

**{}**   data   >   **a..**   greetz-returned

```json
string.concat("Hartelijk '", $flow.pathParams.tekst, "' terug gewenst ...")
```

Je geeft natuurlijk altijd 200 OK terug :-)



------

### Testen van de Flow[4/5]

Met de blauwe *Test* button kunnen we nu deze Flow testen. *Create a Launch Configuration* door het invullen van je flow test input bij Mapping settings:

**{}**   flowInputs   >   **{}**   pathParams   **a..**   tekst

```json
"Hallootjes!"
```

Na *Next* zie je je input settings bevestigd en *Run* start de flow test. De test output (INFO log) op je console output:

```json
2020-08-25T19:20:43.206Z INFO [flogo] - Starting event publisher
TIBCO Flogo® Runtime - 2.9.0 (Powered by Project Flogo™ - v1.0.0)
2020-08-25T19:20:43.250Z INFO [flogo.flow] - Instance [7a4767900a94e547d7263d3339865c78] Done
Flow execution successful
{
"code": 200,
"data": {
"greetz-returned": "Hartelijk 'Hallootjes!' terug gewenst ..."
}
}
```





-----------------------------

### Testen RESTService - API[5/5] 

Test de RESTService API met de blauwe *Push* button. De Flogo App is nu deployed en je kunt 'm runnen door op te schalen van 0 naar 1 met ^ en *Scale* (of door op je applications pagina gewoon *Run* te clicken).

Je hebt nu 1 running instance die je kunt testen op het (onder "Endpoints") aangegeven public Endpoint. Kies daar *Test*.

Vul een tekst value in en *Try it out!*

#### Curl

```
curl -X GET "https://eu-west-1.integration.cloud.tibcoapps.com:443/zolm26ucoicnhu5tjciohjuv3rizxtyt/groeten/Daaaag" -H "accept: application/json"
```

#### Request URL

```
https://eu-west-1.integration.cloud.tibcoapps.com:443/zolm26ucoicnhu5tjciohjuv3rizxtyt/groeten/Daaaag
```

#### Response Body

```json
{
  "greetz-returned": "Hartelijk 'Daaaag' terug gewenst ..."
}
```

#### Response Code

```
200
```

#### Response Headers

```json
{  "cache-control": "max-age=0, no-cache, no-store, must-revalidate",  "content-length": "59",  "content-type": "application/json; charset=UTF-8",  "expires": "-1",  "pragma": "no-cache"}
```



Je kunt uiteraard ook met Postman en/of soapUI testen, of de [Request URL](#Request URL) van je eerste Flogo app gewoon in je browser plakken ...

[Next Lab](https://github.com/Bart-2020/rubix-tci-flogo/blob/master/lab_02/lab_02.md)
