# How I automated charging process for my Macbook

## Why
I wanted to properly cycle my battery and not leave the plug on all the time, just to preserve my battery life as well.

## Requirements
1. Macbook
2. Smart Plug that has IFTTT integration
3. IFTTT account

## How


IFTTT:
1. Signup for an [IFTTT](https://ifttt.com) account
2. Create an applet
3. Select `WebHook` service for `If This` condition
4. Select your smart plug integration for `Then That` condition
5. Get your maker webhook url via maker documentation


https://maker.ifttt.com/trigger/{event}/with/key/{yourMakerKey}

replace {event} with the events created in step 2.

example maker url to turn off: 
`https://maker.ifttt.com/trigger/mac_battery_low/with/key/{yourMakerKey}`





Open AppleScript Editor, and paste this code block:
```applescript
set chargeState to do shell script "pmset -g batt | awk '{printf \"%s %s\\n\", $4,$5;exit}'"
set percentLeft to do shell script "pmset -g batt | awk -F '[[:blank:]]+|;' 'NR==2 { print $4 }'"

considering numeric strings
    -- here you can set the percentage which you want to turn on the smart plug (currently set to 15)
	if chargeState contains "Battery Power" and percentLeft ≤ 15 then
        -- replace {{makerUrlToTurnOnPlug}} to the webhook you get from your IFTTT webhook maker
		do shell script "curl -X POST {{makerUrlToTurnOnPlug}}"
	end if

	-- here you can set the percentage which you want to turn off the smart plug (currently set to 95)
	if chargeState contains "AC Power" and percentLeft ≥ 95 then
        -- replace {{makerUrlToTurnOffPlug}} to the webhook you get from your IFTTT webhook maker
		do shell script "curl -X POST {{makerUrlToTurnOffPlug}}"
	end if
end considering
```

Save it wherever you want.

Next, 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>automate-battery.job</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/osascript</string>
        <string>location/of/your/applescript.scpt</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>300</integer>
</dict>
</plist>
```

Save with `.plist` extension.

_for example, let's save it as `automated-battery.plist`_


Quick explanation for this `.plist` file

Taken from [launchd.plist](https://www.manpagez.com/man/5/launchd.plist/) documentation

>`RunAtLoad` key is used to control whether your job is launched once at the time the job is loaded. The default is false.

> `StartInterval` this key causes the job to be started every N seconds. If the system is asleep, the job will be started the next time the computer wakes up.  If multiple intervals transpire before the computer is woken, those events will be coalesced into one event upon wake from sleep.

