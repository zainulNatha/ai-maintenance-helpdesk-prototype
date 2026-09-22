# Google Form Integration

## Overview

I wanted the maintenance help desk to support more than one way of reporting a fault.

The AI phone workflow was the main route, but I also wanted someone to be able to submit the same type of request through a simple online form.

Rather than building a completely separate ticketing process for the form, I designed it so that both routes eventually send the same type of structured maintenance data into the same Make.com workflow.

The overall process is:

```text
Google Form submitted
        ↓
Google Apps Script runs automatically
        ↓
Form answers are read
        ↓
Selected site is checked against the Sites table
        ↓
Official site information is added
        ↓
A structured JSON payload is created
        ↓
Payload is sent to the Make webhook
        ↓
Existing ticket workflow runs
        ↓
Ticket is stored and emails are sent
```

This means the downstream ticketing workflow does not need to know whether the maintenance request originally came from a phone call or a form.

---

# Why I Used Google Apps Script

Google Forms can automatically write responses into Google Sheets.

However, I needed more than simply storing the form answers.

The form only asks the user for information they should reasonably know, such as:

- which site they are reporting from
- which system is affected
- what the problem is
- their name
- their callback telephone number

It does not ask the user for internal information such as:

- Site ID
- customer email address
- internal customer record information

That information is already stored in the Sites table.

Google Apps Script acts as the bridge between the submitted form and the automation workflow.

Its job is to:

```text
1. Read the submitted form response

2. Find the selected site in the Sites table

3. Retrieve the official site information

4. Combine the form answers with the stored site information

5. Build a structured payload

6. Send the payload to Make.com
```

---

# Form Structure

The form used the following fields:

| Position | Form field | Required? |
|---|---|---|
| A | Timestamp | Automatically created |
| B | Which Site? | Yes |
| C | Which System affected? | Yes |
| D | Please describe the maintenance issue | Yes |
| E | Fault or error message | No |
| F | How is the issue affecting the site? | No |
| G | Your name | Yes |
| H | Best callback telephone number | Yes |

A key decision was not to ask the customer for their email address.

The customer email is retrieved from the maintained Sites table instead.

This means the workflow uses the business's stored customer information rather than relying on someone entering an email address correctly every time.

---

# Sites Table

The prototype Sites table is structured like this:

| Column | Field |
|---|---|
| A | Site ID |
| B | Customer / Organisation |
| C | Branch |
| D | Site Display Name |
| E | Telephone |
| F | Customer Email |
| G | Address |

For public examples, fictional values can be used:

| Site ID | Customer | Branch | Site Display Name | Customer Email |
|---|---|---|---|---|
| SITE-001 | Demo Academy | North | Demo Academy North | customer@example.com |
| SITE-002 | Demo Academy | South | Demo Academy South | customer@example.com |

The Site Display Name in the form matches the Site Display Name stored in this table.

---

# Apps Script

The public version of the script below uses a placeholder webhook URL.

The real webhook URL is intentionally not stored in this repository.

```javascript
function onFormSubmit(e) {

  // GOOGLE FORM COLUMNS
  // A = Timestamp
  // B = Which Site?
  // C = Which System affected?
  // D = Please describe the maintenance issue
  // E = Fault or error message
  // F = How is the issue affecting the site?
  // G = Your name
  // H = Best callback telephone number

  const values = e.values;

  const siteDisplayName = values[1];
  const systemType = values[2];
  const reportedFault = values[3];
  const faultMessage = values[4] || 'None reported';
  const siteImpact = values[5] || '';
  const callerName = values[6];
  const callbackPhone = values[7];

  const spreadsheet = e.source;

  const sitesSheet = spreadsheet.getSheetByName('Sites');

  if (!sitesSheet) {
    throw new Error('Sites sheet could not be found.');
  }

  const lastRow = sitesSheet.getLastRow();

  if (lastRow < 2) {
    throw new Error('No site records were found in the Sites sheet.');
  }

  // SITES TABLE COLUMNS
  // A = Site ID
  // B = Customer / Organisation
  // C = Branch
  // D = Site Display Name
  // E = Telephone
  // F = Customer Email
  // G = Address

  const siteData = sitesSheet
    .getRange(2, 1, lastRow - 1, 7)
    .getDisplayValues();

  const matchedSite = siteData.find(function(row) {
    return row[3].trim().toLowerCase() ===
           siteDisplayName.trim().toLowerCase();
  });

  if (!matchedSite) {
    throw new Error('Site not found: ' + siteDisplayName);
  }

  const siteId = matchedSite[0];
  const customerName = matchedSite[1];
  const officialSiteName = matchedSite[3];
  const customerEmail = matchedSite[5];

  const payload = {
    site_id: siteId,
    customer_name: customerName,
    site_name: officialSiteName,
    customer_email: customerEmail,

    system_type: systemType,
    reported_fault: reportedFault,
    fault_message: faultMessage,
    site_impact: siteImpact,

    caller_name: callerName,

    use_calling_number: 'false',
    incoming_caller_number: '',
    alternate_callback_phone: callbackPhone
  };

  const webhookUrl = 'YOUR_MAKE_WEBHOOK_URL';

  const response = UrlFetchApp.fetch(webhookUrl, {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  });

  const statusCode = response.getResponseCode();
  const responseText = response.getContentText();

  console.log('Make status code: ' + statusCode);
  console.log('Make response: ' + responseText);

  if (statusCode < 200 || statusCode >= 300) {
    throw new Error(
      'Make returned HTTP ' +
      statusCode +
      ': ' +
      responseText
    );
  }

  console.log('Ticket successfully sent to Make.');
}
```

---

# Understanding the Script

## 1. The Function

The script starts with:

```javascript
function onFormSubmit(e) {
```

A **function** is a group of instructions that performs a particular task.

This function is called:

```text
onFormSubmit
```

Its job is to handle a newly submitted form response.

The `e` stands for the **event object**.

The event object contains information about what has just happened.

In this case, it contains information about the new row created by the Google Form submission.

A simple way to think about it is:

```text
Form submission happens
        ↓
Google creates event information
        ↓
That event information is passed into the function as "e"
```

---

# Why the Script Should Not Be Manually Run

During development, manually pressing **Run** produced an error because:

```text
e was undefined
```

This was expected.

The function relies on information supplied by a real form submission.

If the function is manually started from the Apps Script editor, there is no form submission event.

Therefore:

```javascript
e.values
```

does not exist.

The correct way to test the script is to submit a real response through the Google Form.

---

# 2. Reading the Form Values

This line:

```javascript
const values = e.values;
```

takes the submitted form values and stores them in a variable called:

```text
values
```

The answers are stored in order.

For example:

```text
values[0] = Timestamp
values[1] = Site
values[2] = System
values[3] = Reported fault
values[4] = Fault message
values[5] = Site impact
values[6] = Caller name
values[7] = Callback telephone
```

---

# Zero-Based Indexing

JavaScript starts counting from zero.

This is called **zero-based indexing**.

So:

```text
0 = first item
1 = second item
2 = third item
3 = fourth item
```

This is why:

```javascript
const siteDisplayName = values[1];
```

reads the second column rather than the first.

The first value:

```javascript
values[0]
```

is the automatically generated timestamp.

---

# 3. Creating Meaningful Variables

The script then creates variables for each form answer:

```javascript
const siteDisplayName = values[1];
const systemType = values[2];
const reportedFault = values[3];
const faultMessage = values[4] || 'None reported';
const siteImpact = values[5] || '';
const callerName = values[6];
const callbackPhone = values[7];
```

Instead of repeatedly using values such as:

```text
values[2]
values[3]
values[6]
```

the rest of the program can use names such as:

```text
systemType
reportedFault
callerName
```

This makes the code easier to read.

---

# Optional Fields

Some form fields are optional.

For example:

```javascript
const faultMessage = values[4] || 'None reported';
```

The `||` means:

> Use the value on the left if it exists. Otherwise use the value on the right.

So if the customer enters an error message:

```text
Fault message
→ Error 45
```

that value is used.

If they leave the field blank:

```text
Fault message
→ None reported
```

is used instead.

The site impact field works similarly:

```javascript
const siteImpact = values[5] || '';
```

If nothing was entered, it remains blank.

---

# 4. Accessing the Spreadsheet

The script uses:

```javascript
const spreadsheet = e.source;
```

The event tells the script which spreadsheet generated the event.

That spreadsheet is stored in the variable:

```text
spreadsheet
```

The script then finds the Sites worksheet:

```javascript
const sitesSheet = spreadsheet.getSheetByName('Sites');
```

In simple terms:

> Go to the spreadsheet and find the sheet called Sites.

---

# 5. Checking That the Sites Sheet Exists

The script includes:

```javascript
if (!sitesSheet) {
  throw new Error('Sites sheet could not be found.');
}
```

This is basic **error handling**.

It means:

> If there is no worksheet called Sites, stop the script and report an error.

Without this check, the script could continue and fail later in a less obvious way.

---

# 6. Finding the Last Site Row

The script uses:

```javascript
const lastRow = sitesSheet.getLastRow();
```

This finds the last row containing data.

For example:

```text
Row 1 = Headers
Row 2 = Site 1
Row 3 = Site 2
Row 4 = Site 3
```

The last row would be:

```text
4
```

The script also checks:

```javascript
if (lastRow < 2) {
  throw new Error('No site records were found in the Sites sheet.');
}
```

This prevents the workflow continuing if the Sites table only contains headings and no actual site records.

---

# 7. Reading the Site Data

This section reads the site records:

```javascript
const siteData = sitesSheet
  .getRange(2, 1, lastRow - 1, 7)
  .getDisplayValues();
```

The four numbers passed into `getRange()` mean:

```text
2
→ Start at row 2

1
→ Start at column 1 / Column A

lastRow - 1
→ Read all site rows beneath the heading

7
→ Read seven columns
```

This means:

```text
Read A2:G(last row)
```

The first row is skipped because it contains the headings.

---

# 8. Why getDisplayValues() Was Used

The script uses:

```javascript
.getDisplayValues();
```

This reads the values as they appear in the spreadsheet.

That is useful for information such as:

- telephone numbers
- site IDs
- formatted text

For example, telephone numbers may start with a zero.

Reading displayed text helps preserve the value in the form the user expects to see it.

---

# 9. Finding the Selected Site

This is one of the most important parts of the script:

```javascript
const matchedSite = siteData.find(function(row) {
  return row[3].trim().toLowerCase() ===
         siteDisplayName.trim().toLowerCase();
});
```

The script checks each site row until it finds a Site Display Name matching the site selected in the Google Form.

The logic is effectively:

```text
Form says:
Demo Academy North

        ↓

Look through Sites table

        ↓

Find:
Demo Academy North

        ↓

Return that complete site row
```

---

# Why row[3] Is Used

The site data starts with:

```text
row[0] = Column A = Site ID
row[1] = Column B = Customer
row[2] = Column C = Branch
row[3] = Column D = Site Display Name
row[4] = Column E = Telephone
row[5] = Column F = Customer Email
row[6] = Column G = Address
```

Because JavaScript starts counting from zero:

```javascript
row[3]
```

means the fourth value, which is Column D.

Column D contains the Site Display Name.

---

# 10. trim()

The comparison uses:

```javascript
.trim()
```

This removes unnecessary spaces from the beginning or end of text.

For example:

```text
"Demo Academy North "
```

becomes:

```text
"Demo Academy North"
```

This prevents an accidental space from stopping a valid match.

---

# 11. toLowerCase()

The comparison also uses:

```javascript
.toLowerCase()
```

This converts the text to lowercase before comparing it.

For example:

```text
Demo Academy North

DEMO ACADEMY NORTH

demo academy north
```

all become:

```text
demo academy north
```

This makes the site matching less sensitive to capitalisation.

---

# 12. Handling a Missing Site

If no match is found, the script runs:

```javascript
if (!matchedSite) {
  throw new Error('Site not found: ' + siteDisplayName);
}
```

This deliberately stops the process.

It is better to stop and report:

```text
Site not found
```

than to guess which customer or site the request belongs to.

---

# 13. Retrieving Official Site Information

Once a matching site has been found, the script retrieves information from that row:

```javascript
const siteId = matchedSite[0];
const customerName = matchedSite[1];
const officialSiteName = matchedSite[3];
const customerEmail = matchedSite[5];
```

This maps to:

```text
matchedSite[0]
→ Column A
→ Site ID

matchedSite[1]
→ Column B
→ Customer / Organisation

matchedSite[3]
→ Column D
→ Site Display Name

matchedSite[5]
→ Column F
→ Customer Email
```

This means the user only needs to select the site.

The system retrieves the remaining official information automatically.

---

# 14. Data Enrichment

This is an example of **data enrichment**.

The form originally contains:

```text
Site
System
Fault
Caller
Callback number
```

The Sites table adds information such as:

```text
Site ID
Customer name
Official site name
Customer email
```

The final maintenance request therefore contains more useful information than the form alone.

Conceptually:

```text
FORM DATA
+
SITE REFERENCE DATA
=
COMPLETE MAINTENANCE REQUEST
```

---

# 15. Building the Payload

The script then builds this object:

```javascript
const payload = {
  site_id: siteId,
  customer_name: customerName,
  site_name: officialSiteName,
  customer_email: customerEmail,

  system_type: systemType,
  reported_fault: reportedFault,
  fault_message: faultMessage,
  site_impact: siteImpact,

  caller_name: callerName,

  use_calling_number: 'false',
  incoming_caller_number: '',
  alternate_callback_phone: callbackPhone
};
```

This creates the structured information that Make.com expects.

---

# Why the Payload Structure Matters

The AI phone workflow already sends fields such as:

```text
site_id
customer_name
site_name
customer_email
system_type
reported_fault
fault_message
site_impact
caller_name
use_calling_number
incoming_caller_number
alternate_callback_phone
```

I designed the form integration to use the same field names.

This means Make receives the same basic structure regardless of the source.

```text
AI Phone Call
      ↓
Standard maintenance payload
      ↓
Make

Google Form
      ↓
Standard maintenance payload
      ↓
Make
```

This is useful because the ticket creation logic only needs to be built once.

---

# 16. Callback Number Logic

The AI phone workflow has two possible callback methods.

A caller can either:

```text
Use the number they are currently calling from
```

or:

```text
Provide a different telephone number
```

The Google Form is different because there is no live incoming telephone call.

The form therefore sends:

```javascript
use_calling_number: 'false',
incoming_caller_number: '',
alternate_callback_phone: callbackPhone
```

This tells the Make workflow:

> There is no incoming call number. Use the callback number entered on the form instead.

---

# 17. Webhook URL

The script contains:

```javascript
const webhookUrl = 'YOUR_MAKE_WEBHOOK_URL';
```

In the working prototype, this value contains the real Make webhook endpoint.

The real URL should not be included in a public GitHub repository.

A webhook URL can act as an entry point into an automation.

Publishing it could allow unwanted requests to be sent into the workflow.

For that reason, the portfolio version uses a placeholder.

---

# 18. Sending the Request

The script sends the data using:

```javascript
const response = UrlFetchApp.fetch(webhookUrl, {
  method: 'post',
  contentType: 'application/json',
  payload: JSON.stringify(payload),
  muteHttpExceptions: true
});
```

This is the point where Google Apps Script communicates with Make.com.

---

# UrlFetchApp.fetch()

`UrlFetchApp.fetch()` is a Google Apps Script function used to make web requests.

In simple terms:

> Send a request to this internet address.

The destination is:

```text
webhookUrl
```

which points to the Make webhook.

---

# method: 'post'

This tells Google to use an HTTP **POST** request.

POST is commonly used when sending information to another system for processing.

In this project:

```text
POST
→ Send this new maintenance request to Make
```

---

# contentType: 'application/json'

This tells the receiving system:

> The information I am sending is formatted as JSON.

Make can then interpret the incoming payload correctly.

---

# JSON.stringify(payload)

Inside JavaScript, the payload is initially a JavaScript object.

This:

```javascript
JSON.stringify(payload)
```

converts it into JSON text that can be transmitted across the web.

Conceptually:

```text
JavaScript object
        ↓
JSON.stringify()
        ↓
JSON text
        ↓
Sent to Make
```

---

# muteHttpExceptions

The script includes:

```javascript
muteHttpExceptions: true
```

Normally, some unsuccessful HTTP responses can immediately stop the request with an exception.

Using this option allows the script to receive the response first.

The code can then inspect:

```text
status code
response body
```

and decide how to handle the failure.

This made debugging easier.

---

# 19. Reading the Make Response

After sending the request, the script stores the response:

```javascript
const statusCode = response.getResponseCode();
const responseText = response.getContentText();
```

This captures:

```text
statusCode
→ numerical HTTP result

responseText
→ message returned by Make
```

The values are also written to the Apps Script execution log:

```javascript
console.log('Make status code: ' + statusCode);
console.log('Make response: ' + responseText);
```

This is useful when troubleshooting.

---

# 20. Checking Whether the Request Succeeded

The script then checks:

```javascript
if (statusCode < 200 || statusCode >= 300) {
```

HTTP codes in the 200 range normally indicate success.

So this condition effectively means:

> If the response is not a successful 2xx status, treat it as an error.

The script then throws:

```javascript
throw new Error(
  'Make returned HTTP ' +
  statusCode +
  ': ' +
  responseText
);
```

This prevents a failed automation from being silently treated as successful.

---

# 21. Successful Completion

If no error occurs, the final line writes:

```javascript
console.log('Ticket successfully sent to Make.');
```

This confirms that the request was successfully accepted by the automation endpoint.

---

# Installing the Trigger

Writing the function alone is not enough.

Google Apps Script also needs to know when to run it.

An installable trigger was created with:

```text
Function
→ onFormSubmit

Event source
→ From spreadsheet

Event type
→ On form submit
```

The result is:

```text
New form submission
        ↓
Trigger activates
        ↓
onFormSubmit runs automatically
```

No manual execution is required.

---

# Why the Trigger Is Attached to the Spreadsheet

The Google Form writes its submissions into the linked spreadsheet.

The script needs access to:

```text
e.values
```

and:

```text
e.source
```

from the spreadsheet form-submission event.

The trigger was therefore created from the spreadsheet Apps Script project using:

```text
From spreadsheet
→ On form submit
```

---

# Permissions

The first time the script was configured, Google requested permission for the script to access the services it needed.

For this workflow, that included access required to:

- read spreadsheet information
- work with the linked form response environment
- make an external web request to Make

This is because the script operates on behalf of the Google account that owns or authorises the automation.

---

# Debugging Example 1: Incorrect Customer Email Mapping

One of the first problems found during testing was an email failure.

The Make email module reported that the recipient was not a valid email address.

Inspecting the incoming data showed something similar to:

```text
customer_email:
61 Example Road, London
```

The field called:

```text
customer_email
```

was clearly receiving a postal address.

---

## Finding the Cause

The Sites table used:

```text
Column F
→ Customer Email

Column G
→ Address
```

Because JavaScript uses zero-based indexing:

```text
matchedSite[5]
→ sixth value
→ Column F

matchedSite[6]
→ seventh value
→ Column G
```

The script had originally used the wrong index.

It was corrected to:

```javascript
const customerEmail = matchedSite[5];
```

After this change, Make received the actual email address.

---

# What I Learned From This Error

This was a useful example of a **data mapping problem**.

The automation itself was working.

The issue was that the right type of data had been connected to the wrong field.

Tracing the data through each stage made it possible to identify where the incorrect value first appeared.

The debugging process was roughly:

```text
Email fails
        ↓
Inspect email recipient in Make
        ↓
Notice postal address instead of email
        ↓
Inspect webhook payload
        ↓
Confirm customer_email contains address
        ↓
Check Apps Script mapping
        ↓
Find incorrect array index
        ↓
Correct Column F mapping
```

---

# Debugging Example 2: Duplicate Tickets

Another issue appeared when one form submission created two identical tickets.

Both tickets had:

- the same content
- the same ticket ID
- nearly identical timestamps

The first step was to determine whether the duplication happened inside Make or before the data reached Make.

The Apps Script trigger page showed two identical triggers:

```text
Trigger 1
onFormSubmit

Trigger 2
onFormSubmit
```

This meant one form submission caused both triggers to run.

The flow was effectively:

```text
ONE FORM SUBMISSION
        ↓
        ├───────────────┐
        ↓               ↓
Trigger 1          Trigger 2
        ↓               ↓
Apps Script        Apps Script
        ↓               ↓
Make request       Make request
        ↓               ↓
Ticket 1           Ticket 2
```

One duplicate trigger was deleted.

After that:

```text
One form submission
        ↓
One trigger
        ↓
One script execution
        ↓
One Make execution
        ↓
One ticket
```

---

# What I Learned From the Duplicate Ticket Issue

The problem was not in the ticket-generation logic itself.

It was caused by the same event being handled twice.

This helped reinforce the idea of **event-driven systems**.

When debugging an automated workflow, it is important to ask:

```text
Did one event happen twice?

or

Did one event cause one workflow to create something twice?
```

Those are different problems and need to be investigated differently.

---

# Shared Backend Design

One of the parts of this integration I found most useful was reusing the existing ticket workflow.

I could have created:

```text
Phone workflow
→ Make workflow A

Form workflow
→ Make workflow B
```

Instead, I used:

```text
Phone
    \
     \
      → Standard ticket payload → One Make ticket workflow
     /
    /
Form
```

This reduced duplication.

It also means changes to the core ticket creation logic can apply to both intake methods.

For example, both routes can use the same:

- ticket format
- ticket register
- email templates
- Away Mode settings
- business notifications
- customer confirmations

---

# Input vs Enriched vs Final Data

The transformation can be viewed in three stages.

## Stage 1: User-provided form data

```text
Site
→ Demo Academy North

System
→ CCTV

Fault
→ Entrance camera not displaying

Caller
→ Ahmed

Callback
→ 020 0000 0000
```

## Stage 2: Site lookup adds reference data

```text
Site ID
→ SITE-001

Customer
→ Demo Academy

Official Site Name
→ Demo Academy North

Customer Email
→ customer@example.com
```

## Stage 3: Final structured payload

```json
{
  "site_id": "SITE-001",
  "customer_name": "Demo Academy",
  "site_name": "Demo Academy North",
  "customer_email": "customer@example.com",
  "system_type": "CCTV",
  "reported_fault": "Entrance camera not displaying",
  "fault_message": "None reported",
  "site_impact": "Front entrance cannot be monitored",
  "caller_name": "Ahmed",
  "use_calling_number": "false",
  "incoming_caller_number": "",
  "alternate_callback_phone": "020 0000 0000"
}
```

That final structure is what allows the rest of the workflow to operate consistently.

---

# Why This Matters From a Data Perspective

Although the Google Form looks simple to the user, there is more happening behind it.

The process includes:

- data collection
- reference-data lookup
- validation
- data enrichment
- field mapping
- transformation
- structured data exchange
- event-driven automation
- error handling

The important part is not simply that a form creates a spreadsheet row.

The form response is being transformed into a structured business record that can be used by multiple systems.

---

# Security and Privacy

The public repository does not contain:

- the real Make webhook URL
- customer email addresses
- customer telephone numbers
- real addresses
- customer names
- live maintenance records
- credentials
- API secrets

Example and fictional data are used instead.

The real webhook should be treated as a private integration endpoint and should not be published in a public repository.

---

# Possible Future Improvements

For a production version, I would consider several improvements.

### Move secrets into configuration

Rather than placing integration URLs directly inside the script, production systems should normally store secrets and environment-specific configuration separately.

### Stronger validation

The script could validate:

- email format
- telephone format
- required fields
- allowed system types

before sending the request.

### Idempotency

A production system could include a unique form submission identifier.

This could be used to prevent the same request from accidentally being processed more than once.

### Logging and monitoring

A production version could store:

- processing status
- error reason
- retry attempts
- ticket ID returned by the backend

against each form response.

### More scalable data storage

Google Sheets works well for a prototype.

A larger implementation could use:

- a relational database
- a CRM
- a help-desk platform
- a field-service management system

while keeping the same general integration pattern.

---

# Key Learning

The main thing I learned from this part of the project was that integration work is not only about connecting applications.

The quality of the workflow depends heavily on:

- understanding the structure of the data
- mapping fields correctly
- knowing when automations are triggered
- validating reference information
- checking what happens when something fails
- making sure different input channels produce consistent outputs

The Google Form itself is simple.

The more interesting part is how its data is validated, enriched, transformed and passed into the wider maintenance workflow.
