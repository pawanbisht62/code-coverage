# Generate code coverage report and validate in CI check 

The Rust compiler includes two code coverage implementations:
* gcov-based coverage
* Source-based code coverage

This repository follows [Source-based code coverage](https://doc.rust-lang.org/rustc/instrument-coverage.html).
```
A source-based code coverage implementation, enabled with -C instrument-coverage, which uses LLVM's native, efficient 
coverage instrumentation to generate very precise coverage data.
```

On top of that, it uses Mozilla's [grcov](https://github.com/mozilla/grcov) tool for generating code coverage reports.
```
grcov collects and aggregates code coverage information for multiple source files. grcov processes .profraw and .gcda files which can be generated from llvm/clang or gcc.
```

### Setup:
1. Install the llvm-tools or llvm-tools-preview component:
    ```
    rustup component add llvm-tools-preview
    ```
2. Ensure that the following environment variable is set up:
    ```
    export RUSTFLAGS="-Cinstrument-coverage"
    ```
3. Build your code:
   ```
   cargo build
   ```
4. [Optional] Set a specific file name or path for the generated .profraw files by using the environment variable LLVM_PROFILE_FILE, else it will create new file named like default_...
   ```
   export LLVM_PROFILE_FILE="your_name-%p-%m.profraw"
   ```
5. Run your tests:
   ```
   cargo test
   ```

After running this, new files named like default_* should be created in the current working directory. These files will be used to generate the code coverage report.

### Generate code coverage report using grcov
```
grcov . --binary-path ./target/debug/ -s . -t html --branch --ignore-not-existing --keep-only "pallets/template/src/lib.rs" -o target/coverage/html
```
After running this, a html report named `index.html` is generated under `target/coverage/html`.

In the above command we have used `--keep-only` option to generate report for specific file as we care about this file's code coverage only. It can be customized as per requirements.

### Validate code coverage in CI check:
`grcov` also provides code coverage data which can be found in `target/coverage/html/coverage.json`, now this file can be used in CI check to validate the code-coverage, like this:

```
- name: check coverage
  run: |
    json_data=`cat target/coverage/html/coverage.json`
    coverage=`echo $(jq -r '.message' <<< "$json_data")`
    coverage="${coverage%%.*}"
    if [ $coverage -lt 80  ];
    then
        echo "Low coverage, current ${coverage} %, expected: >= 80 %"
        exit 1
    fi
```
And a complete yml file can be found [here](.github/workflows/test.yml).
