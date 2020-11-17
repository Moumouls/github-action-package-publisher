# Github Action Package Publisher

**Release and update automatically your package on master push**


## Requirements on your package.json

You need to have [standard-version](https://www.npmjs.com/package/standard-version) package installed.

```
npm i standard-version -D
```
OR

```
yarn add standard-version -D
```

## How it works

The magic happen in `.github/workflows/release.yml`

You can copy past the `release.yml` into your Github project.

Note: the release branch should exist

```yaml
# Name of the Action
name: Release

# Only run the release system on master
on:
  push:
    branches:
      - master

jobs:
  release:
    # Adjust time (it's to avoid infinite CI)
    timeout-minutes: 5
    runs-on: ubuntu-latest
	 
	 # Choose node version
    strategy:
      matrix:
        node-version: [15.x]
        
    steps:
      - uses: actions/checkout@v2
        with:
          # Needed to avoid unrelated history merge error
          fetch-depth: 0

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
		
		# Checkout the release branch
      - name: Configure
        run: git config pull.rebase true && git config --global user.email "actions@github.com" && git config --global user.name "Github Actions"

      - name: Merge master into release
        run: git checkout --track origin/release && git pull && git merge master

		# Adjust the output folder of your app
      - name: Remove lib from git ignore
        run: sed -i.bak '/lib/d' .gitignore && rm .gitignore.bak

      - name: Install
        run: yarn # or npm i
        env:
          NODE_ENV: development

      - name: Build
        run: yarn build # or npm run build
		
		# If new addition detected it will create a new git tag
		# and update the package version into package.json
      - name: Version
        run: yarn standard-version # or npm run standard-version
		
		# If changes detected push the new release and the new tag
		# then your custom package can be used like:
		# "your-package": "githubUsername/repoName#v1.0.0"
      - name: Release
        run: git add -A && git commit -m "Release" && git push --follow-tags

```