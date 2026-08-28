<div align="center">
<img src="./assets/header_esp.png" alt="GitHub Copilot - Adoption Program" width="100%" />

# ⚙️ Workshop 3 — GitHub Copilot CLI: Automation, Security, and End-to-End Workflow

**Hands-on with `copilot -p`, GitHub Actions, `/fleet`, automated analysis, and an end-to-end feature**

**⏱️ Total duration: 90 minutes**

</div>

---

This workshop turns the automation, security, and delivery content into a 90-minute execution-focused session. Participants use **Copilot CLI in headless mode**, integrate it into a GitHub Actions workflow, try parallel execution with `/fleet`, build an automated review script, and finish with an end-to-end activity on an Express API.

> [!IMPORTANT]
> The theoretical security topics are turned into concrete controls applied during the labs. The goal is to practice secure behaviors, not repeat the conceptual slides.

---

## 📋 Table of Contents

1. [🎯 Learning objectives](#-learning-objectives)
2. [⏱️ Estimated duration](#️-estimated-duration)
3. [📚 Prerequisites](#-prerequisites)
4. [🤖 Step 1. Practice `copilot -p` in headless mode](#-step-1-practice-copilot--p-in-headless-mode)
5. [🔁 Step 2. Create a Daily AI Summary with GitHub Actions](#-step-2-create-a-daily-ai-summary-with-github-actions)
6. [🧠 Step 3. Parallelize work with `/fleet`](#-step-3-parallelize-work-with-fleet)
7. [🔍 Step 4. LAB 4 — Automated analysis script](#-step-4-lab-4--automated-analysis-script)
8. [🚀 Step 5. End-to-end challenge — add Swagger to the API](#-step-5-end-to-end-challenge--add-swagger-to-the-api)
9. [🔐 Step 6. Hardening and security review](#-step-6-hardening-and-security-review)
10. [✅ Completion checklist](#-completion-checklist)
11. [🧭 Final reflections](#-final-reflections)

---

## 🎯 Learning objectives

By the end of this workshop, you will be able to:

- ✅ Run GitHub Copilot CLI non-interactively with `copilot -p`.
- ✅ Apply `--no-ask-user`, `--silent`, `--output-format`, `--allow-tool`, and `--deny-tool` according to the type of automation.
- ✅ Integrate a headless prompt into a GitHub Actions workflow.
- ✅ Understand when a task has natural parallelism and use `/fleet` to dispatch subagents.
- ✅ Monitor `/tasks` and `/usage` during a multi-agent execution.
- ✅ Build a `scripts/ai-review.sh` script for security, cyclomatic complexity, and tests.
- ✅ Run an end-to-end workflow based on `/plan → review → Autopilot → validation`.
- ✅ Apply security controls before delegating tasks in sensitive repositories.

---

## ⏱️ Estimated duration

| Block | Time |
| --- | ---: |
| 🤖 `copilot -p` and essential flags | 10 min |
| 🔁 Daily AI Summary in GitHub Actions | 15 min |
| 🧠 `/fleet` and subagent monitoring | 15 min |
| 🔍 LAB 4 — `ai-review.sh` | 20 min |
| 🚀 End-to-end challenge with Swagger | 20 min |
| 🔐 Hardening and security checklist | 10 min |
| **Total** | **90 min** |

> [!NOTE]
> The timing adapts an original session with demos and discussion to a lab format. The priority is for each participant to run each work pattern at least once.

---

## 📚 Prerequisites

Before you begin:

- GitHub Copilot CLI installed and authenticated.
- The workshop repository cloned locally.
- Access to Git and the project's tests.
- Access to GitHub Actions for the practice repository.
- For the headless exercise in Actions: a **fine-grained PAT** with the permission specified in the material (`Copilot Requests`) and the ability to store it as a repository secret.
- `AGENTS.md` available if you completed the previous workshop.

> [!WARNING]
> Use only credentials and permissions authorized for the workshop. Do not copy tokens into version-controlled files.

---

## 🤖 Step 1. Practice `copilot -p` in headless mode

**⏱️ 10 minutes**

**Objective:** run a one-shot task without opening an interactive session.

### 1.1 First headless command

From the repository root:

```bash
copilot -p "Review vulnerabilities in src/"
```

Notice that the command runs the prompt and exits.

### 1.2 Avoid interactive waits

Try:

```bash
copilot -p "Review vulnerabilities in src/" --no-ask-user
```

> [!IMPORTANT]
> In the material, `--no-ask-user` is critical for CI/CD: a process without a TTY must not wait for a human response.

### 1.3 Compare formats and permissions

Examples from the material:

```bash
copilot -p "task" --silent
```

```bash
copilot -p "task" --output-format=json
```

```bash
copilot -p "task" --allow-tool='shell(git:*)' --allow-tool='read'
```

```bash
copilot -p "task" --deny-tool='shell(rm)' --deny-tool='git(push)'
```

### Checkpoint

- [ ] Ran a headless prompt.
- [ ] Tried `--no-ask-user`.
- [ ] Identified at least one tool that should be allowed and one that should be denied for your use case.

---

## 🔁 Step 2. Create a Daily AI Summary with GitHub Actions

**⏱️ 15 minutes**

**Objective:** run Copilot CLI inside a runner to summarize repository changes.

### 2.1 Create the workflow

Create:

```text
.github/workflows/daily-summary.yml
```

Use the workflow shown in the slides as a starting point:

```yaml
name: Daily AI Summary

on:
  schedule:
    - cron: '30 17 * * *'

jobs:
  summary:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - run: npm install -g @github/copilot

      - env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.PAT }}
        run: |
          copilot -p "Summarize today's changes and write them to summary.md" \
            --allow-tool='shell(git:*)' \
            --allow-tool=write \
            --no-ask-user
```

### 2.2 Configure the secret

Create a secret in the repository named:

```text
PAT
```

Its value must be the fine-grained PAT authorized for the lab.

### 2.3 Run it during the workshop

The workflow in the slide uses a `schedule`. The transcript also mentions launching it manually through a dispatcher.

To complete the exercise within the timebox, the instructor can enable an equivalent manual trigger in the workflow or trigger the run using the authorized mechanism in the repository.

> [!NOTE]
> The material describes `summary.md` as an output/artifact, but the available slide does not include the concrete artifact-publishing step. This workshop does not invent that step: first validate that `summary.md` is generated during the job, then apply your organization's artifact standard if you need to persist it.

### Expected result

The job should:

1. Check out the repository.
2. Install Copilot CLI.
3. Run the prompt in headless mode using the secret.
4. Generate `summary.md` in the runner workspace.

### Checkpoint

- [ ] Workflow created.
- [ ] `PAT` secret configured.
- [ ] The job ran `copilot -p` or was prepared up to the environment's permission boundary.
- [ ] Identified where to validate the agent's output.

---

## 🧠 Step 3. Parallelize work with `/fleet`

**⏱️ 15 minutes**

**Objective:** split a task into independent tracks and observe how the main agent orchestrates subagents.

### 3.1 Plan before parallelizing

The material recommends this sequence:

```text
/plan → review the plan → verify natural parallelism → /fleet
```

Choose a task that can be divided without every track modifying the same file.

### 3.2 Run the feature example

The material proposes:

```text
/fleet Implement the notifications module:
- Track 1: Node.js service with SendGrid
- Track 2: unit tests with mocks
- Track 3: API documentation in docs/notifications.md
```

If the repository already contains part of this functionality, adapt the exercise to another feature of the same type as long as the tracks remain independent.

### 3.3 Monitor subagents

Run:

```text
/tasks
```

Then:

```text
/usage
```

Observe:

- How many subagents were dispatched.
- What task each one is performing.
- Which tasks had to wait for dependencies.
- The usage reported by the session.

### Expected result

The main agent acts as an orchestrator and separates work that can be performed in parallel.

> [!WARNING]
> The material highlights that there is no file locking between subagents. If two tracks modify the same file, conflicts may appear when they converge. Choose genuinely independent tracks.

> [!TIP]
> If `/fleet` does not identify the parallelism correctly, explicitly ask it: `Break this down into independent tracks, then run them in parallel`.

---

## 🔍 Step 4. LAB 4 — Automated analysis script

**⏱️ 20 minutes**

**Objective:** create an executable script that uses Copilot in headless mode to automate three repository reviews.

The material defines three required tasks:

```text
1. Security analysis.
2. Detection of functions with high cyclomatic complexity.
3. Verification that the tests pass.
```

### 4.1 Create the file

```bash
mkdir -p scripts
touch scripts/ai-review.sh
```

On Windows, you can create the file from VS Code or from the equivalent terminal available to you.

### 4.2 Ask the agent to build the script

In Copilot CLI, use a prompt equivalent to:

```text
Create scripts/ai-review.sh to run an automated review of this repository using GitHub Copilot CLI in headless mode.

The script must perform the following, in this order:
1. A security analysis without allowing writes and without requesting interactive input.
2. An analysis to detect functions with high cyclomatic complexity.
3. A verification that the project's tests pass, allowing the shell tool when necessary.

At the end, it must leave a readable summary with the results of all three checks.
Do not add tasks other than these three.
```

### 4.3 Verify the required controls

Check that the security section uses the flags required by the slide:

```text
--deny-tool='write'
--no-ask-user
```

And that the test execution uses the specified control:

```text
--allow-tool='shell'
```

> [!NOTE]
> The material defines the objective and flags, but does not provide complete canonical content for `ai-review.sh`. Therefore, participants must generate and review the script with the agent instead of copying a prefabricated solution.

### 4.4 Run it

```bash
chmod +x scripts/ai-review.sh
./scripts/ai-review.sh
```

### 4.5 Review the summary

Confirm that you can clearly distinguish the results for:

- Security.
- Cyclomatic complexity.
- Tests.

### Bonus

If the result identifies independent remediation tasks, try using `/fleet` to dispatch them in parallel.

### Checkpoint

- [ ] `scripts/ai-review.sh` created.
- [ ] Security analysis ran without writes.
- [ ] Cyclomatic complexity analyzed.
- [ ] Tests ran.
- [ ] Summary reviewed.

---

## 🚀 Step 5. End-to-end challenge — add Swagger to the API

**⏱️ 20 minutes**

**Objective:** run a complete planning, implementation, validation, and documentation workflow on the workshop's Express API.

The transcript uses **adding Swagger/OpenAPI** to the application as a realistic feature.

### 5.1 Ask for the plan first

Enter Plan Mode and request an implementation equivalent to:

```text
Analyze this Express API and create a plan to add Swagger/OpenAPI.
The plan must cover the necessary dependencies, the OpenAPI module/configuration, documentation for the existing routes, and validation that the application and tests continue to work.
It must also update AGENTS.md so that new endpoints/controllers maintain the corresponding Swagger documentation.
Do not implement anything until I review the plan.
```

### 5.2 Review the scope

Verify that the plan includes the elements observed in the demo:

- Required Swagger dependencies.
- OpenAPI configuration or module.
- Route/controller documentation.
- Endpoint wiring.
- Build/test validation.
- An update to `AGENTS.md`.

### 5.3 Accept the plan and run it

Once you have reviewed the scope, allow the implementation and, if the practice environment permits, use Autopilot to complete the task within the timebox.

### 5.4 Validate

Check:

- Successful build.
- Tests passing, with pre-existing problems clearly distinguished.
- Application running locally.
- Swagger/OpenAPI accessible at the port/configuration generated for your instance.

### 5.5 Validate AGENTS.md

Review that the added rule is consistent with the lab's objective and does not contradict the repository's existing conventions.

### 5.6 Prepare a PR

Ask the agent to prepare a PR with a description based on the change made.

### Lab evidence

The original material asks participants to provide a **screenshot of Swagger running locally**. Keep the same evidence for this workshop.

### Checkpoint

- [ ] Plan reviewed before implementation.
- [ ] Swagger/OpenAPI added.
- [ ] Build/tests reviewed.
- [ ] Swagger visible locally.
- [ ] `AGENTS.md` updated.
- [ ] PR prepared or created.
- [ ] Screenshot captured.

---

## 🔐 Step 6. Hardening and security review

**⏱️ 10 minutes**

**Objective:** apply the risks from module M5 as an operational checklist to what you just built.

### 6.1 Verify the working directory

Before any session:

```bash
pwd
```

Confirm that Copilot is running inside the repository and not from `~`.

### 6.2 Review Autopilot

Ask yourself:

- Is this repository sensitive or production-related?
- Do I really need Autopilot?
- Should I use Normal or Plan mode and limit tools?

The material recommends avoiding unrestricted Autopilot in sensitive repositories.

### 6.3 Define tools that should be blocked

At a minimum, check whether you should deny:

```text
shell(rm)
git(push)
```

And any tool that could perform irreversible operations on infrastructure or data.

### 6.4 Review MCP

If MCP servers connected to databases or other services exist, review whether their scope can be limited to read-only operations when the lab does not require writes.

### 6.5 Review organization policies

If a message appears indicating that Copilot CLI is disabled for the organization, the material states that enablement depends on the GitHub Copilot policies managed by the organization.

### Hardening checklist

- [ ] Session launched from the correct directory.
- [ ] Autopilot used only where appropriate.
- [ ] Destructive tools blocked when unnecessary.
- [ ] MCP servers reviewed according to the principle of least privilege.
- [ ] Tokens stored as secrets and not in the repository.
- [ ] Organization policies respected.

---

## ✅ Completion checklist

- [ ] Ran `copilot -p` with `--no-ask-user`.
- [ ] Created or reviewed a GitHub Actions workflow with headless Copilot.
- [ ] Tried `/fleet` with independent tracks.
- [ ] Used `/tasks` and `/usage` to observe parallel execution.
- [ ] Generated and ran `scripts/ai-review.sh`.
- [ ] Reviewed security, complexity, and tests from the script.
- [ ] Completed the Swagger/OpenAPI challenge or reached a verifiable checkpoint within the timebox.
- [ ] Updated or validated `AGENTS.md`.
- [ ] Prepared a Pull Request for the end-to-end change.
- [ ] Applied the final security checklist.

---

## 🧭 Final reflections

- `copilot -p` makes it possible to move agentic capabilities from the interactive terminal into **non-interactive scripts and pipelines**.
- `--no-ask-user` prevents automation without a TTY from waiting for human input.
- `/fleet` provides speed only when the specification has **truly independent tracks**.
- A review script with Copilot can combine security analysis, complexity analysis, and test execution into a single automatable task.
- The **/plan → review scope → run → validate → PR** workflow maintains a human control step before delegating implementation.
- Security and governance must be applied to every execution: correct directory, least-privilege permissions, restricted tools, and credentials kept out of the code.

> [!IMPORTANT]
> Automation does not mean eliminating review. The more autonomous the workflow, the more important it is to define scope, tools, restrictions, and validation criteria correctly.

---
