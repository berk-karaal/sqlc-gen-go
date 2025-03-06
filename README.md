# sqlc-gen-go separate models file

This fork of sqlc-gen-go introduces exporting models file to a different package.

There is a related issue on the sqlc repository:
[github.com/sqlc-dev/sqlc/issues/835](https://github.com/sqlc-dev/sqlc/issues/835)

## Added Options

- `output_models_package`:
  - Package name of the models file. Used when models file is in a different package. Defaults to value of `package` option.
- `models_package_import_path`:
  - Import path of the models package when models file is in a different package. Optional.
- `output_query_files_directory`:
  - Directory where the generated query files will be placed. Defaults to the value of `out` option.

## How to use for separate models file

Lets say you want to export models to `internal/business/entities/database.go` file and keep other
generated files in `internal/sqlcrepo/` directory. You can use the following configuration:

```yaml
version: "2"
plugins:
  - name: golang
    wasm:
      url: https://github.com/berk-karaal/sqlc-gen-go/releases/download/v1.5.1/berk-karaal-sqlc-gen-go_1.5.1.wasm
      sha256: 95bc2009c94bdac0f8c5af8207bd1cf43723f8e33fb17e06fce1d96b83da242e
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
          models_package_import_path: "github.com/example/module-path/internal/business/entities"  # Import path for the separated models file package
```

For working examples, you can check the
[berk-karaal/sqlc-gen-go-demo](https://github.com/berk-karaal/sqlc-gen-go-demo) repository.

This feature implementaion is backward compatible, so you can still use your old ["sqlc-gen-go"
configuration](https://github.com/sqlc-dev/sqlc-gen-go?tab=readme-ov-file#migrating-from-sqlcs-built-in-go-codegen)
without separating models file. Options defined in the original sqlc-gen-go plugin should still be
working (I couldn't test them all, to be honest). You can test it by your use case and report any
issues.

### Why we have to explicitly define query file directory, db file, querier file, batch file, copyfrom file?

The problem is that, sqlc [does not
allow](https://github.com/sqlc-dev/sqlc/blob/6b2ed2024ccd66eea7eb7d97cb36338a7fb46f3d/internal/cmd/generate.go#L223-L226)
writing to a file outside of the `out` directory. If it was allowed, we could simply set `out` to
`internal/sqlcrepo` and `output_models_file_name` to `../business/entities/database.go` and we
wouldn't need to specify `output_query_files_directory`, `output_db_file_name`,
`output_querier_file_name`, `output_batch_file_name`, `output_copyfrom_file_name` options.

Another solution could be creating another option for the base directory for the files except the
models file. But I think it would increase the complexity of the configuration file.
