# Assignment 5 — AI-Assisted Azure DevOps Dual-Pipeline Failure Triage

Part of the DevOps Micro Internship (DMI) — Agentic AI Track

---

## Student Information

**Full Name:** Anthonia Akwuohia

**GitHub Repository or Fork URL:** https://github.com/Tonia-onyeka

**Public LinkedIn Post URL:** https://www.linkedin.com/in/anthonia-akwuohia-5b00681b0/

---

## Purpose

In this assignment, I configured an AI-assisted, read-only failure-triage workflow for the EpicBook Infrastructure and Application Pipelines. The workflow uses Bash to gather Azure DevOps pipeline evidence and Claude Code to analyze the evidence and recommend a recovery action while keeping all changes under human control.

---

# Task 0 — Verify Tools, Authentication, and Pipeline Details

## Goal

Verify the required tools, Azure DevOps authentication, organization and project details, and numeric pipeline IDs.

No screenshot is required for this task.

---

# Task 1 — Capture the Healthy Baseline and Prepare the Supplied Files

## Goal

Confirm that both EpicBook pipelines are healthy and place the supplied assignment files in the correct repository locations.

## Evidence

### Screenshot 1 — Healthy Baseline for Both Pipelines

Terminal output showing the latest completed Infrastructure and Application Pipeline runs with successful results.

![Screenshot](screenshots/Ass5.Task1.ss1.png)

## Notes

### 1. What proves that both pipelines were healthy before the drill?

The healthy baseline is proven by the latest completed runs of both the Infrastructure Pipeline and the Application Pipeline showing successful results. The successful pipeline runs confirm that the existing infrastructure deployment and application deployment workflows were functioning correctly before the controlled failure was introduced.

### 2. Why is a healthy baseline necessary before introducing a controlled failure?

A healthy baseline establishes a known-good starting point for the drill. It ensures that any failure observed after the controlled change can be compared with the previous successful state and attributed to the intentional test rather than to an existing problem.

This makes it possible to measure whether the monitoring, detection, diagnosis, and recovery workflow responds correctly to the controlled failure.

---

# Task 2 — Configure and Review the Supplied CLAUDE.md

## Goal

Configure the supplied project context and verify the safety boundaries Claude must follow.

## Evidence

### Screenshot 2 — CLAUDE.md Context and Safety Rules

`CLAUDE.md` open in the editor with the Project Overview, Incident Workflow, Safety Rules, and Output Rules visible.

![Screenshot](screenshots/Ass5.Task2.ss2.png)

## Notes

### 1. Why does Claude need project-specific operational context?

Claude needs project-specific context so it understands the purpose of each EpicBook pipeline, the expected triage workflow, and the boundaries it must follow. This helps Claude interpret the evidence correctly without making assumptions about how the environment operates.

### 2. Which rules keep the human responsible for the recovery action?

The safety rules that prohibit Claude from automatically performing recovery actions keep the human responsible. In particular, Claude must not independently run destructive or recovery operations, modify production infrastructure, or approve changes without human review.

The workflow requires Claude to gather evidence, analyze the incident, and provide a recommendation, while the human operator reviews the evidence and decides whether the recovery action should be executed.

### 3. Which rules protect pipeline credentials and application secrets?

The rules requiring Claude to never expose, print, log, or include credentials, tokens, passwords, private keys, or other secrets in reports or output protect sensitive information.

Claude should also avoid displaying secret values when inspecting pipeline configuration or logs. If credentials are required by the pipeline, they should remain stored in the appropriate secure Azure DevOps or cloud secret-management mechanism rather than being written directly into scripts, reports, or configuration files.

---

# Task 3 — Configure and Validate the Supplied Pipeline Triage Script

## Goal

Configure the supplied Bash script and verify that it retrieves and classifies evidence from both Azure DevOps pipelines without modifying them.

## Evidence

### Screenshot 3 — Pipeline Triage Script Configuration

Editor showing the script configuration variables, report filenames, check-function array, and read-only log-retrieval functions. Ensure that no token is visible.

![Screenshot](screenshots/Ass5.Task3.ss3.png)

![Screenshot](screenshots/Ass5.Task3.ss3i.png)

---

### Screenshot 4 — Script Validation

Terminal showing successful Bash syntax validation and executable file permission.

![Screenshot](screenshots/Ass5.Task3.ss4.png)

## Notes

### 1. Why are pipeline metadata and step console logs handled separately?

Pipeline metadata and step console logs provide different types of evidence.

Pipeline metadata identifies information such as the pipeline run, status, result, and run details. Step console logs provide the detailed output produced during individual pipeline tasks.

Handling them separately allows the script to use metadata for the overall run classification while using console logs to investigate and identify specific failure evidence.

### 2. How does the script obtain the actual console logs?

The script first retrieves the pipeline run and its associated timeline or step information. It then uses the relevant Azure DevOps API information to retrieve the actual console log output for the required pipeline steps.

The retrieved logs are saved as evidence files so that the classification checks can inspect the real output instead of relying only on the pipeline's summary status.

### 3. How does the check-function array control the classification loop?

The check-function array contains the names of the evidence-check functions that the script should execute.

The classification loop iterates through the array and calls each function in sequence. This makes the triage process modular because individual checks can be added, removed, or updated without rewriting the main classification loop.

### 4. What prevents a failed but unmatched run from being reported as healthy?

The script distinguishes the pipeline's actual result from the specific failure classifications.

If a run has failed but none of the known check functions matches the available evidence, the script does not classify it as healthy. Instead, it reports the failure as an unmatched or unknown condition that requires further review.

This prevents the absence of a recognized failure pattern from being incorrectly interpreted as a successful run.

### 5. Why are different exit codes useful to another automation tool?

Different exit codes allow another automation tool to understand the result programmatically without having to parse the entire report.

For example, separate exit codes can indicate successful execution, a classified failure, an unmatched failure requiring investigation, or a script/API error. This allows downstream automation to make appropriate decisions based on the triage result.

---

# Task 4 — Run and Understand the Healthy-State Report

## Goal

Run the supplied script against the healthy baseline and verify the initial pipeline health report.

## Evidence

### Screenshot 5 — Healthy Pipeline Report

Healthy pipeline report showing your Full Name, both successful pipelines, Overall Status `HEALTHY`, and captured exit code `0`.

![Screenshot](screenshots/Ass5.Task4.ss5.png)

## Notes

### 1. What evidence proves that both pipelines are healthy?

The healthy baseline is proven by the latest completed Infrastructure and Application pipeline runs. Infrastructure Run 26 and Application Run 27 both show a completed status with a result of succeeded. The triage script also retrieved the console logs for both runs and found no dependency, build, test, authentication/authorization, agent availability, Terraform, deployment, or unclassified failure. The final report shows Overall Status: HEALTHY, with WARN: 0, FAIL: 0, and a script exit code of 0.

### 2. Why must the baseline exit code be verified before the incident drill?

The baseline exit code must be verified before the incident drill to establish that the environment is healthy before introducing the controlled failure. An exit code of 0 confirms that the triage script can successfully retrieve and classify the current pipeline evidence without detecting an existing failure or configuration problem. This provides a reliable comparison point for the later incident and helps distinguish the controlled failure from any pre-existing issue.

---

# Task 5 — Configure and Test the Supplied /pipeline-triage Skill

## Goal

Configure the supplied Claude Code skill and verify that it runs the Bash tool as a reusable, manually invoked workflow.

## Evidence

### Screenshot 6 — Pipeline-Triage Skill Definition

`SKILL.md` showing the frontmatter, manual-invocation setting, narrowly scoped tools, safety rules, and required output structure.

![Screenshot](screenshots/Ass5.Task5.ss6.png)


---

### Screenshot 7 — Healthy Skill Result

Healthy `/pipeline-triage` result showing that both pipelines are healthy and no fix is required.

![Screenshot](screenshots/Ass5.Task5.ss7.png)

![Screenshot](screenshots/Ass5.Task5.ss7i.png)

## Notes

### 1. Why is `disable-model-invocation: true` appropriate for this skill?

disable-model-invocation: true is appropriate because pipeline triage is an operational workflow that should only start when the engineer explicitly requests it. Manual invocation prevents the model from automatically deciding to run the triage workflow during unrelated work. This keeps the workflow predictable and preserves human control over when pipeline evidence is collected.

### 2. Why should the skill avoid broad Bash approval?

The skill should avoid broad Bash approval because unrestricted shell access would give Claude the ability to execute commands outside the intended read-only triage workflow. A narrowly scoped command reduces the risk of modifying files, changing infrastructure, running Terraform or Ansible, or performing pipeline mutations. The goal is to give Claude only the command execution capability required for evidence collection.

### 3. What work is performed by Bash, and what work is performed by Claude?

Bash performs the deterministic evidence-gathering work. It retrieves pipeline metadata and console logs, runs the configured checks, saves the evidence, and produces the initial classification.

Claude interprets the evidence produced by Bash. It explains the findings, identifies the relevant incident condition, distinguishes evidence from assumptions, and presents the result according to the skill's required output structure.

This separates deterministic evidence collection from AI-assisted analysis.

### 4. Why are permission rules required in addition to written safety instructions?

Permission rules provide a technical enforcement layer in addition to the written instructions. Written safety rules explain what Claude should and should not do, while permission restrictions can prevent dangerous tools or commands from being executed even if the model attempts to use them. Combining both layers reduces the risk of accidental file changes, infrastructure modifications, pipeline mutations, or unauthorized access to sensitive information.
---

# Task 6 — Introduce a Safe Failure in the Application Pipeline

## Goal

Create a controlled Application Pipeline failure that can be diagnosed without changing Azure infrastructure or production data.

## Evidence

### Screenshot 8 — Controlled Application Pipeline Failure

Failed Application Pipeline run showing the temporary branch, failed status, failed step, and relevant non-sensitive error evidence.

![Screenshot](screenshots/Ass5.Task6.ss8.png)

## Notes

### 1. What exact failure did you introduce?

I introduced a controlled failure in the Application Pipeline by temporarily modifying the pipeline on a separate test branch so that a designated test step returned a failure status.

The Application Pipeline was intentionally failed using a clearly labeled SAFE FAILURE DRILL step that exits with code 1.

### 2. Which category should detect it?

The failure should be detected as an Application Pipeline failure by the pipeline triage checks.

TFailure category: This is a validation/build-pipeline failure because the failure occurs directly in the Application Pipeline before any deployment action.

### 3. Why is the failure safe and easily reversible?

The failure was introduced only as a controlled test condition in the CI/CD pipeline. It did not require changes to the AWS infrastructure, production data, or deployed application.

The failure exists only on the temporary drill/pipeline-failure branch. It does not modify Terraform infrastructure, service connections, credentials, SSH keys, databases, or production data. Removing the temporary commit restores the normal pipeline behavior.

### 4. How did you prevent the deliberate failure from reaching `main` or changing the deployed application?

I introduced the failure on a temporary test branch rather than directly on main. The branch was used only to execute the controlled failure test.

The intentional failure was committed only to drill/pipeline-failure, which was never merged into main. The pipeline stops at the failing step, so no subsequent deployment action can execute.

---

# Task 7 — Diagnose and Save the Incident Evidence

## Goal

Use `/pipeline-triage` to classify the failed Application Pipeline without allowing Claude to apply the recovery action.

## Evidence

### Screenshot 9 — Failed-State Diagnosis and Incident Report

`/pipeline-triage` output and saved incident report showing the affected pipeline, failure category, sanitized evidence, recommendation, and your Full Name.

![Screenshot](screenshots/Ass5.Task7.ss9.png)

![Screenshot](screenshots/Ass5.Task7.ss9i.png)

## Notes

### 1. Which failure category was identified?

The /pipeline-triage workflow identified the failure as an Application Pipeline failure. The classification was based on the failed Application Pipeline run and the evidence retrieved from its pipeline metadata and step console logs.

### 2. What exact evidence supported the diagnosis?

The diagnosis was supported by the Application Pipeline run showing a failed result, together with the console output from the failed step containing the relevant non-sensitive error message.

The incident report preserved the affected pipeline, failed step, failure status, and sanitized error evidence. This provided direct evidence for the classification rather than relying on an assumption about the cause.

### 3. Did Claude apply the fix or rerun the pipeline? Why is that important?

No. Claude did not apply the fix or rerun the pipeline.

This is important because the /pipeline-triage skill is designed to gather and analyze evidence, not independently perform recovery actions. Keeping recovery under human control prevents an AI-assisted diagnosis from automatically causing additional pipeline executions or changes before the operator has reviewed the evidence and approved the appropriate action.

### 4. Which part represents Gather, and which part represents Analyze?

Gather: The pipeline-triage.sh script retrieved the latest Azure DevOps run information and sanitized application pipeline logs, producing pipeline-health-report.txt and app-last-run.log.

Analyze: Claude reviewed the generated report and sanitized log evidence, identified the failed Application Pipeline and reported failure category, and provided the information needed for a human to determine the recovery action.

Incident Evidence

The failed report was preserved before any recovery action:

reports/incident-failure-report.txt

It records Full Name, pipeline name, pipeline ID, Run ID 31, branch, failed result, failure category, sanitized evidence, and the overall FAIL status.

---

# Task 8 — Apply the Human-Reviewed Fix and Verify Recovery

## Goal

Apply the recommended fix manually and verify that the Application Pipeline and triage report return to a healthy state.

## Evidence

### Screenshot 10 — Corrected Application Pipeline Run

Corrected Application Pipeline run showing the temporary branch and successful status.

![Screenshot](screenshots/Ass5.Task8.ss10.png)

---

### Screenshot 11 — Recovery Triage Result

Recovery `/pipeline-triage` output showing Overall Status `HEALTHY`, exit code `0`, your Full Name, and both saved report filenames.

![Screenshot](screenshots/Ass5.Task8.ss11.png)

## Notes

### 1. What exact fix did you apply?

I manually reverted the temporary change that had been introduced to deliberately fail the Application Pipeline. The failing test condition was removed from the temporary branch, and the corrected pipeline configuration was committed and pushed to the same temporary branch.

The Application Pipeline was then run again to verify the correction.

### 2. Did the fix match Claude’s recommendation? Explain briefly.

Yes. The manual fix matched Claude's recommendation because it addressed the specific failure condition identified from the pipeline metadata and console-log evidence.

Claude provided the diagnosis and recommended recovery action, while I reviewed the evidence and performed the corrective change manually.

### 3. What evidence proves that the pipeline recovered?

The corrected Application Pipeline completed successfully on the temporary branch. In addition, the recovery /pipeline-triage run reported:

Overall Status: HEALTHY
Exit Code: 0

The recovery report also confirmed the successful results of both pipelines. These provide evidence that the Application Pipeline returned to the expected healthy state.

### 4. Why is a second triage run required after the pipeline becomes green?

A second triage run is required because a green pipeline run alone does not provide the complete incident-review evidence.

Running /pipeline-triage again confirms the state of both pipelines using the same evidence-gathering and classification process used during the incident. It verifies that the failure condition has actually been resolved and that the overall environment has returned to HEALTHY.

### 5. What risk would be created if Claude could automatically edit, push, approve, and rerun the pipeline?

Allowing Claude to automatically edit, push, approve, and rerun a pipeline would remove important human review points from the recovery process.

An incorrect diagnosis or unintended change could be automatically promoted through the CI/CD workflow, potentially causing application disruption, unauthorized changes, security issues, or deployment of an incorrect configuration.

Keeping the recovery action under human control ensures that the evidence and recommendation are reviewed before a change is committed or a pipeline is rerun.

---

# LinkedIn Post — Mandatory

## LinkedIn Post URL

https://www.linkedin.com/posts/anthonia-akwuohia-5b00681b0_devops-azuredevops-agenticai-share-7508506499359633408-9ewm/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADEhX1QBTHiW-kQPmKjn3MVixQzj4IzJO1Q

## Evidence

### Screenshot 12 — Published LinkedIn Post

Published LinkedIn post showing its text and at least one image or link.

![Screenshot](screenshots/Ass5Linkedin.png)

---

# Required Repository Files

Confirm that the following files are available in your repository:

* [ ] `CLAUDE.md`
* [ ] `pipeline-triage.sh`
* [ ] `.claude/skills/pipeline-triage/SKILL.md`
* [ ] `reports/incident-failure-report.txt`
* [ ] `reports/recovery-report.txt`

---

# Submission Instructions

* Complete all tasks in sequence.
* Include all 12 required screenshots.
* Answer every Notes question in your own words.
* Include your GitHub repository or fork URL.
* Include your public LinkedIn post URL.
* Ensure your Full Name appears in the required reports.
* Do not include raw logs containing sensitive information.
* Do not expose PATs, tokens, authorization headers, passwords, SSH keys, Service Connection credentials, or database credentials.

---

# Completion Checklist

* [ ] Both Azure DevOps pipelines were healthy before the drill.
* [ ] The supplied files were copied to the correct repository locations.
* [ ] Only the required student-specific placeholders were updated.
* [ ] `CLAUDE.md` contains the required context and safety rules.
* [ ] `pipeline-triage.sh` passed Bash syntax validation.
* [ ] The script has executable permission.
* [ ] The script uses read-only Azure DevOps operations.
* [ ] The script retrieves pipeline metadata and console logs.
* [ ] No token or password is stored in the script.
* [ ] The healthy baseline reported `HEALTHY` with exit code `0`.
* [ ] `/pipeline-triage` was invoked manually.
* [ ] The skill does not have broad Bash approval.
* [ ] The controlled failure affected only the Application Pipeline.
* [ ] The failure occurred before deployment changes were applied.
* [ ] The deliberate failure was not merged into `main`.
* [ ] The failed-state report was saved before applying the fix.
* [ ] Claude diagnosed the failure but did not apply the fix.
* [ ] The fix was reviewed and applied manually.
* [ ] The corrected Application Pipeline completed successfully.
* [ ] The recovery triage reported `HEALTHY` with exit code `0`.
* [ ] `incident-failure-report.txt` exists.
* [ ] `recovery-report.txt` exists.
* [ ] All Notes questions have been answered.
* [ ] All 12 screenshots have been added.
* [ ] The GitHub repository or fork URL has been included.
* [ ] The LinkedIn post is public.
* [ ] The LinkedIn post URL has been included.
* [ ] No sensitive information is exposed.

---

# Final Submission

**Full Name:** [Enter your full name]

**GitHub Repository or Fork URL:** [Paste your repository URL]

**LinkedIn Post URL:** [Paste your public LinkedIn post URL]

---

*This submission is part of the DevOps Micro Internship (DMI) — Agentic AI Track.*
