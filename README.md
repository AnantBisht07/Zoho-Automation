# Zoho Sign + Google Sheets Automation Workflow

## What This Does

I built this n8n workflow to completely automate our student enrollment process. Before this, our team was manually sending Zoho Sign documents, copying data into 5 different Google Sheets, and constantly checking if students had signed their agreements. It was tedious and error-prone.

Now? A student fills out one form, and everything happens automatically - the right agreement gets sent, data goes to the correct sheet, and we even have scheduled checks that update signature statuses every hour.

## The Problem I Was Solving

Our admissions team was drowning in manual work:
- Copy-pasting student info into different Google Sheets for each course
- Manually creating Zoho Sign requests for every single student
- Checking Zoho every few hours to see who had signed
- Updating sheets again once signatures came in
- Dealing with data entry mistakes

For 250+ students per week, this was taking days of someone's time.

## How It Works

### The Main Flow

1. **Student submits the enrollment form** - I built a custom form that collects all the info we need upfront
2. **System figures out which course** - Based on their course selection, it routes to the right workflow branch
3. **Pulls the correct Zoho template** - Each course has its own agreement template
4. **Creates and sends the signature request** - Automatically fills in student details and sends via Zoho Sign
5. **Saves everything to Google Sheets** - Writes to the course-specific sheet tab with all the enrollment data
6. **Monitors signature status** - Background jobs run hourly to check if students have signed
7. **Updates completion** - When someone signs, it automatically updates the sheet with timestamp and document link

## Tech Stack

Built entirely in **n8n** using:
- **Zoho Sign API** for document management
- **Google Sheets API** for data storage
- **HTTP Request nodes** for all API calls
- **Schedule Triggers** for the hourly status checks
- **Form Trigger** for the custom enrollment form

Authentication is handled through OAuth2 for both Zoho and Google.

## What I Learned Building This

### Working with APIs
The Zoho Sign API is a bit finicky. You have to:
- First GET the template to grab the action_id
- Then POST with form-encoded data (not JSON!)
- Handle their specific field naming conventions
- Deal with their rate limits

I ended up adding error handling nodes everywhere because API calls can fail for random reasons.

### Batch Processing
The status monitoring piece was interesting. I used Split in Batches to loop through all "in progress" rows, check each one's status in Zoho, then update the sheet if it's complete. Had to be careful about the loop logic to avoid infinite loops.

### Data Mapping
Each course needs different Google Sheet columns and platform assignments (LMS field, course platform field). I used Set nodes to transform the form data into the exact format each sheet expects.

### Error Logging
Added a dedicated "Errors" sheet that catches any failures - which template failed, what the error was, which student it was for. Super helpful for debugging.

## The Results

This has been running in production for a few months now:
- Went from ~6-7 hours of manual work per day to basically zero
- No more data entry mistakes
- Students get their agreements within seconds instead of hours
- Team can see signature status in real-time
- Complete audit trail of everything

The automation handles about 250 students per week without any issues.

## Some Technical Details

### Form Setup
I customized the n8n form with CSS to match our branding. It's got validation, required fields, dropdowns for consistency - pretty standard stuff but it works well.

### The Switch Node
This is the heart of the routing logic. It evaluates the course name and sends the data down one of 5 branches. Each branch:
- Fetches the right Zoho template
- Maps data to course-specific fields  
- Updates the correct Google Sheet tab

### Schedule Triggers
Four separate schedule triggers run every hour:
- One for each sheet (Agentic Standard, Data Analytics, EADP, Senior Professionals)
- Filters for rows where Status = "inprogress"
- Calls Zoho API to check current status
- Updates if status changed to "completed"

### Error Handling
Almost every API node has "Continue On Error" enabled, with the error output piped to an error logging node. This way if Zoho times out or Google Sheets throws an error, the workflow doesn't just die - it logs what happened and keeps going.

## If You Want to Build Something Similar

Key things to know:

1. **Zoho Sign API quirks**: Use form-encoded data, not JSON. Get template details first to grab field IDs.

2. **Google Sheets**: Use "Append or Update" operation with a unique ID (I use RequestId) to avoid duplicates.

3. **Schedule Triggers**: Be careful with infinite loops. Always have a clear exit condition.

4. **Error handling**: Log everything to a sheet. You'll thank yourself later.

5. **Testing**: Test each branch independently before connecting everything.

## What's Next

Some ideas I haven't implemented yet:
- Send Slack notifications when errors happen
- Add reminder emails for unsigned agreements
- Build a simple dashboard showing enrollment metrics
- Integrate with our CRM to close the loop

## Running This

You'll need:
- An n8n instance (I'm using self-hosted)
- Zoho Sign account with API credentials
- Google Cloud project with Sheets API enabled
- OAuth2 setup for both services

The workflow itself is pretty plug-and-play once you have the credentials configured.

---
By Anant Bisht - AI Software Engineer
