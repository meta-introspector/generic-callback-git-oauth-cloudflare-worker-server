# Generic Callback Git Authenticator

This simple [Cloudflare Workers](https://workers.cloudflare.com/) script allows generic plugin users to authenticate with [GitHub](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps) or [GitLab](https://docs.gitlab.com/ee/api/oauth2.html#authorization-code-flow).

## How to use it

### Step 1. Create a token in github with permissions

https://github.com/settings/personal-access-tokens/

```
This token has access to all repositories owned by the organization.
Organization permissions
 Read and Write access to organization actions variables, organization hooks, and organization secrets
 
Repository permissions
 Read access to metadata
 Read and Write access to actions, actions variables, administration, secrets, and workflows
  

  
 ```

Save it to ~.github file or somewhere

### Step 2. import token into gh cli

`gh auth login --with-token < ~/.github`


### Step 3. create a cf token
In cloudflare, [Create a token](https://dash.cloudflare.com/profile/api-tokens) 

### Step 4. add token to github via .env file

in .env add 
```
CF_API_TOKEN=XXXX
CF_ACCOUNT_ID=YYY
```

### Step 5. import token into gh cli
Set
`gh secret set -f .env  -R meta-introspector/generic-callback-git-oauth-cloudflare-worker-server`


### Step 6. Deploy this project to Cloudflare Workers

Sign up with Cloudflare, and click the button below to start deploying.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/meta-introspector/generic-callback-git-oauth-cloudflare-worker-server)

Alternatively, you can clone the project and run [`wrangler deploy`](https://developers.cloudflare.com/workers/wrangler/commands/#deploy) locally.

Once deployed, open your Cloudflare Workers dashboard, select the `generic-callback-git-oauth-cloudflare-worker-server` service, then the worker URL (`https://generic-callback-git-oauth.<SUBDOMAIN>.workers.dev`) will be displayed. Copy it for Step 2. It will also be used in Step 4.

### Step 7. Register the Worker as an OAuth app

#### GitHub

[Register a new OAuth application](https://github.com/settings/applications/new) on GitHub ([details](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app)) with the following properties, including your Worker URL from Step 1:

- Application name: `Generic Callback Git Authenticator` (or whatever)
- Homepage URL: `https://github.com/meta-introspector/generic-callback-git-oauth-cloudflare-worker-server` (or whatever)
- Application description: (can be left empty)
- Authorization callback URL: `https://generic-callback-git-oauth.jmikedupont2.workers.dev/callback`

Once registered, click on the **Generate a new client secret** button. The app’s **Client ID** and **Client Secret** will be displayed. We’ll use them in Step 3 below.

#### GitLab

[Register a new OAuth application](https://gitlab.com/-/user_settings/applications) on GitLab ([details](https://docs.gitlab.com/ee/integration/oauth_provider.html#create-a-user-owned-application)) with the following properties, including your Worker URL from Step 1:

- Name: `Generic Callback Git Authenticator` (or whatever)
- Redirect URI: `https://generic-callback-git-oauth.jmikedupont2.workers.dev/callback`
- Confidential: Yes
- Scopes: `api` only

Once registered, the app’s **Application ID** and **Secret** will be displayed. We’ll use them in Step 3 below.

### Step 3. Configure the Worker

Go back to the `generic-callback-git-oauth` service page on the Cloudflare dashboard, select **Settings** > **Variables**, and add the following Environment Variables to your worker ([details](https://developers.cloudflare.com/workers/platform/environment-variables/#environment-variables-via-the-dashboard)):

#### GitHub

- `GITHUB_CLIENT_ID`: **Client ID** from Step 2
- `GITHUB_CLIENT_SECRET`: **Client Secret** from Step 2; click the **Encrypt** button to hide it
- `GITHUB_HOSTNAME`: Required only if you’re using GitHub Enterprise Server. Default: `github.com`

#### GitLab

- `GITLAB_CLIENT_ID`: **Application ID** from Step 2
- `GITLAB_CLIENT_SECRET`: **Secret** from Step 2; click the **Encrypt** button to hide it
- `GITLAB_HOSTNAME`: Required only if you’re using a self-hosted instance. Default: `gitlab.com`

#### Both GitHub and GitLab

- `ALLOWED_DOMAINS`: (Optional) Your site’s hostname, e.g. `www.example.com`
  - Multiple hostnames can be defined as a comma-separated list, e.g. `www.example.com, www.example.org`
  - A wildcard (`*`) can be used to match any subdomain, e.g. `*.example.com` that will match `www.example.com`, `blog.example.com`, `docs.api.example.com`, etc. (but not `example.com`)
  - To match a `www`-less naked domain and all the subdomains, use `example.com, *.example.com`

Save and deploy.

### Step 4. Test github

Following these instructions
https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authenticating-to-the-rest-api-with-an-oauth-app

With the client id of the new github app as `Ov23liR8XH7uebRfEQGU`,

We can now authorize this app:
https://github.com/login/oauth/authorize?scope=user:email&client_id=Ov23liR8XH7uebRfEQGU

in the worker page we can see the log event show up 
https://dash.cloudflare.com/$ID/workers/services/view/generic-callback-git-oauth/production/logs/live

```
{
  "truncated": false,
  "outcome": "ok",
  "scriptVersion": {
    "id": "76112ae5-dd56-4b87-..."
  },
  "scriptName": "generic-callback-git-oauth",
  "diagnosticsChannelEvents": [],
  "exceptions": [],
  "logs": [],
  "eventTimestamp": 1725870049357,
  "event": {
    "request": {
      "url": "https://generic-callback-git-oauth.jmikedupont2.workers.dev/callback?code=7c0d1f7e3eb07f5088b4",
      "method": "GET",
      "headers": {
        "accept-encoding": "gzip, br",
        "cf-connecting-ip": "66.249",
        "cf-ipcountry": "US",
        "cf-ray": "8c05c2e07ee98274",
        "cf-visitor": "{\"scheme\":\"https\"}",
        "connection": "Keep-Alive",
        "host": "generic-callback-git-oauth.jmikedupont2.workers.dev",
        "user-agent": "Mozilla/5.0 (Linux; Android 7.0; SM-G930V Build/NRD90M) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.125 Mobile Safari/537.36 (compatible; Google-Read-Aloud; +https://support.google.com/webmasters/answer/1061943)",
        "x-forwarded-proto": "https",
        "x-real-ip": "66.249...."
      },
      "cf": {
        "clientTcpRtt": 8,
        "longitude": "-97.82200",
        "httpProtocol": "HTTP/1.1",
        "tlsCipher": "AEAD-AES128-GCM-SHA256",
        "continent": "NA",
        "asn": 15169,
        "clientAcceptEncoding": "gzip, deflate, br",
        "country": "US",
        "tlsClientAuth": {
          "certIssuerDNLegacy": "",
          "certIssuerSKI": "",....
        },
        "tlsExportedAuthenticator": {
          "clientFinished": "522bd09b9e3e8effa8f3f",
          "clientHandshake": "c75a338dda375176fd63",
          "serverHandshake": "201d4213225f3fdc9db0",
          "serverFinished": "bd1c35aa6270441e102d1"
        },
        "tlsVersion": "TLSv1.3",
        "colo": "IAD",
        "timezone": "America/Chicago",
        "verifiedBotCategory": "Accessibility",
        "edgeRequestKeepAliveStatus": 1,
        "tlsClientRandom": "Beyb76DRy58tvVeL",
        "tlsClientExtensionsSha1": "BizQtC8a5",
        "tlsClientHelloLength": "508",
        "asOrganization": "Google Proxy",
        "requestPriority": "",
        "latitude": "37.75100"
      }
    },
    "response": {
      "status": 200
    }
  },
  "id": 4
}```


### Step 4. Test gitlab 

Now we plugin in the application id, the target callback and the scope of the access into this and let the user login

https://gitlab.com/oauth/authorize?client_id=3c45c9d073b105b999744708e40756f1d19c4b23950fbc6ededf715cc06c8a33&redirect_uri=https://generic-callback-git-oauth.jmikedupont2.workers.dev/callback&response_type=code&state=STATE&scope=api&code_challenge=CODE_CHALLENGE&code_challenge_method=S256

```
{
  "truncated": false,
  "outcome": "ok",
  "scriptVersion": {
    "id": "76112ae5-dd56-4b87-b7d9-43eb0f94bd70"
  },
  "scriptName": "generic-callback-git-oauth",
  "diagnosticsChannelEvents": [],
  "exceptions": [],
  "logs": [],
  "eventTimestamp": 1725870880589,
  "event": {
    "request": {
      "url": "https://generic-callback-git-oauth.jmikedupont2.workers.dev/callback?code=REDACTED&state=STATE",
      "method": "GET",
      "headers": {
        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
        "accept-encoding": "gzip, br",
        "accept-language": "en-US",
        "cf-connecting-ip": "66.249",
        "cf-ipcountry": "US",
        "cf-ray": "8c05d72bab9f173f",
        "cf-visitor": "{\"scheme\":\"https\"}",
        "connection": "Keep-Alive",
        "host": "generic-callback-git-oauth.jmikedupont2.workers.dev",
        "upgrade-insecure-requests": "1",
        "user-agent": "Mozilla/5.0 (Linux; Android 7.0; SM-G930V Build/NRD90M) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.125 Mobile Safari/537.36 (compatible; Google-Read-Aloud; +https://support.google.com/webmasters/answer/1061943)",
        "x-forwarded-proto": "https",
        "x-real-ip": "66.249"
      },
      "cf": {
        "clientTcpRtt": 8,
        "longitude": "-97.82200",
        "httpProtocol": "HTTP/1.1",
        "tlsCipher": "AEAD-AES128-GCM-SHA256",
        "continent": "NA",
        "asn": 15169,
        "clientAcceptEncoding": "gzip, deflate, br",
        "country": "US",
        "tlsClientAuth": {
          "certIssuerDNLegacy": "",
          "certIssuerSKI": "",
          "certSubjectDNRFC2253": "",
          "certSubjectDNLegacy": "",
          "certFingerprintSHA256": "",
          "certNotBefore": "",
          "certSKI": "",
          "certSerial": "",
          "certIssuerDN": "",
          "certVerified": "NONE",
          "certNotAfter": "",
          "certSubjectDN": "",
          "certPresented": "0",
          "certRevoked": "0",
          "certIssuerSerial": "",
          "certIssuerDNRFC2253": "",
          "certFingerprintSHA1": ""
        },
        "tlsExportedAuthenticator": {
          "clientFinished": "0fff7a0ba3673a50915",
          "clientHandshake": "5fab4d262b48e2a67a",
          "serverHandshake": "84aa8220b27e372703",
          "serverFinished": "eea8bf29f07ea836648"
        },
        "tlsVersion": "TLSv1.3",
        "colo": "IAD",
        "timezone": "America/Chicago",
        "verifiedBotCategory": "Accessibility",
        "edgeRequestKeepAliveStatus": 1,
        "tlsClientRandom": "AI9y1n1Z6GnDxDW9T3",
        "tlsClientExtensionsSha1": "xvnX7c3fTl",
        "tlsClientHelloLength": "508",
        "asOrganization": "Google Proxy",
        "requestPriority": "",
        "latitude": "37.75100"
      }
    },
    "response": {
      "status": 200
    }
  },
  "id": 13
}
```


## Plan for callbacks

We will allow for callbacks to call custom code in webworkers, the plan is to run o1js zkapps from typescript compiled to wasm to run mina wallet operations as part of the callback code.

## FAQ


## Acknowledgements

This project was inspired by [`sveltia/sveltia-cms-auth`](https://github.com/sveltia/sveltia-cms-auth)
which was inspired by [`netlify-cms-oauth-firebase`](https://github.com/Herohtar/netlify-cms-oauth-firebase).
