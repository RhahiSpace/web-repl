---
title: "project overview"
date: 2023-04-16T13:32:56+02:00
mathjax: false
draft: false
tags: [programming, dream]
---

Here is a list of features/packages I want to implement.

## Goal

- Create a library for controlling spacecraft in Kerbal Space Program.
- Fly the automated spacecrafts to various heavenly bodies.
- Fly them in optimal trajectories.
- Publish the process of learning and project progress.

## Packages

Ordered by importance, descending.

- SpaceLib: The main library
- KerbalGuidance: Optimal trajectory
- MissionLib: Mission scripts
- KerbalMath: Some common math operations
- RemoteLogging: Remotely get debug logs and telemetry
- MissionControl: UI for SpaceLib
- KerbalAssembly: Build optimal rocket with given constraints
- KerbalRobotics

## Infrastructure

- Make a CI server that runs Kerbal Space Program and verifies the installation.
- Create a GitHub organization to organize the code.

## Programming

- Write programs with REPL and Notebook usage in mind.
- Write tests, and do continuous integration.
- Automatic documentation
- Mission recovery and abort modes
- Tolerance to high latency
