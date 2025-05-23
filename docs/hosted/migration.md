# Migrating From an Existing Minder Instance

If you have an existing Minder configuration on one server but want to migrate to another (either hosted -> on premises, or hosted -> a different host), the following steps should enable you to move your existing profiles (policy) and entities (repositories).   You'll lose existing evaluation history, but it should be safe to run have _two_ Minder instances running at the same time -- they may race to remediate issues, but remediation should generally be idempotent.  If you have remediations that open PRs, you'll probably get two PRs, but if you merge one, both Minder instances should see the problem resolved, and the other PR should be closed.

## Migrating Profiles

With the latest [0.0.87 Minder CLI release](https://github.com/mindersec/minder/releases/v0.0.87) (also available on `brew` and `winget`), the new [`profile export`](https://mindersec.github.io/ref/cli/minder_profile_export) and [`apply`](https://mindersec.github.io/ref/cli/minder_apply) commands should make this fairly easy.  By April 8, there will be an additional change to make it easier to keep credentials for multiple servers at once.

First, authenticate to the source Minder instance (using the Stacklok-hosted cloud instance as an example):

```
minder auth login --grpc-host api.stacklok.com
```

For each profile you want to migrate, export the profile and supporting rule types:
```
minder profile list --grpc-host api.stacklok.com'
```

Get the profile as yaml:

```
minder profile export --grpc-host api.stacklok.com -n $NAME -o $NAME.yaml
```

Now, log in to the new Minder instance (for example, the [Custcodian hosted instance](./index.md)):

```
minder auth login --grpc-host api.custcodian.dev
```

Apply the profile to the target instance:

```
minder apply --grpc-host api.custcodian.dev -f $NAME.yaml
```

## Migrating Repositories

To register repositories on the new Minder instance, you'll need to start by enrolling each organization into the Minder instance as a separate provider.  You can get a list of providers and repositories from the source Minder instance (in this example, the Stacklok-hosted cloud instance) with:

```
minder auth login --grpc-host api.stacklok.com
minder provider list --grpc-host api.stacklok.com
minder repo list --grpc-host api.stacklok.com
```

In particular, take note of the provider names (which will look like `github-app-$ORGNAME`) and the `OWNER` and `NAME` fields in the repository listing.

To enroll the organizations in the new Minder instance (using the [Custcodian hosted instance](./index.md) as an example):

```
minder auth login --grpc-host api.custcodian.dev
# For each GitHub organization, run the following command, and select
# the organization in the GitHub App installation flow that follows:
minder provider enroll --grpc-host api.custcodian.dev
```

Once you have enrolled the organizations with the Minder GitHub app, you can register each repository with the following command:

```
minder repo register --grpc-host api.custcodian.dev -n owner1/repo1,owner1/repo2,owner2/repo3
```