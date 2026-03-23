# Linux Commands

## Initial Commands

* `whoami`: Shows the current logged-in user in the terminal.
* `id`: Displays your user ID (UID) and the groups you belong to (GID).
* `cd [directory]`: Changes the current directory. Use `cd /` to go to the root directory, or `cd ..` to go back one level.
* `ls`: Lists files and directories in the current path.
  * `-a`: Flag that shows all elements, including hidden files (starting with a dot).
  * `-l`: Flag that shows detailed information (permissions, owner, size, date).
* `pwd`: Print Working Directory. Shows the absolute path you are currently in.
* `exit`: Logs out of the current session or closes the terminal.
* `su [user]`: Switch User. Allows you to switch the current session to the specified user.
* `sudo [command]`: SuperUser DO. Executes a specific command with root (administrator) privileges.
* `cat [file]`: Print the contents of the specified file via terminal.
* `which [executable]`: Displays the absolute path of an executable or command. Useful for checking if something is installed.

## User and Groups Commands

* `adduser`: It's an interactive command to create users; it asks for your information and then automatically creates the user directory in `/home/[user]`.
* `useradd`: It creates the user but doesn't add a user folder or set a password, so it's disabled until it's configured manually (useful for automations).
* `addgroup`: It's a more interactive and user-friendly command; the differences are similar to those of adduser and useradd.
* `groupadd`: Adds groups to the system directly, without questions or interaction.
