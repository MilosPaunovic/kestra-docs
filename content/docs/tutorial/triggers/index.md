---
order: 4
---

# Triggers

You used triggers to start a flow. They can be scheduled or triggered by an event.
Kestra core provides three types of triggers:

* [Schedule](../../developer-guide/triggers/schedule.md)
  * Trigger your flow based on a cron expression
* [Webhook](../../developer-guide/triggers/webhook.md)
  * Trigger your flow based on an HTTP request
* [Flow trigger](../../developer-guide/triggers/flow.md)
  * Trigger your flow based on another flow end

Many other triggers are available from the plugins, such as triggers based on file creation or a message queue.

## Defining triggers

The Trigger definition is close to the task definition. We define them in the `triggers` section of the flow. And as a Task, it contains an `id`, a `type`, and some properties related to his type. A Flow can have multiple Triggers.

The below triggers will start the flow every day at 10 am or when the `trigger-flow` flow ends.

```yaml
triggers:
  - id: schedule
    type: io.kestra.core.models.triggers.types.Schedule
    cron: 0 10 * * *
  - id: listenFlow
    type: io.kestra.core.models.triggers.types.Flow
    conditions:
      - type: io.kestra.core.models.conditions.types.ExecutionFlowCondition
        namespace: io.kestra.tutorial
        flowId: trigger-flow
```

## Add a trigger to your flow

Let's schedule our flow. This trigger will start our flow every monday at 10 am.

```yaml
triggers:
  - id: schedule
    type: io.kestra.core.models.triggers.types.Schedule
    cron: 0 10 * * 1
```

::collapse{title="Click here to see the full flow"}
```yaml
id: kestra-tutorial
namespace: io.kestra.tutorial
labels:
  env: PRD
description: |
  # Kestra Tutorial
  As you notice, we can use markdown here.
tasks:
  - id: download
    type: io.kestra.plugin.fs.http.Download
    uri: "https://www.data.gouv.fr/fr/datasets/r/d33eabc9-e2fd-4787-83e5-a5fcfb5af66d"
  - id: analyze-data
    type: io.kestra.core.tasks.scripts.Python
    runner: DOCKER
    dockerOptions:
      dockerHost: unix:///dind/docker.sock
      image: python
    inputFiles:
      data.csv: "{{outputs.download.uri}}"
      main.py: |
        import pandas as pd
        from kestra import Kestra
        data = pd.read_csv("data.csv", sep=";")
        data.info()
        sumOfConsumption = data['conso'].sum()
        Kestra.outputs({'sumOfConsumption': int(sumOfConsumption)})
    requirements:
      - pandas
triggers:
  - id: schedule
    type: io.kestra.core.models.triggers.types.Schedule
    cron: 0 10 * * *
```
::


::next-link{href="../flowable/"}
Now, let's dive into how to create complex workflows through flowable.
::