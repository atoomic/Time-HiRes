# CLAUDE.md — Time::HiRes

## Project Overview

Time::HiRes is a dual-life Perl core XS module providing high-resolution time functions:
`usleep`, `ualarm`, `gettimeofday`, `time`, `sleep`, `alarm`, `clock_gettime`,
`clock_getres`, `clock`, `clock_nanosleep`, `stat`, `lstat`, `utime`.

- **Repository**: atoomic/Time-HiRes (fork — upstream is Perl core)
- **Version**: 1.9764
- **Language**: C (HiRes.xs) + Perl (HiRes.pm) + ExtUtils::MakeMaker (Makefile.PL)
- **Minimum Perl**: 5.006

## Key Files

| File | Description |
|------|-------------|
| `HiRes.xs` | XS/C implementation (~49K) — all time functions |
| `HiRes.pm` | Perl module — exports, AUTOLOAD, POD (~25K) |
| `Makefile.PL` | Build config — probe system capabilities (~30K) |
| `ppport.h` | Devel::PPPort portability header (v3.36, ~200K) |
| `typemap` | XS type mappings |
| `hints/` | OS-specific build hints (AIX, Solaris, Linux, etc.) |
| `fallback/` | Fallback constant definitions |
| `t/Watchdog.pm` | Test helper — alarm-based watchdog for hung tests |

## Build & Test

```bash
# Build
perl -I$(pwd) Makefile.PL && make

# Run all tests (~90s — many sleep-based tests)
make test

# Run a single test
make test TEST_FILES=t/time.t

# Verbose
make test TEST_VERBOSE=1
```

The `-I$(pwd)` flag is needed for Makefile.PL to find the local module during build.

## Test Suite

12 test files in `t/`, each covering a specific function:

| Test | Covers |
|------|--------|
| `t/alarm.t` | `alarm()` |
| `t/clock.t` | `clock_gettime()`, `clock_getres()`, `clock()` |
| `t/gettimeofday.t` | `gettimeofday()` |
| `t/itimer.t` | `getitimer()`, `setitimer()` |
| `t/nanosleep.t` | `nanosleep()` |
| `t/sleep.t` | `sleep()` |
| `t/stat.t` | `stat()`, `lstat()` |
| `t/time.t` | `time()` |
| `t/tv_interval.t` | `tv_interval()` |
| `t/ualarm.t` | `ualarm()` |
| `t/usleep.t` | `usleep()` |
| `t/utime.t` | `utime()` |

Tests are time-sensitive. Heavy system load can cause spurious failures.

## CI

Single workflow: `.github/workflows/testsuite.yml`

- **Linux**: ubuntu-latest system Perl + full Perl version matrix via `perl-actions/perl-versions` (5.8+, including devel) using `perldocker/perl-tester` containers
- **macOS**: `shogo82148/actions-setup-perl` (latest)
- **Windows**: `shogo82148/actions-setup-perl` with `distribution: strawberry` — uses `gmake test` (not `make test`)

## Coding Conventions

- C code in HiRes.xs follows C89 style (no `//` comments, declarations at block start)
- Uses `PERL_NO_GET_CONTEXT` for threaded Perl efficiency
- `ppport.h` provides backward compatibility macros for older Perls
- Extensive `#ifdef` gating for platform-specific features
- Makefile.PL probes system capabilities via `try_compile_and_link()`

## Repository Notes

- This is a fork of the Perl core module — changes may need to be coordinated with perl5-porters
- Branch `update-to-1.9765` has a pending blead sync (includes ppport.h v3.36 → v3.59 update)
- Generated files (const-c.inc, const-xs.inc, HiRes.c, HiRes.o, blib/) are in .gitignore
