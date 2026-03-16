# Akash-Vault — Setup Guide

Follow these 2 steps to get your site live. After this, you'll never need to touch code again.

The repo is already pushed to: `https://github.com/akashgupta200/Akash-Vault`

---

## Step 1: Create a GitHub OAuth App (5 minutes)

This lets you login to the admin panel on your website to edit content.

1. Go to [github.com/settings/developers](https://github.com/settings/developers)
2. Click **OAuth Apps** tab
3. Click **New OAuth App**
4. Fill in:
   - **Application name**: `Akash Vault CMS`
   - **Homepage URL**: `https://akashgupta200.github.io/Akash-Vault/`
   - **Authorization callback URL**: `https://akashgupta200.github.io/Akash-Vault/admin/`
5. Click **Register application**
6. You'll see a **Client ID** (something like `Ov23lixyz123abc456`)
7. Copy that Client ID

Now update ONE file in your repo:

### Option A: Update via GitHub website (easiest)
1. Go to your repo: [github.com/akashgupta200/Akash-Vault](https://github.com/akashgupta200/Akash-Vault)
2. Navigate to `admin/config.yml`
3. Click the pencil icon (Edit)
4. Replace `YOUR_GITHUB_OAUTH_CLIENT_ID` with the Client ID you copied
5. Click **Commit changes**

### Option B: Update locally
Open `admin/config.yml` and change line 6:

```yaml
  app_id: YOUR_GITHUB_OAUTH_CLIENT_ID      # <-- paste your Client ID here
```

Then push:
```bash
git add admin/config.yml
git commit -m "Configure CMS with GitHub OAuth"
git push
```

---

## Step 2: Enable GitHub Pages (1 minute)

1. Go to [github.com/akashgupta200/Akash-Vault/settings/pages](https://github.com/akashgupta200/Akash-Vault/settings/pages)
2. Under **Source**, select: **GitHub Actions**
3. That's it! The GitHub Actions workflow will build and deploy your site.

Wait 1-2 minutes for the first build. Check progress under the **Actions** tab.

---

## You're Done!

Your site is now live at:

**https://akashgupta200.github.io/Akash-Vault/**

### To read your reference:
Just visit the URL above. Use the sidebar to navigate.

### To edit content:
1. Go to `https://akashgupta200.github.io/Akash-Vault/admin/`
2. Click **Login with GitHub**
3. Pick a page to edit
4. Use the rich text editor
5. Click **Publish**
6. Wait ~60 seconds for the site to rebuild

### To add a new DBA Reference Sheet page:
1. Go to `/admin/`
2. Click **DBA Reference Sheet** section
3. Click **New DBA Reference Sheet**
4. Fill in title and content
5. Click **Publish**

---

## Troubleshooting

### "Page not found" after first deploy
- Wait 2-3 minutes for GitHub Actions to complete
- Check the **Actions** tab on your repo for build status

### "Login failed" on admin panel
- Make sure the OAuth App callback URL matches exactly: `https://akashgupta200.github.io/Akash-Vault/admin/`
- Make sure the Client ID in `admin/config.yml` is correct
- The repo must be public

### Site looks broken
- Make sure `_config.yml` has `baseurl: "/Akash-Vault"`
- Check GitHub Actions build logs for errors
