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

### Permissions in Octal Notation
In Linux, permissions can also be represented in binary, where an active permission is `1` and an inactive one is `0`. For example, a directory with `r-x` permissions equals `101` in binary, and `rwx` equals `111`.

By converting these binary values to octal numbers, we can assign permissions much faster and more comfortably. To do this, we use a simple numerical trick where **r = 4**, **w = 2**, and **x = 1**.
* **Example 1:** If we have `r-x`, we know it's `101` in binary. Using our trick: `4 (r) + 0 (-) + 1 (x) = 5`. So the permission for that group is exactly 5.
* **Complete Example:** `rwx|r-x|r--`.
  * Owner (`rwx`): 4 + 2 + 1 = **7**
  * Group (`r-x`): 4 + 0 + 1 = **5**
  * Others (`r--`): 4 + 0 + 0 = **4**
  * To assign these exact permissions, we would simply run: `chmod 754 [file]`.

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
    
 ### SUID and SGID (Privilege Escalation)
* **SUID (Set Owner User ID):** If we list permissions and see an `s` instead of an `x` in the owner's permissions (e.g., `rwsr-xr-x`), or if it's represented in octal as `4xxx` (like `4755`), **DANGER! WE HAVE A POTENTIAL EXPLOITATION CANDIDATE.** * If a binary has the SUID bit set, any user who runs it will execute it with the privileges of the file's owner (which is usually `root`). This means we could potentially use it to escalate privileges permanently.
  * To find all binaries with SUID permissions on our system, we use the following command:
    `find / -type f -perm -4000 2>/dev/null`
    * `/`: Searches the entire system starting from the root.
    * `-type f`: Looks specifically for files.
    * `-perm -4000`: Looks for files with the SUID bit set (`4000`). The `-` means it matches regardless of the other standard permissions.
    * `2>/dev/null`: *(Pro-tip)* Redirects all "Permission denied" error messages to the void so our terminal output stays clean and readable.
  * **Example:** If the `python` binary has SUID privileges, we could execute a specific payload to spawn a root shell. (To check exactly how to exploit specific binaries, we use: [GTFOBins](https://gtfobins.github.io/)).

* **SGID (Set Group ID):** Represented by an `s` in the group's permissions (e.g., `rwxr-sr-x`) or octal `2xxx` (like `2755`). SGID has two different functions depending on where it's applied:
  * **On a file:** Allows any user to execute the file with the permissions of the file's group owner.
  * **On a directory:** Any new file or directory created inside it will automatically inherit the group ownership of the parent directory, rather than the primary group of the user who created it.

## File Descriptors (Pointers) and Redirection

In Linux, data flows are managed using file descriptors, which we can conceptualize as default pointers. The main output pointers are:
* `1` (stdout): Pointer for standard output (successes/results).
* `2` (stderr): Pointer for standard errors.

Redirectors are simply a way to make a pointer point to another direction; they redirect the output of a command to a specific file or stream.
* `>`: The simplest redirector. It sends the standard output (stdout) of a command to a specified file so it no longer prints to the screen (e.g., `cat /etc/passwd > file.txt`). **Caution:** This will completely overwrite the existing contents of the file.
* `>>`: Similar to the previous one, but with the difference that it **does not overwrite**. It simply appends the new output to the end of the file.
* `&>`: Sends both successful outputs and errors (stdout and stderr) to the same place.

### Useful Redirector Combinations
* `2>/dev/null`: Sends the execution errors of a command to the Linux void (`/dev/null`) so they don't clutter our screen (remember that pointer `2` refers to errors).
* `&>/dev/null`: Redirects the entire output of a command (both successes and errors) to `/dev/null`, resulting in absolute silence on the screen when executing a command.
* `3>&1 1>&2 2>&3`: A technique to swap pointers so `1` outputs errors and `2` outputs successes. Let's break this down step by step:
  * `3>&1`: We create our own backup pointer (`3`) and make it point to where `1` (successes) is currently pointing.
  * `1>&2`: We change pointer `1` (successes) to point to where pointer `2` (errors) is pointing.
  * `2>&3`: Finally, we make pointer `2` point to pointer `3`, which holds the original path for successes.
  * *Result:* We have perfectly swapped the pointers without losing data. Pointer `1` now holds errors, and pointer `2` holds successes.

## Cron Jobs

Cron jobs are simply automated tasks scheduled to run at a specific time, either repeatedly or just once.

The trickiest part of cron jobs is understanding how to configure the exact moment(s) they should execute. To master this, we need to understand the 5 configurable parameters: `* * * * *`. If we configure a task literally with these five asterisks, it will execute every single minute of every single day.

* `First *`: Refers to the **minute** (0-59). For example, `2 * * * *` will run at minute 2 of every hour, every day. If we use a slash like `*/2 * * * *`, it will run EVERY 2 minutes.
* `Second *`: Refers to the **hour** (0-23). For example, `0 14 * * *` will run exactly at 14:00 (2 PM) every day. *Note: If you use `* 14 * * *`, it will run every single minute during the 2 PM hour!* If we use `0 */2 * * *`, it will run exactly on the hour, every 2 hours.
* `Third *`: Refers to the **day of the month** (1-31). `0 0 15 * *` will run at midnight on the 15th of every month. If we use `0 0 */2 * *`, it will run every two days.
* `Fourth *`: Refers to the **month** (1-12). `0 0 * 5 *` will run daily at midnight, but only in May. If we use `0 0 * */5 *`, it will run every 5 months.
* `Fifth *`: Refers to the **day of the week** (0-6, where 0 is Sunday and 6 is Saturday). `0 0 * * 5` will run at midnight every Friday. If we use `0 0 * * */2`, it will run every 2 days of the week.

---

> **🌟 Note:** This cheat sheet will be constantly updated and growing. I hope you find it as helpful as I do! If it did help you, leaving a **Star (⭐)** on this repository would mean a lot to me and help me keep pushing forward. Happy Hacking! 👾
