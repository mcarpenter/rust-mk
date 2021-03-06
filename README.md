rust-mk
=======

This make script intend to help Rusties to compile Rust programs with a Makefile.

Usage
-----

### Get source ###

```sh
git submodule add git://github.com/KokaKiwi/rust-mk.git
```

Or, if you don't work in a local Git repository:

```sh
git clone git://github.com/KokaKiwi/rust-mk.git
```

### Basic usage ###

This Makefile will compile a crate located in src/mycrate/{lib.rs,main.rs}

```make
RUSTCRATES          =   mycrate

include             rust-mk/rust.mk
```

### Advanced usage ###

```make
RUSTCRATES          =   mycrate mydep

mycrate_TYPE        =   bin        # Automatically detected if not specified.
mycrate_CRATE_DEPS  +=  mydep      # mydep will be build before mycrate.
mycrate_BUILD_DEPS  +=  libtest.a  # Raw dependency of libtest.a for mycrate.
mycrate_RUSTCFLAGS  +=  -g         # Add some custom flags as you want.

include             rust-mk/rust.mk
```

### Disable auto crate rules ###

```make
RUSTAUTORULES       =   0

include             rust-mk/rust.mk

$(eval $(call RUST_CRATES_RULES))
```

### Import external crates (using rust-mk) ###

```make
RUSTCRATES          =   extcrate

extcrate_ROOTDIR    =   deps/extcrate

include             rust-mk/rust.mk
```

### Import external crates (not using rust-mk) ###

```make
RUSTCRATES          =   extcrate

extcrate_ROOT       =   deps/extcrate/src/lib.rs
extcrate_TYPE       =   lib # Only if the crate root filename is not lib.rs or main.rs

include             rust-mk/rust.mk
```

### Add a LLVM-passe rules on targets ###

```make
RUSTCRATES          =   mycrate

# LLVM
define RUST_CRATE_RULES_ADD
$(1)_LLVM_NAME      = $(1).ll

_llvm_$(1):         $$($(1)_LLVM_NAME)
.PHONY llvm:        _llvm_$(1)

_clean_llvm_$(1):
    rm -f $$($(1)_LLVM_NAME)
.PHONY clean:       _clean_llvm_$(1)

$(1).ll:            $$($(1)_NAME) $$($(1)_ROOT)
    $$(RUSTC) $$(RUSTCFLAGS) $$($(1)_RUSTCFLAGS_BUILD) $$($(1)_RUSTCFLAGS) --emit ir -o $(1).ll $$($(1)_ROOT)
endef

include rust-mk/rust.mk

llvm:
.PHONY:             llvm
```

Rules
-----

```sh
$ make help
 Common rules:
  make all                 - Build all crates (alias of 'build' target).
  make build               - Build all crates.
  make clean               - Clean crates targets.
  make fclean              - Clean crates targets and build directories.
  make rebuild             - Rebuild all crates.
  make test                - Build and run tests.
  make bench               - Build and run benchs.
  make doc                 - Generate crates documentation.
  make install             - Install crates targets in ~/.rust
  make uninstall           - Uninstall crates targets.
  make crates              - Print available crates.
  make help                - Print this help.

 Crates rules:
  make build_<crate>       - Build <crate>.
  make clean_<crate>       - Clean <crate> targets.
  make rebuild_<crate>     - Rebuild <crate>.
  make test_<crate>        - Build and run <crate> tests.
  make bench_<crate>       - Build and run <crate> benchs.
  make doc_<crate>         - Generate <crate> documentation.
  make install_<crate>     - Install <crate> targets in ~/.rust
  make uninstall_<crate>   - Uninstall <crate> targets.

 Available crates:         [...]
```

Special variables
-----------------

These special variables can be set, either by setting them in env or by passing them to make:

```sh
make <varname>=<varvalue>
```

### `RUSTC` ###

Default: `rustc`

Path to `rustc` executable.

### `RUSTDOC` ###

Default: `rustdoc`

Path to `rustdoc` executable.

### `RUSTCFLAGS` ###

Default: depending on others vars

Flags used to compile crates.

### `RUSTDOCFLAGS` ###

Default: nothing

Flags used to generate docs.

### `RUSTDEBUG` ###

Default: `0`

If set to `1`, activate debug flags (`-g`) else optimization flags will be activated (`--opt-level=3`)

### `RUSTBUILDDIR` ###

Default: `.rust`

This directory will be used to store build files (like test binaries).

### `RUSTSRCDIR` ###

Default: `src`

This directory is where crates must be find.

### `RUSTBINDIR` ###

Default: `.`

This directory is where runnable crates' binaries will be stored.

### `RUSTLIBDIR` ###

Default: `lib`

This directory is where library crates will be stored.

### `RUSTDOCDIR` ###

Default: `doc`

This directory is where crate docs will be stored.

### `RUSTINSTALLDIR` ###

Default: `~/.rust`

This directory is where all crates will be installed.

### `RUST_CRATE_RULES_ADD` ###

Default: Not defined

You can define this variable to add some rules for all crates.

Crate variables
---------------

### `<crate>_ROOTDIR` ###

Default: .

Directory where rust-mk will search for source directory containing the crate (usually `src/<crate>`).

Useful when compiling an external crate.

### `<crate>_ROOT` ###

Default: Determined with existing file (`main.rs` or `lib.rs`) in crate dir.

Specify the crate root file (entry file when compiling the crate).

### `<crate>_ROOT_TEST` ###

Default: Crate root file

Specify the crate root file used for tests.

### `<crate>_TEST_DOC` ###

Default: `1`

Specify if you want to test the Rust code in your doc comments.

In fact, it add `--test` to `rustdoc`

### `<crate>_DONT_TEST` ###

Default: `0`

- `0` if you want to test `<crate>`
- `1` if you don't want to test `<crate>`

### `<crate>_DONT_BENCH` ###

Default: Value of `<crate>_DONT_TEST` variable

Same as `<crate>_DONT_TEST` variable, but for bench.

### `<crate>_DONT_DOC` ###

Default: `0`

- `0` if you want to generate doc for `<crate>`
- `1` if you don't want to generate doc for `<crate>`

### `<crate>_TYPE` ###

Default: Determined with crate root (`main.rs` or `lib.rs`).

Values: `bin` or `lib`

Specify the crate type of `<crate>` (indicate if it cannot be automatically determined or if you want to force the value).

### `<crate>_CRATE_DEPS` ###

Default: nothing

Indicate other crate names on which `<crate>` depends.

These crates will be built before `<crate>`

### `<crate>_BUILD_DEPS` ###

Default: nothing

Indicate raw make rule dependencies on which `<crate>` depends.

These rules will be built before `<crate>`

### `<crate>_RUSTCFLAGS` ###

Default: nothing

Add crate-specific flags at build-time.

### `<crate>_DONT_ADD_RULES` ###

Default: `0`

- `0` if you want to extends this crate with `RUST_CRATE_RULES_ADD` variable.
- `0` if you don't want to extends this crate with `RUST_CRATE_RULES_ADD` variable.

License
-------

`rust-mk` is licensed under MIT license.
