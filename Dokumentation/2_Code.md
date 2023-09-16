# Interaktive 360 Anwendung

## Three.js (Frontend)
Dies ist der komplette Code für das Frontend.

```html
<!DOCTYPE html>
<html>
<body>
    <!-- Ein Container-Bereich, der das gerenderte 360 Grad Bild anzeigt. -->
    <div id="viewer"></div>

    <!-- Ein Container-Bereich für den Radiergummi-Button (Maskieren-Button). -->
    <div id="eraser">
        <button id="eraserButton">Maskieren</button>
    </div>

    <!-- Ein Container-Bereich für die Eingabe und den Ausführungs-Button. -->
    <div id="commands">
        <!-- Ein Feld für den Benutzer, um die gewünschte Änderung anzugeben. -->
        <input type="text" id="commandInput" placeholder="Was moechten Sie veraendern?...">
        <!-- Ein Button zum Ausführen der gewählten Änderung. -->
        <button id="inpaintingButton">Ausfuehren</button>
    </div>

    <script type="module"> // Erstelle ein Javascript Tag.
        // Importe
        import * as THREE from 'three';
        import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
    
        // Initsalisierung
        var textureUrl = "YOUR_PANO.png"; // 360 Grad Panorama Bild.
        var eraserEnabled = false;
        var mouseDown = false;

        // Erstelle eine Scene und Kamera.
        var scene = new THREE.Scene(); 
        const width = window.innerWidth / 1.1;
        const height = window.innerHeight / 1.1;
        var camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
        
        // Erstelle einen WebGL-Renderer (Web Graphics Library), der zur Darstellung der Szene im Browser verwendet wird.
        var renderer = new THREE.WebGLRenderer(); 
        renderer.setSize(width, height);

        // Füge das gerenderte Canvas-Element des WebGL-Renderers in das HTML-Dokument ein.
        document.getElementById("viewer").appendChild(renderer.domElement);
    
        // Erstelle eine geometrische Form einer Kugel.
        var geometry = new THREE.SphereGeometry(1, 32, 32);

        // Spiegel die X-Achse der Geometrie.
        geometry.scale(-1, 1, 1);
    
        var maskCanvas, maskContext, maskTexture;
    
        // Lade die Textur und erstelle die Masken-Leinwand nachdem die Textur geladen ist.
        var texture = new THREE.TextureLoader().load(textureUrl, function(texture) {
            // Erstelle eine Canvas-Leinwand im Hintergrund und den zugehörigen Kontext.
            maskCanvas = document.createElement('canvas');
            maskCanvas.width = texture.image.width;
            maskCanvas.height = texture.image.height;
            maskContext = maskCanvas.getContext('2d');
    
            // Setze alle Pixel auf weiß.
            maskContext.fillStyle = 'white';
            maskContext.fillRect(0, 0, maskCanvas.width, maskCanvas.height);
    
            // Erstelle eine Canvas Texture
            maskTexture = new THREE.CanvasTexture(maskCanvas);
    
            // Das Material, muss die Masken-Textur als Alpha Kanal verwenden.
            material.alphaMap = maskTexture;
            material.transparent = true;

            // Event Listener für Mausbewegung
            renderer.domElement.addEventListener('mousemove', function(event) {
                if (eraserEnabled && mouseDown) {
                    // Konvertiere die Mausposition (-1 bis +1).
                    var mouse = new THREE.Vector2();
                    mouse.x = (event.clientX / renderer.domElement.clientWidth) * 2 - 1;
                    mouse.y = -(event.clientY / renderer.domElement.clientHeight) * 2 + 1;
    
                    // Erstelle einen Raycaster, um einen Strahl vom Mauszeiger aus in das Bild zu projizieren.
                    var raycaster = new THREE.Raycaster();

                    // Dabei wird der Strahl wie die Kamera ausgerichtet.
                    raycaster.setFromCamera(mouse, camera);    
                    
                    // Führe eine Schnittprüfung (Intersection Test) zwischen dem Strahl und dem Bild durch.
                    var intersects = raycaster.intersectObject(sphere);    

                    // Überprüfe, ob es Schnittpunkte gibt.
                    if (intersects.length > 0) {
                        // Wenn es Schnittpunkte gibt, hole die UV-Koordinaten des ersten Schnittpunkts.
                        var uv = intersects[0].uv;    

                        // Konvertiere die UV-Koordinaten in Pixelkoordinaten.
                        var x = Math.floor(uv.x * maskCanvas.width);
                        var y = Math.floor((1 - uv.y) * maskCanvas.height);  // Die y-Achse ist gespiegelt.
    
                        // Lösche auf der Masken-Leinwand.
                        // Setze den Composite-Modus des Masken-Kontexts auf "destination-out".
                        // Dies bewirkt das überschneidende Inhalte entfernt werden.
                        maskContext.globalCompositeOperation = 'destination-out';

                        // Beginne die Maskierung.
                        maskContext.beginPath();

                        // Zeichne einen Kreis mit der funktion .arc (arcuate), um Bildinhalt zu entfernen.
                        // Die Kreismitte befindet sich bei den gegebenen Koordinaten (x, y).
                        maskContext.arc(x, y, 20, 0, 2 * Math.PI);

                        // Fülle den gezeichneten Kreis.
                        maskContext.fill();
    
                        // Zurück zur Standard-Zusammensetzung.
                        maskContext.globalCompositeOperation = 'source-over';
    
                        // Sage Three.js Bescheid, dass die Textur aktualisiert werden muss.
                        maskTexture.needsUpdate = true;
                    }
                }
            }, false);
        });
        
        // Erstelle ein Material für das 360 Grad Panorama Bild.
        // Die Textur wird von beiden Seiten der Kugel angezeigt, damit beim Maskieren die Kugel vollständig freigelegt wird.
        var material = new THREE.MeshBasicMaterial({map: texture, side: THREE.DoubleSide});

        // Erstelle ein Spähre mit der zuvor definierten Geometrie und dem Material.
        var sphere = new THREE.Mesh(geometry, material);

        // Füge die Sphäre der Szene hinzu.
        scene.add(sphere);

        // Setze die Ausgangsposition der Kamera im Raum.
        camera.position.set(0, 0, 0.1);

        // Erstelle Orbit-Steuerungen, um die Kamera im Raum zu bewegen.
        var controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true; 
        controls.dampingFactor = 0.05;
        
        // Funktion zum Rendern des 360 Grad Panorama Bild, die in einer Schleife aufgerufen wird.
        function render() {
            requestAnimationFrame(render);
            renderer.render(scene, camera);
            controls.update(); // Aktualisiere die Kamerasteuerung.
        }

        //Event Listener für Button, sendet userInput an updateTexture Funktion
        document.getElementById('inpaintingButton').addEventListener('click', function() {
            var userInput = document.getElementById('commandInput').value;
            updateTexture(userInput);
        });
    
        // Event Listener für Button, Radiergummi und Orbit 
        document.getElementById('eraserButton').addEventListener('click', function() {
            eraserEnabled = !eraserEnabled;  // Schalte den Radiergummi-Modus ein oder aus
            controls.enabled = !eraserEnabled; // Sperre oder entsperre die Orbit-Steuerung
        });
    
        // Event Listener für Mausbewegung
        document.addEventListener('mousedown', function() {
            mouseDown = true;
        });
    
        // Event Listener für Mausbewegung
        document.addEventListener('mouseup', function() {
            mouseDown = false;
        });
    
        // Hole die Daten der Masken Leinwand (Canvas)
        function getMaskDataURL() {
            return maskCanvas.toDataURL("image/png");
        }
    
        // Funktion zum Aktualisieren der Textur basierend auf Benutzereingabe und Maske.
        function updateTexture(userInput) {

            // Hole die Daten-URL der aktuellen Masken-Leinwand.
            var maskDataURL = getMaskDataURL();

            // Führe eine HTTP-Anfrage aus, um die gewünschte Änderung auf der Textur durchzuführen.
            fetch('http://localhost:3000/process', {
            method: 'POST', // Verwendung der HTTP-Methode "POST"
            headers: { 'Content-Type': 'application/json' }, // Setze den Header auf JSON-Format
            // Sende die Anfrage im JSON-Format mit den Daten der Textur, Benutzereingabe und Maske.
            body: JSON.stringify({ url: textureUrl, command: userInput, mask: maskDataURL })
            })
            // Verarbeite die Antwort der vorherigen Anfrage als JSON.
            .then(response => response.json())
            .then(data => {
                // Führe eine HTTP-Anfrage aus, um die aktualisierte Textur-URL vom Server zu erhalten.
                fetch('http://localhost:3000/getImage?url=' + encodeURIComponent(data.url))
                .then(response => response.json())
                .then(data => {
                    // Lade die aktualisierte Textur mit einem TextureLoader.
                    var textureLoader = new THREE.TextureLoader();
                    textureLoader.load(data.localUrl, function(loadedTexture) {

                    // Aktualisiere die Textur und das zugehörige Material in der Szene.
                    texture = loadedTexture;
                    material.map = loadedTexture;
                    material.needsUpdate = true;
                    
                    // Setze die Masken-Leinwand zurück.
                    maskContext.fillStyle = 'white';
                    maskContext.fillRect(0, 0, maskCanvas.width, maskCanvas.height);
    
                    // Aktualisiere die Masken-Textur.
                    maskTexture.needsUpdate = true;
                    
                    // Erhalte das aktuelle Arbeitsverzeichnis.
                    const currentDirectory = process.cwd();

                    // Aktualisiere den textureUrl Pfad des neuen 360 Bild.
                    textureUrl = currentDirectory + data.localUrl;
                    });
                });
            });
        };

        // Rufe die Rendering-Funktion auf, um das Bild darzustellen.
        render();
    </script>    
</body>
</html>
```


## Express (Backend)
Dies ist der komplette Code für das Backend.

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

// Lade die Umgebungsvariablen aus der .env-Datei in das aktuelle Umgebungsobjekt.
dotenv.config();

// Erstelle eine Express-Anwendung.
const app = express();

// Verwende die CORS, um Cross-Origin Resource Sharing (CORS) zu ermöglichen.
app.use(cors());

// Verwende die Middleware, um JSON-Anfragedaten zu verarbeiten.
app.use(express.json());

// Bestimme den Verzeichnispfad der aktuellen Datei.
const __dirname = path.dirname(new URL(import.meta.url).pathname);

// Verwende das 'express.static', um statische Dateien im Pfad "/images" bereitzustellen.
app.use('/images', express.static(path.join(__dirname, 'images')));

// Konfiguriere die OpenAI-Bibliothek mit den Umgebungsvariablen für Organisation und API-Schlüssel.
const configuration = new Configuration({
    organization: process.env.OPENAI_ORGANIZATION,
    apiKey: process.env.OPENAI_API_KEY,
});

// Erstelle eine Instanz der OpenAIApi-Klasse mit der konfigurierten Instanz.
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

    } catch (error) { // Errors.

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

// Definiere eine Route, die auf GET-Anfragen reagiert und ein Bild von einer URL herunterlädt.
app.get('/getImage', (req, res) => {
    // Hole die URL-Query-Parameter aus der Anfrage.
    const url = req.query.url;

    // Bestimme den Dateinamen und Pfad, an dem das Bild gespeichert wird.
    const filename = path.join(__dirname, 'images', path.basename(url) + '.png');

    // Überprüfe, ob das Verzeichnis für den Dateipfad existiert. Wenn nicht, erstelle es rekursiv.
    if (!fs.existsSync(path.dirname(filename))) {
        fs.mkdirSync(path.dirname(filename), { recursive: true });
    }

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

// Starte den Server und höre auf Port 3000.
app.listen(3000, () => {
    console.log('Server listening on port 3000');
});
```
