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

## Main technologies used

- Retell AI - voice-based conversational intake
- Make.com - workflow automation and orchestration
- Google Sheets - prototype site register, ticket register and settings
- Google Forms - alternative maintenance request channel
- Google Apps Script - connects form submissions to the automation workflow
- Webhooks and JSON - used to pass structured data between systems

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

All examples and screenshots in this repository will use anonymised or fictional data.
