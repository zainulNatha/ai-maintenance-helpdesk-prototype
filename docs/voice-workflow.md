# Voice AI Workflow

## Overview

The phone route is the main conversational part of the prototype.

I used Retell AI to handle the maintenance call and guide the caller through a structured intake process.

The aim was not simply to have an AI answer the phone.

The important part was making sure the conversation collected the right information, validated the site, allowed corrections and only confirmed a ticket once the backend automation had successfully created it.

At a high level, the voice workflow is:

```text
Caller phones maintenance number
        ↓
AI answers the call
        ↓
Site is identified
        ↓
Site is validated
        ↓
Maintenance issue is collected
        ↓
Caller details are collected
        ↓
Information is reviewed
        ↓
Caller can correct anything that is wrong
        ↓
Confirmed request is sent to Make.com
        ↓
Ticket is created
        ↓
Ticket reference is returned
        ↓
Caller receives confirmation
        ↓
Call ends
```

---

# Main Goal of the Voice Workflow

The main goal was to turn a natural phone conversation into structured maintenance data.

A caller should not need to know technical field names or internal IDs.

For example, the caller may simply say:

> “I’m calling from the Romford branch. One of the CCTV cameras at the front entrance isn’t showing anything.”

The workflow then needs to identify things such as:

```text
Site
→ Romford branch

System Type
→ CCTV

Reported Fault
→ Front entrance camera not displaying

Site Impact
→ Front entrance cannot be monitored
```

The caller can speak naturally while the system extracts the information required for the maintenance ticket.

---

# 1. Opening the Call

The AI begins by asking the caller which site they are contacting from.

The caller is not expected to know an internal Site ID.

Instead, they can provide a normal site or branch name.

For example:

```text
"Demo Academy North"
```

or:

```text
"Demo School Romford"
```

The system then extracts the information needed for the site lookup.

---

# 2. Extracting Site Information

The first extraction step separates the site information provided by the caller.

The main values are:

```text
site_name_spoken
branch_name_spoken
```

This is useful where one organisation may have several locations.

For example:

```text
Caller:
"Demo Academy North"

        ↓

Extracted information:

Organisation
→ Demo Academy

Branch
→ North
```

The extracted values are then sent to the site lookup workflow.

---

# 3. Site Lookup

The voice workflow calls a Make.com site lookup automation.

Conceptually:

```text
Retell
   ↓
Send site details
   ↓
Make.com
   ↓
Search Sites table
   ↓
Matching record found
   ↓
Return official site information
   ↓
Retell
```

The returned information can include:

```text
Site ID
Customer / Organisation
Official Site Name
Customer Email
```

The important point is that the AI does not create or guess this information.

It is retrieved from maintained reference data.

---

# 4. Confirming the Site

Once the lookup returns a match, the AI confirms the official site name with the caller.

For example:

> “I’ve located Demo Academy North. Is that the correct site?”

This gives the caller a chance to confirm that the system matched the right location.

If the site is wrong, the caller can correct it and the lookup process runs again.

---

# Site Correction Loop

A correction can follow this pattern:

```text
AI identifies site
        ↓
AI confirms site
        ↓
Caller says it is incorrect
        ↓
Caller provides corrected site
        ↓
Site details are extracted again
        ↓
Site lookup runs again
        ↓
Correct site is returned
        ↓
AI confirms the corrected site
```

This was important because a conversational system should not simply continue after an incorrect match.

The caller needs a way to correct the information before a ticket is created.

---

# 5. Collecting the Maintenance Issue

Once the site has been confirmed, the AI asks the caller to describe the maintenance issue.

The caller can explain the problem naturally.

For example:

> “The camera at the front entrance isn’t displaying anything.”

The system then extracts structured maintenance information from that description.

---

# 6. Extracting Maintenance Details

The maintenance extraction stage creates fields such as:

```text
system_type
reported_fault
fault_message
site_impact
```

The `system_type` is categorised into a controlled set of values.

Examples include:

- Intercom
- CCTV
- Fire Alarm
- Access Control
- Intruder Alarm
- Other

This helps keep the resulting ticket data consistent.

---

# Example: Natural Language to Structured Fault Data

A caller may say:

> “The front camera isn’t working and we can’t see anyone coming through the entrance.”

The system can interpret that as:

```text
System Type
→ CCTV

Reported Fault
→ Front entrance camera not working

Fault Message
→ None reported

Site Impact
→ Entrance cannot be monitored
```

This is one of the main examples of turning unstructured information into structured data.

---

# 7. System Type Mapping

The voice workflow maps common phrases into the relevant maintenance category.

Examples include:

```text
"door entry"
"entry phone"
        ↓
Intercom
```

```text
"camera"
"video"
        ↓
CCTV
```

```text
"fire panel"
"fire system"
        ↓
Fire Alarm
```

```text
"card"
"fob"
"door access"
        ↓
Access Control
```

```text
"burglar alarm"
"intruder alarm"
"alarm"
        ↓
Intruder Alarm
```

This means the caller does not have to know the exact category name used in the ticket register.

---

# 8. Collecting Caller Details

The AI then collects:

```text
caller_name
```

and determines which telephone number should be used for the callback.

The caller can either:

```text
Use the number they are currently calling from
```

or:

```text
Provide a different callback number
```

The workflow stores values such as:

```text
caller_name

use_calling_number

incoming_caller_number

alternate_callback_phone
```

---

# Callback Logic

Conceptually:

```text
Caller wants to use current number
        ↓
use_calling_number = true
        ↓
Use incoming caller number
```

or:

```text
Caller provides different number
        ↓
use_calling_number = false
        ↓
Use alternate callback number
```

This means the ticket contains a usable contact number without forcing the caller to repeat the number they are already calling from.

---

# 9. Reviewing the Request

Before the ticket is created, the AI reviews the information with the caller.

The purpose of this step is to make sure the structured data matches what the caller intended to report.

The review can include:

- site
- system type
- reported fault
- caller name
- callback details

The caller can either confirm the information or ask for something to be corrected.

---

# 10. Correcting Ticket Details

If the caller says something is wrong, the workflow enters a correction step.

The corrected information is extracted and the review is shown again.

Conceptually:

```text
AI reviews request
        ↓
Caller identifies incorrect information
        ↓
Corrected details are collected
        ↓
Structured fields are updated
        ↓
AI reviews the request again
```

This means the request is not submitted until the caller is satisfied with the information.

---

# 11. Creating the Maintenance Ticket

Once the caller confirms the request, Retell calls the main Make.com ticket creation workflow.

The request contains structured fields such as:

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

The information is sent as a structured payload.

---

# Example Payload Structure

A simplified example looks like:

```json
{
  "site_id": "SITE-001",
  "customer_name": "Example Organisation",
  "site_name": "Example Organisation North",
  "customer_email": "example@example.com",
  "system_type": "CCTV",
  "reported_fault": "Front entrance camera not displaying",
  "fault_message": "None reported",
  "site_impact": "Entrance cannot be monitored",
  "caller_name": "Example Caller",
  "use_calling_number": "true"
}
```

This example is fictional and does not contain live customer information.

---

# 12. Waiting for the Backend Result

An important design decision was that the AI should not immediately tell the caller that the ticket has been created.

Retell waits for the Make.com workflow to finish processing the request.

While this happens, the caller can hear a short message such as:

> “Thank you. I’m creating your maintenance ticket now. This may take a few seconds.”

This makes the processing delay feel intentional rather than making the caller think the call has stopped responding.

---

# Why the AI Waits

Without waiting for the backend result, the conversation could incorrectly say:

```text
"Your ticket has been created."
```

even if:

- the automation failed
- the ticket was not added to the register
- the email process failed
- no ticket reference was generated

Instead, the workflow follows:

```text
Caller confirms request
        ↓
Retell sends request to Make
        ↓
Make creates ticket
        ↓
Make returns result
        ↓
Retell confirms success
```

This makes the confirmation dependent on the real backend outcome.

---

# 13. Ticket Result

The backend can return values such as:

```text
ticket_created
ticket_id
customer_confirmation_sent
```

The ticket reference can then be included in the closing confirmation.

For example:

> “Your maintenance ticket is [ticket reference]. An engineer will contact you within one hour.”

The one-hour wording refers to engineer contact rather than engineer arrival.

---

# 14. Ending the Call

Once the successful ticket result has been returned, the final confirmation is spoken and the call ends.

The closing message confirms:

- the ticket has been created
- the ticket reference
- that an engineer will make contact
- that a confirmation email has been sent

The call then terminates without waiting for another response from the caller.

---

# Security Boundaries

Because the workflow may receive maintenance calls relating to security and life-safety systems, I deliberately restricted the AI's role.

The AI is used for maintenance intake, not for detailed troubleshooting.

It must not provide:

- alarm PINs
- engineer codes
- disarm codes
- passwords
- security credentials
- instructions for bypassing a security system
- instructions for disabling or defeating security equipment
- instructions for defeating life-safety equipment

The AI should also avoid guessing missing information.

If required information cannot be validated, the safer behaviour is to ask the caller for clarification rather than inventing an answer.

---

# Recording Notice

The voice workflow includes a notice that the call may be recorded and transcribed for maintenance records and service quality.

This provides transparency around the use of call recording and transcription in the prototype.

Any public portfolio material should avoid publishing real customer recordings or transcripts.

---

# What the AI Does vs What the Backend Does

I found it useful to separate these responsibilities.

## Retell AI

Retell is responsible for:

```text
Conversation
Understanding natural language
Extracting structured fields
Asking follow-up questions
Confirming information
Allowing corrections
Calling backend functions
Speaking the final result
```

## Make.com

Make.com is responsible for:

```text
Site lookup
Ticket creation
Ticket ID generation
Writing to Google Sheets
Reading settings
Routing notifications
Sending emails
Returning the ticket result
```

This separation means the conversational layer does not need to directly handle every backend action.

---

# Why This Workflow Was Useful

The voice workflow combines conversational flexibility with structured processing.

The caller can speak naturally, while the business still receives consistent data.

Conceptually:

```text
Natural conversation
        ↓
AI understanding
        ↓
Structured fields
        ↓
Reference-data validation
        ↓
Caller confirmation
        ↓
Backend automation
        ↓
Structured ticket
```

This was one of the main goals of the prototype.

---

# What I Learned

This part of the project helped me understand that designing a voice AI workflow involves more than writing a prompt.

The conversation needs to account for:

- missing information
- incorrect site identification
- corrections
- structured data extraction
- reference-data validation
- backend delays
- failed automation
- safe confirmation behaviour
- security boundaries
- clean call termination

The most useful learning was separating the conversation layer from the backend workflow.

Retell handles the interaction with the caller, while Make.com and the data layer handle the business process behind it.
