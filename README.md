# How are the docs on openliberty.io built and what is a docs playbook?

The docs on https://openliberty.io/ are built using a tool called [Antora](https://antora.org/). It is a tool that helps streamline writing, managing, and versioning documentation.

The [docs playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml) is a key file used by Antora see https://docs.antora.org/antora/latest/playbook/. It tells the build what branches and versions of the [docs repo](https://github.com/OpenLiberty/docs) to use. Antora aggregates all of the content specified in the playbook and creates versions of pages based on the `version` attribute in the `antora.yml` of each doc branch. See https://github.com/OpenLiberty/docs/blob/vNext/antora.yml for an example. If Antora finds the same doc in multiple doc branches, it will create a version picker on the site that lets you easily switch between versions of the doc.

There are three doc playbooks used in the builds, located in the following branches in this repo: [prod](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml), [staging](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml), and [draft](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml).

https://openliberty.io/ is built using the [prod playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). This file explicitly only contains versions of the documentation for released Open Liberty versions. This is the file that needs to be [updated each release](#publishing-a-new-release-of-open-liberty-docs) to add the new release to be published to openliberty.io. This is intentionally manual and does not use wildcards for the branches to prevent any unintended branches/content from being published to the site.

The [staging site](http://staging-openlibertyio.mybluemix.net/) is built using the [staging playbook](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml). This playbook contains the `staging` branch which is used to stage in changes for the next documentation version to be released. It also contains a wildcard `V*.0.0.*-staging` rule which pulls in all of the branches used to stage in changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

The [draft site](https://draft-openlibertyio.mybluemix.net/) is built using the [draft playbook](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml). This playbook contains the `draft` branch which pulls in all of the content currently being drafted. It also contains a wildcard `V*.0.0.*-draft` which pulls in all of the branches used to draft changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

# Publishing a new release of Open Liberty Docs

This is the flow that must be followed when releasing a new version of docs on openliberty.io. These steps also update the `vNext`, `staging` and `draft` branches of the [docs repo](https://github.com/OpenLiberty/docs) so that the next version of docs can be worked on and ensures that fixes can still be made to the docs of the version being released.

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mybluemix.net/docs/), and all of the `staging` branch changes have been merged into the `vNext` branch, then create a branch off of the `vNext` branch with the next release version number. E.g. if the most recent version on openliberty.io is `20.0.0.8`, then create a `v20.0.0.9`  branch off of `vNext` by checking out the `vNext` branch and running `git checkout -b v20.0.0.9` and `git push` the branch.

2. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the version branch just released which will be used for retroactive fixes to this version in the future. E.g. if `v20.0.0.9` is the version being released, then from that branch run `git checkout -b v20.0.0.9-staging` and push that branch to the docs repo. Switch back to the `v20.0.0.9` branch and run `git checkout -b v20.0.0.9-draft` and push that branch to the docs repo.

3. Create a pull request to add this new release version to the `branches` section of the [`prod playbook`](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml).

4. Create a pull request to change the `version` in `antora.yml` in the [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) branches to the next release `vX.0.0.X` version incremented from the current release being published to openliberty.io. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released.

5. Once the antora.yml [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) versions have been updated, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

6. (Temporary for now). Until Antora natively supports 'latest' redirecting to the latest version, we have to update what version `latest` in the url redirects to: Update the redirect for both `/docs/latest/*` and `/docs/latest/` in the development branch of the UI [redirect file](https://github.com/OpenLiberty/openliberty.io/blob/development/src/main/webapp/WEB-INF/redirects.properties) to redirect to the new released doc version (without a `v` in the url).

7. (Temporary for now) Rebuild the [development site](http://dev-openlibertyio.mybluemix.net/) using the [IBM Cloud development pipeline](https://cloud.ibm.com/devops/pipelines/61002d76-1c13-4502-a109-3b794b3f2350?env_id=ibm:yp:us-south) and test that the docs link works and redirects to the newest doc version once the site is done building.

8. (Temporary for now) Make a pull request from the [UI development branch](https://github.com/OpenLiberty/openliberty.io/tree/development) into the [UI master branch](https://github.com/OpenLiberty/openliberty.io/tree/master).

9.  Rebuild the openliberty.io site using the IBM Cloud pipeline(https://cloud.ibm.com/devops/pipelines/fcc7c3e9-9c40-4a58-8a7f-09c08413ab7d?env_id=ibm:yp:us-south).