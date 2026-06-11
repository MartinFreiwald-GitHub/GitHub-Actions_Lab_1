Understanding GitHub Actions

Learn the basics of core concepts and essential terminology in GitHub Actions.


![components_pic](../GitHub%20Actions/pics/banneractions.jpg)


In this article

Overview

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production.

GitHub Actions goes beyond just DevOps and lets you run workflows when other events happen in your repository. For example, you can run a workflow to automatically add the appropriate labels whenever someone creates a new issue in your repository.

GitHub provides Linux, Windows, and macOS virtual machines to run your workflows, or you can host your own self-hosted runners in your own data center or cloud infrastructure.

The components of GitHub Actions

You can configure a GitHub Actions workflow to be triggered when an event occurs in your repository, such as a pull request being opened or an issue being created. Your workflow contains one or more jobs which can run in sequential order or in parallel. Each job will run inside its own virtual machine runner, or inside a container, and has one or more steps that either run a script that you define or run an action, which is a reusable extension that can simplify your workflow.


![components_pic](../GitHub%20Actions/pics/component.jpg)


Workflows

A workflow is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.

Workflows are defined in the .github/workflows directory in a repository. A repository can have multiple workflows, each of which can perform a different set of tasks such as:

Building and testing pull requests

Deploying your application every time a release is created

Adding a label whenever a new issue is opened

You can reference a workflow within another workflow. For more information, see Reuse workflows.
    
    Reuse workflows
    https://docs.github.com/en/actions/using-workflows/reusing-workflows

    Learn how to avoid duplication when creating a workflow by reusing existing workflows.

    In this article
    
    Creating a reusable workflow
    
    Reusable workflows are YAML-formatted files, very similar to any other workflow file. As with other workflow files, you locate reusable workflows in the .github/workflows directory of a repository. Subdirectories of the workflows directory are not supported.

    For a workflow to be reusable, the values for on must include workflow_call:

        on:
        workflow_call:

    Using inputs and secrets in a reusable workflow
    You can define inputs and secrets, which can be passed from the caller workflow and then used within the called workflow. There are three stages to using an input or a secret in a reusable workflow.

    1. In the reusable workflow, use the inputs and secrets keywords to define inputs or secrets that will be passed from a caller workflow.

    on:
    workflow_call:
        inputs:
        config-path:
            required: true
            type: string
        secrets:
        personal_access_token:
            required: true
    
    For details of the syntax for defining inputs and secrets, see **on.workflow_call.inputs** https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#onworkflow_callinputs
     and **on.workflow_call.secrets.** "both are in the link provided"

    2. In the reusable workflow, reference the input or secret that you defined in the on key in the previous step.

    Note

    If the secrets are inherited by using secrets: inherit in the calling workflow, you can reference them even if they are not explicitly defined in the on key. For more information, see Workflow syntax for GitHub Actions.

    jobs:
    reusable_workflow_job:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/labeler@v6
        with:
            repo-token: ${{ secrets.personal_access_token }}
            configuration-path: ${{ inputs.config-path }}
    
    In the example above, personal_access_token is a secret that's defined at the repository or organization level.

    Warning

    Environment secrets cannot be passed from the caller workflow as on.workflow_call does not support the environment keyword. If you include environment in the reusable workflow at the job level, the environment secret will be used, and not the secret passed from the caller workflow. For more information, see Managing environments for deployment  https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments#environment-secrets  and Workflow syntax for GitHub Actions.  https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idsecretsinherit
    "GraphGL API is in first link at bottom."

    3. Pass the input or secret from the caller workflow.

    To pass named inputs to a called workflow, use the with keyword in a job. Use the secrets keyword to pass named secrets. For inputs, the data type of the input value must match the type specified in the called workflow (either boolean, number, or string).

    jobs:
    
    call-workflow-passing-data:
        uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
        with:
        config-path: .github/labeler.yml
        secrets:
        personal_access_token: ${{ secrets.token }}
    
    Workflows that call reusable workflows in the same organization or enterprise can use the inherit keyword to implicitly pass the secrets.

    jobs:
    
    call-workflow-passing-data:
        uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
        with:
        config-path: .github/labeler.yml
        secrets: inherit
    
    Example reusable workflow
    
    This reusable workflow file named workflow-B.yml (we'll refer to this later in the example caller workflow) takes an input string and a secret from the caller workflow and uses them in an action.

    YAML
    name: Reusable workflow example

    on:
    
    workflow_call:
        inputs:
        config-path:
            required: true
            type: string
        secrets:
        token:
            required: true

    jobs:
    
    triage:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/labeler@v6
        with:
            repo-token: ${{ secrets.token }}
            configuration-path: ${{ inputs.config-path }}
    
    Calling a reusable workflow
    
    You call a reusable workflow by using the uses keyword. Unlike when you are using actions within a workflow, you call reusable workflows directly within a job, and not from within job steps.

    jobs.<job_id>.uses 
    
     see: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_iduses

    You reference reusable workflow files using one of the following syntaxes:

    {owner}/{repo}/.github/workflows/{filename}@{ref} for reusable workflows in public and private repositories.
    ./.github/workflows/{filename} for reusable workflows in the same repository.
    
    In the first option, {ref} can be a SHA, a release tag, or a branch name. If a release tag and a branch have the same name, the release tag takes precedence over the branch name. Using the commit SHA is the safest option for stability and security. For more information, see Secure use reference.
    https://docs.github.com/en/actions/reference/security/secure-use#reusing-third-party-workflows


    If you use the second syntax option (without {owner}/{repo} and @{ref}) the called workflow is from the same commit as the caller workflow. Ref prefixes such as refs/heads and refs/tags are not allowed. You cannot use contexts or expressions in this keyword.

    You can call multiple workflows, referencing each in a separate job.

    jobs:
    
    call-workflow-1-in-local-repo:
        uses: octo-org/this-repo/.github/workflows/workflow-1.yml@172239021f7ba04fe7327647b213799853a9eb89
    call-workflow-2-in-local-repo:
        uses: ./.github/workflows/workflow-2.yml
    call-workflow-in-another-repo:
        uses: octo-org/another-repo/.github/workflows/workflow.yml@v1
    
    Example caller workflow
   
    This workflow file calls two workflow files. The second of these, workflow-B.yml (shown in the example **reusable** **workflow**) https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows#example-reusable-workflow  , is passed an input (config-path) and a secret (token).

    YAML
    
    name: Call a reusable workflow

    on:
    
    pull_request:
        branches:
        - main

    jobs:
    
    call-workflow:
        uses: octo-org/example-repo/.github/workflows/workflow-A.yml@v1

    call-workflow-passing-data:
        permissions:
        contents: read
        pull-requests: write
        uses: octo-org/example-repo/.github/workflows/workflow-B.yml@main
        with:
        config-path: .github/labeler.yml
        secrets:
        token: ${{ secrets.GITHUB_TOKEN }}
    
    Passing inputs and secrets to a reusable workflow
    To pass named inputs to a called workflow, use the with keyword in a job. Use the secrets keyword to pass named secrets. For inputs, the data type of the input value must match the type specified in the called workflow (either boolean, number, or string).

    jobs:
    
    call-workflow-passing-data:
        uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
        with:
        config-path: .github/labeler.yml
        secrets:
        personal_access_token: ${{ secrets.token }}
    
    Workflows that call reusable workflows in the same organization or enterprise can use the inherit keyword to implicitly pass the secrets.

    jobs:
    
    call-workflow-passing-data:
        uses: octo-org/example-repo/.github/workflows/reusable-workflow.yml@main
        with:
        config-path: .github/labeler.yml
        secrets: inherit
    
    Using a matrix strategy with a reusable workflow
    Jobs using the matrix strategy can call a reusable workflow.

    A matrix strategy lets you use variables in a single job definition to automatically create multiple job runs that are based on the combinations of the variables. For example, you can use a matrix strategy to pass different inputs to a reusable workflow. For more information about matrices, see Running variations of jobs in a workflow.  https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/run-job-variations

    This example job below calls a reusable workflow and references the matrix context by defining the variable target with the values [dev, stage, prod]. It will run three jobs, one for each value in the variable.

    YAML
    
    jobs:
    ReusableMatrixJobForDeployment:
        strategy:
        matrix:
            target: [dev, stage, prod]
        uses: octocat/octo-repo/.github/workflows/deployment.yml@main
        with:
        target: ${{ matrix.target }}
   
    Nesting reusable workflows
   
    You can connect a maximum of ten levels of workflows - that is, the top-level caller workflow and up to nine levels of reusable workflows. For example: caller-workflow.yml → called-workflow-1.yml → called-workflow-2.yml → called-workflow-3.yml → ... → called-workflow-9.yml.

    Loops in the workflow tree are not permitted.

    Note

    Nested reusable workflows require all workflows in the chain to be accessible to the caller, and permissions can only be maintained or reduced—not elevated—throughout the chain. For more information, see Reusing workflow configurations.

    From within a reusable workflow you can call another reusable workflow.

    YAML
    
    name: Reusable workflow

    on:
    
    workflow_call:

    jobs:
    
    call-another-reusable:
        uses: octo-org/example-repo/.github/workflows/another-reusable.yml@v1
    
    Passing secrets to nested workflows
    
    You can use **jobs.<job_id>.secrets** in a calling workflow to pass named secrets to a directly called workflow. Alternatively, you can use **jobs.<job_id>.secrets.inherit** to pass all of the calling workflow's secrets to a directly called workflow. For more information, see the section Reuse workflows above, and the reference article Workflow syntax for GitHub Actions  https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idsecretsinherit  
    . Secrets are only passed to directly called workflow, so in the workflow chain A > B > C, workflow C will only receive secrets from A if they have been passed from A to B, and then from B to C.

    In the following example, workflow A passes all of its secrets to workflow B, by using the inherit keyword, but workflow B only passes one secret to workflow C. Any of the other secrets passed to workflow B are not available to workflow C.

    jobs:
    
    workflowA-calls-workflowB:
        uses: octo-org/example-repo/.github/workflows/B.yml@main
        secrets: inherit # pass all secrets
    jobs:
   
    workflowB-calls-workflowC:
        uses: different-org/example-repo/.github/workflows/C.yml@main
        secrets:
        repo-token: ${{ secrets.personal_access_token }} # pass just this secret
   
    Using outputs from a reusable workflow
    
    A reusable workflow may generate data that you want to use in the caller workflow. To use these outputs, you must specify them as the outputs of the reusable workflow.

    If a reusable workflow that sets an output is executed with a matrix strategy, the output will be the output set by the last successful completing reusable workflow of the matrix which actually sets a value. That means if the last successful completing reusable workflow sets an empty string for its output, and the second last successful completing reusable workflow sets an actual value for its output, the output will contain the value of the second last completing reusable workflow.

    The following reusable workflow has a single job containing two steps. In each of these steps we set a single word as the output: "hello" and "world." In the outputs section of the job, we map these step outputs to job outputs called: output1 and output2. In the on.workflow_call.outputs section we then define two outputs for the workflow itself, one called firstword which we map to output1, and one called secondword which we map to output2.

    The value must be set to the value of a job-level output within the called workflow. Step-level outputs must first be mapped to job-level outputs as shown below.

    For more information, see Passing information between jobs https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/pass-job-outputs#overview  
     and Workflow syntax for GitHub Actions.


    YAML
    
    name: Reusable workflow

    on:
    
    workflow_call:
        # Map the workflow outputs to job outputs
        outputs:
        firstword:
            description: "The first output string"
            value: ${{ jobs.example_job.outputs.output1 }}
        secondword:
            description: "The second output string"
            value: ${{ jobs.example_job.outputs.output2 }}

    jobs:
    
    example_job:
        name: Generate output
        runs-on: ubuntu-latest
        # Map the job outputs to step outputs
        outputs:
        output1: ${{ steps.step1.outputs.firstword }}
        output2: ${{ steps.step2.outputs.secondword }}
        steps:
        - id: step1
            run: echo "firstword=hello" >> $GITHUB_OUTPUT
        - id: step2
            run: echo "secondword=world" >> $GITHUB_OUTPUT
   
    We can now use the outputs in the caller workflow, in the same way you would use the outputs from a job within the same workflow. We reference the outputs using the names defined at the workflow level in the reusable workflow: firstword and secondword. In this workflow, job1 calls the reusable workflow and job2 prints the outputs from the reusable workflow ("hello world") to standard output in the workflow log.

    YAML
    
    name: Call a reusable workflow and use its outputs

    on:
    
    workflow_dispatch:

    jobs:
    job1:
        uses: octo-org/example-repo/.github/workflows/called-workflow.yml@v1

    job2:
        runs-on: ubuntu-latest
        needs: job1
        steps:
        - run: echo ${{ needs.job1.outputs.firstword }} ${{ needs.job1.outputs.secondword }}
   
    For more information on using job outputs, see Workflow syntax for GitHub Actions. If you want to share something other than a variable (e.g. a build artifact) between workflows, see Store and share data with workflow artifacts.   https://docs.github.com/en/actions/tutorials/store-and-share-data

    Monitoring which workflows are being used

    Organizations that use GitHub Enterprise Cloud can interact with the audit log via the GitHub REST API to monitor which workflows are being used. For more information, see the GitHub Enterprise Cloud documentation.

    # END OF READ


For more information, see Writing workflows.
https://docs.github.com/en/actions/how-tos/write-workflows

Events

An event is a specific activity in a repository that triggers a workflow run. For example, an activity can originate from GitHub when someone creates a pull request, opens an issue, or pushes a commit to a repository. You can also trigger a workflow to run on a schedule  

    Events that trigger workflows 

    https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule
    
    You can configure your workflows to run when specific activity on GitHub happens, at a scheduled time, or when an event outside of GitHub occurs.

    In this article
    About events that trigger workflows
    Workflow triggers are events that cause a workflow to run. For more information about how to use workflow triggers, see Triggering a workflow.  https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow

    Some events have multiple activity types. For these events, you can specify which activity types will trigger a workflow run. For more information about what each activity type means, see Webhook events and payloads.

    Note

    Not all webhook events trigger workflows.

    branch_protection_rule
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    branch_protection_rule	- created
    - edited
    - deleted	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Runs your workflow when branch protection rules in the workflow repository are changed. For more information about branch protection rules, see About protected branches. For information about the branch protection rule APIs, see Branches in the GraphQL API documentation or REST API endpoints for branches and their settings.

    For example, you can run a workflow when a branch protection rule has been created or deleted:

    on:
    branch_protection_rule:
        types: [created, deleted]
    check_run
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    check_run	- created
    - rerequested
    - completed
    - requested_action	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    To prevent recursive workflows, this event does not trigger workflows if the check run's check suite was created by GitHub Actions or if the check suite's head SHA is associated with GitHub Actions.
    Runs your workflow when activity related to a check run occurs. A check run is an individual test that is part of a check suite. For information, see Using the REST API to interact with checks. For information about the check run APIs, see Checks in the GraphQL API documentation or REST API endpoints for check runs.

    For example, you can run a workflow when a check run has been rerequested or completed.

    on:
    check_run:
        types: [rerequested, completed]
    check_suite
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    check_suite	- completed	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. Although only the completed activity type is supported, specifying the activity type will keep your workflow specific if more activity types are added in the future. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    To prevent recursive workflows, this event does not trigger workflows if the check suite was created by GitHub Actions or if the check suite's head SHA is associated with GitHub Actions.
    Runs your workflow when check suite activity occurs. A check suite is a collection of the check runs created for a specific commit. Check suites summarize the status and conclusion of the check runs that are in the suite. For information, see Using the REST API to interact with checks. For information about the check suite APIs, see Checks in the GraphQL API documentation or REST API endpoints for check suites.

    For example, you can run a workflow when a check suite has been completed.

    on:
    check_suite:
        types: [completed]
    create
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    create	Not applicable	Last commit on the created branch or tag	Branch or tag created
    Note

    An event will not be created when you create more than three tags at once.

    Runs your workflow when someone creates a Git reference (Git branch or tag) in the workflow's repository. For information about the APIs to create a Git reference, see Git in the GraphQL API documentation or REST API endpoints for Git references.

    For example, you can run a workflow when the create event occurs.

    on:
    create
    delete
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    delete	Not applicable	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.
    An event will not be created when you delete more than three tags at once.
    Runs your workflow when someone deletes a Git reference (Git branch or tag) in the workflow's repository. For information about the APIs to delete a Git reference, see Git in the GraphQL API documentation or REST API endpoints for Git references.

    For example, you can run a workflow when the delete event occurs.

    on:
    delete
    deployment
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    deployment	Not applicable	Commit to be deployed	Branch or tag to be deployed (empty if created with a commit SHA)
    Runs your workflow when someone creates a deployment in the workflow's repository. Deployments created with a commit SHA may not have a Git ref. For information about the APIs to create a deployment, see Deployments in the GraphQL API documentation or REST API endpoints for repositories.

    For example, you can run a workflow when the deployment event occurs.

    on:
    deployment
    deployment_status
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    deployment_status	Not applicable	Commit to be deployed	Branch or tag to be deployed (empty if commit)
    Note

    When a deployment status's state is set to inactive, a workflow run will not be triggered.

    Runs your workflow when a third party provides a deployment status. Deployments created with a commit SHA may not have a Git ref. For information about the APIs to create a deployment status, see Deployments in the GraphQL API documentation or REST API endpoints for deployments.

    For example, you can run a workflow when the deployment_status event occurs.

    on:
    deployment_status
    discussion
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    discussion	- created
    - edited
    - deleted
    - transferred
    - pinned
    - unpinned
    - labeled
    - unlabeled
    - locked
    - unlocked
    - category_changed
    - answered
    - unanswered	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Webhook events for GitHub Discussions are currently in public preview and subject to change.
    Runs your workflow when a discussion in the workflow's repository is created or modified. For activity related to comments on a discussion, use the discussion_comment event. For more information about discussions, see About discussions. For information about the GraphQL API, see Discussions.

    For example, you can run a workflow when a discussion has been created, edited, or answered.

    on:
    discussion:
        types: [created, edited, answered]
    discussion_comment
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    discussion_comment	- created
    - edited
    - deleted
    Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Webhook events for GitHub Discussions are currently in public preview and subject to change.
    Runs your workflow when a comment on a discussion in the workflow's repository is created or modified. For activity related to a discussion as opposed to comments on the discussion, use the discussion event. For more information about discussions, see About discussions. For information about the GraphQL API, see Discussions.

    For example, you can run a workflow when a discussion comment has been created or deleted.

    on:
    discussion_comment:
        types: [created, deleted]
    fork
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    fork	Not applicable	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    Runs your workflow when someone forks a repository. For information about the REST API, see REST API endpoints for forks.

    For example, you can run a workflow when the fork event occurs.

    on:
    fork
    gollum
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    gollum	Not applicable	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    Runs your workflow when someone creates or updates a Wiki page. For more information, see About wikis.

    For example, you can run a workflow when the gollum event occurs.

    on:
    gollum
    image_version
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    Not applicable	Not applicable	Last commit on default branch	Default branch
    Runs your workflow when a new version of a specified image becomes available for use. This event is typically triggered after a successful image version creation, allowing you to automate actions such as deployment or notifications in response to new image versions.

    This event supports glob patterns for both image names and versions. The example below triggers when a new image version matches any of the specified name and version combinations. For example, ["MyNewImage", 1.0.0], ["MyNewImage", 2.53.0], ["MyOtherImage", 1.0.0], and ["MyOtherImage", 2.0.0].

    on:
    image_version:
        names:
        - "MyNewImage"
        - "MyOtherImage"
        versions:
        - 1.*
        - 2.*
    issue_comment
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    issue_comment	- created
    - edited
    - deleted
    Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Runs your workflow when an issue or pull request comment is created, edited, or deleted. For information about the issue comment APIs, see Issues in the GraphQL API documentation or Webhook events and payloads in the REST API documentation.

    For example, you can run a workflow when an issue or pull request comment has been created or deleted.

    on:
    issue_comment:
        types: [created, deleted]
    issue_comment on issues only or pull requests only
    The issue_comment event occurs for comments on both issues and pull requests. You can use the github.event.issue.pull_request property in a conditional to take different action depending on whether the triggering object was an issue or pull request.

    For example, this workflow will run the pr_commented job only if the issue_comment event originated from a pull request. It will run the issue_commented job only if the issue_comment event originated from an issue.

    on: issue_comment

    jobs:
    pr_commented:
        # This job only runs for pull request comments
        name: PR comment
        if: ${{ github.event.issue.pull_request }}
        runs-on: ubuntu-latest
        steps:
        - run: |
            echo A comment on PR $NUMBER
            env:
            NUMBER: ${{ github.event.issue.number }}

    issue_commented:
        # This job only runs for issue comments
        name: Issue comment
        if: ${{ !github.event.issue.pull_request }}
        runs-on: ubuntu-latest
        steps:
        - run: |
            echo A comment on issue $NUMBER
            env:
            NUMBER: ${{ github.event.issue.number }}
    issues
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    issues	- opened
    - edited
    - deleted
    - transferred
    - pinned
    - unpinned
    - closed
    - reopened
    - assigned
    - unassigned
    - labeled
    - unlabeled
    - locked
    - unlocked
    - milestoned
    - demilestoned
    - typed
    - untyped
    - field_added
    - field_removed	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Runs your workflow when an issue in the workflow's repository is created or modified. For activity related to comments in an issue, use the issue_comment event. For more information about issues, see About issues. For information about the issue APIs, see Issues in the GraphQL API documentation or REST API endpoints for issues.

    For example, you can run a workflow when an issue has been opened, edited, or milestoned.

    on:
    issues:
        types: [opened, edited, milestoned]
    You can also run a workflow when an issue field value is set, changed, or cleared. The field_added activity type fires both when a field value is initially set and when an existing value is updated. The field_removed activity type fires when a field value is cleared.

    on:
    issues:
        types: [field_added, field_removed]
    label
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    label	- created
    - edited
    - deleted
    Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Runs your workflow when a label in your workflow's repository is created or modified. For more information about labels, see Managing labels. For information about the label APIs, see Issues in the GraphQL API documentation or REST API endpoints for labels.

    If you want to run your workflow when a label is added to or removed from an issue, pull request, or discussion, use the labeled or unlabeled activity types for the issues, pull_request, pull_request_target, or discussion events instead.

    For example, you can run a workflow when a label has been created or deleted.

    on:
    label:
        types: [created, deleted]
    merge_group
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    merge_group	checks_requested	SHA of the merge group	Ref of the merge group
    Note

    More than one activity type triggers this event. Although only the checks_requested activity type is supported, specifying the activity type will keep your workflow specific if more activity types are added in the future. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    If your repository uses GitHub Actions to perform required checks on pull requests in your repository, you need to update the workflows to include the merge_group event as an additional trigger. Otherwise, status checks will not be triggered when you add a pull request to a merge queue. The merge will fail as the required status check will not be reported. The merge_group event is separate from the pull_request and push events.
    Runs your workflow when a pull request is added to a merge queue, which adds the pull request to a merge group. For more information see Merging a pull request with a merge queue.

    For example, you can run a workflow when the checks_requested activity has occurred.

    on:
    pull_request:
        branches: [ "main" ]
    merge_group:
        types: [checks_requested]
    milestone
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    milestone	- created
    - closed
    - opened
    - edited
    - deleted
    Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Runs your workflow when a milestone in the workflow's repository is created or modified. For more information about milestones, see About milestones. For information about the milestone APIs, see Issues in the GraphQL API documentation or REST API endpoints for milestones.

    If you want to run your workflow when an issue is added to or removed from a milestone, use the milestoned or demilestoned activity types for the issues event instead.

    For example, you can run a workflow when a milestone has been opened or deleted.

    on:
    milestone:
        types: [opened, deleted]
    page_build
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    page_build	Not applicable	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    Runs your workflow when someone pushes to a branch that is the publishing source for GitHub Pages, if GitHub Pages is enabled for the repository. For more information about GitHub Pages publishing sources, see Configuring a publishing source for your GitHub Pages site. For information about the REST API, see REST API endpoints for repositories.

    For example, you can run a workflow when the page_build event occurs.

    on:
    page_build
    public
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    public	Not applicable	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    Runs your workflow when your workflow's repository changes from private to public. For information about the REST API, see REST API endpoints for repositories.

    For example, you can run a workflow when the public event occurs.

    on:
    public
    pull_request
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    pull_request	- assigned
    - unassigned
    - labeled
    - unlabeled
    - opened
    - edited
    - closed
    - reopened
    - synchronize
    - converted_to_draft
    - locked
    - unlocked
    - enqueued
    - dequeued
    - milestoned
    - demilestoned
    - ready_for_review
    - review_requested
    - review_request_removed
    - auto_merge_enabled
    - auto_merge_disabled	Last merge commit on the GITHUB_REF branch	PR merge branch refs/pull/PULL_REQUEST_NUMBER/merge
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, a workflow only runs when a pull_request event's activity type is opened, synchronize, or reopened. To trigger workflows by different activity types, use the types keyword. For more information, see Workflow syntax for GitHub Actions.
    Workflows will not run on pull_request activity if the pull request has a merge conflict. The merge conflict must be resolved first. Conversely, workflows with the pull_request_target event will run even if the pull request has a merge conflict. Before using the pull_request_target trigger, you should be aware of the security risks. For more information, see pull_request_target.
    The pull_request webhook event payload is empty for merged pull requests and pull requests that come from forked repositories.
    When a pull request is created or updated by a workflow using GITHUB_TOKEN, pull_request events with the opened, synchronize, or reopened activity types create workflow runs that require approval. A user with write access to the repository can approve these runs from the pull request page. With the exception of workflow_dispatch and repository_dispatch, other GITHUB_TOKEN-triggered events do not create workflow runs at all.
    The value of GITHUB_REF varies for a closed pull request depending on whether the pull request has been merged or not. If a pull request was closed but not merged, it will be refs/pull/PULL_REQUEST_NUMBER/merge. If a pull request was closed as a result of being merged, it will be the fully qualified ref of the branch it was merged into, for example /refs/heads/main.
    Runs your workflow when activity on a pull request in the workflow's repository occurs. For example, if no activity types are specified, the workflow runs when a pull request is opened or reopened or when the head branch of the pull request is updated. For activity related to pull request reviews, pull request review comments, or pull request comments, use the pull_request_review, pull_request_review_comment, or issue_comment events instead. For information about the pull request APIs, see Pull requests in the GraphQL API documentation or REST API endpoints for pull requests.

    Note that GITHUB_SHA for this event is the last merge commit of the pull request merge branch. If you want to get the commit ID for the last commit to the head branch of the pull request, use github.event.pull_request.head.sha instead. For more information about merge branches, see About pull requests.

    How the merge branch affects your workflow
    For open, mergeable pull requests, workflows triggered by the pull_request event set GITHUB_REF to the merge branch. Because actions/checkout uses GITHUB_REF by default, it checks out the merge branch. Your CI tests run against the merged result, not just the head branch alone:

    GITHUB_REF is set to refs/pull/PULL_REQUEST_NUMBER/merge
    GITHUB_SHA is the SHA of the merge commit on the merge branch
    To test only the head branch commits without simulating a merge, check out the head branch using github.event.pull_request.head.sha in your workflow.

    For example, you can run a workflow when a pull request has been opened or reopened.

    on:
    pull_request:
        types: [opened, reopened]
    You can use the event context to further control when jobs in your workflow will run. For example, this workflow will run when a review is requested on a pull request, but the specific_review_requested job will only run when a review by octo-team is requested.

    on:
    pull_request:
        types: [review_requested]
    jobs:
    specific_review_requested:
        runs-on: ubuntu-latest
        if: ${{ github.event.requested_team.name == 'octo-team'}}
        steps:
        - run: echo 'A review from octo-team was requested'
    Running your pull_request workflow based on the head or base branch of a pull request
    You can use the branches or branches-ignore filter to configure your workflow to only run on pull requests that target specific branches. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when someone opens a pull request that targets a branch whose name starts with releases/:

    on:
    pull_request:
        types:
        - opened
        branches:
        - 'releases/**'
    Note

    If you use both the branches filter and the paths filter, the workflow will only run when both filters are satisfied. For example, the following workflow will only run when a pull request that includes a change to a JavaScript (.js) file is opened on a branch whose name starts with releases/:

    on:
    pull_request:
        types:
        - opened
        branches:
        - 'releases/**'
        paths:
        - '**.js'
    To run a job based on the pull request's head branch name (as opposed to the pull request's base branch name), use the github.head_ref context in a conditional. For example, this workflow will run whenever a pull request is opened, but the run_if job will only execute if the head of the pull request is a branch whose name starts with releases/:

    on:
    pull_request:
        types:
        - opened
    jobs:
    run_if:
        if: startsWith(github.head_ref, 'releases/')
        runs-on: ubuntu-latest
        steps:
        - run: echo "The head of this PR starts with 'releases/'"
    Running your pull_request workflow based on files changed in a pull request
    You can also configure your workflow to run when a pull request changes specific files. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when a pull request includes a change to a JavaScript file (.js):

    on:
    pull_request:
        paths:
        - '**.js'
    Note

    If you use both the branches filter and the paths filter, the workflow will only run when both filters are satisfied. For example, the following workflow will only run when a pull request that includes a change to a JavaScript (.js) file is opened on a branch whose name starts with releases/:

    on:
    pull_request:
        types:
        - opened
        branches:
        - 'releases/**'
        paths:
        - '**.js'
    Running your pull_request workflow when a pull request merges
    When a pull request merges, the pull request is automatically closed. To run a workflow when a pull request merges, use the pull_request closed event type along with a conditional that checks the merged value of the event. For example, the following workflow will run whenever a pull request closes. The if_merged job will only run if the pull request was also merged.

    on:
    pull_request:
        types:
        - closed

    jobs:
    if_merged:
        if: github.event.pull_request.merged == true
        runs-on: ubuntu-latest
        steps:
        - run: |
            echo The PR was merged
    Workflows in forked repositories
    Workflows don't run in forked repositories by default. You must enable GitHub Actions in the Actions tab of the forked repository.

    With the exception of GITHUB_TOKEN, secrets are not passed to the runner when a workflow is triggered from a forked repository. The GITHUB_TOKEN has read-only permissions in pull requests from forked repositories. For more information, see Use GITHUB_TOKEN for authentication in workflows.

    Pull request events for forked repositories
    For pull requests from a forked repository to the base repository, GitHub sends the pull_request, issue_comment, pull_request_review_comment, pull_request_review, and pull_request_target events to the base repository. No pull request events occur on the forked repository.

    When a first-time contributor submits a pull request to a public repository, a maintainer with write access may need to approve running workflows on the pull request. For more information, see Approving workflow runs from forks.

    For pull requests from a forked repository to a private repository, workflows only run when they are enabled, see Managing GitHub Actions settings for a repository.

    Note

    Workflows triggered by Dependabot pull requests are treated as though they are from a forked repository, and are also subject to these restrictions.

    pull_request_comment (use issue_comment)
    To run your workflow when a comment on a pull request (not on a pull request's diff) is created, edited, or deleted, use the issue_comment event. For activity related to pull request reviews or pull request review comments, use the pull_request_review or pull_request_review_comment events.

    pull_request_review
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    pull_request_review	- submitted
    - edited
    - dismissed	Last merge commit on the GITHUB_REF branch	PR merge branch refs/pull/PULL_REQUEST_NUMBER/merge
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.

    Runs your workflow when a pull request review is submitted, edited, or dismissed. A pull request review is a group of pull request review comments in addition to a body comment and a state. For activity related to pull request review comments or pull request comments, use the pull_request_review_comment or issue_comment events instead. For information about the pull request review APIs, see Pull requests in the GraphQL API documentation or REST API endpoints for pull requests.

    For example, you can run a workflow when a pull request review has been edited or dismissed.

    on:
    pull_request_review:
        types: [edited, dismissed]
    Running a workflow when a pull request is approved
    To run your workflow when a pull request has been approved, you can trigger your workflow with the submitted type of pull_request_review event, then check the review state with the github.event.review.state property. For example, this workflow will run whenever a pull request review is submitted, but the approved job will only run if the submitted review is an approving review:

    on:
    pull_request_review:
        types: [submitted]

    jobs:
    approved:
        if: github.event.review.state == 'approved'
        runs-on: ubuntu-latest
        steps:
        - run: echo "This PR was approved"
    Workflows in forked repositories
    Workflows don't run in forked repositories by default. You must enable GitHub Actions in the Actions tab of the forked repository.

    With the exception of GITHUB_TOKEN, secrets are not passed to the runner when a workflow is triggered from a forked repository. The GITHUB_TOKEN has read-only permissions in pull requests from forked repositories. For more information, see Use GITHUB_TOKEN for authentication in workflows.

    Pull request events for forked repositories
    For pull requests from a forked repository to the base repository, GitHub sends the pull_request, issue_comment, pull_request_review_comment, pull_request_review, and pull_request_target events to the base repository. No pull request events occur on the forked repository.

    When a first-time contributor submits a pull request to a public repository, a maintainer with write access may need to approve running workflows on the pull request. For more information, see Approving workflow runs from forks.

    For pull requests from a forked repository to a private repository, workflows only run when they are enabled, see Managing GitHub Actions settings for a repository.

    Note

    Workflows triggered by Dependabot pull requests are treated as though they are from a forked repository, and are also subject to these restrictions.

    pull_request_review_comment
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    pull_request_review_comment	- created
    - edited
    - deleted	Last merge commit on the GITHUB_REF branch	PR merge branch refs/pull/PULL_REQUEST_NUMBER/merge
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.

    Runs your workflow when a pull request review comment is modified. A pull request review comment is a comment on a pull request's diff. For activity related to pull request reviews or pull request comments, use the pull_request_review or issue_comment events instead. For information about the pull request review comment APIs, see Pull requests in the GraphQL API documentation or REST API endpoints for pull requests.

    For example, you can run a workflow when a pull request review comment has been created or deleted.

    on:
    pull_request_review_comment:
        types: [created, deleted]
    Workflows in forked repositories
    Workflows don't run in forked repositories by default. You must enable GitHub Actions in the Actions tab of the forked repository.

    With the exception of GITHUB_TOKEN, secrets are not passed to the runner when a workflow is triggered from a forked repository. The GITHUB_TOKEN has read-only permissions in pull requests from forked repositories. For more information, see Use GITHUB_TOKEN for authentication in workflows.

    Pull request events for forked repositories
    For pull requests from a forked repository to the base repository, GitHub sends the pull_request, issue_comment, pull_request_review_comment, pull_request_review, and pull_request_target events to the base repository. No pull request events occur on the forked repository.

    When a first-time contributor submits a pull request to a public repository, a maintainer with write access may need to approve running workflows on the pull request. For more information, see Approving workflow runs from forks.

    For pull requests from a forked repository to a private repository, workflows only run when they are enabled, see Managing GitHub Actions settings for a repository.

    Note

    Workflows triggered by Dependabot pull requests are treated as though they are from a forked repository, and are also subject to these restrictions.

    pull_request_target
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    pull_request	- assigned
    - unassigned
    - labeled
    - unlabeled
    - opened
    - edited
    - closed
    - reopened
    - synchronize
    - converted_to_draft
    - locked
    - unlocked
    - enqueued
    - dequeued
    - milestoned
    - demilestoned
    - ready_for_review
    - review_requested
    - review_request_removed
    - auto_merge_enabled
    - auto_merge_disabled	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, a workflow only runs when a pull_request_target event's activity type is opened, synchronize, or reopened. To trigger workflows by different activity types, use the types keyword. For more information, see Workflow syntax for GitHub Actions.

    Runs your workflow when activity on a pull request in the workflow's repository occurs. For example, if no activity types are specified, the workflow runs when a pull request is opened or reopened or when the head branch of the pull request is updated.

    This event runs in the context of the default branch of the base repository, rather than in the context of the merge commit, as the pull_request event does. This prevents execution of unsafe code from the head of the pull request that could alter your repository or steal any secrets you use in your workflow. This event allows your workflow to do things like label or comment on pull requests from forks. Avoid using this event if you need to build or run code from the pull request.

    To ensure repository security, branches with names that match certain patterns (such as those which look similar to SHAs) may not trigger workflows with the pull_request_target event.

    Warning

    Running untrusted code on the pull_request_target trigger may lead to security vulnerabilities. These vulnerabilities include cache poisoning and granting unintended access to write privileges or secrets. For more information, see Secure use reference in the GitHub Enterprise Cloud documentation, and Preventing pwn requests on the GitHub Security Lab website.

    For example, you can run a workflow when a pull request has been assigned, opened, synchronize, or reopened.

    on:
    pull_request_target:
        types: [assigned, opened, synchronize, reopened]
    Running your pull_request_target workflow based on the head or base branch of a pull request
    You can use the branches or branches-ignore filter to configure your workflow to only run on pull requests that target specific branches. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when someone opens a pull request that targets a branch whose name starts with releases/:

    on:
    pull_request_target:
        types:
        - opened
        branches:
        - 'releases/**'
    Note

    If you use both the branches filter and the paths filter, the workflow will only run when both filters are satisfied. For example, the following workflow will only run when a pull request that includes a change to a JavaScript (.js) file is opened on a branch whose name starts with releases/:

    on:
    pull_request_target:
        types:
        - opened
        branches:
        - 'releases/**'
        paths:
        - '**.js'
    To run a job based on the pull request's head branch name (as opposed to the pull request's base branch name), use the github.head_ref context in a conditional. For example, this workflow will run whenever a pull request is opened, but the run_if job will only execute if the head of the pull request is a branch whose name starts with releases/:

    on:
    pull_request_target:
        types:
        - opened
    jobs:
    run_if:
        if: startsWith(github.head_ref, 'releases/')
        runs-on: ubuntu-latest
        steps:
        - run: echo "The head of this PR starts with 'releases/'"
    Running your pull_request_target workflow based on files changed in a pull request
    You can use the paths or paths-ignore filter to configure your workflow to run when a pull request changes specific files. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when a pull request includes a change to a JavaScript file (.js):

    on:
    pull_request_target:
        paths:
        - '**.js'
    Note

    If you use both the branches filter and the paths filter, the workflow will only run when both filters are satisfied. For example, the following workflow will only run when a pull request that includes a change to a JavaScript (.js) file is opened on a branch whose name starts with releases/:

    on:
    pull_request_target:
        types:
        - opened
        branches:
        - 'releases/**'
        paths:
        - '**.js'
    Running your pull_request_target workflow when a pull request merges
    When a pull request merges, the pull request is automatically closed. To run a workflow when a pull request merges, use the pull_request_target closed event type along with a conditional that checks the merged value of the event. For example, the following workflow will run whenever a pull request closes. The if_merged job will only run if the pull request was also merged.

    on:
    pull_request_target:
        types:
        - closed

    jobs:
    if_merged:
        if: github.event.pull_request.merged == true
        runs-on: ubuntu-latest
        steps:
        - run: |
            echo The PR was merged
    push
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    push	Not applicable	Tip commit pushed to the ref. When you delete a branch, the SHA in the workflow run (and its associated refs) reverts to the default branch of the repository.	Updated ref
    Note

    The webhook payload available to GitHub Actions does not include the added, removed, and modified attributes in the commit object. You can retrieve the full commit object using the API. For information, see Commits in the GraphQL API documentation or REST API endpoints for commits.
    Events will not be created if more than 5,000 branches are pushed at once. Events will not be created for tags when more than three tags are pushed at once.
    Runs your workflow when you push a commit or tag, or when you create a repository from a template. This includes workflows that are not merged into the default branch. For more information, see Events that trigger workflows.

    For example, you can run a workflow when the push event occurs.

    on:
    push
    Note

    When a push webhook event triggers a workflow run, the Actions UI's "pushed by" field shows the account of the pusher and not the author or committer. However, if the changes are pushed to a repository using SSH authentication with a deploy key, then the "pushed by" field will be the repository admin who verified the deploy key when it was added it to a repository.

    Running your workflow only when a push to specific branches occurs
    You can use the branches or branches-ignore filter to configure your workflow to only run when specific branches are pushed. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when someone pushes to main or to a branch that starts with releases/.

    on:
    push:
        branches:
        - 'main'
        - 'releases/**'
    Note

    If you use both the branches filter and the paths filter, the workflow will only run when both filters are satisfied. For example, the following workflow will only run when a push that includes a change to a JavaScript (.js) file is made to a branch whose name starts with releases/:

    on:
    push:
        branches:
        - 'releases/**'
        paths:
        - '**.js'
    Running your workflow only when a push of specific tags occurs
    You can use the tags or tags-ignore filter to configure your workflow to only run when specific tags are pushed. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when someone pushes a tag that starts with v1..

    on:
    push:
        tags:
        - v1.**
    Running your workflow only when a push affects specific files
    You can use the paths or paths-ignore filter to configure your workflow to run when a push to specific files occurs. For more information, see Workflow syntax for GitHub Actions.

    For example, this workflow will run when someone pushes a change to a JavaScript file (.js):

    on:
    push:
        paths:
        - '**.js'
    registry_package
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    registry_package	- published
    - updated	Commit of the published package	Branch or tag of the published package
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    When pushing multi-architecture container images, this event occurs once per manifest, so you might observe your workflow triggering multiple times. To mitigate this, and only run your workflow job for the event that contains the actual image tag information, use a conditional:
    jobs:
        job_name:
            if: $true
    Runs your workflow when activity related to GitHub Packages occurs in your repository. For more information, see GitHub Packages Documentation.

    For example, you can run a workflow when a new package version has been published.

    on:
    registry_package:
        types: [published]
    release
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    release	- published
    - unpublished
    - created
    - edited
    - deleted
    - prereleased
    - released	Last commit in the tagged release	Tag ref of release refs/tags/<tag_name>
    Note

    More than one activity type triggers this event. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    Workflows are not triggered for the created, edited, or deleted activity types for draft releases. When you create your release through the GitHub UI, your release may automatically be saved as a draft.
    The prereleased type will not trigger for pre-releases published from draft releases, but the published type will trigger. If you want a workflow to run when stable and pre-releases publish, subscribe to published instead of released and prereleased.
    Runs your workflow when release activity in your repository occurs. For information about the release APIs, see Releases in the GraphQL API documentation or REST API endpoints for releases and release assets in the REST API documentation.

    For example, you can run a workflow when a release has been published.

    on:
    release:
        types: [published]
    repository_dispatch
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    repository_dispatch	Custom	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    You can use the GitHub API to trigger a webhook event called repository_dispatch when you want to trigger a workflow for activity that happens outside of GitHub. For more information, see REST API endpoints for repositories.

    When you make a request to create a repository_dispatch event, you must specify an event_type to describe the activity type. By default, all repository_dispatch activity types trigger a workflow to run. You can use the types keyword to limit your workflow to run when a specific event_type value is sent in the repository_dispatch webhook payload.

    on:
    repository_dispatch:
        types: [test_result]
    Note

    The event_type value is limited to 100 characters.

    Any data that you send through the client_payload parameter will be available in the github.event context in your workflow. For example, if you send this request body when you create a repository dispatch event:

    {
    "event_type": "test_result",
    "client_payload": {
        "passed": false,
        "message": "Error: timeout"
    }
    }
    then you can access the payload in a workflow like this:

    on:
    repository_dispatch:
        types: [test_result]

    jobs:
    run_if_failure:
        if: ${{ !github.event.client_payload.passed }}
        runs-on: ubuntu-latest
        steps:
        - env:
            MESSAGE: ${{ github.event.client_payload.message }}
            run: echo $MESSAGE
    Note

    The maximum number of top-level properties in client_payload is 10.
    The payload can contain a maximum of 65,535 characters.
    schedule
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    Not applicable	Not applicable	Last commit on default branch	Default branch
    Note

    The schedule event can be delayed during periods of high loads of GitHub Actions workflow runs. High load times include the start of every hour. If the load is sufficiently high enough, some queued jobs may be dropped. To decrease the chance of delay, schedule your workflow to run at a different time of the hour.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Scheduled workflows will only run on the default branch.
    In a public repository, scheduled workflows are automatically disabled when no repository activity has occurred in 60 days. For information on re-enabling a disabled workflow, see Disabling and enabling a workflow.
    The schedule event allows you to trigger a workflow at a scheduled time.

    Example:

    on:
    schedule:
        - cron: "15 4,5 * * *"
    Use POSIX cron syntax to schedule workflows to run at specific times. By default, scheduled workflows run in UTC. You can optionally specify a timezone using an IANA timezone string for timezone-aware scheduling. Scheduled workflows run on the latest commit on the default branch. The shortest interval you can run scheduled workflows is once every 5 minutes.

    Note

    For schedules that set timezone to a time zone that observes daylight saving time (DST), during DST spring-forward transitions, scheduled workflows in skipped hours advance to the next valid time. For example, a 2:30 AM schedule advances to 3:00 AM.

    Cron syntax has five fields separated by a space, and each field represents a unit of time.

    ┌───────────── minute (0 - 59)
    │ ┌───────────── hour (0 - 23)
    │ │ ┌───────────── day of the month (1 - 31)
    │ │ │ ┌───────────── month (1 - 12 or JAN-DEC)
    │ │ │ │ ┌───────────── day of the week (0 - 6 or SUN-SAT)
    │ │ │ │ │
    * * * * *
    You can use these operators in any of the five fields:

    Operator	Description	Example
    *	Any value	15 * * * * runs at every minute 15 of every hour of every day.
    ,	Value list separator	2,10 4,5 * * * runs at minute 2 and 10 of the 4th and 5th hour of every day.
    -	Range of values	30 4-6 * * * runs at minute 30 of the 4th, 5th, and 6th hour.
    /	Step values	20/15 * * * * runs every 15 minutes starting from minute 20 through 59 (minutes 20, 35, and 50).
    This example triggers the workflow to run at 5:30 AM in the America/New_York timezone every Monday through Friday:

    on:
    schedule:
        - cron: '30 5 * * 1-5'
        timezone: "America/New_York"
    A single workflow can be triggered by multiple schedule events. Access the schedule event that triggered the workflow through the github.event.schedule context. This example triggers the workflow to run at 5:30 UTC every Monday-Thursday, and 17:30 UTC on Tuesdays and Thursdays, but skips the Not on Monday or Wednesday step on Monday and Wednesday.

    on:
    schedule:
        - cron: '30 5 * * 1,3'
        - cron: '30 5,17 * * 2,4'

    jobs:
    test_schedule:
        runs-on: ubuntu-latest
        steps:
        - name: Not on Monday or Wednesday
            if: github.event.schedule != '30 5 * * 1,3'
            run: echo "This step will be skipped on Monday and Wednesday"
        - name: Every time
            run: echo "This step will always run"
    Note

    GitHub Actions does not support the non-standard syntax @yearly, @monthly, @weekly, @daily, @hourly, and @reboot.

    You can use crontab guru to help generate your cron syntax and confirm what time it will run. To help you get started, there is also a list of crontab guru examples.

    actor for scheduled workflows
    Certain repository events change the actor associated with the workflow. For example, a user who changes the default branch of the repository, which changes the branch on which scheduled workflows run, becomes actor for those scheduled workflows.

    For a deactivated scheduled workflow, if a user with write permissions to the repository makes a commit that changes the cron schedule on the workflow, the workflow will be reactivated, and that user will become the actor associated with any workflow runs.

    Notifications for scheduled workflows are sent to the user who last modified the cron syntax in the workflow file. For more information, see Notifications for workflow runs.

    Note

    For an enterprise with Enterprise Managed Users, triggering a scheduled workflow requires that the status of the actor user account associated with the workflow is currently active (i.e. not suspended or deleted).

    Scheduled workflows will not run if the last actor associated with the scheduled workflow has been deprovisioned by the Enterprise Managed User identity provider (IdP). However, if the last actor Enterprise Managed User has not been deprovisioned by the IdP, and has only been removed as a member from a given organization in the enterprise, scheduled workflows will still run with that user set as the actor.
    Similarly, for an enterprise without Enterprise Managed Users, removing a user from an organization will not prevent scheduled workflows which had that user as their actor from running.
    Thus, the user account's status, in both Enterprise Managed User and non-Enterprise Managed User scenarios, is what's important, not the user's membership status in the organization where the scheduled workflow is located.
    status
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    status	Not applicable	Last commit on default branch	Default branch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    Runs your workflow when the status of a Git commit changes. For example, commits can be marked as error, failure, pending, or success. If you want to provide more details about the status change, you may want to use the check_run event. For information about the commit status APIs, see Commits in the GraphQL API documentation or REST API endpoints for commits.

    For example, you can run a workflow when the status event occurs.

    on:
    status
    If you want to run a job in your workflow based on the new commit state, you can use the github.event.state context. For example, the following workflow triggers when a commit status changes, but the if_error_or_failure job only runs if the new commit state is error or failure.

    on:
    status
    jobs:
    if_error_or_failure:
        runs-on: ubuntu-latest
        if: >-
        github.event.state == 'error' ||
        github.event.state == 'failure'
        steps:
        - env:
            DESCRIPTION: ${{ github.event.description }}
            run: |
            echo The status is error or failed: $DESCRIPTION
    watch
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    watch	- started	Last commit on default branch	Default branch
    Note

    More than one activity type triggers this event. Although only the started activity type is supported, specifying the activity type will keep your workflow specific if more activity types are added in the future. For information about each activity type, see Webhook events and payloads. By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.
    This event will only trigger a workflow run if the workflow file exists on the default branch.
    Runs your workflow when the workflow's repository is starred. For information about the pull request APIs, see Activity in the GraphQL API documentation or REST API endpoints for starring.

    For example, you can run a workflow when someone stars a repository, which is the started activity type for a watch event.

    on:
    watch:
        types: [started]
    workflow_call
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    Same as the caller workflow	Not applicable	Same as the caller workflow	Same as the caller workflow
    workflow_call is used to indicate that a workflow can be called by another workflow. When a workflow is triggered with the workflow_call event, the event payload in the called workflow is the same event payload from the calling workflow. For more information see, Reuse workflows.

    The example below only runs the workflow when it's called from another workflow:

    on: workflow_call
    workflow_dispatch
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    workflow_dispatch	Not applicable	Last commit on the GITHUB_REF branch or tag	Branch or tag that received dispatch
    Note

    This event will only trigger a workflow run if the workflow file exists on the default branch.

    To enable a workflow to be triggered manually, you need to configure the workflow_dispatch event. You can manually trigger a workflow run using the GitHub API, GitHub CLI, or the GitHub UI. For more information, see Manually running a workflow.

    on: workflow_dispatch
    Providing inputs
    You can configure custom-defined input properties, default input values, and required inputs for the event directly in your workflow. When you trigger the event, you can provide the ref and any inputs. When the workflow runs, you can access the input values in the inputs context. For more information, see Contexts reference.

    Note

    The workflow will also receive the inputs in the github.event.inputs context. The information in the inputs context and github.event.inputs context is identical except that the inputs context preserves Boolean values as Booleans instead of converting them to strings. The choice type resolves to a string and is a single selectable option.
    The maximum number of top-level properties for inputs is 25 .
    The maximum payload for inputs is 65,535 characters.
    This example defines inputs called logLevel, tags, and environment. You pass values for these inputs to the workflow when you run it. This workflow then prints the values to the log, using the inputs.logLevel, inputs.tags, and inputs.environment context properties.

    on:
    workflow_dispatch:
        inputs:
        logLevel:
            description: 'Log level'
            required: true
            default: 'warning'
            type: choice
            options:
            - info
            - warning
            - debug
        tags:
            description: 'Test scenario tags'
            required: false
            type: boolean
        environment:
            description: 'Environment to run tests against'
            type: environment
            required: true

    jobs:
    log-the-inputs:
        runs-on: ubuntu-latest
        steps:
        - run: |
            echo "Log level: $LEVEL"
            echo "Tags: $TAGS"
            echo "Environment: $ENVIRONMENT"
            env:
            LEVEL: ${{ inputs.logLevel }}
            TAGS: ${{ inputs.tags }}
            ENVIRONMENT: ${{ inputs.environment }}
    If you run this workflow from a browser you must enter values for the required inputs manually before the workflow will run.


![workflow triggers workflow run setup](../GitHub$20%Actions/pics/workflowsrun.jpg)


    You can also pass inputs when you run a workflow from a script, or by using GitHub CLI. For example:

    gh workflow run run-tests.yml -f logLevel=warning -f tags=false -f environment=staging

    For more information, see the GitHub CLI information in Manually running a workflow.


        Manually running a workflow
        
        When a workflow is configured to run on the workflow_dispatch event, you can run the workflow using the Actions tab on GitHub, GitHub CLI, or the REST API.

        Tool navigation
        
        GitHub CLI       "I selected the GitHub CLI for this edit as is below, as it make easy sense."

        Web browser     
        In this article
        
        Configuring a workflow to run manually
        
        To run a workflow manually, the workflow must be configured to run on the workflow_dispatch event.

        To trigger the workflow_dispatch event, your workflow must be in the default branch. For more information about configuring the workflow_dispatch event, see Events that trigger workflows. "We covered this page earlier"

        Write access to the repository is required to perform these steps.

        Running a workflow
        
        Note
        
        To learn more about GitHub CLI, see About GitHub CLI.
        https://docs.github.com/en/github-cli/github-cli/about-github-cli

        To run a workflow, use the workflow run subcommand. Replace the workflow parameter with either the name, ID, or file name of the workflow you want to run. For example, "Link Checker", 1234567, or "link-check-test.yml". If you don't specify a workflow, GitHub CLI returns an interactive menu for you to choose a workflow.

        gh workflow run WORKFLOW

        If your workflow accepts inputs, GitHub CLI will prompt you to enter them. Alternatively, you can use -f or -F to add an input in key=value format. Use -F to read from a file.

        gh workflow run greet.yml -f name=mona -f greeting=hello -F data=@myfile.txt
        
        You can also pass inputs as JSON by using standard input.

        echo '{"name":"mona", "greeting":"hello"}' | gh workflow run greet.yml --json
        
        To run a workflow on a branch other than the repository's default branch, use the --ref flag.

        gh workflow run WORKFLOW --ref BRANCH
        
        To view the progress of the workflow run, use the run watch subcommand and select the run from the interactive list.

        gh run watch
        
        Running a workflow using the REST API
        
        When using the REST API, you configure the inputs and ref as request body parameters. If the inputs are omitted, the default values defined in the workflow file are used.

        Note

        You can define up to 25 inputs for a workflow_dispatch event.

        For more information about using the REST API, see REST API endpoints for workflows.

        # END OF READ


    workflow_run
    Webhook event payload	Activity types	GITHUB_SHA	GITHUB_REF
    workflow_run	- completed
    - requested
    - in_progress	Last commit on default branch	Default branch

    Note

    More than one activity type triggers this event. The requested activity type does not occur when a workflow is re-run. For information about each activity type, see Webhook events and payloads. 
    
    By default, all activity types trigger workflows that run on this event. You can limit your workflow runs to specific activity types using the types keyword. For more information, see Workflow syntax for GitHub Actions.

    This event will only trigger a workflow run if the workflow file exists on the default branch.
    You can't use workflow_run to chain together more than three levels of workflows. For example, if you attempt to trigger five workflows (named B to F) to run sequentially after an initial workflow A has run (that is: A → B → C → D → E → F), workflows E and F will not be run.

    This event occurs when a workflow run is requested or completed. It allows you to execute a workflow based on execution or completion of another workflow. The workflow started by the workflow_run event is able to access secrets and write tokens, even if the previous workflow was not. This is useful in cases where the previous workflow is intentionally not privileged, but you need to take a privileged action in a later workflow.

    Warning

    Running untrusted code on the workflow_run trigger may lead to security vulnerabilities. These vulnerabilities include cache poisoning and granting unintended access to write privileges or secrets. For more information, see Secure use reference in the GitHub Enterprise Cloud documentation, and Preventing pwn requests on the GitHub Security Lab website.

    In this example, a workflow is configured to run after the separate "Run Tests" workflow completes.

    on:
    workflow_run:
        workflows: [Run Tests]
        types:
        - completed

    If you specify multiple workflows for the workflow_run event, only one of the workflows needs to run. For example, a workflow with the following trigger will run whenever the "Staging" workflow or the "Lab" 
    workflow completes.

    on:
    workflow_run:
        workflows: [Staging, Lab]
        types:
        - completed

    Running a workflow based on the conclusion of another workflow

    A workflow run is triggered regardless of the conclusion of the previous workflow. If you want to run a job or step based on the result of the triggering workflow, you can use a conditional with the github.event.workflow_run.conclusion property. For example, this workflow will run whenever a workflow named "Build" completes, but the on-success job will only run if the "Build" workflow succeeded, and the on-failure job will only run if the "Build" workflow failed:

    on:
    workflow_run:
        workflows: [Build]
        types: [completed]

    jobs:
    on-success:
        runs-on: ubuntu-latest
        if: ${{ github.event.workflow_run.conclusion == 'success' }}
        steps:
        - run: echo 'The triggering workflow passed'
    on-failure:
        runs-on: ubuntu-latest
        if: ${{ github.event.workflow_run.conclusion == 'failure' }}
        steps:
        - run: echo 'The triggering workflow failed'

    Limiting your workflow to run based on branches

    You can use the branches or branches-ignore filter to specify what branches the triggering workflow must run on in order to trigger your workflow. For more information, see Workflow syntax for GitHub Actions. For example, a workflow with the following trigger will only run when the workflow named Build runs on a branch named canary.

    on:
    workflow_run:
        workflows: [Build]
        types: [requested]
        branches: [canary]

    Using data from the triggering workflow

    You can access the workflow_run event payload that corresponds to the workflow that triggered your workflow. For example, if your triggering workflow generates artifacts, a workflow triggered with the workflow_run event can access these artifacts.

    The following workflow uploads data as an artifact. (In this simplified example, the data is the pull request number.)

    name: Upload data

    on:
    pull_request:

    jobs:
    upload:
        runs-on: ubuntu-latest

        steps:
        - name: Save PR number
            env:
            PR_NUMBER: ${{ github.event.number }}
            run: |
            mkdir -p ./pr
            echo $PR_NUMBER > ./pr/pr_number
        - uses: actions/upload-artifact@v4
            with:
            name: pr_number
            path: pr/

    When a run of the above workflow completes, it triggers a run of the following workflow. The following workflow uses the github.event.workflow_run context and the GitHub REST API to download the artifact that was uploaded by the above workflow, unzips the downloaded artifact, and comments on the pull request whose number was uploaded as an artifact.

    name: Use the data

    on:
    workflow_run:
        workflows: [Upload data]
        types:
        - completed

    jobs:
    download:
        runs-on: ubuntu-latest
        steps:
        - name: 'Download artifact'
            uses: actions/github-script@v8
            with:
            script: |
                let allArtifacts = await github.rest.actions.listWorkflowRunArtifacts({
                owner: context.repo.owner,
                repo: context.repo.repo,
                run_id: context.payload.workflow_run.id,
                });
                let matchArtifact = allArtifacts.data.artifacts.filter((artifact) => {
                return artifact.name == "pr_number"
                })[0];
                let download = await github.rest.actions.downloadArtifact({
                owner: context.repo.owner,
                repo: context.repo.repo,
                artifact_id: matchArtifact.id,
                archive_format: 'zip',
                });
                const fs = require('fs');
                const path = require('path');
                const temp = '${{ runner.temp }}/artifacts';
                if (!fs.existsSync(temp)){
                fs.mkdirSync(temp);
                }
                fs.writeFileSync(path.join(temp, 'pr_number.zip'), Buffer.from(download.data));

        - name: 'Unzip artifact'
            run: unzip "${{ runner.temp }}/artifacts/pr_number.zip" -d "${{ runner.temp }}/artifacts"

        - name: 'Comment on PR'
            uses: actions/github-script@v8
            with:
            github-token: ${{ secrets.GITHUB_TOKEN }}
            script: |
                const fs = require('fs');
                const path = require('path');
                const temp = '${{ runner.temp }}/artifacts';
                const issue_number = Number(fs.readFileSync(path.join(temp, 'pr_number')));
                await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue_number,
                body: 'Thank you for the PR!'
                });

        # END OF READ

, by posting to a REST API, or manually.  https://docs.github.com/en/rest/repos/repos?apiVersion=2026-03-10#create-a-repository-dispatch-event

For a complete list of events that can be used to trigger workflows, see Events that trigger workflows.  https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows

Jobs

A job is a set of steps in a workflow that is executed on the same runner. Each step is either a shell script that will be executed, or an action that will be run. Steps are executed in order and are dependent on each other. Since each step is executed on the same runner, you can share data from one step to another. For example, you can have a step that builds your application followed by a step that tests the application that was built.

You can configure a job's dependencies with other jobs; by default, jobs have no dependencies and run in parallel. When a job takes a dependency on another job, it waits for the dependent job to complete before running.

You can also use a matrix to run the same job multiple times, each with a different combination of variables—like operating systems or language versions.

For example, you might configure multiple build jobs for different architectures without any job dependencies and a packaging job that depends on those builds. The build jobs run in parallel, and once they complete successfully, the packaging job runs.

For more information, see Choosing what your workflow does.  https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do

Actions

An action is a pre-defined, reusable set of jobs or code that performs specific tasks within a workflow, reducing the amount of repetitive code you write in your workflow files. Actions can perform tasks such as:

Pulling your Git repository from GitHub

Setting up the correct toolchain for your build environment
Setting up authentication to your cloud provider

You can write your own actions, or you can find actions to use in your workflows in the GitHub Marketplace.

For more information on actions, see Reusing automations.  https://docs.github.com/en/actions/how-tos/reuse-automations 
"Has reuseing workflows"

Runners

A runner is a server that runs your workflows when they're triggered. Each runner can run a single job at a time. GitHub provides Ubuntu Linux, Microsoft Windows, and macOS runners to run your workflows. Each workflow run executes in a fresh, newly-provisioned virtual machine.

GitHub also offers larger runners, which are available in larger configurations. For more information, see Using larger runners.  https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners 

If you need a different operating system or require a specific hardware configuration, you can host your own runners.

For more information about self-hosted runners, see Managing self-hosted runners.
https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners
"Has running jobs in a container"

Next steps

GitHub Actions can help you automate nearly every aspect of your application development processes. Ready to get started? Here are some helpful resources for taking your next steps with GitHub Actions:

To create a GitHub Actions workflow, see Using workflow templates.  https://docs.github.com/en/actions/how-tos/write-workflows/use-workflow-templates

For continuous integration (CI) workflows, see Building and testing your code.  https://docs.github.com/en/actions/tutorials/build-and-test-code

For building and publishing packages, see Publishing packages.  https://docs.github.com/en/actions/tutorials/publish-packages

For deploying projects, see Deploying to third-party platforms.  https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms

For automating tasks and processes on GitHub, see Managing your work with GitHub Actions.  https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms

For examples that demonstrate more complex features of GitHub Actions, see Managing your work with GitHub Actions.  https://docs.github.com/en/actions/tutorials/manage-your-work  These detailed examples explain how to test your code on a runner, access the GitHub CLI, and use advanced features such as concurrency and test matrices.

To certify your proficiency in automating workflows and accelerating development with GitHub Actions, earn a GitHub Actions certificate with GitHub Certifications. For more information, see About GitHub Certifications.  https://docs.github.com/en/get-started/showcase-your-expertise-with-github-certifications/about-github-certifications

