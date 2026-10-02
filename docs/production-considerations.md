# Production Considerations

## Overview

This project was built as a personal proof of concept rather than a production system.

The aim was to prove that the overall process could work:

```text
Customer request
        ↓
AI or form intake
        ↓
Validation
        ↓
Structured data
        ↓
Ticket creation
        ↓
Notifications
        ↓
Tracking
```

The prototype uses lightweight tools because they allowed me to build and test the complete workflow quickly.

If the solution were moved into a live production environment, I would review several areas before relying on it for day-to-day business operations.

---

# 1. Data Storage

Google Sheets works well for a prototype because it is:

- quick to set up
- easy to inspect
- easy to edit
- easy to connect to automation tools

For a larger production system, I would consider replacing the ticket register with a more scalable data platform.

Possible options could include:

- a relational database
- a CRM
- a help-desk platform
- a field-service management platform
- a cloud data platform

The general data model could remain similar:

```text
Sites
Tickets
Settings
Users
Status history
```

The main difference would be stronger control over:

- data integrity
- concurrent updates
- permissions
- audit history
- scalability
- reporting

---

# 2. Authentication and Access Control

The prototype relies on access to tools such as Google Sheets, Make.com and the connected email services.

In production, I would define more clearly:

- who can view tickets
- who can update ticket status
- who can change settings
- who can access customer details
- who can manage the automation
- who can access call recordings or transcripts

This could be handled through role-based access control.

For example:

```text
Administrator
→ manage configuration and integrations

Engineer
→ view and update assigned tickets

Office staff
→ view and manage maintenance requests

Customer
→ receive only their own ticket information
```

---

# 3. Secrets Management

The working integrations require private values such as:

- webhook URLs
- API credentials
- service tokens
- telephony credentials

These should not be stored in public repositories or exposed in screenshots.

For a production implementation, secrets should be stored using a secure configuration or secrets-management service.

The application or automation should retrieve them securely when needed.

---

# 4. Monitoring

A production system should make it easy to see when something has failed.

For example:

```text
Ticket request received
        ↓
Ticket creation fails
        ↓
Alert generated
        ↓
Someone investigates
```

Useful monitoring could include:

- failed webhook requests
- failed email sends
- unsuccessful site lookups
- telephony errors
- API timeouts
- unusually high failure rates
- workflows that stop before completion

The prototype mainly relied on manual inspection while testing.

A production implementation should provide more proactive monitoring.

---

# 5. Logging

Logging provides a record of what the system has done.

Useful events to log could include:

```text
Call received
Site matched
Ticket request submitted
Ticket created
Email sent
Status updated
Automation failed
```

Logs can help answer questions such as:

- When was the request received?
- Which workflow processed it?
- Was the customer confirmation sent?
- Where did a failure occur?
- Was the request retried?

This becomes increasingly important as the system grows.

---

# 6. Error Handling and Retry Logic

The prototype includes some error handling, but a production system would need a clearer strategy for partial failures.

For example:

```text
Ticket created successfully
        ↓
Customer email fails
```

The system should not necessarily create a second ticket just because one notification failed.

Instead, it may need to retry only the failed action.

Possible production behaviour could include:

```text
Ticket created
        ↓
Email fails
        ↓
Keep existing ticket
        ↓
Retry email
        ↓
Log retry attempt
```

This would reduce the risk of duplicate records.

---

# 7. Duplicate Request Protection

During testing, I encountered duplicate ticket creation when two identical form triggers were active.

A production system should have additional protection against duplicate processing.

One approach would be to assign each incoming request a unique identifier.

For example:

```text
Request ID
→ FORM-123456
```

Before creating a ticket, the workflow could check:

```text
Has FORM-123456 already been processed?
```

If yes:

```text
Do not create another ticket
```

If no:

```text
Continue ticket creation
```

This is related to the concept of idempotency.

---

# 8. Validation

The production version should include stronger validation before creating a ticket.

Examples include:

- required fields are present
- site exists
- system type is allowed
- email addresses are valid
- callback numbers follow an expected format
- incoming payload contains the expected structure

This would reduce the chance of incorrect data entering the workflow.

---

# 9. Data Retention

The prototype captures information such as:

- customer details
- maintenance faults
- caller information
- call transcripts
- potentially call recordings

A production system would need a clear retention policy.

This should define:

- what information is stored
- why it is stored
- how long it is retained
- who can access it
- when it should be deleted

Call recordings and transcripts may require particularly careful handling.

---

# 10. Privacy

A production implementation should minimise the personal information collected.

The system should only capture data required for the maintenance process.

For example:

```text
Needed
→ caller name
→ callback number
→ maintenance issue

Not needed
→ unrelated personal information
```

Customer information should also only be made available to people who need it to carry out their role.

---

# 11. Telephony Resilience

The phone route depends on several connected services.

Conceptually:

```text
Caller
   ↓
Telephony provider
   ↓
AI voice platform
   ↓
Automation
```

A production design would need to consider what happens if one of these services is unavailable.

For example:

```text
AI service unavailable
        ↓
Route caller to voicemail or alternative number
```

A fallback process would help make sure maintenance requests are not completely lost during an outage.

---

# 12. AI Behaviour and Guardrails

Because this prototype can receive calls about security and life-safety systems, the AI should remain limited to maintenance intake.

The production version should continue to prevent the AI from:

- providing alarm codes
- revealing credentials
- explaining security bypass methods
- giving unsafe troubleshooting instructions
- inventing missing site data

The role of the AI should remain clear:

```text
Collect
Clarify
Validate
Structure
Create request
Confirm
```

rather than:

```text
Diagnose every technical fault
```

---

# 13. Human Escalation

Not every request should necessarily be handled entirely through automation.

A production process could include escalation rules.

For example:

```text
Unknown site
        ↓
Send to human review
```

or:

```text
Incomplete request
        ↓
Office staff follow-up
```

or:

```text
Critical life-safety issue
        ↓
Escalate immediately
```

This would allow automation to handle routine intake while humans remain involved where judgement is required.

---

# 14. Audit History

In the prototype, the ticket register contains the current status.

A production system could also keep a history of every change.

For example:

```text
10:15
Ticket created

10:20
Engineer acknowledged

10:45
Status changed to In Progress

12:10
Status changed to Resolved
```

This would make it easier to understand what happened to a ticket over time.

---

# 15. Resolution Workflow

The prototype focused mainly on the initial maintenance intake process.

A fuller production solution could extend the workflow after the engineer has completed the job.

For example:

```text
Ticket created
        ↓
Engineer assigned
        ↓
Work completed
        ↓
Resolution notes entered
        ↓
Ticket marked resolved
        ↓
Customer receives resolution confirmation
```

This would turn the prototype from an intake system into a more complete ticket lifecycle.

---

# 16. SMS and WhatsApp

The current prototype uses email notifications.

A future version could add channels such as:

- SMS
- WhatsApp

These could be useful for:

```text
New ticket alerts
Engineer updates
Customer confirmations
Resolution notifications
```

However, I would only add additional channels where they provide clear business value rather than adding them purely because the technology is available.

---

# 17. Reporting and Analytics

Once maintenance requests are stored as structured data, they can also be analysed.

Possible reporting could include:

- tickets by customer
- tickets by site
- tickets by system type
- most common reported faults
- average response times
- unresolved ticket volume
- repeat faults by location

This is one of the advantages of converting natural-language requests into structured records.

The same operational data used for ticketing can later support reporting and decision-making.

---

# 18. Scalability

The prototype was designed for a relatively small workflow.

If usage increased significantly, I would review:

- number of calls
- number of tickets
- workflow execution limits
- API rate limits
- spreadsheet performance
- concurrent submissions
- email sending limits
- telephony capacity

Scaling may require replacing some prototype tools while keeping the overall architecture.

---

# Prototype vs Production Summary

The prototype proves the main idea:

```text
Natural customer request
        ↓
Structured and validated data
        ↓
Automated business workflow
        ↓
Trackable maintenance ticket
```

A production version would require stronger controls around:

- security
- access
- monitoring
- resilience
- data storage
- duplicate protection
- auditability
- privacy
- operational support

The important point is that the prototype establishes the process and data flow.

The underlying architecture can then be strengthened depending on the scale and requirements of a real implementation.

---

# Key Learning

One of the most useful lessons from this project was understanding the difference between proving that an idea works and designing something that is ready for production.

For a proof of concept, the priority was:

```text
Can the process work end to end?
```

For production, the questions become broader:

```text
Is it secure?

Is it reliable?

Can failures be detected?

Can requests be retried safely?

Can access be controlled?

Can the system scale?

Can every action be audited?
```

Thinking about these differences helped me understand the wider considerations around moving an automation from prototype to operational use.
