# How I automated charging my Macbook

## Why

I know MacOS already has a smart battery management.<br>
This is just solely my preference, I don't like leaving my Mac plugged in all the time, also I want to cycle my battery properly to prolong battery life(?).<br>
Lastly, I'm lazy to turn on/off my charger every time my battery is full or low.<br>
So if your in the same boat as me, feel free to follow the guide.

To achieve what I want, I decided to buy a cheap [Smart Power Strip](https://www.lazada.com.ph/products/joylab-wifi-smart-power-stripwifi-smart-socket-with-overload-protection-with-4-ac-outlets-and-4-usb-31a-no-hub-required-works-with-smart-lifetuya-amazon-alexa-and-google-home-i874650801-s2776138630.html) that has individual control and supports IFTTT.<br>
You can also do this with a smart plug that has IFTTT support if you just want one specifically for your MacBook.

## Requirements
1. Macbook
2. Smart Plug or Smart Power Strip with individual control that has IFTTT support
3. IFTTT account

## How
#### IFTTT:

1. Signup for an [IFTTT](https://ifttt.com) account
2. Create an IFTTT applet
3. Select `WebHook` service for `If This` condition
4. Select your smart plug integration for `Then That` condition
5. Get your maker webhook url via maker documentation
    * to get this, go to your account > services > WebHooks > Documentation
    * it would like something like this: [https://maker.ifttt.com/trigger/{event}/with/key/{yourMakerKey}](https://maker.ifttt.com/trigger/%7Bevent%7D/with/key/%7ByourMakerKey)
6. replace `{event}` with the event name you created

> example maker url to turn off:
> 
> event name = mac\_battery\_low<br>
> So my url is: [https://maker.ifttt.com/trigger/mac\_battery\_low/with/key/{yourMakerKey}](https://maker.ifttt.com/trigger/mac_battery_low/with/key/%7ByourMakerKey%7D)


#### MacBook:

Open AppleScript Editor on your Mac, paste the code block below:

``` applescript
set chargeState to do shell script "pmset -g batt | awk '{printf \"%s %s\\n\", $4,$5;exit}'"
set percentLeft to do shell script "pmset -g batt | awk -F '[[:blank:]]+|;' 'NR==2 { print $4 }'"

considering numeric strings
    -- here you can set the percentage which you want to turn on the smart plug (currently set to 15)
	if chargeState contains "Battery Power" and percentLeft ≤ 15 then
        -- replace {{makerUrlToTurnOnPlug}} to your webhook url to turn on the smart plug
	    do shell script "curl -X POST {{makerUrlToTurnOnPlug}}"
	end if

	-- here you can set the percentage which you want to turn off the smart plug (currently set to 90)
	if chargeState contains "AC Power" and percentLeft ≥ 90 then
        -- replace {{makerUrlToTurnOffPlug}} to your webhook url to turn off the smart plug
	    do shell script "curl -X POST {{makerUrlToTurnOffPlug}}"
	end if
end considering
```
Save this wherever you want.

#### launchd:
Next, we would need a launchd or a cron in Linux terms to execute the AppleScript we've just created in intervals.<br>
We can do this in MacOS by creating a `.plist` file in `/Library/LaunchAgents/`<br>
*to understand more about*`launchd.plist`*, you can read this [documentation](https://www.manpagez.com/man/5/launchd.plist/).*

Open your favorite code editor, then paste the code block below:
``` xml
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
1. Make sure to update the file location in line 10 to the location where you stored the AppleScript you've created
2. You can change the interval in line 15 based to your liking, currently set to `300` seconds
3. For best practice save this file similarly to your label, example would be `automate-battery.plist`
4. If you get an error saving the `.plist `file in `/Library/LaunchAgents/`, try saving it again with admin privilages

Automation all set!
