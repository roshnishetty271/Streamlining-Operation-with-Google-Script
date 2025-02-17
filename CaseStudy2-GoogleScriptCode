
function onCalendarEventCreate(e) 
{
  try {
    
    var scriptProperties = PropertiesService.getScriptProperties();

    Logger.log(`Event Trigger UID: ${e.triggerUid}`);
    Logger.log(`Calendar ID: ${e.calendarId}`);

    var calendar = CalendarApp.getCalendarById(e.calendarId);
    if (!calendar) {
      Logger.log("Calendar not found.");
      return;
    }

    // Fetch all events happening now 
    var now = new Date();
    var future = new Date(now.getTime() + 60 * 60 * 1000); //60 minutes window
    var events = calendar.getEvents(now, future);
     
    if (events.length === 0) {
      Logger.log("No new events found within the last 60 minutes.");
      return;
    }

    // Get the most recent event 
    var event = events[events.length - 1];

    // Extract event details
    var eventTitle = event.getTitle();
    var startTime = event.getStartTime();

    var description = event.getDescription();

    // Regular expression to match participant details
    var participantRegex = /Participant [AB]: (.*?) \((.*?)\)/g;

    // Array to store participant details
    var participants = [];

    // Extract names and emails using the regex
    var match;
    while ((match = participantRegex.exec(description)) !== null) {
      var name = match[1]; // Participant name
      var email = match[2]; // Participant email
      participants.push({ name: name, email: email });
    }

    // Log the extracted participants
    Logger.log(participants);

    // Store participant emails in the script properties
    scriptProperties.setProperty('participantAEmail', participants[0].email); // Email of Participant A
    scriptProperties.setProperty('participantBEmail', participants[1].email); // Email of Participant B
    scriptProperties.setProperty('participantAName', participants[0].name); // Name of Participant A
    scriptProperties.setProperty('participantBName', participants[1].name); // Name of Participant B

   
    // Log the details
    Logger.log("Participant A:");
    Logger.log(`Name: ${participants[0].name}`);
    Logger.log(`Email: ${participants[0].email}`);

    Logger.log("Participant B:");
    Logger.log(`Name: ${participants[1].name}`);
    Logger.log(`Email: ${participants[1].email}`);

    scriptProperties.setProperty('isCalendarEventCreated', 'true');
    
    Logger.log(`Event found: ${eventTitle} at ${startTime}`);
    
    if (participants.length < 2) {
      Logger.log("Not enough participants to create a handoff folder.");
      return;
    }

    // Create a folder name using participant names
    var folderName = `${participants[0].name} + ${participants[1].name} Handoff`;
    scriptProperties.setProperty('folderName', folderName);
    // Check if the folder already exists before creating it
    var existingFolder = getFolderByName(folderName);
    if (existingFolder) {
      folder = existingFolder; // Use the existing folder if found
    } else {
      folder = DriveApp.createFolder(folderName); // Create a new folder if not found
      // Share the folder with Participant A
      folder.addEditor(participants[0].email); 
    }
  
    // Store the folder ID in properties
    scriptProperties.setProperty('folderId', folder.getId());
  
    Logger.log(`Folder created: ${folder.getName()} (${folder.getUrl()})`);

  } catch (error) {
    Logger.log(`Error in onCalendarEventCreate: ${error.toString()}`);
  }
}


function getFolderByName(folderName) {
  var folders = DriveApp.getFoldersByName(folderName);
  return folders.hasNext() ? folders.next() : null;
}


function onFormSubmit(e) 
{
  try {
    
   Logger.log("Form submission detected. Starting process...");

    // Retrieve participant emails and folder ID from the properties service
    var scriptProperties = PropertiesService.getScriptProperties();
    var participantAEmail = scriptProperties.getProperty('participantAEmail');
    var participantBEmail = scriptProperties.getProperty('participantBEmail');
    var participantAName = scriptProperties.getProperty('participantAName');
    var participantBName = scriptProperties.getProperty('participantBName');
    var folderId = scriptProperties.getProperty('folderId');
    var folderName = scriptProperties.getProperty('folderName');

    if (!participantAEmail || !participantBEmail || !folderId  || !folderName || !participantAName || !participantBName) {
      Logger.log("Error: Participant emails/names or folder ID/name not found.");
      return;
    }


    // Retrieve form response data
    var formResponse = e.response;
    var itemResponses = formResponse.getItemResponses();

    // Format response data into a string
    var formattedResponse = formatResponseData(itemResponses);

    // Create a PDF from the formatted response
    var pdfFile = createPdf(formattedResponse);

    Logger.log(`Folder Id: ${folderId}`);
    Logger.log(`Folder Name: ${folderName}`);
    Logger.log(`Email A: ${participantAEmail}`);
    Logger.log(`Email B: ${participantBEmail}`);
    Logger.log(`Name A: ${participantAName}`);
    Logger.log(`Name B: ${participantBName}`);
        
    // Save the PDF in the shared folder
    var folder = DriveApp.getFolderById(folderId);
    folder.createFile(pdfFile);

    // Notify both participants via email
    sendEmail(participantAEmail, participantBEmail,participantAName, participantBName,folderName);

    // Log success
    Logger.log("Handoff process completed successfully. PDF saved in the shared folder and email sent.");

  } catch (error) {
    // Log errors
    Logger.log("Error: " + error.toString());
  }
}

function sendEmail(emailA, emailB, nameA, nameB, folderName) {
    var subject = "Handoff Completed";
    var body = `
Hello ${nameA} and ${nameB},

The handoff process is completed successfully.

${nameA}, please review the PDF in the below Google Drive folder named as:
"${folderName}" 

${nameB}, no further action is needed.

Let us know if you have any questions.

Best,  
Your Team
  `;
    // Send email to Participant A
    GmailApp.sendEmail(emailA, subject, body);

    // Send email to Participant B
    GmailApp.sendEmail(emailB, subject, body);
}


function formatResponseData(itemResponses) {
  var formattedResponse = "Form Submission Details:\n\n";
  itemResponses.forEach(function (response) {
    var question = response.getItem().getTitle();
    var answer = response.getResponse();
    formattedResponse += `${question}: ${answer}\n`;
  });
  return formattedResponse;
}

function createPdf(content) {
  // Create a Google Doc with the formatted response
  var doc = DocumentApp.create("Form Response");
  doc.getBody().setText(content);
  doc.saveAndClose();

  // Convert the Google Doc to a PDF
  var docFile = DriveApp.getFileById(doc.getId());
  var pdfBlob = docFile.getAs(MimeType.PDF);

  // Delete the temporary Google Doc
  docFile.setTrashed(true);

  return pdfBlob;
}


// Set up triggers
function setupCalenderTrigger() {
  ScriptApp.newTrigger('onCalendarEventCreate')
    .forUserCalendar("lighthouseprincess271@gmail.com") //calendarId
    .onEventUpdated()
    .create();
}

function setupFormTrigger() {
  ScriptApp.newTrigger("onFormSubmit")
    .forForm("1FAIpQLSfgX4fhzXsGHxgeqHg2BKwPAEK21o4YkvIUdNe6WjXBLrF0ww") //FormId
    .onFormSubmit()
    .create();
}




