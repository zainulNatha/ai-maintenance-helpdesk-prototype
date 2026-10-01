# AI Maintenance Help Desk Prototype

## About the project

I built this as a personal project to explore how a small maintenance business could make it easier to log and manage customer faults.

The idea came from a fairly common problem for small maintenance businesses. Customers may call or email with an issue while engineers are already out working on-site, which can make it difficult to respond straight away. Details may then need to be gathered manually, including which site the issue relates to, what the problem is, and who needs to follow it up.

I wanted to see how much of that first stage could be automated without making the experience difficult for the customer or business.

The prototype allows someone to report a maintenance issue in two ways:

- by speaking to an AI voice assistant over the phone
- by completing a simple online form

Both routes feed into the same ticketing process.

The aim was not just to create a chatbot, but to turn an unstructured maintenance request into structured data that could be validated, recorded and used by the rest of the workflow.

---

## What the prototype does

At a high level, the system:

1. identifies the customer site
2. validates the site against a maintained site list
3. captures the maintenance issue
4. identifies the type of system affected
5. records the caller and callback details
6. allows the information to be reviewed and corrected
7. creates a maintenance ticket
8. sends confirmation emails to the business and customer
9. records the ticket in a central register

A web form was also added as an alternative reporting route, using the same ticket creation process as the phone workflow.

---

## How the system works

The prototype has two ways of reporting a maintenance issue, but both routes eventually use the same ticket creation process.

### Phone route

```text
Customer calls maintenance number
        ↓
AI voice assistant answers
        ↓
Customer identifies the site
        ↓
Site is checked against the maintained site register
        ↓
Customer explains the maintenance issue
        ↓
AI extracts the relevant information
        ↓
Caller and callback details are collected
        ↓
Customer reviews and confirms the request
        ↓
Structured data is sent to the automation workflow
        ↓
Maintenance ticket is created
        ↓
Business and customer notifications are sent
        ↓
Ticket reference is returned to the caller
```

### Online form route

```text
Customer submits maintenance form
        ↓
Google Apps Script runs automatically
        ↓
Selected site is matched against the Sites register
        ↓
Stored site information is added to the form data
        ↓
Structured data is sent to the same automation workflow
        ↓
Maintenance ticket is created
        ↓
Business and customer notifications are sent
```

The main design decision was to avoid building separate ticketing processes for each input method.

Whether the request starts as a phone conversation or a form submission, it is converted into the same type of structured maintenance request before being passed into the ticket workflow.

---

## What the AI is actually doing

The voice assistant is not only having a conversation with the caller.

Its main role is to turn information provided naturally during the conversation into structured fields that can be used by the rest of the system.

For example, a caller may describe a problem in their own words:

> “The camera at the front entrance isn't showing anything and we can't see people arriving.”

The useful information can then be separated into structured fields:

| Information | Structured result |
|---|---|
| Affected system | CCTV |
| Reported fault | Front entrance camera not displaying |
| Site impact | Entrance cannot be monitored |

Other information is collected during the same conversation:

| Information | Purpose |
|---|---|
| Site | Identifies where the problem has occurred |
| Site ID | Links the request to the internal site record |
| Caller name | Records who reported the issue |
| Callback number | Gives the engineer a contact number |
| System type | Categorises the affected equipment |
| Reported fault | Records the maintenance problem |
| Site impact | Captures how the issue is affecting the customer |

Before the ticket is created, the caller is given an opportunity to review and correct the information.

The system only confirms that a maintenance ticket has been created after the backend automation successfully creates the ticket and returns confirmation.

---

## Site validation and reference data

I created a Sites table to act as the reference source for known customer locations.

The table contains fields such as:

| Field | Purpose |
|---|---|
| Site ID | Unique internal identifier for the site |
| Site Name | Customer or organisation |
| Branch | Branch or location where required |
| Site Display Name | Name used when confirming the site |
| Telephone | Stored site contact number |
| Customer Email | Used for ticket confirmation |
| Address | Stored site address |

The actual customer records are not included in this repository.

When someone reports a fault, the system does not simply accept or invent site information.

The site provided by the caller or form submission is matched against this maintained reference data.

Once a site is matched, the workflow can use trusted information already stored against that location.

Conceptually:

```text
Customer provides site
        ↓
Site register searched
        ↓
Matching site found
        ↓
Official site information retrieved
        ↓
Maintenance request enriched with reference data
```

This separates information the customer provides from information already maintained by the business.

It also means the customer does not need to know internal values such as a Site ID.

---

## Data transformation

One of the most important parts of the project is the transformation from unstructured information into structured business data.

A caller may speak naturally:

```text
"The front entrance camera is not displaying and we cannot see visitors arriving."
```

The workflow turns that into something more structured:

```text
System Type
→ CCTV

Reported Fault
→ Front entrance camera not displaying

Site Impact
→ Entrance cannot be monitored
```

Once the information is structured, it becomes much easier to:

- store
- search
- filter
- pass between systems
- use in automation
- report on later

This was one of the main reasons I wanted to build the prototype.

---

## Main technologies used

### Retell AI

Used for voice-based conversational intake.

The voice workflow collects the maintenance information, confirms it with the caller and passes the structured request into the automation workflow.

### Make.com

Used as the automation and orchestration layer.

Make.com receives the structured maintenance request and coordinates the rest of the process, including ticket creation, Google Sheets updates, business settings and email notifications.

### Google Sheets

Used as a lightweight prototype data store.

The workbook contains different logical areas for:

- Sites
- Tickets
- Settings
- Form Responses

### Google Forms

Used as an alternative maintenance request channel.

This allows someone to report the same type of maintenance issue without needing to call the voice assistant.

### Google Apps Script

Used to connect Google Form submissions to the same automation workflow used by the AI phone route.

The script reads the form submission, looks up the selected site, adds the official site information and sends the structured request to Make.com.

### Twilio

Used as part of the telephony setup for the dedicated maintenance number.

### Webhooks and JSON

Used to pass structured data between systems.

---

## Google Form route

The Google Form collects:

- site
- affected system
- maintenance issue
- optional fault or error message
- site impact
- caller name
- callback telephone number

The user does not need to enter internal site information such as a Site ID or stored customer email address.

Instead, Google Apps Script retrieves that information from the Sites table.

The process is:

```text
Form submitted
        ↓
Apps Script reads the response
        ↓
Selected site is found in the Sites table
        ↓
Official site information is retrieved
        ↓
Form data and reference data are combined
        ↓
Structured JSON payload is created
        ↓
Payload is sent to Make.com
        ↓
Existing ticket workflow runs
```

This means the phone route and form route share the same backend ticket process.

---

## Automation workflow

Make.com acts as the orchestration layer between the different parts of the prototype.

At a high level:

```text
Maintenance request received
        ↓
Ticket information prepared
        ↓
Ticket added to Google Sheets
        ↓
Business settings retrieved
        ↓
Business notification sent
        ↓
Customer confirmation sent
        ↓
Away Mode checked
        ↓
Additional notification sent if required
        ↓
Confirmation returned
```

The workflow also includes a configurable Away Mode.

When Away Mode is enabled, an additional internal ticket notification can be sent to an alternative email address without changing the automation itself.

---

## Ticket data

Each maintenance request is stored as a structured ticket.

The ticket register contains fields such as:

| Field |
|---|
| Ticket ID |
| Date Created |
| Site ID |
| Customer |
| Site Name |
| Caller Name |
| Caller Telephone |
| Customer Email |
| System Type |
| Reported Fault |
| Fault / Error Message |
| Site Impact |
| Status |
| Resolution Information |
| Confirmation Status |

The public repository does not contain live customer or ticket records.

---

## Testing and debugging

A large part of the project involved testing how information moved between the different systems.

Two useful examples were:

### Incorrect customer email mapping

During the Google Form integration, the customer email field was initially mapped to the wrong spreadsheet column.

This resulted in a postal address being passed into the email recipient field.

I traced the data through the automation, identified the incorrect array and column mapping, and corrected the Apps Script.

The debugging process was roughly:

```text
Email fails
        ↓
Inspect email recipient in Make.com
        ↓
Notice postal address instead of email address
        ↓
Inspect incoming payload
        ↓
Trace value back to Apps Script
        ↓
Find incorrect spreadsheet column index
        ↓
Correct mapping
```

This was a useful example of why data mapping matters when connecting different systems.

### Duplicate ticket creation

During another test, one form submission created two identical maintenance tickets.

The ticket creation logic itself was working correctly.

The issue was traced to two identical Google Apps Script form submission triggers running at the same time.

The result was effectively:

```text
One form submission
        ↓
Two triggers
        ↓
Two script executions
        ↓
Two Make.com requests
        ↓
Two tickets
```

Removing the duplicate trigger restored the expected behaviour:

```text
One form submission
        ↓
One trigger
        ↓
One script execution
        ↓
One automation execution
        ↓
One ticket
```

These issues helped reinforce the importance of tracing both data and events through an end-to-end workflow.

---

## Security considerations

The prototype deals with maintenance requests that can involve security and life-safety systems.

The voice workflow was therefore deliberately restricted.

The AI is not intended to:

- provide alarm or engineer codes
- reveal passwords or security credentials
- explain how to bypass or defeat a security system
- give instructions for disabling security or life-safety equipment
- invent missing site information

The AI's role is maintenance intake and ticket creation rather than detailed technical or security troubleshooting.

The system also avoids confirming that a ticket has been created until the backend automation has successfully returned a ticket reference.

---

## Prototype vs production

This project is a personal proof of concept rather than a production deployment.

Tools such as Google Sheets and Google Forms were useful because they allowed me to build and test the complete workflow quickly and at relatively low cost.

For a production implementation, I would review areas such as:

- database or service-management platform integration
- authentication and role-based access
- secrets management
- monitoring and logging
- retry and failure handling
- duplicate-request protection
- data retention
- audit history
- scalability
- CRM or help-desk integration
- SMS or WhatsApp notifications
- automated resolution notifications

The individual technologies could change while the overall architecture remains similar.

---

## What I learned

This project helped me understand how several areas work together in an end-to-end solution:

- conversational AI
- structured and unstructured data
- reference data
- data validation
- data enrichment
- data mapping
- APIs
- webhooks
- JSON payloads
- workflow automation
- conditional routing
- event-driven processing
- testing and debugging
- error handling
- business process design

One of the main things I learned was that the AI interface is only one part of the solution.

The reliability of the overall process depends on how the information is structured, validated, passed between systems and handled when something goes wrong.

---

## Project status

This is a personal proof-of-concept and portfolio project.

The current prototype successfully supports:

- AI-assisted phone intake
- site validation
- structured fault capture
- callback number capture
- ticket creation
- customer and business email notifications
- configurable away-email routing
- Google Form submissions using the same ticket workflow

---

## Documentation

More detailed technical documentation is being added in the `docs` folder.

Planned and completed documentation includes:

- system architecture
- voice AI workflow
- Google Form integration
- technical glossary
- testing and debugging
- production considerations
- lessons learned

---

## Walkthrough video

A short walkthrough video will be added to this repository.

The walkthrough will demonstrate:

- the AI maintenance call
- site identification and validation
- structured information capture
- ticket creation
- automated notifications
- how the different parts of the workflow connect

The walkthrough will not expose live customer information, credentials or private integration endpoints.

---

## Privacy

This repository documents the design and implementation of the prototype without publishing customer or business-sensitive information.

Real customer names, contact details, addresses, maintenance records, call recordings, credentials, webhook URLs and integration secrets are excluded or redacted.
