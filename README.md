# Snowflake-query

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

This github action runs a list of SQL queries against a Snowflake DB, with the configuration described below.

# Note
This has been forked from the anecdotesai/snowflake-query repo because it seems that is no longer managed.
This was designed to fix a bug that arose from oscrypto delayed updates. I plan to update and maintain this in moving forward.

## Inputs

- `snowflake_account` - Account name for Snowflake DB. Your account name is the full/entire string to the left of snowflakecomputing.com.
- `snowflake_warehouse` - Set the warehouse context for the queries.
- `snowflake_username`, `snowflake_password` - Credentials for your DB.
  - It's recommended to use [Github's Secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) for those arguments.
- `snowflake_role` (optional) - Set a role for the user.
- `queries` - SQL queries to execute **asynchronously and independently**.
  - May contain multiple queries, seperated by ';'
  - Don't use ';' with the last query.
  - If you need to contain a single-quote in one or more queries, escape it with another single-quote.

## Output

`queries_results` - Json string contains the results from all queires executed. For example, for the query `SELECT CURRENT_VERSION()`, we'll get - `{'019ca1a1-0000-43ef-0000-15ed05cb1c4e': ['('5.20.1',)']}`.

It may be accessed in following steps by `${{steps.run_queries.outputs.queries_output}}`. See [this](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#tojson) guide for more github's action expressions examples.

## Usage

### Run multiple queries in one job

```yaml
steps:
  - name: Run queries
    uses: Kirksten3/snowflake-query@v1.2.1
    id: run_queries
    with:
        snowflake_account: ${{ secrets.SNOWFLAKE_ACCOUNT }}
        snowflake_warehouse: ${{ secrets.SNOWFLAKE_WAREHOUSE }}
        snowflake_username: ${{ secrets.SNOWFLAKE_USER }}
        snowflake_password: ${{ secrets.SNOWFLAKE_PASSWORD }}
        queries: 'call system$wait(5);
                  select CURRENT_VERSION();
                  select * from "<TABLE_NAME>" where <column_name>=''<value>'''
        # single quote is escaped with another single quote

    - name: Version Query Validation
        run: |
          ${{contains(steps.run_queries.outputs.queries_output, '5.20.1')}}
```

### Run multiple queries as a job-matrix

```yaml
strategy:
  matrix: 
    table: ['TABLE1', 'TABLE2', 'TABLE3',
            'TABLE4', 'TABLE5']
steps:
  - name: Run Delete Queries
    uses: Kirksten3/snowflake-query@v1.2.1
    id: run_delete_queries
    with:
      snowflake_account: ${{ secrets.SNOWFLAKE_ACCOUNT }}
      snowflake_warehouse: ${{ secrets.SNOWFLAKE_WAREHOUSE }}
      snowflake_username: ${{ secrets.SNOWFLAKE_USER }}
      snowflake_password: ${{ secrets.SNOWFLAKE_PASSWORD }}
      queries: 'DELETE FROM  "${{matrix.table}}"'
```

