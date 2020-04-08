# sysinfo [![][img_travis-ci]][travis-ci] [![Build status](https://ci.appveyor.com/api/projects/status/nhep876b3legunwd/branch/master?svg=true)](https://ci.appveyor.com/project/GuillaumeGomez/sysinfo/branch/master) [![][img_crates]][crates] [![][img_doc]][doc]

[img_travis-ci]: https://api.travis-ci.org/GuillaumeGomez/sysinfo.png?branch=master
[img_crates]: https://img.shields.io/crates/v/sysinfo.svg
[img_doc]: https://img.shields.io/badge/rust-documentation-blue.svg

[travis-ci]: https://travis-ci.org/GuillaumeGomez/sysinfo
[crates]: https://crates.io/crates/sysinfo
[doc]: https://docs.rs/sysinfo/

A system handler to interact with processes.

Support the following platforms:

 * Linux
 * Raspberry
 * Android
 * macOS
 * Windows

It also compiles for Android but never been tested on it.

### Running on Raspberry

It'll be difficult to build on Raspberry. A good way-around is to be build on Linux before sending it to your Raspberry.

First install the arm toolchain, for example on Ubuntu: `sudo apt-get install gcc-multilib-arm-linux-gnueabihf`.

Then configure cargo to use the corresponding toolchain:

```bash
cat << EOF > ~/.cargo/config
[target.armv7-unknown-linux-gnueabihf]
linker = "arm-linux-gnueabihf-gcc"
EOF
```

Finally, cross compile:

```bash
rustup target add armv7-unknown-linux-gnueabihf
cargo build --target=armv7-unknown-linux-gnueabihf
```

## Code example

You have an example into the `examples` folder. Just run `cargo run` inside the `examples` folder to start it. Otherwise, here is a little code sample:

```rust
use sysinfo::{NetworkExt, NetworksExt, ProcessExt, System, SystemExt};

let mut sys = System::new_all();

// We display the disks:
println!("=> disk list:");
for disk in sys.get_disks() {
    println!("{:?}", disk);
}

// Network data:
for (interface_name, data) in sys.get_networks() {
    println!("{}: {}/{} B", interface_name, data.get_received(), data.get_transmitted());
}

// Components temperature:
for component in sys.get_components() {
    println!("{:?}", component);
}

// Memory information:
println!("total memory: {} KiB", sys.get_total_memory());
println!("used memory : {} KiB", sys.get_used_memory());
println!("total swap  : {} KiB", sys.get_total_swap());
println!("used swap   : {} KiB", sys.get_used_swap());

// Number of processors
println!("NB processors: {}", sys.get_processors().len());

// To refresh all system information:
sys.refresh_all();

// We show the processes and some of their information:
for (pid, process) in sys.get_processes() {
    println!("[{}] {} {:?}", pid, process.name(), process.disk_usage());
}
```

## C interface

It's possible to use this crate directly from C. Take a look at the `Makefile` and at the `examples/src/simple.c` file.

To build the C example, just run:

```bash
> make
> ./simple
# If needed:
> LD_LIBRARY_PATH=target/release/ ./simple
```

### Benchmarks

You can run the benchmarks locally with rust **nightly** by doing:

```bash
> cargo bench
```

Here are the current results:

**Linux**

<details>

```text
test bench_new                     ... bench:     182,536 ns/iter (+/- 21,074)
test bench_new_all                 ... bench:  19,911,714 ns/iter (+/- 1,612,109)
test bench_refresh_all             ... bench:   5,649,643 ns/iter (+/- 444,129)
test bench_refresh_components      ... bench:      25,293 ns/iter (+/- 1,748)
test bench_refresh_components_list ... bench:     382,331 ns/iter (+/- 31,620)
test bench_refresh_cpu             ... bench:      13,633 ns/iter (+/- 1,135)
test bench_refresh_disks           ... bench:       2,509 ns/iter (+/- 75)
test bench_refresh_disks_list      ... bench:      51,488 ns/iter (+/- 5,470)
test bench_refresh_memory          ... bench:      12,941 ns/iter (+/- 3,023)
test bench_refresh_networks        ... bench:     256,506 ns/iter (+/- 37,196)
test bench_refresh_networks_list   ... bench:     266,751 ns/iter (+/- 54,535)
test bench_refresh_process         ... bench:     117,372 ns/iter (+/- 8,732)
test bench_refresh_processes       ... bench:   5,125,929 ns/iter (+/- 560,050)
test bench_refresh_system          ... bench:      52,526 ns/iter (+/- 6,786)
test bench_refresh_users_list      ... bench:   2,479,582 ns/iter (+/- 1,063,982)
```
</details>

**Windows**

<details>

```text
test bench_new                     ... bench:   7,335,755 ns/iter (+/- 469,000)
test bench_new_all                 ... bench:  32,233,480 ns/iter (+/- 1,567,239)
test bench_refresh_all             ... bench:   1,433,015 ns/iter (+/- 126,322)
test bench_refresh_components      ... bench:           1 ns/iter (+/- 0)
test bench_refresh_components_list ... bench:   9,835,060 ns/iter (+/- 407,072)
test bench_refresh_cpu             ... bench:      33,873 ns/iter (+/- 2,177)
test bench_refresh_disks           ... bench:      58,951 ns/iter (+/- 6,128)
test bench_refresh_disks_list      ... bench:     125,199 ns/iter (+/- 2,741)
test bench_refresh_memory          ... bench:       1,004 ns/iter (+/- 56)
test bench_refresh_networks        ... bench:      39,013 ns/iter (+/- 2,676)
test bench_refresh_networks_list   ... bench:   1,341,850 ns/iter (+/- 78,258)
test bench_refresh_process         ... bench:       2,116 ns/iter (+/- 58)
test bench_refresh_processes       ... bench:   1,032,447 ns/iter (+/- 57,695)
test bench_refresh_system          ... bench:      35,374 ns/iter (+/- 3,200)
test bench_refresh_users_list      ... bench:   3,321,140 ns/iter (+/- 135,160)
```
</details>

**macOS**

<details>

```text
test bench_new                     ... bench:      87,569 ns/iter (+/- 11,078)
test bench_new_all                 ... bench:  21,445,081 ns/iter (+/- 523,973)
test bench_refresh_all             ... bench:   1,915,573 ns/iter (+/- 296,132)
test bench_refresh_components      ... bench:     293,904 ns/iter (+/- 63,492)
test bench_refresh_components_list ... bench:     894,462 ns/iter (+/- 161,599)
test bench_refresh_cpu             ... bench:       8,636 ns/iter (+/- 1,244)
test bench_refresh_disks           ... bench:         937 ns/iter (+/- 97)
test bench_refresh_disks_list      ... bench:      25,116 ns/iter (+/- 990)
test bench_refresh_memory          ... bench:       2,172 ns/iter (+/- 67)
test bench_refresh_networks        ... bench:     183,552 ns/iter (+/- 2,253)
test bench_refresh_networks_list   ... bench:     183,623 ns/iter (+/- 11,183)
test bench_refresh_process         ... bench:       5,571 ns/iter (+/- 443)
test bench_refresh_processes       ... bench:     764,125 ns/iter (+/- 28,568)
test bench_refresh_system          ... bench:     333,610 ns/iter (+/- 53,204)
test bench_refresh_users_list      ... bench:  16,816,081 ns/iter (+/- 1,039,374)
```
</details>

## Donations

If you appreciate my work and want to support me, you can do it here:

[![Become a patron](https://c5.patreon.com/external/logo/become_a_patron_button.png)](https://www.patreon.com/GuillaumeGomez)
