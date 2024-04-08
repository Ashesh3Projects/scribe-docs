---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Scribe Integration

This section of the documentation details the integration of the Care Scribe feature into the Care application backend, describing the functionalities, database migrations, API endpoints, and background job scheduling.

## Modules

The Care Scribe feature is composed of several key components that work together to enable voice-assisted form filling. Below we will detail each module's functionality and how they interact within the system.

**Frontend Voice Recorder with Audio Size Splitting**

The voice recorder is a critical component that manages the audio input from the user. It uses the `useSegmentedRecorder` custom hook to handle the recording process. This hook facilitates the splitting of audio data into manageable chunks to prevent file size from becoming too large, which is essential for efficient data transfer and processing.

```jsx
// Example usage of useSegmentedRecorder hook
const {
  isRecording,
  startRecording,
  stopRecording,
  audioBlobs,
} = useSegmentedRecorder();
```

When the audio reaches a predefined size limit or time duration, the recorder stops the current recording session and starts a new one, ensuring seamless user experience and data integrity.

**Uploading Files to S3**

Once the audio is recorded, the files are prepared for upload. This process involves creating a signed URL for secure file transfer and then uploading the audio data to an Amazon S3 bucket. This is handled within the `Scribe` component, which interacts with backend API routes designed for file management.

```jsx
// Example of the upload function within the Scribe component
const uploadAudio = async (audioBlob, associatingId) => {
  // ...code to handle file upload to S3...
};
```

**Backend Transcription and Data Extraction Task**

On the backend, a task is triggered to process the uploaded audio files. This involves several steps:

1. **Fetching Audio Files**: Retrieving the audio files from S3.
2. **Transcription**: Using OpenAI's Whisper to convert speech to text.
3. **Data Extraction**: Applying GPT-4 to interpret the transcript and extract structured form data.
4. **Task Completion**: Updating the `Scribe` object's status to 'Completed' and storing the extracted form data.

These steps are handled by a background worker task that ensures the process is asynchronous and does not block other operations.

**Frontend Form Filling**

After the backend processing is complete, the frontend `Scribe` component receives the structured form data. It then updates the relevant form fields accordingly, providing a smooth user experience. The component listens for changes in the `Scribe` object's status and reacts by filling in the form fields when the data becomes available.

```jsx
// Example callback function to update form state
onFormUpdate={(fields) => {
  // Update form state with the new data
}};
```

**Module Interaction**

The frontend voice recorder module starts the process by capturing the user's voice and splitting the audio as needed. It then interacts with the S3 upload module to transfer the files securely. Once uploaded, the backend task takes over to transcribe and extract data from the audio files, utilizing powerful AI models. Finally, the structured form data is sent back to the frontend, where the form-filling module populates the form fields with the extracted data, completing the cycle.

By understanding the functionalities and interactions of these modules, developers can maintain, troubleshoot, and extend the Care Scribe feature as required.

## Functionalities

The Care Scribe module adds voice recognition and form autofill capabilities to the Care application. The key functionalities include:

* **Voice Recording Management**: Handling the start and stop of voice recordings, managing file buffers, and pushing audio data to storage.
* **Form Data Processing**: Collecting and structuring form data, validating it against a JSON schema, and preparing it for AI processing.
* **AI Interaction**: Integrating with AI services for transcribing audio to text and generating AI responses to fill the form data.

Key components added with this feature include:

* `Scribe`: A Django model that represents the form filling session and stores the transcription, AI response, and the status of the processing.
* `ScribeFile`: Extends the `FileUpload` model to handle specific file types associated with the Scribe feature.
* `ScribeSerializer`: Handles the serialization and deserialization of Scribe data for RESTful interaction.

## Database Migrations

The integration includes migrations to create new models in the database:

* `0001_initial.py`: This migration sets up the initial structure of the `Scribe` and `ScribeFile` models in the database.

## API Endpoints

New endpoints are added to manage the Scribe feature:

* `/api/scribe/`: Endpoints for creating, retrieving, and updating `Scribe` objects.
* `/api/scribe_file/`: Endpoints for managing the file uploads related to the `Scribe` objects.

These endpoints allow the frontend to start a form filling session, manage audio file uploads, and fetch the status and results of the AI processing.

## Background Jobs and Scheduling

To process the audio recordings asynchronously and update the form data with AI-generated responses, Care Scribe leverages Celery, a distributed task queue:

* `tasks/scribe.py`: Defines the Celery tasks for processing form fillings, such as `process_ai_form_fill`, which coordinates the AI transcription and response generation.

These tasks are queued up when a `Scribe` object's status is set to 'Ready' and are processed in the background without blocking the main application flow.

### Example Celery Task

Here's an example of a Celery task that orchestrates the AI form filling process:

```python
@shared_task
def process_ai_form_fill(external_id):
    # Fetch the Scribe object and associated audio files
    # Generate transcript using AI service
    # Process transcript with AI to fill form
    # Update Scribe object with the AI response and set status to 'Completed'
```

The task is designed to handle various steps in the AI form filling process, including error handling to update the `Scribe` status to 'Failed' if any step encounters an issue.
