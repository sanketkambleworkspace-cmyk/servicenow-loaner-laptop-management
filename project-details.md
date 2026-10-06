# ServiceNow Loaner Laptop Management

An end-to-end ServiceNow application for managing company loaner laptop requests, approvals, device assignment, availability, and returns.

> **Portfolio / Capstone Project**  
> Platform: ServiceNow  
> Application: Loaner Laptop Desk

## Project Overview

The application allows users to request a loaner laptop through the Service Catalog. The solution manages the request lifecycle from catalog submission and manager approval through loan assignment, return tracking, and request closure.

A key feature is dynamic device availability: laptops that currently have an **Active** loan assignment are automatically removed from the Loaner Device selection.

## Architecture

```text
User
  |
  v
Service Catalog: Loaner Laptop Request
  |
  +--> Catalog Client Scripts
  |      - Requester population
  |      - Manager population
  |      - Date validation
  |      - >14-day reason validation
  |
  +--> Advanced Reference Qualifier
  |      |
  |      v
  |   LoanerDeviceFilter
  |      |
  |      v
  |   u_loan_assignment
  |      |
  |      +--> Status = Active
  |      |
  |      v
  |   Exclude actively loaned devices
  |
  v
Flow Designer
  |
  +--> Manager Approval
  +--> Create Loan Assignment
  +--> Wait for Returned
  +--> Loaner Laptop - Return
  +--> Update Requested Item
```

## Main Components

### Service Catalog
- Category: Hardware
- Catalog Item: Loaner Laptop Request
- Variables:
  - Requester
  - Manager
  - Loan Start Date
  - Loan End Date
  - Loaner Device
  - Reason

### Custom Table

`u_loan_assignment`

Fields include:
- Loaner Device
- Requested For
- Start Date
- End Date
- Return Date
- Status

Status values:
- Requested
- Active
- Returned
- Cancelled

### Flow Designer

**Loaner Laptop Request**

Lifecycle:

`Request → Manager Approval → Loan Assignment → Wait for Return → Return Processing → Closure`

### Script Include

`LoanerDeviceFilter`

The Script Include queries `u_loan_assignment` for active assignments and builds a dynamic reference qualifier.

```javascript
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
```

Reference Qualifier:

```javascript
javascript:new LoanerDeviceFilter().excludeLoanedDevices()
```

## Loaner Device Availability

The application currently uses the naming convention:

`LOANER-LAPTOP-01` through `LOANER-LAPTOP-08`

The reference qualifier first identifies these loaner computers and then excludes any device referenced by a Loan Assignment whose status is `active`.

This avoids hardcoding individual device sys_ids.

## Security

The application includes:
- Roles
- Access Controls
- User Criteria
- Application navigation
- Loan assignment access restrictions

Relevant roles include:
- `u_loan_assignment_user`
- `loaner_admin`

## Automation

The project includes:
- Loaner Laptop - Request flow
- Loaner Laptop - Return flow
- Catalog Client Scripts
- Script Includes
- Dynamic Reference Qualifier

## Screenshots

### Flow Designer

![Loaner Laptop Request Flow](screenshots/flow-loaner-laptop-request.png)

### Update Set

![Update Set - Part 1](screenshots/update-set-01.png)

![Update Set - Part 2](screenshots/update-set-02.png)

![Update Set - Part 3](screenshots/update-set-03.png)

## Update Set

The complete application is currently captured in the ServiceNow Update Set:

**Loaner Laptop Desk**

The Update Set contains approximately 59 customer updates covering the application's configuration and development artifacts.

> The Update Set XML can be added to this repository after exporting it from the ServiceNow instance.

## Technologies / ServiceNow Features

- ServiceNow Service Catalog
- Flow Designer
- GlideRecord
- Script Includes
- Catalog Client Scripts
- Advanced Reference Qualifiers
- Custom Tables
- Dictionary / Field configuration
- ACLs
- Roles
- User Criteria
- Application Menus and Modules
- Update Sets

## Portfolio Notes

This project demonstrates practical ServiceNow development across both client-side and server-side configuration, workflow automation, data modeling, security, and dynamic reference filtering.
