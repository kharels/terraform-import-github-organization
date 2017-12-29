# WHAT
It's a simple bash script to simply import a Github Organization into Terraform. It uses the Github API and Terraform CLI to import the following resources:
- all public repos (includes pagination support for Orgs with 100+ repos)
- all private repos (includes pagination support for Orgs with 100+ repos)
- all team repos (includes pagination support for Orgs with 100+ repos)
- all teams
- all team memberships
- all organization users

After importing the resources the script _also_ writes a basic terraform.tf config for **each resource**, making this a fully automated process.

# WHY
I like managing things with Terraform. Sometimes those things were created before Terraform supported them. Manually importing hundreds of repos/teams/users/etc is a tedious process.

# HOW
## How the script works
### Public repos
Imports all public repos owned by the organization (includes full pagination support for Orgs with 100+ repos). Also writes a Terraform resource block in a single file (`gihtub-public-repos.tf`), using the following template and populating it with values pulled via the Github API:

```
resource "github_repository" "$PUBLIC_REPO_NAME" {
  name        = "$PUBLIC_REPO_NAME"
  private     = false
  description = "$PUBLIC_REPO_DESCRIPTION"
  has_wiki    = "$PUBLIC_REPO_WIKI"
  has_downloads = "$PUBLIC_REPO_DOWNLOADS"
  has_issues  = "$PUBLIC_REPO_ISSUES"
}
```

### Private repos
Imports all private repos owned by the organization (includes full pagination support for Orgs with 100+ repos). Also writes a Terraform resource block in a single file (`gihtub-private-repos.tf`), using the following template and populating it with values pulled via the Github API:

```
resource "github_repository" "$PRIVATE_REPO_NAME" {
  name        = "$PRIVATE_REPO_NAME"
  private     = true
  description = "$PRIVATE_REPO_DESCRIPTION"
  has_wiki    = "$PRIVATE_REPO_WIKI"
  has_downloads = "$PRIVATE_REPO_DOWNLOADS"
  has_issues  = "$PRIVATE_REPO_ISSUES"
}
```

### Team repos
Imports all team repos owned by the organization (includes full pagination support for Orgs with 100+ repos). Also writes a Terraform resource block in a unique file per team (`gihtub-teams-$TEAM_NAME.tf`), using the following template and populating it with values pulled via the Github API:

```
resource "github_team_repository" "$TEAM_NAME-$TERRAFORM_TEAM_REPO_NAME" {
  team_id    = "$TEAM_ID"
  repository = "$REPO_NAME"
  permission = "admin" or "push" or "pull"
}
```

### Teams
Imports all teams belonging to the organization. Also writes a Terraform resource block in a unique file per team (`gihtub-teams-$TEAM_NAME.tf`), using the following template and populating it with values pulled via the Github API:

```
resource "github_team" "$TEAM_NAME" {
  name        = "$TEAM_NAME"
  description = "$TEAM_DESCRIPTION"
  privacy     = "closed" or "secret"
}
```

### Team memberships
Imports the team membership for all teams owned by the organization (what users belong to what teams). Also writes a Terraform resource block in a unique file per team (`gihtub-team-memberships-$TEAM_NAME.tf`), using the following template and populating it with values pulled via the Github API:

```
resource "github_team_membership" "$TEAM_NAME-$USER_NAME" {
  username    = "$USER_NAME"
  team_id     = "$TEAM_ID"
  role        = "maintainer" or "member"
}
```

### Organization users
Imports all users belonging to the organization. Also writes a Terraform resource block in a single file (`gihtub-users.tf`), using the following template and populating it with values pulled via the Github API:

```
resource "github_membership" "$USER_NAME" {
  username        = "$USER_NAME"
  role            = "member"
}
```

## How to use the script
### Requirements
- An existing Github account with a user that belongs to an organization
- A github [personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) with the following permissions:
  - repo (all)
  - admin:org (all)
  - delete_repo
- Terraform
- jq

### Do it
- `git clone` this repo
- configure the variables at the top of the script
  - GITHUB_TOKEN=''
  - ORG=''
- run the script
- run a terraform plan to see that everything was imported and that no changes are required.
  - some manual modifications _could_ be required since not every field supported by Terraform has been implemented by this script.

# FAQ
- Q) Why bash?
  - A) I like bash.
- Q) Do you plan on extending this?
  - A) Sure, see the TODO section.

# TODO
[Issues](https://github.com/chrisanthropic/terraform-import-github-organization/issues?q=is%3Aopen+is%3Aissue+label%3Aenhancement)
