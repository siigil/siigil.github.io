---
layout: post
title:  "Becoming a Stratus Red Team Contributor"
date:   2024-10-17 09:30:00 -0400
categories: technical other
image: /assets/img/stratus-contributor/stratus-contributor-header.png
---

*This post documents the process to create and test a new attack technique for Stratus Red Team, a threat emulation tool built in Terraform and Go.*

## Introduction
I recently had the opportunity to contribute to [Stratus Red Team](https://github.com/DataDog/stratus-red-team) as a part of my research into [Entra ID administrative units](https://securitylabs.datadoghq.com/articles/abusing-entra-id-administrative-units/). 

I've found open-source contributions can feel daunting if you haven't been through them before. Based on that, my hope is that this guide can be a friendly nudge for those new to contributing to projects.

The following guide is meant as both:
- A direct reference how to contribute to Stratus Red Team
- An example of what contributing to open-source can look like

The rest of this post will be a fairly intensive walkthrough of this process.

If you're already familiar with Stratus and open-source contributions, you can skip ahead to "[#4. Creating a new attack technique](/posts/stratus-contributor/#4-creating-a-new-attack-technique)".

## 1. Get familiar with Stratus Red Team
Details on Stratus Red Team (a.k.a. "Stratus" for the rest of this post) can be found in its GitHub repository and website:
- [https://github.com/DataDog/stratus-red-team](https://github.com/DataDog/stratus-red-team)
- [https://stratus-red-team.cloud/user-guide/getting-started/](https://stratus-red-team.cloud/user-guide/getting-started/)

Review details on Stratus states (under the documentation's "[Getting Started](https://stratus-red-team.cloud/user-guide/getting-started//)" and "[Examples](https://stratus-red-team.cloud/user-guide/examples/)") sections, as these cover a full example of using Stratus to emulate a technique.

At this stage, it's also a good idea to run Stratus against your test environment if you haven't already. If you don't already have a test environment, create an account in your cloud(s) of choice:
- [Azure](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account)
- [AWS](https://aws.amazon.com/getting-started/guides/setup-environment/module-one/)
- [GCP](https://www.chrisfarris.com/post/gcp-create-domain/)

You can follow the [Stratus intro blog](https://securitylabs.datadoghq.com/articles/cyber-attack-simulation-with-stratus-red-team/) to set up, review, and detonate techniques of interest.

## 2. Project structure
Now that we've seen Stratus in action, let's review the states of Stratus in the context of what code is used to run them:
- **Warm Up:** *Terraform.* Create attack prerequisites. 
- **Detonate:** *Go.* Execute the attack.
- **Revert:** *Go.* Undo the attack so it can be detonated again.
- **Clean Up:** *Terraform.* Destroy attack prerequisites.

Individual techniques are stored under `https://github.com/DataDog/stratus-red-team/tree/main/v2/internal/attacktechniques`, organized by platform and attack type (e.g., `persistence`, `execution`, etc) folders.

For example, looking at the path for "[vm-custom-script-extension](https://github.com/DataDog/stratus-red-team/tree/main/v2/internal/attacktechniques/azure/execution/vm-custom-script-extension)":
![Stratus Structure](/assets/img/stratus-contributor/6-stratus-structure.png)
- `azure` folder: All techniques for `azure`.
- `execution` folder: All [MITRE ATT&CK Tactics](https://attack.mitre.org/tactics/enterprise/) of the `execution` category.
- `vm-custom-script-extension` folder: The folder for this specific technique.
- `main.go`: Go code containing functions required to initialize (`init`), `detonate`, and `revert` this technique.
- `main.tf`: Terraform code containing all prerequisite resources used in `warmup` and `cleanup` stages.

## 3. Development prerequisites
### Forks
Before starting any work, we'll create a fork of Stratus. GitHub's documentation on forks is very clear and helpful for those new to them. If you're new to this, I recommend reviewing this guide:
- "[Working with forks: Fork a repository](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo#forking-a-repository)"

This process can be started from the "Fork" menu in the Stratus repository:
<img src="/assets/img/stratus-contributor/1-stratus-fork.png" width="500">

Creating your fork through Github will allow you to easily sync new upstream changes from Stratus into your fork as you're working.

Once you've created your fork, create a local copy:

`git clone https://github.com/[your-username]/stratus-red-team`

### Developer's notes
You'll also want to read the project's own notes on contribution and philosophy:
- [Stratus Red Team: Philosophy](https://stratus-red-team.cloud/attack-techniques/philosophy/)
- [Stratus Red Team: Contributing](https://stratus-red-team.cloud/contributing/)

This is important to ensure any techniques you contribute meet the project's standards and are likely to be accepted. In particular, Stratus values techniques being as granular as possible. This means PRs that submit one technique at a time.

### Issues
Project issues are where suggestions on new techniques and improvements are stored:
- [Stratus Issues](https://github.com/DataDog/stratus-red-team/issues)

Tags within this list can help you find good ideas to start with. For example:
- [Stratus: "good first issue"](https://github.com/DataDog/stratus-red-team/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)
- [Stratus: "azure"](https://github.com/DataDog/stratus-red-team/issues?q=is%3Aissue+is%3Aopen+label%3Aplatform%2Fazure)

### Planning
Choose a technique to try out. The issues above are a great place to start!

Define for the selected technique:
1. **Warmup/Clean Up:** What resources or configurations need to exist as prerequisites for an attack (for an attacker to exploit)? These will be deployed in Terraform.
2. **Detonate:** What specific actions does the attack consist of? These actions will be performed in Go.
3. **Revert:** How can the attack be undone? For example: If an attacker created an external link to share data, then un-sharing the link would be the revert step. These actions will be performed in Go.

You'll need these notes for development, as well as documentation of the technique.

## 4. Creating a new attack technique

### Configure a new technique
Let's start by creating a new "placeholder technique" to build on in later steps, and ensuring it builds into Stratus. Perform these steps in your development environment of choice (e.g. Visual Studio Code, command line, whatever suits you).

1. Create a new branch off of your fork: `git checkout -b [branch-name]`
2. Create a new folder for your technique under `/main/v2/internal/attacktechniques/[cloud]/[attack-type]/[technique-name]`. It's best to pick a name that's action-oriented, and follows the example of current techniques.
3. For now, copy an existing technique's `main.go` and `main.tf` files into this folder to act as placeholders to be modified into your new technique. Your final structure should look something like the below:
    <img src="/assets/img/stratus-contributor/2-stratus-structure.png" width="400">
4. Under `init()` in your new technique's `main.go`, modify the `ID`, `FriendlyName`, `Platform` and `MitreAttackTactics` to match your new technique's name and MITRE ATT&CK Tactic (category):
    <img src="/assets/img/stratus-contributor/3-stratus-init.png" width="600">
    <img src="/assets/img/stratus-contributor/4-stratus-tactic.png" width="600">
5. In `/main/v2/internal/attacktechniques/main.go`, add the path of your new technique: `"github.com/datadog/stratus-red-team/v2/internal/attacktechniques/[your-path]"`. This creates a reference to your new technique, so the main code of Stratus can find it:
    <img src="/assets/img/stratus-contributor/5-stratus-imports.png" width="800">
6. In your terminal, `cd` to the root project folder (`stratus-red-team`) and run `make`. This will build a new version of Stratus, with your new technique included.
7. Run `./bin/stratus status` and ensure your new technique shows up in the list. (`./bin/stratus` will ensure the locally built version of Stratus is in use, and not another version.)

At this stage, you should have all the files you need in place to begin developing a new attack technique! Now we need to update it to do something new.

There are two major files that require updating: `main.tf`, and `main.go`. We'll split this out by stage to make it simpler to effectively create our attack technique.

### Create Terraform prerequisites (warmup)
Creating initial infrastructure in Terraform means that we can easily test our `main.tf` outside of the Stratus project code. This will focus our efforts on *just* prerequisites prior to integrating the code back into the attack technique.

1. Create a `testing` folder *outside* of your Stratus project folder, and copy `main.tf` into it.
2. Update the `main.tf` file to include the prerequisites you've identified:
	- If you're newer to Terraform, run the original code first and play with it to get a feel for how this works.
	- Terraform publishes clear documentation for its providers, such as [azuread](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs) and [azurerm](https://registry.terraform.io/providers/hashicorp/azuread/latest/docs). These docs should always be your first stop to figure out how things work.
3. Run `terraform init` + `terraform apply` to test your infrastructure out.
4. Ensure naming of techniques, resources, and accounts align with those of other techniques. This is a great opportunity to review similar techniques, and see how they've handled this.
5. Include an `output` block for any variables that will be needed in your `detonate` and `revert` stages. Example: [Outputs for an Entra ID technique](https://github.com/DataDog/stratus-red-team/blob/main/v2/internal/attacktechniques/entra-id/persistence/restricted-au/main.tf#L49).
6. Once you're comfortable with your code, remember to clean up with `terraform destroy`.
7. Run `terraform fmt -recursive` to ensure formatting is clean. This is a requirement for Stratus contributions.

When you're confident that your `warmup` stage is ready to go, copy it back into the `[technique-name]` directory you've created (`/main/v2/internal/attacktechniques/[cloud]/[attack-type]/[technique-name]`).

### Create technique in Go (detonate, revert)
1. Modify `main.go` to create (`detonate`) and `revert` any attack infrastructure and commands.
	- If you're newer to Go, take time to review patterns and methods already in use elsewhere in Stratus. How have similar problems been solved in the codebase? Does your code align with their formatting?
	- Update messaging and error handling to ensure the technique will be clear and well-handled.
	- Review previous techniques for your target platform to see what SDKs they use, and documentation for those packages. (Example: MS Graph APIs have a tab for each SDK, such as this one on [Get Users](https://learn.microsoft.com/en-us/graph/api/user-get?view=graph-rest-1.0&tabs=go).)
	- Ensure [`IsIdempotent`](https://stratus-red-team.cloud/user-guide/commands/revert/) and `IsSlow` (if the technique takes a long time) align with your technique.
2. **From the directory `stratus-red-team/v2`**, run `go get [path]@[version]`  for any additional dependencies to ensure `go.mod` and `go.sum` are up to date.

### Update documentation (init, .md)
1. Fully update `init()` in `main.go` to match your technique's category, name, friendly name, description, detection, and other relevant fields.
2. Run `make docs` to automatically add documentation to `stratus-red-team/docs/attack-techniques/[platform]` based on your technique's documentation in `main.go`.

## 5. Testing your technique
### Build and run
1. **From the directory `stratus-red-team`**, run `make`  to debug and compile your version of Stratus.
2. Run `./bin/stratus detonate [technique-name]` from `stratus-red-team` to ensure your technique runs.
3. Repeat these steps as needed until it's working as expected!
4. Remember to `cleanup` your test attacks, especially if you plan on making any modifications to your `main.tf` file.
	- This is better than manually destroying any state or resources, which can mess with Terraform.
	- If you need to manually reset your technique's Terraform state, it can be found under `$HOME/.stratus-red-team/[technique-name]/`.

### Check your work
1. Read over everything you've changed in `main.tf`, `main.go`, and your technique documentation. Some common checks I make are:
	- Is anything left over from a previous attack technique? Descriptions, environment variables, and other details should all be updated to your new technique.
	- Did you include all your reference links?
	- Is your warm-up stage just prerequisites?
	- Is your detonate stage just attacker activities?
	- Does the full attack flow work as expected? (e.g. `warmup`, `detonate`, `revert`, `cleanup`?)
	- Did you accidentally change any irrelevant files? (Discard changes on them.)
	- Are you consistently using the same methods for output and warnings? Do they line up with other Stratus methods?
2. Ensure your work is up to date with the main branch:
	* Commit and push your work to your fork's branch.
	* In the Github page for your fork, at the top of the page, does your fork show as up to date with `DataDog/stratus-red-team:main`? Use "sync fork" option if it is not up to date, and resolve any conflicts.
	* Remember to `git pull` your remote sync to your local system before making any more changes!
![Update Fork](/assets/img/stratus-contributor/7-stratus-sync-fork.png)

## 6. PR process
### Make your request
When you're ready to go, the Github page for your fork will guide you through opening a pull request (PR) against the main project.

You can start this process by using "Contribute," then "Open pull request":
![Stratus PR](/assets/img/stratus-contributor/8-stratus-pr.png)

Fill out details of your request. Include any background someone would need. For example:
- Background on why you've implemented the technique
- Open issues the technique addresses
- Details on why you've chosen certain actions for the `warmup` or `detonate` stages

![PR Description](/assets/img/stratus-contributor/9-stratus-description.png)

### Respond to feedback
You'll likely receive some comments from the project maintainer to understand your contribution and ensure it's up to the project's standards. This is completely normal!

Be open to feedback. Try to keep in mind that while your work is a valued contribution, it's still a contribution to someone else's codebase. Their preferences and standards should come first in how your work is structured.

Your PR will automatically update to include any new contributions you make based on feedback.

### Merge!
Once approved, you should be ready to merge in your change! At this stage, the process is complete. This is a great time to capture any notes on what you've learned for next time.

## Conclusion
We've now covered the full process of contributing a new attack technique to Stratus Red Team! While this post was specific to Stratus, the overall workflow of understanding first the tool, then the project, and finally looking to mirror its codebase in a new contribution can be applied to nearly any project.

I hope you found this helpful! If you have any questions, feel free to reach out.

<meta name="twitter:card" content="summary" />
<meta name="twitter:site" content="@_sigil" />
<meta name="twitter:creator" content="@_sigil" />
<meta property="og:url" content="https://kknowl.es/posts/stratus-contributor/" />
<meta property="og:title" content="Becoming a Stratus Red Team Contributor" />
<meta property="og:description" content="I recently had the opportunity to contribute to Stratus Red Team as a part of my research into Entra ID administrative units. Open-source contributions can feel daunting if you haven’t been through them before..." />
<meta property="og:image" content="https://kknowl.es/assets/img/stratus-contributor/stratus-contributor-header.png" />