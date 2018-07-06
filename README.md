~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
|                                                                                       |
|               *Windtastic Productions* _PRESENT_                                      |
|                               ~~                                                      |
|                     "SO YOU'RE GOING TO PROD"                                         |
|                                                                                       |
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You're going to prod. This is the culmination of months of work. You're happy, the client is expectant.
You're going to deploy this to a live instance for all the users.

THAT'S GREAT.

Now let's make sure we don't nuke everything.
Salesforce Change Sets and metadata deploy are non-reversible. What is done is done forever, and Salesforce has no backup. This is why we're writing this today. You can't just deploy Metadata for some users - it's for everybody.

**SHIT YOU DO BEFORE YOU DO ANYTHING ELSE**
- Login to Prod. Is there already a config and data, even if it's simple enough to be an amoeba sliding out from the ocean ?
	> Nope
		Wow, a Greenfield implementation! Nice. Skip this section.
	> Yup.
		You're touching a system these people use to store live data and use processes.
		Are you deploying stuff that changes existing objects, permissions or processes ?
		> Nope
			You're not completely out of the woods but you can skip "Regression testing" from this guide
		> Yup.
			So you have a major deployment that can, but is not limited to:
				- break existing workflows
				- delete existing data
				- impact all users.
			Great, welcome to our guide. You can't skip anything, so go through everything. Congrats!

- Get a full data export.
	You can do this by going to Setup > Data Export and exporting EVERYTHING. This will take hours.
		>> MAKE SURE YOU CHANGE THE ENCODING. Salesforce has some dumb rule of not defaulting to UTF-8. YOU NEED UTF-8. This is Europe, not Texas. Accents and ḍîáꞓȑîȶîꞓs exist.
	You can also do this by using Jitterbit to Query all the relevant tables, if you so prefer. Jitterbit is like Dataloader but better. If you don't have it installed, install it now. IT IS FREE. It is great. If you tried doing a full data migration with Dataloader, you will not be helped. https://www.jitterbit.com/solutions/salesforce-integration/salesforce-data-loader/

- Get a full metadata backup.
	First, go to prod and create a new Sandbox. Call it Backup010818 if it's the first of August 2018. This will be your backup in case you fucked something up.
	If you have the technical capacity for it, download an IDE with a Salesforce plugin (IntelliJ community edition is free, and the plugin Jetforcer is free for 90 days, for example). Download the entire metadata. Yes, everything. Now that you've done that, use Git to start a new repo and commit everything. You can Use Gitkraken (https://www.gitkraken.com/) for that if you're new to git and just need to store it. Git will allow you to "save" changes but also revert anything you do. Great for panic rollbacks.

- Make Sure that your Change Sets contain everything, and that you ahve logged any pre- or post-install steps on the Change Set description. If you'r deploying Service Cloud, this  means your Change Set description contains:
	"Before deploy, activate Person Accounts
	Run tests ContactTest, AccountTest, Object__cTest when deploying
	After deploy, activate Process Builders PB1, PB3, PB24
	After deploy, send out test email to xxxx@mytest.com to validate routing.	"

- If you're deploying to a "mature" org that has existing processes
	- You have gained the right to do a fake deployment before. They are mature, so they already use a Partial sandbox.
	First, make sure it's refreshed recently, or that they can refresh it. DO NOT DO THAT YOURSELF. Ask them if you can.
	- You deploy stuff from your DEV sandbox to the new partial one. Once it's deployed, uyou test everything with the client's help ( alot of testing is on their side ).
	ANY MODIFICATION IS DONE IN YOUR DEV SANDBOX AND THEN DEPLOYED TO UAT.
	This allows you to have a final Change Set from prod.
	The client MUST approve that everything is fine after deployment to UAT, including tests from their side to ensure there are no regressions.

- If there's Batch Apex involved ALWAYS TEST SHIT IN PARTIAL.

Do you have backups of absolutely fucking everything? Yes? GREAT.

**GETTING READY TO DEPLOY**

You've got backups of absolutely fucking everything.
Your Change Sets have been built and you have them uploaded.

First thing you do is validate them to make sure validations pass.
	Tests don't work ? Stuff is missing ? Back to dev sandbox to fix it.

Second thing you do is estimate how long all pre- and post-install steps will take.
	IF you're on a mature org or are impacting existing processes, this is the time window in which you will freeze all users to ensure data consistency. You will need to warn the client about this. This time needs to be short.

Being ready to deploy means that you can confidently deploy all Change Sets and do any post-install steps required as fast as possible. Meaning shit is Validated and ready to go. Note that in some cases you can't validate Change Sets because features are missing. If go live timeframe is important, DEPLOY TO PARTIAL FIRST to give yourself a sense of how long this takes and check your Change Sets.

If any pre-install steps do not impact users, warn the client and carry them out now. (e.g. New feature activation that requires support involvement but is invisible to users before setup).

Set Deliverability to "None". You ain't sending emails without testing.

**DEPLOY**

You have a set hour and date on which to deploy.
The client is warned and ready.
Change Sets are ALREADY validated.

- You freeze all necessary users.
- If you have that, activate the validation rule bypass on the admin user. No use fighting against that.
- You carry out all pre-install steps that you could not do before.
- You Deploy all Change Sets
- You carry out all post-install steps.
All of this is written on your Change Set description, so you have no surprises (hopefully).

If workflows or process builders are preventing deployment, and you deactivate them, NOTE ALL OF THEM DOWN SOMEWHERE.
Fix any bugs that pop up due to the client not keeping their prod in sync with their sandboxes.


**POST-DEPLOY**
Make sure everything looks fine, that you carried everything over.
Deactivate the validation rule bypass.
Go over each page, create a record and immediately delete it, etc.
Warn their PM that deployement is done and request testing from their side.
If you deactivated Workflows or PBs or somthing so the deployment passes, ACTIVATE THEM BACK AGAIN.

> You deployed a BATCH APEX.
	DO NOT SCHEDULE THAT THING.
	Test it with a query that only affects some records.
	Make sure everything is still fine.
	Then schedule it.

**REGRESSION TESTING**
You deployed to a live org.
With their PM and staff helping you, carry out day to day tasks from their users on selected records. Make sure there are no adverse effects that were not identified in sandbox.
Set Email deliverability back to "full". Test emails with their staff. CAREFULLY.

Stay on alert for the next few days if critical stuff happens.

**REACTIVATION**
Notify PM all is well
Unfreeze users
go drink champagne.

**REVERTING CHANGES**
- Data is affected
	You have a backup. Don't panic.
	Identify WTF is causing data to be wrong.
	Fix that.
	Get your backup, restore data to where it was before the fuckup.

- Metadata is affected
	You have a backup. Don't panic.
	Identify what was changed that shouldn't be (probably page layouts, clients never keep those up to date)
	Revert from your backup sandbox
	Add any missing elements from your deployment.
