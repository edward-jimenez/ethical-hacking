## Link to tryhackme room

[committed](https://tryhackme.com/room/committed)

This challenge is a bit different from previous challenges in that you don't need to scan for vulnerabilites or try to escalate privileges. Instead, you need to have some basic understanding of how git works.  To be honest, my skill level when it comes to git is very limited, but I can "google" like nobody's business.  

To get started, access the target via the "Show Split Screen" button.  Once it has launched, open a terminal and navigate to /home/ubuntu/committed.  Unzip the comitted.zip file and cd to the extracted folder. If we do an 'ls -altr' we see there is a .git folder:  
```
 ubuntu@thm-comitted:~/commited/commited$ ls -altr
total 20
-rw-rw-r-- 1 ubuntu ubuntu  982 Feb 13  2022 main.py
-rw-rw-r-- 1 ubuntu ubuntu  393 Feb 13  2022 Readme.md
drwxrwxr-x 8 ubuntu ubuntu 4096 Feb 13  2022 .git
drwxrwxr-x 3 ubuntu ubuntu 4096 Feb 13  2022 .
drwxrwxr-x 3 ubuntu ubuntu 4096 Sep 29 04:46 ..
```
First thing we can try is to list the git logs and compare each commit to the previous commit:  

### Git log

```
ubuntu@thm-comitted:~/commited/commited$ git log
commit 28c36211be8187d4be04530e340206b856198a84 (HEAD -> master)
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:49:32 2022 -0800

    Finished

commit 9ecdc566de145f5c13da74673fa3432773692502
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:40:19 2022 -0800

    Database management features added.

commit 26bcf1aa99094bf2fb4c9685b528a55838698fbe
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:32:49 2022 -0800

    Create database logic added

commit b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:30:43 2022 -0800

    Connecting to db logic added

commit 441daaaa600aef8021f273c8c66404d5283ed83e
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:28:16 2022 -0800

    Initial Project.
```
### Git Diff

```ubuntu@thm-comitted:~/commited/commited$ git diff 441daaaa600aef8021f273c8c66404d5283ed83e b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67
diff --git a/main.py b/main.py
index dfe24c9..44f3cb3 100644
--- a/main.py
+++ b/main.py
@@ -1 +1,10 @@
-print("Hello World\n")
+import mysql.connector
+
+mydb = mysql.connector.connect(
+  host="localhost",
+  user="", # Username Goes Here
+  password="" # Password Goes Here
+)
+
+print(mydb)
+
```

I did this for each commit, but didn't find anything interesting so I asked my frienemy, google, who pointed to this link [Common Git Mistakes](https://www.edureka.co/blog/common-git-mistakes/).  Two commands that can assist us in getting more information are: `git reflog` and `git reset --hard <commit-id>.

### git reflog output

```
ubuntu@thm-comitted:~/commited/commited$ git reflog
28c3621 (HEAD -> master) HEAD@{0}: checkout: moving from dbint to master
4e16af9 (dbint) HEAD@{1}: checkout: moving from master to dbint
28c3621 (HEAD -> master) HEAD@{2}: commit: Finished
9ecdc56 HEAD@{3}: checkout: moving from dbint to master
4e16af9 (dbint) HEAD@{4}: commit: Reminder Added.
**c56c470 HEAD@{5}: commit: Oops**
3a8cc16 HEAD@{6}: commit: DB check
6e1ea88 HEAD@{7}: commit: Note added
9ecdc56 HEAD@{8}: checkout: moving from master to dbint
9ecdc56 HEAD@{9}: commit: Database management features added.
26bcf1a HEAD@{10}: commit: Create database logic added
b0eda7d HEAD@{11}: commit: Connecting to db logic added
441daaa HEAD@{12}: commit (initial): Initial Project.
```

This looks promising, there is a commit message "Oops".  Let's reset the git repository to this commit:  

```
ubuntu@thm-comitted:~/commited/commited$ git reset --hard HEAD@{5}
HEAD is now at c56c470 Oops
```

and now let's view the git logs:  
```
ubuntu@thm-comitted:~/commited/commited$ git log
commit c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5 (HEAD -> master)
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:46:39 2022 -0800

    Oops

commit 3a8cc16f919b8ac43651d68dceacbb28ebb9b625
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:45:14 2022 -0800

    DB check

commit 6e1ea88319ae84175bfe953b7791ec695e1ca004
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:43:34 2022 -0800

    Note added

commit 9ecdc566de145f5c13da74673fa3432773692502
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:40:19 2022 -0800

    Database management features added.
:...skipping...
commit c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5 (HEAD -> master)
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:46:39 2022 -0800

    Oops

commit 3a8cc16f919b8ac43651d68dceacbb28ebb9b625
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:45:14 2022 -0800

    DB check

commit 6e1ea88319ae84175bfe953b7791ec695e1ca004
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:43:34 2022 -0800

    Note added

commit 9ecdc566de145f5c13da74673fa3432773692502
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:40:19 2022 -0800

    Database management features added.

commit 26bcf1aa99094bf2fb4c9685b528a55838698fbe
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:32:49 2022 -0800

    Create database logic added

commit b0eda7db60a1cb0aea86f053816a1bfb7e2d6c67
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:30:43 2022 -0800

    Connecting to db logic added

commit 441daaaa600aef8021f273c8c66404d5283ed83e
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:28:16 2022 -0800

    Initial Project.
```

finally, let's issue the git diff before and after the "Oops" commit:  

```
ubuntu@thm-comitted:~/commited/commited$ git diff 3a8cc16f919b8ac43651d68dceacbb28ebb9b625 c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5
diff --git a/main.py b/main.py
index 54d0271..0e1d395 100644
--- a/main.py
+++ b/main.py
@@ -4,7 +4,7 @@ def create_db():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root", # Username Goes Here
-    password=**"<REDACTED>"** # Password Goes Here
+    password="" # Password Goes Here
     )
 
     mycursor = mydb.cursor()
@@ -16,7 +16,7 @@ def create_tables():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root", #username Goes here
-    password=**"<REDACTED>"**, #password Goes here
+    password="", #password Goes here
     database="commited"
     )
 
@@ -29,7 +29,7 @@ def populate_tables():
     mydb = mysql.connector.connect(
     host="localhost",
     user="root",
-    password=**"<REDACTED>"**,
+    password="",
     database="commited"
     )
```

Submit the flag and you're done.  


