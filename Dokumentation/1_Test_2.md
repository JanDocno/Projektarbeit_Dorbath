# Anwendung mit festgelegter Maske

Es wurde ein weiterer Code auf dem Weg zur fertigen Anwendung zu Übungszwecken geschrieben. Dieser Code erlaubt es einem "flachem" 360 Grad Panorama, ohne Web-GL-Render, mit einer zuvor festgelegten Maske in einer Webanwendung über die API von Dall-E 2 zu bearbeiten.

## HTML-EJS-Seite (index.ejs): 
Dies ist die Hauptseite der Anwendung und enthält die Benutzeroberfläche. Es gibt einen Abschnitt zum Hochladen eines Bildes per Drag & Drop und ein Eingabefeld für Benutzerbefehle. Das hochgeladene Bild wird im "imageContainer" angezeigt, und es gibt einen "Ausführen"-Button, um die Manipulation des Bildes zu starten.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>360 Grad Panoramabilder</title>
    <link rel="stylesheet" href="styles.css">
    <script src="script.js"></script>
</head>
<body>
    <h1>360 Grad Panorama Bildmanipulation</h1>
    <p>Bitte fügen Sie über Drag and Drop ein Bild ein.</p>

    <div id="imageContainer">
        <div id="image">
        <img id="uploadedImage" src="<% imageUrl %>" alt="Bild hier ablegen"></img>
        </div>
    </div>

    <div id="commands">
        <input type="text" id="commandInput" placeholder="Was möchten Sie verändern?...">
        <button id="inpaintingButton">Ausführen</button>
    </div>

</body>
</html>
```

## JavaScript (script.js):
Hier wird die Funktionalität der Benutzeroberfläche gesteuert. Dieses Skript ermöglicht das Hochladen von Bildern per Drag & Drop, die Anzeige des hochgeladenen Bildes und die Ausführung von Befehlen zur Bildmanipulation. Es verwendet auch die Fetch-API, um Anfragen an den Server zu senden.

```javascript
document.addEventListener('DOMContentLoaded', function() {
    const imageContainer = document.getElementById('imageContainer');
    const uploadedImage = document.getElementById('uploadedImage');
    const inpaintingButton = document.getElementById('inpaintingButton');
    const commandInput = document.getElementById('commandInput');
    
    let imageFile = null;

    imageContainer.addEventListener('dragover', event => {
        event.preventDefault();
    });

    imageContainer.addEventListener('dragleave', event => {
        event.preventDefault();
    });

    imageContainer.addEventListener('drop', event => {
        event.preventDefault();
        imageFile = event.dataTransfer.files[0];

        if (imageFile && imageFile.type.startsWith('image/')) {
            const img = new Image();
            img.src = URL.createObjectURL(imageFile);
            img.onload = () => {
                if (img.width <= 1024 && img.height <= 1024) {
                    uploadedImage.src = img.src;
                } 
                else {
                    alert('Bitte laden Sie ein Bild mit der maximalen Größe von 1024 x 1024 hoch.');
                }
            };
        } else {
            alert('Bitte laden Sie ein Bild hoch.');
        }
    });

    inpaintingButton.addEventListener('click', async () => {
        const formData = new FormData();
        formData.append('image', imageFile);
        formData.append('command', commandInput.value);

        fetch('/upload', {
            method: 'POST',
            body: formData
        })
        .then(response => {
            if (response.ok) {
                return response.json();
            } else {
                return response.json().then(err => {
                    throw new Error(err.error || 'Etwas ist schief gelaufen.');
                });
            }
        })
        .then(data => {
            if (data.imageUrl) {
                uploadedImage.src = data.imageUrl;
            } else {
                console.error('Fehler beim Generieren des Bildes.');
            }
        })
        .catch(error => {
            console.error('Fehler beim Generieren des Bildes:', error);
        });
    });
});
```
## Serverseite (app.js):
Dies ist der Servercode, der auf Node.js basiert. Er verwendet Express.js als Web-Framework. Der Server akzeptiert HTTP-Anfragen, insbesondere POST-Anfragen, um Bilder und Manipulationsbefehle entgegenzunehmen. Die Bilder werden vorübergehend gespeichert und dann an die OpenAI API gesendet, um die gewünschten Bildmanipulationen durchzuführen. Das generierte Bild wird gespeichert und dem Benutzer zur Verfügung gestellt.


```javascript
require("dotenv").config();
const express = require('express');
const multer = require('multer');
const path = require("path");
const fs = require('fs');
const axios = require('axios');
const app = express();

app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));
app.use(express.static("public"));
app.use(express.json());
app.use(express.urlencoded({extended: false}));

const { Configuration, OpenAIApi } = require("openai");
const configuration = new Configuration({
    organization: process.env.OPENAI_ORGANIZATION,
    apiKey: process.env.OPENAI_API_KEY,
});
const openai = new OpenAIApi(configuration);

const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads');
    },
    filename: (req, file, cb) => {
        cb(null, file.fieldname + '-' + Date.now() + path.extname(file.originalname));
    }
});

const upload = multer({ storage: storage });

app.get("/", (req, res) => {
    res.render("index", { imageUrl: "" });
});

app.post('/upload', upload.single("image"), async (req, res) => {
    try {
        if (!req.file) {
            throw new Error('File erforderlich');
        }

        const userInput = req.body.command;
        const uploadedFilePath = req.file.path;
        console.log(`Sending file from ${uploadedFilePath} to OpenAI API`);
       
        const response = await openai.createImageEdit(
            fs.createReadStream(uploadedFilePath),
            userInput,
            "1024x1024",
        );

        const generatedImageUrl = response.data.data[0].url;
        console.log(response);
        console.log(generatedImageUrl);

        const imageResponse = await axios.get(generatedImageUrl, { responseType: 'arraybuffer' });
        const imageSavePath = path.join(__dirname, 'public', 'images', Date.now() + '-' + 'generatedImage.png');
        fs.writeFileSync(imageSavePath, imageResponse.data);

        res.json({ 
          success: true,
          imageUrl: generatedImageUrl });

    } catch (error) {

        console.log(error);

        if (error.response && error.response.data) {
          console.log("OpenAI API Status:", error.response.status);
          console.log("OpenAI API Error:", error.response.data);

        }else {
          console.log(error.message);
        }
        
        return res.status(400).json({
            success: false,
            error: error.message || "Server Problem",
        });
    }
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
```
