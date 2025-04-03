# shared-files
A simple way to share files across multiple repos without editing them manually.  
Write once, push anywhere 😄

## Premise
This repo is meant to be used in a GitHub organization or user account with multiple repos.  
It allows you to share files across those repos without having to edit them manually one repo at a time.  
It uses GitHub Actions to automatically open PRs in the repos you want to update whenever you push changes to the files in this repo. If a PR already exists, it simply updates it.  
The idea of this repo emerged from my internship at Nexelec.

## Usage
### First time usage
0. Copy the `.github` and `data` folders to the root of your repo.
1. In [`.github/workflows/auto-update.yml`](.github/workflows/auto-update.yml), edit the `ORG` env var (line 82) to your GitHub username/organization name. If your ORG/user default branch isn't `master`, change that in line 115. If the default branch of that repo isn't `master` either, change that on line 6.
2. Edit [`data/files_to_check.json`](data/files_to_check.json) to add the files you want to check. The format is as follows : the key is the path to the file in this repo and the value is the path to the file in the repos you want to update. This is done in order to allow some nested files to be updated without creating unnecessary folders here. Example :
```json
{
  "release.yml": ".github/release.yml",
  "FUNDING.yml": ".github/FUNDING.yml",
  "bug_report.md": ".github/ISSUE_TEMPLATE/bug_report.md"
}
```
3. Edit the [`data/repos.json`](data/repos.json) file to add the repos you want to update. It's an array of repo names, and wildcards are allowed, so for example `"repo"` will match `"ORG/repo"`, `"project-*"` will match `"ORG/project-1"`, `"ORG/project-2"`, ... Beware ! It isn't possible to just specify `"*"`, as it will create an endless loop since it will want to update this very repo too. The format is as follows : 
```json
[ "repo", "project-*" ]
```
4. If it's the first time you run this repo, empty the content of [`data/last_processed_commit.json`](data/last_processed_commit.json). This will force the workflow to run fresh. After processing your repos, it's automatically updated with the last commit hash oh this repo in order to process the delta at each run. Format is as follows :
```json
{ "commit": "123456abc" }
```
5. You must create a [PAT](https://github.com/settings/personal-access-tokens) (can be a fine-grained one) in order to access the ORG/user (yes it works for private repos too !). The user who creates the PAT will be the user opening the PRs. Here's the token properties :
  - **Resource owner** : The user/organization name of the repos you want to update
  - **Expiration** : No expiration (or as you want, but it will need to be updated when it expires)
  - **Repository access** : All repositories
  - **Repository permissions** :
    - **Contents** : Read and write
    - **Metadata** : Read-only
    - **Pull requests** : Read and write
    - **Workflows** : Read and write  
Go in your repo's Settings -> Secrets and variables -> Actions -> New repository secret. Name it exactly `PAT` and paste the token you just created.
6. Commit and push the changes to your repo. The workflow will run automatically and update the files in the repos you specified by opening a PR. You can check the logs of the workflow in the Actions tab of your repo.

### Further updates
Whenever you need, either adding a new file to propagate, editing one of the already existing ones or adding/removing a new repo, simply add/edit the needed files and push the changes.
