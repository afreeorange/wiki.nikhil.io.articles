That would be [this script](https://github.com/GAM-team/got-your-back). I installed on my Mac (10.13) via

```bash
brew install gyb
```

All config will be kept in `/opt/homebrew/etc/gyb`

I then ran this:

```bash
gyb --action create-project --email my-email@gmail.com
```

This will create a new Project in Google Cloud (you will need to allow it to do so). You can delete/manage it via the [Resource Manager](https://console.cloud.google.com/cloud-resource-manager).

You will have to set up an OAuth API Client Key and Secret via the "[API and Services](https://console.cloud.google.com/apis/dashboard)" tab. This will require you to set up a _Consent Screen_. Make sure you do that (it will be External) via the wizard and then set up a Test User.

Once that's done, click "[Credentials](https://console.cloud.google.com/apis/credentials)" (making sure that you're on the right project in the top dropdown!) and create credentials by clicking "Create Credentials" &rarr; "OAuth Client ID".

Give the creds to the GYB app in your terminal. Now run this:

```bash
gyb --email my-email@gmail.com \
    --action backup \
    --local-folder "/path/to/folder"
```

You will then need to give GYB full backup and restore permissions else you'll see this error: [403: Insufficient Permission - insufficientPermissions](https://github.com/GAM-team/got-your-back/issues/112). If you see that, you should delete the `.cfg` file in `/opt/homebrew/etc/gyb/my-email\@gmail.com.cfg` and re-run that command, making sure you have given GYB backup and restore access.
