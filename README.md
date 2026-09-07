# elephentity-codegen

The code generator for [Elephentity](https://github.com/hsimah-services/elephentity).

Elephentity compiles human-readable specs into an IR. This program takes it from there:
it resolves the language builders a project has configured, runs each one, and signs and
writes what they return.

```
eleph generate
  │ compiles specs → IR
  ▼
eleph-codegen generate
  ├→ eleph-gen-php   ─ files
  └→ eleph-gen-ts    ─ files
  │ sign, write, diff
  ▼
generated/
```

**It knows no language.** The IR passes through encoded and is never decoded; a builder
turns it into PHP or TypeScript or anything else, returns paths and bodies, and this
signs and writes them. That keeps one signer as the authority on what "locked" means,
rather than every builder reimplementing it and one of them getting it subtly wrong — and
it means `--check` works for every target without a builder knowing `--check` exists.

[docs/PROTOCOL.md](docs/PROTOCOL.md) is the contract, on both sides.

## Commands

```bash
eleph-codegen generate --project .            # request on stdin; writes the tree
eleph-codegen generate --project . --check    # writes nothing; fails on any difference
eleph-codegen targets  --project .            # the configured targets, as JSON
eleph-codegen doctor   --project .            # are the builders installed and runnable?
```

`generate` is run by `eleph generate`, which compiles the specs and pipes them in. The
others are for humans and for tools that need to know what a project generates.

## Working on it

There is no local PHP; everything runs in a container:

```bash
./tools/php composer ci          # style, static analysis, tests
./tools/php composer style:fix
./tools/php vendor/bin/phpunit --filter SomeTest
```

PHPStan runs at **level max** with no baseline exclusions.

## Written in PHP, for now

The rewrite target is Rust. The protocol is what makes that a rewrite rather than a
rebuild: `tests/Command/GenerateCommandTest.php` drives the real binary with JSON on
stdin and asserts bytes on disk — including a signed header's exact digest — so a
reimplementation in another language has a suite it must satisfy unchanged.
