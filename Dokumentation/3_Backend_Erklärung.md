# Backend 

Der Backend-Code dient dazu, Anfragen vom Frontend zu verarbeiten, das Bild zu bearbeiten und herunterzuladen und eine API-Schnittstelle für diese Aktionen bereitzustellen.

Zunnächst werden alle Module und Pakete importiert, die für die Backend-Entwicklung benötigt werden. Dazu gehören die Umgebungsvariablen aus einer .env-Datei, die Verwendung des Express-Frameworks zur Erstellung eines HTTP-Servers, das Aktivieren von CORS (Cross-Origin Resource Sharing) für die sichere Kommunikation mit verschiedenen Domains. Des Weitern die Einbindung der OpenAI-Bibliothek um die OpenAI-API zu verwenden. Das fs-Modul dient der Verwaltung von Dateien auf dem Server, während das request-Modul für HTTP-Anfragen an andere Server verwendet wird. Schließlich ermöglicht das path-Modul die effiziente Verwaltung von Dateipfaden und Verzeichnissen.

```javascript
// Lade Umgebungsvariablen aus der .env-Datei.
import dotenv from 'dotenv';

// Erstelle eine Express-Anwendung, um einen HTTP-Server für die API bereitzustellen.
import express from 'express';

// Verwende CORS, um Cross-Origin Resource Sharing (CORS) zu ermöglichen.
import cors from 'cors';

// Importiere die OpenAI-Bibliothek, um die OpenAI-API zu nutzen.
import { Configuration, OpenAIApi } from 'openai';

// Verwende das fs-Modul, um Dateien auf dem Server zu verwalten.
import fs from 'fs';

// Importiere das request-Modul, um HTTP-Anfragen an andere Server zu senden.
import request from 'request';

// Verwende das path-Modul, um Dateipfade und Verzeichnisse zu verwalten.
import path from 'path';
```

Zunächst werden Umgebungsvariablen aus einer .env-Datei geladen, um vertrauliche Informationen wie API-Schlüssel sicher zu speichern. Dann wird eine Express-Anwendung erstellt, die als Grundlage für den HTTP-Server der API dient. Der Verzeichnispfad der aktuellen Datei wird bestimmt, um später auf Dateien zuzugreifen.

```javascript
// Lade die Umgebungsvariablen aus der .env-Datei in das aktuelle Umgebungsobjekt.
dotenv.config();

// Erstelle eine Express-Anwendung.
const app = express();

// Verwende die CORS, um Cross-Origin Resource Sharing (CORS) zu ermöglichen.
app.use(cors());

// Verwende die Middleware, um JSON-Anfragedaten zu verarbeiten.
app.use(express.json());

```
Hier wird der Verzeichnispfad der aktuellen Datei ermittelt und in der Variable __dirname gespeichert. Dies ist wichtig, um später auf Dateien und Ressourcen im Kontext der Anwendung zuzugreifen.

```javascript

// Bestimme den Verzeichnispfad der aktuellen Datei.
const __dirname = path.dirname(new URL(import.meta.url).pathname);
```

Das express.static wird verwendet, um statische Dateien im Pfad "/images" bereitzustellen, was hilfreich ist, um Bilder und andere Ressourcen für die Anwendung zugänglich zu machen, alle Bilder und Masken werden dort gespeichert. Zum Beispiel wird das Original Bild dort gespeichert und die Masken-Leinwand. Aus dem Ordner /images werden dann die Bilder für weiterne Aktionen aufgerufen.
```javascript

// Verwende das 'express.static', um statische Dateien im Pfad "/images" bereitzustellen.
app.use('/images', express.static(path.join(__dirname, 'images')));

```

Die OpenAI-Bibliothek wird ebenfalls konfiguriert und mit den Umgebungsvariablen für die Organisation und den API-Schlüssel initialisiert, um später die OpenAI-API für die Bildbearbeitung nutzen zu können.

```javascript
// Konfiguriere die OpenAI-Bibliothek mit den Umgebungsvariablen für Organisation und API-Schlüssel.
const configuration = new Configuration({
    organization: process.env.OPENAI_ORGANIZATION,
    apiKey: process.env.OPENAI_API_KEY,
});
```

In diesem Abschnitt wird die Funktionalität für die Verarbeitung von Bildern und die Interaktion mit der OpenAI-API implementiert. Wenn eine POST-Anfrage auf dem Endpunkt '/process' empfangen wird, werden die in der Anfrage enthaltenen Daten, darunter die URL des Originalbildes, Benutzereingaben und eine Masken-URL, extrahiert. Die Masken-URL wird in eine Bilddatei umgewandelt und gespeichert. Anschließend wird die OpenAI-API aufgerufen, um die Bildbearbeitung durchzuführen. Die bearbeitete Bilddatei wird von der API erhalten, und die URL dieser neuen Textur wird als JSON-Antwort zurückgesendet.

```javascript
const openai = new OpenAIApi(configuration);

app.post('/process', async (req, res) => {
    try {
        // Extrahiere die URL, den Command und die Maske
        const url = req.body.url;
        const userInput = req.body.command;
        const maskDataURL = req.body.mask;

        console.log(url);
        console.log(userInput);

        // Verarbeite die Maskendaten und speichere sie als Bilddatei.
        const base64Data = maskDataURL.replace(/^data:image\/png;base64,/, ""); // Extrahiere den Base64-kodierten Bildteil.
        const buffer = Buffer.from(base64Data, 'base64'); // Erstelle einen Binärdatenpuffer aus dem Base64-kodierten Bildteil.

        // Erstelle einen Dateinamen für die Maskendatei mit dem aktuellen Zeitstempel.
        const maskFilename = path.join(__dirname, 'images', 'mask_' + Date.now() + '.png');

        // Schreibe den Binärdatenpuffer in die Maskendatei.
        fs.writeFileSync(maskFilename, buffer);

        console.log(maskFilename);

        // Rufe die OpenAI-API auf, um die Bildbearbeitung durchzuführen.
        const response = await openai.createImageEdit(
            fs.createReadStream(url),           // Lese das Originalbild.
            userInput,                          // Befehl des Benutzers.
            fs.createReadStream(maskFilename),  // Lese das generierte Maskenbild.
            1,                                  // Anzahl der Bearbeitungen.
            "1024x1024",                        // Gewünschte Ausgabegröße des Bildes.
        );

        // Erhalte die URL des bearbeiteten Bildes aus der API-Antwort.
        const newTextureSrc = response.data.data[0].url;

        console.log(newTextureSrc)

        // Sende die URL des bearbeiteten Bildes als JSON-Antwort.
        res.json({ url: newTextureSrc });

    } 
```
Bei einem Error wird der Fehler geloggt, und wenn es sich um einen OpenAI-API-Fehler handelt, werden auch der HTTP-Statuscode und die API-Fehlerdaten geloggt. Anschließend wird eine JSON-Antwort mit einem Statuscode von 400 gesendet, um das Auftreten eines Problems anzuzeigen.

```javascript
catch (error) { // Errors.

        // Logge den aufgetretenen Fehler.
        console.log(error);

        // Wenn der Fehler eine Antwort (response) und Daten (data) enthält, handelt es sich möglicherweise um einen Fehler von der OpenAI-API.
        if (error.response && error.response.data) {
            console.log("OpenAI API Status:", error.response.status); // Logge den HTTP-Statuscode der API-Antwort.
            console.log("OpenAI API Error:", error.response.data); // Logge die Fehlerdaten von der API.
        } else {
            // Andernfalls logge die Fehlermeldung.
            console.log(error.message);
        }

        // Sende eine Fehlermeldung als JSON-Antwort mit Statuscode 400, wenn ein Problem auftritt.
        return res.status(400).json({
            success: false,
            error: error.message || "Server Problem",
        });
    }
});
```

Diese Route reagiert auf GET-Anfragen und ermöglicht das Herunterladen eines Bildes von einer angegebenen URL. Zunächst werden die URL-Query-Parameter aus der Anfrage extrahiert. 

```javascript
// Definiere eine Route, die auf GET-Anfragen reagiert und ein Bild von einer URL herunterlädt.
app.get('/getImage', (req, res) => {
    // Hole die URL-Query-Parameter aus der Anfrage.
    const url = req.query.url;
```

Dann wird der Dateipfad bestimmt, an dem das heruntergeladene Bild gespeichert werden soll. Falls das Verzeichnis für den Dateipfad nicht existiert, wird es rekursiv erstellt. Rekursiv bedeutet, dass er auch alle übergeordneten Ordner erstellt, falls diese ebenfalls nicht vorhanden sind. 

```javascript
    // Bestimme den Dateinamen und Pfad, an dem das Bild gespeichert wird.
    const filename = path.join(__dirname, 'images', path.basename(url) + '.png');

    // Überprüfe, ob das Verzeichnis für den Dateipfad existiert. Wenn nicht, erstelle es rekursiv.
    if (!fs.existsSync(path.dirname(filename))) {
        fs.mkdirSync(path.dirname(filename), { recursive: true });
    }
```

Eine HTTP-Anfrage wird durchgeführt, um das Bild von der URL herunterzuladen und in eine Datei zu schreiben. Schließlich wird eine JSON-Antwort gesendet, die die lokale URL des heruntergeladenen Bildes enthält, wenn der Vorgang erfolgreich abgeschlossen wurde. Bei Fehlern wird eine Fehlermeldung protokolliert.


```javascript
    // Führe eine HTTP-Anfrage aus, um das Bild von der URL herunterzuladen.
    request(url)
    .pipe(fs.createWriteStream(filename)) // Leite die heruntergeladenen Daten in eine Datei.
    .on('close', () => {
        console.log('File downloaded!');
        // Sende eine JSON-Antwort mit der lokal gespeicherten URL des heruntergeladenen Bildes.
        res.json({ localUrl: '/images/' + path.basename(url)  + '.png' });
    })
    .on('error', (err) => {
        console.error('Error downloading the file:', err);
    });
}); 
```

Schließlich wird der Server gestartet und auf Port 3000 gehört, um Anfragen entgegenzunehmen.

```javascript
// Starte den Server und höre auf Port 3000.
app.listen(3000, () => {
    console.log('Server listening on port 3000');
});
```
