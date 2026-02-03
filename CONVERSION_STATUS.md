# NDI Python Conversion Status

**Last Updated:** 2026-02-03

**Plan Document:** [PYTHON_CONVERSION_PLAN.md](./PYTHON_CONVERSION_PLAN.md)

---

## Phase Summary

| Phase | Component | Status | Tests | Blocked By | Notes |
|-------|-----------|--------|-------|------------|-------|
| 0 | did-python | ⬜ Not Started | 0% | - | **START HERE** - Critical dependency |
| 1 | ndi.util | ⬜ Not Started | 0% | Phase 0 | 87 functions to port |
| 2 | ndi.document, ndi.query, ndi.ido | ⬜ Not Started | 0% | Phase 0-1 | Core data structures |
| 3 | ndi.database | ⬜ Not Started | 0% | Phase 2 | SQLite + JSON backends |
| 4 | ndi.time | ⬜ Not Started | 0% | Phase 2-3 | Syncgraph is complex |
| 5 | ndi.daq | ⬜ Not Started | 0% | Phase 4 | spikeinterface integration |
| 6 | ndi.element, ndi.epoch, ndi.probe | ⬜ Not Started | 0% | Phase 5 | |
| 7 | ndi.session | ⬜ Not Started | 0% | Phase 6 | Central orchestrator |
| 8 | ndi.dataset, ndi.subject, ndi.neuron | ⬜ Not Started | 0% | Phase 7 | |
| 9 | ndi.app, ndi.calculator | ⬜ Not Started | 0% | Phase 8 | Analysis apps |
| 10 | ndi.cloud | ⬜ Not Started | 0% | Phase 8 | REST client |

### Legend
- ⬜ Not Started
- 🟡 In Progress
- ✅ Complete
- 🔴 Blocked

---

## Phase 0: did-python (CRITICAL - DO FIRST)

This is a **separate repository** that must be created before NDI conversion can proceed.

### Repository Setup
- [ ] Create `did-python` repository
- [ ] Set up pyproject.toml
- [ ] Set up pytest and CI

### did.ido (Identifier Generation)
- [ ] `unique_id()` - Generate UUID-based identifiers
- [ ] `is_valid()` - Validate identifier format
- [ ] Unit tests

### did.document (Document Base Class)
- [ ] `Document` class with `document_properties`
- [ ] `from_json()` / `to_json()` serialization
- [ ] `read_blank_definition()` - Load from JSON schema
- [ ] `id` property
- [ ] Unit tests

### did.query (Query System)
- [ ] `Query` class with field, operation, params
- [ ] `QueryOp` enum (all 12 operations)
- [ ] `__and__`, `__or__`, `__invert__` operators
- [ ] `to_search_structure()` conversion
- [ ] Unit tests

### did.datastructures (Utilities)
- [ ] `empty_dict()` / `emptystruct()` equivalent
- [ ] `json_encode_nan()` - Handle NaN in JSON
- [ ] Unit tests

### did.file (File Utilities)
- [ ] `FileObj` class
- [ ] `ReadOnlyFileObj` class
- [ ] `write_text()` / `read_text()` helpers
- [ ] Unit tests

### did.common.PathConstants
- [ ] Path definitions map
- [ ] NDI schema path integration
- [ ] Unit tests

### did.implementations.SQLiteDB
- [ ] `__init__()` with database initialization
- [ ] `add_docs()` - Add documents
- [ ] `get_docs()` - Retrieve by ID
- [ ] `remove_docs()` - Delete documents
- [ ] `search()` - Query execution
- [ ] `get_doc_ids()` - List all IDs
- [ ] Binary file storage (open/close)
- [ ] Unit tests
- [ ] Integration tests

---

## Phase 1: ndi.util

### ndi.util.data (69 functions → ~30 Python functions)

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

## Session Notes

*Add notes from each work session here*

### 2026-02-03 - Initial Planning
- Created REPOSITORY_ANALYSIS.md with full codebase understanding
- Created PYTHON_CONVERSION_PLAN.md with detailed conversion strategy
- Created this status tracking file
- Analyzed all did.*, vlt.*, and ndr.* dependencies
- Ready to begin Phase 0: did-python

---

## Next Actions

1. **Create did-python repository** (separate from NDI)
2. **Implement did.ido** with `unique_id()` and `is_valid()`
3. **Implement did.document** with JSON serialization
4. **Continue through Phase 0 checklist**
