# MASV Web Uploader Integration with React

This guide outlines how to integrate the MASV Web Uploader into a React application for uploading files to a MASV Portal. This integration simplifies browser-based file uploading using MASV's infrastructure.

## Overview

The MASV Web Uploader allows you to upload files from your web application directly to a MASV Portal without managing the complexities of the MASV API. This uploader provides flexibility to customize your file picker and workflows.

## Prerequisites

- **MASV Account**: [Login or Sign Up](https://massive.io/) to create a Portal.
- **Portal Subdomain**: Create a new Portal or use an existing one and note its subdomain (e.g., if your Portal domain is `example1234.portal.massive.io`, your subdomain is `example1234`).
- **React Setup**: Ensure you have an existing React project or create one using [Create React App](https://reactjs.org/docs/create-a-new-react-app.html).

## Installation

Install the MASV Web Uploader package via `yarn` or `npm`:

```bash
yarn add @masvio/uploader
```

or

```bash
npm install @masvio/uploader
```

## Integration Steps

### 1. Fetch Portal ID

Use the MASV API to fetch the Portal ID based on its subdomain:

```js
const fetchPortalID = async (subdomain) => {
  const response = await fetch(`https://api.massive.app/v1/subdomains/portals/${subdomain}`);
  const { id } = await response.json();
  return id;
};
```

### 2. Create a MASV Package

Create a package in the portal using the following function:

```js
const createPackage = async (portalID, name, sender, description) => {
  const response = await fetch(`https://api.massive.app/v1/portals/${portalID}/packages`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name, sender, description }),
  });

  const { id, access_token } = await response.json();
  return { id, access_token };
};
```

### 3. Initialize the Uploader

Once the package is created, initialize the MASV Web Uploader using the package ID and access token:

```js
import { Uploader } from "@masvio/uploader";

const initializeUploader = async (subdomain, name, sender, description) => {
  const portalID = await fetchPortalID(subdomain);
  const { id, access_token } = await createPackage(portalID, name, sender, description);
  
  const uploader = new Uploader(id, access_token, 'https://api.massive.app');
  return uploader;
};
```

### 4. Handle File Selection

Use an HTML `<input>` element to handle file selection:

```js
const handleFileChange = (event, setFiles) => {
  const selectedFiles = Array.from(event.target.files).map((file) => ({
    id: file.name,
    file,
    path: file.name,
  }));
  setFiles(selectedFiles);
};
```

### 5. Upload Files

Add files to the uploader and initiate the upload process:

```js
const handleUpload = (uploader, files) => {
  if (uploader) {
    uploader.addFiles(...files);
  }
};
```

### 6. React Component Example

Below is a complete example of how to integrate the MASV Web Uploader into a React component:

```jsx
import React, { useState, useEffect } from 'react';
import { Uploader } from '@masvio/uploader';

const Upload = () => {
  const [uploader, setUploader] = useState(null);
  const [files, setFiles] = useState([]);
  const [uploadProgress, setUploadProgress] = useState([]);
  const [name, setName] = useState('My Uploads');
  const [sender, setSender] = useState('your-email@example.com');
  const [description, setDescription] = useState('Upload Description');

  useEffect(() => {
    const initialize = async () => {
      const subdomain = 'example1234'; // Replace with your portal subdomain
      const uploaderInstance = await initializeUploader(subdomain, name, sender, description);
      setUploader(uploaderInstance);

      // Attach event handlers
      uploaderInstance.on('emit', (event) => {
        if (event.name === Uploader.UploaderEvents.Progress) {
          setUploadProgress((prev) => [...prev, { id: event.data.id, progress: event.data.progress }]);
        }
      });
    };
    initialize();
  }, []);

  return (
    <div className="max-w-4xl mx-auto p-6 bg-white shadow-lg rounded-lg">
      <h1 className="text-2xl font-semibold text-center mb-6">MASV File Upload</h1>

      <form className="mb-6 space-y-4">
        <div>
          <label className="block text-gray-700 font-medium mb-2">Package Name:</label>
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            className="w-full p-3 border border-gray-300 rounded-lg"
          />
        </div>
        <div>
          <label className="block text-gray-700 font-medium mb-2">Sender Email:</label>
          <input
            type="email"
            value={sender}
            onChange={(e) => setSender(e.target.value)}
            className="w-full p-3 border border-gray-300 rounded-lg"
          />
        </div>
        <div>
          <label className="block text-gray-700 font-medium mb-2">Description:</label>
          <textarea
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            className="w-full p-3 border border-gray-300 rounded-lg"
          />
        </div>
      </form>

      <div className="mb-4">
        <input
          type="file"
          id="fileInput"
          multiple
          onChange={(e) => handleFileChange(e, setFiles)}
          className="w-full p-3 border border-gray-300 rounded-lg"
        />
      </div>

      <button
        onClick={() => handleUpload(uploader, files)}
        className="w-full py-3 px-4 bg-green-500 text-white font-semibold rounded-lg hover:bg-green-600"
      >
        Upload
      </button>

      <div className="space-y-4">
        {uploadProgress.map((fileProgress) => (
          <div key={fileProgress.id} className="bg-gray-100 p-3 rounded-lg">
            <span className="text-gray-700 font-medium">
              {fileProgress.id}: {fileProgress.progress}%
            </span>
          </div>
        ))}
      </div>
    </div>
  );
};

export default Upload;
```

### 7. MASV Uploader API Methods

- `uploader.addFiles(...files)`: Adds files to the uploader.
- `uploader.pause()`: Pauses the upload.
- `uploader.start()`: Resumes a paused upload.
- `uploader.cancel()`: Cancels the upload.
- `uploader.finalize()`: Finalizes the upload.
- `uploader.getPerformanceStats()`: Retrieves performance statistics.

### 8. Event Handling

- **Progress**: Fired periodically during the file upload.
- **Finished**: Fired when the upload is complete.
- **Error**: Fired in case of an error during the upload.

For more information, refer to the [MASV API Documentation](https://massive.io/docs/).

---
