### Release [1.5.3], 2023-07-27
#### Bug Fixes


### Release [1.4.4], 2023-07-19
#### Bug Fixes
- Fixed two logging/exception handling issues that would cause exception on startup or when handling some exceptions
from the ClickHouse server.  Partially addresses https://github.com/ClickHouse/dbt-clickhouse/issues/169.
- Fixed issue with the `on_cluster` macro that would break the exchange tables step of incremental materializations
with an active cluster.  Thanks to [Andrew Davis](https://github.com/Savid) for the PR.  Closes
https://github.com/ClickHouse/dbt-clickhouse/issues/167

### Release [1.4.3], 2023-06-27
#### Bug Fix
- Use correct return value for `execute`.  This would cause an exception when running hooks.  Thanks to
[Sergey Reshetnikov](https://github.com/PrVrSs) for the PR.  Closed https://github.com/ClickHouse/dbt-clickhouse/issues/161

#### Improvement
- Added macros for creating distributed tables.  See the `distributed_table.sql` include file.  Thanks to
[gladkikhtutu](https://github.com/gladkikhtutu) for the contribution.  

### Release [1.4.2], 2023-05-14
#### Bug fixes
- Create initial dbt database (if not found) on the defined cluster on first run, instead of just the execution node.
Thanks to [Jens Hoevenaars](https://github.com/codenation-nl) for the PR
- Fix the SYSTEM SYNC REPLICA statement when exchanging tables ON CLUSTER for incremental materializations.  Thanks to
[Schum](https://github.com/Schum-io) for PR.  Closed https://github.com/ClickHouse/dbt-clickhouse/issues/157.

### Release [1.4.1], 2023-05-11
#### Improvements
- Reduce the number of SQL calls for Modify Comment operations.  Thanks to [Konstantin Ilchenko](https://github.com/simpl1g).
- Add "on cluster" to several additional macros to better support distributed tables.  Thanks to [Saurabh Bikram](https://github.com/saurabhbikram)
- Add the unique invocation id to temporary "new data" used in `delete+insert` incremental materializations to allow parallel transformations on the same
table.  In general parallel transformations are risky, so this approach should only be used when transformations are explicitly limited
to non-overlapping data ranges.  Closes https://github.com/ClickHouse/dbt-clickhouse/issues/150

### Release [1.4.0], 2023-02-06
#### Improvements
- Support dbt [1.4.1] https://github.com/ClickHouse/dbt-clickhouse/issues/135
  - Adds support for Python 3.11
  - Adds additional dbt 1.4.0 tests
  - Adds support for incremental_predicates.  This only applies to `delete+insert` incremental strategy.  Note that incremental
predicates that depend on "non-deterministic" data (such as a subquery using a table that is accepting inserts) could lead to
unexpected results for ReplicatedMergeTree tables depending on the timing of the incremental materialization.  
  - Replaces deprecated Exception classes
- Setting the `use_lw_deletes` profile value to True will now attempt to enable the `allow_experimental_lightweight_delete`
setting for the dbt session (if user has such permissions on the ClickHouse server).  See https://github.com/ClickHouse/dbt-clickhouse/issues/133

#### Bug Fix
- Composite unique keys specified as a list would not work with incremental materializations.  This has been fixed.


### Release [1.3.3], 2023-01-18
#### Documentation Update
- The documentation has been updated to reflect that dbt-clickhouse does support ephemeral models, and ephemeral model tests do pass.
However, due to a [ClickHouse limitation](https://github.com/ClickHouse/ClickHouse/issues/30323), CTEs will not work directly
with INSERT statements so table models will fail if they include ephemeral models in the SELECT.  View models and other SQL 
statements using ephemeral models should work correctly.

#### Bug Fix
- Client connections would incorrectly reuse a previous session_id when initializing, causing session locked errors.  This has been fixed.  This
closes https://github.com/ClickHouse/dbt-clickhouse/issues/127.  Multiple threads should now work correctly in dbt projects,
and dbt-clickhouse automated tests now use `threads=4` (soon to be the dbt default).

### Release [1.3.2], 2022-12-23
#### Improvements
- Added *experimental* support for the `delete+insert` incremental strategy.  In most cases this strategy should be significantly faster
than the existing "legacy" custom strategy (which is still the default).  To enable `delete+insert` incremental materialization, the flag `allow_experimental_lightweight_delete`
must be enabled on your ClickHouse server.  This flag is NOT currently considered production ready, so use at your own risk.  To use this strategy
as the "default" incremental strategy, the new configuration value `use_lw_deletes` must be set to True.  Otherwise, to use it for a particular model,
set the model's `incremental_strategy` to `delete+insert`.  Important caveats about this strategy:
  - This strategy directly uses lightweight deletes on the target model/table.  It does not create a temporary or intermediate table.  Accordingly,
if there is an issue during that transformation, the materialization can not be rolled back and the model may contain inconsistent data.
  - If the incremental materialization includes schema changes, or lightweight deletes are not available, dbt-clickhouse will fall back to the much
slower "legacy" strategy.
- Allow `append` as an incremental strategy.  This is the same as using the custom model configuration value `inserts_only`.
- New s3source macro.  This simplifies the process of using the ClickHouse s3 table function in queries.  See the
[tests](https://github.com/ClickHouse/dbt-clickhouse/blob/main/tests/integration/adapter/test_s3.py) for example usage.

#### Bug Fixes
- The ON CLUSTER clause has been added to additional DDL statements including incremental models processing. 
Closes https://github.com/ClickHouse/dbt-clickhouse/issues/117 and should close https://github.com/ClickHouse/dbt-clickhouse/issues/95
for Replicated tables that use the `{uuid}` macro in the path to avoid name conflicts.  Thanks to [Saurabh Bikram](https://github.com/saurabhbikram)
- The `apply` and `revoke` grants macros now correctly work with roles as well as user.  Again thanks to [Saurabh Bikram](https://github.com/saurabhbikram)
- A compound unique_key (such as `key1, key2`) now works correctly with incremental models

### Release [1.3.1], 2022-11-17
#### Improvements
- Improve error message when atomic "EXCHANGE TABLES" operation is not supported

### Release [1.3.0], 2022-10-30
#### Improvements
- Support dbt [1.3.0]  https://github.com/ClickHouse/dbt-clickhouse/issues/105
  - Adds additional dbt 1.3.0 core tests
  - Adds array utility macros ported from dbt-utils
  - Does NOT add support for Python models (not implemented)
  - Does NOT utilize default/standard incremental materialization macros (standard strategies do not work in ClickHouse)

#### Bug Fix
- Require exact match for relations.  ClickHouse databases and tables are all case-sensitive, so all searches are now case-sensitive.  Closes https://github.com/ClickHouse/dbt-clickhouse/issues/100 and https://github.com/ClickHouse/dbt-clickhouse/issues/110

</br></br>

### Release [1.2.1], 2022-09-19
#### Improvements
- Support dbt 1.2.1  https://github.com/ClickHouse/dbt-clickhouse/issues/79
    - Grants support on models
    - Support the cross database dbt-core macros (migrated from dbt-utils)
    - Add docs tests
    - Validate Python 3.10 support
    - Implement retry logic for small set of "retryable" connection errors
- Don't close connection on release to improvement model performance
- Allow specifying database engine for schema creation
- Extend support for ClickHouse Cloud and Replicated databases

#### Bug Fix
- Validate that atomic EXCHANGE TABLES is supported on the target ClickHouse server https://github.com/ClickHouse/dbt-clickhouse/issues/94

</br></br>
### Release [1.1.8], 2022-09-02
#### Improvement
- Support use of atomic EXCHANGE TABLES when supported by ClickHouse

</br></br>
### Release [1.1.7], 2022-07-11
#### Improvement
- Support ClickHouse Arrays with Nullable value types

#### Bug Fix
- Fix brackets regex in columns)
</br></br>
### Release [1.1.6], 2022-07-04
#### Improvement
- Add support for CREATE AS SELECT in Replicated Databases

</br></br>
### Release [1.1.5], 2022-06-27
#### Bug Fixes
- Ensure database exists before connection
- Safely handle dropping working database

</br></br>
### Release [1.1.4], 2022-06-27
#### Improvement
- Allow selection of either native protocol driver (clickhouse-driver) or http driver (clickhouse-connect)

</br></br>
### Release [1.1.2], 2022-06-22
#### Rename from v 1.1.0.2

</br></br>
### Release [1.1.0.2], 2022-06-17

#### Improvements
- Support for inserting SETTINGS section to CREATE AS and INSERT INTO queries through model configuration
- Support adding custom session settings to the connection through profile credentials.
- Allow using custom databases/schemas for connection
- Support Boolean fields in CSV seeds

#### Bug Fix
- Fixed prehooks and empty fields in CSV seeds

</br></br>
### Release [1.1.0], 2022-06-02

### Release [1.1.0.1], 2022-06-09

#### Bug Fix
- Fixed prehooks and empty fields in CSV seeds

</br></br>
### Release [1.1.0], 2022-06-02

#### Improvements
- 1.1.0 dbt support
- Snapshots timestamps
- Replace temporary in-memory tables with MergeTree table - removing memory limitations over Incremental model and snapshots creation
- Moved to use clickhouse-connect driver - an officially supported HTTP driver by ClickHouse Inc.
</br></br>
### Release [1.0.4], 2022-04-02

### Improvements
- 1.0.4 dbt support
- New logger
</br></br>
### Release [1.0.1] - 2022-02-09

#### Improvement
- Support 1.0.1 dbt

#### Bug Fixes
- Skip the order columns if the engine is Distributed https://github.com/ClickHouse/dbt-clickhouse/issues/14
- Fix missing optional "as" https://github.com/ClickHouse/dbt-clickhouse/issues/32
- Fix cluster name quoted https://github.com/ClickHouse/dbt-clickhouse/issues/31
</br></br>
### Release [1.0.0], 2022-01-02

#### Add
- dbt 1.0.0 support
</br></br>
### Release [0.21.1], 2022-01-01

#### Improvements
- dbt 0.21.1 support
- Extended settings for clickhouse-driver https://github.com/ClickHouse/dbt-clickhouse/issues/27

### Bug Fixes
- Fix types in CSV seed https://github.com/ClickHouse/dbt-clickhouse/issues/24

### Release [0.21.0], 2021-11-18

#### Improvement
- Support 0.21.0 dbt

#### Bug Fixes
- Fix FixString column https://github.com/ClickHouse/dbt-clickhouse/issues/20
- Default behavior for a quoted https://github.com/ClickHouse/dbt-clickhouse/issues/21
- Fix string expand https://github.com/ClickHouse/dbt-clickhouse/issues/22
</br></br>
### Release [0.20.2], 2021-10-16

#### Improvement
- dbt 0.20.1 support

#### Improvement
- Rewrite logic incremental materializations https://github.com/ClickHouse/dbt-clickhouse/issues/12

#### Bug Fixes
- Fix dbt tests with https://github.com/ClickHouse/dbt-clickhouse/pull/18 (thx @artamoshin)
- Fix relationships test https://github.com/ClickHouse/dbt-clickhouse/issues/19
</br></br>
### Release [0.20.1], 2021-08-15

#### Improvement
- dbt 0.20.1 support
</br></br>
### Release [0.20.0], 2021-08-14

#### Improvement
- dbt 0.20.0 support
</br></br>
### Release [0.19.1.1], 2021-08-13

#### Improvement
- Add verify and secure to connection configuration

#### Bug Fix
- Fix the delete expression https://github.com/ClickHouse/dbt-clickhouse/issues/12
</br></br>
### Release [0.19.1], 2021-05-07

#### Improvement
- Add support the `ON CLUSTER` clause for main cases

#### Usage Change
- Engine now require brackets `()`

#### Bug Fix
- Fix a missing sample profile
</br></br>
### Release [0.19.0.2], 2021-04-03

#### Bug Fix
- Fix name of partition
</br></br>
### Release [0.19.0.1], 2021-03-30

#### Bug Fix
- Fix version of clickhouse-driver in setup.py
</br></br>
### Release [0.19.0], 2021-02-14
#### Initial Release

[1.3.3]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.3.2..v1.3.3
[1.3.2]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.3.1..v1.3.2
[1.3.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.3.0..v1.3.1
[1.3.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.2.1..v1.3.0
[1.2.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.2.0..v1.2.1
[1.2.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.8..v1.2.0
[1.1.8]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.7..v1.1.8
[1.1.7]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.6..v1.1.7
[1.1.6]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.5..v1.1.6
[1.1.5]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.4..v1.1.5
[1.1.4]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.2..v1.1.4
[1.1.2]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.0.1..v1.1.2
[1.1.0.2]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.1.0.1..v1.1.0.2
[1.1.0.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.0.1...v1.1.0.1
[1.1.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.0.4...v1.1.0
[1.0.4]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.0.1...v1.0.4
[1.0.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.21.1...v1.0.0
[0.21.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.21.0...v0.21.1
[0.21.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.20.2...v0.21.0
[0.20.2]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.20.1...v0.20.2
[0.20.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.20.0...v0.20.1
[0.20.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.19.1.1...v0.20.0
[0.19.1.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.19.1...v0.19.1.1
[0.19.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.19.0.2...v0.19.1
[0.19.0.2]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.19.0.1...v0.19.0.2
[0.19.0.1]: https://github.com/ClickHouse/dbt-clickhouse/compare/v0.19.0...v0.19.0.1
[0.19.0]: https://github.com/ClickHouse/dbt-clickhouse/compare/eb3020a...v0.19.0