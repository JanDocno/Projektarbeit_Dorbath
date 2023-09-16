# Entwicklung der interaktiven Anwendung mittels OpenAI Dall-E 2
Die folgenden Seiten beinhalten die Dokumentation zum Erstellen einer interaktiven Anwendung des ausgewählten KI-Modells Dall-E 2.

Um die Verbindung zu der API von Open AI zu testen, wird zunächst an zwei einfachen Beispielen der Workflow erklärt.

Zuerst ist sicherzustellen, das folgende Programme installiert sind.
1. Installieren von Visual Code (VC)
2. Installieren von Node.js

In VC wird eine Javascript Datei geöffnet.

```java script
// Express Module laden
const express = require('express');                     //Express Modul
const app = express();                                  //Eine Instanz der Express-App erstellen
app.use(express.json());                                //Anforderungen mit JSON-Daten zu parsen
```

Das Modul Express ist ein Framework für Node.js, das die Entwicklung von Webanwendungen vereinfacht. Es bietet eine Reihe von Funktionen und Middleware, die es Entwicklern ermöglichen, schnell und effizient APIs, Webanwendungen und andere serverseitige Anwendungen zu erstellen.

```java script
//OpenAI Modul laden
require("dotenv").config();                             //Lädt das dotenv-Modul 
const { Configuration, OpenAIApi } = require("openai"); //Lädt das OpenAI Modul
const configuration = new Configuration({
    apiKey: process.env.OPENAI_API_KEY,                 //Lädt den API KEY aus .env
  });
const openai = new OpenAIApi(configuration);            //Erstellt eine Instanz der OpenAIApi-Klasse und die Configuration-Instanz übergeben
```

Dieser Code implementiert den API Key, dieser von Open AI generiert wird und über den eigenen Account zu Verfügung gestellt wird. require("dotenv").config() lädt das dotenv-Modul und ruft die config()-Funktion auf, um die Umgebungsvariablen aus der ".env"-Datei zu laden da der API-KEY nicht öffentlich sein soll. Eine Instanz der Configuration-Klasse wird erstellt. Der API-Schlüssel für die OpenAI API wirf aus der Umgebungsvariable OPENAI_API_KEY gelesen, die zuvor aus der ".env"-Datei geladen wurde. Eine Instanz der OpenAIApi-Klasse wird erstellt und die Configuration-Instanz übergeben. Es wird eine Verbindung zur OpenAI-API hergestellt und die openai Variable kann verwendet werden.

## Beispielcode 1 Text Modell Davinci

Dies ist ein Beispiel die API mit dem Text-Modell "Text-Davinci-003" anzusprechen. Das Modell wird anfangs verwendet da die Kosten des Modells sehr günstig sind.

```java script
// Routen definieren
app.post('/', async (req, res) => {                         //POST-Anfrage auf den Endpunkt /.req = request und res= response
    try{                                                    //Beginn eines try-catch-Blocks zur Fehlerbehandlung
        const response = await openai.createCompletion({    //Aufruf an die OpenAI-API, um Textgenerierung durchzuführen
            model: "text-davinci-003",                      //Das Modell "text-davinci-003" wird verwendet
            prompt: "Hello OPEN AI API!",
            max_tokens: 7,
          });
          
    return res.status(200).json({                            //Sendet eine erfolgreiche Antwort mit dem generierten Text an den Client.
            success: true,
            data: response.data.choices[0].text,
        });
    }catch(error){                                           //Fangen von Fehlern, falls welche auftreten.
        return res.status(400).json({                        //Fehlerantwort an den Client. Fehler wird aus der API-Antwort oder als allgemeine Fehlermeldung zurückgegeben.
            success: false,                                                     
            error: error.response
            ? error.response.data: "Server Problem",
        })
    }
});
```
Der Code definiert eine POST-Route für den Endpunkt "/". Die POST-Anfrage soll eine Textgenerierung mit Hilfe der OpenAI-API durchzuführen. 
Als Prompt wird die API begrüßt. Das Modell "text-davinci-003" wird verwendet, um die Generierung mit einer 
maximalen Tokenlänge von 7 durchzuführen. Die generierte Antwort wird als erfolgreiche JSON-Antwort an den Client zurückgesendet. Im Falle eines 
Fehlers wird eine Fehlerantwort mit der entsprechenden Fehlermeldung zurückgegeben.

```java script
// Server starten und auf Verbindungen lauschen
const port = 3000;
app.listen(port, () => {
  console.log(`Server läuft auf Port ${port}`);
});
```
Der Code startet den Express-Server und lässt ihn auf Verbindungen auf dem Port 3000 lauschen, wobei eine Konsolenausgabe "Server läuft auf Port 3000" erfolgt.

### Beispielcode 1 Ausführen

Der Code wird nun in VS Code über den Terminal gestartet. Dazu werden die folgenden Befehle ausgeführt.
```
npm install express dotenv openai
node app.js   
```

Nun wird über die Postman-Software die API Anfrage dokumentiert. Dazu wird eine neue Collection Namens Localhost angelegt und in Variables ein New Response. Eine Post-Anfrage an http://localhost:3000 ergibt folgendes Ergebnis.

```Jason
{
    "success": true,
    "data": "\n\nHi there! Welcome to"
}
```
Aus der JASON-Antwort ist die API-Funktionalität ersichtlich.

## Beispielcode 2 Bildgenerierung Modell Dall-E 2

Dies ist ein Beispiel die API von DALL-E 2 anzusprechen. Das Modell wird verwendet um ein Bild zu generieren.

```java script
// Routen definieren
app.post('/', async (req, res) => {                                             //POST-Anfrage auf den Endpunkt /.req = request und res= response
    try{                                                                        //Beginn eines try-catch-Blocks zur Fehlerbehandlung
        const response = await openai.createImage({                             
        prompt: "A cute baby sea otter",
        n: 1,
        size: "256x256",
        });      

        const generatetdImageUrl = response.data.data[0].url;                   //Speichert die URL des Bildes

    return res.status(200).json({                                               //Sendet eine erfolgreiche Antwort mit dem generierten Text an den Client.
            success: true,
            data: generatetdImageUrl,
        });

    }catch(error){                                                              //Fangen von Fehlern, falls welche auftreten.
        return res.status(400).json({                                           //Fehlerantwort an den Client. Fehler wird aus der API-Antwort oder als allgemeine Fehlermeldung zurückgegeben.
            success: false,                                                     
            error: error.response
            ? error.response.data: "Server Problem",
        })
    }
});
```

Im Vergleich zu Beispielcode 1 wird hier nun die Funktion openai.createImage verwendet und die Parameter "Promt", "n= Anzahl" und "size" verwendet. Die URL des entstandenen Codes erhält man durch den Aufruf response.data.data[0].url.

### Beispielcode 2 Ausführen

Der Code wird nun in VS Code über den Terminal gestartet. Dazu werden die folgenden Befehle ausgeführt.
```
npm install express dotenv openai
node app.js   
```

Nun wird über die Postman-Software die API Anfrage dokumentiert. Dazu wird eine neue Collection namens Localhost angelegt und in Variables ein New Response. Eine Post-Anfrage an http://localhost:3000 ergibt folgendes Ergebnis.

```Jason
{
    "success": true,
    "data": "https://oaidalleapiprodscus.blob.core.windows.net/private/org-1Q2Yri36ix52Mo5PBkAySuey/user-oabycNFQ69CTjYz7hYvze0x6/imgXMIc1jVAkFrDO9GKrcWiWhIc.png?st=2023-05-26T08%3A40%3A13Z&se=2023-05-26T10%3A40%3A13Z&sp=r&sv=2021-08-06&sr=b&rscd=inline&rsct=image/png&skoid=6aaadede-4fb3-4698-a8f6-684d7786b067&sktid=a48cca56-e6da-484e-a814-9c849652bcb3&skt=2023-05-25T20%3A46%3A58Z&ske=2023-05-26T20%3A46%3A58Z&sks=b&skv=2021-08-06&sig=C42k4UZyfuPrvnJ55UR9h1LVCJx6etIgk3MkcKKX5Xs%3D"
}
```
Aus der JASON-Antwort die API-Funktionalität ersichtlich und der URL Code kann aufgerufen werden, um das Bild zu sehen.

## Bildgenerator Web-Anwendung mittels Dall-E

Das ist der HTML-Code für eine einfache Web-Anwendung, die es dem Benutzer ermöglicht, eine Bildbeschreibung einzugeben und ein Bild mit Hilfe des KI-Modells Dall-E 2 zu generieren. 

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>OpenAI Bildgenerator</title>
    <link rel="stylesheet" type="text/css" href="./styles.css">                     <!--Link zu einer CSS-Datei-->
</head>
<body>
    <h1>OpenAI Bildgenerierung</h1>

    <div id="imageContainer">                                                       <!--Bild positionieren-->
        <div id="image">
            <img id="uploadedImage" src="<%= imageUrl %>" alt="Image"></img>        <!--src = dynamische Variable zum Pfad des generierten Bild-->
        </div>
    </div>

    <div id="commands">                                                             <!--Container für die Benutzereingabe-->
        <input type="text" id="commandInput" placeholder="Was möchten Sie sehen?">  <!--ID zum Wert auszulesen-->
        <button id="generateButton">Ausführen</button>                              <!--Button zum Generieren des Bildes-->
    </div>
</body>
</html>
```
In dem gegebenen HTML-Code wird EJS verwendet, um dynamische Inhalte einzufügen. EJS ist eine Template für Node.js, welches ermöglicht, JavaScript in einer HTML Datei einzubetten.

Die dynamische Variable <%= imageUrl %> ist ein EJS-Tags, der darauf hinweist, dass ein Wert aus einer serverseitigen Variable oder einer JavaScript-Funktion eingefügt werden soll. Der genaue Wert für imageUrl wird später in einem serverseitigen Skript gesetzt werden, bevor die HTML-Seite an den Client gesendet wird. Die Variable imageUrl könnte beispielsweise den Pfad zum generierten Bild enthalten, sodass das Bild im <img>-Tag korrekt angezeigt wird.

./styles ist ein relativer Pfad zu einer CSS-Datei namens "styles.css". Der ./ am Anfang des Pfads bedeutet, dass die CSS-Datei relativ zur aktuellen HTML-Datei gefunden werden soll. In diesem Fall wird davon ausgegangen, dass sich die "styles.css"-Datei im selben Verzeichnis wie die HTML-Datei befindet.

Um die EJS-Tags in HTML-Code zu verarbeiten und den dynamischen Inhalt zu generieren, muss die HTML-Datei von einem EJS-Renderer verarbeitet werden. Dies geschieht serverseitig, indem die HTML-Datei in einem serverseitigen Framework wie Express.js mit einem EJS-Renderer gerendert wird. Dabei werden die EJS-Tags durch die entsprechenden Werte ersetzt, bevor die HTML-Seite an den Client gesendet wird.

Folgender JavaScript Code erfasst die Texteingabe des Benutzers und sendet diese an das Backend.

```Javascript

    <script>
        document.getElementById("generateButton").addEventListener("click", () => { //"generateButton" ruft folgende Funktion aus
            
            const commandInput = document.getElementById("commandInput").value;     //Der Input aus dem Texteingabefeld wird aufgerufen und in der Variabel "commandInput" gespeichert
            fetch("/", {
                method: "POST",                                                     //Eine POST-Anfrage wird an den Serverendpunkt "/" gesendet
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({ command: commandInput })                     //Der Input aus dem Texteingabefeld wird als JSON im Body der Anfrage gesendet

            })
            .then((response) => response.json())                                    //Die Antwort der Anfrage wird in JSON-Format umgewandelt
            .then((data) => {                                                       //Die JSON Daten werden in der folgenden Funktion verarbeitet
                if (data.imageUrl) {                                                //Wenn die Antwort Daten eine "imageUrl" enthält (also das generierte Bild), wird der folgende Code ausgeführt
                    
                    document.getElementById("uploadedImage").src = data.imageUrl;   //Das "src"-Attribut des Bildes mit der ID "uploadedImage" wird auf die generierte Bild-URL gesetzt
                } else {
                    console.error("Fehler beim Generieren des Bildes.");            //Wenn die Antwort keine "imageUrl" enthält, wird eine Fehlermeldung in der Konsole ausgegeben
                }
            })
            .catch((error) => {
                console.error("Fehler beim Generieren des Bildes:", error);         //Bei einem Fehler während der Anfrage oder Verarbeitung der Antwort wird eine Fehlermeldung in der Konsole ausgegeben
            });
        });
    </script>

```
### Beispielcode 2 verändern

Beispielcode 2 wird nun wie folgt verändert:
Dieser Code definiert die Serverlogik für die GET- und POST-Anfrage. Die GET-Anfrage rendert die Index-Seite, während die POST-Anfrage den generierten Befehl verarbeitet und das generierte Bild als JSON-Antwort zurückgibt.

``` Javascript
// GET Route, um die Index-Seite zu rendern
app.get("/", (req, res) => {
    res.render("index", { imageUrl: "" });              //res.render rendert die "index"-Vorlage und sendet diese an den Client
                                                        //Die Vorlage initialisiert "imageUrl" mit einem leeren String
                                                        //Dies ermöglicht eine dynamische Darstellung des Bildes auf der Webseite
});

// POST Route, um den generierten Befehl zu verarbeiten
app.post('/', async (req, res) => {
    try {
        const userInput = req.body.command; // Benutzereingabe erhalten
        
        const response = await openai.createImage({     //openai.createImage-Methode generiert das Bild
            prompt: userInput,                          //Die Benutzereingabe übergeben
            n: 1,                                       //Anzahl der generierten Bilder
            size: "256x256",                            // Gewünschte Bildgröße
        });
    
        const generatedImageUrl = response.data.data[0].url;    //Die Bild-URL wird extrahiert
    
        res.json({ imageUrl: generatedImageUrl });              //Generierte Bild-URL wird als JSON-Antwort an den Client gesendet

  
    } catch (error) {                                   //Fangen von Fehlern, falls welche auftreten
        return res.status(400).json({                   //Fehlerantwort an den Client
            success: false,
            error: error.response ? error.response.data : "Server Problem",
        });
    }
});
  });
  ```
  
Um die Anwendung zu starten müssen nun folgende Programme über das Terminal initialisiert werden.

```
npm install dotenv express openai ejs
```

Die Web Anwendung ist nun unter https://localhost:3000 aufrufbar.
