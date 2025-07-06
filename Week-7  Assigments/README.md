 # **Week:-7 {Assignment }**




---

#  **Project Title: User Groups and Policy Management**

---

### PART A: **On Linux (Ubuntu/CentOS)**

#### Goal:

* Create multiple user groups.
* Add users to those groups.
* Apply policies (like file/folder access, command restrictions, etc.) per group.

---

#### Step 1: Create User Groups

```bash
sudo groupadd devteam
sudo groupadd testteam
sudo groupadd admintools
```

---

#### Step 2: Create Users and Add to Groups

```bash
sudo useradd -m -s /bin/bash alice
sudo usermod -aG devteam alice

sudo useradd -m -s /bin/bash bob
sudo usermod -aG testteam bob

sudo useradd -m -s /bin/bash charlie
sudo usermod -aG admintools charlie
```

---

#### Step 3: Verify Group Membership

```bash
groups alice
groups bob
groups charlie
```

---

#### Step 4: Create Folders and Set Group Permissions

```bash
sudo mkdir /opt/dev
sudo mkdir /opt/test
sudo mkdir /opt/admin

sudo chown :devteam /opt/dev
sudo chown :testteam /opt/test
sudo chown :admintools /opt/admin

sudo chmod 770 /opt/dev
sudo chmod 770 /opt/test
sudo chmod 770 /opt/admin
```

> Now only group members can access respective directories.

---

#### Step 5: Restrict Command Usage (Optional Policy Example)

You can restrict some commands like `shutdown`, `reboot` for non-admin groups using `/etc/sudoers.d`.

```bash
sudo visudo -f /etc/sudoers.d/testteam
```

Add:

```
%testteam ALL=(ALL) NOPASSWD: /usr/bin/ls, /usr/bin/cat
```

> This restricts `testteam` users to only `ls` and `cat` via sudo.

---

#### Step 6: Create Group-Based Shell Profiles (Optional)

```bash
sudo nano /etc/profile.d/devteam.sh
```

```bash
if groups $USER | grep &>/dev/null '\bdevteam\b'; then
    export DEV_ENV=true
    echo "Welcome to Dev Environment"
fi
```

---

### Linux Project Complete

---

### PART B: **On Windows Server (using Group Policy)**

> Prerequisites: Windows Server + Active Directory + Group Policy Management Console (GPMC)

---

#### Goal:

* Create user groups (OU-based or Security Groups).
* Apply group policies like Desktop restrictions, software control, etc.

---

#### Step 1: Open Active Directory Users and Computers

* `Start` → `Server Manager` → `Tools` → `Active Directory Users and Computers`

---

#### Step 2: Create Organizational Units (OU)

* Right-click domain → New → Organizational Unit
* Example:

  * `OU_Developers`
  * `OU_Testers`
  * `OU_Admins`

---

#### Step 3: Create User Accounts and Add to Groups

* Right-click on OU → New → User
* Assign users:

  * Alice in `OU_Developers`
  * Bob in `OU_Testers`
  * Charlie in `OU_Admins`

---

#### Step 4: Create Security Groups (Optional)

* Right-click OU → New → Group → Choose `Global` + `Security`
* Add users to groups:

  * `Developers`, `Testers`, `Admins`

---

#### Step 5: Open Group Policy Management Console

* `Start` → `Run` → `gpmc.msc`

---

#### Step 6: Create and Link GPO to OUs

* Right-click OU (e.g. `OU_Developers`) → "Create a GPO in this domain, and Link it here"
* Name it: `Dev_GPO`

---

#### Step 7: Edit the GPO (Examples)

##### Example 1: Restrict Access to Control Panel

* User Config → Policies → Admin Templates → Control Panel → Prohibit access → **Enabled**

##### Example 2: Disable Task Manager

* User Config → Policies → System → Ctrl+Alt+Del options → Remove Task Manager → **Enabled**

##### Example 3: Set Wallpaper

* User Config → Admin Templates → Desktop → Desktop → Desktop Wallpaper → **Set path**

##### Example 4: Logoff Inactivity

* User Config → Admin Templates → System → Logon → “Automatically log off users” → Set timeout

---

#### Step 8: Force Policy Update

On client machines:

```powershell
gpupdate /force
```

---

#### Step 9: Test Group Policies

Log in as each user (e.g., Alice, Bob, Charlie) and confirm restrictions or changes applied as expected.

---









------------------------------------------------------
Apply branch policies such that only the project administrator can access the master branch; contributors cannot.
----------------------------------------------------

To apply **branch policies** where **only the project administrator can access the `master` branch** and **contributors cannot**, follow the steps based on your version control system. Below are detailed instructions for **Azure Repos (Azure DevOps)** and **GitHub**, the two most common platforms.

---

## Option 1: Azure DevOps (Azure Repos)

### Goal:

Restrict access to the `master` branch such that only users in the "Project Administrators" group can push or modify it.

### Prerequisites:

* You must have **Project Administrator** privileges.

---

### Step-by-Step Guide:

### Step 1: Open Azure DevOps Project

1. Go to your Azure DevOps project.
2. Navigate to **Repos > Branches**.
3. Find the `master` branch.

---

### Step 2: Set Branch Security

1. Click on the **ellipsis (⋯)** next to `master`.
2. Select **Branch security**.

---

### Step 3: Remove Contributor Permissions

1. Select the **Contributors** group.
2. Set the following permissions to **Deny**:

   * Contribute
   * Force push (rewrite history, delete branches)
   * Create branch
   * Create tag
3. Click **Save changes**.

---

### Step 4: Grant Access to Project Administrators

1. Select **Project Administrators** group.
2. Ensure the following permissions are set to **Allow**:

   * Contribute
   * Create branch
   * Force push (if required)
   * Delete branch (if required)
3. Save the changes.

---

### Step 5: Optional – Enable Branch Policies

1. Go to **Project > Repos > Branches**.
2. Click **Policies** under the `master` branch.
3. Enable:

   * Require pull request reviews before merging
   * Limit merge types
   * Automatically include reviewers

> These policies further protect the branch from unauthorized changes.

---

## Option 2: GitHub

### Goal:

Allow only admins to push to `master` and block contributors from accessing it.

---

### Step-by-Step Guide:

### Step 1: Go to Repository Settings

1. Open your GitHub repository.
2. Click **Settings** → **Branches**.

---

### Step 2: Add Branch Protection Rule

1. Click **Add rule**.
2. Under "Branch name pattern", type: `master`

---

### Step 3: Configure Protection

Check these options:

* **Require pull request before merging**
* **Require approvals**
* **Restrict who can push to matching branches**

---

### Step 4: Restrict Push Access

1. Under **Restrict who can push to matching branches**, select only the **admin** or a specific team (e.g., `project-admins`).
2. Do **not** include contributors.
3. Save the rule.

---

## Final Test:

1. Log in as a contributor.
2. Attempt to push to `master`:

   * It should be denied.
3. Log in as an administrator.
4. Push to `master`:

   * It should succeed.

---



-------------------------------------------------
## Azure DevOps – Apply Branch Security and Locks
-----------------------------------------------------

### Step 1: Open Azure DevOps Project

1. Go to your Azure DevOps portal.
2. Select your **project**.
3. Navigate to **Repos > Branches**.

---

### Step 2: Locate the `master` Branch

1. Find the `master` (or `main`) branch in the list.
2. Click on the **⋯ (ellipses)** next to the branch.
3. Select **Branch security**.

---

### Step 3: Configure Branch Security

#### 3.1. Deny Access to Contributors:

1. Select the **Contributors** group.
2. Set the following permissions to **Deny**:

   * Contribute
   * Force push
   * Create branch
   * Delete

#### 3.2. Allow Access to Project Administrators:

1. Select **Project Administrators** group.
2. Set the following permissions to **Allow**:

   * Contribute
   * Create branch
   * Force push (optional)
   * Delete (optional)

Click **Save changes** after each group configuration.

---

### Step 4: Apply Branch Lock

1. On the **Branches** page, click the **⋯** next to the `master` branch.
2. Select **Lock**.

**Effect of Locking a Branch:**

* Prevents everyone (including admins) from pushing directly.
* Only unlockers (typically admins) can make updates after unlocking it.

---

### Step 5 (Optional): Enable Branch Policies

1. On the `master` branch row, click **Policies**.
2. Configure:

   * Require minimum number of reviewers.
   * Limit merge types.
   * Require status checks.
   * Automatically include reviewers.

---

## GitHub – Apply Branch Security and Locks

### Step 1: Open GitHub Repository

1. Go to your repository.
2. Click on **Settings**.
3. Select **Branches**.

---

### Step 2: Create a Branch Protection Rule

1. Click **Add rule**.
2. In **Branch name pattern**, type `main` or `master`.

---

### Step 3: Configure Security Settings

Enable the following options:

* Require a pull request before merging
* Require approvals before merging
* Require status checks to pass
* Restrict who can push to this branch

> In **Restrict who can push**, allow only specific admins or teams.

---

### Step 4: Lock the Branch (Manual Lock)

GitHub doesn’t have a branch lock like Azure DevOps, but you can emulate it:

* Set **"Include administrators"** to enforce protection on admins.
* Remove all push access by restricting it to **nobody** unless through PR.

---

### Step 5 (Optional): Archive the Branch (for hard lock)

If the branch is obsolete and should be preserved:

1. Protect it.
2. Add a tag or archive it.
3. Lock repository via organization settings (read-only).

---




-------------------------------------------------------------

### Azure DevOps – Branch and Path Filters (for Pipelines)

-------------------------------------------------
### Use Case:

Run pipeline only when:

* Code is pushed to a **specific branch** (branch filter)
* Changes are made to **specific files/folders** (path filter)

---

### Step 1: Open Pipeline Configuration

1. Go to your Azure DevOps project.
2. Navigate to **Pipelines > Pipelines**.
3. Click the pipeline you want to configure.
4. Edit the YAML file or pipeline trigger settings.

---

### Step 2A: Apply **Branch Filters** (YAML)

In your `azure-pipelines.yml`:

```yaml
trigger:
  branches:
    include:
      - master
      - release/*
    exclude:
      - dev
```

> This pipeline runs only when changes are pushed to `master` or any branch under `release/`, and not for `dev`.

---

### Step 2B: Apply **Path Filters** (YAML)

```yaml
trigger:
  branches:
    include:
      - master
  paths:
    include:
      - src/**
      - docs/*
    exclude:
      - tests/**
```

> Pipeline runs **only if files inside `src/` or `docs/` are modified** and **not if only `tests/` files are changed**.

---

### Step 3 (Classic Pipeline – UI)

For classic pipelines (no YAML):

1. Go to **Pipelines > \[Your Pipeline] > Edit**.
2. Click on **Triggers**.
3. Under **Continuous Integration (CI)**:

   * Add **branch filters** using `+` → e.g., `master`, `release/*`
   * Add **path filters** below (include/exclude specific folders like `/src`, `/docs`, etc.)

---

### GitHub Actions – Branch and Path Filters

### Step 1: Open Your Workflow File (e.g., `.github/workflows/build.yml`)

---

### Step 2: Apply Branch Filters

```yaml
on:
  push:
    branches:
      - main
      - release/*
    tags:
      - 'v*'
```

> Runs workflow only on pushes to `main` or `release/*` branches, or version tags.

---

### Step 3: Apply Path Filters

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'src/**'
      - 'docs/**'
```

> Runs only if a file in `src/` or `docs/` is changed.

You can also exclude paths:

```yaml
    paths-ignore:
      - 'README.md'
      - 'tests/**'
```

---

### Full Example in GitHub Actions

```yaml
name: CI Build

on:
  push:
    branches:
      - main
    paths:
      - 'src/**'
      - 'docs/**'
    paths-ignore:
      - 'tests/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: echo "Running build..."
```

---



------------------------------------------
## Git + Azure DevOps: Create and Apply a Pull Request
--------------------------------------------

### Prerequisites:

* A repository in Azure Repos.
* A personal or feature branch created.
* Changes committed and pushed to that branch.

---

### Step 1: Clone the Repo and Create a Feature Branch

```bash
git clone https://dev.azure.com/YourOrg/YourProject/_git/YourRepo
cd YourRepo

git checkout -b feature-branch
```

---

### Step 2: Make Changes and Commit

```bash
echo "New feature" >> newfile.txt
git add newfile.txt
git commit -m "Add new feature"
```

---

### Step 3: Push the Branch to Azure Repos

```bash
git push origin feature-branch
```

---

### Step 4: Create Pull Request in Azure DevOps

1. Go to Azure DevOps → Repos → Pull Requests.
2. Click **New Pull Request**.
3. Set:

   * **Source branch**: `feature-branch`
   * **Target branch**: `master` (or `main`)
4. Add title, description, and reviewers.
5. Click **Create**.

---

### Step 5: Apply (Merge) the Pull Request

1. After review, open the pull request.
2. Click **Complete** or **Approve + Complete**.
3. Optionally, choose:

   * **Squash** commits
   * **Delete** the source branch after merge
4. Click **Complete merge**.

---

### Git + GitHub: Create and Apply a Pull Request

### Step 1: Create and Push Feature Branch

```bash
git checkout -b feature-branch
# Make changes
git add .
git commit -m "New feature"
git push origin feature-branch
```

---

### Step 2: Create a Pull Request on GitHub

1. Visit your repo: `https://github.com/username/repo`.
2. GitHub will show a "Compare & pull request" button → click it.
3. Review the changes.
4. Set:

   * **Base branch**: `main`
   * **Compare branch**: `feature-branch`
5. Add title, description, and assign reviewers.
6. Click **Create Pull Request**.

---

### Step 3: Apply (Merge) the Pull Request

1. After approval and checks pass:
2. Click **Merge pull request**.
3. Select the merge strategy:

   * Create a merge commit
   * Squash and merge
   * Rebase and merge
4. Click **Confirm merge**.

---

---------------------------------------------------------------------
# **Apply branch policy, Apply branch security.**
---------------------------------------------------------------


---

##  Azure DevOps

### 1. Apply Branch Policy (e.g., Require Pull Request, Approvals)

#### Step 1: Open Branch Policies

1. Go to your Azure DevOps project.
2. Navigate to **Repos > Branches**.
3. Click the **ellipsis (⋯)** next to the branch (e.g., `master` or `main`).
4. Select **Branch policies**.

#### Step 2: Configure Policy Settings

Enable the following options based on your need:

* **Require a minimum number of reviewers** (e.g., 2)
* **Check for linked work items**
* **Check for comment resolution**
* **Limit merge types** (e.g., only squash or rebase)
* **Build validation** (connect a pipeline that must pass)
* **Status checks** (ensure external checks pass)
* **Automatically include reviewers** (teams or individuals)

Click **Save** after setting policies.

---

### 2. Apply Branch Security (e.g., Restrict who can push, contribute)

#### Step 1: Open Branch Security Settings

1. Go to **Repos > Branches**.
2. Click the **ellipsis (⋯)** next to the branch name.
3. Select **Branch security**.

#### Step 2: Configure Security for Groups/Users

* Select a user group (e.g., **Contributors**, **Project Administrators**, or individual users).

Modify permissions:

| Permission                   | Project Administrators | Contributors |
| ---------------------------- | ---------------------- | ------------ |
| Contribute                   | Allow                  | Deny         |
| Force Push (Rewrite History) | Allow (optional)       | Deny         |
| Create Branch                | Allow                  | Deny         |
| Delete Branch                | Allow                  | Deny         |

Click **Save changes** after setting permissions.

---

## GitHub

### 1. Apply Branch Protection Policies

#### Step 1: Open Branch Rules

1. Go to your repository on GitHub.
2. Click on **Settings > Branches**.
3. Click **Add rule**.

#### Step 2: Configure Protection Rule

Set `Branch name pattern`: `main` or `master`

Check or enable the following:

* **Require pull request before merging**
* **Require approvals** (1 or more reviewers)
* **Require status checks to pass before merging**
* **Require conversation resolution before merging**
* **Include administrators** (if you want admins to follow rules too)
* **Restrict who can push to matching branches**

Click **Create** or **Save changes**.

---

### 2. Apply Branch Security (Push Access Control)

If you enable **“Restrict who can push”**, only selected users or teams can push to the branch. All others will be blocked.

1. Under the branch protection rule, find **“Restrict who can push to matching branches”**
2. Select allowed users/teams (e.g., `admin-team`)
3. Save changes

---


-------------------------------------------
# **Apply triggers in build and release.**
--------------------------------------------


---

##  Azure DevOps – Build Triggers (YAML Pipeline)

### 1. **Trigger on Branch Push (CI Trigger)**

In your `azure-pipelines.yml`:

```yaml
trigger:
  branches:
    include:
      - main
      - release/*
    exclude:
      - experimental/*
```

> This automatically starts the build when someone pushes to `main` or `release/*`.

---

### 2. **Trigger on Pull Request (PR Trigger)**

```yaml
pr:
  branches:
    include:
      - main
```

> This triggers the pipeline when a pull request targets the `main` branch.

---

### 3. **Scheduled Trigger**

```yaml
schedules:
  - cron: "0 2 * * 1"  # Every Monday at 2 AM UTC
    displayName: Weekly Build
    branches:
      include:
        - main
    always: true
```

> Automatically triggers builds on a schedule using cron syntax.

---

##  Azure DevOps – Release Triggers (Classic Release Pipeline)

### 1. **Trigger on Artifact Publish**

1. Go to **Pipelines > Releases**.
2. Edit or create a release pipeline.
3. Click the **lightning bolt ⚡ icon** on the artifact.
4. Enable **"Continuous deployment trigger"**.

> This starts the release automatically when a new build artifact is available.

---

### 2. **Scheduled Release**

1. In your **release stage**, click on the **clock icon**.
2. Set a **recurring schedule** (e.g., daily at 4 AM).
3. Save the release pipeline.

---

### 3. **Manual Trigger (Optional)**

* If no triggers are set, release can still be triggered manually from the **Releases > New release** button.

---

##  GitHub Actions – Build Triggers

### 1. **On Push**

```yaml
on:
  push:
    branches:
      - main
      - release/*
```

### 2. **On Pull Request**

```yaml
on:
  pull_request:
    branches:
      - main
```

### 3. **On Schedule**

```yaml
on:
  schedule:
    - cron: "0 3 * * *"  # Daily at 3 AM UTC
```

---


-------------------------------------------------------------
##  Apply Gates in Azure DevOps (Classic Release Pipeline)
-------------------------------------------------------------



> Gates are available only in **Release Pipelines** (not YAML-based pipelines).
> Use them to **pause deployment** until external conditions are met.

---

### Step-by-Step: Add Gates to a Release Pipeline

### Step 1: Open Your Release Pipeline

1. Go to **Azure DevOps Portal**.
2. Navigate to **Pipelines > Releases**.
3. Click your release pipeline.
4. Click **Edit**.

---

### Step 2: Select a Stage and Add Pre-deployment Conditions

1. Click the **Stage name** (e.g., Staging, Production).
2. Click the **Pre-deployment conditions (lightning bolt icon)**.

---

### Step 3: Enable Gates

1. Turn on the **Gates** toggle.
2. Click **Add** under “Gates” section.

---

### Step 4: Choose Gate Types

Azure DevOps supports the following gate types:

| Gate Type                | Description                                    |
| ------------------------ | ---------------------------------------------- |
| **Invoke REST API**      | Call external services to check readiness      |
| **Azure Monitor Alerts** | Wait for no active alerts in Azure Monitor     |
| **Azure Function**       | Call a custom Azure Function                   |
| **Work Item Query**      | Validate existence or status of work items     |
| **Query Azure DevOps**   | Check for new bugs, approvals, or test results |

Example: Invoke a REST API

```http
https://api.example.com/deployment-approval?env=prod
```

If the API returns the expected success response (e.g., 200 OK or custom JSON), the gate passes.

---

### Step 5: Configure Gate Options

* **Gate timeout**: How long to wait for the gate to succeed (e.g., 30 mins)
* **Sampling interval**: How often to re-evaluate (e.g., every 5 mins)
* **Minimum success duration**: How long the gate result must be successful (e.g., 10 mins)

---

### Step 6: Save and Queue Release

1. Click **Save**.
2. Create a new release to test the gate behavior.

---

## Optional: Add Manual Approval After Gates

* Go back to the **Pre-deployment conditions**.
* Under **Approvals**, add required users or groups for manual approval.

This enforces a two-step validation:
**Automated gate check** + **Manual approval**

---

## Use Cases for Gates

* Wait until a change ticket is approved in a tracking system.
* Ensure there are no active alerts in Azure Monitor.
* Validate that recent test results pass.
* Enforce external REST API conditions (e.g., security scan, business validation).

---

## Note:

Gates **cannot** be directly configured in YAML pipelines.
For YAML pipelines, similar checks can be implemented with:

* **Custom scripts**
* **External API calls**
* **Environment approvals**



------------------------------------------------------------------------------------------
# **Apply security on branches so that contributors can create a pull request but cannot directly merge code to master.**
------------------------------------------------------------------------------------------



##  Azure DevOps

### Goal:

* Contributors **can** create pull requests to `master`.
* Contributors **cannot** push or merge directly to `master`.
* Only **Project Administrators** or authorized users can complete PRs.

---

### Step 1: Set Branch Security on `master`

1. Go to **Repos > Branches**.
2. Click the **⋯** next to the `master` branch → Select **Branch security**.

#### For `Contributors` group:

* `Contribute`: **Deny**
* `Force push (rewrite history)`: **Deny**
* `Create branch`: **Allow** (optional)
* `Read`: **Allow**

> This blocks direct pushes to `master`, but allows contributors to view the branch and work with other branches.

#### For `Project Administrators` or lead developers:

* `Contribute`: **Allow**
* `Force push`: **Allow** (optional)

Click **Save changes**.

---

### Step 2: Set Branch Policies on `master`

1. Click **⋯ > Branch policies** for `master`.
2. Configure:

   * **Require pull request**: Enabled
   * **Minimum number of reviewers**: 1 or more
   * **Check for linked work items**: Enabled (optional)
   * **Check for comment resolution**: Enabled (optional)
   * **Limit merge types**: Choose “Squash” or “No fast-forward” if preferred
   * **Automatically include reviewers**: Add relevant teams

> This ensures all changes to `master` come via pull requests that meet review criteria.

---

### Step 3: Test the Setup

1. Log in as a **Contributor**:

   * Create a new branch.
   * Push changes to the new branch.
   * Create a PR to `master` → ✅ Allowed
   * Try pushing directly to `master` → ❌ Denied
2. Log in as an **Admin**:

   * Can approve and complete the PR → ✅ Allowed

---

##  GitHub

### Goal:

* Contributors can submit PRs to `main`/`master`.
* Direct push to `main`/`master` is blocked.
* Only admins can merge.

---

### Step 1: Create a Branch Protection Rule

1. Go to your repo → **Settings > Branches**.
2. Click **Add rule**.

#### Configure the rule:

* **Branch name pattern**: `main` or `master`
* ✅ Require a pull request before merging
* ✅ Require approvals (e.g., 1 or more reviewers)
* ✅ Require status checks (optional)
* ✅ Include administrators (optional if you want rules enforced on admins)
* ✅ Restrict who can push:

  * Add **only admins or specific team**, **do not include contributors**

Click **Create rule**.

---

### Step 2: Verify

1. A contributor:

   * Can create a PR to `main` → ✅ Allowed
   * Cannot push directly to `main` → ❌ Denied
2. An admin:

   * Can approve and merge the PR → ✅ Allowed

---


--------------------------------------------------------------
# **Work Items in Build and Release Pipelines** 
--------------------------

## ✅ 1. Link Work Items Automatically in Azure DevOps Pipelines

### A. In Build Pipelines (YAML or Classic)

When you run a build pipeline, Azure DevOps can automatically associate **related work items** if:

* You **mention a work item ID in your commit message**
  (e.g., `git commit -m "Fix login issue #123"`)

* The work item is **linked to the changeset** or associated **via pull requests**

---

### B. In Release Pipelines

In **classic release pipelines**, you can choose to link work items from the build stage or between environments.

#### Step-by-Step (Release Pipeline):

1. Go to **Pipelines > Releases**.
2. Click **Edit** your release pipeline.
3. Go to the **Pre-deployment conditions** of a stage.
4. Under **Artifacts**, make sure the source build pipeline is linked.
5. In the **Options tab**, make sure **"Report deployment status to work items"** is checked.

> This will update work items with release deployment status (e.g., “deployed to staging”).

---

## ✅ 2. Enable Work Item Association in YAML Pipelines

Even in YAML, work items are automatically associated when:

* Commits are linked to work items using `#ID` syntax
* PRs are linked to work items manually or using keywords like:

  * `Fixes #123`
  * `Resolves AB#456` (Azure Boards ID)
  * `Closes AB#789`

Example:

```bash
git commit -m "Added validation to form, fixes AB#101"
```

This links work item `AB#101` to the build run.

---

## ✅ 3. Use Work Items in Release Notes

You can extract work items in your pipeline release notes.

In Classic Releases:

1. In the **Release Stage** → **Post-deployment conditions**
2. Click **Add task** → **Work items** → Use the **Generate Release Notes** extension
3. This fetches associated work items from the pipeline

You can also use `Generate Release Notes (modern)` task from [Azure DevOps Marketplace](https://marketplace.visualstudio.com/items?itemName=richardfennellBM.BM-VSTS-XplatGenerateReleaseNotes)

---

## ✅ 4. View Work Items in Build/Release Summary

After a build or release runs:

* Go to the **Pipeline run summary**
* Click the **"Work Items"** tab

You’ll see all automatically or manually linked work items.

---

