---
title: "Demo"
linkTitle: "Demo"
weight: 40
description: >
  KAI Server demo project.
---


The [Kai Server demo Email Classificator](https://github.com/konstellation-io/demo-email-classificator)
is a training exercise and demo repo designed to introduce new users to the usage and development
of KRT projects. You can use the training exercise to build and deploy your first KRT project and
also to reverse engineer it and learn how to develop your own projects.

## Introduction

This example consists of a workflow called _Classificator_ that will classify 20,000 mocked emails.
The workflow is made up of 6 different nodes, each one in charge of a different process through the pipeline:

- **Entrypoint**: A node provided by KAI organization, provides service access point to GRPC requests.
- **Exitpoint**: Defined by the user, is in charge of giving a single and final response to the incoming request.
                 Also for this example, it will log each time 50 emails are processed.
- **ETL**: This node will read mocked emails from a predefined CSV file and output them to be classified.
- **Email Classificator**: Mocks a trained AI model that classifies emails in different categories.
- **Stats Storer**: Stores the email's given classification as a metric.
- **Repairs Handler**: Stores in BD emails that have been marked as in the _repairs_ category.

### How to build

For the user's convenience there is a _build script_ placed within the _scripts_ folder in the repo's
root, for some usage examples consult the repo's `readme` file.

### How to deploy

You can find information about how to upload built krt files in
[Creating new versions](../30_user/30_versions/#creating-new-versions).

### How to use

Once deployed the generated KRT file, you can open an entrypoint to your local pod. To easy up usage,
there is also a _test script_ inside the `scripts`, for some usage examples consult the repo's `readme` file.

### Descriptor example

{{< imgproc kai-training-example Resize "1000x" />}}

### Testing example

{{< imgproc kai-training-result-example Resize "1000x" />}}

### Results example

For a much clearer data visualization of the results obtained you can check the version's dashboard.  
Here is an example for 500 processed messages:

{{< imgproc results_dashboard Resize "1500x" />}}
