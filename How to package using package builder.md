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

In case you don't have a debian folder in your git source repo, package builder will place a basic debian folder into the repo. The package will be created using defaults. This may be good for basic libraries, but may create problems for custom services and openstack components which require specific packaging rules. Please review openstack debian folders on Launchpad.net and include those for packaging openstack components.

## Deleting a package source repository

1. To delete a package source, go to "Sources" link on the left.
2. Click on the pencil icon at the left of the source.
3. Click "Delete" button to delete the git source.

## Adding an external repository to a mirror-set

If you log in on overcastcloud.com, the "Profile" link in the top right corner will show you an access token. You can use this token to interact with the API. You need to set an HTTP header like so: "Authorization: Token 3124d5d4ffeeeed6236"

Using Postman on chrome / httpie gem on REST call is recommended,
make a GET call to https://aasemble.com/api/v1/repositories
This gives a list of repositories belonging to you.
Note down the id of repository as highlighted by the result
```
"count": 2,
"next": null,
"previous": null,
"results": [
{
"self": "https://aasemble.com/api/v1/repositories/7/",
"user": "JioCloud",
"name": "JioCloud",
"key_id": "8FD735B6",
"sources": "https://aasemble.com/api/v1/repositories/7/sources/",
"binary_source_list": "deb http://apt.overcastcloud.com/JioCloud/JioCloud overcast main",
"source_source_list": "deb-src http://apt.overcastcloud.com/JioCloud/JioCloud overcast main",
"external_dependencies": "https://aasemble.com/api/v1/repositories/7/external_dependencies/"
},
{
"self": "https://aasemble.com/api/v1/repositories/9/",
"user": "akash1808",
"name": "akash1808",
"key_id": "7869CD25",
"sources": "https://aasemble.com/api/v1/repositories/9/sources/",
"binary_source_list": "deb http://apt.overcastcloud.com/akash1808/akash1808 overcast main",
"source_source_list": "deb-src http://apt.overcastcloud.com/akash1808/akash1808 overcast main",
"external_dependencies": "https://aasemble.com/api/v1/repositories/9/external_dependencies/"
}
]
}
```

2. POST /external_dependency/
Again, JSON document with the following keys:
url: The url for the repository (e.g. http://ubuntu-cloud.canonical.com/ubuntu)
series: The series (for the cloud archive, this is e.g. "trusty-updates/kilo")
repository: a URL referencing the repository that you want to add this dependency for
key: The APT key for the given repo

If cloud.key is the signing key for the cloud-archive, you can use httpie, like so:


```
http POST https://aasemble.com/api/v1/external_dependencies/ 'Authorization:Token 123455413532123' url=http://ubuntu-cloud.archive.canonical.com/ubuntu series=trusty-updates/kilo components=main repository=https://aasemble.com/api/v1/repositories/<id from above>/ key=@cloud.key
```

Please ensure that format of key file is not modified i.e. line endings are not truncated.

3. Once added one can make a GET call to list dependency.

https://aasemble.com/api/v1/repositories/<id>/external_dependencies/

Example output:
```
{
"count": 1,
"next": null,
"previous": null,
"results": [
{
"self": "https://aasemble.com/api/v1/external_dependencies/10/",
"url": "http://ubuntu-cloud.archive.canonical.com/ubuntu",
"series": "trusty-updates/kilo",
"components": "main",
"repository": "https://aasemble.com/api/v1/repositories/9/",
"key": "-----BEGIN PGP PUBLIC KEY BLOCK-----\nVersion: GnuPG v1\n\n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\nOa4g9309p/+af/HU1smBrOfIeRoxb8jQoHu3\n=xg4S\n-----END PGP PUBLIC KEY BLOCK-----"
}
]
}
```

## How to create mirror-sets and snapshots

The Mirror-set/snapshots feature comprises of 3 steps
1. Create a mirror of an external/internal repo
2. Combine multiple mirrors and create a mirror-set
3. From the mirror-set create snapshots

To give an example of how it works (from a software developer perspective)
Lets say you are a nova-api developer, you are intrested in developing a feature/fixing a bug. The above mentioned steps will help you to package the code and place it into a repository. So the next step is how can you deploy this feature/bug-fix with sufficient testing into production. So you follow the below steps

1. First create a mirror from the repository you wish to deploy
2. From that mirror, create a mirror-set , here you can add other mirrors which you want (maybe dependecies of your code, upstream ubuntu repo)
3. From this mirror set you will create a snapshot , which give you the URL of the snapshot of your mirrorset.
4. Use this URL in your sources.list to get your feature/bug fix into the environment for testing
5. Lets say your fix works, all you need to do is change the URL in the next environment to point it to your snapshot
6. And thereby get your patch into production.

The primariy benefit of this feature is, if lets say you push a bad code and it gets placed into the repository, it will not impact other environments during next upgrade, because the sources.list will always point to a snapshot of a repo and never the actual repo (For eg : Contrail repo is :-http://jiocloud.rustedhalo.com/contrailv2/, whereas the snapshot of it is :- https://aasemble.com/snapshots/21/jiocloud.rustedhalo.com/contrailv2/). So you can see even if bad code gets into the main repo, none of the environment gets impacted because it always refer to the snapshot. It also helps in terms of rollbacks as you always know the last snapshot which worked well.

Currently all the below steps is possible only from API and not from UI
##Creating a Mirror

For creating a mirror , do an HTTP POST with the below JSON ( the header should contain the Authorization Token as described in above steps)

```
URL :- https://aassemble.com/api/v2/mirrors/

```
An example JSON data for creating the mirrors is given below

```
{
  "url": "http://jiocloud.rustedhalo.com/contrailv2",
  "series": ["trusty"],
  "components": ["main"],
  "public": true
}
```
The JSON data is pretty self explantory, but a quick look into it

1. URL :- The url of the APT-repository which you want to mirror
2. series :- 
3. Components:- what components in that repo do you want to mirror (main, restricted etc)
4. Public :- If the mirror needs to be visible to all 

You will get a response from the server as below (if all the fields are true)

```

{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "self": "https://aasemble.com/api/v2/mirrors/1895442f-5615-43bf-9afe-dfdbdedeaa46/",
            "url": "http://jiocloud.rustedhalo.com/contrailv2",
            "series": [
                "trusty"
            ],
            "components": [
                "main"
            ],
            "public": true,
            "refresh_in_progress": false
        }
    ]
}

```
From the response we can see the UUID of the mirror we created, this we will refer in later steps. 

##Refreshing a Mirror
From the output of the above , there is one field namely 'refresh_in_progress'. This is updated whenver a refresh of a mirror is called. The above POST, will create a mirror but it still does not hold any data. For the backend to start the mirroring process, a refresh to that mirror needs to be called

```
HTTP POST URL :- 'https://aasemble.com/api/v2/mirrors/1895442f-5615-43bf-9afe-dfdbdedeaa46/refresh/'

```
The UUID should be replaced by the mirror UUID got in the first POST
Now if we do a HTTP GET to the below URL

```
URL :- https://aasemble.com/api/v2/mirrors/1895442f-5615-43bf-9afe-dfdbdedeaa46/

```
We get the below response

```
{
        {
            "self": "https://aasemble.com/api/v2/mirrors/1895442f-5615-43bf-9afe-dfdbdedeaa46/",
            "url": "http://jiocloud.rustedhalo.com/contrailv2",
            "series": [
                "trusty"
            ],
            "components": [
                "main"
            ],
            "public": true,
            "refresh_in_progress": true
        }
}

```

We can see that the 'refresh_in_progress' field is now true, depending upon the size of the APT-repo, this will take anywhere from few minutes to couple of hours for large repos (eg:- Ubuntu Apt Repo)

Keep doing the the GET call till you see the refresh_in_progress is back to 'false'

## Creating Mirror Sets
Now that we have created mirrors of different repos we can combine the mirrors into mirror sets.

To create mirror sets do an HTTP POST to the below URL

```
URL:- https://aasemble.com/api/v2/mirror_sets/ 

```
with the below JSON

```
{
  "mirrors": ["https://aasemble.com/api/v2/mirrors/1895442f-5615-43bf-9afe-dfdbdedeaa46/", "https://aasemble.com/api/v2/mirrors/9815442f-5615-43bf-9afe-dfdbdedeeb4f/"]
}

```
So the above JSON data contains all the mirrors you need to create a mirror set from.

The reponse to a successful post is as below

```
{
    "self": "https://aasemble.com/api/v2/mirror_sets/4367dff4-c740-4c73-817c-8bf4eaf81ab2/",
    "mirrors": [
        "https://aasemble.com/api/v2/mirrors/1895442f-5615-43bf-9afe-dfdbdedeaa46/",""https://aasemble.com/api/v2/mirrors/9815442f-5615-43bf-9afe-dfdbdedeeb4f/"
    ]
}

You can get the URL of the mirror_set created with its UUID. From this we proceed to create snapshots

##Creating Snapshots
To create snapshots we do a HTTP POST to the below URL

```
URL:- https://aasemble.com/api/v2/snapshots/

```
with the following JSON data, which contains the mirrorset URL which we got from above

```
{
  "mirrorset": "https://aasemble.com/api/v2/mirror_sets/4367dff4-c740-4c73-817c-8bf4eaf81ab/"
}

```
Following a successful Post we get the following

```
{
    "self": "https://aasemble.com/api/v2/snapshots/44ffde23-c740-4c73-817c-8bf4eafab81/",
    "timestamp": "2015-11-05T06:49:56.430630Z",
    "mirrorset": "http://localhost:8000/api/v1/mirror_sets/4367dff4-c740-4c73-817c-8bf4eaf81ab/"
}

```

Now this snapshot can be added into the sources.list in your environment.

Lets say you created a snapshot from a mirror set which had 2 mirrors (namely: jiocloud.rustedhalo.com and archive.ubuntu.com). Then the URL for jiocloud.runstedhalo.com in your sources.list will be

```
URL:- https://aasemble.com/snapshots/44ffde23-c740-4c73-817c-8bf4eafab81/jiocloud.rustedhalo.com/

```

and similarly for your archive.ubuntu.com will be

```
URL:- https://aasemble.com/snapshots/44ffde23-c740-4c73-817c-8bf4eafab81/archive.ubuntu.com/

```

So if there is a new snapshot is created then all we need to change is the UUID in URL with the latest snapshot UUID


