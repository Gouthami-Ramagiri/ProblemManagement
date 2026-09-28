# Problem Management System

A ServiceNow-based Problem Management application designed to manage problems, control problem lifecycle behavior, automate problem task creation, and improve form usability through UI Policies, UI Actions, Business Rules, and Flow Designer.

## Project Overview

This project implements a customized Problem Management workflow in ServiceNow.

The application provides functionality for:

- Automatic problem number generation
- Impact and urgency guidance using help icons
- Read-only priority
- Conditional configuration item validation
- Customized Save & Exit functionality
- Automatic problem task creation
- Manual problem task creation
- Save button customization
- Conditional field visibility
- Capturing the initial assignment group
- Problem lifecycle-based form behavior

The implementation uses ServiceNow platform features such as:

- Tables
- Dictionary Entries
- UI Policies
- UI Policy Actions
- UI Actions
- Business Rules
- Flow Designer
- GlideRecord
- Form conditions and scripting


## Key Requirements

### 1. Automatic Problem Number Generation

The system automatically generates the problem number using a predefined number format.

The custom Problem table uses an auto-number configuration with a prefix.
### 2. Impact Help Icon

A help icon is displayed beside the Impact field to guide users when selecting the appropriate impact.

The form decoration is added using:
```JavaScript
function onLoad() {
    g_form.addDecoration(
        'impact',
        'icon-help',
        'please select correct impact'
    );
}
```
### 3. Urgency Help Icon

A similar help icon is added to the Urgency field.
```JavaScript
function onLoad() {
    g_form.addDecoration(
        'impact',
        'icon-help',
        'please select correct impact'
    );

    g_form.addDecoration(
        'urgency',
        'icon-help',
        'please select correct urgency'
    );
}
```
### 4. Priority Field

The Priority field is configured as read-only.

This prevents users from manually modifying the priority value.

The configuration is implemented through the Dictionary Entry for the Priority field.

### 5. Configuration Item Mandatory During Root Cause Analysis

When the Problem State reaches Root Cause Analysis, the Configuration Item becomes mandatory.

This behavior is implemented using a UI Policy.

### UI Policy Behavior
```
Condition:
State = Root Cause Analysis

Field:
Configuration item

Action:
Mandatory = True
```
### 6. Save & Exit Button

The standard Update button is customized to provide Save & Exit behavior.

The button should not be displayed when the Problem State is Closed.

The existing Update UI Action is cloned and modified rather than directly changing the original system action.

The additional condition is:
```javascript
current.state != '107'
```
This prevents the button from being displayed when the problem is in the Closed state.

### 7. Automatic Problem Task Creation

When a problem reaches the Assess state and the user saves the record, a new Problem Task is automatically created.

This provides automatic task generation as part of the Problem Management workflow.

*Business Rule Logic*
```javascript
(function executeRule(current, previous /*null when async*/) {

    if (current.state.changesTo('assess')) {

        var prbtask = new GlideRecord('problem_task');

        prbtask.initialize();

        prbtask.number = current.sys_id;
        prbtask.short_description = 'creating problem task';

        prbtask.insert();
    }

})(current, previous);
```
*Behavior*
```
Problem State
     |
     v
   Assess
     |
     v
User clicks Save
     |
     v
Business Rule executes
     |
     v
Problem Task created
```
### 8. Manual Problem Task Creation

Users can also manually create a Problem Task from the Problem Tasks related list.

The Problem Tasks section contains a New button that allows users to create tasks manually.

Therefore, the project supports both:
```text
Automatic Task Creation
        +
Manual Task Creation
```
### 9. Flow Designer Implementation

The automatic Problem Task creation can also be implemented using ServiceNow Flow Designer.

#### *Trigger*

The flow is triggered when the Problem record is updated and:
```
State is Assess
```
#### *Trigger Configuration*
```text
Trigger:
Record Updated

Table:
POC_Problem_Table

Condition:
State is Assess

Run Trigger:
Once
```
#### *Action*

The flow uses a Create Record action to create a Problem Task.

The trigger record provides values that can be mapped into the newly created record.

Example fields shown in the implementation include:
```
Number
Description
Configuration Item
Assignment Group
```
This provides a low-code alternative to implementing the same automation through a Business Rule.

### 10. Save Button

A Save button is required at the top of the Problem form.

The Save action should:
- Save the record
- Keep the user on the same page
- Work for new records

Instead of creating a completely new UI Action, the existing ServiceNow "Save and stay" action can be copied and configured for the application.

Example action configuration:
```
Name:
Save and stay

Action name:
sysverb_update_and_stay

Form button:
Enabled

Show Insert:
Enabled

Active:
Enabled
```
This approach demonstrates reuse of existing ServiceNow UI functionality.

### 11. Conditional Description Visibility

The Description field should only become visible when the Problem Statement contains a value.

#### *Condition*
```
Problem statement is not empty
```
#### *UI Policy Action*
```
Field:
Description

Visible:
True
```
Therefore:
```
Problem Statement Empty
        |
        v
Description Hidden


Problem Statement Contains Value
        |
        v
Description Visible
```
### 12. Initial Assignment Group

The application maintains a custom field called:

```
Initial assignment group
```
The purpose of this field is to capture the first Assignment Group selected on the Problem record.

Once captured, the initial value should not be overwritten when the Assignment Group changes later.

### 13. Initial Assignment Group Business Rule

A Business Rule is used to capture the first Assignment Group value.

#### Condition

The Business Rule executes when:
```
Assignment Group has a value
AND
Initial Assignment Group is empty
```
```JavaScript
(function executeRule(current, previous /*null when async*/) {

    if (
        current.assignment_group &&
        current.u_initial_assignment_group.nil()
    ) {
        current.u_initial_assignment_group =
            current.assignment_group;
    }

})(current, previous);
```
Example

Initial Assignment Group:
```
Database Administration
```
If the Assignment Group later changes to:
```
Application Support
```
the Initial Assignment Group remains:
```
Database Administration
```
This preserves the original assignment information.

### ServiceNow Components Used
| Component	| Purpose |
| ------- | --------- |
| Custom Table | 	Stores Problem records | 
| Auto Number | 	Generates Problem numbers | 
| Dictionary Entry | 	Makes Priority read-only | 
| UI Policy	 | Controls field behavior | 
| UI Policy Action | 	Controls visibility and mandatory state | 
| UI Action	 | Implements Save/Save & Exit behavior | 
| Business Rule	 | Automates Problem Task creation | 
| Business Rule | 	Captures Initial Assignment Group | 
| Flow Designer | 	Provides low-code task automation  | 
| GlideRecord | 	Creates Problem Task records | 
| Form Decoration	 | Adds help icons to | 
## Project Workflow

The overall Problem Management workflow can be represented as:
```text
Create Problem
      |
      v
System Generates Problem Number
      |
      v
Enter Problem Information
      |
      v
Impact / Urgency Selection
      |
      v
Priority Controlled by System
      |
      v
Assignment Group Selected
      |
      v
Initial Assignment Group Captured
      |
      v
Problem State = Assess
      |
      +----------------------+
      |                      |
      v                      v
Save Problem          Manual Task Creation
      |                      |
      v                      v
Automatic Task         Problem Task
Creation                   |
      |                    |
      +---------+----------+
                |
                v
       Problem Lifecycle
                |
                v
      Root Cause Analysis
                |
                v
 Configuration Item Mandatory
                |
                v
             Closed
                |
                v
      Save & Exit Hidden
```
## Business Rules
### Business Rule 1: Create Problem Task
#### Purpose

Automatically create a Problem Task when a Problem enters the Assess state.

#### Condition
```
Problem State changes to Assess
```
#### Action
```
Create Problem Task
```
### Business Rule 2: Capture Initial Assignment Group
#### Purpose

Store the first Assignment Group selected for a Problem.

#### Condition
```
Assignment Group is not empty
AND
Initial Assignment Group is empty
```
#### Action
Initial Assignment Group = Assignment Group
### UI Policies
#### Configuration Item Mandatory
```
When:
State = Root Cause Analysis

Then:
Configuration Item = Mandatory
```
#### Description Visibility
```
When:
Problem Statement is not empty

Then:
Description = Visible
```
### UI Actions

The project uses UI Actions to customize the Problem form.

*Save*
```
Save
|
+-- Saves the current record
+-- Remains on the same page
```
*Save & Exit*
```
Save & Exit
|
+-- Saves the current record
+-- Exits the current form
+-- Hidden when State = Closed
```
### Flow Designer

The task automation can be implemented using Flow Designer.
```
Trigger
  |
  | Problem record updated
  |
  v
Condition
  |
  | State = Assess
  |
  v
Create Record
  |
  v
Problem Task
```
This provides an alternative to the Business Rule implementation.

## Technologies

- ServiceNow
- JavaScript
- GlideRecord
- Business Rules
- UI Policies
- UI Actions
- Flow Designer
- Custom Tables
- Dictionary Entries
## Project Structure

A logical ServiceNow implementation can be organized as:
```
Problem Management
|
+-- Custom Problem Table
|     |
|     +-- Problem Number
|     +-- Impact
|     +-- Urgency
|     +-- Priority
|     +-- Configuration Item
|     +-- Problem Statement
|     +-- Description
|     +-- Assignment Group
|     +-- Initial Assignment Group
|     +-- State
|
+-- UI Policies
|     |
|     +-- Configuration Item Mandatory
|     +-- Description Visibility
|
+-- UI Actions
|     |
|     +-- Save
|     +-- Save & Exit
|
+-- Business Rules
|     |
|     +-- Create Problem Task
|     +-- Capture Initial Assignment Group
|
+-- Flow Designer
|     |
|     +-- Problem Task Creation
|
+-- Dictionary Entries
      |
      +-- Priority Read Only
```
## Key Learning Outcomes

This project demonstrates practical implementation of ServiceNow Problem Management functionality, including:

- Creating and configuring custom tables
- Configuring auto-generated numbers
- Working with Dictionary Entries
- Creating UI Policies
- Creating UI Policy Actions
- Customizing UI Actions
- Writing Client-side JavaScript
- Writing Business Rules
- Using GlideRecord
- Creating records programmatically
- Implementing conditional field behavior
- Capturing field history through Business Rules
- Building automation using Flow Designer
- Reusing existing ServiceNow UI Actions
## Conclusion

The Problem Management System demonstrates how ServiceNow platform capabilities can be combined to create a structured and automated Problem Management solution.

The implementation focuses on form customization, lifecycle-based validation, task automation, field visibility, assignment tracking, and reusable ServiceNow configurations.

The project provides two approaches for automated Problem Task creation:

Business Rule using GlideRecord
Flow Designer using a Create Record action
