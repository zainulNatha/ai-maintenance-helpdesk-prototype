# Testing and Debugging

## Overview

A large part of this project was not just building the workflow, but testing how information moved between the different systems and fixing issues when the behaviour was not what I expected.

Because the prototype connects several tools together, a problem in one part of the workflow can affect the rest of the process.

The main areas I tested were:

- site lookup
- caller corrections
- structured data extraction
- callback number handling
- ticket creation
- customer notifications
- business notifications
- Away Mode routing
- Google Form submissions
- duplicate processing
- backend confirmation
- end-of-call behaviour

---

## Testing Approach

I tested the workflow in stages rather than only testing the final result.

For example:

```text
Input
   ↓
Was the correct data captured?
   ↓
Was the correct site returned?
   ↓
Was the payload correct?
   ↓
Did Make.com receive it?
   ↓
Was the ticket created?
   ↓
Were the emails sent?
   ↓
Was confirmation returned?
```

This made it easier to identify where a problem had started.

---

# Site Lookup Testing

The site lookup was one of the first parts of the voice workflow I tested.

The goal was to make sure that a caller could provide a normal site name or branch and have it matched against the maintained Sites data.

I tested scenarios such as:

- a valid site name
- an organisation with more than one branch
- correcting an incorrectly identified site
- confirming the correct site before continuing

The correction flow was especially important.

The intended behaviour was:

```text
AI identifies a site
        ↓
Caller says it is incorrect
        ↓
Caller provides corrected information
        ↓
Lookup runs again
        ↓
Correct site is returned
        ↓
Caller confirms it
```

This helped confirm that the workflow was not locked into the first interpretation made by the AI.

---

# Real Phone Call Testing

Once the main voice workflow was connected to the telephony setup, I tested the system using a real inbound phone call.

This helped confirm that the full path worked:

```text
Phone call
        ↓
Twilio
        ↓
Retell AI
        ↓
Site lookup
        ↓
Maintenance details collected
        ↓
Ticket request sent
        ↓
Make.com
        ↓
Ticket created
        ↓
Emails sent
```

One important part of this test was confirming that the real incoming caller number could be captured and used as the callback number when requested.

This validated the phone route end to end rather than only testing individual modules.

---

# Callback Number Testing

The workflow supports two callback options.

The caller can:

```text
Use the number they are calling from
```

or:

```text
Provide a different callback number
```

I tested both routes to make sure the correct number was passed into the ticket workflow.

Conceptually:

```text
use_calling_number = true
        ↓
Use incoming caller number
```

or:

```text
use_calling_number = false
        ↓
Use alternate callback number
```

This was important because the callback number is operational information used by the business after the ticket is created.

---

# Ticket Creation Testing

The main Make.com workflow was tested to make sure one confirmed maintenance request created one ticket.

The expected result was:

```text
Confirmed request
        ↓
Make.com receives payload
        ↓
Ticket ID generated
        ↓
Ticket added to Google Sheets
        ↓
Notifications sent
        ↓
Confirmation returned
```

The ticket register was then checked to make sure the fields had been populated correctly.

This included fields such as:

- Ticket ID
- Site ID
- Customer
- Site Name
- Caller Name
- Caller Telephone
- Customer Email
- System Type
- Reported Fault
- Fault Message
- Site Impact
- Status

---

# Debugging Example 1: Incorrect Customer Email Mapping

One of the main issues found during the Google Form integration was an email failure.

The customer confirmation email could not be sent because the email recipient field contained a postal address instead of an email address.

The failure appeared later in the workflow, but the actual cause was earlier in the data mapping.

The debugging process was:

```text
Customer email fails
        ↓
Inspect the email module
        ↓
Recipient contains a postal address
        ↓
Inspect the incoming webhook data
        ↓
customer_email contains the wrong value
        ↓
Trace the value back to Apps Script
        ↓
Check the Sites table column mapping
        ↓
Incorrect array index found
        ↓
Mapping corrected
```

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
→ Column F

matchedSite[6]
→ Column G
```

The script was corrected to use:

```javascript
const customerEmail = matchedSite[5];
```

After the change, the correct email address was passed into Make.com and the confirmation email succeeded.

---

## What this taught me

This was a useful example of a data mapping problem.

The email module itself was not the root cause.

The wrong value had entered the workflow earlier and only caused an error when the email module tried to use it.

This reinforced the importance of tracing data back to its source.

---

# Debugging Example 2: Duplicate Tickets

Another issue appeared when one Google Form submission created two identical tickets.

The two records had the same content and were created at almost the same time.

The first question was whether:

```text
One Make.com execution created two tickets
```

or:

```text
The same request entered Make.com twice
```

The Apps Script trigger page showed two identical `onFormSubmit` triggers.

This meant one form submission caused the same script to execute twice.

The actual flow was:

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

One duplicate trigger was removed.

The corrected flow became:

```text
One form submission
        ↓
One trigger
        ↓
One Apps Script execution
        ↓
One Make.com request
        ↓
One ticket
```

---

## What this taught me

The ticket creation logic was working correctly.

The problem was that the same event was being processed twice.

This helped reinforce the difference between:

```text
One event producing two outputs
```

and:

```text
One event accidentally being triggered twice
```

That distinction is useful when debugging event-driven workflows.

---

# Debugging Example 3: Form Script Event Object

During development, I manually ran the Google Apps Script and received an error because:

```text
e was undefined
```

This initially looked like a script problem.

However, the function was designed to receive the event object from a real form submission.

The function starts with:

```javascript
function onFormSubmit(e)
```

The `e` value only exists when the form submission trigger runs.

When the script is manually started from the editor, there is no form-submission event.

Therefore:

```javascript
e.values
```

does not exist.

The correct testing method was to submit a real Google Form response.

---

# Away Mode Testing

The workflow includes an Away Mode setting.

The expected behaviour was:

```text
Away Mode = No

Business email
+
Customer confirmation
```

and:

```text
Away Mode = Yes

Business email
+
Customer confirmation
+
Additional alternative email
```

I tested both settings.

This helped confirm that the conditional route only ran when the stored Away Mode value matched the required condition.

---

# Email Notification Testing

The ticket workflow sends different types of notification.

I checked:

- internal business notification
- customer confirmation
- alternative Away Mode notification

The important part was making sure that the correct recipient came from the correct source.

For example:

```text
Business email
→ Settings table

Customer email
→ Sites table

Away email
→ Settings table
```

This separates customer reference data from configurable business routing.

---

# Backend Confirmation Testing

A key design decision was that the voice assistant should not confirm ticket creation before the backend workflow had completed successfully.

The intended sequence is:

```text
Caller confirms request
        ↓
Retell sends structured payload
        ↓
Make.com processes request
        ↓
Ticket created
        ↓
Result returned
        ↓
Retell confirms success
```

This prevents the AI from saying that a ticket exists when the backend has not actually created one.

The ticket ID returned by the workflow acts as the confirmation that the process succeeded.

---

# End-of-Call Testing

The end of the call also needed testing.

An earlier version used a final conversation step before ending the call.

This could make the system wait unnecessarily for another caller response.

The closing behaviour was changed so that the final confirmation is delivered as part of the end-call stage.

The intended result is:

```text
Ticket result returned
        ↓
Final confirmation spoken
        ↓
Goodbye message
        ↓
Call ends immediately
```

This creates a cleaner end to the interaction.

---

# Form and Phone Workflow Consistency

The phone and form routes collect information differently, but both need to produce the same type of maintenance request.

I tested that both routes passed equivalent fields into the ticket workflow.

For example:

```text
Phone route
        ↓
site_id
system_type
reported_fault
caller_name
callback information
```

and:

```text
Form route
        ↓
site_id
system_type
reported_fault
caller_name
callback information
```

This helped validate the shared-backend design.

---

# What I Learned From Testing

One of the main lessons from this project was that testing an integrated workflow is different from testing one application on its own.

A visible failure may not be caused by the component where the failure appears.

For example:

```text
Email error
```

was actually caused by:

```text
Incorrect spreadsheet field mapping
```

and:

```text
Duplicate tickets
```

were actually caused by:

```text
Duplicate triggers
```

The most useful debugging approach was to follow the data or event through the system step by step.

```text
Where did the data start?
        ↓
What value was captured?
        ↓
What was sent?
        ↓
What was received?
        ↓
What action happened next?
        ↓
Where did the behaviour first become incorrect?
```

This made it easier to identify the real cause rather than only fixing the visible symptom.

---

# Production Testing Considerations

If the prototype moved towards production, I would expand testing to include:

- invalid or unknown sites
- missing required fields
- webhook failures
- email provider failures
- retry behaviour
- duplicate-request protection
- API timeouts
- concurrent submissions
- malformed payloads
- permission failures
- telephony outages
- monitoring and alerting
- recovery after partial workflow failure

The prototype testing focused mainly on proving the core end-to-end process and understanding how the connected components behaved.
