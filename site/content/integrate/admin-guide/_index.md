---
title: "Development Guides"
heading: "Mattermost Integration Guides"
description: "Learn how to build Mattermost integrations via demos, example code, and guided content."
weight: 90
---

## Better Control for System Admins

The main benefits of the Mattermost Developer Toolkit from a system administration perspective are first, that it puts all of the resources and information that developers will need in one central location and second, it simplifies integrations with Mattermost.

Consequently this will save time for system administrators as well as developers.

## Features of the Developer Toolkit

Below is a list of planned features of the developer toolkit with estimated time of delivery.

1. Completed:

 - Webhooks and slash commands to allow easy, low-effort extension and integration.
 - Mattermost HTTP REST APIv4 allowing for much more powerful server interaction.
 - Mattermost webapp moved over to Redux infrustructure.
 - API developer token to provide a simple method to authenticate to the Mattermost REST API.
 - The ability to build webapp client plugins to override existing UI components (replace posts with your custom components, use your own video services etc.), modify/extend client drivers to interact with custom server API endpoints, and add whole new UI views in predetermined places.
 - The ability to build server plugins to hook directly into server events (e.g., new post events, user update events, etc.), have some form of database access (possibly access to certain tables, and the ability to create new tables) and to add custom endpoints to extend the Mattermost REST API.

2. Upcoming:

 - The ability to build plugins similar to the webapp but for React Native apps for iOS and Android.

All of the documentation required to support the use and building of the toolkit will be done as we work through the above systems.

### Example Uses

Examples of uses for the plugin architecture include:

1. Building common integrations such as Jira and GitHub, and including them as default integrations for Mattermost.
2. Providing tools to interact with user posts.
3. Redesigning the current video and audio calling to use the plugin architecture, and offering it as one of many video and audio calling solutions.
4. Incorporating other third-party applications such as annual performance reviews right from the Mattermost interface.

## What's the difference between incoming and outgoing webhooks?

A webhook is a way for one app to send real-time data to another app.

In Mattermost, incoming webhooks receive data from external applications and make a post in a specified channel. They're great for setting up notifications when something happens in an external application.

Outgoing webhooks take data from Mattermost, and send it to an external application. Then the outgoing webhook can post a response back in Mattermost. They're great for listening in on channels, and then notifying external applications when a trigger word is used.

## What is a slash command?

A slash command is similar to an outgoing webhook, but instead of listening to a channel it is used as a command tool. This means if you type in a slash command it will not be posted to a channel, whereas an outgoing webhook is only triggered by posted messages.

## What does Slack-compatible mean?

Slack compatible means that Mattermost accepts integrations that have a payload in the same format as Slack.  

If you have a Slack integration, you should be able to set it up in Mattermost without changing the format.   

## What if I have a webhook from somewhere other than Slack?

If you have an integration that outputs a payload in a different format you need to write an intermediate application to act as a translation layer to change it to the format Mattermost uses. Since there’s currently no general standard for webhook formatting, this is unavoidable and just a part of how webhooks work.

If there's no translation layer, Mattermost won't understand the data you're sending.

## What are attachments?

When "attachments" are mentioned in the integrations documentation, it refers to Slack's Message Attachments. These "attachments" can be optionally added as an array in the data sent by an integration, and are used to customize the formatting of the message.

We currently don't support the ability to attach files to a post made by an integration.

## Where can I find existing integrations?

[Visit our app directory](https://mattermost.com/marketplace/) for dozens of open source integrations to common tools like Jira, Jenkins, GitLab, Trac, Redmine, and Bitbucket, along with interactive bot applications (Hubot, mattermost-bot), and other communication tools (Email, IRC, XMPP, Threema) that are freely available for use and customization.

## Where should I install my integrations? 

For self-hosted deployments in small setups you might host integrations on the same server on which Mattermost is installed. For larger deployments you can setup a separate server for integrations, or add them to the server on which the external application is hosted - for example, if you're self-hosting a Jira server you could deploy a Jira integration on the Jira server itself.

When self-hosting restrictions are less strict, AWS, Heroku, and other public cloud options could also be used.

## How do I create a bot account with personal access tokens?

See [bot accounts documentation](../admin-bot-accounts) to learn more about how to create and manage bot accounts in Mattermost.

## How do I create a bot account without personal access tokens or webhooks?

Deployments that cannot create bot accounts via webhooks due to security reasons and do not want to use [personal access tokens](../admin-personal-access-tokens) with no expiry time, can use the following approach:

1. Create a bot account using a secure email and strong password.
2. Manually add the account to all teams and channels it needs access to. If your deployment has a lot of teams or channels, you may create a CLI script to automate the process.
   - In a testing environment, you may also make the bot account a System Admin, giving the bot permissions to post to any channel. Not recommended in production due to potential security vulnerabilities.
3. Provide the email and password to your integration, and store it in a secure location with restricted access.
4. Have your integration use the email and password with an [`/api/v4/login`](https://api.mattermost.com/v4/#tag/authentication) endpoint to retrieve a session token. The session token is used to authenticate to the Mattermost system.
   - Set up your bot to make an HTTP POST to `your-mattermost-url.com/api/v4/users/login` with a JSON body, including the bot account's email and password.
  
     ```
     POST /api/v4/users/login HTTP/1.1
     Host: your-mattermost-url.com
     Content-Length: 66
     Content-Type: application/json
     
     {"login_id":"someone@nowhere.com","password":"thisisabadpassword"}
     ```
  
     where we assume there is a Mattermost instance running at http://localhost:8065.
   - If successful, the response will contain a `Token` header and a user object in the body:
   
     ```
     HTTP/1.1 200 OK
     Set-Cookie: MMSID=hyr5dmb1mbb49c44qmx4whniso; Path=/; Max-Age=2592000; HttpOnly
     Token: hyr5dmb1mbb49c44qmx4whniso
     X-Ratelimit-Limit: 10
     X-Ratelimit-Remaining: 9
     X-Ratelimit-Reset: 1
     X-Request-Id: smda55ckcfy89b6tia58shk5fh
     X-Version-Id: developer
     Date: Fri, 11 Sep 2015 13:21:14 GMT
     Content-Length: 657
     Content-Type: application/json; charset=utf-8
     
     {{user object as json}}
     ```
     
    The bot should retrieve the session token from the `Token` header and store it in memory for use with future requests.
   
**Note:** Each session token has an expiry time, set depending on the server's configuration. If the session token your bot is using expires, it will receive a `401 Unauthorized` response from requests using that token. When your bot receives this response, it should reapply the login logic (using the above steps) to get another session token. Then resend the request that received the `401` status code.

5. Include the `Token` as part of the `Authorization` header on API requests from your integration.
   - To confirm the token works, you can have your bot make a simple `GET` request to `/api/v4/users/me` with the `Authorization: bearer <yourtokenhere>` in the header. If it returns a `200` with the bot's user object in the response, the API request was made successfully.
  
  ```
     GET /api/v4/users/me HTTP/1.1
     Authorization: bearer <yourtokenhere>
     Host: your-mattermost-url.com
  ```

**Note:** The Mattermost development team is also working on an [API developer token](https://docs.google.com/document/d/1ey4eNQmwK410pNTvlnmMWTa1fqtj8MV4d9XkCumI384), which allows you to authenticate the bot account via the API token rather than retrieving a session token from a user account.

## How should I automate the install and upgrade of Mattermost when included in another application?

Automating Mattermost installation within another application:

1. Review the [Mattermost installation documentation](https://docs.mattermost.com/guides/install-deploy-upgrade-scale.html#install-mattermost) to understand configuration steps of the production deployment.
2. Install Mattermost files to a dedicated `/opt/mattermost` directory by decompressing the `tar.gz` file of the latest release for your target platform (for example `linux-amd64`).
3. Review [Configuration Settings](https://docs.mattermost.com/configure/configuration-settings.html) in `config.json` and set your automation to customize your Mattermost deployment based on your requirements.
4. For directory locations defined in `config.json`, such as the location of the local file storage directory (`./data/`) or logs directory (`./logs`), you can redefine those locations in your `config.json` settings and move the directories.
   - All other directories should remain as they are in `/mattermost`.
5. Test that your Mattermost server is running with your new configuration.
6. Also, from the command line run `./bin/mattermost -version` to test that the command line interface is functioning properly.

Automating Mattermost upgrade within another application:

1. Review the [upgrade guide](https://docs.mattermost.com/upgrade/upgrading-mattermost-server.html) for an overview of the upgrade procedure.
2. Create automation to upgrade to the next Mattermost versions:
    - Back up the `config.json` file to preserve any settings a user may have made.
    - Back up the `./data` directory if local storage is used for files.
    - Replace the contents of `/mattermost` directory with the decompressed contents of the latest release.
    - Restore `config.json` and `./data` to their previous locations (which may have been overwritten).
    - If you need to overwrite any `config.json` parameters use a [`sed` command](https://stackoverflow.com/questions/20568515/how-to-use-sed-to-replace-a-config-files-variable) or similar tool to update `config.json`
    - Starting the Mattermost server to upgrade the database, `config.json` file, and `./data` as necessary.
3. Optionally the upgrade procedure can be chained so users can upgrade across an arbitrary number of Mattermost versions rather than to just the latest release.

Come [join our Contributors community channel](https://community.mattermost.com/core/channels/tickets) on our daily build server, where you can discuss questions with community members and the Mattermost core team. Join our [Developers channel](https://community.mattermost.com/core/channels/developers) for technical discussions and our [Integrations channel](https://community.mattermost.com/core/channels/integrations) for all integrations and plugins discussions.
