# frugalmail
Frugal Mail - Webmail front end to transactional email providers such as Mailgun and Mandrill

##Service Architecture
### Front-end and API interface
- Webmail frontend will be static based on the [AdminLTE Bootstrap theme](https://github.com/almasaeed2010/AdminLTE/) and use AngularJS to consume JSON from an API written in Laravel 5
- Static Frontend will be hosted as files on Rackspace Cloud Files and API servers will live behind a Haproxy or Nginx load balancer
- Web front end will be written in Laravel 5 and use MariaDB for account detail storage

### Sending Messages
- Outbound email first travels through the FrugalMail API and is enqueued within the Laravel Queue for delivery to Mailgun
- 1 minute after sending the message through Mailgun, Frugalmail hits the mailgun API to get status of the sent messages to alert the sender of any problems

### Incoming Messages
- Inbound email will use Mailgun's storage feature. Optional webhooks can be specified to inform FrugalMail when there is a message waiting to be picked up, but FrugalMail will check automatically on a scheduled basis
- FrugalMail will learn how often the domain gets email and pick a sane API query rate. If you only get a few messages an hour, your mail might only be pulled from Mailgun every 15 minutes especially if you're not logged in at the moment. If you get mail more often, it'll check more frequently as quickly as once a minute. 
- Optional webhook notification will trigger frugalmail queue the request to download all pending messages within a minute

### Message Storage
- All email headers are stored in MongoDB
- All emails are indexed for keywords and those are stored in MongoDB for search purposes
- All emails, including attachments, are stored statically on Rackspace Cloud Block Storage
- A request to view a message is received by the API and then pulls the message directly from Block Storage and fed to the front-end interface
- Sent messages are stored in the same manner

### Account Management
- Deleted emails are marked as deleted in the database and purged from the search index nightly
- Purged messages are deleted from object store nightly
- Users can download an archive of all un-purged messages
- Deleting an account will cause the purge of all data during nightly maintence routines, with a confirmation email being sent once the work is done
- Account Manager sets up domains and user accounts
- Users can be given several mailboxes or catch-all email boxes
- One mailbox can be assigned to several users
- Users have their own signatures that are applied to outgoing messages, and email history shows which user read messages or sent replies, regardless if it's a shared mailbox

### Message Notification
- Users can receive notifications of new messages through several push services such as Pushover, Pushbullet, and Slack
- No POP3/IMAP/SMTP interface planned
- Native IOS/Android interfaces welcome, but not planned in the short-term
- Optional Activity reports are possible
