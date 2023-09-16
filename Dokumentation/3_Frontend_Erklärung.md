# Fronted
### HTML

In diesem Code Abschnitt werden 3 Div Elemente angelegt, diese folgende Dinge beschreiben.

1. Ein Container-Bereich, der das gerenderte 360 Grad Bild anzeigt.
- Das erste Div Element und dessen ID "viewer" wird im späteren Verlauf des Javascript Codes verwendet, um das 360 Grad Panoramabild über die Biliothek Three.js anzuzeigen.
3. Ein Container-Bereich für den Radiergummi-Button.
- Das zweite Div Element ist ein einfacher Button, um den Radiergummi/Maskierer zu aktivieren bzw. zu deaktivieren. Über die ID "eraser" kann dieses Div Element im Javascript Code verwendet werden.
4. Ein Container-Bereich für die Eingabeaufforderung und den Ausführungs-Button.
- Das dritte Div Element besteht aus einem Input Feld für den Benutzer, um die gewünschte Änderung anzugeben. Über den Button sollen die gewünschten Änderungen an die API gesendet werden.

```html
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
```

### Javascript

Das 360 Grad Panoramabild wird mit dem WebGL Renderer Three.js angezeigt. Three.js ist ideal für die Anzeige von 360 Grad Panoramabildern, da es WebGL-basierte 3D-Grafiken und interaktive Steuerelemente bietet.

Als erster Schritt muss ein Script Tag gesetzt werden um den Javascript Code einzuführen, dieser muss vom Type Module sein, da Three.js das ES6-Modulsystem verwendet. Da nur über Import Anweisungen die Module von Three.js verwendet werden können.

```javascript
    <script type="module">
        // Importe
        import * as THREE from 'three';
        import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
```

Die Initialisierung des Codes beginnt mit der Festlegung verschiedener Variablen wie textureUrl, die den Pfad zu einem 360 Grad Panorama Bild enthält, sowie eraserEnabled und mouseDown, die für die Interaktion relevant sind. Eine Szene (scene) und eine Kamera (camera) werden initialisiert.

```javascript
        // Initsalisierung
        var textureUrl = "YOUR_PANO.png"; // 360 Grad Panorama Bild.
        var eraserEnabled = false;
        var mouseDown = false;

        // Erstelle eine Scene und Kamera.
        var scene = new THREE.Scene(); 
        const width = window.innerWidth / 1.1;
        const height = window.innerHeight / 1.1;
        var camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
```

Ein WebGL-Renderer (renderer) wurde erstellt, um die Szene und die Kamera zu nutzen und das 360 Pano im Browser darzustellen. Das gerenderte Ergebnis wird in ein Canvas-Element eingefügt, das wiederum in das HTML-Dokument eingebettet wird.

```javascript
        // Erstelle einen WebGL-Renderer (Web Graphics Library), der zur Darstellung der Szene im Browser verwendet wird.
        var renderer = new THREE.WebGLRenderer(); 
        renderer.setSize(width, height);

        // Füge das gerenderte Canvas-Element des WebGL-Renderers in das HTML-Dokument ein.
        document.getElementById("viewer").appendChild(renderer.domElement);
```
 
Die Szene erfordert eine geometrische Form, die einer Kugel (geometry) entspricht. Das Material (material) enthält die Textur des Panoramas.

```javascript
        // Erstelle eine geometrische Form einer Kugel.
        var geometry = new THREE.SphereGeometry(1, 32, 32);

        // Spiegel die X-Achse der Geometrie.
        geometry.scale(-1, 1, 1);
```

Damit das 360 Panorama maskiert werden kann, wurde zur Textur zusätzlich eine Leinwand (maskCanvas) erstellt. Die Textur des Panoramas wird geladen und so konfiguriert, dass  die Leinwand/Masken Textur als Alpha Kanal verwendetwird. Das ermöglicht, Teile der Textur zu bearbeiten.

```javascript
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
```

Ein Event Listener für die Mausinteraktion verfolgt die Mausbewegungen. Wenn der inpainting Button und die Maus gedrücht ist, wird die Position der Maus berechnet.


```javascript
            // Event Listener für Mausbewegung
            renderer.domElement.addEventListener('mousemove', function(event) {
                if (eraserEnabled && mouseDown) {
                    // Konvertiere die Mausposition (-1 bis +1).
                    var mouse = new THREE.Vector2();
                    mouse.x = (event.clientX / renderer.domElement.clientWidth) * 2 - 1;
                    mouse.y = -(event.clientY / renderer.domElement.clientHeight) * 2 + 1;
```

Ein Raycaster projiziert einen Strahl von der Mausposition aus auf das 360 Pano, der Raycaster passt sich der Position der Kamera an. 

```javascript
    // Erstelle einen Raycaster, um einen Strahl vom Mauszeiger aus in das Bild zu projizieren.
    var raycaster = new THREE.Raycaster();

    // Dabei wird der Strahl wie die Kamera ausgerichtet.
    raycaster.setFromCamera(mouse, camera);    
                    
    // Führe eine Schnittprüfung (Intersection Test) zwischen dem Strahl und dem Bild durch.
    var intersects = raycaster.intersectObject(sphere);    
```

Die if-Schleife überprüft (Intersection Test), ob es Schnittpunkte zwischen dem projizierten Strahl und dem Bild gibt. Wenn Schnittpunkte vorhanden sind, wird der Code innerhalb der if-Schleife ausgeführt und das Maskieren durchgeführt.

```javascript
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
```

Im nächsten Abschnitt wird das 360 Grad Panorama Bild und sein Material erstellt. Das Material verwendet die Textur. Außerdem wird die Textur von beiden Seiten der Kugel angezeigt, damit beim Maskieren die Kugel vollständig freigelegt wird. Dann wird die Sphäre (Geometrie) zur Szene hinzugefügt. Die Kamera wird auf eine Ausgangsposition festgelegt, und Orbit-Steuerungen ermöglichen es, die Kamera im 3D-Raum zu bewegen.

```javascript
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
```

Die Funktion render sorgt dafür, dass die Szene und die Kamera in einer Schleife aktualisiert werden, um die  Darstellung des 360 Grad Panorama Bild zu gewährleisten.

```javascript
        // Funktion zum Rendern des 360 Grad Panorama Bild, die in einer Schleife aufgerufen wird.
        function render() {
            requestAnimationFrame(render);
            renderer.render(scene, camera);
            controls.update(); // Aktualisiere die Kamerasteuerung.
        }
```

Ein Event Listener für Buttons wird hinzugefügt, um Benutzereingaben zu verarbeiten. Der erste Button (inpaintingButton) erfasst Benutzereingaben und aktualisiert die Textur basierend auf der Eingabe und der Maske. Der zweite Button (eraserButton) aktiviert oder deaktiviert den Radiergummi-Modus sowie die Orbit-Steuerung.

```javascript
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
```

Mithilfe von Event Listener für Mausaktionen wird der Status von mouseDown verfolgt, was für die Radiergummi-Interaktion wesentlich ist.

```javascript
        // Event Listener für Mausbewegung
        document.addEventListener('mousedown', function() {
            mouseDown = true;
        });
    
        // Event Listener für Mausbewegung
        document.addEventListener('mouseup', function() {
            mouseDown = false;
        });
```

Die Funktion getMaskURL dient dazu, die Daten-URL der aktuellen Masken-Leinwand (Canvas) zu generieren. Sie ermöglicht es, die aktuelle Darstellung der Maske als Bild im PNG-Format zu erhalten.

```javascript
        // Hole die Daten der Masken Leinwand (Canvas)
        function getMaskDataURL() {
            return maskCanvas.toDataURL("image/png");
        }
```

Die Funktion updateTexture aktualisiert die Textur des 3D-Modells basierend auf Benutzereingaben. Zuerst wird die Daten-URL der aktuellen Masken-Leinwand abgerufen. Dann erfolgt eine HTTP-Anfrage an einen Server (http://localhost:3000/process), um die gewünschte Texturänderung durchzuführen. Dabei werden die Textur-URL, Benutzereingabe und die Masken-Daten als JSON übermittelt. 

```javascript
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
```

Nachdem die Serverantwort verarbeitet wurde, erfolgt eine HTTP-Anfrage an die Adresse http://localhost:3000/getImage?url=. Diese Anfrage enthält als Parameter die URL der bearbeiteten Textur von OpenAI, die zuvor von der Serverantwort empfangen wurde. Diese encodeURIComponent-Funktion wandelt bestimmte Zeichen in ihrer URL-codierten Form um, um sicherzustellen, dass sie sicher über das Internet übertragen werden können. Zum Beispiel werden Leerzeichen zu %20 und Sonderzeichen wie & oder ? werden entsprechend codiert. Dies gewährleistet die korrekte Formatierung und Sicherheit der URL in der HTTP-Anfrage.

```javascript
            .then(data => {
                // Führe eine HTTP-Anfrage aus, um die aktualisierte Textur-URL vom Server zu erhalten.
                fetch('http://localhost:3000/getImage?url=' + encodeURIComponent(data.url))
                .then(response => response.json())
```

Die Masken-Leinwand wird zurückgesetzt, um sicherzustellen, dass weitere Bearbeitungen mit einer neuen Leinwand beginnen können. Schließlich wird auch die Textur-URL aktualisiert, um sicherzustellen, dass zukünftige Anfragen die aktuelle Version der Textur verwenden. Dieser Prozess ermöglicht es dem 360 Grad Panoramabild, seine Textur dynamisch zu ändern und auf die Benutzereingaben zu reagieren.

```javascript
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
    
                    // Erhalte das aktuelle Arbeitsverzeichnis
                    const currentDirectory = process.cwd();

                    // Verwende es, um den Pfad zu aktualisieren
                    textureUrl = currentDirectory + data.localUrl;
    
                    });
                });
            });
        };
```

Schließlich wird die render-Funktion aufgerufen, um die 3D-Szene im Browser darzustellen. Dieser Code ermöglicht die Erstellung eines interaktiven 3D-Panoramas im Web, das dynamisch bearbeitet werden kann.

```javascript
      // Rufe die Rendering-Funktion auf, um die 3D-Szene darzustellen.
        render();
```
