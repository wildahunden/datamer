# DevOps Alert Tamer (DaTamer)
Devops Alert Tamer is an open source project applying machine learning and ai techniques to tame the massive amounts of notifications sent to devops group emails.

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
The DaTamer service is assigned its own, unique address.  All MSEs are directed to that email address and DaTamer will organize the incoming emails into folders based on the classification information below.  In the early phase of development of DaTamer, engineers will log into the shared email account to address the MSEs.  There will be two types of emails that engineers will need to address: categorized and uncategorized emails.  
##### Uncategorized emails
Uncategorized emails are those which the DaTamer service was unable to classify.   The role of the engineer is to create a mail rule to classify the email properly.  This new mail rule becomes the beginning training for the machine learning tools that come later. The engineer creates the mail rul then moves the MSE to the correct folder in the email.  From there, the MSE can be processed based on the categorized mail rules.
##### Categorized emails
One of the problems with current DevOps notification processes is that the email goes to a group of people and nobody is sure if the MSE has been addressed by someone else.  So, it happens that MSEs are not addressed or addressed by more than one engineer.  This process addresses that concern.  Engineers log into the shared email account.  When they take an MSE to address, they move it into a folder with their name under the folder where it was classified.  This communicates to the other engineers that the MSE has been or is being addressed.   

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
Most MSEs are related to a server or parhaps a set of servers.  So, each MSE will include information about the server(s) to which it refers.  This will be the fully qualified domain name (fqdn).  If more than one server is associated with the MSE, then mutiple strings will be included in the body in a format similar to the following:

* :svr:fqdn




