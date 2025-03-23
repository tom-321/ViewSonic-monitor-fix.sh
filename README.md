Needed: 

-[displayplacer](https://github.com/jakehilborn/displayplacer)
    
-[Hammerspoon](https://www.hammerspoon.org/)

If you have a slow Monitor on your MacBook you can do something like this, to continue working until the monitor's up:
 there is a tool **displayplacer** for MacOS, to switch the monitor, so it could be a work around for the 3-5 seconds til the monitors up. 
This is the script I wrote: 

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
<br/>
<br/>
<br/>

For loading the script at wake up of the display you need to trigger it via [Hammerspoon](https://www.hammerspoon.org/):
  
PS if any errors occur, check to console in Hammerspoon for logs
```
local lastScreenIDs = {}
local lastExecutionTime = 0
local cooldownTime = 15 -- Cooldown in seconds

function runMonitorFixScript(reason)
    local currentTime = hs.timer.secondsSinceEpoch()
    if currentTime - lastExecutionTime < cooldownTime then
        print("⏳ Cooldown active – Script will not run again.")
        return
    end
    lastExecutionTime = currentTime

    print("🚀 runMonitorFixScript() started by: " .. reason)
    hs.execute("/path/to/your/script.sh")
end

function getScreenIDs()
    local ids = {}
    for _, screen in ipairs(hs.screen.allScreens()) do
        table.insert(ids, screen:id())
    end
    table.sort(ids)
    return ids
end

function hasScreenChanged()
    local newIDs = getScreenIDs()
    if #newIDs ~= #lastScreenIDs then return true end

    for i, id in ipairs(newIDs) do
        if id ~= lastScreenIDs[i] then return true end
    end

    return false
end

-- Screen Watcher
screenWatcher = hs.screen.watcher.new(function()
    local screenCount = #hs.screen.allScreens()
    print("Screen Watcher: New monitor count: " .. screenCount .. " (Previously: " .. #lastScreenIDs .. ")")

    if hasScreenChanged() then
        print("✅ Changes detected: Monitor setup has changed!")
        lastScreenIDs = getScreenIDs()
        hs.timer.doAfter(2, function()
            print("⏳ Delay finished, script will run...")
            runMonitorFixScript("Screen Watcher: Monitor changed")
        end)
    else
        print("⚠️ No changes detected in the monitors.")
    end
end)

-- Caffeinate Watcher for Display Wakeup
caffeinateWatcher = hs.caffeinate.watcher.new(function(event)
    if event == hs.caffeinate.watcher.screensDidWake then
        print("🖥️ Screen awakened – Checking monitors...")

        -- Check immediately
        if hasScreenChanged() then
            print("✅ Change detected immediately after wakeup!")
            lastScreenIDs = getScreenIDs()
            runMonitorFixScript("Caffeinate Watcher: ScreensDidWake")
        else
            print("🔄 No immediate change detected – rechecking in 5 seconds...")
            
            -- Check again after 5 seconds
            hs.timer.doAfter(5, function()
                if hasScreenChanged() then
                    print("✅ Change detected after 5 seconds!")
                    lastScreenIDs = getScreenIDs()
                    runMonitorFixScript("Caffeinate Watcher: Delayed detection")
                else
                    print("⚠️ No detectable change, running script anyway.")
                    runMonitorFixScript("Caffeinate Watcher: Safety execution after wakeup")
                end
            end)
        end
    end
end)

-- Start Watchers
lastScreenIDs = getScreenIDs()
screenWatcher:start()
caffeinateWatcher:start()


```
