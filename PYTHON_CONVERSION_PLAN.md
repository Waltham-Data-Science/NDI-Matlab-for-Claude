# NDI-Matlab to Python Conversion Plan

## Executive Summary

This document provides a complete, authoritative plan for converting NDI-Matlab to Python. The conversion is designed to:

1. **USE EXISTING VH-Lab Python packages** - DID-python, vhlab-toolbox-python already exist
2. **NOT convert external dependencies inline** - This was the failure mode of previous attempts
3. **Preserve the architecture** - Maintain the same patterns and APIs
4. **Be incrementally verifiable** - Each phase produces working, tested code
5. **Support context continuity** - Any developer (human or AI) can pick up from any phase

**Estimated Scope:** ~747 MATLAB files → ~150 Python files (less because dependencies exist)
**Key Insight:** Core dependencies (DID, VLT) are ALREADY PORTED by VH-Lab

---

## CRITICAL: Existing VH-Lab Python Repositories

**These packages ALREADY EXIST and must be used as dependencies, NOT recreated:**

| Repository | URL | Status |
|------------|-----|--------|
| **DID-python** | https://github.com/VH-Lab/DID-python | ✅ Active (170 commits, 5 contributors) |
| **vhlab-toolbox-python** | https://github.com/VH-Lab/vhlab-toolbox-python | ✅ Active (46 commits, partial port for NDI) |
| **vhlab-library-python** | https://github.com/VH-Lab/vhlab-library-python | ✅ Available |
| **vhlab-NewStim-python** | https://github.com/VH-Lab/vhlab-NewStim-python | ✅ Available |

**Previous conversion attempts failed because Claude wrote all dependency code inline.
This plan uses these existing packages as pip dependencies.**

---

## Table of Contents

1. [Dependency Mapping](#1-dependency-mapping)
2. [Conversion Order](#2-conversion-order)
3. [Phase Breakdown](#3-phase-breakdown)
4. [Python Package Structure](#4-python-package-structure)
5. [API Design Specifications](#5-api-design-specifications)
6. [Testing Strategy](#6-testing-strategy)
7. [Schema Migration](#7-schema-migration)
8. [Progress Tracking](#8-progress-tracking)
9. [Risk Mitigation](#9-risk-mitigation)
10. [Session Handoff Protocol](#10-session-handoff-protocol)

---

## 1. Dependency Mapping

### 1.1 External Repository → Python Library Mapping

| MATLAB Repository | Python Equivalent | Status |
|-------------------|-------------------|--------|
| **DID-matlab** | **[DID-python](https://github.com/VH-Lab/DID-python)** | ✅ EXISTS - use as dependency |
| **vhlab-toolbox-matlab** | **[vhlab-toolbox-python](https://github.com/VH-Lab/vhlab-toolbox-python)** | ✅ EXISTS - partial port for NDI |
| **vhlab-library-matlab** | **[vhlab-library-python](https://github.com/VH-Lab/vhlab-library-python)** | ✅ EXISTS - use as dependency |
| **vhlab-NewStim-matlab** | **[vhlab-NewStim-python](https://github.com/VH-Lab/vhlab-NewStim-python)** | ✅ EXISTS - use as dependency |
| **NDR-matlab** | **`spikeinterface`** + **`neo`** | ✅ External libs cover 95% |
| **vhlab-thirdparty-matlab** | Native Python libs | sigTOOL → neo.io handles Spike2 |
| **mksqlite** | **`sqlite3`** (builtin) | ✅ Direct replacement |
| **openMINDS_MATLAB** | **`openminds`** | ✅ Already exists in Python |
| **NDI-compress-matlabp** | **`zlib`** + **`lz4`** | ✅ Standard compression |

### 1.2 DID-python Package (ALREADY EXISTS)

**Repository:** https://github.com/VH-Lab/DID-python

**Package Structure (already implemented):**
```
did/
├── __init__.py
├── binarydoc.py         # Binary document handling
├── database.py          # did.database base class
├── document.py          # did.document
├── query.py             # did.query
├── common/              # Common utilities
├── datastructures/      # did.datastructures
├── db/                  # Database utilities
├── file/                # did.file utilities
├── fun/                 # Helper functions
└── implementations/     # Database implementations (SQLite, etc.)
```

**NDI should import from this package:**
```python
from did.document import Document
from did.query import Query
from did.database import Database
from did.implementations import SQLiteDB
```

### 1.3 vhlab-toolbox-python Package (ALREADY EXISTS - PARTIAL)

**Repository:** https://github.com/VH-Lab/vhlab-toolbox-python

**Explicitly stated purpose:** "partial port for supporting NDI-python"

**Already Ported (per PORTING_PROGRESS.md):**

| Module | Functions Ported | Count |
|--------|------------------|-------|
| `vlt.app` | `vlt.app.log.Log` | 1 |
| `vlt.data` | `cellarray2mat`, `flattenstruct2table`, `structmerge`, `isint`, `islikevarname`, + 29 more | 34 |
| `vlt.file` | `isfilepathroot`, `fullfilename`, `createpath`, `touch`, `text2cellstr`, `checkout_lock_file`, `release_lock_file`, + 2 more | 9 |

**If additional functions needed:** Contribute to vhlab-toolbox-python, don't duplicate.

**NDI should import from this package:**
```python
from vlt.data import cellarray2mat, structmerge, flattenstruct2table
from vlt.file import text2cellstr, createpath
```

### 1.4 Functions That May Need Adding to vhlab-toolbox-python

Based on NDI-matlab analysis, these vlt.* functions are heavily used but may not be ported yet:

| Function | Calls in NDI | Priority |
|----------|--------------|----------|
| `vlt.data.emptystruct()` | 69 | HIGH |
| `vlt.data.assign()` | 26 | MEDIUM (use kwargs in Python) |
| `vlt.data.colvec()` | 16 | HIGH |
| `vlt.data.eqlen()` | 14 | HIGH |
| `vlt.data.celloritem()` | 9 | MEDIUM |
| `vlt.file.textfile2char()` | 14 | HIGH |
| `vlt.file.loadStructArray()` | 9 | HIGH |
| `vlt.file.dumbjsondb` | 9 | HIGH |

**Action:** Check vhlab-toolbox-python for these. If missing, contribute PRs to that repo.

### 1.3 VLT Toolbox Functions → Python Implementation

415 calls across 87 unique functions. Here's the mapping:

#### vlt.data.* (237 calls) → numpy/pandas/custom

```python
# ndi/util/data.py

import numpy as np
import pandas as pd
from dataclasses import dataclass, field
from typing import Any, Dict, List

def empty_struct(*fields: str) -> Dict[str, None]:
    """vlt.data.emptystruct() - Create dict with None values"""
    return {f: None for f in fields}

def colvec(x: np.ndarray) -> np.ndarray:
    """vlt.data.colvec() - Convert to column vector"""
    return np.asarray(x).reshape(-1, 1)

def rowvec(x: np.ndarray) -> np.ndarray:
    """vlt.data.rowvec() - Convert to row vector"""
    return np.asarray(x).reshape(1, -1)

def eqlen(*arrays) -> bool:
    """vlt.data.eqlen() - Check if arrays have equal length"""
    lengths = [len(a) for a in arrays]
    return len(set(lengths)) <= 1

def findclosest(array: np.ndarray, value: float) -> int:
    """vlt.data.findclosest() - Find index of nearest value"""
    return int(np.argmin(np.abs(np.asarray(array) - value)))

def dropnan(x: np.ndarray) -> np.ndarray:
    """vlt.data.dropnan() - Remove NaN values"""
    arr = np.asarray(x)
    return arr[~np.isnan(arr)]

def struct_merge(d1: dict, d2: dict) -> dict:
    """vlt.data.structmerge() - Merge dictionaries"""
    return {**d1, **d2}

def cell_or_item(cell_array, index: int, force_cell: bool = False):
    """vlt.data.celloritem() - Handle cell array or item indexing"""
    if isinstance(cell_array, list):
        return cell_array[index] if index < len(cell_array) else None
    return cell_array

def has_all_fields(d: dict, fields: List[str]) -> bool:
    """vlt.data.hasAllFields() - Check if dict has all fields"""
    return all(f in d for f in fields)

def flatten_struct_to_table(d: dict, abbreviations: dict = None) -> pd.DataFrame:
    """vlt.data.flattenstruct2table() - Flatten nested dict to DataFrame"""
    return pd.json_normalize(d)
```

#### vlt.file.* (115 calls) → pathlib/json/custom

```python
# ndi/util/file.py

import json
from pathlib import Path
from typing import List, Union
import os

def textfile2char(filepath: Union[str, Path]) -> str:
    """vlt.file.textfile2char() - Read text file"""
    return Path(filepath).read_text()

def text2cellstr(filepath: Union[str, Path]) -> List[str]:
    """vlt.file.text2cellstr() - Read text file to list of lines"""
    return Path(filepath).read_text().splitlines()

def str2text(filepath: Union[str, Path], content: str) -> None:
    """vlt.file.str2text() - Write string to file"""
    Path(filepath).write_text(content)

def load_struct_array(filepath: Union[str, Path]) -> dict:
    """vlt.file.loadStructArray() - Load JSON struct"""
    with open(filepath) as f:
        return json.load(f)

def save_struct_array(filepath: Union[str, Path], data: dict) -> None:
    """vlt.file.saveStructArray() - Save struct to JSON"""
    with open(filepath, 'w') as f:
        json.dump(data, f, indent=2)

def find_file_groups(directory: Union[str, Path], pattern: str) -> List[List[Path]]:
    """vlt.file.findfilegroups() - Find grouped files"""
    # Implementation depends on pattern format
    pass

def manifest(directory: Union[str, Path]) -> List[Path]:
    """vlt.file.manifest() - List files in directory"""
    return list(Path(directory).iterdir())

class DumbJsonDB:
    """vlt.file.dumbjsondb - Simple JSON file database"""

    def __init__(self, filepath: Union[str, Path]):
        self.filepath = Path(filepath)
        self._data = {}
        if self.filepath.exists():
            self._load()

    def _load(self):
        with open(self.filepath) as f:
            self._data = json.load(f)

    def _save(self):
        with open(self.filepath, 'w') as f:
            json.dump(self._data, f, indent=2)

    def add(self, key: str, value: Any):
        self._data[key] = value
        self._save()

    def get(self, key: str) -> Any:
        return self._data.get(key)

    def remove(self, key: str):
        if key in self._data:
            del self._data[key]
            self._save()

    def search(self, **criteria) -> List[dict]:
        # Implement search logic
        pass
```

#### vlt.neuro.* (14 calls) → scipy/custom

```python
# ndi/util/neuro.py

import numpy as np
from scipy import signal
from sklearn.decomposition import PCA

def stimulus_response_scalar(
    data: np.ndarray,
    stimulus_times: np.ndarray,
    pre_time: float,
    post_time: float,
    sample_rate: float
) -> dict:
    """vlt.neuro.stimulus.stimulus_response_scalar()"""
    # Extract response windows around each stimulus
    pass

def spikewaves_to_pca(waveforms: np.ndarray, n_components: int = 3) -> dict:
    """vlt.neuro.spikesorting.spikewaves2pca()"""
    pca = PCA(n_components=n_components)
    scores = pca.fit_transform(waveforms)
    return {
        'scores': scores,
        'loadings': pca.components_,
        'explained_variance': pca.explained_variance_ratio_
    }

def oversample_spikes(spike_times: np.ndarray, factor: int) -> np.ndarray:
    """vlt.neuro.spikesorting.oversamplespikes()"""
    return np.interp(
        np.linspace(0, len(spike_times)-1, len(spike_times)*factor),
        np.arange(len(spike_times)),
        spike_times
    )
```

### 1.4 NDR-matlab → spikeinterface/neo Mapping

NDR provides unified file reading. Python equivalents:

| NDR Reader | Python Library | Class/Function |
|------------|----------------|----------------|
| Intan RHD | `spikeinterface` | `si.read_intan()` |
| Blackrock NSx/NEV | `spikeinterface` | `si.read_blackrock()` |
| CED Spike2 | `neo` | `neo.io.Spike2IO` |
| Axon ABF | `neo` | `neo.io.AxonIO` |
| TDT SEV | `spikeinterface` | `si.read_tdt()` |
| SpikeGadgets | `spikeinterface` | `si.read_spikegadgets()` |

**Wrapper Interface:**

```python
# ndi/daq/reader.py

from typing import Protocol, List, Tuple, Optional
import numpy as np

class DAQReader(Protocol):
    """Abstract interface matching ndi.daq.reader"""

    def get_channels_epoch(self, epoch_files: List[str]) -> List[dict]:
        """List available channels"""
        ...

    def read_channels_epoch_samples(
        self,
        channel_type: str,
        channel_ids: List[int],
        epoch_files: List[str],
        s0: int,
        s1: int
    ) -> Tuple[np.ndarray, np.ndarray]:
        """Read samples from channels"""
        ...

    def epoch_clock(self, epoch_files: List[str]) -> List[str]:
        """Get available clock types"""
        ...

    def t0_t1(self, epoch_files: List[str]) -> Tuple[float, float]:
        """Get epoch time boundaries"""
        ...

    def sample_rate(self, epoch_files: List[str], channel_type: str, channel_id: int) -> float:
        """Get sampling rate"""
        ...


class SpikeInterfaceReader:
    """Adapter wrapping spikeinterface for NDI compatibility"""

    def __init__(self, format_name: str):
        self.format_name = format_name
        self._reader_map = {
            'intan': self._read_intan,
            'blackrock': self._read_blackrock,
            'tdt': self._read_tdt,
            'spikegadgets': self._read_spikegadgets,
        }

    def _get_recording(self, epoch_files: List[str]):
        import spikeinterface as si
        reader_fn = self._reader_map.get(self.format_name)
        if reader_fn:
            return reader_fn(epoch_files)
        raise ValueError(f"Unknown format: {self.format_name}")

    def _read_intan(self, files):
        import spikeinterface as si
        return si.read_intan(files[0])

    # ... other format readers ...

    def get_channels_epoch(self, epoch_files: List[str]) -> List[dict]:
        recording = self._get_recording(epoch_files)
        channels = []
        for ch_id in recording.get_channel_ids():
            channels.append({
                'name': str(ch_id),
                'type': 'analog_in',
                'id': ch_id
            })
        return channels

    def read_channels_epoch_samples(
        self,
        channel_type: str,
        channel_ids: List[int],
        epoch_files: List[str],
        s0: int,
        s1: int
    ) -> Tuple[np.ndarray, np.ndarray]:
        recording = self._get_recording(epoch_files)
        data = recording.get_traces(
            channel_ids=channel_ids,
            start_frame=s0,
            end_frame=s1
        )
        times = np.arange(s0, s1) / recording.get_sampling_frequency()
        return data, times
```

---

## 2. Conversion Order

### 2.1 Dependency Graph

```
Level 0 (No internal dependencies):
├── did-python (NEW package - must create first)
├── ndi.util.data
├── ndi.util.file
└── ndi.util.neuro

Level 1 (Depends on Level 0):
├── ndi.document
├── ndi.query
├── ndi.ido
└── ndi.time.clocktype

Level 2 (Depends on Level 1):
├── ndi.database (abstract)
├── ndi.time.timemapping
├── ndi.time.timereference
└── ndi.validate

Level 3 (Depends on Level 2):
├── ndi.database.sqlite (implementation)
├── ndi.time.syncgraph
├── ndi.time.syncrule
└── ndi.daq.reader (abstract + spikeinterface adapter)

Level 4 (Depends on Level 3):
├── ndi.daq.system
├── ndi.daq.metadatareader
├── ndi.element
├── ndi.epoch
└── ndi.probe

Level 5 (Depends on Level 4):
├── ndi.session
├── ndi.subject
└── ndi.neuron

Level 6 (Depends on Level 5):
├── ndi.dataset
├── ndi.app (base)
└── ndi.calculator (base)

Level 7 (Depends on Level 6):
├── ndi.app.spikeextractor
├── ndi.app.spikesorter
├── ndi.app.stimulus.*
├── ndi.cloud.* (REST client)
└── ndi.gui.* (optional)
```

### 2.2 Conversion Phases

| Phase | Components | Est. Files | Dependencies |
|-------|-----------|------------|--------------|
| **Phase 0** | did-python package | 10 | None |
| **Phase 1** | ndi.util.* | 5 | numpy, scipy |
| **Phase 2** | ndi.document, ndi.query, ndi.ido | 4 | Phase 0-1 |
| **Phase 3** | ndi.database.* | 3 | Phase 2 |
| **Phase 4** | ndi.time.* | 6 | Phase 2-3 |
| **Phase 5** | ndi.daq.* | 8 | Phase 4, spikeinterface |
| **Phase 6** | ndi.element, ndi.epoch, ndi.probe | 5 | Phase 5 |
| **Phase 7** | ndi.session | 1 | Phase 6 |
| **Phase 8** | ndi.dataset, ndi.subject, ndi.neuron | 3 | Phase 7 |
| **Phase 9** | ndi.app.*, ndi.calculator.* | 10 | Phase 8 |
| **Phase 10** | ndi.cloud.* | 15 | Phase 8 |

---

## 3. Phase Breakdown

### Phase 0: Verify and Integrate Existing Dependencies (ALREADY DONE)

**Status:** ✅ COMPLETE - These packages already exist

**DID-python:** https://github.com/VH-Lab/DID-python
- 170 commits, 5 contributors
- Contains: `did.document`, `did.query`, `did.database`, `did.implementations/`

**vhlab-toolbox-python:** https://github.com/VH-Lab/vhlab-toolbox-python
- 46 commits, explicitly for NDI support
- Contains: 34 `vlt.data` functions, 9 `vlt.file` functions
- See `PORTING_PROGRESS.md` in that repo for details

**Action Items for Phase 0:**
1. [ ] Clone and test DID-python locally
2. [ ] Clone and test vhlab-toolbox-python locally
3. [ ] Verify all NDI-required functions exist (see Section 1.4)
4. [ ] If functions missing, contribute PRs to those repos
5. [ ] Document any gaps

**DO NOT recreate these packages. Use them as dependencies:**

```python
# pyproject.toml for ndi-python
[project]
dependencies = [
    "did-python @ git+https://github.com/VH-Lab/DID-python.git",
    "vhlab-toolbox-python @ git+https://github.com/VH-Lab/vhlab-toolbox-python.git",
    "vhlab-newstim-python @ git+https://github.com/VH-Lab/vhlab-NewStim-python.git",
    # ... other deps
]
```

**Key Classes:**

```python
# did/document.py
from dataclasses import dataclass, field
from typing import Any, Dict, List, Optional
import json

@dataclass
class Document:
    """Base document class for DID database"""
    document_properties: Dict[str, Any] = field(default_factory=dict)

    @classmethod
    def from_json(cls, json_str: str) -> 'Document':
        """Create document from JSON string"""
        props = json.loads(json_str)
        return cls(document_properties=props)

    @classmethod
    def read_blank_definition(cls, json_file_location: str) -> 'Document':
        """Create blank document from schema definition"""
        # Resolve path using PathConstants
        # Read JSON schema
        # Recursively merge superclasses
        pass

    def to_json(self) -> str:
        """Serialize to JSON"""
        return json.dumps(self.document_properties, indent=2)

    @property
    def id(self) -> str:
        return self.document_properties.get('base', {}).get('id', '')

    def set_property(self, path: str, value: Any) -> 'Document':
        """Set nested property by dot-path"""
        parts = path.split('.')
        d = self.document_properties
        for part in parts[:-1]:
            d = d.setdefault(part, {})
        d[parts[-1]] = value
        return self


# did/query.py
from dataclasses import dataclass
from typing import Any, List, Union
from enum import Enum

class QueryOp(Enum):
    EXACT_STRING = 'exact_string'
    EXACT_STRING_ANYCASE = 'exact_string_anycase'
    CONTAINS_STRING = 'contains_string'
    REGEXP = 'regexp'
    EXACT_NUMBER = 'exact_number'
    LESSTHAN = 'lessthan'
    LESSTHANEQ = 'lessthaneq'
    GREATERTHAN = 'greaterthan'
    GREATERTHANEQ = 'greaterthaneq'
    HASFIELD = 'hasfield'
    ISA = 'isa'
    DEPENDS_ON = 'depends_on'

@dataclass
class Query:
    """Search query for DID database"""
    field: str
    operation: Union[QueryOp, str]
    param1: Any = None
    param2: Any = None

    def __and__(self, other: 'Query') -> 'Query':
        """Combine with AND"""
        return AndQuery(self, other)

    def __or__(self, other: 'Query') -> 'Query':
        """Combine with OR"""
        return OrQuery(self, other)

    def __invert__(self) -> 'Query':
        """Negate query"""
        return NotQuery(self)

    def to_search_structure(self) -> dict:
        """Convert to search dict for database"""
        return {
            'field': self.field,
            'operation': self.operation.value if isinstance(self.operation, QueryOp) else self.operation,
            'param1': self.param1,
            'param2': self.param2
        }


# did/ido.py
import uuid
import re

def unique_id() -> str:
    """Generate unique identifier"""
    return uuid.uuid4().hex

def is_valid(id_str: str) -> bool:
    """Validate identifier format"""
    return bool(re.match(r'^[a-f0-9]{32}$', id_str))


# did/implementations/sqlitedb.py
import sqlite3
from pathlib import Path
from typing import List, Optional
import json

class SQLiteDB:
    """SQLite-based document database"""

    def __init__(self, db_path: str):
        self.db_path = Path(db_path)
        self.conn = None
        self._init_db()

    def _init_db(self):
        self.conn = sqlite3.connect(str(self.db_path))
        self.conn.execute('''
            CREATE TABLE IF NOT EXISTS documents (
                id TEXT PRIMARY KEY,
                doc_class TEXT,
                properties TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        self.conn.execute('''
            CREATE TABLE IF NOT EXISTS files (
                doc_id TEXT,
                file_name TEXT,
                file_path TEXT,
                PRIMARY KEY (doc_id, file_name)
            )
        ''')
        self.conn.commit()

    def add_docs(self, documents: List['Document']) -> None:
        for doc in documents:
            self.conn.execute(
                'INSERT OR REPLACE INTO documents (id, doc_class, properties) VALUES (?, ?, ?)',
                (doc.id, doc.document_properties.get('document_class', {}).get('class_name', ''),
                 json.dumps(doc.document_properties))
            )
        self.conn.commit()

    def get_docs(self, doc_ids: List[str]) -> List['Document']:
        results = []
        for doc_id in doc_ids:
            row = self.conn.execute(
                'SELECT properties FROM documents WHERE id = ?', (doc_id,)
            ).fetchone()
            if row:
                from did.document import Document
                results.append(Document(document_properties=json.loads(row[0])))
        return results

    def remove_docs(self, doc_ids: List[str]) -> None:
        for doc_id in doc_ids:
            self.conn.execute('DELETE FROM documents WHERE id = ?', (doc_id,))
            self.conn.execute('DELETE FROM files WHERE doc_id = ?', (doc_id,))
        self.conn.commit()

    def search(self, query: 'Query') -> List['Document']:
        """Execute query and return matching documents"""
        # Convert query to SQL WHERE clause
        # Execute and return results
        pass

    def get_doc_ids(self) -> List[str]:
        rows = self.conn.execute('SELECT id FROM documents').fetchall()
        return [row[0] for row in rows]
```

**Acceptance Criteria:**
- [ ] `did.ido.unique_id()` generates valid UUIDs
- [ ] `did.Document` can load/save JSON
- [ ] `did.Query` supports all operators
- [ ] `did.implementations.SQLiteDB` passes all CRUD tests
- [ ] Package installable via pip

---

### Phase 1: ndi.util Package

**Files to Create:**

```
ndi/
├── util/
│   ├── __init__.py
│   ├── data.py          # vlt.data.* equivalents
│   ├── file.py          # vlt.file.* equivalents
│   ├── neuro.py         # vlt.neuro.* equivalents
│   ├── signal.py        # vlt.signal.* equivalents
│   └── string.py        # vlt.string.* equivalents
```

**Functions Required:** See Section 1.3 for complete mapping.

**Acceptance Criteria:**
- [ ] All 87 vlt.* functions have Python equivalents
- [ ] Unit tests cover edge cases (NaN, empty arrays, etc.)
- [ ] Performance comparable to numpy operations

---

### Phase 2: ndi.document, ndi.query, ndi.ido

**Files to Create:**

```
ndi/
├── document.py
├── query.py
├── ido.py
└── validate.py
```

**ndi.document API:**

```python
# ndi/document.py
from typing import Any, Dict, List, Optional, Tuple
from did.document import Document as DIDDocument
from ndi.ido import IDO
from ndi.util.file import load_struct_array
import ndi.common.path_constants as paths

class Document(DIDDocument):
    """NDI Document - extends DID Document with NDI-specific features"""

    def __init__(self, document_type: str = 'base', **kwargs):
        """Create new NDI document

        Args:
            document_type: Schema type name (without .json extension)
            **kwargs: Property name/value pairs (e.g., 'base.name'='my_doc')
        """
        if isinstance(document_type, dict):
            # Creating from existing properties
            super().__init__(document_properties=document_type)
        else:
            # Creating blank from definition
            props = self.read_blank_definition(document_type)
            props['base']['id'] = IDO().id()
            props['base']['datestamp'] = self._timestamp()
            for key, value in kwargs.items():
                self._set_nested(props, key, value)
            super().__init__(document_properties=props)

    def set_session_id(self, session_id: str) -> 'Document':
        """Set the session ID"""
        self.document_properties['base']['session_id'] = session_id
        return self

    def add_file(self, name: str, location: str,
                 ingest: bool = True, delete_original: bool = True) -> 'Document':
        """Add file to document"""
        # Validate name is in file_list
        # Add to file_info
        pass

    def dependency(self) -> Tuple[List[str], List[dict]]:
        """Return dependency names and structures"""
        deps = self.document_properties.get('depends_on', [])
        names = [d['name'] for d in deps]
        return names, deps

    def dependency_value(self, name: str, error_if_not_found: bool = True) -> Optional[str]:
        """Get dependency value by name"""
        deps = self.document_properties.get('depends_on', [])
        for dep in deps:
            if dep['name'] == name:
                return dep['value']
        if error_if_not_found:
            raise KeyError(f"Dependency '{name}' not found")
        return None

    def set_dependency_value(self, name: str, value: str,
                             error_if_not_found: bool = True) -> 'Document':
        """Set dependency value"""
        deps = self.document_properties.setdefault('depends_on', [])
        for dep in deps:
            if dep['name'] == name:
                dep['value'] = value
                return self
        if error_if_not_found:
            raise KeyError(f"Dependency '{name}' not found")
        deps.append({'name': name, 'value': value})
        return self

    def doc_isa(self, document_class: str) -> bool:
        """Check if document is of given class (including superclasses)"""
        classes = [self.doc_class()] + self.doc_superclass()
        return document_class in classes

    def doc_class(self) -> str:
        """Get document class name"""
        return self.document_properties['document_class']['class_name']

    def doc_superclass(self) -> List[str]:
        """Get list of superclass names"""
        superclasses = self.document_properties['document_class'].get('superclasses', [])
        result = []
        for sc in superclasses:
            sc_doc = Document.read_blank_definition(sc['definition'])
            result.append(sc_doc['document_class']['class_name'])
        return list(set(result))

    def to_table(self) -> 'pd.DataFrame':
        """Convert to pandas DataFrame"""
        import pandas as pd
        return pd.json_normalize(self.document_properties)

    def __eq__(self, other: 'Document') -> bool:
        """Equality by ID"""
        return self.id == other.id

    def __add__(self, other: 'Document') -> 'Document':
        """Merge documents"""
        # Merge superclasses
        # Merge depends_on
        # Merge files
        # Merge other properties
        pass
```

**ndi.query API:**

```python
# ndi/query.py
from did.query import Query as DIDQuery, QueryOp

class Query(DIDQuery):
    """NDI Query - extends DID Query"""

    @classmethod
    def isa(cls, document_class: str) -> 'Query':
        """Shorthand for ISA query"""
        return cls('', QueryOp.ISA, document_class)

    @classmethod
    def depends_on(cls, dependency_name: str, value: str) -> 'Query':
        """Shorthand for DEPENDS_ON query"""
        return cls('', QueryOp.DEPENDS_ON, dependency_name, value)

    @classmethod
    def by_id(cls, doc_id: str) -> 'Query':
        """Query by document ID"""
        return cls('base.id', QueryOp.EXACT_STRING, doc_id)
```

---

### Phase 3: ndi.database

**Files to Create:**

```
ndi/
├── database.py                    # Abstract base
└── database/
    ├── __init__.py
    └── implementations/
        ├── __init__.py
        ├── matlabdumbjsondb.py    # JSON file backend
        └── didsqlite.py           # SQLite backend
```

**ndi.database API:**

```python
# ndi/database.py
from abc import ABC, abstractmethod
from typing import List, Optional, BinaryIO
from ndi.document import Document
from ndi.query import Query

class Database(ABC):
    """Abstract base class for NDI database"""

    @abstractmethod
    def add(self, document: Document) -> None:
        """Add or update document"""
        pass

    @abstractmethod
    def read(self, doc_id: str) -> Optional[Document]:
        """Read document by ID"""
        pass

    @abstractmethod
    def remove(self, doc_id: str) -> None:
        """Remove document"""
        pass

    @abstractmethod
    def search(self, query: Query) -> List[Document]:
        """Search for documents matching query"""
        pass

    @abstractmethod
    def all_doc_ids(self) -> List[str]:
        """Get all document IDs"""
        pass

    @abstractmethod
    def open_binary_doc(self, document: Document, filename: str) -> BinaryIO:
        """Open binary file associated with document"""
        pass

    @abstractmethod
    def close_binary_doc(self, file_obj: BinaryIO) -> None:
        """Close binary file"""
        pass

    def clear(self, confirm: str = '') -> None:
        """Delete all documents (requires confirmation)"""
        if confirm != 'yes':
            raise ValueError("Must pass confirm='yes' to clear database")
        for doc_id in self.all_doc_ids():
            self.remove(doc_id)
```

---

### Phase 4: ndi.time

**Files to Create:**

```
ndi/
└── time/
    ├── __init__.py
    ├── clocktype.py
    ├── timereference.py
    ├── timemapping.py
    ├── syncgraph.py
    └── syncrule/
        ├── __init__.py
        ├── base.py
        ├── filematch.py
        └── filefind.py
```

**Key Classes:**

```python
# ndi/time/clocktype.py
from enum import Enum

class ClockType(Enum):
    """Clock types for time synchronization"""
    UTC = 'utc'
    APPROX_UTC = 'approx_utc'
    EXP_GLOBAL_TIME = 'exp_global_time'
    APPROX_EXP_GLOBAL_TIME = 'approx_exp_global_time'
    DEV_GLOBAL_TIME = 'dev_global_time'
    APPROX_DEV_GLOBAL_TIME = 'approx_dev_global_time'
    DEV_LOCAL_TIME = 'dev_local_time'
    NO_TIME = 'no_time'
    INHERITED = 'inherited'

    def ndiglobal(self) -> bool:
        """Is this a global NDI clock?"""
        return self in [
            ClockType.UTC, ClockType.APPROX_UTC,
            ClockType.EXP_GLOBAL_TIME, ClockType.APPROX_EXP_GLOBAL_TIME
        ]


# ndi/time/timemapping.py
import numpy as np
from typing import List, Union

class TimeMapping:
    """Polynomial time mapping between clocks

    t_out = coefficients[0]*t_in^N + ... + coefficients[N]
    """

    def __init__(self, coefficients: List[float]):
        self.coefficients = np.array(coefficients)

    def map(self, t: Union[float, np.ndarray]) -> Union[float, np.ndarray]:
        """Apply mapping to time value(s)"""
        t = np.asarray(t)
        result = np.zeros_like(t)
        for i, coef in enumerate(self.coefficients):
            power = len(self.coefficients) - 1 - i
            result += coef * (t ** power)
        return result

    @classmethod
    def linear(cls, scale: float = 1.0, shift: float = 0.0) -> 'TimeMapping':
        """Create linear mapping: t_out = scale * t_in + shift"""
        return cls([scale, shift])

    @classmethod
    def identity(cls) -> 'TimeMapping':
        """Create identity mapping"""
        return cls.linear(1.0, 0.0)


# ndi/time/syncgraph.py
from typing import Dict, List, Tuple, Optional
import networkx as nx
from ndi.time.timemapping import TimeMapping
from ndi.time.clocktype import ClockType

class SyncGraph:
    """Graph-based time synchronization"""

    def __init__(self, session: 'Session'):
        self.session = session
        self._graph = nx.DiGraph()
        self._rules = []

    def add_rule(self, rule: 'SyncRule') -> None:
        """Add synchronization rule"""
        self._rules.append(rule)

    def build_graph_info(self) -> None:
        """Build synchronization graph from rules"""
        for rule in self._rules:
            edges = rule.apply(self.session)
            for src, dst, mapping in edges:
                self._graph.add_edge(src, dst, mapping=mapping)

    def time_convert(
        self,
        t: float,
        src_device: str,
        src_clock: ClockType,
        src_epoch: str,
        dst_device: str,
        dst_clock: ClockType,
        dst_epoch: str
    ) -> float:
        """Convert time between devices/clocks"""
        src_node = (src_device, src_clock, src_epoch)
        dst_node = (dst_device, dst_clock, dst_epoch)

        try:
            path = nx.shortest_path(self._graph, src_node, dst_node)
        except nx.NetworkXNoPath:
            raise ValueError(f"No path from {src_node} to {dst_node}")

        result = t
        for i in range(len(path) - 1):
            edge_data = self._graph.edges[path[i], path[i+1]]
            mapping = edge_data['mapping']
            result = mapping.map(result)

        return result
```

---

### Phase 5: ndi.daq

**Files to Create:**

```
ndi/
└── daq/
    ├── __init__.py
    ├── system.py
    ├── reader/
    │   ├── __init__.py
    │   ├── base.py
    │   ├── mfdaq/
    │   │   ├── __init__.py
    │   │   ├── base.py
    │   │   └── spikeinterface_adapter.py
    ├── metadatareader/
    │   ├── __init__.py
    │   ├── base.py
    │   └── newstim.py
    └── file/
        ├── __init__.py
        ├── navigator/
        │   ├── __init__.py
        │   ├── base.py
        │   └── epochdir.py
```

---

### Phase 6-10: Remaining Components

See Section 2.2 for component list. Each phase follows the same pattern:
1. Define Python API matching MATLAB functionality
2. Implement with Python idioms (dataclasses, type hints, etc.)
3. Write tests against MATLAB behavior
4. Update CONVERSION_STATUS.md

---

## 4. Python Package Structure

```
ndi-python/
├── pyproject.toml
├── README.md
├── LICENSE
├── docs/
│   ├── getting_started.md
│   └── api/
├── ndi/
│   ├── __init__.py
│   ├── session.py
│   ├── document.py
│   ├── query.py
│   ├── database.py
│   ├── element.py
│   ├── epoch.py
│   ├── probe.py
│   ├── neuron.py
│   ├── subject.py
│   ├── dataset.py
│   ├── validate.py
│   ├── ido.py
│   ├── cache.py
│   ├── version.py
│   ├── common/
│   │   ├── __init__.py
│   │   └── path_constants.py
│   ├── util/
│   │   ├── __init__.py
│   │   ├── data.py
│   │   ├── file.py
│   │   ├── neuro.py
│   │   └── signal.py
│   ├── database/
│   │   ├── __init__.py
│   │   └── implementations/
│   │       ├── __init__.py
│   │       ├── jsondb.py
│   │       └── sqlitedb.py
│   ├── daq/
│   │   ├── __init__.py
│   │   ├── system.py
│   │   ├── reader/
│   │   │   ├── __init__.py
│   │   │   └── mfdaq/
│   │   │       ├── __init__.py
│   │   │       └── spikeinterface.py
│   │   ├── metadatareader/
│   │   │   ├── __init__.py
│   │   │   └── newstim.py
│   │   └── file/
│   │       ├── __init__.py
│   │       └── navigator/
│   │           ├── __init__.py
│   │           └── epochdir.py
│   ├── time/
│   │   ├── __init__.py
│   │   ├── clocktype.py
│   │   ├── timemapping.py
│   │   ├── timereference.py
│   │   ├── syncgraph.py
│   │   └── syncrule/
│   │       ├── __init__.py
│   │       ├── base.py
│   │       └── filematch.py
│   ├── app/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── spikeextractor.py
│   │   ├── spikesorter.py
│   │   └── stimulus/
│   │       ├── __init__.py
│   │       ├── decoder.py
│   │       └── tuning_response.py
│   ├── calc/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── stimulus/
│   │       └── tuningcurve.py
│   ├── cloud/
│   │   ├── __init__.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── datasets.py
│   │   │   ├── documents.py
│   │   │   └── files.py
│   │   ├── sync/
│   │   │   ├── __init__.py
│   │   │   └── sync.py
│   │   ├── upload/
│   │   │   └── __init__.py
│   │   └── download/
│   │       └── __init__.py
│   └── ontology/
│       ├── __init__.py
│       ├── base.py
│       ├── rrid.py
│       └── snomed.py
├── ndi_common/                    # SHARED with MATLAB (symlink or copy)
│   ├── database_documents/        # 83 JSON schemas
│   ├── schema_documents/          # 83 JSON Schema validators
│   ├── daq_systems/               # Lab configs
│   ├── probe/                     # Probe definitions
│   └── ...
└── tests/
    ├── conftest.py
    ├── test_document.py
    ├── test_query.py
    ├── test_database.py
    ├── test_session.py
    ├── test_daq/
    │   └── test_spikeinterface.py
    ├── test_time/
    │   └── test_syncgraph.py
    └── integration/
        └── test_matlab_compatibility.py
```

**pyproject.toml:**

```toml
[project]
name = "ndi"
version = "0.1.0"
description = "Neuroscience Data Interface - Python Implementation"
requires-python = ">=3.9"
dependencies = [
    "numpy>=1.21",
    "scipy>=1.7",
    "pandas>=1.3",
    "spikeinterface>=0.99",
    "neo>=0.12",
    "networkx>=2.6",
    "requests>=2.28",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "mypy>=1.0",
    "ruff>=0.1",
]
cloud = [
    "boto3>=1.28",
    "pyjwt>=2.8",
]

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=ndi --cov-report=term-missing"

[tool.mypy]
python_version = "3.9"
strict = true

[tool.ruff]
line-length = 100
select = ["E", "F", "I", "N", "W"]
```

---

## 5. API Design Specifications

### 5.1 Naming Conventions

| MATLAB | Python |
|--------|--------|
| `ndi.session` | `ndi.Session` (class) |
| `ndi_document_obj.doc_class()` | `document.doc_class()` |
| `ndi_document_obj.setproperties()` | `document.set_properties()` |
| `my_func(arg, 'Name', value)` | `my_func(arg, name=value)` |
| `varargin` | `**kwargs` |
| `varargout` | `tuple` return |
| `emptystruct('a','b')` | `empty_dict('a', 'b')` |

### 5.2 Return Value Conventions

| MATLAB Pattern | Python Pattern |
|----------------|----------------|
| `[a, b] = func()` | `return a, b` (tuple unpacking) |
| `b = func()` with error | `return value` or raise exception |
| Empty `[]` | `None` or empty `[]`/`{}` |
| Struct array | `List[dict]` or `List[dataclass]` |
| Cell array | `List[Any]` |

### 5.3 Error Handling

```python
# MATLAB: error('Message')
# Python: raise specific exception

class NDIError(Exception):
    """Base exception for NDI"""
    pass

class DocumentNotFoundError(NDIError):
    """Document not found in database"""
    pass

class ValidationError(NDIError):
    """Document validation failed"""
    pass

class SyncError(NDIError):
    """Time synchronization error"""
    pass
```

---

## 6. Testing Strategy

### 6.1 Test Categories

1. **Unit Tests** - Individual function/method tests
2. **Integration Tests** - Component interaction tests
3. **Compatibility Tests** - Verify Python matches MATLAB output
4. **Performance Tests** - Ensure acceptable performance

### 6.2 Compatibility Testing Approach

```python
# tests/integration/test_matlab_compatibility.py

import pytest
import scipy.io
import numpy as np
from pathlib import Path

MATLAB_OUTPUT_DIR = Path("tests/fixtures/matlab_outputs")

class TestDocumentCompatibility:
    """Verify Python documents match MATLAB documents"""

    def test_document_creation(self):
        """Document properties should match MATLAB output"""
        # Load MATLAB-created document
        matlab_doc = scipy.io.loadmat(
            MATLAB_OUTPUT_DIR / "spike_clusters_doc.mat"
        )['doc']

        # Create equivalent Python document
        from ndi import Document
        py_doc = Document('spike_clusters')
        py_doc.set_dependency_value('element_id', 'test123')

        # Compare structure
        assert py_doc.doc_class() == matlab_doc['document_class']['class_name']

    def test_query_results(self):
        """Query should return same results as MATLAB"""
        # Load MATLAB query results
        # Execute same query in Python
        # Compare document IDs
        pass
```

### 6.3 Test Data

Use `ndi_common/example_sessions/` for integration tests:
- Pre-recorded data with known structure
- Expected query results documented
- Time synchronization test cases

### 6.4 CI/CD Pipeline

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          pip install -e ".[dev]"

      - name: Run tests
        run: pytest --cov=ndi --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

---

## 7. Schema Migration

### 7.1 JSON Schemas Are Language-Agnostic

The 83 JSON schemas in `ndi_common/database_documents/` require **NO conversion**. They define:
- Document structure
- Field types
- Inheritance relationships
- File lists

Python code reads these directly using `json.load()`.

### 7.2 Schema Validation

```python
# ndi/validate.py
import json
import jsonschema
from pathlib import Path

SCHEMA_DIR = Path(__file__).parent / "ndi_common" / "schema_documents"

def validate_document(document: 'Document') -> bool:
    """Validate document against its schema"""
    doc_class = document.doc_class()
    schema_path = SCHEMA_DIR / f"{doc_class}_schema.json"

    if not schema_path.exists():
        raise FileNotFoundError(f"Schema not found: {schema_path}")

    with open(schema_path) as f:
        schema = json.load(f)

    try:
        jsonschema.validate(document.document_properties, schema)
        return True
    except jsonschema.ValidationError as e:
        raise ValidationError(str(e))
```

### 7.3 Document Type Catalog

All 83 document types are listed in `REPOSITORY_ANALYSIS.md`. Key categories:

| Category | Count | Examples |
|----------|-------|----------|
| Foundation | 9 | base, session, subject, element, app |
| Epoch/Temporal | 4 | epochid, element_epoch, epochclocktimes |
| DAQ | 7 | daqsystem, daqreader, syncgraph |
| Stimulus | 11 | stimulus_presentation, stimulus_response |
| Electrophysiology | 10 | spike_clusters, probe_geometry |
| Analysis | 10 | tuningcurve_calc, sorting_parameters |
| Subject | 6 | treatment, virus_injection |
| Data Ingestion | 6 | epochfiles_ingested |
| Metadata | 9 | openminds, dataset_remote |

---

## 8. Progress Tracking

### 8.1 Status File

Create `CONVERSION_STATUS.md` to track progress:

```markdown
# NDI Python Conversion Status

Last Updated: YYYY-MM-DD

## Phase Status

| Phase | Component | Status | Tests | Notes |
|-------|-----------|--------|-------|-------|
| 0 | did-python | ⬜ Not Started | 0% | |
| 1 | ndi.util | ⬜ Not Started | 0% | |
| 2 | ndi.document | ⬜ Not Started | 0% | |
| ... | ... | ... | ... | ... |

## Legend
- ⬜ Not Started
- 🟡 In Progress
- ✅ Complete
- 🔴 Blocked

## Detailed Component Status

### did-python
- [ ] did.ido
- [ ] did.document
- [ ] did.query
- [ ] did.database
- [ ] did.implementations.sqlitedb

### ndi.util
- [ ] ndi.util.data (69 functions)
- [ ] ndi.util.file (25 functions)
- [ ] ndi.util.neuro (10 functions)
- [ ] ndi.util.signal (6 functions)
- [ ] ndi.util.string (6 functions)

...
```

### 8.2 Git Workflow

```bash
# Feature branch per phase
git checkout -b phase-0-did-python
# ... implement ...
git commit -m "Phase 0: Implement did.ido with unique_id() and is_valid()"
git commit -m "Phase 0: Implement did.document with JSON serialization"
# ... etc ...

# Merge when phase complete and tests pass
git checkout main
git merge phase-0-did-python
```

---

## 9. Risk Mitigation

### 9.1 Technical Risks

| Risk | Mitigation |
|------|------------|
| MATLAB-specific file formats | Use spikeinterface/neo which support same formats |
| Complex time synchronization | Port syncgraph logic directly; well-tested in MATLAB |
| Performance regression | Benchmark critical paths; numpy is usually faster |
| Cloud API compatibility | Use same REST endpoints; test against dev server |
| Custom binary formats (.vhsb) | Port format readers directly or create Python equivalents |

### 9.2 Process Risks

| Risk | Mitigation |
|------|------------|
| Context loss between sessions | Reference docs (this file), CONVERSION_STATUS.md |
| Scope creep | Strict phase boundaries; no "improvements" |
| Dependency drift | Pin versions in pyproject.toml |
| MATLAB version updates | Track NDI-matlab releases; sync schemas |

### 9.3 Quality Gates

Before marking any phase complete:

1. ✅ All functions implemented
2. ✅ Unit tests pass (>90% coverage)
3. ✅ Type hints complete (mypy passes)
4. ✅ Docstrings for all public APIs
5. ✅ Integration test with example data
6. ✅ CONVERSION_STATUS.md updated

---

## 10. Session Handoff Protocol

### 10.1 Starting a New Session

When beginning work on conversion:

1. **Read these files in order:**
   - `REPOSITORY_ANALYSIS.md` - Understand the system
   - `PYTHON_CONVERSION_PLAN.md` - Understand the plan (this file)
   - `CONVERSION_STATUS.md` - See current progress

2. **Check git status:**
   ```bash
   git status
   git log --oneline -10
   ```

3. **Identify next task:**
   - Find first incomplete item in CONVERSION_STATUS.md
   - Review the phase requirements in this document

### 10.2 Ending a Session

Before stopping work:

1. **Commit all changes:**
   ```bash
   git add -A
   git commit -m "Phase X: Description of what was done"
   ```

2. **Update CONVERSION_STATUS.md:**
   - Mark completed items
   - Note any blockers or issues
   - Update "Last Updated" date

3. **Push to remote:**
   ```bash
   git push origin <branch-name>
   ```

4. **Document any context:**
   - Add notes to CONVERSION_STATUS.md for next session
   - Note any decisions made or issues encountered

### 10.3 Resuming Work

The status file and git history provide full context. Any developer (human or AI) can:
1. Read status to see what's done
2. Read this plan to see what's next
3. Continue from the exact stopping point

---

## Appendix A: Complete VLT Function Mapping

See Section 1.3 for the 87 vlt.* functions and their Python equivalents.

## Appendix B: Complete DID Function Mapping

See Section 1.2 for the complete did.* function mapping.

## Appendix C: File Format Support Matrix

| Format | MATLAB Reader | Python Library | Status |
|--------|---------------|----------------|--------|
| Intan RHD | NDR + ndi.daq.reader.mfdaq.intan | spikeinterface | ✅ Supported |
| Blackrock NSx/NEV | NDR + ndi.daq.reader.mfdaq.blackrock | spikeinterface | ✅ Supported |
| CED Spike2 | ndi.daq.reader.mfdaq.cedspike2 | neo.io.Spike2IO | ✅ Supported |
| Axon ABF | NDR | neo.io.AxonIO | ✅ Supported |
| TDT SEV | NDR | spikeinterface | ✅ Supported |
| SpikeGadgets | ndi.daq.reader.mfdaq.spikegadgets | spikeinterface | ✅ Supported |
| VH Lab .vhsb | vlt.file.custom_file_formats | Custom port needed | ⚠️ Port required |

---

## Revision History

| Date | Author | Changes |
|------|--------|---------|
| 2026-02-03 | Claude | Initial comprehensive plan |
