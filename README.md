# Generic Callback Git Authenticator

This simple [Cloudflare Workers](https://workers.cloudflare.com/) script allows generic plugin users to authenticate with [GitHub](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps) or [GitLab](https://docs.gitlab.com/ee/api/oauth2.html#authorization-code-flow).

## How to use it

### Step 1. Deploy this project to Cloudflare Workers

Sign up with Cloudflare, and click the button below to start deploying.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/meta-introspector/generic-callback-git-oauth-cloudflare-worker-server)

Alternatively, you can clone the project and run [`wrangler deploy`](https://developers.cloudflare.com/workers/wrangler/commands/#deploy) locally.

Once deployed, open your Cloudflare Workers dashboard, select the `generic-callback-git-oauth-cloudflare-worker-server` service, then the worker URL (`https://generic-callback-git-oauth.<SUBDOMAIN>.workers.dev`) will be displayed. Copy it for Step 2. It will also be used in Step 4.

### Step 2. Register the Worker as an OAuth app

#### GitHub

[Register a new OAuth application](https://github.com/settings/applications/new) on GitHub ([details](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app)) with the following properties, including your Worker URL from Step 1:

- Application name: `Generic Callback Git Authenticator` (or whatever)
- Homepage URL: `https://github.com/sveltia/sveltia-cms-auth` (or whatever)
- Application description: (can be left empty)
- Authorization callback URL: `<YOUR_WORKER_URL>/callback`

Once registered, click on the **Generate a new client secret** button. The app’s **Client ID** and **Client Secret** will be displayed. We’ll use them in Step 3 below.

#### GitLab

[Register a new OAuth application](https://gitlab.com/-/user_settings/applications) on GitLab ([details](https://docs.gitlab.com/ee/integration/oauth_provider.html#create-a-user-owned-application)) with the following properties, including your Worker URL from Step 1:

- Name: `Generic Callback Git Authenticator` (or whatever)
- Redirect URI: `<YOUR_WORKER_URL>/callback`
- Confidential: Yes
- Scopes: `api` only

Once registered, the app’s **Application ID** and **Secret** will be displayed. We’ll use them in Step 3 below.

### Step 3. Configure the Worker

Go back to the `sveltia-cms-auth` service page on the Cloudflare dashboard, select **Settings** > **Variables**, and add the following Environment Variables to your worker ([details](https://developers.cloudflare.com/workers/platform/environment-variables/#environment-variables-via-the-dashboard)):

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

### Step 4. Update your CMS configuration

Open `admin/config.yml` locally or remotely, and add your Worker URL from Step 1 as the new `base_url` property under `backend`:

```diff
 backend:
   name: github # or gitlab
   repo: username/repo
   branch: main
+  base_url: <YOUR_WORKER_URL>
```

Commit the change. Once deployed, you can sign into Sveltia CMS remotely with GitHub or GitLab!

## FAQ


## Acknowledgements

This project was inspired by [`sveltia/sveltia-cms-auth`](https://github.com/sveltia/sveltia-cms-auth)
which was inspired by [`netlify-cms-oauth-firebase`](https://github.com/Herohtar/netlify-cms-oauth-firebase).
