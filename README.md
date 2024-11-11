<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Key Verification</title>
</head>
<body>
  <script>
    // Function to encode a string with SHA-256 hash function
    function sha256Encode(str) {
      const encoder = new TextEncoder();
      const data = encoder.encode(str);
      return crypto.subtle.digest("SHA-256", data).then(hash => {
        return Array.from(new Uint8Array(hash)).map(b => b.toString(16).padStart(2, '0')).join('');
      });
    }

    // Function to fetch IPv6 address
    function fetchIPv6() {
      return fetch('https://api6.ipify.org?format=json')
        .then(response => response.json())
        .then(data => data.ip)
        .catch(error => null); // Return null if fetching fails
    }

    // Function to fetch IPv4 address
    function fetchIPv4() {
      return fetch('https://api.ipify.org?format=json')
        .then(response => response.json())
        .then(data => data.ip)
        .catch(error => null); // Return null if fetching fails
    }

    function getCurrentTimestamp() {
      return Date.now();
    }

    // Function to parse the cp cookie and return its values
    function parseCpCookie() {
      const cookies = document.cookie.split(';');
      for (let cookie of cookies) {
        const parts = cookie.trim().split('=');
        if (parts[0] === 'cp') {
          const values = parts[1].split('-');
          return { timestamp: parseInt(values[0]), count: parseInt(values[1]) };
        }
      }
      return null;
    }

    // Function to check if the user came from Linkvertise based on referer header
    function isRedirectedFromLinkvertise() {
      const referer = document.referrer;
      return referer.includes('linkvertise.com');
    }

    // Function to send the key to the specified URL
    function sendKeyToServer(key) {
      fetch('https://i-want-a-bf-2.glitch.me/keysys', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ key: key })
      })
      .then(response => response.json())
      .then(data => {
        console.log('Success:', data);
      })
      .catch(error => {
        console.error('Error:', error);
      });
    }

    // Function to create and display the key UI
    function displayKeyUI(encodedKey) {
      // Set black background
      document.body.style.backgroundColor = 'black';
      document.body.style.color = 'white';
      document.body.style.display = 'flex';
      document.body.style.flexDirection = 'column';
      document.body.style.justifyContent = 'center';
      document.body.style.alignItems = 'center';
      document.body.style.height = '100vh';
      document.body.style.margin = '0';

      // Create and append the textbox
      const textbox = document.createElement('textarea');
      textbox.value = encodedKey;
      textbox.style.width = '80%';
      textbox.style.height = '50px';
      textbox.style.marginBottom = '20px';
      document.body.appendChild(textbox);

      // Create and append the copy button
      const button = document.createElement('button');
      button.textContent = 'Copy Key';
      button.onclick = () => {
        textbox.select();
        document.execCommand('copy');
        alert('Key copied to clipboard');
      };
      document.body.appendChild(button);

      // Send the key to the server
      sendKeyToServer(encodedKey);
    }

    // Combine IPv4/IPv6 address and current timestamp into JSON format
    Promise.all([fetchIPv6(), fetchIPv4()])
      .then(([ipv6Address, ipv4Address]) => {
        const ipAddress = ipv6Address || ipv4Address; // Use IPv6 if available, otherwise IPv4
        const timestamp = getCurrentTimestamp();

        console.log("Timestamp:", timestamp);
        console.log("IP Address:", ipAddress);

        // Parse cp cookie
        const cpCookie = parseCpCookie();
        if (cpCookie) {
          const { timestamp: cpTimestamp, count } = cpCookie;
          const diff = timestamp - cpTimestamp;
          if (isRedirectedFromLinkvertise()) {
            if (count === 3) {
              // Delete cp cookie if count is 3
              document.cookie = 'cp=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
              console.log("Deleted cp cookie.");
              // Display textbox with the key
              sha256Encode(timestamp.toString()).then(encodedKey => {
                displayKeyUI(encodedKey);
              }).catch(error => {
                console.error("Error encoding key:", error);
              });
            } else {
              // Increment count and update cp cookie
              const newCount = count + 1;
              const newCpValue = `${timestamp}-${newCount}`;
              document.cookie = `cp=${newCpValue}; path=/`;
              console.log("Updated cp cookie:", newCpValue);
            }
          } else if (diff > 5 * 60 * 1000) {
            // Update cp cookie with current timestamp and count 2 if the difference exceeds 5 minutes
            const newCpValue = `${timestamp}-2`;
            document.cookie = `cp=${newCpValue}; path=/`;
            console.log("Updated cp cookie:", newCpValue);
          }
        } else {
          // Create cp cookie with initial timestamp and count 1
          const newCpValue = `${timestamp}-1`;
          document.cookie = `cp=${newCpValue}; path=/`;
          console.log("Created new cp cookie:", newCpValue);
        }

        // Redirect to the link if count is not 3 or not redirected from Linkvertise
        if ((!cpCookie || cpCookie.count !== 3) && !isRedirectedFromLinkvertise()) {
          const link = "https://link-target.net/677069/script-key-roblox";
          window.location.href = link;
        }
      })
      .catch(error => {
        console.error('Error:', error);
      });
  </script>
</body>
</html>
