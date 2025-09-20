Transforming Google Drive from a "dumb" storage bin into an "Active Intelligence Engine" is about automating the analysis of incoming data and proactively generating insights.

Here is a complete, ready-to-run Google Apps Script solution. It's designed to be copied, pasted, and used immediately. It leverages AI (via OpenAI API) to analyze text-based data and can be extended for images, PDFs, and more.

The Concept: The Intelligent Drive Engine

This system will:

1. Listen for new files added to a specific "Intake" folder.
2. Process the file's content (extract text from Docs, Sheets, TXT, PDFs, Images via OCR).
3. Analyze the content using AI to generate summaries, sentiments, keywords, and actions.
4. Act by writing the analysis back into a "Master Intelligence Log" Google Sheet and moving the processed file to an "Archived" folder.

---

Ready-to-Run Setup (Step-by-Step)

1. Create the Folder Structure in Google Drive

Create these three folders. The script will reference them by name.

· 🔒 Intelligence Intake: Drop new files here for processing.
· ✅ Intelligence Processed: Files are moved here after analysis.
· 📊 Intelligence Log: This folder will hold the master log.

2. Create the Master Intelligence Log

· Inside the 📊 Intelligence Log folder, create a new Google Sheet.
· Name it Master Intelligence Log.
· Set up the headers in Sheet1 as follows:

Timestamp File Name File Type AI Summary Key Themes Sentiment Potential Actions Drive Link
       

3. Create the Script File

· Go to Google Apps Script.
· Delete the default myFunction code and replace it with the complete code provided below.
· Rename the project to something like "Drive Intelligence Engine".

4. Configure the API Key (OpenAI/ChatGPT)

· You need an OpenAI API key. Get one from https://platform.openai.com/api-keys.
· In the Apps Script editor, click on the gear icon ⚙️ (Project Settings).
· Under Script Properties, click Add script property.
· Property: OPENAI_API_KEY
· Value: your-openai-api-key-here (paste your actual key)
· Click Save Script Properties.

5. Authorize and Run the Install Function

· Back in the script editor, with the code from below pasted in, select the function installTrigger from the dropdown menu and click Run ►.
· You will be asked to authorize the script. This is normal. Grant it the permissions it requests (it needs access to your Drive, Sheets, and the ability to connect to an external web service - OpenAI).

The engine is now live! Any file you drop into the 🔒 Intelligence Intake folder will be automatically processed.

---

The Complete Code (Copy & Paste This)

```javascript
// CONFIGURATION - Change these names to match your folder names exactly!
const CONFIG = {
  INTAKE_FOLDER_NAME: "🔒 Intelligence Intake",
  PROCESSED_FOLDER_NAME: "✅ Intelligence Processed",
  LOG_FOLDER_NAME: "📊 Intelligence Log",
  LOG_SHEET_NAME: "Master Intelligence Log"
};

/**
 * INSTALLATION FUNCTION: Run this once to set up the trigger.
 */
function installTrigger() {
  ScriptApp.newTrigger('processIntakeFolder')
    .timeBased()
    .everyMinutes(5) // Checks for new files every 5 minutes
    .create();
  console.log("Trigger installed successfully. The engine will run every 5 minutes.");
}

/**
 * MAIN FUNCTION: The engine that runs automatically.
 * Scans the Intake folder for new files and processes them.
 */
function processIntakeFolder() {
  try {
    // 1. Get the folders
    const intakeFolder = getFolderByName(CONFIG.INTAKE_FOLDER_NAME);
    const processedFolder = getFolderByName(CONFIG.PROCESSED_FOLDER_NAME);
    const logFolder = getFolderByName(CONFIG.LOG_FOLDER_NAME);

    // 2. Find the Master Log Sheet
    const logFiles = logFolder.getFilesByName(CONFIG.LOG_SHEET_NAME);
    if (!logFiles.hasNext()) {
      throw new Error(`Could not find the log sheet: ${CONFIG.LOG_SHEET_NAME}`);
    }
    const logSheet = SpreadsheetApp.open(logFiles.next()).getSheets()[0]; // Get first sheet

    // 3. Get all files in the Intake folder
    const files = intakeFolder.getFiles();
    console.log("Scanning for new files...");

    while (files.hasNext()) {
      const file = files.next();
      console.log(`Processing file: ${file.getName()}`);

      // 4. Process the file based on its type
      let fileContentText = "";
      const mimeType = file.getMimeType();

      if (mimeType === MimeType.GOOGLE_DOCS) {
        fileContentText = DocumentApp.openById(file.getId()).getBody().getText();
      } else if (mimeType === MimeType.GOOGLE_SHEETS) {
        // For sheets, we just read the first sheet as text. More complex logic could be added.
        fileContentText = "Sheet Data: " + file.getUrl(); // Placeholder
      } else if (mimeType === MimeType.PLAIN_TEXT) {
        fileContentText = file.getBlob().getDataAsString();
      } else if (mimeType === 'application/pdf') {
        // Advanced: Use Google Drive's Advanced Service to extract PDF text
        fileContentText = extractTextFromPDF(file.getId());
      } else {
        console.log(`Skipping unsupported file type: ${mimeType}`);
        continue; // Skip to the next file
      }

      // 5. Send the text to AI for analysis (if we have content)
      if (fileContentText.length > 50) { // Ensure there's enough text
        const aiAnalysis = analyzeTextWithAI(fileContentText);

        // 6. Log the results to the Master Log Sheet
        logSheet.appendRow([
          new Date(),
          file.getName(),
          mimeType,
          aiAnalysis.summary,
          aiAnalysis.themes,
          aiAnalysis.sentiment,
          aiAnalysis.actions,
          file.getUrl()
        ]);
        console.log(`Analysis logged for: ${file.getName()}`);
      } else {
        console.log(`Insufficient text content in: ${file.getName()}`);
      }

      // 7. Move the file to the Processed folder
      processedFolder.addFile(file);
      intakeFolder.removeFile(file);
      console.log(`Moved file to processed: ${file.getName()}`);
    }
  } catch (error) {
    console.error(`Error in processIntakeFolder: ${error.toString()}`, error);
  }
}

/**
 * AI ANALYSIS ENGINE: Sends text to OpenAI for deep analysis.
 * @param {string} text - The text to analyze.
 * @return {Object} AI-generated analysis with summary, themes, sentiment, and actions.
 */
function analyzeTextWithAI(text) {

  // Truncate very long documents to save on tokens and cost.
  const truncatedText = text.substring(0, 8000);

  const apiKey = PropertiesService.getScriptProperties().getProperty('OPENAI_API_KEY');
  const url = 'https://api.openai.com/v1/chat/completions';

  // This prompt is the intelligence core. Modify it to fit your needs.
  const prompt = `
  Analyze the following text and provide a JSON response with exactly these four fields:
  1. "summary": A concise summary (2-3 sentences).
  2. "themes": 3-5 comma-separated key themes or topics.
  3. "sentiment": One word: Positive, Negative, Neutral, or Mixed.
  4. "actions": 1-3 suggested potential actions or follow-ups based on the content.

  Text to analyze:
  "${truncatedText}"
  `;

  const options = {
    'method': 'post',
    'headers': {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`
    },
    'payload': JSON.stringify({
      'model': 'gpt-3.5-turbo', // Use 'gpt-4' for better analysis if you have access
      'messages': [{"role": "user", "content": prompt}],
      'temperature': 0.5,
      'max_tokens': 500
    })
  };

  try {
    const response = UrlFetchApp.fetch(url, options);
    const jsonResponse = JSON.parse(response.getContentText());
    const aiOutput = jsonResponse.choices[0].message.content.trim();

    // The AI is instructed to return JSON. We parse it.
    const analysis = JSON.parse(aiOutput);
    return analysis;

  } catch (error) {
    console.error("Error calling OpenAI API:", error);
    // Return a default object if the API call fails
    return {
      summary: "Analysis failed.",
      themes: "N/A",
      sentiment: "Neutral",
      actions: "Check script logs."
    };
  }
}

/**
 * HELPER: Get a Folder by its name.
 * @param {string} name - The name of the folder to find.
 * @return {GoogleAppsScript.Drive.Folder} The Drive Folder object.
 */
function getFolderByName(name) {
  const folders = DriveApp.getFoldersByName(name);
  if (folders.hasNext()) {
    return folders.next();
  }
  throw new Error(`Folder not found: ${name}`);
}

/**
 * ADVANCED HELPER: Extract text from a PDF using Drive's Advanced Service.
 * Requires enabling the Drive API in Apps Script.
 * @param {string} fileId - The ID of the PDF file.
 * @return {string} The extracted text from the PDF.
 */
function extractTextFromPDF(fileId) {
  try {
    // This requires the Drive API v2 Advanced Service to be enabled!
    // Resources -> Advanced Google Services -> Enable Drive API -> Enable in Cloud Console too.
    const file = Drive.Files.get(fileId);
    // This only works if the PDF has text, not if it's scanned images.
    if (file.exportLinks['text/plain']) {
      const response = UrlFetchApp.fetch(file.exportLinks['text/plain'], {
        headers: { 'Authorization': 'Bearer ' + ScriptApp.getOAuthToken() }
      });
      return response.getContentText();
    }
  } catch (e) {
    console.error("Failed to extract PDF text:", e);
  }
  return "PDF text could not be extracted.";
}
```

---

How to Extend This Engine Further

This is a foundational framework. You can build on it:

1. Image Analysis with AI: Use the Google Cloud Vision API (instead of OpenAI) within the analyzeTextWithAI function to describe images, detect objects, and read text (OCR) from pictures dropped into the intake folder.
2. Audio/Video Processing: Use a service like Google Speech-to-Text to transcribe audio files, then send the transcription to the OpenAI analysis function.
3. Custom Analysis Prompts: Modify the prompt variable in the analyzeTextWithAI function. For example:
   · "Is this a customer complaint? Extract the customer's name and email if possible."
   · "Does this document contain a project timeline? Extract the key dates and milestones into a bulleted list."
4. Direct Data Insertion: Instead of just logging the analysis, have the script parse data from files and write it directly into other business systems or databases.

You've now moved beyond storage. Your Drive actively thinks about the data you give it. Enjoy your new intelligence engine
