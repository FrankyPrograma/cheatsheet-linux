# Linux Commands

## Initial Commands

* `whoami`: Shows the current logged-in user in the terminal.
* `id`: Displays your user ID (UID) and the groups you belong to (GID).
* `cd [directory]`: Changes the current directory. Use `cd /` to go to the root directory, or `cd ..` to go back one level.
* `ls`: Lists files and directories in the current path.
  * `-a (-a = all)`: Flag that shows all elements, including hidden files (starting with a dot).
  * `-l (-l = long)`: Flag that shows detailed information (permissions, owner, size, date).
* `pwd`: Print Working Directory. Shows the absolute path you are currently in.
* `exit`: Logs out of the current session or closes the terminal.
* `su [user]`: Switch User. Allows you to switch the current session to the specified user.
* `sudo [command]`: SuperUser DO. Executes a specific command with root (administrator) privileges.
* `cat [file]`: Print the contents of the specified file via terminal.
* `which [executable]`: Displays the absolute path of an executable or command. Useful for checking if something is installed.
* `touch [file]`: Command to create empty files or update their timestamps. 

## Create User and Groups Commands

* `adduser`: It's an interactive command to create users; it asks for your information and then automatically creates the user directory in `/home/[user]`.
* `useradd`: It creates the user but doesn't add a user folder or set a password, so it's disabled until it's configured manually (useful for automations).
  * `useradd -m -s /bin/bash -d /home/[user] [user]`: Allows us to easily create a user, assigning the specified shell (-s = shell) and automatically creating (-m = make) (-d = directory) their home directory.
  * `passwd [user]`: Allows us to change or assign a password for a specific user. Root can change any user's password, while standard users can only change their own.
* `addgroup`: It's a more interactive and user-friendly command; the differences are similar to those of adduser and useradd.
* `groupadd`: Adds groups to the system directly, without questions or interaction.
* `usermod -aG [group] [user]`: Appends the specified user to the specified supplementary group.
  * `-a` (append): If we don't use `-a` combined with `-G`, it will remove the user from all previously assigned supplementary groups, leaving them only in the newly specified one.

## Reading and Modifying Permissions

* `---|---|---`: It is important to understand that permissions are read in 3 groups, which belong to (Owner|Group|Others).
* `rwx|rwx|rwx`: R = Read, W = Write, X = Execute. Read is the ability to view the contents of a file or directory. Write is the ability to modify said file or directory. Execute is the ability to run a file as a program or traverse into a directory.
* At the very beginning of the permission string, we can find a `-` or `d`. The `-` refers to a regular file, while the `d` refers to a directory.
* `chmod`: This command allows us to modify the permissions of a file or directory.
  * `chmod u+w`: We grant write permissions (+w) to the owner (u).
  * `chmod g+x`: We grant execution permissions (+x) to the group (g).
  * `chmod o-wx,o+r`: For others (o), we remove write and execute permissions (-wx) and explicitly grant read permissions (+r).
  * These are just a few examples of the many combinations we can create; testing them on your own is highly recommended.
 
* `chown [user] [file/directory]`: Allows us to change the owner of a file or directory.
  * `chown [user]:[group]`: Also allows us to change both the user and the group of a file or directory simultaneously.
  * `chown -R (-R = Recursive) [user]:[group]`: Allows us to change the user and/or group of a directory recursively, meaning the changes apply to the specified directory and all its contents (children). 

* `chgrp [newGroup] [directory/file]`: Allows us to change the group ownership of a file or directory.
  * `chgrp -R [newGroup] [directory]`: Does this recursively, applying the group change to the specified directory and all its contents.
