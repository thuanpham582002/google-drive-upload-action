[![build](https://github.com/thuanpham582002/google-drive-upload-git-action/actions/workflows/build-and-push.yml/badge.svg?branch=main)](https://github.com/thuanpham582002/google-drive-upload-git-action/actions)
[![Go Report Card](https://goreportcard.com/badge/github.com/adityak74/google-drive-upload-git-action)](https://goreportcard.com/report/github.com/adityak74/google-drive-upload-git-action)
[![Docker Image](https://ghcr-badge.egpl.dev/thuanpham582002/google-drive-upload-git-action/latest_tag?trim=major&label=Docker%20Image)](https://github.com/thuanpham582002/google-drive-upload-git-action/pkgs/container/google-drive-upload-git-action)

# google-drive-upload-git-action
Github action that uploads files to Google Drive.

![star-history-2025323 (1)](https://github.com/user-attachments/assets/5f54c80b-96cd-4314-a8b5-3f1ac5504fd4)

**This only works with a Google Service Account!**

## 🐳 Docker Image
This action now runs using a pre-built Docker image hosted on GitHub Container Registry:
- **Image**: `ghcr.io/thuanpham582002/google-drive-upload-git-action:latest`
- **Package URL**: https://github.com/thuanpham582002/google-drive-upload-git-action/pkgs/container/google-drive-upload-git-action

Thanks to [Team Tumbleweed](https://github.com/team-tumbleweed) for developing the initial version of this actions package.

Forked from [adityak74/google-drive-upload-git-action](https://github.com/adityak74/google-drive-upload-git-action) with Docker image optimization.

## 🔧 Setup

To make a GSA go to the [Credentials Dashboard](https://console.cloud.google.com/apis/credentials). You will need to download the **.json key** and base64 encode it. You will use this string as the `credentials` input. To convert the *json* file to base64 without having to use an online tool (which is insecure), use this command:

`base64 credentials.json -w0`

On mac this the base64 by default opts for -w as 0, you can skip and just use base64 without any params.

You will also need to **share the drive with the servie account.** To do this, just share the folder like you would normally with a friend, except you share it with the service account email address. Additionally you will need to give the service account acccess to the google drive API. 
Go to `https://console.developers.google.com/apis/api/drive.googleapis.com/overview?project={PROJECT_ID}`. Where `{PROJECT_ID}` is the id of your GCP project. Find more info about that [here.](https://support.google.com/googleapi/answer/7014113?hl=en)

# Inputs

## ``filename``
Required: **YES**.  

The name of the file you want to upload. Wildcards can be used to upload more than one file.

## ``name``
Required: **NO**

The name you want the file to have in Google Drive. If this input is not provided, it will use only the filename of the source path. It will be ignored if there are more than one file to be uploaded.

## ``overwrite``
Required: **NO**

If you want to overwrite the filename with existing file, it will use the target filename.
## ``mimeType``
Required: **NO**

file MimeType. If absent, Google Drive will attempt to automatically detect an appropriate value.

## ``useCompleteSourceFilenameAsName``
Required: **NO**

If true, the target file name will be the complete source filename and `name` parameter will be ignored.

## ``mirrorDirectoryStructure``
Required: **NO**

If true, the directory structure of the source file will be recreated relative to ``folderId``.

## ``namePrefix``
Required: **NO**

Prefix to be added to target filename.

## ``folderId``
Required: **YES**. 

The [ID of the folder](https://ploi.io/documentation/database/where-do-i-get-google-drive-folder-id) you want to upload to.

## ``credentials``
Required: **YES**.

A base64 encoded string with the [GSA credentials](https://stackoverflow.com/questions/46287267/how-can-i-get-the-file-service-account-json-for-google-translate-api/46290808).

# Usage Examples

## 📋 Simple Workflow
In this example we stored the folderId and credentials as action secrets. This is highly recommended as leaking your credentials key will allow anyone to use your service account.

```yaml
# .github/workflows/main.yml
name: Main
on: [push]

jobs:
  my_job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Archive files
        run: |
          sudo apt-get update
          sudo apt-get install zip
          zip -r archive.zip *

      - name: Upload to Google Drive
        uses: thuanpham582002/google-drive-upload-git-action@main
        with:
          credentials: ${{ secrets.CREDENTIALS }}
          filename: "archive.zip"
          folderId: ${{ secrets.FOLDER_ID }}
          name: "documentation.zip" # optional string
          overwrite: "true" # optional boolean
```

## 📁 Mirror Directory Structure
```yaml
# .github/workflows/mirror.yml
name: Mirror Directory Structure
on: [push]

jobs:
  upload_with_structure:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Make Directory Structure
        run: |
          mkdir -p project/src/components
          echo "console.log('Hello World');" > project/src/components/App.js
          echo "# Project Documentation" > project/README.md

      - name: Upload with Directory Structure
        uses: thuanpham582002/google-drive-upload-git-action@main
        with:
          credentials: ${{ secrets.CREDENTIALS }}
          filename: "project/**/*"
          folderId: ${{ secrets.FOLDER_ID }}
          overwrite: "true"
          mirrorDirectoryStructure: "true"
```

## 🔄 Multiple Files Upload
```yaml
# .github/workflows/multiple-files.yml
name: Upload Multiple Files
on: [push]

jobs:
  upload_multiple:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Generate multiple files
        run: |
          echo "File 1 content" > file1.txt
          echo "File 2 content" > file2.txt
          echo "File 3 content" > file3.txt

      - name: Upload multiple files
        uses: thuanpham582002/google-drive-upload-git-action@main
        with:
          credentials: ${{ secrets.CREDENTIALS }}
          filename: "*.txt"
          folderId: ${{ secrets.FOLDER_ID }}
          namePrefix: "backup-"
          overwrite: "true"
```

## 🏷️ Using Specific Version
For production use, it's recommended to pin to a specific version instead of `@main`:

```yaml
- name: Upload to Google Drive
  uses: thuanpham582002/google-drive-upload-git-action@v1.0.0
  with:
    credentials: ${{ secrets.CREDENTIALS }}
    filename: "build/**/*"
    folderId: ${{ secrets.FOLDER_ID }}
```

## 🔐 Setting up Secrets
1. Go to your repository → Settings → Secrets and Variables → Actions
2. Add the following secrets:
   - `CREDENTIALS`: Your base64 encoded Google Service Account JSON
   - `FOLDER_ID`: The Google Drive folder ID where files will be uploaded

## 📦 Docker Usage
You can also use the Docker image directly:

```bash
docker pull ghcr.io/thuanpham582002/google-drive-upload-git-action:latest
```

## 🚀 Latest Updates
- ✅ Migrated to Docker image for faster execution
- ✅ Automated builds via GitHub Actions
- ✅ Support for directory structure mirroring
- ✅ Multiple file upload with wildcards
- ✅ Customizable file naming with prefixes
