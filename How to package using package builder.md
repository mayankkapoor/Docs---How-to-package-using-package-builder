# How to package a GitHub source repository using the JioCloud Package Build System.

1. Open the Package Build system web interface by going to www.url-of-package-build-sysem.com
2. Click on the "Log in" button and authenticate via GitHub. Once you authenticate the app via Github, you should arrive at the Dashboard/Overview screen.
3. The first step is to add a source git repo for the package builder to build from. Click on "Sources" on the left side of the screen.
4. Click on "New" to add a new source to package from.
5. Add the git url of the source repo (e.g. https://github.com/akash1808/nova) and the branch (e.g. package-builder) you want to package from. Choose the destination apt repo where the built packages will be placed (e.g. mayank/overcast). Click Submit.  
The source repo will show up under your package sources.

## Build process and packages
Once a source git repo is added to "Sources", a build to create a package is automatically triggered. You'll see your running builds under the "Builds" link on the left. After the initial build, the package builder will poll the source git repo every 10 seconds to check for changes, and create new packages if something has changed.

1. You can check if a build has finished by going to Builds>Build log for that source repo. The build log should show "Build successful" at the end.
2. After a build is successful, you can find the built packages stored on your apt repo "http://apt.overcastcloud.com/username/username". Click on "Repositories" link on the left to see your apt repo.
3. You'll find your built package on http://apt.overcastcloud.com/username/username/ under pool/.

## Deleting a package source repository

1. To delete a package source, go to "Sources" link on the left.
2. Click on the pencil icon at the left of the source.
3. Click "Delete" button to delete the git source.

