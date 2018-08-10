# DevOps Alert Tamer (DaTamer)
Devops Alert Tamer is an open source project applying machine learning and ai techniques to tame the massive amounts of notifications sent to devops group emails.

## The Problem
When a new employee starts with a DevOps group, they subscribe to a distribution list.  That subscription brings with it an avalanche of hundreds or thousands of monitoring update emails (mue's) every day.  This is a problem in three ways:

* It's overwhelming to the new engineer who doesn't understand the architecture of the system and has a difficult time understanding the emails and how to organize them
* Each of the engineers on the team spends significant time each day to process the exact same information.  This is inefficient
* If a scheduled email doesn't come as expected, it's easy to miss that there is a problem

For the first two weeks the new engineer wades through the mue's trying to understand what they are, which are noise and which are indications of issues to which they need to respond.  This process usually results in a large set of email rules that sort the emails into buckets that make it easier to work on sets of emails more efficiently.

The other members of the team have already gone through the process, but there is no way to share the knowlege expect by interrupting them, typically with another email, asking what each email means and if anyone is responding to it.  Thus the other engineers get a repeat of the notification (which they already processed) from the new engineer asking about it.

Additionally, a single event can generate multiple emails from different systems.  For example, a network glitch causing the NFS server to not respond in a timely fashion can generate emails from numerous servers.  It's left up to the engineer to interpret the flood of emails to identify event that needs to be addressed.

And to complicate matters, there are scheduled emails that are expected on a regular basis.  If the email doesn't come, it's an indication of a problem.  But, the lack of the email is not a visible indicator and the event can be missed until there is a significant problem.

For example, there is a cron job that runs twice a day checking to see if the backups completed successfully and it sends out an email with the status (success or failure).  But, for whatever reason, the cron job stops running.  Now, there is not email generated and the lack of email is an indication of a failure in the system.  If you are lucky one or more of the engineers is watching for the emails and will respond if it doesn't come in.  But, if they are on vacation or quit, nobody else is aware that this needs to be checked.  And the lack of of email is missed.

## How The Problem Evolves
It starts out innocently enough.  Monitoring systems are set up and a way to send monitoring status updates is needed.  Since there is no system like DaTamer available, the decision is to send email to a central distribution list and the monitoring status email (mue) is born.  For a small system, this works fine.  But, as the number of services, applications, servers and monitoring systems grows, the number of mue's grows causing an avalance that buries the engineers and also obscures real events with noise.

## The Solution:  DevOps Alert Tamer (DaTamer)

DaTamer improves the situation in four ways:

* Organizes the flood of emails in a sharable, subscribe-able organization using a combination of traditional organizational principles and machine learning / ai
* Applies machine learning / ai techniques to identify events from groups of emails and send event alerts about events cutting the number of emails dramatically
* Provides an interface to review events and see all of the mue's associated with an event w/o flooding the user's email
* Keeps track of scheduled emails and sends and alerts if an expected email is not received
