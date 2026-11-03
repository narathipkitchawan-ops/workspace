# Recreating the GCP Project

This guide provides step-by-step instructions to recreate the Google Cloud
Platform (GCP) project and infrastructure required for the Google Workspace
Extension.

## Overview

The extension uses a hybrid OAuth flow:

1. **Local client:** Requests authorization from the user.
2. **Cloud Function:** Exchanges the authorization code for tokens. It reads the
   `CLIENT_SECRET` from Secret Manager.
3. **Secret Manager:** Stores the OAuth client secret.

## Prerequisites

- A Google Cloud project with billing enabled.
- [Google Cloud CLI (`gcloud`)](https://cloud.google.com/sdk/docs/install),
  installed and authenticated.
- Node.js 20 or newer and npm.
- The repository cloned locally, with its root directory as the working
  directory when running the commands below.

## Step 1: Install dependencies and run setup

The setup script uses the repository's `ts-node` dependency to derive OAuth
scopes from the feature configuration. Install the locked dependencies first:

```bash
npm ci
```

Set your Google Cloud project ID:

```bash
gcloud config set project YOUR_PROJECT_ID
```

Then run the guided setup script from the repository root:

```bash
bash scripts/setup-gcp.sh
```

The script will:

1. Enable the required GCP APIs.
2. Guide you through OAuth consent-screen configuration and show the scopes
   required by the enabled features.
3. Deploy the Cloud Function and display its URL.
4. Prompt you to create an OAuth 2.0 Client ID in Google Cloud Console, using
   the function URL as the authorized redirect URI.
5. Store the client secret in Secret Manager, configure the function, and grant
   it access to the secret.

Complete the consent-screen setup and add test users when the app is in Testing
mode. The script asks you to confirm before continuing.

## Step 2: Local configuration

After setup, set the environment variables in your shell (for example, in
`.zshrc` or `.bashrc`):

```bash
export WORKSPACE_CLIENT_ID="your-client-id"
export WORKSPACE_CLOUD_FUNCTION_URL="https://your-cloud-function-url"
```

The script prints the values to use. Alternatively, configure the defaults in
`workspace-server/src/utils/config.ts`.

## Why a Cloud Function?

The Cloud Function keeps the OAuth client secret in your GCP project rather
than embedding it in the local extension. The local client receives OAuth
 tokens but does not receive the client secret.
