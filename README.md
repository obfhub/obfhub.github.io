<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script
      type="module"
      src="https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.esm.js"
    ></script>
    <script
      nomodule
      src="https://unpkg.com/ionicons@5.5.2/dist/ionicons/ionicons.js"
    ></script>
    <link rel="stylesheet" href="style.css" />
    <title>Language Translater</title>
  </head>
  <body>
    <div class="mode">
      <label class="toggle" for="dark-mode-btn">
        <div class="toggle-track">
          <input type="checkbox" class="toggle-checkbox" id="dark-mode-btn" />
          <span class="toggle-thumb"></span>
          <img src="images/sun.png" alt="" />
          <img src="images/moon.png" alt="" />
        </div>
      </label>
    </div>
    <div class="container">
      <div class="card input-wrapper">
        <div class="from">
          <span class="heading">From :</span>
          <div class="dropdown-container" id="input-language">
            <div class="dropdown-toggle">
              <ion-icon name="globe-outline"></ion-icon>
              <span class="selected" data-value="auto">Auto Detect</span>
              <ion-icon name="chevron-down-outline"></ion-icon>
            </div>
            <ul class="dropdown-menu">
              <li class="option active">DropDown Menu Item 1</li>
              <li class="option">DropDown Menu Item 2</li>
            </ul>
          </div>
        </div>
        <div class="text-area">
          <textarea
            id="input-text"
            cols="30"
            rows="10"
            placeholder="Enter your text here"
          ></textarea>
          <div class="chars"><span id="input-chars">0</span> / 5000</div>
        </div>
        <div class="card-bottom">
          <p>Or choose your document!</p>
          <label for="upload-document">
            <span id="upload-title">Choose File</span>
            <ion-icon name="cloud-upload-outline"></ion-icon>
            <input type="file" id="upload-document" hidden />
          </label>
          <button id="mic-btn">
            <ion-icon name="mic-outline"></ion-icon>
            <span>Speak</span>
          </button>
        </div>
      </div>

      <div class="center">
        <div class="swap-position">
          <ion-icon name="swap-horizontal-outline"></ion-icon>
        </div>
      </div>

      <div class="card output-wrapper">
        <div class="to">
          <span class="heading">To :</span>
          <div class="dropdown-container" id="output-language">
            <div class="dropdown-toggle">
              <ion-icon name="globe-outline"></ion-icon>
              <span class="selected" data-value="en">Englsih</span>
              <ion-icon name="chevron-down-outline"></ion-icon>
            </div>
            <ul class="dropdown-menu">
              <li class="option active">DropDown Menu Item 1</li>
              <li class="option">DropDown Menu Item 2</li>
            </ul>
          </div>
        </div>
        <textarea
          id="output-text"
          cols="30"
          rows="10"
          placeholder="Translated text will appear here"
          disabled
        ></textarea>
        <div class="card-bottom">
          <p>Download as a document!</p>
          <button id="download-btn">
            <span>Download</span>
            <ion-icon name="cloud-download-outline"></ion-icon>
          </button>
          <button id="speak-btn">
            <ion-icon name="volume-high-outline"></ion-icon>
            <span>Listen</span>
          </button>
        </div>
      </div>
    </div>

    <script src="languages.js"></script>
    <script src="script.js"></script>
    <script>
      // Text-to-Speech
      document.getElementById("speak-btn").addEventListener("click", () => {
        const text = document.getElementById("output-text").value;
        if (!text.trim()) {
          alert("No text to read!");
          return;
        }
        const utterance = new SpeechSynthesisUtterance(text);
        utterance.lang =
          document.querySelector("#output-language .selected").dataset.value ||
          "en";
        window.speechSynthesis.speak(utterance);
      });

      // Speech-to-Text
      document.getElementById("mic-btn").addEventListener("click", () => {
        if (!("webkitSpeechRecognition" in window)) {
          alert("Speech recognition is not supported in this browser.");
          return;
        }

        const recognition = new webkitSpeechRecognition();
        recognition.lang =
          document.querySelector("#input-language .selected").dataset.value ||
          "en";
        recognition.interimResults = false;
        recognition.maxAlternatives = 1;

        recognition.onstart = () => {
          document.getElementById("mic-btn").innerHTML = "Listening...";
        };

        recognition.onerror = (event) => {
          alert("Speech recognition error: " + event.error);
          document.getElementById("mic-btn").innerHTML = "Speak";
        };

        recognition.onend = () => {
          document.getElementById("mic-btn").innerHTML = "Speak";
        };

        recognition.onresult = (event) => {
          const transcript = event.results[0][0].transcript;
          document.getElementById("input-text").value = transcript;
        };

        recognition.start();
      });
    </script>
  </body>
</html>
