# Sentinel — Class Diagram

This document provides a class diagram illustrating the Object-Oriented design and structure of the Sentinel application.

![Sentinel Class Diagram](./class-diagram.png)
---

## 🔗 Interactive Mermaid Source

```mermaid
classDiagram
    direction TB

    namespace Contracts {
        class IExecutable {
            <<interface>>
            +execute(plan PlanSchema) ExecutionResult
            +rollback() void
        }
        class IUndoable {
            <<interface>>
            +undo() void
            +canUndo() boolean
        }
    }

    namespace ExecutorLayer {
        class Executor {
            -dryRun : boolean
            -executedOps : UndoOperation[]
            +constructor(dryRun boolean)
            +execute(plan PlanSchema) ExecutionResult
            +rollback() void
            -validatePlan(plan PlanSchema) void
            -logAction(entry ExecutionLogEntry) void
        }
        class UndoOperation {
            -taskId : string
            -actionType : ActionType
            -originalPath : string
            -newPath : string
            -canUndoFlag : boolean
            -undoReason : string
            +undo() void
            +canUndo() boolean
        }
        class ExecutionResult {
            -taskId : string
            -totalActions : number
            -successfulActions : number
            -failedActions : number
            -rollbackPerformed : boolean
            -executionLogs : ExecutionLogEntry[]
            +isSuccessful() boolean
            +getSummary() string
        }
        class ExecutionLogEntry {
            -id : number
            -taskId : string
            -timestamp : Date
            -actionType : ActionType
            -sourcePath : string
            -destinationPath : string
            -status : string
            -errorMessage : string
        }
    }

    namespace PersistenceLayer {
        class PreferencePattern {
            -patternType : string
            -sourcePattern : string
            -destinationPattern : string
            -confidence : number
            -occurrenceCount : number
            -approvalCount : number
            +record(approved boolean) void
            +updateConfidence() void
        }
        class UserDecision {
            -taskId : string
            -actionType : ActionType
            -sourcePath : string
            -decision : string
            -originalSuggestion : string
            -reasonCode : string
        }
        class TaskRecord {
            -taskId : string
            -userPrompt : string
            -mode : TaskMode
            -createdAt : Date
            -status : TaskStatus
            -undoAvailable : boolean
            +constructor(taskId, prompt, mode)
            +updateStatus(status TaskStatus) void
            +markUndoAvailable(flag boolean) void
        }
    }

    namespace ScannerLayer {
        class ScanResult {
            -rootPath : string
            -files : FileMetadata[]
            -ignoredCount : number
            -errors : string[]
            -scannedAt : Date
            +constructor(rootPath)
            +getFiles() FileMetadata[]
            +getIgnoredCount() number
            +getErrors() string[]
        }
        class FileMetadata {
            -path : string
            -name : string
            -extension : string
            -sizeBytes : number
            -fileType : FileType
            -isHidden : boolean
            -hash : string
            +constructor(path, name, ext, size, type)
            +getPath() string
            +getFileType() FileType
            +getHash() string
        }
    }

    namespace PlannerLayer {
        class PlanSchema {
            -taskId : string
            -scopePath : string
            -foldersToCreate : string[]
            -actions : PlanAction[]
            -ambiguousFiles : AmbiguousFile[]
            -summary : string
            +getActions() PlanAction[]
            +getAmbiguousFiles() AmbiguousFile[]
            +getSummary() string
        }
        class PlanAction {
            -type : ActionType
            -sourcePath : string
            -destinationPath : string
            -reason : string
            -confidence : number
            +constructor(type, src, dest, reason, conf)
            +validate() void
            +getConfidence() number
        }
        class AmbiguousFile {
            -path : string
            -suggestedAction : ActionType
            -reason : string
        }
    }

    namespace Enumerations {
        class FileType {
            <<enumeration>>
            DOCUMENT
            IMAGE
            VIDEO
            AUDIO
            ARCHIVE
            EXECUTABLE
            CODE
            UNKNOWN
        }
        class ActionType {
            <<enumeration>>
            MOVE
            RENAME
            DELETE
            CREATE_FOLDER
            SKIP
        }
        class TaskStatus {
            <<enumeration>>
            SCANNING
            PLANNING
            REVIEW
            EXECUTING
            COMPLETED
            FAILED
        }
        class TaskMode {
            <<enumeration>>
            FILE_ORGANIZATION
            PC_CLEANUP
            MEDIA_SORT
            SAFE_DELETE
            ASK_AI
        }
    }

    %% Relationships
    IExecutable <|.. Executor : implements
    IUndoable <|.. UndoOperation : implements
    Executor ..> ExecutionResult : produces
    Executor ..> PlanSchema : reads
    Executor --> UndoOperation : references

    ExecutionResult *-- ExecutionLogEntry : owns
    ExecutionLogEntry ..> ActionType : logs

    ScanResult *-- FileMetadata : owns
    FileMetadata ..> FileType : classified as

    PlanSchema *-- PlanAction : owns
    PlanSchema *-- AmbiguousFile : flags
    PlanAction ..> ActionType : type
    UndoOperation ..> ActionType : type

    TaskRecord --> ScanResult : stores scan
    TaskRecord --> PlanSchema : stores plan
    TaskRecord ..> TaskStatus : has status
    TaskRecord ..> TaskMode : has mode
    TaskRecord ..> ExecutionResult : records

    UserDecision --> TaskRecord : belongs to
    PreferencePattern ..> UserDecision : learned from
```

---

## 🗂️ Overview

The class diagram outlines the core components and their relationships within Sentinel, including:

-   **Interface Layer**: Classes responsible for user interactions (`CLIHandler`, `WebAPIHandler`).
-   **Core Agent**: The central orchestration classes managing file scanning, classification, AI planning, safety validation, and execution operations (`SentinelOrchestrator` and its dependencies).
-   **Data Models**: The structural representation of data entities handling files, action plans, and execution logs.
-   **Providers**: Abstractions for interacting with external AI runtimes like `OllamaClient`.

*The interactive diagram provides a visual high-level overview of the OOP structure, avoiding the need for external image files.*
