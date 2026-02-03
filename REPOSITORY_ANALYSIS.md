# NDI-Matlab Repository Analysis

## Overview

**NDI (Neuroscience Data Interface)** is a comprehensive MATLAB toolbox for managing neuroscience research data. It provides a cross-platform, standardized interface for reading diverse neuroscience data formats, storing analysis results, and enabling FAIR-compliant (Findable, Accessible, Interoperable, Reusable) data management.

**Key Statistics:**
- ~747 MATLAB files (.m)
- ~210 JSON configuration/schema files
- 201+ class definitions
- 83 database document schemas
- Minimum MATLAB Release: R2021b
- License: CC BY-NC-SA 4.0

---

## External Helper Repositories (Dependencies)

NDI-Matlab depends on the following external repositories to operate:

### Core Dependencies (from `requirements.txt`)

| Repository | Purpose | Critical? |
|------------|---------|-----------|
| **[vhlab_vhtools](https://github.com/VH-lab/vhlab_vhtools)** | Startup/initialization management, provides `vhtools_startup.m` that auto-initializes NDI | Yes |
| **[vhlab-toolbox-matlab](https://github.com/VH-lab/vhlab-toolbox-matlab)** | Core utility library (`vlt.*` namespace) - provides data structures, file I/O, string manipulation, struct operations | Yes |
| **[vhlab-library-matlab](https://github.com/VH-lab/vhlab-library-matlab)** | Additional library utilities for neuroscience analysis | Yes |
| **[vhlab-thirdparty-matlab](https://github.com/VH-lab/vhlab-thirdparty-matlab)** | Third-party dependencies bundled together (e.g., sigTOOL for Spike2 files) | Yes |
| **[vhlab-NewStim-matlab](https://github.com/VH-Lab/vhlab-NewStim-matlab)** | Visual stimulus presentation and metadata format support | Recommended |
| **[DID-matlab](https://github.com/VH-lab/DID-matlab)** | Document/ID database system - provides `did.document`, `did.query`, `did.database` base classes | Yes |
| **[NDR-matlab](https://github.com/VH-lab/NDR-matlab)** | Neuroscience Data Reader - universal file format reader for multiple electrophysiology formats | Yes |
| **[mksqlite](https://github.com/a-ma72/mksqlite)** | SQLite MEX interface for MATLAB database backend | Yes |
| **[NDI-compress-matlabp](https://github.com/Waltham-Data-Science/NDI-compress-matlabp)** | Data compression utilities for ingesting raw data | Yes |
| **[Catalog](https://github.com/ehennestad/Catalog)** | File catalog system for managing file references | Recommended |
| **[Violinplot-Matlab](https://github.com/bastibe/Violinplot-Matlab)** | Visualization - violin plot generation | Optional |
| **[openMINDS_MATLAB](https://github.com/openMetadataInitiative/openMINDS_MATLAB)** | OpenMINDS metadata standard integration (also available as File Exchange addon `fex://134212`) | Recommended |

### MATLAB Toolbox Dependencies

**Required Toolboxes:**
- Control System Toolbox
- Curve Fitting Toolbox
- Image Processing Toolbox
- Optimization Toolbox
- Signal Processing Toolbox
- Statistics and Machine Learning Toolbox

**Recommended Toolboxes:**
- Database Toolbox

### External Services

- **NDI Cloud API** (`api.ndi-cloud.com` / `dev-api.ndi-cloud.com`) - Cloud data repository
- **AWS S3 or compatible** - File storage via presigned URLs

---

## Repository Structure and Element Operation

### Entry Points and Initialization

#### `ndi_Init.m`
The primary initialization function:
1. Removes stale NDI paths from MATLAB path
2. Adds `src/ndi/` directory recursively to path
3. Validates MATLAB toolbox availability via `ndi.fun.check_Matlab_toolboxes`
4. Runs platform-specific checks via `ndi.fun.run_Linux_checks`

**Usage:** Call from MATLAB's `startup.m` or manually before using NDI.

#### `ndi_install.m`
Full installation orchestrator:
1. Verifies git is available on system
2. Clones/updates all dependencies from `requirements.txt`
3. Modifies user's `startup.m` to call `vhtools_startup.m`
4. Supports three dependency modes: minimal (1), standard vhtools (2), developer (3)

#### `ndi_setup.m`
Modern setup using MatBox package manager:
- Downloads and installs MatBox if needed
- Uses `matbox.installRequirements()` for dependency management

---

### Core Classes (`/src/ndi/+ndi/`)

#### `ndi.session` (session.m:1-954)
The central container for experimental data.

**Properties:**
- `reference` - Human-readable session name
- `identifier` - Unique UUID
- `syncgraph` - Time synchronization graph
- `cache` - Performance caching
- `database` - Associated database instance

**Key Methods:**
- `daqsystem_add/rm/load/clear()` - Manage data acquisition systems
- `database_add/rm/search/clear()` - Document CRUD operations
- `database_openbinarydoc/closebinarydoc()` - Binary file access
- `syncgraph_addrule/rmrule()` - Time synchronization rules
- `ingest()` - Import raw data into database
- `getprobes()` - Discover recording probes
- `getelements()` - Retrieve all elements
- `newdocument()` - Create new documents

**Implementations:**
- `ndi.session.dir` - File system-based session
- `ndi.session.mock` - Testing mock session

#### `ndi.document` (document.m:1-979)
The fundamental data storage unit (NoSQL document).

**Properties:**
- `document_properties` - Structure containing all document data

**Key Methods:**
- `setproperties()` - Set document field values
- `add_file()` - Attach binary files
- `dependency()/dependency_value()` - Manage document dependencies
- `set_dependency_value()` - Update dependency references
- `doc_isa()/doc_class()/doc_superclass()` - Type checking
- `to_table()` - Convert to MATLAB table
- `plus()` - Merge two documents
- `validate()` - Schema validation

**Static Methods:**
- `readblankdefinition()` - Create blank document from JSON schema

#### `ndi.query` (query.m:1-156)
Search interface for finding documents (extends `did.query`).

**Operations Supported:**
- `regexp` - Regular expression matching
- `exact_string` / `exact_string_anycase` - Exact string match
- `contains_string` - Substring search
- `exact_number` / `lessthan` / `lessthaneq` / `greaterthan` / `greaterthaneq` - Numeric comparisons
- `hasfield` - Field existence check
- `isa` - Document type inheritance check
- `depends_on` - Dependency relationship search
- `or` / `and` - Logical combination
- `~...` - Negation prefix

**Usage:**
```matlab
q = ndi.query('base.name', 'exact_string', 'my_probe');
q = ndi.query('', 'isa', 'element');
q1 & q2  % AND combination
q1 | q2  % OR combination
```

#### `ndi.database` (database.m)
Abstract base class for document storage.

**Key Methods:**
- `add(document)` - Add/update document
- `read(document_id)` - Retrieve document
- `search(query)` - Query documents
- `remove(document_id)` - Delete document
- `openbinarydoc()/closebinarydoc()` - Binary file access
- `alldocids()` - List all document IDs
- `clear()` - Delete all documents

**Implementations:**
- `ndi.database.implementations.database.matlabdumbjsondb` - JSON file storage
- `ndi.database.implementations.database.didsqlite` - SQLite storage

#### `ndi.element` (element.m)
Represents a recording element (electrode, channel, etc.).

**Properties:**
- `name` - Element name
- `reference` - Reference number
- `type` - Element type (e.g., 'n-trode')
- `underlying_element` - Parent element if derived

**Key Methods:**
- `epochtable()` - Get epochs for this element
- `readtimeseries()` - Read time series data
- `epochclock()` - Get clock types
- `t0_t1()` - Get time boundaries

**Subclasses:**
- `ndi.element.timeseries` - Time series data element

#### `ndi.epoch` (epoch.m)
Time period with experimental metadata.

**Key Properties:**
- `epochid` - Unique epoch identifier
- `session` - Parent session
- `epochprobemap` - Probe mapping for epoch

#### `ndi.probe` (probe.m)
Recording probe metadata.

**Types:**
- `ndi.probe.timeseries` - Standard time series probe
- Configured via probe definition files in `ndi_common/probe/`

#### `ndi.subject` (subject.m)
Experimental subject information.

**Key Fields:**
- `local_identifier` - Lab-specific ID
- `description` - Subject description
- Species, age, sex metadata

#### `ndi.neuron` (neuron.m)
Identified single unit.

**Links to:**
- Element (recording channel)
- Spike sorting results
- Cluster assignment

---

### Data Acquisition System (`/src/ndi/+ndi/+daq/`)

#### `ndi.daq.system`
Abstract base class for DAQ systems.

**Properties:**
- `name` - System name
- `filenavigator` - File discovery
- `daqreader` - Data reader
- `daqmetadatareader` - Stimulus/metadata reader(s)

**Key Methods:**
- `epochtable()` - Build epoch table
- `getepochprobemap()` - Get probe mapping
- `getmetadata()` - Read epoch metadata
- `ingest()` - Compress and store data
- `newdocument()` - Create DAQ documents

#### `ndi.daq.reader` (and `ndi.daq.reader.mfdaq`)
Abstract reader for data files.

**Implementations (in `+reader/+mfdaq/`):**
| Reader | Formats | Description |
|--------|---------|-------------|
| `blackrock.m` | `.nsx`, `.nev` | Blackrock Microsystems |
| `intan.m` | `.rhd` | Intan Technologies RHD2000 |
| `cedspike2.m` | `.smr`, `.son` | CED Spike2 (via sigTOOL) |
| `spikegadgets.m` | `.rec` | SpikeGadgets |
| `ndr.m` | `.abf`, `.mat`, `.bin`, `.sev`, `.rhd` | NDR wrapper (universal) |

**Channel Types:**
- `analog_in/out` (ai/ao)
- `digital_in/out` (di/do)
- `auxiliary_in/out` (ax)
- `time` (t)
- `event` (e)
- `marker` (mk)
- `text` (tx)

#### `ndi.daq.metadatareader`
Reads stimulus/experiment metadata.

**Implementations:**
- `NewStimStims` - VH Lab visual stimulus format
- `BriggsStims` - Briggs Lab format
- `AngelucciStims` - Angelucci Lab format
- `NielsenLabStims` - KJ Nielsen Lab format
- Generic TSV reader

#### Lab-Specific Configurations (`ndi_common/daq_systems/`)

| Lab | Systems | Description |
|-----|---------|-------------|
| **vhlab** | vhintan, vhspike2, vhvis_spike2 | VH Lab configurations |
| **angeluccilab** | angelucci_blackrock5, angelucci_visstim | Blackrock + visual stim |
| **marderlab** | marder_abf, marder_ced | ABF and CED formats |
| **kjnielsenlab** | nielsen_intan, nielsen_vis | Intan + visual stim |
| **dbkatzlab** | narendra_intan | Intan with custom metadata |
| **gluckmanlab** | gluckman_bjgbin | BJG binary format |
| **sjbirrenlab** | sjbirren_abf | Axon ABF format |
| **dabrowskalab** | dabrowska_mat | MATLAB .mat format |
| **yangyangwang** | yangyang_tdt_sev | TDT SEV format |

---

### Time Synchronization (`/src/ndi/+ndi/+time/`)

#### `ndi.time.syncgraph`
Central time synchronization coordinator.

**Properties:**
- `session` - Parent session
- `rules` - Array of syncrules

**Key Methods:**
- `addrule()` - Add synchronization rule
- `buildgraphinfo()` - Construct sync graph
- `time_convert()` - Convert time between devices
- `ingest()` - Store mappings in database

#### `ndi.time.clocktype`
Defines clock types:
- `utc` / `approx_utc` - Universal Coordinated Time
- `exp_global_time` / `approx_exp_global_time` - Experiment-wide time
- `dev_global_time` / `approx_dev_global_time` - Device-wide time
- `dev_local_time` - Epoch-local time only
- `no_time` - No timing information
- `inherited` - Time from another device

#### `ndi.time.timemapping`
Polynomial transformation between time bases.

**Format:** `t_out = mapping(1)*t_in^N + ... + mapping(N+1)`

Usually linear: `t_out = scale * t_in + shift`

#### `ndi.time.syncrule`
Abstract base class for synchronization strategies.

**Implementations (`+syncrule/`):**
- `filematch.m` - Sync by file overlap
- `filefind.m` - Sync using explicit sync files
- `commontriggers.m` - Sync by trigger channels (planned)

#### `ndi.time.timereference`
Specifies time relative to a clock and device.

**Properties:**
- `referent` - Device/system
- `clocktype` - Clock type
- `epoch` - Epoch (for dev_local_time)
- `time` - Time offset

---

### Database Documents (`ndi_common/database_documents/`)

83 JSON schemas organized by category:

#### Foundation & Core
- `base.json` - Root document type
- `session.json` - Session metadata
- `subject.json` / `animalsubject.json` - Subject information
- `element.json` - Recording element
- `app.json` - Analysis application

#### Epoch & Temporal
- `epochid.json` - Epoch identifier mixin
- `element_epoch.json` - Element-epoch association
- `epochclocktimes.json` - Clock info per epoch

#### Data Acquisition
- `daqsystem.json` - DAQ system configuration
- `daqreader.json` / `daqreader_ndr.json` - Reader configs
- `daqmetadatareader.json` - Metadata reader config
- `filenavigator.json` - File organization
- `syncgraph.json` / `syncrule.json` - Time sync

#### Stimulus & Response
- `stimulus_presentation.json` - Stimulus events
- `stimulus_response.json` / `stimulus_response_scalar.json`
- `stimulus_tuningcurve.json` - Tuning curve results
- `orientation_direction_tuning.json` - OD tuning analysis

#### Electrophysiology
- `probe_geometry.json` / `probe_location.json`
- `spike_extraction_parameters.json`
- `spike_clusters.json` / `spikewaves.json`
- `neuron_extracellular.json`

#### Data Storage & Imaging
- `image.json` / `imageStack.json` / `imageCollection.json`
- `ngrid.json` - N-dimensional grid data
- `fitcurve.json` - Curve fit results

#### Data Ingestion
- `epochfiles_ingested.json`
- `daqreader_epochdata_ingested.json`
- `daqreader_mfdaq_epochdata_ingested.json`
- `daqmetadatareader_epochdata_ingested.json`

#### Metadata & Interoperability
- `openminds*.json` - OpenMINDS standard integration
- `dataset_remote.json` - Cloud dataset link
- `session_in_a_dataset.json` - Dataset membership

---

### Application Framework (`/src/ndi/+ndi/+app/`)

#### `ndi.app`
Base class for analysis applications.

**Properties:**
- `session` - NDI session
- `name` - Application name

**Key Methods:**
- `newdocument()` - Create app documents
- Version tracking via git

#### `ndi.calculator`
Base class for automated analysis.

**Key Methods:**
- `run()` - Execute calculator on all matching inputs
- `calculate()` - Perform analysis (override in subclass)
- `search_for_input_parameters()` - Find applicable inputs
- `search_for_calculator_docs()` - Check for existing results

**Conflict Resolution:**
- `'Error'` - Fail if exists
- `'NoAction'` - Skip if exists
- `'Replace'` - Delete and recalculate
- `'ReplaceIfDifferent'` - Replace only if parameters changed

#### Built-in Applications (`+app/`)

| App | Purpose |
|-----|---------|
| `spikeextractor` | Extract spike waveforms from continuous data |
| `spikesorter` | Cluster spikes into units |
| `markgarbage` | Annotate invalid time intervals |
| `oridirtuning` | Orientation/direction tuning analysis |
| `stimulus.decoder` | Parse stimulus presentations |
| `stimulus.tuning_response` | Compute stimulus responses |

#### Calculator Implementations (`+calc/`)

| Calculator | Purpose |
|------------|---------|
| `example.simple` | Demonstration calculator |
| `stimulus.tuningcurve` | Build tuning curves from responses |

---

### Cloud Integration (`/src/ndi/+ndi/+cloud/`)

#### API Package (`+api/`)
REST API interface to NDI Cloud.

**Endpoints:**
- Authentication: `/auth/login`, `/auth/logout`, etc.
- Datasets: CRUD operations, publish/unpublish, submit
- Documents: bulk upload/download
- Files: presigned URL generation
- Compute: remote session management

**Environment Configuration:**
- `CLOUD_API_ENVIRONMENT`: `prod` or `dev`
- `NDI_CLOUD_USERNAME` / `NDI_CLOUD_PASSWORD`
- `NDI_CLOUD_TOKEN`: JWT token
- `NDI_CLOUD_ORGANIZATION_ID`

#### Sync Package (`+sync/`)
Bidirectional synchronization engine.

**Sync Modes:**
- `DownloadNew` - Download remote-only documents
- `UploadNew` - Upload local-only documents
- `MirrorFromRemote` - Local matches remote exactly
- `MirrorToRemote` - Remote matches local exactly
- `TwoWaySync` - Bidirectional additive sync

#### Upload/Download Packages
- Batch upload via ZIP archives (50MB limit)
- Presigned URL-based file transfer
- Automatic retry with exponential backoff

---

### GUI Components (`/src/ndi/+ndi/+gui/`)

- `+component/` - Reusable UI elements
- `+docViewer/` - Document inspection
- `+Lab/` - Lab-specific interfaces
- `+Data/` - Data browsing
- `+Icon/` - Icon resources

---

### Utility Functions (`/src/ndi/+ndi/+fun/`)

| Package | Purpose |
|---------|---------|
| `+data/` | Data evaluation functions |
| `+docTable/` | Document-to-table conversion |
| `+epoch/` | Epoch utilities |
| `+table/` | Table operations (join, stack, filter) |
| `+probe/` | Probe utilities |
| `+session/` | Session utilities |
| `+dataset/` | Dataset utilities |

---

### Testing Framework (`/tests/+ndi/`)

- `+unittest/` - Unit test base classes
- `+test/` - Integration and feature tests
- MatBox integration for test execution
- GitHub Actions CI/CD (R2021b - R2025a)

---

### Java Validator (`/src/ndi/java/ndi-validator-java/`)

Cross-platform JSON Schema validation using everit-org json-schema library.

---

## Data Flow Summary

```
Raw Data Files
    |
    v
[ndi.daq.system] -- filenavigator discovers files
    |              -- daqreader reads data
    |              -- daqmetadatareader reads metadata
    v
[ndi.session.ingest()] -- compress data
                        -- create ingested documents
                        -- store in database
    |
    v
[ndi.database] -- NoSQL document storage
               -- Binary file storage
    |
    v
[ndi.app / ndi.calculator] -- Analysis pipelines
                            -- spike extraction
                            -- spike sorting
                            -- stimulus responses
                            -- tuning curves
    |
    v
[ndi.cloud.sync] -- Upload to cloud
                 -- Download from cloud
                 -- Bidirectional sync
```

---

## Key Design Patterns

1. **Document-Centric Architecture** - All data stored as structured documents
2. **Schema-Driven Validation** - JSON schemas define all document types
3. **Dependency Tracking** - Documents reference dependencies by ID
4. **Abstract Base Classes** - Extensible via inheritance
5. **Namespace Organization** - Clear `+` package hierarchy
6. **Graph-Based Time Sync** - Shortest-path time conversion
7. **Lab-Specific Configuration** - Pre-built DAQ system configs
8. **Cloud-First Design** - Built-in cloud sync capabilities

---

## References

- **Publication:** eNeuro (RRID: SCR_023368)
- **Funding:** NIH BRAIN Initiative
- **Website:** [ndi-cloud.com](https://ndi-cloud.com)
- **Repository:** [github.com/VH-Lab/NDI-matlab](https://github.com/VH-Lab/NDI-matlab)
