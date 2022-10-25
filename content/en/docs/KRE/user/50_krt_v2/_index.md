---
title: "KRT V2"
linkTitle: "KRT V2"
weight: 50
description: >
  Understading how the new version V2 of KRT works
---

- [Branched workflows](#branched-workflows)
- [Reactive approach](#reactive-approach)
- [Exitpoint](#exitpoint)
- [KRT YAML File](#krt-yaml-file)
- [Diagram of an architecture defined by KRT V2](#diagram-of-an-architecture-defined-by-krt-v2)

## Branched workflows

The main purpose of KRT V2 is to bring branched workflows on board. This means nodes are now capable
of subscribing to several other nodes at the same time, thus workflows are now capable of branching out.

It is advised to previously read and fully understand [KRT](40_krt) before moving on to V2, as this page focuses on explaining V2 related changes and new concepts.

## Reactive approach

Nodes will publish their results onto their own subject, is up to other nodes whether to subscribe and process its messages or not. This allows for organic growth as nodes can be developed then hooked up to other nodes freely.

### Multiple responses

Nodes now also have the ability to publish several to none messages from the same request. So now the _handler_ contract should look like this:

- `handler(ctx *kre.HandlerContext, data *any.Any) (proto.Message, error)`

Messages will be published using the function _SendOutput_ provided by the context.

### Filter messages

Nodes can choose to which subtopic from their own subject they publish to. At the same time, they can choose to take messages from any subtopic of another node. This allows users to filter out messages and reduce stress on nodes.

## Exitpoint

The exitpoint is a user defined node and can take up any name, however it has to be later assigned in the krt.yaml file to the exitpoint attribute. Users can only define one exitpoint as this is the final node to which the entrypoint will subscribe.

An exitpoint is required in all workflows. As the exitpoint is the only node that will communicate back to the entrypoint, it is recommended to handle merge strategies in it. For example, if users wish to implement "early exit" and "early reply" strategies, they should subscribe the exitpoint to nodes emitting these types of messages.

## KRT YAML file

The KRT YAML file serves the same porpoise as its predecessor. However, now nodes are declared and defined inside the workflows, an exitpoint must be described per workflow and a _v2_ tag must be added to the _krtVersion_ field.

Also, users must define node's subscriptions inside each node, subscription names must match declared node's names. _'entrypoint'_ name is reserved to the entrypoint KRE provides, in order for a node to subscribe to the entrypoint just type the reserved name in its subscriptions.

Learn about the fields in the [KRT YAML specs]({{< relref "docs/KRE/user/50_krt_v2/specs" >}}).

Example below is directly taken from our krt v2 demo repo:

```yaml
version: classificator-v1
krtVersion: v2 # new tagged krt version
description: Demo email classificator for branching features.
entrypoint:
  proto: public_input.proto
  image: konstellation/kre-entrypoint:latest
config:
  variables:
workflows:
  - name: classificator
    entrypoint: Classificator
    exitpoint: exitpoint # name must match node's
    nodes:
      - name: etl
        image: konstellation/kre-py:latest
        src: src/etl/main.py
        subscriptions:
          - "entrypoint"

      - name: email-classificator
        image: konstellation/kre-py:latest
        src: src/email_classificator/main.py
        subscriptions:
          - "etl"

      - name: repairs-handler
        image: konstellation/kre-go:latest
        src: bin/repairs_handler
        subscriptions:
          - "email-classificator.repairs" #consume from a subtopic

      - name: stats-storer
        image: konstellation/kre-go:latest
        src: bin/stats_storer
        subscriptions:
          - "email-classificator"

      - name: exitpoint
        image: konstellation/kre-go:latest
        src: bin/exitpoint
        subscriptions:
          - "etl"
          - "stats-storer"
```

## Diagram of an architecture defined by KRT V2

Here is an example of an architecture defined by a KRT V2 file, this example is taken from our KRT V2 demo repo:

{{< imgproc diagram_krt_v2 Resize "1200x" />}}
