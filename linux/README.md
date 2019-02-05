# Welcome to the FCHS Treatise of Linux Security v0.1

## PREFACE
---
This checklist is by no means a complete representatition of all activities you should focus on in your images. Please use these commands only in reference to the README provided with the images -- make sure you are not doing anything that runs contrary to the README, or you might lose points!

Otherwise, we hope this checklist helps out. Good luck!

## EXPOSITION
---
1. Before **_absolutely anything_**, make sure you read the README provided with the images in full to make sure you know what to do and not to do. 

2. Complete the Forensics questions next. It is advised you do the questions _before_ you do anything else, as you might delete referenced data accidentally.
	* If you do not recieve points for these forensics questions, 95% of the time it is because you have the wrong answer. For the other 5%, check up on the [CyberPatriot Discord](https://discord.gg/7r8NxXk), or ask about on reddit

3. It is also advised to change your user password... you'll be using the `sudo` command a bunch, and it's much easier to work with a password you can easily use! 
	* use the `passwd` command to do this

## RISING ACTION
---
1. Update the system  
    * Open the System Settings from the top left dropdown menu. They can also be accessed from terminal using `unity-control-center`. 
        * Find `Software & Updates`. Check all boxes, except for `Source code` under the `Ubuntu Software` tab.  
        * In the `Updates` tab, select everything except for `backports`. Make sure to set update period to `Daily`, and to notify `Immediately`. Download and install security updates automatically.    
    	* In the terminal, run `sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade`. Wait for this to complete.  

2. Remove obvious games & hacking tools  
    * Open `Software` from the top left tab, or run `ubuntu-software-center`.  
        * Scan for games, or hacking materiel. Delete anything that looks like it could concievably be a form of entertainment.
   	    * An alternative method is to run `sudo dpkg -l` and hunt for any bad programs from there.  

3. Secure user vulnerabilities  
    * Disable the guest user account  
        * Open `/etc/lightdm/lightdm.conf` and add `allow-guest=false` to the bottom
    	* If README specifies "only allow users root permissions with sudo", run the command `sudo passwd -l root`. 
    	This will effectively disable the root account, so make sure you maintain access to your sudo permissions  
    	* Open `/etc/passwd`, and compare it with the list of allowed users in the README.  
        	* Check for users that should not exist 
			* Delete these users with `sudo userdel -r <username>`  
       		 Make sure only root has UID 0  
        	*  Make sure user accounts open with `/bin/bash`, and not `/usr/sbin/nologin` or related fields.  
                * *NOTE*: Some users are simply processes, such as `usbmux` and `avahi-autoipd`. These you can *usually* ignore.
    	    * Check `/etc/group` and find the `sudo` and `adm` groups.  
        	    * Check for non-administrators in either group. Remove users from group with `sudo deluser <username> <groupname>`  
        	    * Look for suspicious groups, such as a group of non-administrators. Delete these groups with `sudo groupdel <groupname>`   
	        * Run `sudo visudo` and make sure only users of `%sudo` can sudo.  

3. Change password requirements  
    * In `/etc/login.defs`, set:  
   	    ```
    	PASS_MIN_DAYS 7
    	PASS_MAX_DAYS 90
    	PASS_WARN_AGE 14
    	```
    *  In `/etc/pam.d/common-password`:  
        * Append `minlen=8 remember=5` to the end of the line with `pam_unix.so`  
        * Run `sudo apt-get install libpam-cracklib`, then re-open `/etc/pam.d/common-password`.  
            * Append `ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1` to the end of the line with `pam-cracklib.so` in it  
    	*  Open `/etc/pam.d/common-auth` with sudo  
        	* Search for `pam_tally2.so`. If it is not present, add:  
        	`auth required pam_tally2.so deny=5 unlock_time=1800`   
        	to the bottom of the file.   
        	* If the file is present, just append `deny=5 unlock_time=1800` to the end of the line  
    	* Change every user password. Sometimes, users will not have a password set.
        	* `sudo passwd <user>` works with this  
        	* You can set every user password to the same thing (like `cb5b4de$TNY2p`), or random passwords at your discretion.  


## CLIMAX
---
1. Hunt down illegal files
    * Traditionally, CP only puts 'disallowed' files in the `/usr/share/` and `/home/` directories, but this rule is not set in stone
        * Be wary of where they may put the files, though. Sometimes the directories are invisible or difficult to find, or the files may be hidden
        * Try running the following command to find the majority of possible files (run it in both /home/ and /usr/share/), **it will output the results to** `find_results.txt`.  
        	```
            find . -type f \( -name "*.mp3" -o -name "*.mov" -o -name "*.mp4" -o -name "*.avi" -o -name "*.mpg" -o -name "*.mpeg" -o -name "*.flac" -o -name "*.m4a" -o -name "*.flv" -o -name "*.ogg" -o -name "*.gif" -o -name "*.png" -o -name "*.jpg" -o -name "*.jpeg" \) > find_results.txt
            ```
        * *NOTE:* You might end up with a lot of junk (z.B. from program data, like `~/<youruser>/.mozilla/...)`, this stuff is normal and you can just skip over it.  
            * Hunt for conspicuous stuff, like Piano Concertos or [Dank Memes](https://thumbs.gfycat.com/AgitatedGaseousKitfox-mobile.jpg)  

2. Check cron  
    * Run `sudo crontab -r`. This will reset the root crontab file, as there should be nothing in it.
        * For extra verification, create a `croncheck.sh` file. Open it and add:  
    	`for user in $(getent passwd | cut -f1 -d: ); do echo $user; crontab -u $user -l; done`  
            * run `sudo chmod +x croncheck.sh`  
        	* then `sudo bash croncheck.sh`  
                * the output of this will show the user, followed by their cron. If no cron jobs are present, it will display `no crontab for <user>`  
3. SSH Security  
    * Open `/etc/ssh/sshd_config`  
         * Confirm `PermitRootLogin no`  
         * Confirm `Protocol 2`  
        * Make sure all `HostKeys` are uncommented under `# HostKeys for protocol version 2`  
        * Confirm `UsePrivilegeSeparation yes`  
        * Confirm `LoginGraceTime 120`  
        * Confirm `IgnoreUserKnownHosts yes` is uncommented  
        * Confirm `PermitEmptyPasswords no`  
        * Confirm `UsePAM yes`  
        * Look around on the web for other examples of securing `sshd_config`.  
4. Ports   
    * Using `sudo netstat -lntu`  
        * You can see ports that are listening if their `State` is `LISTEN`. These are pretty much the only ports that you need to worry about.  
            *  Check to see what ports should be open & listening.   
        		* For instance, if the README specifies that `Samba` is a critical service, then ignore opened ports that belong to the process `smbd` and it's requirements.
        	    * Check the names of programs associated with each port with `sudo lsof -i :$port`, while `$port` == port you want to check  
        	    * If the program is unneeded, delete with `sudo apt-get purge $program`  
        	    * Confirm port is closed w/ `sudo netstat -lntu`  
5. Firewall  
    * Enable the firewall with `sudo ufw enable`  
    	* Setting up the firewall:  
        	`sudo sh -c "echo 'net.ipv6.conf.all.disable_ipv6 = 1' >> /etc/sysctl.conf"`  
        	`sudo sh -c "echo 0 > /proc/sys/net/ipv4/ip_forward"`  
        	`sudo sh -c "echo 'nospoof on' >> /etc/host.conf"`  
    * Securing `/etc/sysctl.conf`  
        ```
	    sudo sysctl -w net.ipv4.tcp_syncookies=1  
	    sudo sysctl -w net.ipv4.ip_forward=0  
	    sudo sysctl -w net.ipv4.conf.all.send_redirects=0
	    sudo sysctl -w net.ipv4.conf.default.send_redirects=0
	    sudo sysctl -w net.ipv4.conf.all.accept_redirects=0
	    sudo sysctl -w net.ipv4.conf.default.accept_redirects=0
	    sudo sysctl -w net.ipv4.conf.all.secure_redirects=0
	    sudo sysctl -w net.ipv4.conf.default.secure_redirects=0
	    sudo sysctl -p
        ```  
6. Scanning services  
	* Run `sudo service --status-all`  
        * Most services will be legitimate Linux systems. Make sure to confirm the identity of **every single one** before you delete or disable unknowns.  
        	* if you stop or purge the wrong service, most of the time reinstalling it will give you points back.
   	* If the service is not listed in the README or is not a dependency (and again, make sure is not system-critical)
         `sudo service <service> stop`  
         `sudo apt-get purge <service>`  

## FALLING ACTION
---
1. Install & run some virus scanners (might take a while -- play cards with your mates or solitaire if you're soloing)  
    * `sudo apt-get install chkrootkit rkhunter lynis clamav`  
        * rkhunter:  
         `sudo rkhunter --update`  
         `sudo rkhunter --propupd`  
         `sudo rkhunter -c --enable-all --disable none`  
    	* ClamAV:  
         `sudo freshclam`  
         `sudo clamscan -r --bell -i /`  
            * this will check all files on computer, and will ring bell if infected files are found  
    	* chrootkit:  
        `sudo chkrootkit -q`

## Acknowledgements
---
Many, many thanks to Team A (Gentleman Ryan and Larry, and my gentlewoman Megan and Carolyn of Fort Collins HS). Hopefully you guys can fulfill the dream of Taffy Pizza at some point 😇.

Please take a look at some more comprehensive guides than this one too. Debian has an excellent security guide, and Reddit is plentiful with good checklists (shoutout to Forty-Bot).

![xkcd.com/2102](https://imgs.xkcd.com/comics/internet_archive.png "It's vaguely disquieting how pretty much all modern Linux distros exist as Frankensteinian mashups of 1990s and present day technology")
