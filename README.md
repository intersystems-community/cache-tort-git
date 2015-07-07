#Caché Git 

Source Version Control plug-in for Caché Studio. Caché Git allows working with git-repos straight from Caché Studio.

## Features
Caché Git provides interface from Caché Studio to TortoiseGit. Caché Git supports one repository for each Caché namespace. User chooses files to include in version control.

## Requirements
You should install [TorgoiseGit](http://code.google.com/p/tortoisegit/) first. Then [msysgit](http://msysgit.github.com/). Choose "Run Git from the Windows Command Prompt" option when installing.

Caché Studio before 2011.1 cannot work with executables that have space in its name (like "C:\Program Files\TortoiseGit\bin\TortoiseGitProc.exe"). In that case you should either install TortoiseGit in the folder without spaces or use newer version of Caché Studio. Caché Studio can work with previous version of Caché server. For example, Caché Studio 2013.1 [can](http://docs.intersystems.com/cache_latest/csp/docbook/DocBook.UI.Page.cls?KEY=ISP_interop) work with Caché 2009.

_On *Nix systems Caché Git provides only Export/Import without calling TortoiseGit._

## Installation in Caché:
10. Enable [write-access](http://docs.intersystems.com/ens20151/csp/docbook/DocBook.UI.Page.cls?KEY=GSA_config#GSA_config_database_edit) to CACHELIB namespace, it is required for csp-page with settings import.
20. Import project in %SYS:

    %SYS>do $system.OBJ.ImportDir("c:\path\to\project","*.xml","ck",,1)

25. Now you can disable write-access to CACHELIB.
30. Select Version Control Class in System Management Portal for desired namespace. You should select %SourceControl.Git class.

## Setup
In Studio, in any namespce with %SourceControl.Git as a Version Control Class choose menu "Git > Settings"

10. Enter full path to TortoiseGitProc.exe (usually "C:\Program Files\TortoiseGit\bin\TortoiseGitProc.exe")
20. Enter default temp folder (usually "C:\Temp"). ATTENTION! This folder shouldn't contain anything valuable. There will be subfolders for each Caché  namespace.
30. You can also specify temp folder for this particular namespace. Otherwise temp folder for this namespace will be &lt;default-temp-folder>\\&lt;namespace>

  * If you don't want to enter you credentials every time you use git create in temp folder file _netrc and add string `machine your-git-server login your-git-login password your-git-password`.
Instead of your-git-server, your-git-login and your-git-password write actual values. Or use [Git Credentials](http://stackoverflow.com/a/15351702).


##How to work with plug-in
###How to start
5.  Select menu Tortoise Git Settings and set up options for Git (e.g. Proxy). This options are stored in folder that you chose as temp.
10. Chose menu Create Repo. Press OK. One more time.
20. Now menu has full view.
30. A new menu item appears in context menu — "Git -> Add to Source Control".
40. Chose elements you want to add to Git. These programs, classes, csp-pages etc. are exported to hdd-folder and automatically synchronized with it every time when you open or save them in Studio.
60. Chose menu Commit.
70. Select all files, right-click on them and choose "Add".
80. Write comment and press "Commit"
90. Now you can immediately push changes to server
  * You should press Push
  * Then click Manage button and specify git-server
  * For example:
  * Remote: assembla
  * URL: http://git.assembla.com/demobld.git
  * Choose the server you have just specified.
  * Press OK

###How to join existing repository
10. Choose menu Clone.
20. Enter repository URL.
30. [If your TortoiseGit Version < 1.8.5] In appeared window TortoiseGit adds repository name to folder. You should delete it (Write c:\temp\git\user instead of c:\temp\git\user\reponame)
40. Press 'OK'.
50. Select menu Import All to load changes from temp folder to Caché.

###Work Cycle
10. Choose menu item Pull.
20. Resolve conflicts in TortoiseGit if any.
30. Select menu Import All to load changes from temp folder to Caché.
40. Do some changes in Caché Studio.
45. [Optional] Select menu Export All if you want to force-unload changes to temp folder from Caché.
50. Select Commit
60. Select Push from Caché Studio menu or immediately after Commit.

###Global's structure
Data used by Caché-Git is stored in global which name is defined by Storage parameter of %SourceControl.Git.Utils class. Its value by default — ^Git.
Options (path to tortoiseproc.exe and temp folder) are stored in %SYS namespace in nodes ^Git("%gitBinPath") and ^Git("%defaultTemp").
In each namespace we also store path to temp folder for this namespace that can override default path. Global name is ^Git("settings","namespaceTemp")

In ^Git("items") node namespace elements are stored that traced by Caché-Git. We store names of particular items with extension in lower-case.

In ^Git("TSH") node timestamps of last synchronization for each routine are stored.

To run all tests:
    
	%SYS>do ##class(%SourceControl.Git.Test.Git).runall()
	
