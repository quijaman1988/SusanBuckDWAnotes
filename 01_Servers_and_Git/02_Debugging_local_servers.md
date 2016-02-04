## Log files
If your MAMP or XAMPP Apache or MySQL server won't start, the first place you want to check for clues is your log files.

Here are the log file locations:
+ MAMP: PHP, Apache and MySQL logs are all found in `/Applications/MAMP/logs`
+ XAMPP:
    + PHP errors: `C:\xampp\php\logs`
    + Apache errors: `C:\xampp\apache\logs`
    + MySQL errors: `C:\xampp\mysql\data\mysql_error`
    + Alternatively, from the XAMPP Control Panel, there's a *logs* button next to each service.


## Port issues
The #1 cause of MAMP or XAMPP not being able to start Apache or MySQL is a conflicting port issue, that is, some other service on your computer is already using the port that Apache or MySQL is trying to use.

Ideally, we'd like to use Apache's default port `80` and MySQL's default port `3306`.




## Freeing ports on Windows
First, __restart your computer__. Some students have reported that after a fresh install of XAMPP, this step is needed.

The next easiest fix to try is close Skype if you have it open, then trying to start your server again. Skype on Windows is known for causing port conflicts with local servers.

If the above &ldquo;low hanging fruit&rdquo; steps don't work, download and open [CurrPorts](http://www.softpedia.com/get/Network-Tools/Network-Information/CurrPorts.shtml), a free program to easily see and terminate processes on Windows. There are other ways you can manage your ports on Windows, including via the Command Line or Task Manager, but I've found CurrPorts to be the most straightforward so it's worth the extra download.

__If you're having a port issue with MySQL:__
With CurrPorts running, look for processes named `mysqld.exe` (MySQL), right click them and choose *Kill Processes of Selected Ports*. Then right click again and choose *Close Selected TCP connections*.

__If you're having a port issue with Apache:__
Follow the same steps as above, but search for processes labeled `httpd.exe` (short for *HTTP daemon*; it's equivalent to Apache) and terminate them.

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-curr-ports@2x.png' style='max-width:729px; width:100%' alt='CurrPorts'>

You can also look under the __Local Port__ column to find processes that are using your default MySQL (`3306`) or Apache (`80`) port.

After you've identified and terminated the appropriate processes, return to XAMPP and attempt to restart your server.




## Freeing ports on a Mac
Load `Activity Monitor`, a program default to your Mac that gives lots of details about your system.

You can find this program by typing `Activity Monitor` into Spotlight (the little magnify glass on the top right of your screen).

__If you're having a port issue with MySQL:__
Under the `CPU` column, search for `mysql`. Look for processes labeled `mysql`, select them, and then click the X on the top left to terminate them.

<img src='http://making-the-internet.s3.amazonaws.com/sysadmin-mac-activity-monitor@2x.png' style='max-width:771px; width:100%' alt='Activity Monitor'>

__If you're having a port issue with Apache:__
Follow the same steps as above, but search for processes labeled `httpd` (short for HTTP daemon, it's equivalent to Apache) and terminate them.

After you've identified and terminated the appropriate processes, return to MAMP and attempt to restart your server.


## Conclusion
If the above actions don't set you on the right track, here are your next steps:

+ Open your Apache or MySQL log files and look at the recent errors.
+ Google search for key phrases in these errors to see solutions provided by other users with the same issue.
+ Post a thread in Piazza documenting your issue, including relevant log information.
