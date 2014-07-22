
LOGIN
--------------------------------------------------------------------------
GUI connect processing should be as follows:

1. Authenticate user against Active Directory

2. Connect to provisioning db with credentials from
   /rdreplt/dbProv/config/setEnv and
	/rdreplt/dbProv/config/private/systemCredentials-${GUI_DB_USER}/${PROV_DB_PASSWORD}@${PROV_DB_SERVICE}

3. Set current_schema
	alter session set current_schema=&PROV_DB_USER;

3. Get list of teams the user is authorized for
	see getTeamsForUser.sql
	
4. Run PROV_CONTEXT.SET_CONTEXT procedure to set the Oracle security context.
	SET_CONTEXT should be run initially at login and whenever the user changes the active team.
	PROV_CONTEXT.SET_CONTEXT(end_user_id, team_name)


CLONE DESTINATION SUMMARY
--------------------------------------------------------------------------

Summary on dashboard
	see getCloneDestSummary.sql

ShowAll button
	open new window and see getCloneDestDetails.sql


QUEUED REQUESTS
--------------------------------------------------------------------------

Summary on dashboard
	see getQueuedRequestsSummary.sql

	Display like 
		                          Last 2 days
		Team/Request	Active	Success	Failure
		------------	------	-------	--------
		Current team name 
		  Backup        999     999        999
		  Create        999     999        999
		  Drop          999     999        999
		  Update        999     999        999
		(Others)
		  Backup        999     999        999
		  Create        999     999        999
		  Drop          999     999        999
		  Update        999     999        999

ShowQueue button
	open new window and 
	select * from v_request_queue
	see getQueuedRequestsDetails.sql


MESSAGES
--------------------------------------------------------------------------

Dashboard
	select * from v_current_group_message
		display queue_id, message, and message_ts
		msg_type could be shown as an icon or by row color.  Types are Info, 
			Warning, or Error.
		Acknowledge button to left of message should run 
			GROUP_MESSAGE_PKG.ACKNOWLEDGE_MESSAGE(msg_num);
			commit;

ShowAll button
	select gm.QUEUE_ID,
	   gm.MSG_TYPE,
  	 gm.MESSAGE_TS,
  	 um.USER_ID ACKNOWLEDGED_BY_USER,
  	 gm.ACK_TS ACKNOWLEDGED_TS,
  	 gm.MESSAGE
	from GROUP_MESSAGE gm,
  	 USER_MASTER um
	where GROUP_NAME = sys_context('PROV_CTX', 'GROUP_NAME')
	  and um.USER_GUID = gm.ACK_BY_USER_GUID

AcknowledgeAll button
		GROUP_MESSAGE_PKG.ACKNOWLEDGE_MESSAGE(NULL);
		commit;


CLONE SUMMARY
--------------------------------------------------------------------------

see cloneSummary.sql- returns
	CLONE_STATE				- for all clone definitions, do not display
	CLONE_NAME				- for all clone definitions
	DESCRIPTION				- for all clone definitions
	SOURCE_PROFILE			- for all clone definitions
   QUEUE_ID					- only if clone currently exists, do not display
   SID  						- only if clone currently exists
	REQUEST_STATE			- only if clone currently exists
	STATUS_MESSAGE			- only if clone currently exists
	SUBMIT_TS				- only if clone currently exists
	INSTANCE_LIST_FILE	- only if clone currently exists, do not display

[Create New Clone Definition] button at bottom of screen

	Pulldown command list for each row with commands determined as follows:
		- if CLONE_STATE is NULL
			Create Clone
			Edit Clone Definition
		- if CLONE_STATE is Active
			Drop Clone
			Update Clone
			Renew Lease
			Edit Clone Definition
			Show Clone Details
		- otherwise
			Edit Clone Definition

	To submit Create, Drop, Update, or Renew Lease jobs
		queueId = BUILD_MGT.SUBMIT_QUEUE_JOB(request_type, request_name)
		commit;

	If user clicks on SID column
		open Show Clone Details screen

	If user clicks on Clone_Name column
		open Edit Clone Definition screen

	If user clicks on Create New Clone Definition button
		open Edit Clone Definition screen with Clone_Name = NULL

----------
----------
----------
----------
----------
----------
----------
----------
----------
----------

CLONE DETAIL
--------------------------------------------------------------------------
The clone detail screen has 5 sections.  The first section list the
team_name and clone_name, which are already in memory. Subsequent sections 
are:
   - Clone Definition,
   - Current Clone Details,
   - Current Clone Messages,
   - Clone Contents.
Each of those sections is fed by a separate query, as follows:



Team Name: |__________|       Clone Name: |__________|

+----------------------------------------------------+
|                  Clone Definition                  |
+----------------------------------------------------+
Description: |_______________________________________|
Source Profile:               |__________|
Label Prefix:                 |__________|
Specific Destination SID:     |__________|
Preferred Hostpool:           |__________|
Alternate Hostpool:           |__________|
Parameter Set Group Name:     |__________|
Email Notify Flag:            |__________|

[Edit Clone Definition Button] <- switch to the edit clone definition screen.
[Create New Clone Definition Button] <- switch to the edit clone definition screen.

+----------------------------------------------------+
|                Current Clone Details               |
+----------------------------------------------------+
QueueID:     |__________|            SID: |__________|
Clone Label: |_______________________________________|
Source Backup Label:   |_____________________________|
Clone XML File:        |_____________________________|
Task XML File:         |_____________________________|
Instance List File:    |_____________________________|
Lease Expiration Time: |__________|
Request State:         |__________|
Submitted:             |__________|
Run Start:             |__________|
Completion:  |__________|   Run duration: |__________|

+----------------------------------------------------+
|               Current Clone Messages               |
+----------------------------------------------------+
                                         Acknowledged
Type      Message           Time        By          Time
--------  ----------------  ----------  ------      ------
MSG_TYPE  MESSAGE           MESSAGE_TS  ACK_BY      ACK_TS

+----------------------------------------------------+
|                   Clone Contents                   |
+----------------------------------------------------+
Open window displaying the instance Table of Contents.  
	There will be a TOC database in each instance.  A script which queries it 
	and generates command line output is in /rdreplt/dbProv/user/bin/listTableOfContents.  
	You can duplicate that functionality in your GUI.

See cloneDetailAndEdit.sql for queries
If the Current Clone Details query does not return a record, display 
"This clone is not currently loaded."





Team Name: |__________|       Clone Name: |__________|
+----------------------------------------------------+
|                CLONE DEFINITION EDIT               |
+----------------------------------------------------+
Description: |_______________________________________|
Source Profile:               |__________|
Label Prefix:                 |__________|
Specific Destination SID:     |__________|
Preferred Hostpool:           |__________|
Alternate Hostpool:           |__________|
Parameter Set Group Name:     |__________|
Email Notify Flag:            |__________|
Edited by:   |__________| at  |__________|

[Delete Button]
[Save Button]
[SaveAs Button]
[Quit Button]
[Help Button]

Notes:
	Team_name is readonly
	Clone_name is readony if this screen was called as an edit but is writable
	if the screen was called with Clone_name NULL to create new clone definition
	All other fields are editable except change_by and change_ts.


Field                       Field Type
-------------------------   ----------------------------------------------
DESCRIPTION                 Free-form text
SOURCE_PROFILE              Combo box (see below for query)
BACKUP_LBL_PREFIX           Free-form text
SPECIFIC_DEST_SID           Combo box (see below for query)
PREFERRED_HOSTPOOL          Combo box (see below for query)
ALTERNATE_HOSTPOOL          Combo box (see below for query)
PARAM_SET_GROUP_NAME        Combo box (see below for query)
EMAIL_NOTIFY_FLG            Combo box ('All', 'None', 'Failure')


See cloneDetailAndEdit.sql for queries
See /rdreplt/dbProv/user/bin/listTableOfContents. for TOCs

----------
----------
----------
----------
----------
----------
----------
----------
----------
----------

+------------------------+
| cloneDetailAndEdit.sql |
+------------------------+


