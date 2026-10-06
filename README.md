# ServiceNow Loaner Laptop Management

An end-to-end ServiceNow application for managing company loaner laptop requests, approvals, device assignments, availability, and returns.

## Overview

The Loaner Laptop Desk application provides a controlled workflow for employees to request temporary loaner laptops.

The application automates:

- Loaner laptop requests through Service Catalog
- Requester and manager population
- Loan-period validation
- Manager approval
- Dynamic laptop availability
- Loan assignment creation
- Return tracking
- Request closure
- Role-based access and security

## Key Features

### Service Catalog

**Catalog Item:** Loaner Laptop Request

Users provide:

- Requester
- Manager
- Loan start date
- Loan end date
- Loaner device
- Reason for loan

### Automated Approval Workflow

The request follows:

`Catalog Request → Manager Approval → Loan Assignment → Active Loan → Return → Closure`

Manager approval is handled using Flow Designer.

### Dynamic Device Availability

The loaner device reference field dynamically excludes laptops that are currently assigned through an active loan.

This prevents the same laptop from being assigned to multiple active loans.

Implemented using:

- GlideRecord
- Script Include
- Reference Qualifier
- Dynamic query generation

### Loan Assignment Management

A custom `u_loan_assignment` table tracks each loan.

Lifecycle states:

- Requested
- Active
- Returned
- Cancelled

### Client-Side Validation

Catalog Client Scripts handle:

- Loan date validation
- Requester population
- Manager population
- Extended-loan justification

### Security

The application includes:

- Custom roles
- ACLs
- User Criteria
- Application menu
- Loan Assignment module

## Technical Implementation

| Component | Implementation |
|---|---|
| Service Catalog | Loaner Laptop Request |
| Flow Designer | Request and Return workflows |
| Custom Table | `u_loan_assignment` |
| Scripting | JavaScript / GlideRecord |
| Script Include | `LoanerDeviceFilter` |
| Reference Qualifier | Dynamic device filtering |
| Client Scripts | Request validation and field automation |
| Security | Roles, ACLs and User Criteria |
| Update Set | Loaner Laptop Desk |

## Architecture

```text
Employee
   ↓
Service Catalog
   ↓
Loaner Laptop Request
   ↓
Catalog Client Scripts
   ↓
Flow Designer
   ↓
Manager Approval
   ↓
Loan Assignment
   ↓
Active Loan
   ↓
Return
   ↓
Request Closure

Dynamic Device Filtering

The application identifies currently active loan assignments and excludes their associated devices from the catalog reference field.

Example:
var LoanerDeviceFilter = Class.create();
LoanerDeviceFilter.prototype = {
    initialize: function() {},

    excludeLoanedDevices: function() {
        var loanedIds = [];

        var gr = new GlideRecord('u_loan_assignment');
        gr.addQuery('u_status', 'active');
        gr.addNotNullQuery('u_loaner_device');
        gr.query();

        while (gr.next()) {
            loanedIds.push(gr.getValue('u_loaner_device'));
        }

        var query = 'nameSTARTSWITHLOANER-LAPTOP-';

        if (loanedIds.length > 0) {
            query += '^sys_idNOT IN' + loanedIds.join(',');
        }

        return query;
    },

    type: 'LoanerDeviceFilter'
};

Flow Designer
Loaner Laptop Request
Service Catalog request is submitted
Catalog variables are retrieved
Manager approval is requested
Approved requests create a Loan Assignment
Assignment remains active until the laptop is returned
Return workflow updates the request

Rejected or failed requests are handled through dedicated flow branches.

Loaner Laptop Return

The return process updates the loan assignment and completes the associated request workflow.

Screenshots
Flow Designer

Update Set

Installation

This project can be imported into a ServiceNow personal developer instance using the included Update Set XML.

Steps
Download Loaner_Laptop_Desk.xml
Open a ServiceNow instance
Navigate to System Update Sets → Retrieved Update Sets
Import the XML
Preview the Update Set
Resolve any instance-specific conflicts if required
Commit the Update Set
Verify the application, catalog item, flows, scripts, roles, and tables
Project Files
Loaner_Laptop_Desk.xml - ServiceNow Update Set export
README.md - Project documentation
project-details.md - Detailed project information
flow-details.md - Flow Designer documentation
UPDATE-SET-EXPORT.md - Update Set information
screenshots/ - Project screenshots
Technologies
ServiceNow
Service Catalog
Flow Designer
JavaScript
GlideRecord
Script Includes
Catalog Client Scripts
Reference Qualifiers
ACLs
Roles
User Criteria
Update Sets
Future Enhancements
Automated email notifications
Scheduled overdue-return notifications
Laptop condition tracking
Automated availability dashboard
Asset lifecycle integration
Reporting and analytics
Automated return reminders
Project Purpose

This project demonstrates practical ServiceNow development skills across application configuration, scripting, automation, Service Catalog, workflow design, dynamic reference filtering, and platform security.
