# DevOps Alert Tamer (DaTamer)
Devops Alert Tamer is an open source project applying machine learning and ai techniques to tame the massive amounts of notifications sent to devops group emails.

## Terms
MSE - monitoring status email.  An email sent from a monitoring tool to provide a status of the system being monitored (e.g., daily backup status, disk space warning email, etc).

## The Problem
When a new employee starts with a DevOps group, they subscribe to a distribution list.  That subscription brings with it an avalanche of hundreds or thousands of monitoring status emails (MSEs) every day.  This is a problem in three ways:

* It's overwhelming to the new engineer who doesn't understand the architecture of the system and has a difficult time understanding the emails and how to organize them
* Each of the engineers on the team spends significant time each day to process the exact same information.  This is inefficient
* If a scheduled email doesn't come as expected, it's easy to miss that there is a problem

For the first two weeks the new engineer wades through the MSEs trying to understand what they are, which are noise and which are indications of issues to which they need to respond.  This process usually results in a large set of email rules that sort the emails into buckets that make it easier to work on sets of emails more efficiently.

The other members of the team have already gone through the process, but there is no way to share the knowlege expect by interrupting them, typically with another email, asking what each email means and if anyone is responding to it.  Thus the other engineers get a repeat of the notification (which they already processed) from the new engineer asking about it.

Additionally, a single event can generate multiple emails from different systems.  For example, a network glitch causing the NFS server to not respond in a timely fashion can generate emails from numerous servers.  It's left up to the engineer to interpret the flood of emails to identify event that needs to be addressed.

And to complicate matters, there are scheduled emails that are expected on a regular basis.  If the email doesn't come, it's an indication of a problem.  But, the lack of the email is not a visible indicator and the event can be missed until there is a significant problem.

For example, there is a cron job that runs twice a day checking to see if the backups completed successfully and it sends out an email with the status (success or failure).  But, for whatever reason, the cron job stops running.  Note, there is no email generated and the lack of email is an indication of a failure in the system, but it's not visible.  If you are lucky one or more of the engineers are watching for the emails and will respond if it doesn't come in.  But, if they are on vacation or if they quit, nobody else is aware that this needs to be checked.  And the lack of of email is missed.

Finally, there is no accountability for responding to the email.  You either have multiple engineers responding at the same time unaware of others working on an alert or each of the engineers thinks that somebody else is responding and thus nobody responds.

## How The Problem Evolves
It starts out innocently enough.  Monitoring systems are set up and a way to send monitoring status updates is needed.  Since there is no system like DaTamer available, the decision is to send email to a central distribution list and the monitoring status email (MSE) is born.  For a small system, this works fine.  But, as the number of services, applications, servers and monitoring systems grows, the number of MSEs grows causing an avalanche that buries the engineers and also obscures real events with noise.

## The Solution:  DevOps Alert Tamer (DaTamer)

DaTamer improves the situation in five ways:

* Organizes the flood of emails in a sharable, subscribe-able organization using a combination of traditional organizational principles and machine learning / ai
* Applies machine learning / ai techniques to identify events from groups of emails and sends event alerts about events cutting the number of emails dramatically
* Provides an interface to review events and see all of the MSEs associated with an event w/o flooding the user's email
* Keeps track of scheduled emails and sends an alert if an expected email is not received
* Tracks who has responded and is assigned to address an event

## Phases of Completion
The problem will be address in a sequence of phases that will build on previous phases.  Each phase is described in detail in separate "md" files.

### Phase 1:  Lay the Foundation
* Open source code of conduct for this project
* Coding standards (anticipated to be a python project)
* Develop training / testing sets of data
* Architecture design to be able to "eat or own dog food"
* DaTamer will have it's own email address that it works with to receive and process emails

### Phase 2:  Traditional, sharable organization of emails
In this phase we implement traditional organization of MSEs and add communication/tracking about who is handling alerts.  We do this with two steps:

* Move from an email distribution list to a single email account that engineers share (sharing is temporary for early phases and will be replaced in a later phase)
* Add organizational information to those MSEs that can be easily enhanced with the data

The engineers generating the monitoring status emails (MSEs) can often control the content of the subject and body.  By developing a classification system, the content of the MSEs can be adjusted to make classification easier.  Then, an automated system can be developed to organize the MSEs according to the added data in the email subject / body.  This is especially effective for new monitoring systems.  Future phases will address existing monitoring with machine learning, so it's isn't necessary to go back and change all of the existing monitoring.

#### Changing the process engineers use
As new monitoring services are created and new MSE's are generated, they follow the information standards described below so they can be organized with the automated system.  However, older emails may not conform to the new standards and traditional methods of classification may not work.  All MSE's are directed to a unique address for the DaTamer service which will organize the incoming emails into folders based on the classification information below.  In the early phase of development of DaTamer, engineers will log into the shared email account to address the MSEs.  There will be two types of emails that engineers will need to address: categorized and uncategorized emails.  

##### Uncategorized emails
Uncategorized emails are those which the DaTamer service was unable to classify.   The role of the engineer is to create a mail rule to classify the email properly.  Since this is a shared email account, the classification will be shared with the entire engineering team.  In addtion, this new mail rule becomes the beginning training for the machine learning tools that come later. The engineer creates the mail rule then moves the MSE to the correct folder in the email.  From there, the MSE can be processed based on the categorized mail rules described next.

##### Categorized emails
One of the problems with current DevOps notification processes is that the email goes to a group of people and nobody is sure if the MSE has been addressed by someone else.  So, it happens that MSEs are not addressed or are addressed by more than one engineer.  This process addresses that concern.  Engineers log into the shared email account.  When they take an MSE to address, they move it into a folder with their name under the folder where it was classified.  This communicates to the other engineers that the MSE has been or is being addressed. When they have completed processing, they move the MSE from the folder with their name to a folder named, "completed," also under the folder where it was classified.  This provided a mechanism for viewing history for the service. 

#### Severity classification
MSE's typically come in severities of information, warning, critical.  By adding an uncommon string indicating the severity on the subject line, it can be coded into DaTamer how to process the email.  For example:

* :info: - informational
* :warn: - warning message
* :critical: - The system generating the MSE typically handles sending a page for these during this phase.  But the information is added to the email for future use.

#### Schedule classification
While the schedule can vary widely, checking for scheduled emails can be relatively easy.

* :sched-perday-N/V: - Scheduled email expecting N (a number is used here) per day.  The system could check at any given time and find N MSEs in the last 24 hours.  V is the allowed variance.  If count of MSEs is more than that plus/minus the variance, generate an event / alert.  If the "/V" is not provided, it is assumed to be zero.
* :sched-permonth-N/V: - This is a scheduled email expecting N (a number is used here per month.  If count of MSEs is more than that plus/minus the variance, generate an event / alert.
* :sched-perhour-N/V: - Same but for each hour.  If count of MSEs is more than that plus/minus the variance, generate an event / alert.

#### Organizational classification
Applications are typically composed of multiple tiers such as presentation, business logic, data tier, monitoring, etc.  Organization mse's by the classification can help identify patterns of events.  Each company, organization or business unit will have its own unique classification.  Many MSEs will be able to be custominzed to include an organizational identifier such as:

* :org:datamer.frontend.ui.service1:
* :org:datamer.frontend.ui.service2:
* :org:datamer.frontend.api.service1:
* :org:datamer.frontend.api.service2:
* :org:datamer.frontend.haproxy:
* :org:datamer.logic.mse.organizer.standard:
* :org:datamer.logic.mse.organizer.ml:  (where ml stands for machine learning  and is implemented in a future phase)
* :org:datamer.data.db.olap:  (olap meaning the part of the db that manages relational data)
* :org:datamer.data.db.ml:  (ml meaning that part of the database that manages the machine learning data)

#### Server information
Most MSEs are related to a server or parhaps a set of servers.  So, each MSE will include information about the server name(s) to which it refers.  This will be the fully qualified domain name (fqdn).  If more than one server is associated with the MSE, then mutiple strings will be included in the body in a format similar to the following:

* :svr:fqdn

### Features for DaTamer in Phase 2
In phase 2, DaTamer will be able to
-- Classify emails based on the classification standard listed above using traditional methods
-- Track scheduled emails based on the schedule classification emails listed above
-- DaTamer will be monitored using its own methodologies and tools

### Phase 3: Database
In phase 2, we moved from a distribution list to a shared email to quickly add the ability to share organization of emails and provide feedback of who is working on which MSE.  In phase 3 we lay the foundation for for the advanced machine learning features in the next phase and we provide a simple application to allow viewing and processing of messages through the app instead of through the shared email address. We start to introduce the concept of events.  In this phase the system still considers one event the same as one MSE.  In future phases, the machine learning algorithms will be able to recognize when multiple MSEs are related to a single event.  In addition to adding MSEs to the database, the MSE will be deleted from the email account.  Starting with phase 3, all processing of the MSEs will take place in the application.

The database will need to store the MSEs quickly and efficiently as they come in and to classify them appropriately.  In addition, users can subscribe to MSEs by their organization and level of interest.  For example, an engineer can be a "responder" for :org:datamer.frontend.ui.service1: and "interested" for :org:datamer.frontend.ui.*: and can have unlimited associations.  This groupig of subscriptions is always stored as a user group and multiple users can be part of a user group.

#### Application UI specifications
An engineer logs into the application and is presented with a dashboard of the service health showing:
-- Total Events Open & assigned/unassigned based on Info, Warning & Critical

From the dashboard, the user can go to a list view of the open events which is sorted by severity (Critical, Warning, Info) and assignment (unassigned before assigned).  The user could then select a filter of Responder / Interested / Unclassified / All.  Node that the second item (Interested) includes the first (Responder).

The list view shows the date the event was generated, a subject from an MSE, severity, classification, who is assigned or unassigned, date of assignment (or blank), and the user's interest level (responder/interested/all).  The user could then click right-click on any item and assign it to any user.  The user to whom it is assigned is notified via email of the assignment.

From the list view, the user can click on any event to see the details of the event.  The event detail screen shows the details of the event (created date, assigned date, who is assigned, severity, etc).  In addition, this screen show a list of MSEs that are related to the event.  In phase 3 there remains a one-to-one relationship between MSE and event.  In future phases, multiple MSE's can be related with a single event.  The event detail screen will also allow the user to create an MSE rule similar to the mail rules created in in phase 2.  These rules are used for future classification as new MSEs are received.  These classification rules are created by engineers so regex combinitation of subject and body and source fqdn are allowed (to name a few).  The rules should be set up for sevrity, classification and :svr:fqdn: where possible.

#### Reports
This phase has a basic historical report based on a date range including:
-- MSEs received 
-- Events created with statistics
-- -- Total
-- -- By classification
-- -- By Severity
-- -- Open / Closed
-- Statistics
-- -- Min, Max, Mean time delay between event creation, event assignment, event completion by severity

#### Database considerations
This system should be capable of multi-tenancy.  We can have more than one customer in the database and each customer will have its own email address to process.

### Phase 4
In this phase we add the machine learning features to the system.  As a result of the previous phases, the database has data that can be mined and used for training.  MSEs have classification, severity and server information or rules have been created to classify the MSEs without the necessary information.  The system will now work to find patterns of MSEs so that a single event can be created for multiple MSEs.  In addition, new MSEs can be associated with an open event.  Finally, new events can be related to historical events that have already been completed.

