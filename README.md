# STAC Partition Extension

- **Title:** Partition
- **Identifier:** <https://portolan-sdi.github.io/stac-partition-extension/v1.0.0/schema.json>
- **Field Name Prefix:** partition
- **Scope:** Collection, Asset
- **[Extension Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions#extension-maturity):** Proposal

This extension adds Hive-style partition metadata to STAC Collections, enabling clients to discover partition structure and construct efficient queries using DuckDB, PyArrow, or other partition-aware readers.

## Why a Separate Extension?

The [STAC Table Extension](https://github.com/stac-extensions/table) provides schema-level metadata (`table:columns`, `table:row_count`). Partitioning is orthogonal — it describes *how data is organized on storage*, not the logical schema. Per [guidance from the table extension maintainer](https://github.com/stac-extensions/table/issues/12), partition metadata belongs in a separate extension.

## Fields

### Collection Fields

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| partition:scheme | string | **Yes** | Partitioning scheme: `hive` (key=value dirs) or `directory` (plain subdirs) |
| partition:strategy | string | No | Algorithm used: `kdtree`, `h3`, `s2`, `quadkey`, `a5`, `temporal`, `attribute` |
| partition:keys | \[object\] | **Yes** | Partition key definitions (see below) |
| partition:file_count | integer | No | Total number of data files across all partitions |

### Partition Key Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | **Yes** | Column name used as partition key |
| type | string | **Yes** | Data type: `string`, `int32`, `int64`, `date`, `timestamp` |
| description | string | No | Human-readable description |

### Asset Fields

| Field Name | Type | Description |
|------------|------|-------------|
| partition:glob | string | Absolute glob pattern for bulk access (e.g., `s3://bucket/col/key=*/*.parquet`) |

## Example

```json
{
  "type": "Collection",
  "id": "buildings",
  "stac_extensions": [
    "https://stac-extensions.github.io/table/v1.2.0/schema.json",
    "https://portolan-sdi.github.io/stac-partition-extension/v1.0.0/schema.json"
  ],
  "partition:scheme": "hive",
  "partition:strategy": "kdtree",
  "partition:keys": [
    {
      "name": "kdtree_cell",
      "type": "string",
      "description": "KD-tree spatial partition cell identifier"
    }
  ],
  "partition:file_count": 98,
  "assets": {
    "data": {
      "href": "./kdtree_cell=0/*.parquet",
      "type": "application/vnd.apache.parquet",
      "roles": ["data"],
      "partition:glob": "s3://example-bucket/buildings/kdtree_cell=*/*.parquet"
    }
  }
}
```

## Consumer Examples

### DuckDB

```sql
-- Direct glob query with partition pruning
SELECT * FROM read_parquet('s3://bucket/buildings/kdtree_cell=*/*.parquet')
WHERE kdtree_cell = '0_1_0';

-- Hive partitioning auto-detection
SELECT * FROM read_parquet(
  's3://bucket/buildings/kdtree_cell=*/*.parquet',
  hive_partitioning = true
);
```

### PyArrow

```python
import pyarrow.dataset as ds

dataset = ds.dataset(
    "s3://bucket/buildings/",
    partitioning="hive",
    format="parquet"
)

# Partition pruning happens automatically
table = dataset.to_table(
    filter=ds.field("kdtree_cell") == "0_1_0"
)
```

## Relation to Other Extensions

| Extension | Provides | Partition Extension Adds |
|-----------|----------|-------------------------|
| [Table](https://github.com/stac-extensions/table) | Schema, row count, geometry column | Partition structure, strategy, bulk access |
| [Iceberg](https://github.com/portolan-sdi/stac-iceberg-extension) | Catalog connectivity, snapshots | N/A (Iceberg has its own partition spec) |

Use Partition Extension for static partitioned Parquet/GeoParquet files. Use Iceberg Extension for managed Iceberg tables with transactional semantics.

## Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `kdtree` | Data-driven spatial partitioning via KD-tree | Uneven spatial distributions |
| `h3` | Uber H3 hexagonal grid | Global datasets, multi-resolution |
| `s2` | Google S2 spherical geometry | Global datasets, polar regions |
| `quadkey` | Bing Maps quadkey tiles | Web mapping integration |
| `a5` | A5 grid system | European datasets |
| `temporal` | Date/time-based partitioning | Time series data |
| `attribute` | Arbitrary column-based partitioning | Categorical data |

## Building and Testing

```bash
npm install
npm test
```

## Reference Implementation

[Portolan CLI](https://github.com/portolan-sdi/portolan-cli) generates STAC Collections with partition extension fields when adding partitioned GeoParquet datasets. See `portolan_cli/partitioning.py`.

## Contributing

Issues and pull requests are welcome. This extension is maintained by the [Portolan SDI](https://github.com/portolan-sdi) project.

## License

Apache-2.0
