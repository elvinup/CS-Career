Scripting framework that can automate actions on a Macbook

## Type Centro Password

```
#!/usr/bin/osascript

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title Type Centro
# @raycast.mode silent
# @raycast.packageName System

# Optional parameters:
# @raycast.icon 📋

# Documentation:
# @raycast.description Types out Centro Cred
# @raycast.author Thomas W. Holt Jr.

tell application "System Events"
	delay 0.05
	#set pwd to do shell script "lpass show 'Centro' | awk '/Password:/ { print $2 }'"
	set pwd to do shell script "op read op://Private/Centro/Password"
	keystroke pwd
	key code 36
end tell
```

## VPN toggle (Deprecated)

```
#!/usr/bin/osascript

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title Toggle AnyConnect
# @raycast.mode silent
# @raycast.packageName System

# Optional parameters:
# @raycast.icon 🆙

# Documentation:
# @raycast.description Toggles AnyConnect VPN
# @raycast.author Thomas W. Holt Jr.

set appName to "Cisco AnyConnect Secure Mobility Client"
set notificationTitle to "ToggleAnyConnect"
set credentialId to "SSO"
set authWindowTimeOut to 10 -- seconds
set successfulConnTimeOut to 30 -- seconds, may take some time for 2FA

activate application appName
tell application "System Events" to tell process appName
	click menu item "Show AnyConnect Window" of menu appName of menu bar 1

	-- UNDONE: window num can change depending on whether menu bar icon is enabled, need to be able to handle this
	set windowTarget to window 2

	if exists button "Disconnect" of windowTarget then
		display notification "Attempting to disconnect..." with title notificationTitle
		click button "Disconnect" of windowTarget

	else if exists button "Connect" of windowTarget then
		display notification "Attempting to connect..." with title notificationTitle
		click button "Connect" of windowTarget

		set deadline to (current date) + authWindowTimeOut
		set authWindowNamePrefix to "Cisco AnyConnect | "
		repeat until ((first window whose title contains authWindowNamePrefix) exists)
			if (current date) > deadline then my timedOut("waiting for authentication window")
			--delay 0.1
		end repeat
		set authWindow to (first window whose title contains authWindowNamePrefix)

		#set pwd to do shell script "/usr/bin/security find-generic-password -wl " & quoted form of credentialId
		set pwd to do shell script "lpass show 'force.com' | awk '/Password:/ { print $2 }'"

		set value of text field 2 of authWindow to pwd
		click button "OK" of authWindow

		set deadline to (current date) + successfulConnTimeOut
		repeat until window "Cisco AnyConnect - Banner" exists
			if (current date) > deadline then my timedOut("waiting for successful connection")
			--delay 0.1
		end repeat

		click button "Accept" of window "Cisco AnyConnect - Banner"

	end if

	click menu item "Hide Cisco AnyConnect" of menu "Cisco AnyConnect Secure Mobility Client" of menu bar 1
end tell

on timedOut(msg)
	display notification "Timed out " & msg with title notificationTitle
	error "Timed out " & msg
end timedOut

```