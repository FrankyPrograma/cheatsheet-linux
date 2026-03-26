# Linux Commands

## Initial Commands

* `whoami`: Shows the current logged-in user in the terminal.
* `id`: Displays your user ID (UID) and the groups you belong to (GID).
* `cd [directory]`: Changes the current directory. Use `cd /` to go to the root directory, or `cd ..` to go back one level.
* `ls`: Lists files and directories in the current path.
  * `-a` (all): Flag that shows all elements, including hidden files (starting with a dot).
  * `-l` (long): Flag that shows detailed information (permissions, owner, size, date).
* `pwd`: Print Working Directory. Shows the absolute path you are currently in.
* `exit`: Logs out of the current session or closes the terminal.
* `su [user]`: Switch User. Allows you to switch the current session to the specified user.
* `sudo [command]`: SuperUser DO. Executes a specific command with root (administrator) privileges.
* `cat [file]`: Print the contents of the specified file via terminal.
* `which [executable]`: Displays the absolute path of an executable or command. Useful for checking if something is installed.
* `touch [file]`: Command to create empty files or update their timestamps.
* `rm [file/directory]`: Removes files or directories.
  * `-r` (recursive): Applies to directories to delete everything inside them.
  * `-f` (force): Deletes forcefully without prompting for confirmation.
* `mv [source] [destination]`: Allows us to move and/or rename files or directories.
  * `mv file.txt /new_directory/`: Moves the file to the specified directory.
  * `mv file.txt new_name.txt`: Renames the file in the current directory.
  * `mv file.txt /new_directory/new_name.txt`: Moves and renames the file at the same time.

## Create User and Groups Commands

* `adduser`: It's an interactive command to create users; it asks for your information and then automatically creates the user directory in `/home/[user]`.
* `useradd`: It creates the user but doesn't add a user folder or set a password, so it's disabled until it's configured manually (useful for automations).
  * `useradd -m -s /bin/bash -d /home/[user] [user]`: Allows us to easily create a user, assigning the specified shell (`-s`) and automatically creating (`-m`) their home directory (`-d`).
  * `passwd [user]`: Allows us to change or assign a password for a specific user. Root can change any user's password, while standard users can only change their own.
* `addgroup`: It's a more interactive and user-friendly command; the differences are similar to those of `adduser` and `useradd`.
* `groupadd`: Adds groups to the system directly, without questions or interaction.
* `usermod -aG [group] [user]`: Appends the specified user to the specified supplementary group.
  * `-a` (append): If we don't use `-a` combined with `-G`, it will remove the user from all previously assigned supplementary groups, leaving them only in the newly specified one.

## Reading and Modifying Permissions

* `---|---|---`: It is important to understand that permissions are read in 3 groups, which belong to (Owner|Group|Others).
* `rwx|rwx|rwx`: R = Read, W = Write, X = Execute. Read is the ability to view the contents of a file or directory. Write is the ability to modify said file or directory. Execute is the ability to run a file as a program or traverse into a directory.
* At the very beginning of the permission string, we can find a `-` or `d`. The `-` refers to a regular file, while the `d` refers to a directory.
* `chmod`: This command allows us to modify the permissions of a file or directory.
  * `chmod u+w`: We grant write permissions (`+w`) to the owner (`u`).
  * `chmod g+x`: We grant execution permissions (`+x`) to the group (`g`).
  * `chmod o-wx,o+r`: For others (`o`), we remove write and execute permissions (`-wx`) and explicitly grant read permissions (`+r`).
  * *Note: These are just a few examples of the many combinations we can create; testing them on your own is highly recommended.*
 
* `chown [user] [file/directory]`: Allows us to change the owner of a file or directory.
  * `chown [user]:[group]`: Also allows us to change both the user and the group of a file or directory simultaneously.
  * `chown -R [user]:[group]`: Applies the user and/or group change recursively (`-R`), meaning it affects the specified directory and all its contents (children). 

* `chgrp [newGroup] [directory/file]`: Allows us to change the group ownership of a file or directory.
  * `chgrp -R [newGroup] [directory]`: Does this recursively, applying the group change to the specified directory and all its contents.

### The Directory Bottleneck (Inheritance vs. Traversal)
In Linux, permissions are not strictly inherited from parent to child. Instead, parent directories act as bottlenecks. 

* **Scenario 1: Blind Access (`--x`)**
  * We have a `/directory` with `rwx|--x|--x` and a `file.txt` inside with `rwx|rwx|rwx`.
  * As a non-owner user, we **can** access/traverse the directory because we have execution permissions (`x`).
  * We **can** also edit the file's content (if we know its exact name/path) because the file grants us full permissions (`rwx`).
  * However, we **cannot** delete the file, because deleting a file means modifying the parent directory, and we lack write permissions (`w`) on that directory.

* **Scenario 2: The Glass Wall (`r--`)**
  * We have a `/directory` with `rwx|r--|r--`.
  * As a non-owner user, we **cannot** traverse it, but we **can** perform an `ls` to see the list of files inside (thanks to the `r`). However, we cannot see their metadata without traversing.
  * If we see a `file.txt` inside with `rwx|rwx|rwx`, it doesn't matter. Since we lack execution permissions (`x`) to traverse the directory, and write permissions (`w`) to modify it, we **cannot** interact with, edit, or delete the file in any way.

* **Scenario 3: The Useless Write (`-w-`)**
  * What if the directory has `rwx|-w-|-w-`? **Absolutely nothing happens.** Even if we have write permissions (`w`) on the directory, without execution permissions (`x`) to traverse it, the write permission is completely useless. We cannot delete or add files because the system won't even let us enter the directory to execute the action.

### Special Permissions & Attributes
* **Sticky Bit (`+t`):** Understanding directory write permissions reveals a security flaw: if a shared directory has `rwx|rwx|rwx`, any user can delete any other user's files (because everyone has modify permissions on the directory, regardless of the file's individual permissions). 
  * To prevent this, the sticky bit is applied to the parent directory (e.g., `chmod +t /shared_directory`). 
  * Once applied, even if all users have write permissions on the directory, they are strictly restricted to deleting or renaming **only their own files**.

* **Extended Attributes (`lsattr` & `chattr`):**
  * `lsattr [file/directory]`: Command used to list extended file attributes.
  * **Immutable Flag (`+i`):** If we want to make a file completely immutable (preventing absolutely any modification, deletion, or renaming), we apply this attribute using `chattr +i [file]`. 
  * Once applied, **not even `root`** can modify or delete the file until the attribute is explicitly removed (`chattr -i [file]`). This offers a much higher level of security than standard read-only permissions (like `r-x|r--|r--`), which the root user could easily bypass or override.
