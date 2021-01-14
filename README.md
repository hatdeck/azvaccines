# Vaccine Download

Script and launch agent to download AZDHS 'vaccine-phases.pdf' daily

Launch Agents are programs you can create to run automatically in the background
when you are logged in. The following steps will help you edit the launch agent
template in this repository to allow the script in this repository to run
automatically.

## NOTES:

1. Syntax: The syntax \<some text\> means you have to replace that text with 
something you have chosen. For example if you're supposed to run the command
`mv <path to file> /user/local/bin` you need to find the path to the file in
question (e.g., `~/Downloads/vaccine_download`) and run the following:
`mv ~/Downloads/vaccine_download /usr/local/bin`. Do not include the angle
brackets in the command you type into a terminal session.

2. Checking file timestamp: The launch agent is written to run at 12:15 pm your
local time each day. This is to allow enough time for ADHS to update the file for
the day. The script checks the file metadata to be sure that it has been updated 
the sameday the script is run. It will still download the file if it is old, but 
it will alert you to that fact in the log, and keep trying every 30 minutes for 
4 hours until the ADHS updates the file. If ADHS has not updated the file by the
5th try, you will see a starred box in the log file telling you to run it
manually again if you'd like. Step 4 below shows how to run the launch agent
manually.

3. Running on a sleeping computer: If your machine is sleeping when the script is 
scheduled, it will run the next time the computer wakes up. If your machine 
sleeps at some point during a 30 minute interval between download attempts, the 
counter that sets that 30 min interval will also pause, and may not restart. The
solution here is to kickstart it (see troubleshooting). If your machine regularly
sleeps between 12:15 and 4:15, you may want to change the start time, interval
between retries, or number of retries.


## Steps

1. Allow file `vaccine_download` to be used as a command. If you don't know how
to do this, follow these steps for a simple method. Note: this isn't strictly
necessary, but will be helpful if you want to run this script manually.
    1. Place the file in the directory `/usr/local/bin`, e.g., with the command
    `mv <path to file> /usr/local/bin`.
    2. Check the permissions of the file, e.g., with the command
    `ls -l /usr/local/bin/vaccine_download`
    3. If the file doesn't have user and group execute permissions (result of
    prior command does not start with -rwxr-x), run the command
    `chmod 750 /usr/local/bin/vaccine_download`
    
2. Edit the file: category.vaccine_download.plist
    1. Chose a category for your launch agent. It can be anything you'd like,
    but it is helpful to use something that makes finding information about the
    launch agents you create easier. A good choice might be your username on
    this computer.
    2. On line 6, change `**EDIT**category.vaccine_download` to
    `<THE CATEGORY YOU JUST CHOSE>.vaccine_download`
    3. On line 10, verify the location of the vaccine_download executable file.
    If you used the above instructions for how to allow `vaccine_download` to be
    used as a command, the path will be `/usr/local/bin/vaccine_download`, and
    you simply have to remove `**VERIFY**`. If not, use the ABSOLUTE LITERAL
    path to the executable. I recommend not using any variables or aliases in
    that path, as launch agents are run in a sparse environment and variables or
    aliases may not behave as expected.
    4. On line 11, change `**EDIT**Users/username/Projects/COVID` to the
    absolute literal path for whatever directory you want to use for this
    project (i.e., the directory where you want the files to download, and where
    you want the log file to go).
    5. On lines 23 and 25, as above, choose a location and filename for your log
    file and edit the strings accordingly. Yes, you need a log file. In the
    template, the log file and data files are in the same directory, and the log
    file is named AZvaccine_download.log. You can do use if you'd like, or
    choose another directory and name for your log file, but if you do, you'll
    have to edit the executable file, `vaccine_download` by replacing the line
    `open AZvaccine_download.log` with `open <path to the log file you chose>`
    
3. Load the launch agent
    1. Move the .plist file into your user LaunchAgents directory. Run:
    `mv <path to .plist file> ~/Library/LaunchAgents/`
    2. Load the launch agent. Run:
    `launchctl load ~/Library/LaunchAgents/<your.plist>` Note: be sure to use
    the actual full file name (e.g., `<your_username>.vaccine_download.plist`)
    3. If the launch agent loads successfully, there will be no output.
    
4. Test the launch agent.
    1. Run the launch agent manually to confirm it behaves as expected. To run
    manually, run:
    `launchctl start <category>.vaccine_download`
    Note that here you are NOT using the file name for the .plist file, but
    rather the "Label" defined in that .plist, which should be the same as the
    filename, but without the .plist file extension. Both the log and the file
    itself should open fairly quickly, unless there are errors.
    
5. Troubleshooting
    1. File not updated that day. I expect there to be some days where ADHS
    doesn't update this file. If that happens, an old file will be downloaded
    but assigned a filename with the current date. The pdf itself is supposed to
    have a human readable date, so it should be readily apparent that a given
    file named, e.g., AZvaccines2021-04-14.pdf, but with a date stamp of, e.g.,
    Updated: 4/13/2021, represents a record of a day without an update.
    2. File not updated YET. This is a problem that should be caught. As
    discussed above, the script checks the file metadata, and retries once every
    30 minutes for up to 4 hours if it has not updated yet. Watch for late
    updates.
    3. curl error: If the script catches an error in the download utility, curl
    (e.g., unable to reach the server), it will catch that error, stop the
    script, report the error by opening the log. Troubleshoot, e.g., internet
    connectivity, azdhs.gov issues, url changes, and the relaunch manually as in
    item 4, Test the launch agent.
    4. Other errors: If the launch agent or script encounters other errors, the
    log may not open to alert you. Two suggested paths to investigate this are:
    check the log directly, and also check launchctl list for errors with the
    following command:
    `egrep <category> <(launchctl list)`
    Note: \<category\> is the first part of the launch agent label and the .plist
    filename. You chose it above in step 2.1. Note also that the left angle
    bracket in \<(launchctl list) needs to be in the command (unlike the angle
    brackets in \<category\>, which represent a name you should replace)
    5. Editing the launch agent after it has been loaded. If you make changes
    to the launch agent once it has been loaded, unload it and then reload it
    `launchctl unload ~/Library/LaunchAgents/<yourplistfile>`
    `launchctl load ~/Library/LaunchAgents/<yourplistfile>`
    If you make changes to the script itself (`vaccine_download`), they will
    go into effect as soon as they are saved and the script is run. You do not
    need to reload the launch agent for changes to the script only.
    6. Kickstarting the launch agent. If the launch agent has started, but the 
    script does not appear to be making progress, try kickstarting it. You can 
    identify this state by running `egrep <category> <(launchctl list)` while
    viewing the log. The first column in the output is either `-` or a PID, an 
    integer such as 22183. If no PID is displayed, the launch agent isn't 
    running. If you want it to run, you can run it manually as described in 4.1.
    If a PID is displayed, it means the launch agent is currently running. If 
    you expect output in the log, `AZvaccine_download.log`, but none is present, 
    kickstart the launch agent with the following:
    `launchctl kickstart -k gui/$UID/<category>.vaccine_download` 
