# Technical Glossary

This page explains some of the technical terms used throughout this project in simple language.

While building the prototype, I used several technologies and concepts that are common in automation and data projects. I wanted to document what they actually mean, as well as how I used them in this project.

---

## API

**API** stands for **Application Programming Interface**.

The simplest way I think about an API is:

> A defined way for one software system to communicate with another.

For example, one system might ask another system to:

- return some information
- create a record
- update something
- trigger an action

### Simple analogy

Imagine a restaurant.

The kitchen is one system and the customer is another.

The customer does not normally walk into the kitchen and start using the equipment. Instead, there is a defined way of placing an order and receiving the result.

An API provides a similar controlled way for software systems to communicate.

### How this relates to my project

The maintenance help desk uses several different tools, including Retell AI, Make.com, Google Sheets and email services.

These tools need ways of exchanging information with one another.

APIs are part of what makes that communication possible.

---

## Webhook

A **webhook** is a way for one system to automatically send information to another system when something happens.

A simple way to think about it is:

> “Something has just happened, so I am sending you the information now.”

For example, once the AI has collected and confirmed a maintenance request, the information needs to be sent to the automation workflow.

The Make.com webhook acts as a receiving point for that information.

```text
Caller confirms maintenance request
        ↓
Retell sends the ticket information
        ↓
Make webhook receives it
        ↓
Ticket workflow starts
```

The Google Form route also sends maintenance information to the same Make webhook.

This means that both the phone workflow and form workflow can use the same ticket creation process.

---

## API vs Webhook

APIs and webhooks are related, but I found it useful to understand the difference.

| Term | Simple explanation | Example |
|---|---|---|
| API | A defined way for software systems to communicate | One system asks another system to retrieve or create something |
| Webhook | Automatically sends information when an event happens | A confirmed maintenance request is immediately sent to Make |

A webhook normally uses standard web/API technology underneath, which is why the terms can sometimes overlap.

---

## JSON

**JSON** stands for **JavaScript Object Notation**.

Despite the technical name, it is simply a structured way of organising information so that software systems can understand what each value represents.

For example, a caller might say:

> “Ahmed is calling from Demo Academy North because one of the CCTV cameras is not displaying.”

That is useful to a person, but an automation works better if the important information is separated into fields.

For example:

```json
{
  "site_name": "Demo Academy North",
  "system_type": "CCTV",
  "reported_fault": "Camera not displaying",
  "caller_name": "Ahmed"
}
```

Instead of receiving one long sentence, the next system knows exactly what each piece of information represents.

---

## Payload

A **payload** is the actual information being sent from one system to another.

For example:

```json
{
  "site_id": "SITE-001",
  "system_type": "CCTV",
  "reported_fault": "Camera not displaying"
}
```

This could be the payload sent from the voice system to Make.com.

A simple way I remember the relationship is:

```text
Webhook = delivery address

JSON = how the information is organised

Payload = the package being delivered
```

---

## HTTP

**HTTP** stands for **Hypertext Transfer Protocol**.

It is one of the main sets of rules used for communication across the web.

When one application sends information to another through a web request, it commonly uses HTTP.

In this project, the Google Apps Script sends an HTTP request to the Make webhook.

---

## POST Request

**POST** is a common type of HTTP request.

It is normally used when one system wants to send information to another system for processing or to create something.

A simple way to think about POST is:

> “Here is some information for you to process.”

In this project:

```text
Google Form submitted
        ↓
Apps Script prepares the maintenance data
        ↓
POST request sends the data
        ↓
Make webhook receives it
```

---

## Variable

A **variable** is a named place where a program temporarily stores information.

For example:

```javascript
const callerName = values[6];
```

This means:

> Take a particular value and store it under the name `callerName`.

Using meaningful variable names makes code easier to understand.

Instead of repeatedly referring to something such as:

```text
values[6]
```

the code can refer to:

```text
callerName
```

which makes its purpose much clearer.

---

## Function

A **function** is a block of instructions designed to perform a particular task.

For example:

```javascript
function onFormSubmit(e) {

}
```

This creates a function called:

```text
onFormSubmit
```

In this project, that function is used when someone submits the maintenance form.

The function then performs several tasks:

```text
Read form answers
        ↓
Find the selected site
        ↓
Retrieve official site information
        ↓
Build the maintenance payload
        ↓
Send the information to Make
```

---

## Trigger

A **trigger** tells a system when something should happen automatically.

For example:

```text
Event
New Google Form response

        ↓

Trigger
Run onFormSubmit

        ↓

Action
Send maintenance request to Make
```

Without the trigger, somebody would need to manually run the script every time a form was submitted.

### Example from testing

During testing, I accidentally had two identical form submission triggers enabled.

This meant one form submission caused the same script to run twice.

The result was:

```text
One form submission
        ↓
Trigger 1 runs
        ↓
Ticket created

AND

Trigger 2 runs
        ↓
Second identical ticket created
```

Removing the duplicate trigger fixed the issue.

This was a useful example of debugging an event-driven automation.

---

## Event

An **event** is something that happens within a system.

Examples include:

- a form being submitted
- a phone call being received
- a ticket being created
- a spreadsheet row being updated
- a status being changed

In the Google Form workflow, the event is:

```text
A new form response is submitted
```

That event activates the trigger, which runs the Apps Script.

---

## Data Extraction

**Data extraction** means identifying and separating useful pieces of information from a larger input.

For example, a caller might say:

> “The front camera at the Romford branch isn't showing anything.”

The AI can extract:

```text
Location
→ Romford branch

System
→ CCTV

Reported fault
→ Camera not displaying
```

This is one of the most important parts of the voice workflow.

The AI is not simply having a conversation with the customer.

It is turning natural speech into structured information that can be used by the rest of the system.

---

## Structured Data

**Structured data** is information organised into known fields.

For example:

| Field | Value |
|---|---|
| Site | Demo Academy North |
| System Type | CCTV |
| Reported Fault | Camera not displaying |
| Caller Name | Ahmed |
| Status | New |

Because each value has a known meaning, the information can easily be:

- searched
- filtered
- stored
- analysed
- passed between systems
- used in automation

---

## Unstructured Data

**Unstructured data** does not follow a fixed format.

For example:

> “The camera near reception stopped working this morning and now we can't see visitors arriving.”

This is easy for a person to understand, but it is not yet organised into separate business fields.

The AI helps transform that into something like:

```text
System Type
→ CCTV

Reported Fault
→ Reception camera not working

Site Impact
→ Visitors cannot be monitored
```

One of the main goals of this project was to turn unstructured conversations into structured maintenance data.

---

## Validation

**Validation** means checking information before using it.

For example, if someone says they are calling from a particular site, the system should not simply assume that the site exists.

Instead, the site can be checked against an approved site register.

```text
Caller provides site
        ↓
Search site register
        ↓
Matching site found
        ↓
Official site information returned
```

This reduces the chance of a ticket being created against an incorrect or unknown site.

---

## Site Lookup

A **site lookup** is the process of searching the stored Sites data for the location provided by the caller.

For example:

```text
Caller says
"Demo Academy North"

        ↓

Search Sites table

        ↓

Matching site found
```

The stored record might contain:

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

The rest of the workflow can then use the official stored information instead of relying entirely on what the caller said.

---

## Data Mapping

**Data mapping** means making sure information from one system is connected to the correct field in another system.

For example:

```text
Customer Email
        ↓
must map to
        ↓
Email recipient
```

### Example from this project

During testing, the Google Form integration initially used the wrong spreadsheet column for the customer email.

The site table contained:

```text
Column F
Customer Email

Column G
Address
```

The script accidentally read Column G.

This meant Make received something similar to:

```text
customer_email:
61 Example Road, London
```

The email module then failed because a postal address is not a valid email address.

By inspecting the data passing through the workflow, I traced the problem back to the spreadsheet mapping and corrected the script.

This was a useful example of why data mapping is important when connecting different systems.

---

## Orchestration

**Orchestration** means coordinating several systems and actions so that they work together as one process.

Make.com acts as the orchestration layer in this project.

For example:

```text
Receive maintenance request
        ↓
Create ticket information
        ↓
Write ticket to Google Sheets
        ↓
Read business settings
        ↓
Send business email
        ↓
Send customer confirmation
        ↓
Check Away Mode
        ↓
Send additional email if needed
        ↓
Return confirmation
```

Each step may involve a different application, but Make coordinates the overall workflow.

---

## Conditional Routing

**Conditional routing** means choosing what happens next depending on whether a condition is true.

The Away Mode feature is an example.

```text
Away Mode = No

Business email sent
Customer confirmation sent
```

Whereas:

```text
Away Mode = Yes

Business email sent
Customer confirmation sent
Additional copy sent to away email
```

This allows the behaviour of the system to be changed through a simple setting instead of rebuilding the automation.

---

## Data Model

A **data model** describes how information is organised.

For this prototype, I separated different types of information into different tables.

### Sites

Contains relatively stable information about known customer locations.

Examples include:

- Site ID
- organisation
- branch
- official site name
- telephone
- customer email
- address

### Tickets

Contains individual maintenance requests.

Examples include:

- Ticket ID
- date created
- site
- caller
- telephone number
- system type
- fault
- site impact
- status
- resolution information

### Settings

Contains values that can change the behaviour of the workflow.

Examples include:

- primary business email
- Away Mode
- alternative email

### Form Responses

Contains the original information submitted through Google Forms.

Separating the information in this way makes the system easier to maintain and reduces unnecessary duplication.

---

## Unique Identifier / ID

An **ID** is a value used to uniquely identify something.

For example:

```text
SITE-001
```

could uniquely identify a site.

A maintenance request also receives its own ticket ID.

This means two tickets can relate to the same site while still being separate records.

For example:

```text
SITE-001
    ↓
Ticket A

SITE-001
    ↓
Ticket B
```

Both tickets belong to the same site, but each ticket has its own unique identifier.

---

## Ticket ID

The **Ticket ID** is the unique reference generated for each maintenance request.

This allows the business and customer to refer to a particular issue without relying only on descriptions.

For example:

```text
Ticket ID
→ TICKET-2026-001
```

A ticket ID can then be used when discussing, tracking or resolving that particular maintenance request.

---

## Boolean

A **Boolean** is a value that represents one of two states:

```text
True
False
```

An example in this project is whether the caller wants to use their incoming phone number.

Conceptually:

```text
Use calling number = True
```

means:

> Use the telephone number the caller is currently calling from.

Whereas:

```text
Use calling number = False
```

means:

> Use the alternative callback number provided by the caller.

---

## Array

An **array** is a collection of values stored in order.

For example, when Google Apps Script receives a row from the form, the values can be accessed by position.

```javascript
const values = e.values;
```

The values might conceptually look like:

```text
values[0] = Timestamp
values[1] = Site
values[2] = System
values[3] = Reported Fault
```

JavaScript starts counting at zero, which is why the first item is:

```text
values[0]
```

rather than:

```text
values[1]
```

This is known as **zero-based indexing**.

---

## Zero-Based Indexing

Many programming languages, including JavaScript, start counting positions from zero.

For example:

```text
Position 0 = first item
Position 1 = second item
Position 2 = third item
```

This became important when reading columns from the Sites data.

For example:

```javascript
const customerEmail = matchedSite[5];
```

Index `5` refers to the sixth value in the row.

In the Sites table, that corresponds to Column F.

Understanding this helped fix the customer email mapping issue during testing.

---

## Google Apps Script

**Google Apps Script** is Google's scripting platform for automating products such as:

- Google Sheets
- Google Forms
- Gmail
- Google Drive

It is based on JavaScript.

In this project, Apps Script connects Google Forms to the existing maintenance ticket workflow.

Its job is roughly:

```text
Form submitted
        ↓
Read answers
        ↓
Find site in Sites table
        ↓
Retrieve official site information
        ↓
Build JSON payload
        ↓
POST payload to Make webhook
```

This allowed the Google Form to use the same ticket creation process as the AI phone system.

---

## Automation

**Automation** means allowing a system to perform repeatable actions automatically instead of requiring somebody to manually complete every step.

Before automation, a process might look like:

```text
Customer calls
        ↓
Someone writes the information down
        ↓
Someone creates a ticket
        ↓
Someone emails the customer
        ↓
Someone emails the engineer/business
```

The prototype automates much of that initial process:

```text
Customer reports issue
        ↓
Information collected
        ↓
Ticket created
        ↓
Record stored
        ↓
Notifications sent
```

---

## Workflow

A **workflow** is the sequence of steps used to complete a process.

For example, the voice maintenance workflow is:

```text
Identify site
        ↓
Validate site
        ↓
Collect fault
        ↓
Collect caller information
        ↓
Review information
        ↓
Confirm request
        ↓
Create ticket
        ↓
Send notifications
```

Automation is used to make parts of the workflow happen automatically.

---

## Error Handling

**Error handling** means deciding what should happen when something goes wrong.

For example, the Google Apps Script checks the response returned by Make.

If Make returns a successful HTTP status code, the request can continue normally.

If it returns an error, the script raises an error rather than silently assuming the ticket was successfully processed.

This is important because an automated system should not claim that an action has succeeded if it has actually failed.

---

## HTTP Status Code

An **HTTP status code** is a number returned after a web request that indicates what happened.

Common examples include:

```text
200
Request succeeded

400
There was a problem with the request

404
The requested resource could not be found

500
Something went wrong on the receiving server
```

The form integration checks whether Make returned a successful response before treating the request as successfully processed.

---

## Proof of Concept / Prototype

A **proof of concept**, often shortened to **PoC**, is a smaller implementation used to test whether an idea can work.

This project is a prototype rather than a production system.

The aim was to demonstrate that a maintenance process could combine:

- conversational AI
- structured data capture
- site validation
- workflow automation
- ticket creation
- customer notifications
- alternative web-form intake

A production implementation could later replace some prototype components with more scalable or specialised systems.
