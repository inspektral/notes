# GitHub Pages Setup Guide

## Problem
If you're seeing only the README file on your GitHub Pages site instead of the full MkDocs documentation, it's because GitHub Pages is not configured to use GitHub Actions as the deployment source.

## Solution

### Step 1: Update the Workflow (Already Done ✓)
The workflow has been updated to use the modern GitHub Actions Pages deployment method.

### Step 2: Configure GitHub Pages Source (Required)
You need to configure your repository to use GitHub Actions for Pages deployment:

1. Go to your repository on GitHub: https://github.com/inspektral/notes
2. Click on **Settings** (in the top navigation bar)
3. In the left sidebar, click on **Pages** (under the "Code and automation" section)
4. Under **Build and deployment**, you should see a **Source** dropdown
5. Select **GitHub Actions** from the dropdown

![GitHub Pages Settings](https://docs.github.com/assets/cb-28688/mw-1440/images/help/pages/publishing-source-drop-down.webp)

### Step 3: Trigger a Deployment
Once you've changed the source to GitHub Actions:

1. The workflow will automatically run on the next push to the `main` branch
2. Or you can manually trigger it by going to **Actions** tab → **Deploy Wiki to GitHub Pages** → **Run workflow**

### Step 4: Verify
After the workflow completes:

1. Go to the **Actions** tab to verify the deployment succeeded
2. Visit your GitHub Pages URL: https://inspektral.github.io/notes/
3. You should now see your full MkDocs documentation instead of just the README

## Why This Change Was Needed

### Old Method (mkdocs gh-deploy)
- Pushed built site to a separate `gh-pages` branch
- Required manual configuration to serve from that branch
- Less integration with GitHub's deployment features

### New Method (GitHub Actions)
- Uses native GitHub Pages deployment actions
- Automatically configures Pages to use Actions as source
- Better deployment tracking and rollback capabilities
- Includes deployment URLs and environment information
- No need for a separate `gh-pages` branch

## Troubleshooting

### Still seeing the README?
- Make sure you've changed the Pages source to "GitHub Actions"
- Check that the workflow has run successfully in the Actions tab
- Try a hard refresh of your browser (Ctrl+Shift+R or Cmd+Shift+R)
- Clear your browser cache

### Workflow failing?
- Check the Actions tab for error messages
- Ensure all dependencies in `requirements.txt` are correct
- Verify that all markdown files in the `docs/` directory are valid

## Additional Resources
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [MkDocs Documentation](https://www.mkdocs.org/)
- [GitHub Actions for Pages](https://github.com/actions/deploy-pages)
