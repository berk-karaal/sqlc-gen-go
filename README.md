# sqlc-gen-go separate models file

This fork of sqlc-gen-go introduces exporting models file to a different package.

There is a related issue on the sqlc repository:
[github.com/sqlc-dev/sqlc/issues/835](https://github.com/sqlc-dev/sqlc/issues/835)

## How to use for separate models file

Lets say you want to export models to `internal/business/entities/database.go` file and keep other generated
files in `internal/sqlcrepo/` directory. You can use the following configuration:

```yaml
version: "2"
plugins:
  - name: golang
    wasm:
      url: https://github.com/berk-karaal/sqlc-gen-go/releases/download/v1.5.1/berk-karaal-sqlc-gen-go_1.5.1.wasm
      sha256: 415de96b2cf3bbda5a5ebc8fbf2a782f6d9916a35554b3e61183c90a6c504165
sql:
  - engine: "postgresql"
    queries: "query.sql"
    schema: "schema.sql"
    codegen:
      - plugin: golang
        out: "internal/"  # This is the base directory for the generated files
        options:
          sql_package: "pgx/v5"
          package: "sqlcrepo"  # Default package name for the generated files
          output_query_files_directory: "sqlcrepo/"  # Where to put the generated query files
          output_db_file_name: "sqlcrepo/db.go"  # Where to put the generated db file
          output_querier_file_name: "sqlcrepo/querier.go"  # Where to put the generated querier (interface) file
          output_batch_file_name: "sqlcrepo/batch.go"  # Where to put the generated batch file
          output_copyfrom_file_name: "sqlcrepo/copyfrom.go"  # Where to put the generated copyfrom file
          output_models_file_name: "business/entities/database.go"  # Where to put the generated models file. You should use this to separate models file
          output_models_package: "entities"  # Package name that should be used in `output_models_file_name` file
          output_models_package_import_path: "github.com/example/module-path/internal/business/entities"  # Import path for the separated models file package
```

For working examples, you can check the [berk-karaal/sqlc-gen-go-demo](https://github.com/berk-karaal/sqlc-gen-go-demo)
repository.

This feature implementaion is backward compatible, so you can still use your old ["sqlc-gen-go"
configuration](https://github.com/berk-karaal/sqlc-gen-go?tab=readme-ov-file#migrating-from-sqlcs-built-in-go-codegen)
without separating models file. Options defined in the original sqlc-gen-go plugin should still be working (I
couldn't test them all, to be honest). You can test it by your use case and report any issues.

### Why we have to explicitly define file name for batch file, db file, querier file and copyfrom file?

This can be solved by a smarter options design but for now I didn't want to complicate the logic as this is
just a proof of concept. The problem is that, sqlc [does not
allow](https://github.com/sqlc-dev/sqlc/blob/6b2ed2024ccd66eea7eb7d97cb36338a7fb46f3d/internal/cmd/generate.go#L223-L226)
writing to a file outside of the `out` directory. If it was allowed, we could simply set `out` to
`internal/sqlcrepo` and `output_models_file_name` to `../business/entities/database.go` or
`internal/business/entities/database.go` and we wouldn't need to specify `output_query_files_directory`,
`output_batch_file_name`, `output_db_file_name`, `output_querier_file_name`, `output_copyfrom_file_name`
options since they will use the `out` directory by default and models file will have no problem with writing
outside of the `out` directory.

## What are the changes in this fork? (broadly)

- Added `Package` field to `Struct` type to specify the correct type of the Struct since they can be in
  different package now. (`internal/struct.go`)
- `Type()` method of `QueryValue` will return `{Package}.{Name}` for `Struct` types if the `Package` field of
  the `Struct` is not empty. (`internal/query.go`)
- Added `{Package}.` prefix to enum types if configuration specifies separate models package. (only
  `internal/postgresql_type.go` since only postgresql implementation supports typed enums values)
- Deciding whether the genereted file needs to import separated models file package or not.
  (`internal/imports.go`)

Compare complete code changes:
[github.com/sqlc-dev/sqlc-gen-go/compare/main...berk-karaal:sqlc-gen-go:allow-exporting-models-to-separate-package?diff=split&w=](https://github.com/sqlc-dev/sqlc-gen-go/compare/main...berk-karaal:sqlc-gen-go:allow-exporting-models-to-separate-package?diff=split&w=)
