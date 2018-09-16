# Youtube API V3 Demo Vuejs Wrapper

This is just a simple implementation of the Google 
Youtube Data API v3 [subscriptions list](https://developers.google.com/youtube/v3/docs/subscriptions/list#request) javascript sample.

It is wrapped by Vuejs app and styled by Bulma 
Framework.



## Project setup
#### Step One: Create an Google API App and Credentials 
 1.  Use [this wizard](https://console.developers.google.com/start/api?id=youtube) to create or select a project in the Google Developers Console and automatically turn on the API. Click **Continue**, then **Go to credentials**.
2.  On the **Add credentials to your project** page, click the **Cancel** button.
3.  At the top of the page, select the **OAuth consent screen** tab. Select an **Email address**, enter a **Product name** if not already set, and click the **Save** button.
4.  Select the **Credentials** tab, click the **Create credentials** button and select **OAuth client ID**.
5.  Select the application type **Web application**.
6.  In the **Authorized JavaScript origins** field, enter the URL `http://localhost:8080`. Same for **Authorized redirect URIs** field.
7.  Click the **Create** button.
8.  Click **OK** to dismiss the resulting dialog.
9.  Click **Download** button and download JSON file.
10. Save it as **client_secret.json** on your project's root directory.

#### Step Two: Build your project.
```
git clone https://github.com/Crx4/youtube-api-v3-vuejs-wrapper.git
cd youtube-api-v3-vuejs-wrapper
npm install
npm run serve
```
