If you have a slow Monitor on your MacBook you can do something like this, to continue working until the monitor's up:
 there is a tool **displayplacer** for MacOS to switch the monitor so it could be a work around for the 3-5 seconds til the monitors up (for people who use a MacBook). This is the script I wrote: 

```
#!/bin/bash

# Duration for which the internal monitor is continuously enforced (in seconds)
ready_timeout=10

echo "Forcing internal monitor as primary display continuously..."

# Start a loop that forces the internal monitor for 'ready_timeout' seconds
start=$(date +%s)
while [ $(( $(date +%s) - start )) -lt $ready_timeout ]; do
    displayplacer "id:>>BUILD-IN-MONITOR-ID<< res:1512x982 hz:120 color_depth:8 enabled:true scaling:on origin:(0,0) degree:0" \
                  "id:>>EXTERNAL-MONITOR-ID<< res:1920x1080 hz:60 color_depth:8 enabled:true scaling:off origin:(1512,0) degree:0"
    sleep 0.5
done

echo "Time's up – external monitor is assumed to be 'ready'."
echo "Now switching to the external monitor as the primary display..."

# Set the external monitor as the primary display
displayplacer "id:>>EXTERNAL-MONITOR-ID<< res:1920x1080 hz:60 color_depth:8 enabled:true scaling:off origin:(0,0) degree:0" \
              "id:>>BUILD-IN-MONITOR-ID<< res:1512x982 hz:120 color_depth:8 enabled:true scaling:on origin:(-1512,0) degree:0"

echo "External monitor set as the primary display."

```



Just replace >>EXTERNAL-MONITOR-ID<< and >>BUILD-IN-MONITOR-ID<< with the ID of your monitor ;)


```
PS for loading the script at login (wake up of the display) you need to trigger it via Hammerspoon
-- Create a Caffeinate Watcher that reacts to unlocking
local watcher = hs.caffeinate.watcher.new(function(event)
if event == hs.caffeinate.watcher.screensDidUnlock then
hs.execute("/Users/USERNAME/scripts/monitorfix.sh")
end
end)
watcher:start()

```
