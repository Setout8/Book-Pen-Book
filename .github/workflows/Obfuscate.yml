name: Build Obfuscate BPB Panel

on:
  push:
    branches:
      - main
  schedule:
    - cron: "0 1 * * *"

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out the code
        uses: actions/checkout@v4

      - name: Cache npm
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-npm-

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "latest"

      - name: Install dependencies
        run: |
          for i in {1..3}; do npm install -g javascript-obfuscator && break || sleep 5; done
          javascript-obfuscator --version
          sudo apt-get update
          sudo apt-get install -y jq

      - name: Fetch and download latest worker.js from release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          RELEASE_INFO=$(curl -s -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/repos/bia-pain-bache/BPB-Worker-Panel/releases/latest)
          RELEASE_TAG=$(echo "$RELEASE_INFO" | jq -r '.tag_name')
          echo "Latest Release: $RELEASE_TAG"
          DOWNLOAD_URL=$(echo "$RELEASE_INFO" | jq -r '.assets[] | select(.name == "worker.js") | .browser_download_url')
          if [ -z "$DOWNLOAD_URL" ]; then
            echo "Error: Could not find worker.js in latest release"
            exit 1
          fi
          echo "Downloading from: $DOWNLOAD_URL"
          wget -O origin.js "$DOWNLOAD_URL"
          ls -l origin.js
          head -n 10 origin.js

      - name: Obfuscate BPB worker js
        run: |
          javascript-obfuscator origin.js --output _worker.js \
            --compact true \
            --identifier-names-generator hexadecimal \
            --string-array true \
            --string-array-threshold 0.5 \
            --simplify true
          ls -l _worker.js
          head -n 10 _worker.js
          # Check file size (in bytes)
          FILE_SIZE=$(stat -f %z _worker.js || stat -c %s _worker.js)
          echo "Output file size: $FILE_SIZE bytes"
          if [ "$FILE_SIZE" -gt 500000 ]; then
            echo "Warning: _worker.js size exceeds 500KB, may cause Cloudflare CPU limit errors"
          fi

      - name: Commit changes
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          branch: main
          commit_message: ':arrow_up: update latest bpb panel'
          commit_author: 'github-actions[bot] <github-actions[bot]@users.noreply.github.com>'
