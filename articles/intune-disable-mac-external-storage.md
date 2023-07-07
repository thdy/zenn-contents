---
title: "IntuneでmacOSの外部ストレージを禁止する"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Intune", "macOS"]
published: false
---

https://github.com/thdy/shell-intune-samples/tree/master/macOS/Custom%20Profiles/Disable%20External%20Storage

https://learn.microsoft.com/ja-jp/mem/intune/configuration/custom-settings-macos

![](/images/intune-disable-mac-external-storage/1686842461565.png)


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>PayloadContent</key>
	<array>
		<dict>
			<key>PayloadDisplayName</key>
			<string>Media Access:  Finder Settings</string>
			<key>PayloadIdentifier</key>
			<string>F3A4567A-DD40-4C18-A0C5-468A11F31635.finder</string>
			<key>PayloadType</key>
			<string>com.apple.finder</string>
			<key>PayloadUUID</key>
			<string>F3A4567A-DD40-4C18-A0C5-468A11F31635</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>ProhibitBurn</key>
			<true/>
		</dict>
		<dict>
			<key>PayloadDisplayName</key>
			<string>Restrictions</string>
			<key>PayloadIdentifier</key>
			<string>2A2740D0-0FF6-4075-B788-58197F83729D.systemuiserver</string>
			<key>PayloadType</key>
			<string>com.apple.systemuiserver</string>
			<key>PayloadUUID</key>
			<string>2A2740D0-0FF6-4075-B788-58197F83729D</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>mount-controls</key>
			<dict>
				<key>blankcd</key>
				<array>
					<string>eject</string>
					<string>alert</string>
				</array>
				<key>blankdvd</key>
				<array>
					<string>eject</string>
					<string>alert</string>
				</array>
				<key>cd</key>
				<array>
					<string>eject</string>
					<string>alert</string>
				</array>
				<key>dvd</key>
				<array>
					<string>eject</string>
					<string>alert</string>
				</array>
				<key>dvdram</key>
				<array>
					<string>eject</string>
					<string>alert</string>
				</array>
				<key>harddisk-external</key>
				<array>
					<string>eject</string>
					<string>alert</string>
				</array>
				<key>harddisk-internal</key>
				<array>
					<string>deny</string>
				</array>
			</dict>
		</dict>
	</array>
	<key>PayloadDisplayName</key>
	<string>Disable External Media</string>
	<key>PayloadIdentifier</key>
	<string>07242D4B-1FEF-4BA1-9A25-7259E1468109.alacarte</string>
	<key>PayloadOrganization</key>
	<string>Local</string>
	<key>PayloadRemovalDisallowed</key>
	<false/>
	<key>PayloadScope</key>
	<string>User</string>
	<key>PayloadType</key>
	<string>Configuration</string>
	<key>PayloadUUID</key>
	<string>07242D4B-1FEF-4BA1-9A25-7259E1468109</string>
	<key>PayloadVersion</key>
	<integer>1</integer>
</dict>
</plist>
```

![](/images/intune-disable-mac-external-storage/1686842924272.png)
![](/images/intune-disable-mac-external-storage/1686842915023.png)