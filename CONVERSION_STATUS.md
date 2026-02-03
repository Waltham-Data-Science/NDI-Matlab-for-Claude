# NDI Python Conversion Status

**Last Updated:** 2026-02-03

**Plan Document:** [PYTHON_CONVERSION_PLAN.md](./PYTHON_CONVERSION_PLAN.md)

---

## CRITICAL: Existing VH-Lab Python Repositories

**These packages ALREADY EXIST. Do NOT recreate them:**

| Repository | URL | What It Contains |
|------------|-----|------------------|
| **DID-python** | https://github.com/VH-Lab/DID-python | `did.document`, `did.query`, `did.database`, `did.implementations/` |
| **vhlab-toolbox-python** | https://github.com/VH-Lab/vhlab-toolbox-python | 34 `vlt.data` functions, 9 `vlt.file` functions |
| **vhlab-library-python** | https://github.com/VH-Lab/vhlab-library-python | Library utilities |
| **vhlab-NewStim-python** | https://github.com/VH-Lab/vhlab-NewStim-python | Stimulus metadata readers |

**Previous conversion failed because Claude wrote all this code inline. USE THESE AS DEPENDENCIES.**

---

## Phase Summary

| Phase | Component | Status | Tests | Notes |
|-------|-----------|--------|-------|-------|
| 0 | External Dependencies | ✅ EXIST | N/A | Use DID-python, vhlab-toolbox-python as deps |
| 1 | Gap Analysis | ⬜ Not Started | 0% | Check if any vlt.* functions missing |
| 2 | ndi.document, ndi.query, ndi.ido | ⬜ Not Started | 0% | Thin wrappers over DID-python |
| 3 | ndi.database | ⬜ Not Started | 0% | Wrapper over DID implementations |
| 4 | ndi.time | ⬜ Not Started | 0% | Syncgraph, timemapping |
| 5 | ndi.daq | ⬜ Not Started | 0% | spikeinterface integration |
| 6 | ndi.element, ndi.epoch, ndi.probe | ⬜ Not Started | 0% | |
| 7 | ndi.session | ⬜ Not Started | 0% | Central orchestrator |
| 8 | ndi.dataset, ndi.subject, ndi.neuron | ⬜ Not Started | 0% | |
| 9 | ndi.app, ndi.calculator | ⬜ Not Started | 0% | Analysis apps |
| 10 | ndi.cloud | ⬜ Not Started | 0% | REST client |

### Legend
- ⬜ Not Started
- 🟡 In Progress
- ✅ Complete / Exists Externally
- 🔴 Blocked

---

## Phase 0: External Dependencies (ALREADY EXIST)

### DID-python - https://github.com/VH-Lab/DID-python
- [x] Repository exists (170 commits, 5 contributors)
- [x] `did.document` - Document class
- [x] `did.query` - Query class
- [x] `did.database` - Database base class
- [x] `did.binarydoc` - Binary document handling
- [x] `did.datastructures/` - Data structure utilities
- [x] `did.file/` - File utilities
- [x] `did.implementations/` - SQLite and other backends

**Action:** Clone and verify it meets NDI needs. Contribute fixes if needed.

### vhlab-toolbox-python - https://github.com/VH-Lab/vhlab-toolbox-python
- [x] Repository exists (46 commits, partial port for NDI)
- [x] `vlt.data` - 34 functions ported
- [x] `vlt.file` - 9 functions ported
- [x] `vlt.app.log` - Logging

**See their PORTING_PROGRESS.md for details.**

### vhlab-NewStim-python - https://github.com/VH-Lab/vhlab-NewStim-python
- [x] Repository exists
- [x] Stimulus metadata readers

---

## Phase 1: Gap Analysis (Check What's Missing)

### CRITICAL REFERENCE FILE

**The file `src/ndi/docs/developer_notes/vltInNDI.md` contains the COMPLETE list of all 97 vlt.* functions used in NDI with exact file:line locations.**

This file should be the authoritative source for the gap analysis, not estimates.

Before writing any code, verify which vlt.* functions NDI needs are already ported.

### vlt.data Functions - AUDIT RESULTS (36 files exist in vhlab-toolbox-python)

**Source:** https://github.com/VH-Lab/vhlab-toolbox-python/tree/main/vlt/data

| Function | Calls in NDI | In vhlab-toolbox-python? | File |
|----------|--------------|--------------------------|------|
| `vlt.data.emptystruct()` | 69 | ✅ YES | `emptystruct.py` |
| `vlt.data.assign()` | 26 | ✅ YES | `assign.py` |
| `vlt.data.colvec()` | 16 | ✅ YES | `colvec.py` |
| `vlt.data.eqlen()` | 14 | ✅ YES | `eqlen.py` |
| `vlt.data.matrow2cell()` | 11 | ✅ YES | `matrow2cell.py` |
| `vlt.data.celloritem()` | 9 | ✅ YES | `celloritem.py` |
| `vlt.data.cellarray2mat()` | 8 | ✅ YES | `cellarray2mat.py` |
| `vlt.data.var2struct()` | 7 | ✅ YES | `var2struct.py` |
| `vlt.data.rowvec()` | 7 | ✅ YES | `rowvec.py` |
| `vlt.data.fieldsearch()` | 7 | ✅ YES | `fieldsearch.py` |
| `vlt.data.equnique()` | 6 | ✅ YES | `equnique.py` |
| `vlt.data.hasAllFields()` | 6 | ✅ YES | `hasAllFields.py` |
| `vlt.data.conditional()` | 6 | ✅ YES | `conditional.py` |
| `vlt.data.findclosest()` | 5 | ✅ YES | `findclosest.py` |
| `vlt.data.nanstderr()` | 4 | ✅ YES | `nanstderr.py` |
| `vlt.data.dropnan()` | 4 | ✅ YES | `dropnan.py` |
| `vlt.data.hashmatlabvariable()` | 3 | ✅ YES | `hashmatlabvariable.py` |
| `vlt.data.islikevarname()` | 3 | ✅ YES | `islikevarname.py` |
| `vlt.data.string2cell()` | 3 | ✅ YES | `string2cell.py` |
| `vlt.data.structmerge()` | 3 | ✅ YES | `structmerge.py` |
| `vlt.data.flattenstruct2table()` | 3 | ✅ YES | `flattenstruct2table.py` |
| `vlt.data.prettyjson()` | 2 | ✅ YES | `prettyjson.py` |
| `vlt.data.jsonencodenan()` | 2 | ✅ YES | `jsonencodenan.py` |
| `vlt.data.columnize_struct()` | 2 | ✅ YES | `columnize_struct.py` |
| `vlt.data.structwhatvaries()` | 1 | ✅ YES | `structwhatvaries.py` |
| `vlt.data.structfullfields()` | 1 | ✅ YES | `structfullfields.py` |
| `vlt.data.partial_struct_match()` | 1 | ✅ YES | `partial_struct_match.py` |
| `vlt.data.isint()` | 1 | ✅ YES | `isint.py` |
| `vlt.data.findrowvec()` | 1 | ✅ YES | `findrowvec.py` |
| `vlt.data.emptytable()` | 1 | ✅ YES | `emptytable.py` |
| `vlt.data.cell2str()` | 1 | ✅ YES | `cell2str.py` |
| `vlt.data.tabstr2struct()` | 1 | ✅ YES | `tabstr2struct.py` |
| `vlt.data.workspace2struct()` | 1 | ✅ YES | `workspace2struct.py` |

**RESULT: ALL 33 vlt.data functions are already ported!**

### vlt.file Functions - AUDIT RESULTS (7 files exist)

**Source:** https://github.com/VH-Lab/vhlab-toolbox-python/tree/main/vlt/file

| Function | Calls in NDI | In vhlab-toolbox-python? | Notes |
|----------|--------------|--------------------------|-------|
| `vlt.file.textfile2char()` | 14 | ⚠️ UNCLEAR | Not in visible file list |
| `vlt.file.text2cellstr()` | 13 | ⚠️ UNCLEAR | Mentioned in PORTING_PROGRESS but not visible |
| `vlt.file.str2text()` | 10 | ⚠️ UNCLEAR | Not in visible file list |
| `vlt.file.loadStructArray()` | 9 | ⚠️ UNCLEAR | May be in `custom_struct_io.py` |
| `vlt.file.dumbjsondb` | 9 | ⚠️ UNCLEAR | Not visible |
| `vlt.file.findfilegroups()` | 9 | ⚠️ UNCLEAR | Not visible |
| `vlt.file.fileobj` | 7 | ⚠️ UNCLEAR | Not visible |
| `vlt.file.saveStructArray()` | 5 | ⚠️ UNCLEAR | May be in `custom_struct_io.py` |
| `vlt.file.createpath()` | 1 | ✅ Mentioned | Per PORTING_PROGRESS.md |
| `vlt.file.touch()` | 1 | ✅ Mentioned | Per PORTING_PROGRESS.md |
| `vlt.file.custom_file_formats.*` | 5 | ✅ EXISTS | `custom_file_formats.py` |

**ACTION REQUIRED:** Clone repo locally to verify file contents match PORTING_PROGRESS.md

### vlt.signal Functions - AUDIT RESULTS (6 files exist)

**Source:** https://github.com/VH-Lab/vhlab-toolbox-python/tree/main/vlt/signal

| Function | Calls in NDI | In vhlab-toolbox-python? | File |
|----------|--------------|--------------------------|------|
| `vlt.signal.dotdisc()` | 2 | ✅ YES | `dotdisc.py` |
| `vlt.signal.refractory()` | 2 | ✅ YES | `refractory.py` |
| `vlt.signal.value2sample()` | 2 | ✅ YES | `value2sample.py` |

**RESULT: All 3 critical signal functions are ported!**

### vlt.neuro Functions - AUDIT RESULTS (subdirectories exist)

**Source:** https://github.com/VH-Lab/vhlab-toolbox-python/tree/main/vlt/neuro

Subdirectories exist but file contents not visible from web:
- `vlt/neuro/spikesorting/` - needs verification
- `vlt/neuro/stimulus/` - needs verification
- `vlt/neuro/vision/` - needs verification

**ACTION REQUIRED:** Clone repo locally to verify contents

### DID-python Classes - AUDIT RESULTS

**Source:** https://github.com/VH-Lab/DID-python

**did.document.Document class methods:**
- `__init__(document_type, **options)` - Constructor
- `id()` - Get document ID
- `add_dependency(name, value)` - Add dependency
- `get_dependency_value(name, error_if_not_found)` - Get dependency
- `set_dependency_value(name, value, error_if_not_found)` - Set dependency
- `add_dependency_value_n(name, value)` - Add numbered dependency
- `remove_dependency_value_n(name, n)` - Remove numbered dependency
- `add_file(name, location, ...)` - Add file
- `remove_file(name, location)` - Remove file
- `is_in_file_list(name)` - Check file list
- `_read_blank_definition(document_type)` - Load schema
- `__eq__(other)` - Equality by ID

**did.query.Query class methods:**
- `__init__(...)` - Flexible constructor (dict, list, Query, or field/op/params)
- `__and__` - Combine with AND (`&` operator)
- `__or__` - Combine with OR (`|` operator)
- `to_search_structure()` - Export to dict

**RESULT: Core DID classes have all essential methods!**

---

## Phase 2: ndi.document, ndi.query, ndi.ido

These should be **thin wrappers** over DID-python, NOT reimplementations.

### ndi.document (wrapper over did.document)

| MATLAB Function | Python Function | Status |
|-----------------|-----------------|--------|
| `vlt.data.emptystruct()` | `empty_dict()` | ⬜ |
| `vlt.data.colvec()` | `colvec()` | ⬜ |
| `vlt.data.rowvec()` | `rowvec()` | ⬜ |
| `vlt.data.eqlen()` | `eqlen()` | ⬜ |
| `vlt.data.assign()` | N/A (use kwargs) | ⬜ |
| `vlt.data.matrow2cell()` | `matrix_rows_to_list()` | ⬜ |
| `vlt.data.celloritem()` | `cell_or_item()` | ⬜ |
| `vlt.data.var2struct()` | `vars_to_dict()` | ⬜ |
| `vlt.data.fieldsearch()` | `field_search()` | ⬜ |
| `vlt.data.cellarray2mat()` | `np.array()` | ⬜ |
| `vlt.data.hasAllFields()` | `has_all_fields()` | ⬜ |
| `vlt.data.equnique()` | `np.unique()` | ⬜ |
| `vlt.data.conditional()` | Boolean indexing | ⬜ |
| `vlt.data.dropnan()` | `dropnan()` | ⬜ |
| `vlt.data.nanstderr()` | `scipy.stats.sem()` | ⬜ |
| `vlt.data.findclosest()` | `findclosest()` | ⬜ |
| `vlt.data.string2cell()` | `str.split()` | ⬜ |
| `vlt.data.structmerge()` | `struct_merge()` | ⬜ |
| `vlt.data.prettyjson()` | `json.dumps(indent=2)` | ⬜ |
| `vlt.data.flattenstruct2table()` | `pd.json_normalize()` | ⬜ |
| ... | ... | ... |

### ndi.util.file (25 functions)

| MATLAB Function | Python Function | Status |
|-----------------|-----------------|--------|
| `vlt.file.textfile2char()` | `read_text()` | ⬜ |
| `vlt.file.text2cellstr()` | `read_lines()` | ⬜ |
| `vlt.file.str2text()` | `write_text()` | ⬜ |
| `vlt.file.loadStructArray()` | `load_json()` | ⬜ |
| `vlt.file.saveStructArray()` | `save_json()` | ⬜ |
| `vlt.file.manifest()` | `list_files()` | ⬜ |
| `vlt.file.findfilegroups()` | `find_file_groups()` | ⬜ |
| `vlt.file.dumbjsondb` | `DumbJsonDB` class | ⬜ |
| `vlt.file.findfiletype()` | `Path.glob()` | ⬜ |
| ... | ... | ... |

### ndi.util.neuro (10 functions)

| MATLAB Function | Python Function | Status |
|-----------------|-----------------|--------|
| `vlt.neuro.stimulus.stimulus_response_scalar()` | `stimulus_response_scalar()` | ⬜ |
| `vlt.neuro.spikesorting.spikewaves2pca()` | `spikewaves_to_pca()` | ⬜ |
| `vlt.neuro.spikesorting.oversamplespikes()` | `oversample_spikes()` | ⬜ |
| ... | ... | ... |

### ndi.util.signal (6 functions)

| MATLAB Function | Python Function | Status |
|-----------------|-----------------|--------|
| `vlt.signal.dotdisc()` | `detect_spikes()` | ⬜ |
| `vlt.signal.refractory()` | `apply_refractory()` | ⬜ |
| `vlt.signal.value2sample()` | `time_to_sample()` | ⬜ |
| ... | ... | ... |

### ndi.util.string (6 functions)

| MATLAB Function | Python Function | Status |
|-----------------|-----------------|--------|
| `vlt.string.strcmp_substitution()` | `string_match()` | ⬜ |
| `vlt.string.intseq2str()` | `int_sequence_to_str()` | ⬜ |
| `vlt.string.line_n()` | `get_line()` | ⬜ |
| ... | ... | ... |

---

## Phase 2: ndi.document, ndi.query, ndi.ido

### ndi.document
- [ ] Class definition extending did.Document
- [ ] Constructor with schema loading
- [ ] `set_session_id()`
- [ ] `add_file()` / `remove_file()` / `has_files()`
- [ ] `dependency()` / `dependency_value()` / `set_dependency_value()`
- [ ] `doc_isa()` / `doc_class()` / `doc_superclass()`
- [ ] `to_table()` - pandas DataFrame conversion
- [ ] `setproperties()` - bulk property setting
- [ ] `__eq__()` - equality by ID
- [ ] `__add__()` - merge documents
- [ ] Static: `find_doc_by_id()`, `find_newest()`
- [ ] Unit tests
- [ ] Integration tests with JSON schemas

### ndi.query
- [ ] Class extending did.Query
- [ ] `isa()` class method
- [ ] `depends_on()` class method
- [ ] `by_id()` class method
- [ ] Unit tests

### ndi.ido
- [ ] Class extending did.ido
- [ ] Unit tests

### ndi.validate
- [ ] `validate_document()` function
- [ ] JSON Schema validation using `jsonschema`
- [ ] Superclass chain validation
- [ ] Unit tests

---

## Phase 3: ndi.database

### ndi.database (Abstract Base)
- [ ] Abstract `Database` class
- [ ] `add()` abstract method
- [ ] `read()` abstract method
- [ ] `remove()` abstract method
- [ ] `search()` abstract method
- [ ] `all_doc_ids()` abstract method
- [ ] `open_binary_doc()` / `close_binary_doc()`
- [ ] `clear()` with confirmation

### ndi.database.implementations.JsonDB
- [ ] JSON file-based storage
- [ ] Document CRUD
- [ ] Query execution
- [ ] Binary file storage
- [ ] Unit tests

### ndi.database.implementations.SQLiteDB
- [ ] SQLite wrapper using did.implementations.SQLiteDB
- [ ] NDI-specific extensions
- [ ] Unit tests

---

## Phase 4: ndi.time

### ndi.time.clocktype
- [ ] `ClockType` enum with all 9 types
- [ ] `ndiglobal()` method
- [ ] Unit tests

### ndi.time.timemapping
- [ ] `TimeMapping` class
- [ ] Polynomial evaluation
- [ ] `linear()` class method
- [ ] `identity()` class method
- [ ] Unit tests

### ndi.time.timereference
- [ ] `TimeReference` class
- [ ] Properties: referent, clocktype, epoch, time
- [ ] Unit tests

### ndi.time.syncgraph
- [ ] `SyncGraph` class
- [ ] NetworkX graph backend
- [ ] `add_rule()` method
- [ ] `build_graph_info()` method
- [ ] `time_convert()` - shortest path time conversion
- [ ] `ingest()` - store mappings in database
- [ ] Unit tests
- [ ] Integration tests with example data

### ndi.time.syncrule
- [ ] `SyncRule` abstract base
- [ ] `filematch.py` - sync by file overlap
- [ ] `filefind.py` - sync using explicit files
- [ ] Unit tests

---

## Phase 5: ndi.daq

### ndi.daq.reader (Abstract)
- [ ] `DAQReader` Protocol/ABC
- [ ] `get_channels_epoch()`
- [ ] `read_channels_epoch_samples()`
- [ ] `epoch_clock()`
- [ ] `t0_t1()`
- [ ] `sample_rate()`

### ndi.daq.reader.mfdaq.SpikeInterfaceAdapter
- [ ] Wrapper for spikeinterface
- [ ] Intan RHD support
- [ ] Blackrock support
- [ ] TDT support
- [ ] SpikeGadgets support
- [ ] Unit tests

### ndi.daq.reader.mfdaq.NeoAdapter
- [ ] Wrapper for neo
- [ ] CED Spike2 support
- [ ] Axon ABF support
- [ ] Unit tests

### ndi.daq.metadatareader
- [ ] Abstract base
- [ ] NewStim reader
- [ ] Unit tests

### ndi.daq.file.navigator
- [ ] `FileNavigator` abstract base
- [ ] `EpochDir` implementation
- [ ] Unit tests

### ndi.daq.system
- [ ] `DAQSystem` class
- [ ] `epochtable()`
- [ ] `getepochprobemap()`
- [ ] `getmetadata()`
- [ ] `ingest()`
- [ ] Unit tests

---

## Phase 6: ndi.element, ndi.epoch, ndi.probe

### ndi.element
- [ ] `Element` class
- [ ] Properties: name, reference, type, underlying_element
- [ ] `epochtable()`
- [ ] `readtimeseries()`
- [ ] `epochclock()`
- [ ] `t0_t1()`
- [ ] Unit tests

### ndi.element.timeseries
- [ ] Timeseries element subclass
- [ ] Unit tests

### ndi.epoch
- [ ] `Epoch` class
- [ ] Properties: epochid, session, epochprobemap
- [ ] Unit tests

### ndi.probe
- [ ] `Probe` class
- [ ] Probe definition loading
- [ ] Unit tests

### ndi.probe.timeseries
- [ ] Timeseries probe subclass
- [ ] Unit tests

---

## Phase 7: ndi.session

### ndi.session
- [ ] `Session` class
- [ ] Properties: reference, identifier, syncgraph, cache, database
- [ ] `daqsystem_add()` / `daqsystem_rm()` / `daqsystem_load()`
- [ ] `database_add()` / `database_rm()` / `database_search()`
- [ ] `database_openbinarydoc()` / `database_closebinarydoc()`
- [ ] `syncgraph_addrule()` / `syncgraph_rmrule()`
- [ ] `ingest()`
- [ ] `getprobes()`
- [ ] `getelements()`
- [ ] `newdocument()`
- [ ] Unit tests
- [ ] Integration tests with example sessions

### ndi.session.dir
- [ ] Directory-based session implementation
- [ ] Unit tests

---

## Phase 8: ndi.dataset, ndi.subject, ndi.neuron

### ndi.dataset
- [ ] `Dataset` class
- [ ] Multi-session container
- [ ] Unit tests

### ndi.subject
- [ ] `Subject` class
- [ ] Subject metadata
- [ ] Unit tests

### ndi.neuron
- [ ] `Neuron` class
- [ ] Links to element, spike sorting
- [ ] Unit tests

---

## Phase 9: ndi.app, ndi.calculator

### ndi.app (Base)
- [ ] `App` abstract class
- [ ] `newdocument()`
- [ ] Version tracking
- [ ] Unit tests

### ndi.calculator (Base)
- [ ] `Calculator` abstract class
- [ ] `run()`
- [ ] `calculate()` abstract
- [ ] `search_for_input_parameters()`
- [ ] Conflict resolution modes
- [ ] Unit tests

### ndi.app.spikeextractor
- [ ] Spike extraction application
- [ ] Unit tests

### ndi.app.spikesorter
- [ ] Spike sorting application
- [ ] Unit tests

### ndi.app.stimulus.decoder
- [ ] Stimulus decoding
- [ ] Unit tests

### ndi.app.stimulus.tuning_response
- [ ] Tuning response computation
- [ ] Unit tests

### ndi.calc.stimulus.tuningcurve
- [ ] Tuning curve calculator
- [ ] Unit tests

---

## Phase 10: ndi.cloud

### ndi.cloud.api.auth
- [ ] `login()` / `logout()`
- [ ] JWT token management
- [ ] Environment variable support
- [ ] Unit tests

### ndi.cloud.api.datasets
- [ ] CRUD operations
- [ ] Publish/unpublish
- [ ] Submit
- [ ] Unit tests

### ndi.cloud.api.documents
- [ ] Document upload/download
- [ ] Bulk operations
- [ ] ndiquery
- [ ] Unit tests

### ndi.cloud.api.files
- [ ] Presigned URL handling
- [ ] Upload/download
- [ ] Unit tests

### ndi.cloud.sync
- [ ] Sync modes (DownloadNew, UploadNew, Mirror, TwoWay)
- [ ] Index management
- [ ] Unit tests

### ndi.cloud.upload
- [ ] Dataset upload
- [ ] Batch ZIP uploads
- [ ] Unit tests

### ndi.cloud.download
- [ ] Dataset download
- [ ] Local/hybrid modes
- [ ] Unit tests

---

## Blockers and Issues

*Document any issues that block progress here*

| Issue | Phase | Description | Resolution |
|-------|-------|-------------|------------|
| - | - | - | - |

---

## Lessons Learned: Context Discovery

**Problem:** Previous conversion attempt failed because Claude wrote dependency code inline instead of using existing Python repos.

**Root Cause:** The existing VH-Lab Python repos were not discovered because:
1. README.md and AGENTS.md don't mention Python
2. The repos are in a different GitHub organization location
3. User context was needed to reveal their existence

**Key Files That Should Have Been Checked:**
1. `src/ndi/docs/NDI-matlab/index.md` - Line 23 says "a version for Python is well under construction"
2. `src/ndi/docs/developer_notes/vltInNDI.md` - Complete list of all 97 vlt.* functions with file:line locations
3. The VH-Lab GitHub organization directly - has DID-python, vhlab-toolbox-python, etc.

**Context Discovery Protocol for Future Sessions:**
1. Always check `docs/` folder for developer notes, especially `developer_notes/`
2. Search for "python" in all markdown files
3. Check the GitHub organization for sibling repos (not just this repo)
4. Ask the user about existing Python work before proposing to create new packages
5. Read files like index.md that describe the project's ecosystem

**Key Discovery:** vhlab-toolbox-python explicitly states it's a "partial port for supporting NDI-python" - this repo was PURPOSE-BUILT for NDI conversion.

---

## Session Notes

*Add notes from each work session here*

### 2026-02-03 - Initial Planning
- Created REPOSITORY_ANALYSIS.md with full codebase understanding
- Created PYTHON_CONVERSION_PLAN.md with detailed conversion strategy
- Created this status tracking file
- Analyzed all did.*, vlt.*, and ndr.* dependencies
- Ready to begin Phase 0: did-python

### 2026-02-03 - Updated with Existing Repos
- Discovered existing VH-Lab Python repos after user feedback
- Updated plan to use DID-python, vhlab-toolbox-python as dependencies
- Audited actual contents of vhlab-toolbox-python:
  - vlt/data: 36 files - ALL 33 needed functions are ported!
  - vlt/signal: 6 files - all 3 critical functions ported
  - vlt/file: 7 files - some gaps, needs local verification
  - vlt/neuro: subdirectories exist, needs verification
- Audited DID-python document.py and query.py - core classes implemented
- Added "Lessons Learned" section for context discovery

---

## Next Actions

1. **Create did-python repository** (separate from NDI)
2. **Implement did.ido** with `unique_id()` and `is_valid()`
3. **Implement did.document** with JSON serialization
4. **Continue through Phase 0 checklist**
