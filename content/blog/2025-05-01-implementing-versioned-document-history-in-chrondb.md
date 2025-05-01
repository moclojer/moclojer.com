---
title: "Implementing Versioned Document History in ChronDB"
date: 2025-05-01
tags: ["chrondb", "git database", "document history", "version control", "time-travel"]
images: ["/blog/implementing-versioned-document-history-in-chrondb.png"]
url: "/blog/implementing-versioned-document-history-in-chrondb"
description: "Learn how ChronDB implements Git-based document versioning with time-travel capabilities, allowing document history retrieval, point-in-time access, and chronological version restoration without losing historical data."
author: "<a href='https://github.com/avelino' alt='Thiago Avelino' target='_blank'>Avelino</a>"
---

![Implementing Versioned Document History in ChronDB](/blog/implementing-versioned-document-history-in-chrondb.png)

[ChronDB](https://github.com/moclojer/chrondb) is a document-oriented database built with time-travel capabilities at its core. While many databases offer point-in-time recovery, ChronDB takes this a step further by making version history a first-class feature.

The key features we implemented are:

1. **Document History Retrieval**: Get the complete history of a document
2. **Point-in-Time Access**: Access a document at any specific commit in history
3. **Chronological Version Restoration**: Restore previous versions while maintaining full history

Let's explore these features and the Git internals that make them possible.

## The Git Foundation

When we designed ChronDB, we chose Git as our storage backend for several reasons:

- **Built-in versioning**: Git is designed for version control
- **Distributed nature**: Enables resilient replication
- **Mature technology**: Battle-tested and optimized over years
- **Rich commit metadata**: Provides timestamps, authors, and messages

We created a storage layer that saves documents as JSON files in a Git repository, with each change automatically committed.

## Implementing Document History

The first feature we implemented was document history retrieval. This allows users to see all versions of a document, including metadata like who made each change and when.

Our implementation leverages several key Git concepts:

1. **Git Log Command**: We use JGit's `LogCommand` to retrieve all commits that modified a specific document path.
2. **RevWalk and TreeWalk**: These JGit classes `RevWalk` and `TreeWalk` allow us to navigate through the commit history and the file tree structure within each commit.
3. **Path Filtering**: We filter the log results to focus only on commits that affected the specific document we're interested in.
4. **Content Extraction**: For each commit, we extract the document content as it existed at that point in time, along with rich metadata like timestamps, commit messages, and author information.

The result is a chronologically ordered list of document versions, starting with the most recent, that provides a complete audit trail of all changes.

## Accessing Documents at Specific Points in Time

Next, we implemented the ability to retrieve a document as it existed at a specific commit. This lets users access historical versions without changing the current state.

To accomplish this, we leverage several Git internals:

1. **ObjectId Resolution**: We convert the commit hash string to Git's internal `ObjectId` representation.
2. **Commit Parsing**: Using `RevWalk`, we parse the specified commit to access its tree structure.
3. **Tree Navigation**: With `TreeWalk`, we navigate to the document's path in that commit's file tree.
4. **Content Retrieval**: We extract the document content from the Git object store and deserialize it.

This approach allows us to effectively time-travel to any point in a document's history and retrieve its exact state at that moment, without modifying the current version.

## Chronological Version Restoration

The most interesting feature is document restoration. Unlike traditional Git operations like `git reset` or `git revert`, which would modify or undo history, we wanted to maintain the full chronological history when restoring a document.

Our solution leverages Git's commit model in a novel way:

1. **Retrieve Historical Content**: First, we use the techniques described above to get the document content at the specified historical commit.
2. **Create New Commit**: Instead of resetting or modifying history, we create a completely new commit with the content from the historical version.
3. **Restoration Metadata**: The new commit includes a special message indicating this is a restoration and which version was restored, preserving this information in the audit trail.
4. **Parenting Structure**: The new commit has the current `HEAD` as its parent, maintaining the chronological commit history.

This approach maintains a complete audit trail of changes, allowing users to see when a document was restored and from which version, without losing any historical information.

## Git Internals - How It Works

Let's dive deeper into the Git internals that make these features possible:

### Document Storage

In ChronDB, documents are stored as JSON files in a Git repository. We use a structured path convention based on document ID and table name to organize files within the repository.

To ensure proper file organization:

1. **Path Encoding**: We encode document IDs to ensure they're valid filesystem paths, replacing problematic characters.
2. **Table-Based Organization**: Documents are grouped by table in separate directories, creating a natural organizational structure.
3. **JSON Serialization**: Each document is serialized to JSON before storage, making it human-readable and easily parseable.

### Commit Creation

When a document is saved or restored, we create a Git commit to record the change. Our implementation uses a "virtual" commit process that works directly with Git's internal structures:

1. **In-Memory Index Creation**: Rather than using Git's standard staging area (index), we create an in-memory index containing just the document being modified.
2. **Tree Building**: From this index, we generate a new tree structure representing the repository state.
3. **Commit Building**: We create a new commit object using `CommitBuilder`, setting appropriate author and committer information, timestamp, message, and the new tree.
4. **Reference Update**: Finally, we update the branch reference to point to the new commit, making it the new `HEAD`.

This approach avoids filesystem I/O for staging and provides fine-grained control over the commit process.

### Traversing History

To retrieve a document's history, we leverage Git's powerful history traversal capabilities:

1. **Log Command Initialization**: We start with JGit's `LogCommand`, configuring it to follow a specific file path.
2. **Commit Iteration**: The log command gives us an iterator over all commits that affected the specified document.
3. **Commit Metadata Extraction**: For each commit, we extract rich metadata including commit ID, author, timestamp, and message.
4. **Content Extraction Process**:
   - We use `RevWalk` to parse the commit and access its tree
   - `TreeWalk` navigates to the specific document path within that tree
   - The file content is loaded from Git's object store using `ObjectLoader`
   - The JSON content is deserialized into a document object

5. **Result Assembly**: All this information is combined into a comprehensive history record.

### Accessing Specific Versions

To access a document at a specific commit, we use a precise navigation process:

1. **Hash Normalization**: First, we normalize the commit hash, handling potential formatting variations.
2. **ObjectId Resolution**: The normalized hash is converted to Git's internal `ObjectId`.
3. **Commit Lookup**: We use `RevWalk` to locate and parse the specified commit.
4. **Path Navigation**: With `TreeWalk`, we navigate to the document's path within that commit's tree.
5. **Content Extraction**: We extract the blob content with `ObjectLoader` and deserialize it to obtain the document as it existed at that point in time.

### Version Restoration

The chronological restoration process combines several Git techniques:

1. **Historical Content Retrieval**: We use the version access techniques to get the document's content at the target commit.
2. **New Commit Creation**: Rather than modifying history, we create an entirely new commit containing the historical content.
3. **Commit Message**: We use a special commit message format that indicates this is a restoration and which version was restored.
4. **Branch Update**: The branch reference is updated using `RefUpdate` to point to this new commit, making it the current version while preserving the complete history.

This approach is fundamentally different from Git's built-in `revert` or `reset` operations, as it preserves the complete chronological history while still restoring the document to a previous state.

## Testing Our Implementation

We wrote comprehensive tests to verify our implementation behaves correctly. Our test suite verifies several key aspects:

1. **Document Creation and Updates**: Ensuring basic CRUD operations work correctly.
2. **History Retrieval**: Verifying we can retrieve a complete and accurate document history.
3. **Point-in-Time Access**: Confirming we can access a document as it existed at any previous commit.
4. **Version Restoration**: Testing that we can restore a document to a previous version.
5. **History Preservation**: Validating that after restoration, the complete history is preserved with proper chronology.
6. **Metadata Accuracy**: Ensuring commit messages correctly indicate the nature of each change, including restorations.

Our tests demonstrate that our approach successfully maintains a complete audit trail of all document changes, including restorations, making ChronDB suitable for applications with strict compliance and auditing requirements.

## Conclusion

By implementing document version history with chronological restoration in ChronDB, we've created a powerful way to track changes and revert to previous states without losing any history. This approach provides a complete audit trail for compliance purposes and enables advanced time-travel capabilities.

The use of Git as our storage backend has proved to be an excellent choice, providing a solid foundation for versioning with minimal custom code. The ability to push/pull changes also enables simple replication across instances.

Future enhancements could include:

- Diffing between versions to show what changed
- Branching for experimental document changes
- Support for merging concurrent changes to the same document

We hope this deep dive into ChronDB's implementation has been insightful. The full source code is available at [github.com/moclojer/chrondb](https://github.com/moclojer/chrondb).

[Here](https://github.com/moclojer/chrondb/pull/27) you can see the initial implementation of this feature.
