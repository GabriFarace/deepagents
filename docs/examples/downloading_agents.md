# `examples/downloading_agents/`

> A packaging example showing that agents can be distributed as ordinary folders.

## Purpose

This example is deliberately small. It demonstrates the operational idea that a
Deep Agents app can be shared as a folder or zip containing instructions,
skills, config, and other files.

## Entry files

`README.md` explains the flow. `content-writer.zip` is the sample downloadable
agent bundle.

## Tools, subagents, and backends

The example does not define tools, backends, or subagents directly. Those live
inside whatever agent folder is downloaded and unpacked.

## Concept demonstrated

This is about distribution rather than orchestration. Because Deep Agents treats
filesystem artifacts as first-class configuration, an agent can be copied,
downloaded, unzipped, and run without a bespoke package format.
