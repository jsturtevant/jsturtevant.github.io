---
layout: post
title: Practical Ways AI Can Help You Today with Minimal Setup
date: "2026-01-19"
categories:
  - ai
  - productivity
---

There is a lot of hype around AI right now, and I was a skeptic for a long time as many close friends know. In the past it required precise prompts that were long-winded, and even then you didn't always get the result you wanted. During my early attempts I found that I could often just do the task manually faster than I could with AI. But I have been periodically trying new techniques and watching what others do. Recently, I've started to see the value in certain tasks.  

The goal of this post is to give you several practical, minimal-setup ways to get value from AI today. These examples use [VS Code](https://code.visualstudio.com/) with [Copilot](https://github.com/features/copilot), but many of the same concepts work for other coding agents. Since I've started using these techniques, I've become more optimistic about AI productivity. There are still unanswered questions (cost, ethics, repeatability, trust of output), but I can't deny some of the benefits are real.

## Running through tutorials

New projects often require stepping through complex commands with specific environment variables and configurations. When things go wrong, debugging with unfamiliar tools slows everything down.

This prompt helps me step through a project's getting started guide and handle unexpected debugging steps:  

```md
You are going to help me walk through this readme and get the project setup. Do not run any commands without asking. If you need environment variables ask. If we find any issues let's fix them so the readme ends up in better shape at the end. If there is a secret involved, don't ask me for it but tell me how to set it up myself.
```

> IMPORTANT: These days I mostly work with projects where secrets are no longer handled thanks to technology like [federated identities](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation) and [Azure Managed Identities](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview) but not all projects are that way.  Do NOT enter secrets if the AI agent asks you to, only fill in non-secret environment variables and manually set up secret environment variables.

While this might seem simple (why not just copy paste?), what is interesting about using this technique is that I learned I prefer asking the agent to generate complex command lines, which enables me to be more creative since I don't need to memorize some esoteric command structure. I no longer care if `-v` means verbose or version (this seriously drives me crazy, why can't we all just agree ;-) ) and it frees me up to do more interesting things.

## Generate bug reports

Filing bug reports is tedious when creating minimal reproductions and filling out templates takes time to do it well. AI is really good at creating scripts that can reproduce issues. The prompt I've been using goes something like:

```md
I found a bug in agent framework (https://github.com/microsoft/agent-framework/). Can you help me file a report? I would like to generate a minimal reproduction and a way to test it. Find and fill out the bug report in a concise manner using the GitHub issue template (usually located in https://github.com/org/project/tree/main/.github/ISSUE_TEMPLATE) from the project exactly. Present the markdown for the issue to me locally. The bug is: Durable agents using AzureOpenAIResponsesClient with tools fail on the second turn of a conversation. The call_id_to_id mapping is not restored from durable state.  
```

An example of a bug report filled out this way is https://github.com/microsoft/agent-framework/issues/3187. This took significantly less time than if I had needed to design a minimal reproduction myself.

## Fixing CI on my projects

We use [Dependabot](https://docs.github.com/en/code-security/dependabot) to bump dependencies in our projects.  Sometimes Dependabot fails and I have to go figure out what is going on. It's usually a syntax issue or breaking change that needs to be addressed.  I've been using ~[GitHub MCP server](https://github.com/github/github-mcp-server)~ gh cli as a tool to do this quickly via an agent.  (I recently learned that we should avoid MCP in this scenario to avoid context bloat)

```
Go look at the GitHub CI for this pull request https://github.com/hyperlight-dev/hyperlight/pull/1156 using the gh cli. Find out what is wrong and then come up with a plan to fix it and present several options to me. Make sure to identify why the job is failing, considering several options. Don't make any code changes until after I approve the plan to fix this issue.
```

An example of using this technique to fix a failing CI in action is https://github.com/hyperlight-dev/hyperlight/pull/1153 

## Using Slash Commands

So far I've been giving you the prompts and you would need to modify them for each session.  The next tip is to use slash commands to make these reusable.  Here is one I use for submitting commits in a PR.  These [prompt files](https://code.visualstudio.com/docs/copilot/customization/prompt-files) can be stored local to the project at `.github/prompts` or if you want to use them for all projects in VS Code, you can put it in your VS Code profile something like (`%APPDATA%\Code\User\profiles\prompts` or `$HOME/.config/Code/User/profiles`).

Invoke the slash command via: `/whycommitmessage`. Note that `cm` is my personal alias for `commit` (you might need to tweak it).

```
---
name: whyCommitMessage
description: Generate a why-focused git commit message in multi-line format.
argument-hint: The changes or context to generate a commit message for
---
Generate a git commit message for the current changes or discussion.

Requirements:
- make sure we are not on the main branch, create a branch if so
- Focus on **why** the change was made, not just what changed
- Explain the problem being solved or the motivation behind the change
- Use the format: `git cm "short summary" -m "detailed explanation"`
- The short summary should be concise and action-oriented (imperative mood)
- The detailed explanation should provide context about the reasoning, benefits, or problems addressed
- Do not include the changes or a list of changes

Step 1: output the "short summary" and "detailed explanation" and ALWAYS ask user to confirm it
step 2: execute the `git cm "short summary" -m "detailed explanation"`
step 3: Ask the user if they want to open a pr and if so push the code with "git push". Verify we are on a branch not main.
```

And as an additional example, We can turn the bug report prompt into a reusable slash prompt.  As you can see it can be more detailed and provide more context to the agent.  

Invoked with the slash command: `/bugreport https://github.com/microsoft/agent-framework/ Durable agents using AzureOpenAIResponsesClient with tools fail on the second turn of a conversation. The call_id_to_id mapping is not restored from durable state`

```
---
name: bugreport
description: File a bug report using a project's GitHub issue template with minimal reproduction
argument-hint: repo-url followed by bug description
---

Help me file a bug report for a GitHub project.

## Input Format
The user will provide:
1. **Repository URL**: The GitHub repository URL (e.g., `https://github.com/org/project`) otherwise use the project they are current in.
2. **Bug Description**: A description of the bug they encountered, otherwise use the context from the current conversation

## Workflow

### Step 1: Fetch the Issue Template
- Look for bug report templates in the repository at `.github/ISSUE_TEMPLATE/`
- Common template names: `issue.md`, `issue.lang.md`, etc
- If no template exists, use a standard bug report format

### Step 2: Analyze the Bug
- Understand the bug description provided
- Identify the component/feature affected
- Determine expected vs actual behavior

### Step 3: Generate Minimal Reproduction
- Create a minimal code example that reproduces the issue
- Include only the essential code needed to trigger the bug
- Add comments explaining each step
- Create a folder ./bugs/<bugname> to store the reproducible file 

### Step 4: Create Test Case
- Generate a test that demonstrates the bug
- The test should FAIL with the current behavior
- The test should PASS once the bug is fixed

### Step 5: Fill Out the Issue Template
- Use the EXACT template structure from the project
- Be concise but complete
- Include:
  - Clear title
  - Environment details (versions, OS, etc.)
  - Steps to reproduce
  - Expected behavior
  - Actual behavior
  - Minimal reproduction code
  - Test case (if appropriate)

### Step 6: Present the Issue
- Output the complete markdown for the GitHub issue in ./bugs/<bugname>/report.md
- Format it ready to copy/paste into GitHub

## Output Format
Present the final bug report as a markdown code block that can be directly pasted into GitHub Issues.
```

## Running Multiple Code Reviews

Code reviews require checking out branches and carefully examining changes. I now use agents to catch syntax issues and edge cases I might miss. VS Code and Copilot are unique since they can spawn sub-agents for different models. The real power comes from running multiple models in parallel—each catches different things. This is powerful since each model analyzes independently with a different set of abilities, and the results are summarized. Issues reported by multiple models are more likely to be real problems.

Invoked with the slash command: `/codereview`

```
---
name: codereview
agent: agent
---
look at the commits for this branch and run a code review on them in parallel agents using opus, gpt 2 codex and gemini then give the critical feedback and analysis of which bugs were reported across the agents. Only look at issues that are introduced in the changes for this branch. Include line numbers when reporting out.  
```

## Skills

While they are fairly new, I've been starting to explore what skills can do for me.  One I've created is for generating our Release PRs.  Creating [skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) can be done by hand, but why not create one with a skill itself?

Luckily Anthropic has a skill we can leverage.  Ask the agent to download and store this skill: 

```
can you download the skill at https://github.com/anthropics/skills/tree/main/skills/skill-creator and put it in my skill folder
```

This should end up in `.github/skills/` folder for your project but it could also go in `~/.copilot/skills/` or `~/.claude/skills/`.

Next you can ask Copilot to use the newly downloaded skill to create a new skill. I did something like this:

```
Using skill-creator, add a release-prep skill to implement #file:how-to-make-releases.md.  The goal is to have a PR that looks like https://github.com/hyperlight-dev/hyperlight/pull/668 that incorporates the version updates and the output to the Changelog. You can skip steps required by the maintainer like creating tags and invoking the release CI steps.  
```

## A few last tips
I am not going to go into detail on these but if you found this useful you might want to try these out on your own:

- Using [plan mode](https://code.visualstudio.com/docs/copilot/chat/copilot-chat#_planning-mode) - This added step changed the output on more complicated tasks that I've asked the agent to accomplish. While not a silver bullet, it improved the quality significantly.
- [Custom Agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents) - These have been useful in certain scenarios but not as helpful as I've hoped.  I am still trying to figure out the right use cases. The community maintains a set of these: https://github.com/github/awesome-copilot/tree/main/agents
- Asking for ASCII art or [Mermaid](https://mermaid.js.org/) diagrams - Agents do surprisingly well at generating Mermaid diagrams, and I've found that for memory-related visualization, ASCII art can go a long way toward understanding. Just double check the output—sometimes it's not 100% right, but it often gets me started on what it might look like.

Hopefully this shares some practical tips for you to try out. If you have other tips or one of these helps out, please let me know.