# buildUpdatePushGhcr
GitHub composite action to build, update version and push docker images to ghcr.io

## Assumptions ##
This composite action assumes the following:
1. That you have a scoped NPM packages stored in GitHub Packages
2. That you store your docker images in ghcr.io
3. That you're an organization - not a private user - on ghcr.io
4. That you wish to delete 1-week-old images, as long as there are more than 10 images
5. That your workflow consists of the followings:
    1. Checkout repository
    2. Perform static code analysis with Snyk (Optional)
    3. Install Node ver. 16
    4. Install NPM dependencies from GitHub Packages and NPM
    5. Update the "patch" section of the version and push the update to GitHub
    6. Build the system
    7. Run unit tests (optional)
    8. Remove dev dependencies
    9. Build and push docker image to ghcr.io, tagged by the version number
    10. Delete old docker images from ghcr.io

## Usage ##
```yaml
name: Build and push docker image
on:
  push:
    branches:
      - 'main' 
jobs:
  buildAndPushDocker:
    runs-on: ubuntu-latest
    outputs:
      tagversion: ${{ steps.buildAndPush.outputs.tagversion }}
      imagename: ${{ github.event.repository.name }}
    permissions:
      contents: write
      packages: write
    steps:
      - name: Build and push docker image
        id: buildAndPush
        uses: pangea-it/buildUpdatePushGhcr@main
        with:
          token: ${{ secrets.PAT }}
          snyk_token: ${{ secrets.SNYK_TOKEN }}
          npm_registry_scope: '@example-org'
          ghcr_tag_owner: example-org
```

Example without Snyk/unit tests:
```yaml
name: Build and push docker image
on:
  push:
    branches:
      - 'main' 
jobs:
  buildAndPushDocker:
    runs-on: ubuntu-latest
    outputs:
      tagversion: ${{ steps.buildAndPush.outputs.tagversion }}
      imagename: ${{ github.event.repository.name }}
    permissions:
      contents: write
      packages: write
    steps:
      - name: Build and push docker image
        id: buildAndPush
        uses: pangea-it/buildUpdatePushGhcr@main
        with:
          token: ${{ secrets.PAT }}
          use_snyk: false
          perform_tests: false
          npm_registry_scope: '@example-org'
          ghcr_tag_owner: example-org
```
    
## Parameters ##
Name                  | Required                                      
-------------         | -------------                                
token                 | Yes. Defaults to `GITHUB_TOKEN`. Used for the NPM registry as well. 
checkout_branch       | No. Defaults to default branch. Enables specifying the branch to checkout.
use_snyk              | Yes. Defaults to `true`. Set to `false` to ignore Snyk.
perform_tests         | Yes. Defaults to `true`. Set to `false` to ignore unit tests.
snyk_token            | No. Required only if `use_snyk` is set to `true`.                                          
npm_registry          | Yes. Defaults to `https://npm.pkg.github.com/` 
npm_registry_scope    | Yes. Example: `'@example-org'`                                        
docker_login_registry | Yes. Defaults to `ghcr.io`                     
docker_login_user     | Yes. Defaults to `github.actor`                
ghcr_tag_owner        | Yes. Defaults to `github.repository_owner`     
docker_image_name     | Yes. Defaults to `github.event.repository.name`

## Outputs ##
Name                  | Description                                      
-------------         | -------------                                
tagversion            | Updated version number/docker tag number                
imagename             | Repository name

## Permissions ##
- contents: write
- packages: write
