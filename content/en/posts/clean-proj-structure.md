+++
title = "Clean Java Project Structure"
description = "Sample project structure proposal following the Clean architecture guidelines"
date = 2021-12-16T11:55:24+05:30
featured = true
draft = true
comment = true
toc = true
reward = true
diagram = true
categories = [
  "programming"
]
tags = [
  "java",
  "best-practice"
]
series = [
  "Manual"
]
+++

### Domain

Core of the business logic not involving specific toolkits and frameworks

- Entity
- Exception
- IOPort
- Usecase

{{< note title="NOTE" >}}
The domain should be packaged into a single, independent library jar/archive that can be consumed by the Adapters.
{{< /note >}}

### Adapters

Providing specific implementations using a minimal set of libraries

- DataMapper
- RepoDriver
- Controller

{{< note title="NOTE" >}}
The DataMapper, RepoDriver and Controller each should be packaged into independent library jars / archives. They can be
consumed by the Applications.
{{< /note >}}

### Applications

Final runnables or services consuming the domain via adapters, may be implemented using different toolkits and frameworks.

WebAppX:

- rpcService
- apiService
- appConfig
- appController
- mainApplication

WebAppY:

- rpcService
- apiService
- appConfig
- appController
- mainApplication

{{< note title="NOTE" >}}
Each application should be packaged into independent runnable jar file. Its runtime behaviour (ex., running only a
selected set of microservices) can be controlled by the CLI options or via the environment configuration / variables.
{{< /note >}}
