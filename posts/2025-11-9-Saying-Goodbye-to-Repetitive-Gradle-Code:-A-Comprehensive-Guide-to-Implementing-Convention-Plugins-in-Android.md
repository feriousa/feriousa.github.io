---
title: "Saying Goodbye to Repetitive Gradle Code"
date: 2023-10-27T10:00:00+03:30
draft: false
description: "A Comprehensive Guide to Implementing Convention Plugins in Android (Senior Level)"
tags: ["Android", "Gradle", "Kotlin", "Architecture", "Build System"]
categories: ["Development"]
---

# Saying Goodbye to Repetitive Gradle Code: A Comprehensive Guide to Implementing Convention Plugins in Android (Senior Level)

If you have worked on large, multi-module Android projects, you have likely faced this nightmare: ten repetitions of `compileSdk = 34`, identical `defaultConfig` blocks copied everywhere, and updating the Kotlin version requiring changes across 20 different files.

Managing `build.gradle` files at scale, without a proper strategy, quickly descends into "Maintenance Hell."

The modern, standard solution from Google for this problem (utilized in projects like *Now in Android*) is **Gradle Convention Plugins**. In this post, we will explore not just the "why," but the "how" of implementing this architecture in the most professional way possible (using Kotlin classes).

### Why Convention Plugins?

Instead of telling every module *how* to be built (e.g., "set compileSdk to 34"), you define *what* that module is. You create a "Convention":

* "This module is an **Android Library**."
* "This module is an **Android Feature** that also uses **Compose**."

By doing this, your build logic becomes centralized. The benefits are clear:

1.  **Single Source of Truth (SSOT):** A change to `minSdk` is applied to the entire project in seconds.
2.  **DRY (Don't Repeat Yourself):** Elimination of hundreds of lines of repetitive Gradle boilerplate code.
3.  **Composition:** Plugins stack like Lego blocks (e.g., applying a Compose plugin on top of an Android Library plugin).

---

### The Implementation Dilemma: Script vs. Class

There are two main ways to implement these plugins:

1.  **Precompiled Script Plugins (`.gradle.kts` files):** A faster, simpler method that provides direct, magical access to the Version Catalog (`libs`). Great for getting started.
2.  **Kotlin Classes (`class : Plugin<Project>`):** A more professional, type-safe, and testable approach. Large, serious projects usually gravitate towards this direction.

**In this guide, we will implement the second method (Kotlin Classes).** Although slightly more complex to set up, it provides greater control and robustness to your build infrastructure in the long run.

---

### The Implementation Roadmap

We will create a separate module named `build-logic` whose sole responsibility is to hold and compile our build logic.



#### Prerequisite: Version Catalog setup

Ensure your `gradle/libs.versions.toml` file is organized. The crucial point is that the main plugins (AGP and Kotlin) must also be defined as **libraries** so the `build-logic` module can utilize their classes as dependencies:

```toml
# gradle/libs.versions.toml
[versions]
agp = "8.2.2"
kotlin = "1.9.22"

[libraries]
# These are vital for build-logic dependencies
android-gradlePlugin = { group = "com.android.tools.build", name = "gradle", version.ref = "agp" }
kotlin-gradlePlugin = { group = "org.jetbrains.kotlin", name = "kotlin-gradle-plugin", version.ref = "kotlin" }
# ... other libraries
