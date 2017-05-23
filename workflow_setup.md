# Workfow Setup Instructions
1. Download Sublime Text 3.
2. Install Sublime Package Control.
3. Using Package Control, install the package Sublime SFTP.
4. Create a directory on your computer named `devsheaconloncom`.
5. Use `File > Open...` to open this directory in Sublime Text.
6. Right-click on `devsheaconloncom` in the directory tree in the sidebar and select `SFTP/FTP > Map to remote...`.
7. Edit the configuration file to look like this:
	```json
		{
		    "type": "sftp",

		    "upload_on_save": true,
		    "confirm_sync": false,
		    
		    "host": "sheaconlon.com",
		    "user": "ubuntu",
		    
		    "remote_path": "/var/devsheaconloncom",
		    
		    "extra_list_connections": 1,

		    "connect_timeout": 30,
		    "keepalive": 120
		}
	```
8. Right-click on `devsheaconloncom` and select `SFTP/FTP > Sync Remote -> Local...`.
9. Create an SSH key and set it up for authentication with Github.
	1. Generate the key by running `ssh-keygen -t rsa -b 4096 -C "<youremail>"`.
	2. Save it in the file `~/.ssh/github`.
	3. Enter a passphrase of your choosing. Retain the passphrase.
	4. Edit `~/.ssh/config` by adding the following.
		```
		Host *
			AddKeysToAgent yes
			UseKeychain yes
			IdentityFile ~/.ssh/id_rsa
		```
	5. Start the SSH agent by running `eval "$(ssh-agent -s)"`.
	6. Add your key to the SSH agent by running `ssh-add -K ~/.ssh/github`.
	7. Copy your public key to the clipboard by running `pbcopy < ~/.ssh/github.pub`.
	8. In the SSH and GPG Keys pane of your Github settings, choose `New SSH Key`.
	9. Title it and paste in the contents you copied earlier.
9. Create a Github repository.
9. Open `devsheaconloncom` in the command line.
10. Run `git init; git remote add origin <sshaddressofrespository>; git pull origin dev`.
13. On the server, go to `/var/sheaconloncom`.
14. Run `git init; git remote add origin <sshaddressofrespository>; git pull origin master`.

To develop, simply edit the files on your computer in `devsheaconloncom`. Your edits will get sent to the server when you save and will be immediatley visible on the development site, `http://dev.<yourdomainname>`. Commit your edits. On the server, go to `/var/sheaconloncom`, pull the dev branch, and merge the dev branch into master. Then, your changes will be visible on the production site, `http://<yourdomainname>`.
