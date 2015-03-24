# fdr-rot

Designed in a circumstance demanding expedience, `fdr-rot` identifies
and rotates bloated indexes in a concurrent manner.  It happens to
work reasonably well.

By default, it will generate SQL statements that are suitable for
execution, e.g. via `psql`, but by setting an environment variable it
can also perform the rotations in-band.

## Examples

Printing out SQL statements for a database:

```sh
$ FDR_ROT_DATABASE_URL=postgres:///postgres fdr-rot
[...generated SQL statements...]
-- Processing public.resources::resources_search_index
-- Bloat Statistics: bloat=78.9, bytes=11218026496
-- Definition: CREATE INDEX resources_search_index ON resources USING gin (to_tsvector('simple'::regconfig, id), to_tsvector('simple'::regconfig, hstore_as_text(attrs_unparsed)))
CREATE INDEX CONCURRENTLY "_fdr_rot_resources_search_index" ON resources USING gin (to_tsvector('simple'::regconfig, id), to_tsvector('simple'::regconfig, hstore_as_text(attrs_unparsed)));
ANALYZE "resources";
BEGIN;
-- Index confirmed as valid ('indisvalid' attribute)...
DROP INDEX "resources_search_index";
ALTER INDEX "_fdr_rot_resources_search_index" RENAME TO "resources_search_index";
COMMIT;
[...more generated SQL...]
```


Perform the rotations also:

```sh
$ FDR_ROT_REALLY_DO_IT=true FDR_ROT_DATABASE_URL=postgres:///postgres fdr-rot
[...prints out the same output, synchronously as it is run...]
```
