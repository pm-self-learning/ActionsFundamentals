# 🔨 Hands-on: Reusable workflows

In this hands-on lab you will create a [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow) and a workflow that consumes it. You will learn to pass in parameters to the reusable workflow and use output parameters in the consuming workflow.

This hands on lab consists of the following steps:
- [🔨 Hands-on: Reusable workflows](#-hands-on-reusable-workflows)
  - [Create a reusable workflow](#create-a-reusable-workflow)
  - [Add an output parameter](#add-an-output-parameter)
  - [Consume the reusable workflow](#consume-the-reusable-workflow)
  - [Summary](#summary)

## Create a reusable workflow

1. Create a [new file](/../../new/main?filename=.github%2Fworkflows%2Freusable.yml&workflow_template=blank), with the name reusable.yml.
2. Set the workflow name to `Reusable workflow`.

<details>
  <summary>Solution</summary>

```YAML
name: Reusable workflow
```

</details>

3. Add a `workflow_call` trigger with an [input parameter](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call) `who-to-greet` of the type `string` that is required. Set the default value to `World`.

<details>
  <summary>Solution</summary>

```YAML
on:
  workflow_call:
    inputs:
      who-to-greet:
        description: 'The person to greet'
        type: string
        required: true
        default: World
```

</details>

4. Add a job named `reusable-job` that runs on `ubuntu-latest` that echos "Hello <input parameter>" to the console.

<details>
  <summary>Solution</summary>

```YAML
jobs:
  reusable-job:
    runs-on: ubuntu-latest
    steps:
      - name: Greet someone
        run: echo "Hello ${{ inputs.who-to-greet }}"
```

</details>

## Add an output parameter

1. Add an additional step with the id `time` that uses a workflow command to set an output parameter
named `current-time` to the current date and time (use `$(date)` for that).

<details>
  <summary>Solution</summary>

```YAML
      - name: Set time
        id: time
        run: echo "time=$(date)" >> $GITHUB_OUTPUT
```

</details>

2. Add an output called `current-time` to the `reusable-job`.

<details>
  <summary>Solution</summary>

```YAML
   outputs:
      current-time: ${{ steps.time.outputs.time }}
```

</details>

3. Add an output parameter called `current-time` to `workflow_call` and set it to the outputs of the workflow command.

<details>
  <summary>Solution</summary>

```YAML
    outputs:
      current-time:
        description: 'The time when greeting.'
        value: ${{ jobs.reusable-job.outputs.current-time }}
```

</details>


<details>
  <summary>Complete Solution</summary>

```YAML
name: Reusable workflow

on:
  workflow_call:
    inputs:
      who-to-greet:
        description: 'The person to greet'
        type: string
        required: true
        default: World
    outputs:
      current-time:
        description: 'The time when greeting.'
        value: ${{ jobs.reusable-job.outputs.current-time }}

jobs:
  reusable-job:
    runs-on: ubuntu-latest
    outputs:
      current-time: ${{ steps.time.outputs.time }}
    steps:
      - name: Greet someone
        run: echo "Hello ${{ inputs.who-to-greet }}"
      - name: Set time
        id: time
        run: echo "time=$(date)" >> $GITHUB_OUTPUT
```

</details>

## Consume the reusable workflow

1. Create a [new file](/../../new/main?filename=.github%2Fworkflows%2Freuse.yml&workflow_template=blank) with the name `reuse.yml`.
2. Set the workflowname to `Reuse other workflow` and add a manual trigger.

<details>
  <summary>Solution</summary>

```YAML
name: Reuse other workflow

on: [workflow_dispatch]
```

</details>

3. Add a job `call-workflow` that uses the reusable workflow and passes in your user name as an input parameter.

<details>
  <summary>Solution</summary>

```YAML
jobs:
  call-workflow:
    uses: ./.github/workflows/reusable.yml
    with:
      who-to-greet: '@octocat'
```

</details>

4. Add another job `use-output` that writes the output parameter `current-time` to the console. (Hint: use the needs context to access the output)

<details>
  <summary>Solution</summary>

```YAML
  use-output:
    runs-on: ubuntu-latest
    needs: [call-workflow]
    steps:
      - run: echo "Time was ${{ needs.call-workflow.outputs.current-time }}"
```

</details>

5. Run the workflow and observe the output.

## Summary

In this lab you have learned to create a reusable workflow and a workflow that consumes it. You also have learned to pass in parameters to the reusable workflow and to use output parameters in the consuming workflow.

You can continue with the [README](../README.md).
