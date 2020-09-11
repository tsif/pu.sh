# pu.sh

A bash script to send iOS push notifications with the Apple Push Notification service (APNs)


## About

In order to test iOS push notifications you might have your own server setup or maybe you use one of the many push notification services that already exist. All this takes a lot of time and effort to get up and running. Ideally, you would probably like to start sending notifications just with a mac, an iOS device and zero in cash (excluding the money you've paid for your mac and device of course). With the use of web tokens and the shell script contained herein you can start sending push notifications to your devices in minutes while getting some feedback each time something goes wrong.

## Features

- Uses JSON web tokens instead of certificates. Faster and painless
- Displays error messages, if any, returned by APNs
- No external services or libraries (err, except OpenSSL)
- All done with a single bash script
- All done transparently with respect to your data

## Installation

- Clone the repo
- [Install OpenSSL](#install-openssl) if you haven't already fone so
- Get a [device token](#getting-a-device-token) 
- Obtain your [encryption key](#obtain-your-key)
- Enter your [own data in the script](#what-data-you-need-to-supply-to-the-script)
- Run the script (./pu.sh)
- You can optionally pass one of the ready made .json files found here as a parameter to pu.sh

## Install OpenSSL

You can skip this if you have already installed OpenSSL.
On a mac:

- Open a terminal
- Install homebrew: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
- Type `brew install openssl`

## Getting a device token

Below are the methods of your apps application delegate, with some short explanation comments, you need to obtain a device token.

```swift
/*
 * Here you register for remote notifications
 */
func application(_ application: UIApplication, 
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    
    application.registerForRemoteNotifications()
    return true
}

/*
 * Here you print out your device token
 */
func application(_ application: UIApplication, 
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    
    let token = deviceToken.hexEncodedString()
    
    // now the token will appear in the xcode console
    print(token)
}

/*
 * This helper method converts the device token frm Data to String
 * so you could just paste it into the script below
 */
func hexEncodedString() -> String {
    return map { String(format: "%02hhx", $0) }.joined()
}
```

## Obtain your key

You need an APNs authentication token signing key to generate the tokens used by your server in order to send push notifications. You request this key from your developer account on developer.apple.com in the Keys ➙ All section as shown below.

<p align="center">
  <img src="https://user-images.githubusercontent.com/1259736/52068513-77a9b700-2585-11e9-93b3-1737b7f72b3f.png" width="75%">
</p>
  
After continuing to the next step you get:

- A 10-character string with the key ID. Keep this on hand you will need it. If you forget it it is still available in the developer portal.
- A signing key as a .p8 text file. Keep this somewhere safe. For instance, don't keep it in your source code repository. If you lose it it's gone forever and you will have to revoke it and regenerate it. If this key is compromised in any way it can be used to send push notifications to your apps 🙀so if you suspect this has happened, revoke it and request a new one.

## What data you need to supply to the script

After you clone the script you need to supply your own info within the script:

- TEAMID: Found in the developer portal in your account
- KEYID: Found in the developer portal in your account
- SECRET: Full or relative path of the `.p8` you kept in a safe place
- BUNDLEID: The bundle id of your app
- DEVICETOKEN: A device token you have obtained

## Usage

After you set up data, you can just execute script to send test push notification:

```bash
chmod +x pu.sh
./pu.sh
```

## Troubleshooting

Unfortunately, the feedback returned from APNS when something goes wrong leaves a lot to be desired. These are the status codes that are possible to be returned and are shown in the output of the previous script.

```
200: Success
400: Bad request
403: There was an error with the certificate or with the provider authentication token
405: The request used a bad :method value. Only POST requests are supported.
410: The device token is no longer active for the topic.
413: The notification payload was too large.
429: The server received too many requests for the same device token.
500: Internal server error
503: The server is shutting down and unavailable.
```

If the status code is 200 all is well and you received the push notification.
In general, you should always use a recent device token, the device token should actually be from the device you are trying to send to and you should make sure that team IDs, key IDs, and bundle IDs are spelled correctly.
For a successful request, the body of the response is empty. On failure, the response body contains a JSON dictionary with a reason key that gives you a general idea of what went wrong e.g. BadDeviceToken.

_Note_: Development vs Production

Replace the ENDPOINT text in the script with the corresponding URL in the commented out section at the top of the script. Use development when running your app directly from Xcode onto a device, and production in every other case which is for Testflight, Adhoc, Enterprise or for the Appstore.

## License

This project is licensed under the MIT License - see the [LICENSE.md](https://gist.github.com/tsif/e233ee1941045151e7632bb3aacd3420) file for details

## Author

- [Dimitri James Tsiflitzis](https://dimitrijam.es)

