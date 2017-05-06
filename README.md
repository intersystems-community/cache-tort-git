# Caché Git 

Source Version Control plug-in for Caché Studio. Caché Git allows working with git-repos straight from Caché Studio.

## Features
Caché Git provides interface from Caché Studio to TortoiseGit. Caché Git supports one repository for each Caché namespace. User chooses files to include in version control.

## Requirements
You should install [TorgoiseGit](http://code.google.com/p/tortoisegit/) first. Then [msysgit](http://msysgit.github.com/). Choose "Run Git from the Windows Command Prompt" option when installing.

Caché Studio before 2011.1 cannot work with executables that have space in its name (like "C:\Program Files\TortoiseGit\bin\TortoiseGitProc.exe"). In that case you should either install TortoiseGit in the folder without spaces or use newer version of Caché Studio. Caché Studio can work with previous version of Caché server. For example, Caché Studio 2013.1 [can](http://docs.intersystems.com/cache_latest/csp/docbook/DocBook.UI.Page.cls?KEY=ISP_interop) work with Caché 2009.

_On *Nix systems Caché Git provides only Export/Import without calling TortoiseGit._

## Installation in Caché:
1. Enable [write-access](http://docs.intersystems.com/ens20151/csp/docbook/DocBook.UI.Page.cls?KEY=GSA_config#GSA_config_database_edit) to CACHELIB namespace, it is required for csp-page with settings import.
2. Import project in %SYS:

    %SYS>do $system.OBJ.ImportDir("c:\path\to\project","*.xml","ck",,1)

3. Now you can disable write-access to CACHELIB.
4. Select Version Control Class in System Management Portal for desired namespace. You should select %SourceControl.Git class.

## Setup
In Studio, in any namespce with %SourceControl.Git as a Version Control Class choose menu "Git > Settings"

1. Enter full path to TortoiseGitProc.exe (usually "C:\Program Files\TortoiseGit\bin\TortoiseGitProc.exe")
2. Enter default temp folder (usually "C:\Temp"). ATTENTION! This folder shouldn't contain anything valuable. There will be subfolders for each Caché  namespace.
3. You can also specify temp folder for this particular namespace. Otherwise temp folder for this namespace will be &lt;default-temp-folder>\\&lt;namespace>

  * If you don't want to enter you credentials every time you use git create in temp folder file _netrc and add string `machine your-git-server login your-git-login password your-git-password`.
Instead of your-git-server, your-git-login and your-git-password write actual values. Or use [Git Credentials](http://stackoverflow.com/a/15351702).


## How to work with plug-in
### How to start
1.  Select menu Tortoise Git Settings and set up options for Git (e.g. Proxy). This options are stored in folder that you chose as temp.
2. Chose menu Create Repo. Press OK. One more time.
3. Now menu has full view.
4. A new menu item appears in context menu — "Git -> Add to Source Control".
5. Choose elements you want to add to Git. These programs, classes, csp-pages etc. are exported to hdd-folder and automatically synchronized with it every time when you open or save them in Studio.
6. Choose menu Commit.
7. Select all files, right-click on them and choose "Add".
8. Write comment and press "Commit"
9. Now you can immediately push changes to server
  * You should press Push
  * Then click Manage button and specify git-server
  * For example:
  * Remote: assembla
  * URL: http://git.assembla.com/demobld.git
  * Choose the server you have just specified.
  * Press OK

### How to join existing repository
1. Choose menu Clone.
2. Enter repository URL.
3. [If your TortoiseGit Version < 1.8.5] In appeared window TortoiseGit adds repository name to folder. You should delete it (Write c:\temp\git\user instead of c:\temp\git\user\reponame)
4. Press 'OK'.
5. Select menu Import All to load changes from temp folder to Caché. Import All compares timestamps of files and Caché items and import files only if they are newer then Caché versions. There is also Import All Force menu item that import files without any checks.

### Work Cycle
1. Choose menu item Pull.
2. Resolve conflicts in TortoiseGit if any.
3. Select menu Import All to load changes from temp folder to Caché.
4. Do some changes in Caché Studio.
5. [Optional] Select menu Export All if you want to force-unload changes to temp folder from Caché.
6. Select Commit
7. Select Push from Caché Studio menu or immediately after Commit.

### Global's structure
Data used by Caché-Git is stored in global which name is defined by Storage parameter of %SourceControl.Git.Utils class. Its value by default — ^Git.
Options (path to tortoiseproc.exe and temp folder) are stored in %SYS namespace in nodes ^Git("%gitBinPath") and ^Git("%defaultTemp").
In each namespace we also store path to temp folder for this namespace that can override default path. Global name is ^Git("settings","namespaceTemp")

In ^Git("items") node namespace elements are stored that traced by Caché-Git. We store names of particular items with extension in lower-case.

In ^Git("TSH") node timestamps of last synchronization for each routine are stored.

To run all tests:
    
	%SYS>do ##class(%SourceControl.Git.Test.Git).runall()
	
### Extra Repository Settings
#### Mappings
To specify particular folder in repository where Caché Git should place files set value of `^Git("settings","mappings") to relative path. Additionally you can specify subfolder for each particular item type.

For example:

	USER>zwrite ^Git
	^Git("settings","mappings")="src/cache/"
	^Git("settings","mappings","cls")="cls/"
	^Git("settings","mappings","csp")="csp-data/"
	^Git("settings","mappings","dfi")="dfi/"
	^Git("settings","mappings","mac")="mac/"
	^Git("settings","namespaceTemp")="C:\temp\testctg\"
	
In this particular case Caché Git stores all its files (except sc-list.txt) in `src/cache/` subfolder. Classes are stored in `src/cache/cls/` etc. All csp files (including static files -- js, css) are stored in `src/cache/csp-data/`. With this particular path to repository classes will be stored in folder `C:\temp\testctg\src\cache\cls\`.

You can omit root mappings. Mappings for particular item type are still applied.

#### Other Settings
The following sub-nodes of `^Git("settings")` allow for further configuration:
* `"groupByFolder"` - boolean value that defaults to `0`. The setting applies to all INC, MAC and INT routines as well as DFI exports.
  * When this value is true, use nested folders for these files e.g. `Package.Include.INC` would be exported to `<rootfolder>/Package/Include.inc.xml`
  * When this value is false, all these files are exported to the root folder.
* `"includeExtensionInFilename"` - boolean value that defaults to `1`.
  * When true, exported files include the lowercase extension in the filename, e.g. `MyClass.cls.xml`
  * When false, exported files do not include the extension in the filename, e.g. `MyClass.xml`

NOTE: Caché Git does not do any automatic moving of files. If you have classes placed in root folder and change mappings for cls to "src/cache/cls/" you need to move files corresponding to these classes manually. Better -- set mappings on creating of repository, before adding new items to source control.
