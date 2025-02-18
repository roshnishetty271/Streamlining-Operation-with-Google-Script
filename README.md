# Streamlining-Operation-with-Google-Script

Implementation Guide:

 1. Key Features
    
• Detects newly created Google Calendar events.
• Extracts participant names and emails from the event description.
• Creates a Google Drive folder and shares the folder with participant A.
• Detects Google Form submission by participant B and saves the form responses as PDF in the folder.
• Sends email notifications to both the participants.

 2. Setup Instructions
    
A. Add the Script to Google Apps Script
1. Go to Google Apps Script.
2. Click New Project and add the script.
3. Replace Calendar ID in Calendar Event Trigger and Form ID in Form Submission Trigger with your actual IDs.
4. Save the script.
   
B. Set Up Triggers for Automation
• Calendar Event Trigger → Run setupCalendarTrigger() once to detect new events.
• Form Submission Trigger → Run setupFormTrigger() once to process form responses.

 3. How the Script Works
    
A. Calendar Event Handling (onCalendarEventCreate(e))
• Extracts participant details from event description.
• Creates a Google Drive folder named "[Participant A's name + Participant B'sname Handoff]” and shares the folder with participant A.
• Stores participant and folder details using PropertiesService.

Example: Event Description Format

  Participant A: Roshni Shetty (roshni99.shetty@gmail.com)
  
  Participant B: Roshan Shetty (roshnishetty.634@gmail.com)

B. Form Submission Handling (onFormSubmit(e))
• Fetches stored participant and folder details using PropertiesService.
• Converts Google form responses into a PDF submitted by participant B.
• Saves the PDF in the folder shared with participant A.
• Sends email notifications to both the participants.

 4. Running the Script
    
   Run setupCalendarTrigger() and setupFormTrigger() once.
   Create a Google Calendar event with participant details.
   Check Google Drive of participant A for the created folder.
   Submit a Google Form by participant B and verify the saved PDF in the folder shared with participant A.
   Confirm email notifications in both the participants inbox.
   
 5. Logging & Debugging
    
• Go to Apps Script → Executions to check logs.
