# System Architecture

## Overview

The prototype was designed so that customers can report a maintenance issue either by phone or through an online form.

Both routes eventually feed into the same ticket creation workflow.

This was important because I did not want to build two completely separate processes for phone and form requests.

At a high level, the system works like this:

Phone caller  
→ AI voice assistant  
→ Site validation  
→ Maintenance details captured  
→ Automation workflow  
→ Ticket created  
→ Emails sent  
→ Ticket stored in the register

Google Form  
→ Form submission  
→ Google Apps Script  
→ Automation workflow  
→ Ticket created  
→ Emails sent  
→ Ticket stored in the register

## Main components

### Retell AI

Retell AI handles the phone conversation.

Its role is to:

- ask which site the caller is contacting from
- identify and confirm the correct site
- collect the maintenance issue
- classify the affected system
- collect caller and callback details
- allow the caller to correct information
- send the confirmed request to the automation workflow
- receive the generated ticket ID
- confirm the ticket to the caller

The AI does not create the ticket itself.

Instead, it collects and structures the information before passing it to the automation layer.

### Make.com

Make.com acts as the main automation layer.

It receives structured information from either the AI phone workflow or the Google Form workflow.

It then coordinates the rest of the process, including:

- generating the ticket information
- adding the ticket to Google Sheets
- reading business settings
- sending notification emails
- applying conditional routing such as Away Mode
- returning confirmation information

### Google Sheets

Google Sheets is used as a lightweight data store for the prototype.

The main datasets are:

- Sites - approved customer and site information
- Tickets - maintenance ticket register
- Settings - configurable business settings
- Form Responses - raw Google Form submissions

For a production system, this could be replaced with a database, CRM or service management platform.

### Google Forms

Google Forms provides an alternative way to report a maintenance issue.

The form collects information such as:

- site
- affected system
- fault description
- fault or error message
- site impact
- caller name
- callback telephone number

The form does not directly create a ticket.

Instead, a Google Apps Script reads the submitted information, looks up the official site information, and sends the request into the same Make.com workflow used by the AI voice assistant.

## Why this architecture was useful

A key design decision was to make both the phone and form routes produce the same type of structured maintenance request.

This means the downstream automation does not need a completely different process depending on where the request came from.

In simple terms:

Phone request  
→ structured maintenance data

Form request  
→ structured maintenance data

Both  
→ same ticket workflow

This makes the prototype easier to maintain and extend.
