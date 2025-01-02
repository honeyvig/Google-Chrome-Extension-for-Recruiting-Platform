
chrome extension developer who can build a basic extension for our recruiting platform (used by top startups) with the following features:

- When on a LinkedIn page, the extension will show the photo, name, and basic details (headline, location, etc) of the person.
- Will have the ability to select any roles that they are currently assigned to (assignment will happen within our admin)
- Will show if anyone else has already added that candidate for that role.
- Has the ability to click a button to add a candidate for a role, which will then display the candidate added within our platform (just having linkedinURL our platform already creates the profile instantly).
- Ability to show and change the stage the candidate is in, will also update dynamically based on their stage on our platform
- Display basic details of a role, this info will come from our platform
- display in extension whether the candidate has worked for any ideal companies or excluded companies (info that is on our platform)

That is the general functionality - will of course discuss more details, but you will need to have a good understanding and experience of some more advanced extension skills like:

- Working with DOM
- Implementing API calls
- Websockets
- token based authentication
- Roles and permissions
# Google-Chrome-Extension-for-Recruiting-Platform
To build a Chrome extension that integrates with a recruiting platform and works seamlessly with LinkedIn, you'll need to use a combination of front-end JavaScript, Chrome extension APIs, and possibly back-end services to handle authentication, API calls, and WebSocket communication.
Overview of Extension Functionality

The goal is to create a Chrome extension that can:

    Display LinkedIn profile details (name, photo, headline, location, etc.).
    Show current roles assigned to a candidate and allow adding them to roles.
    Display candidate's stage in the recruitment process.
    Show role information from your platform.
    Indicate if the candidate has worked for any "ideal" or "excluded" companies.

The steps to implement this functionality are as follows:
1. Chrome Extension Setup

Create the basic structure for your Chrome extension:

my-extension/
├── manifest.json
├── popup.html
├── popup.js
├── content.js
├── background.js
└── icons/
    └── icon.png

2. Manifest File (manifest.json)

The manifest.json file defines your extension’s configuration. This file specifies the extension’s name, version, permissions, background scripts, and more.

{
  "manifest_version": 3,
  "name": "Recruiting Platform Extension",
  "version": "1.0",
  "description": "Integrates with LinkedIn profiles to manage recruiting tasks.",
  "permissions": [
    "activeTab",
    "storage",
    "identity",
    "https://*.linkedin.com/*",
    "https://your-platform-api.com/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon.png",
      "48": "icons/icon.png",
      "128": "icons/icon.png"
    }
  },
  "content_scripts": [
    {
      "matches": ["https://www.linkedin.com/*"],
      "js": ["content.js"]
    }
  ],
  "host_permissions": [
    "https://your-platform-api.com/*"
  ],
  "icons": {
    "16": "icons/icon.png",
    "48": "icons/icon.png",
    "128": "icons/icon.png"
  }
}

3. Background Script (background.js)

The background script manages your extension's state and communication with your platform API. For authentication (e.g., token-based), you'll need to handle the OAuth flow or token storage.

// background.js

chrome.runtime.onInstalled.addListener(() => {
  console.log("Recruiting Platform Extension installed");
});

// API authentication token example
let authToken = null;

chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'setAuthToken') {
    authToken = request.token;
    sendResponse({ status: 'Token set successfully' });
  }
});

// Function to make API calls
async function makeApiRequest(url, method, data) {
  const response = await fetch(url, {
    method: method,
    headers: {
      'Authorization': `Bearer ${authToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  });

  const result = await response.json();
  return result;
}

4. Content Script (content.js)

The content script runs in the context of the LinkedIn page. It extracts LinkedIn profile data (name, photo, headline, etc.) and sends it to the extension’s popup for display.

// content.js

// Extract profile details from LinkedIn
function getLinkedInProfileData() {
  const name = document.querySelector('.top-card-layout__title')?.innerText || '';
  const headline = document.querySelector('.top-card-layout__headline')?.innerText || '';
  const location = document.querySelector('.top-card-layout__location')?.innerText || '';
  const photo = document.querySelector('.pv-top-card__photo')?.getAttribute('src') || '';

  return { name, headline, location, photo };
}

// Send profile data to the popup
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'getProfileData') {
    const profileData = getLinkedInProfileData();
    sendResponse({ profileData });
  }
});

5. Popup (popup.html)

The popup file is the UI for your extension. It displays the LinkedIn profile details and provides options to manage roles, stages, and more.

<!DOCTYPE html>
<html>
  <head>
    <title>Recruiting Platform</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        width: 300px;
        padding: 10px;
      }
      img {
        max-width: 50px;
        max-height: 50px;
      }
      button {
        margin-top: 10px;
      }
    </style>
  </head>
  <body>
    <h3>LinkedIn Profile</h3>
    <img id="profile-photo" src="" alt="Profile Photo">
    <p id="name"></p>
    <p id="headline"></p>
    <p id="location"></p>
    <button id="add-to-role">Add to Role</button>
    <button id="view-role-details">View Role Details</button>
    <button id="change-stage">Change Stage</button>

    <script src="popup.js"></script>
  </body>
</html>

6. Popup Script (popup.js)

The popup script retrieves LinkedIn profile details via messaging from the content script, and it allows the user to interact with the extension (add candidates, change stages, etc.).

// popup.js

// Fetch LinkedIn profile data from the content script
chrome.runtime.sendMessage({ action: 'getProfileData' }, (response) => {
  const { name, headline, location, photo } = response.profileData;

  // Display profile data in the popup
  document.getElementById('profile-photo').src = photo;
  document.getElementById('name').innerText = name;
  document.getElementById('headline').innerText = headline;
  document.getElementById('location').innerText = location;
});

// Handle button clicks
document.getElementById('add-to-role').addEventListener('click', () => {
  // Send a request to add this candidate to the platform (via API)
  const linkedinUrl = window.location.href; // Extract LinkedIn URL
  chrome.runtime.sendMessage(
    {
      action: 'addCandidateToRole',
      linkedinUrl: linkedinUrl
    },
    (response) => {
      alert(response.status);
    }
  );
});

// Handle changing the stage
document.getElementById('change-stage').addEventListener('click', () => {
  // Show a UI to change the stage (can be a prompt or dropdown)
  const newStage = prompt('Enter new stage:');
  if (newStage) {
    chrome.runtime.sendMessage(
      {
        action: 'updateStage',
        stage: newStage
      },
      (response) => {
        alert('Stage updated to ' + newStage);
      }
    );
  }
});

7. WebSocket Integration (Optional for Real-Time Updates)

To enable real-time updates (e.g., showing if someone else has added the candidate for a role), use WebSockets. Here’s an example of setting up a WebSocket connection in background.js:

// background.js

const socket = new WebSocket('wss://your-platform-api.com/updates');

socket.onopen = function() {
  console.log('WebSocket connection established');
};

socket.onmessage = function(event) {
  const data = JSON.parse(event.data);
  // Handle real-time updates like showing if the candidate was added to the role
  chrome.runtime.sendMessage({ action: 'candidateStatusUpdate', data });
};

8. API Integration

For interactions like adding candidates to roles, changing stages, and retrieving role information, you need to integrate your Chrome extension with your platform’s REST APIs.

Here’s an example of a simple API request in background.js:

async function addCandidateToRole(linkedinUrl) {
  const response = await fetch('https://your-platform-api.com/candidates', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${authToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ linkedinUrl })
  });

  const result = await response.json();
  return result;
}

9. Final Thoughts

This is a basic implementation of a Chrome extension that interacts with LinkedIn and your recruiting platform. The functionality of adding candidates, viewing roles, and changing stages will require backend API integration, WebSocket support, and proper token authentication for security.

Remember, this example provides only the basic structure. You’ll need to implement more detailed functionality, such as role management, profile synchronization, and WebSocket handling, based on your platform's specific API and authentication methods.
