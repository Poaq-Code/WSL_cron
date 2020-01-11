WSL\_cron
================
Poaq
January 10, 2020

What even is this
-----------------

You want to delete some garbage from downloads or temp off your machine on a regular basis? You want to automate this because it's 2020 and doing it manually is for scrubs? Cool. [Cron Jobs](https://en.wikipedia.org/wiki/Cron) can do this. Here's the issue for most people: you're not on a Unix machine. You're on Windows. Is there a way to do this in Windows? Probably, but I don't know Windows. I know Unix. Luckily, Windows10 has WSL, a Linux subsystem, built into Windows. What follows is a tutorial on how to set all this up. So, basically, we're going to:

-   Set up WSL on our machine
-   Write a cron job in WSL that will delete our downloads folder every day

Setting up WSL
--------------

This part is actually very simple. Windows has basically made it a tickbox + reboot situation. [Here's](https://lifehacker.com/how-to-get-started-with-the-windows-subsystem-for-linux-1828952698) an article from lifehacker talking about WSL features, what it is/isn't for, and how to install it. It also has links to other resources. But here's the TL;DR for installing:

-   Open settings
-   Search "turn windows features on or off"
-   Check the tick box next to "Windows Subsystem for Linux"
-   Reboot system
-   Go to Windows Store
-   Search "Linux" and pick whatever distro you want (personally I prefer Ubuntu)

And you're good to go. You now have a linux shell at your disposal. However, there are some caveats, the two main ones being (1) it does not support GUI applications (you can do some stuff with X11 servers to get it working, but... it's not worth it really) and (2) closing the terminal effectively "shuts down" your linux and anything it's running (this will come back when we talk about cron).

Cron
----

Again, people have tackled this before me, so [here's](https://scottiestech.info/2018/08/07/run-cron-jobs-in-windows-subsystem-for-linux/) a good article if you want to read more. But, basically, here are the steps:

-   Open up your linux terminal, and type `crontab -e`. It may ask which editor to use - I personally just like nano.
-   Make a cron job (more about that below) and save the file
-   Open ~/.bashrc
-   Add the line `service cron start` to the end of the file and save.

Now, and perhaps most importantly, **you must leave your linux terminal open for cron to work.** This is the biggest downside to cron via WSL - you *have* to keep that terminal open, or Windows will kill the process and your crontab won't work.

But, anywway, what's the syntax for this crontab file? At least in my WSL, it has an example that you can read (and [here's](https://www.youtube.com/watch?v=QEdHAwHfGPc) a good YouTube video about it). But, to be more explicit, a crontab works as follows:

`* * * * * [user] [command]`

where the asterisks are, in order:

-   minute (0-59)
-   hour (0-23)
-   day of month (1-31)
-   month (1-12)
-   day of week (0-6) (Sunday to Saturday - some systems use 7 for Sunday)

So, for example:

`* * * * * file.sh` would run `file.sh` every minute indefinitely.

`0 * * * * file.sh` would run `file.sh` every hour at the top of the hour.

`0 22 * * * file.sh` would run `file.sh` nightly at 10pm.

And so on.

Notes on WSL
------------

So, WSL is generally stored somewhere really weird on your PC. **Do not** try to edit "linux" components from the Windows environment - it tends to break stuff. Further, your default working directory is linux's `/home`, which is in this weird place. No worries, we can still navigate to our Windows files/directories. In general, Windows is stored under `/mnt`. So, for example, if we wanted to go to the Downloads folder, we might do:

``` bash
cd /mnt/c/Users/Username/Downloads
```

So we could delete all those files with something like:

``` bash
rm /mnt/c/Users/Username/Downloads/* # replace the asterisk with something like *.pdf if you just want to delete PDFs
```

So, if we're trying to delete our downloads folder every 24 hours, we might make a crontab entry like:

`0 22 * * * delDownload.sh`

where `delDownload.sh` is a file with the `rm` command above saved somewhere in your $PATH variable.
