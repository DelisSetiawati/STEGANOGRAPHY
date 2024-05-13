<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Image Steganography - LSB Method</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
<style>
    body {
        font-family: Arial, sans-serif;
        background-color: #f0f8ff;
        color: #333;
        margin: 0;
        padding: 0;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    h1 {
        color: #005f99;
        margin-top: 20px;
    }

    label, textarea, button {
        display: block;
        width: 80%;
        margin: 10px auto;
    }

    textarea {
        padding: 10px;
        border: 2px solid #ccc;
        border-radius: 5px;
    }

    button {
        background-color: #005f99;
        color: #fff;
        border: none;
        padding: 10px;
        border-radius: 5px;
        cursor: pointer;
    }

    button:hover {
        background-color: #004080;
    }

    #outputContainer {
        text-align: center;
        margin-top: 20px;
        width: 80%;
    }

    .outputImage {
        max-width: 100%;
        margin-bottom: 20px;
        border: 1px solid #ccc;
        border-radius: 5px;
    }

    #binaryMessage {
        margin-bottom: 10px;
        color: #005f99;
    }

    #decodedMessage {
        margin-top: 20px;
    }

    #decodeResult {
        margin-top: 20px;
        border: 1px solid #ccc;
        padding: 10px;
        width: 80%;
        margin: auto;
        background-color: #e0f7ff;
        border-radius: 5px;
    }

    #originalImageContainer, #normalizedImageContainer, #hiddenImageContainer, #decodeResult {
        display: none;
    }

    .imageContainer {
        background-color: #fff;
        padding: 10px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }

    .imageContainer h4 {
        margin: 0 0 10px 0;
        color: #333;
    }
</style>
</head>
<body>
<h1>Image Steganography - LSB Method</h1>
<label for="imageInput">Select Image:</label>
<input type="file" id="imageInput" accept="image/*">
<br><br>
<label for="messageInput">Enter Secret Message:</label>
<textarea id="messageInput" rows="4" cols="50"></textarea>
<br><br>
<button onclick="encodeMessage()">Encode Message</button>
<button onclick="decodeMessage()">Decode Message</button>
<br><br>
<div id="outputContainer">
    <h3>Output:</h3>
    <div id="originalImageContainer" style="display: none;">
        <h4>Original Image</h4>
        <img id="originalImage" class="outputImage" src="" alt="Original Image">
    </div>
    <div id="normalizedImageContainer" style="display: none;">
        <h4>Normalized Image</h4>
        <img id="normalizedImage" class="outputImage" src="" alt="Normalized Image">
        <p id="binaryMessage"></p>
    </div>
    <div id="hiddenImageContainer" style="display: none;">
        <h4>Message hidden in image (right-click save as)</h4>
        <a id="downloadLink" href="#" download="hidden_image.jpg">
            <img id="hiddenImage" class="outputImage" src="" alt="Hidden Image">
        </a>
    </div>
    <div id="decodeResult" style="display: none;">
        <h4>Decoded Message:</h4>
        <p id="decodedMessage"></p>
    </div>
</div>

<script>
function encodeMessage() {
    const imageInput = document.getElementById('imageInput');
    const messageInput = document.getElementById('messageInput');
    const binaryMessage = document.getElementById('binaryMessage');
    const originalImage = document.getElementById('originalImage');
    const normalizedImage = document.getElementById('normalizedImage');
    const hiddenImage = document.getElementById('hiddenImage');
    const downloadLink = document.getElementById('downloadLink');

    if (imageInput.files.length === 0 || messageInput.value === '') {
        alert('Please select an image and enter a message.');
        return;
    }

    const reader = new FileReader();
    reader.onload = function(event) {
        const image = new Image();
        image.onload = function() {
            originalImage.src = event.target.result;
            const canvas = document.createElement('canvas');
            canvas.width = image.width;
            canvas.height = image.height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(image, 0, 0);
            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            const data = imageData.data;
            const message = messageInput.value;
            let messageIndex = 0;
            let binaryString = '';

            for (let i = 0; i < data.length; i += 4) {
                if (messageIndex < message.length) {
                    const binaryChar = message.charCodeAt(messageIndex).toString(2).padStart(8, '0');
                    binaryString += binaryChar + ' ';
                    data[i] = (data[i] & 0xFC) | ((binaryChar[0] & 1) << 1) | (binaryChar[1] & 1);
                    data[i + 1] = (data[i + 1] & 0xFC) | ((binaryChar[2] & 1) << 1) | (binaryChar[3] & 1);
                    data[i + 2] = (data[i + 2] & 0xFC) | ((binaryChar[4] & 1) << 1) | (binaryChar[5] & 1);
                    data[i + 3] = (data[i + 3] & 0xFC) | ((binaryChar[6] & 1) << 1) | (binaryChar[7] & 1);
                    messageIndex++;
                } else {
                    break;
                }
            }

            ctx.putImageData(imageData, 0, 0);
            const normalizedDataURL = canvas.toDataURL('image/jpeg');
            normalizedImage.src = normalizedDataURL;
            binaryMessage.textContent = 'Binary Message: ' + binaryString.trim();
            
            // Set original image and hidden image
            const originalDataURL = event.target.result;
            originalImage.src = originalDataURL;
            const blob = dataURItoBlob(normalizedDataURL);
            const url = URL.createObjectURL(blob);
            hiddenImage.src = url;
            downloadLink.href = url;
            
            // Show original image, normalized image, and hidden image container
            originalImageContainer.style.display = 'block';
            normalizedImageContainer.style.display = 'block';
            hiddenImageContainer.style.display = 'block';
        };
        image.src = event.target.result;
    };
    reader.readAsDataURL(imageInput.files[0]);
}

function decodeMessage() {
    const hiddenImage = document.getElementById('hiddenImage');
    const decodedMessage = document.getElementById('decodedMessage');
    const decodeResult = document.getElementById('decodeResult');

    if (hiddenImage.src === '') {
        alert('Please encode a message first.');
        return;
    }

    const canvas = document.createElement('canvas');
    canvas.width = hiddenImage.width;
    canvas.height = hiddenImage.height;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(hiddenImage, 0, 0);
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    let decodedString = '';

    for (let i = 0; i < data.length; i += 4) {
        decodedString += String.fromCharCode(
            ((data[i] & 0x03) << 6) | ((data[i + 1] & 0x03) << 4) | ((data[i + 2] & 0x03) << 2) | (data[i + 3] & 0x03)
        );
    }

    decodedMessage.textContent = decodedString;
    decodeResult.style.display = 'block';
}

// Helper function to convert data URI to Blob
function dataURItoBlob(dataURI) {
    const byteString = atob(dataURI.split(',')[1]);
    const mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];
    const arrayBuffer = new ArrayBuffer(byteString.length);
    const uint8Array = new Uint8Array(arrayBuffer);

    for (let i = 0; i < byteString.length; i++) {
        uint8Array[i] = byteString.charCodeAt(i);
    }

    return new Blob([arrayBuffer], { type: mimeString });
}
</script>
</body>
</html>
