# MASV File Upload Application

This project is a React-based file uploader that integrates with the MASV API to allow users to upload files to a MASV portal. It utilizes the MASV Web Uploader to simplify the upload process, providing an intuitive interface for users to upload multiple files with progress tracking, and customize package details like name, sender email, and description.

## Table of Contents

- [Features](#features)
- [Technologies Used](#technologies-used)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [MASV Web Uploader Integration](#masv-web-uploader-integration)
- [API Overview](#api-overview)
- [Customization](#customization)
- [Known Issues](#known-issues)
- [License](#license)

## Features

- **Dynamic Form Inputs**: Users can provide package details including name, sender email, and description.
- **Multi-File Upload**: Users can select and upload multiple files at once, with real-time progress updates for each file.
- **MASV API Integration**: Integrates with the MASV API for portal and package management.
- **Upload Progress Tracking**: Displays the upload progress of each file in real-time.

## Technologies Used

- **React**: The front-end library for building the user interface and managing application state.
- **MASV Web Uploader (`@masvio/uploader`)**: A JavaScript library that simplifies file uploads to MASV.
- **Tailwind CSS**: A utility-first CSS framework for styling the interface.

## Requirements

To run this application, you will need the following:
- **Node.js** and **npm** (or **Yarn**) installed on your system.
- A **MASV account** with access to create and manage portals.
- MASV portal subdomain and access tokens.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/masv-upload-app.git
   cd masv-upload-app
   ```

2. Install the required dependencies:
   ```bash
   npm install
   ```

3. Install the MASV Web Uploader:
   ```bash
   yarn add @masvio/uploader
   ```

4. Update the MASV subdomain in the `initializeUploader` function:
   In `src/Upload.js`, update the `subdomain` variable with your MASV portal subdomain:
   ```javascript
   const subdomain = 'your-subdomain';
   ```

## Usage

1. **Start the development server:**
   ```bash
   npm start
   ```

2. Open your browser and navigate to `http://localhost:3000` to access the application.

3. **Create a Package**:
   - Fill out the form with the package name, sender email, and description.
   - Click the "Update Package Info" button to create a new MASV package.

4. **Upload Files**:
   - Use the file picker to select files to upload.
   - Click "Upload" to begin the file upload process.
   - The progress of each file upload will be displayed in real-time.

## MASV Web Uploader Integration

The MASV Web Uploader simplifies file uploads by abstracting away the complexities of interacting directly with the MASV API. The application provides a flexible way to integrate your own file picker while maintaining control over upload workflows.

### Setting Up

1. **Install the MASV Web Uploader**:
   ```bash
   yarn add @masvio/uploader
   ```

2. **Get the Portal Subdomain**:
   - Sign in to your MASV account.
   - Create a new portal or use an existing one.
   - Obtain the portal's subdomain (e.g., `example1234` from `example1234.portal.massive.io`).

### Fetch Portal ID

This helper function fetches the portal ID using the portal subdomain:

```javascript
async function fetchPortalID(subdomain) {
    const response = await fetch(`https://api.massive.app/v1/subdomains/portals/${subdomain}`);
    const { id } = await response.json();
    return id;
}
```

### Create a Package

This function creates a MASV package using the portal ID and returns the package ID and access token:

```javascript
async function createPackage(portalID) {
    const response = await fetch(`https://api.massive.app/v1/portals/${portalID}/packages`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            name: 'Package Name',
            sender: 'your-email@example.com',
            description: 'Description of the package'
        })
    });

    const { id, access_token } = await response.json();
    return { id, access_token };
}
```

### Initialize the Uploader

Instantiate the MASV Web Uploader by passing the package ID, access token, and MASV API URL:

```javascript
import { Uploader } from '@masvio/uploader';

const portalID = await fetchPortalID('your-subdomain');
const { id, access_token } = await createPackage(portalID);

const uploader = new Uploader(id, access_token, 'https://api.massive.app');
```

### Adding Files to the Uploader

You can use an HTML `<input>` element with type `file` to retrieve the files to upload:

```javascript
<input type="file" id="fileInput" multiple />
const fileInput = document.getElementById('fileInput');
let files = [];

fileInput.addEventListener('input', (event) => {
    for (let file of fileInput.files) {
        files.push({ id: file.name, file, path: file.name });
    }
});
```

Add these files to the uploader and start the upload process:

```javascript
uploader.addFiles(...files);
```

## API Overview

### Constructor
```javascript
new Uploader(packageID, packageToken, apiURL);
```
- **packageID**: The ID of the package (obtained during package creation).
- **packageToken**: The JWT access token for the package.
- **apiURL**: Base URL of the MASV API (e.g., `https://api.massive.app`).

### Methods

- **addFiles**: Upload one or multiple files to the package.
- **pause**: Pause the upload.
- **start**: Resume or start the upload.
- **cancel**: Cancel the upload.
- **terminate**: Terminate the uploader and stop all processes.

### Events

Listen to various uploader events to track progress and handle errors:

```javascript
uploader.on(Uploader.UploaderEvents.Progress, (event) => {
    console.log(event.data.progress);
});

uploader.on(Uploader.UploaderEvents.Finished, (event) => {
    console.log('Upload finished', event.data);
});
```

## Customization

- Modify the API requests to suit different MASV endpoints if needed.
- Add custom form fields or metadata to the package creation process.
- Enhance the error handling to better manage failed uploads or API errors.

## Known Issues

- **Error Handling**: The current implementation lacks comprehensive error handling. Consider adding more robust handling for API request failures or upload issues.
- **Performance Monitoring**: Performance stats are not currently displayed but can be added using the `getPerformanceStats` method.

## License

This project is licensed under the MIT License.

Feel free to contribute by submitting issues or pull requests!
