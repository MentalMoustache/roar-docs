# Roar server Startup and Shutdown

Instructions for starting up and shutting down a Roar Engine server in production environments.

## Startup
Running a test version of the server can be easily done through `roarengine {game.conf}` - however in production, you'll want to execute the server through our `restarter.bash` file, which acts as a loader for the server, and attempts to restart it in the event of a crash.

### restarter.bash
This file is located in `/engine/tools/restarter.bash`. The restarter uses `mail` to send you an update. Ensure mailutils are installed:

    sudo apt-get install mailutils

And update the restarter `EMAILADDRESS` (line 2) to an address you would like to receive information on.

Run the restarter as follows:

    nohup ./engine/tools/restarter.bash ./engine/roarengine ./config/default/linux.conf &

`nohup` ensures the process doesn't terminate when you exit your shell, and the `&` backgrounds the process.

You may also `tail` the `serverlog` file to observe the output of the server as it starts up. In a separate terminal, prior to restarting, you can:

    tail -f serverlog

### Restarting after shutdown
Having shut down the server using the instructions below, restarting the server should be done in the following order:

1. Restart the server using the instructions above
2. Move/rename the "unavailable" `.htaccess` to return to normal request handling

Watching the `serverlog` (see above) will provide information about startup. Run a simple API call such as `/info/ping/` to ensure the server is responding correctly.

Excellent - you're running Roar! Now to shut it down.

## Shutdown
Roar runs in memory, periodically slow-writing its updates down to a datastore. Which means if you simply kill the `roarengine` server process, there's a good chance that you'll lose a few actions that were in memory, but not yet written to the database. To solve this, you can use the `/engine/tools/syncplayers.py` script. But before you do we recommend putting your web layer into "Server unavailable" mode

### Setting the web layer to redirect
Shutting down the server without modifying the web layer will result in:

- continued activity against your server while you are trying to 'save' a snapshot.
- socket connection errors once the server is down

The former point is the bigger problem. While shutting down the Roar server, you want to save the latest data down to your database. This is impossible to cleanly transact if you are still receiving connections. The best option is to inform the web layer to not route connections, and return an 'unavailable' message. In `/engine/tools/htaccess_unavailable` you will find the following:

    <IfModule mod_rewrite.c>
      RewriteEngine on
      RewriteRule ^$ unavailable.php [L]
      RewriteRule (.*) unavailable.php [L]
    </IfModule>

Rename the `htaccess_unavailable` to `.htaccess` and place this (and the `unavailable.php`) into your web root. Your web layer will no longer connect to the server. You may now save down your server to memory (see next step).

### syncplayers.py
This script simply triggers a write of current memory down to the database (rather than waiting for it to automatically happen every few minutes). To call:

    ./syncplayers.py {serverport}
    
Where `{serverport}` is the port that you have configured to run the `roarengine` server.

### Kill the server
Once you are satisfied that you have written down the server contents, killing the server is as simple as `kill {restarterpid} {serverpid}`.

For the command line junkies, simply change 'YOURGAME.conf' to match the name of the `.conf` file you're using:

    ps axww | grep YOURGAME.conf | egrep '(restarter.bash|roarengine)' | grep -v grep | awk '{ print $1; }' | xargs kill

Note, unless you have updated your web layer `.htaccess` to point to a "Server currently unavailable" file, you will receive socket connection errors if you try and access the API. This is expected, while your server is down. Restart the server to restore connection.
