---
title: "ChronDB Relicensing: From MIT to GPLv3"
date: 2025-05-02
tags: ["chrondb", "licensing", "GPLv3", "MIT", "open source"]
images: ["/blog/chrondb-relicensing-from-mit-to-gplv3.png"]
url: "/blog/chrondb-relicensing-from-mit-to-gplv3"
description: "ChronDB is changing its license from MIT to GPLv3 to ensure legal compliance with Git and JGit dependencies. Learn how this affects current and future users of the Git-based document database."
author: "<a href='https://github.com/avelino' alt='Thiago Avelino' target='_blank'>Avelino</a>"
---

![ChronDB Relicensing: From MIT to GPLv3](/blog/chrondb-relicensing-from-mit-to-gplv3.png)

We officially announce that ChronDB will be relicensed from the MIT license to the GNU General Public License version 3 (GPLv3). This change is necessary to ensure legal compliance with the project's dependencies and maintain the integrity of the free software ecosystem.

## Why change the license?

After a more in-depth analysis of ChronDB's architecture, we identified a complex licensing conflict. ChronDB uses the Git architecture as the fundamental basis for its storage system, as described in our documentation:

> *Chronological **key/value** Database storing based on database-shaped `git` (core) architecture and Lucene for indexing.*

Our project faces a license compatibility challenge due to the following conditions:

1. Git is licensed under GPLv2-only (without the "or later" clause), a strong copyleft license.
2. ChronDB uses **JGit** as its Git interaction layer.
3. JGit is licensed under the **Eclipse Distribution License v1.0 (EDL-1.0)**, which is only compatible with GPLv3 — not with GPLv2.

This situation creates a potential license conflict: we cannot legally combine GPLv2-only (Git) with EDL (JGit) in a single derivative work.

## Legal implications of the change

### Understanding the licenses

**MIT License (previous):**

- Extremely permissive
- Allows use in proprietary software without restrictions
- Requires only the maintenance of the copyright notice

**GPLv3 License (new):**

- Strong copyleft license
- Requires that derivative works also be licensed under GPLv3
- Ensures that software freedoms are preserved in all derivatives
- Requires source code to be made available to users
- Includes protections against tivoization (hardware restrictions)
- Provides more robust patent protections than GPLv2

### Compatibility with dependencies

The change to GPLv3 ensures that ChronDB is in full compliance with the legal obligations related to the use of JGit (EDL-1.0) and its integration patterns with Git. This avoids:

- Potential license violations
- Legal risks for contributors and users
- Ambiguity about the project's legal status

## What does this mean for ChronDB users?

### For current users

- **Proprietary software**: If you are using ChronDB in closed proprietary software, you will need to evaluate your integration to ensure compliance with GPLv3.
- **Open source projects**: If your project is already open source with a GPLv3-compatible license, there will be no significant changes.

### For new users

By adopting ChronDB, you agree to the terms of GPLv3, which means:

1. Your software using ChronDB must be compatible with GPLv3
2. The source code of modifications to ChronDB must be made available
3. You must include the full text of the GPLv3 license with your distribution

It's important to note that GPLv3 has some additional requirements compared to GPLv2, particularly regarding hardware restrictions and patents.

## Why GPLv3 and not GPLv2?

Initially, we considered GPLv2 due to integration with Git (GPLv2-only). However, a deeper analysis revealed:

1. ChronDB uses JGit (EDL-1.0) as its main interface for interacting with Git
2. JGit's EDL-1.0 license is compatible with GPLv3, but not with GPLv2
3. JGit has been actively maintained for years under EDL without legal disputes, despite deeply interacting with Git repositories

Considering these factors, GPLv3 emerges as the most compatible and future-proof solution for ChronDB.

## Alternatives considered

We evaluated various options before deciding on GPLv3:

1. Keeping the MIT license, isolating all Git interactions to external CLI calls
2. Migrating to GPLv2, removing the JGit dependency
3. Implementing an abstraction layer that clearly separated MIT code from code with license restrictions

However, due to the fundamental nature of JGit integration in our architecture, we determined that relicensing to GPLv3 is the most appropriate and transparent approach.

## Transition timeline

Starting May 2025:

1. All ChronDB source code will be relicensed under GPLv3
2. The LICENSE file will be updated in the repository
3. File headers will be updated to reflect the new license
4. Documentation and websites will be updated
5. Request retroactive approval from previous contributors, if any

## Conclusion

This licensing change reflects our commitment to:

1. **Legal compliance**: Ensuring that ChronDB is in full compliance with licensing obligations
2. **Transparency**: Being clear with our community about the legal implications of using ChronDB
3. **Free software philosophy**: Aligning our project with the principles of free software that underpin many of the technologies we rely on

We believe this change will strengthen ChronDB in the long term, ensuring a solid legal foundation for the continued growth and evolution of the project.

## Additional resources

- [Full text of the GPLv3 license](https://www.gnu.org/licenses/gpl-3.0.html)
- [Details of the licensing issue (GitHub Issue #28)](https://github.com/moclojer/chrondb/issues/28)
- [FAQ about GPLv3 from the Free Software Foundation](https://www.gnu.org/licenses/gpl-3.0-faq.html)
- [About the Eclipse Distribution License (EDL-1.0)](https://www.eclipse.org/org/documents/edl-v10.php)
- [JGit Project](https://www.eclipse.org/jgit/)

For specific questions about how this license change affects your use case, we recommend opening an [issue in chrondb](https://github.com/moclojer/chrondb/issues/new) and consulting a legal expert.
