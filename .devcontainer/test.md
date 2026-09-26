# Running tests

Run all of these from the VS Code terminal inside the dev container.
Running the whole suite one file after another (`bin/test`) takes about 15 minutes.
Most of that time is `spec/tapioca/cli/`, where every test builds a mock
project and runs `bundle install`. While you work, run only what you need.

## Run a subset

```bash
# One file
bin/test spec/tapioca/gem/pipeline_spec.rb

# One test, by the line number of its `it "..."` block
bin/test spec/tapioca/gem/pipeline_spec.rb:123

# Tests whose name matches a regex
bin/test spec/tapioca/gem/pipeline_spec.rb -n /prepend/

# One or more folders
bin/test spec/tapioca/gem spec/tapioca/runtime
```

`bin/test` uses the Rails test runner, so it takes file paths, `file:line`,
and `-n /regex/` filters. `bin/test --help` lists every option.

## Run the whole suite in parallel

Each spec class writes its mock projects to its own folder under
`/tmp/tapioca/tests/<SpecName>` (see `spec/spec_with_project.rb`), so
separate processes don't get in each other's way. The command below splits
the spec files across one process per CPU core:

```bash
find spec -name '*_spec.rb' | xargs -P "$(nproc)" -n 5 bin/test > /tmp/test.log 2>&1; \
  grep -E "failures|errors|Failure:|Error:" /tmp/test.log
```

- Output from the processes is mixed together, so read the summary lines
  (`N tests, N assertions, N failures, N errors`) and any `Failure:` blocks.
- Change `-n 5` to set how many files each process runs. Smaller numbers
  spread the work more evenly.
- Minitest's built-in threaded `parallelize_me!` isn't safe for this suite,
  because tests change `ENV` and fork. That's why this runs separate
  processes instead.

## Before pushing

Run the full normal suite once, the same way CI runs it
(`.github/workflows/ci.yml`):

```bash
bin/test
```
