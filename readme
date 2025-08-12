# 📚 Testing Ground Documentation

Welcome to the comprehensive documentation for the "Testing Ground" suite—a modular React/MobX application for comparing FHIR (Fast Healthcare Interoperability Resources) data with Excel-derived ground truth, enabling data scientists and clinicians to review, annotate, and analyze AI extraction performance.

---

## 📖 Index

1. [Architecture Overview](#architecture-overview)
2. [Main Components](#main-components)
3. [Data Structures](#data-structures)
4. [State Management](#state-management)
5. [Data/API Layer](#dataapi-layer)
6. [Data Flow](#data-flow)
7. [API Endpoint Documentation](#api-endpoint-documentation)
8. [File Reference Table](#file-reference-table)

---

## Architecture Overview

This application orchestrates the retrieval, comparison, and annotation of medical data from two sources:
- **FHIR resources** (via REST API)
- **Excel files** (from Firebase Storage)

It leverages **MobX** for state management, supports real-time spreadsheet editing (with Syncfusion), and provides advanced review features including manual match overrides, note-taking with smart referencing, and AI-powered summaries.

---

## Main Components

### index.tsx

- **Entrypoint for a test run.**
- Initializes data loading (from FHIR/Excel or Firestore cache).
- Manages loading overlays and snackbar messages.
- Delegates tab rendering to `TabManager`.

---

### TabManager.tsx

- **Wraps the Dockview tab manager.**
- Registers tab components (Comparison, Excel, Notes, Overview, etc.).
- Passes context and dependencies to tabs via MobX store.

---

### TabComponents.tsx

- **Central registry for all tab panel content and tab bar components.**
- Exports tab components to Dockview.
- Handles tab bar appearance and tooltips.
- Implements robust Excel tab lazy-loading/error boundary.

---

### ComparisonTab.tsx

- **Main comparison UI for reviewing AI vs Excel data.**
- Filter by category, page, or custom range.
- Displays accuracy stats.
- Allows manual match overrides.

---

### ComparisonTable.tsx

- **Renders the main data comparison table.**
- Uses Material React Table.
- Row and cell highlighting based on match state.
- Debounced toggle for manual match status.

---

### ExcelEditorTab.tsx

- **Provides a spreadsheet editor for reviewing and correcting Excel data.**
- Loads Excel from Firebase Storage.
- Tracks unsaved changes.
- Saves changes as JSON to cloud storage.

---

### NotesTab.tsx

- **Note-taking and annotation panel with smart data references.**
- Write notes referencing data points using `$fieldName` syntax.
- Stores notes with references and metadata.

---

### OverviewTab.tsx

- **AI-powered summary and analysis of comparison results and notes.**
- Calls OpenAI LLM for conversational analysis.
- Displays stats summary and LLM output.

---

### RawDataViewer.tsx

- **Developer/debug panel for inspecting raw and parsed FHIR or Excel data.**
- Side-by-side code panels (raw vs parsed).

---

### ReferralViewerTab.tsx

- **Displays referral documents (PDF) in-app.**
- Embeds PDF Viewer.

---

## Data Structures

Below you find detailed tables and diagrams representing the key data structures in the application.

### Table: `ComparisonRow`

| Field                | Type                                     | Description                                           |
|----------------------|------------------------------------------|-------------------------------------------------------|
| `field`              | string                                   | Name of the field being compared (e.g., "firstName")  |
| `aiData`             | string \| null                           | Value extracted by AI                                 |
| `aiDataByPage`       | Array<{ value: string; pages: string[] }> | Page-specific AI data (optional)                      |
| `aiPages`            | number[] (optional)                      | Pages where AI data appears                           |
| `correctData`        | Array<{ value: string; page?: number; pages?: (number \| string)[] }> | Ground truth (Excel) data per page                    |
| `match`              | boolean                                  | Exact match status                                    |
| `caseInsensitiveMatch`| boolean                                 | Case-insensitive match status                         |
| `type`               | string                                   | Category: 'patient', 'observation', 'medication', 'condition' |
| `isBlank`            | boolean (optional)                       | If either side (AI or Excel) is blank                 |

---

### Table: `Note`

| Field      | Type                | Description                                       |
|------------|---------------------|---------------------------------------------------|
| `id`       | number              | Unique ID (usually timestamp)                     |
| `text`     | string              | Raw text of the note                              |
| `content`  | string (optional)   | Old format content                                |
| `data`     | { [key: string]: any } | Lookup dictionary for field references         |
| `timestamp`| string              | ISO date-time of when the note was saved          |
| `jobId`    | string              | The job/test run ID this note is linked to        |

---

### Table: `ComparisonDisplayData` (MobX Store fields)

| Field                        | Type                          | Description                                                 |
|------------------------------|-------------------------------|-------------------------------------------------------------|
| `fileLoaded`                 | boolean                       | Excel file loaded state                                     |
| `excelData`                  | any                           | Structured Excel data                                       |
| `fhirData`                   | any                           | FHIR patient data (structured)                              |
| `comparison`                 | ComparisonRow[]               | List of all fields compared                                 |
| `manualOverrides`            | Record<string, string>        | Manual match status overrides                               |
| `noteHistory`                | Note[]                        | Array of user notes                                         |
| `loading`                    | boolean                       | General loading state                                       |
| `fhirDataLoading`            | boolean                       | FHIR data loading state                                     |
| `excelDataLoading`           | boolean                       | Excel data loading state                                    |
| `comparisonLoading`          | boolean                       | Comparison computation state                                |
| `pageFilter`                 | string                        | Selected page filter                                        |
| `categoryFilter`             | string                        | Selected category filter                                    |
| `matchStatusFilter`          | string                        | Match status filter ('all', 'exact', 'failed', 'blank')     |
| `availableCategories`        | string[]                      | List of available categories for filtering                  |
| `availablePages`             | number[]                      | List of available pages for filtering                       |
| `currentPage`                | number                        | Pagination: current page                                    |
| `itemsPerPage`               | number                        | Pagination: items per page                                  |
| Other fields...              | ...                           | Many more for fine-grained state and UI control             |

---

### Class Diagram: Core Data Structures

```mermaid
classDiagram
    class ComparisonDisplayData {
        +fileLoaded: boolean
        +excelData: any
        +fhirData: any
        +comparison: ComparisonRow[]
        +manualOverrides: Record<string, string>
        +noteHistory: Note[]
        +loading: boolean
        +fhirDataLoading: boolean
        +excelDataLoading: boolean
        +comparisonLoading: boolean
        +pageFilter: string
        +categoryFilter: string
        +matchStatusFilter: string
        +availableCategories: string[]
        +availablePages: number[]
        +currentPage: number
        +itemsPerPage: number
        ...
    }

    class ComparisonRow {
        +field: string
        +aiData: string
        +aiDataByPage: Array<{ value: string; pages: string[] }>
        +aiPages: number[]
        +correctData: Array<{ value: string; page?: number; pages?: (number|string)[] }>
        +match: boolean
        +caseInsensitiveMatch: boolean
        +type: string
        +isBlank: boolean
    }
    
    class Note {
        +id: number
        +text: string
        +content: string
        +data: { [key: string]: any }
        +timestamp: string
        +jobId: string
    }

    ComparisonDisplayData "1" o-- "*" ComparisonRow
    ComparisonDisplayData "1" o-- "*" Note
```

---

## State Management

### ComparisonDisplayData.ts

**Central MobX store for all comparison UI state.**

- Holds all FHIR & Excel data, comparison results, manual overrides, notes, UI state, loading flags, and more.
- Orchestrates data loading, auto-save, and computed properties for filtering and match state.

See the [ComparisonDisplayData](#class-diagram-core-data-structures) class diagram above for relationships.

---

## Data/API Layer

### FhirReader.js

**Handles all API requests to the FHIR backend to retrieve resource data for a test run/job.**

#### API Endpoint

```api
{
    "title": "Fetch FHIR Resources By Job",
    "description": "Fetches all FHIR resource entries associated with a specific job ID",
    "method": "GET",
    "baseUrl": "https://api.dovaxis.com",
    "endpoint": "/FHIR/resources-by-job/{jobId}",
    "headers": [],
    "queryParams": [],
    "pathParams": [
        {
            "key": "jobId",
            "value": "The job ID for which to fetch resources",
            "required": true
        }
    ],
    "bodyType": "none",
    "requestBody": "",
    "responses": {
        "200": {
            "description": "FHIR Bundle of resource entries",
            "body": "{\n  \"entry\": [\n    { \"resource\": { ... } }, ...\n  ]\n}"
        },
        "404": {
            "description": "Job not found",
            "body": "{\n  \"error\": \"Job not found\"\n}"
        }
    }
}
```

---

## Data Flow

### Data Initialization & Load Process

```mermaid
flowchart TD
    Start([User opens job in TestingGround])
    LoadRun[TestRig.testRuns.getById(jobId)]
    InitData[ComparisonDisplayData.initializeData]
    CheckFS[Check Firestore for existing results]
    HydrateFS[Hydrate MobX store from Firestore snapshot]
    FetchFHIR[Fetch FHIR resources (FhirReader.js)]
    ParseFHIR[Parse FHIR: patients, obs, conditions, meds]
    LoadExcel[Load Excel from Firebase Storage]
    ParseExcel[Parse Excel to JSON, structure data]
    BuildComparison[Run createComparison (build rows)]
    PopulateStore[Populate MobX store]
    PersistCache[Auto-save to Firestore/localStorage]

    Start --> LoadRun --> InitData --> CheckFS
    CheckFS -- Found --> HydrateFS --> PersistCache
    CheckFS -- Not Found --> FetchFHIR --> ParseFHIR --> LoadExcel --> ParseExcel --> BuildComparison --> PopulateStore --> PersistCache
```

**Explanation:**
- When the page loads, it tries to load comparison data for a given `jobId`.
- If Firestore has existing results, it hydrates the MobX store from the saved snapshot.
- Otherwise, it fetches FHIR resources and Excel data, structures them, builds comparison rows, and populates state.
- The result is persisted for quick reloads (Firestore + localStorage).

---

## API Endpoint Documentation

### FHIR Resource Fetching

See [FhirReader.js](#fhirreaderjs) above for the API block.

---

## File Reference Table

| File                        | Main Purpose                                                                                  |
|-----------------------------|----------------------------------------------------------------------------------------------|
| **index.tsx**               | Entry point, orchestrates data load, initializes MobX store, renders TabManager              |
| **TabManager.tsx**          | Manages Dockview tab system, passes context/props to tabs                                    |
| **TabComponents.tsx**       | Registers all tab components and custom tab bar UIs for Dockview                             |
| **ComparisonTab.tsx**       | Main table UI for reviewing and filtering comparison data                                    |
| **ComparisonTable.tsx**     | Renders comparison table with match toggles, color-coding, page/category filtering           |
| **ExcelEditorTab.tsx**      | Spreadsheet editor, handles Excel JSON upload/save, triggers comparison rebuild              |
| **NotesTab.tsx**            | Notes panel, supports $field references, saves notes/history with data references            |
| **OverviewTab.tsx**         | AI-powered summary of comparison, LLM integration, stats overview                           |
| **RawDataViewer.tsx**       | Debug panel for inspecting raw and parsed FHIR/Excel data in JSON                            |
| **ReferralViewerTab.tsx**   | Renders PDF viewer for referral documents                                                    |
| **ComparisonDisplayData.ts**| MobX store: centralizes all app state, data loading, saving, filtering, and computed logic   |
| **FhirReader.js**           | FHIR API request logic, fetches all resources for a jobId                                   |
| **ComparisonData.ts**       | [Empty placeholder]                                                                          |

---

# 🎉 End of Documentation

This documentation should serve as a strong resource for both users and developers working with the Testing Ground application. For more implementation insights, consult the MobX store (`ComparisonDisplayData.ts`) and the registered components in `TabComponents.tsx`. Expand any section for in-depth understanding!
