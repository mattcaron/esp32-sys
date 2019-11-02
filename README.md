# Rust esp32-sys support crate #

## Introduction ##

This repository is a little different than most rust crates. Because
the bindings being generated are dependent on the ESP-IDF headers,
which are dependent on the SDK config, this needs to be able to find
those headers and have the bindings regenerated if you change the SDK
config.

## Credit ##

It is based off https://github.com/lexxvir/esp32-hello. Thanks to
lexxvir for their work on this.

## Getting started ##

1. Create an empty rust project with cargo, as normal, but don't do
   anything with it yet.

2. Follow the ESP-IDF
   [instructions](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/)
   and add the example project to the rust project you created. Make
   sure go run through the sdk config, select the options you want,
   and build the example application.
   
   **Note: 1** Add the esp-idf as a submodule to this repository, and
   set your IDF_PATH to it.
   
   **Note: 2** For your environment setup, make sure to define the
   following environment variables. I recommend adding them in a file
   called `setenv.sh` (which this module expects to find when it runs
   `bindgen.sh`) as follows:
   
   ```
   export XARGO_RUST_SRC=${HOME}/workspace/esp32/rust/rust-xtensa/src
   export IDF_PATH=${HOME}/workspace/esp32/rust/esp32-hello/esp-idf
   export PATH=${HOME}/workspace/esp32/rust/llvm_build/bin:${HOME}/workspace/esp32/rust/rust_build/bin:${HOME}/.espressif/tools/xtensa-esp32-elf/esp32-2019r1-8.2.0/xtensa-esp32-elf/bin/:${PATH}
   export TARGET_DIR=target/xtensa-esp32-none-elf/release
   export LIBCLANG_PATH=$HOME/workspace/esp32/rust/llvm_build/lib
   ```
   
   Note that the above are example paths from my environment; you will
   need to modify for your own environment accordingly.
   
3. Add this repository as a submodule to your example application. So,
   now you should have two - esp32-sys and esp-idf.
   
4. I recommend making a new branch in esp32-sys, so you can commit any
   changes to it, as the bindings will get updated based on your SDK
   config changes.
   
5. Make sure your application is built (`make`).

6. Change to the esp32-sys subdir and run `./bindgen.sh`. If all
   works, you should see no output and, if the bindings were changed,
   you'll see that `src/bindings.rs` has been updated. Commit the
   result, if you like, on your branch. (You could even have your
   build script automatically regenerate the bindings on every build,
   if you like).

7. Make sure to add the esp32-sys crate to your Cargo.toml.

8. Set up your .cargo/config accordingly. Either steal one from
   [esp32-hello](https://github.com/mattcaron/esp32-hello) or build
   your own by grabbing the output and splitting out all the linker
   flags and passing them as rustflags for the `xtensa-esp32-none-elf`
   target.

9. Make sure your rust code begins like this:

```
#![no_std]
#![no_main]

extern crate esp32_sys;

use core::panic::PanicInfo;
use core::ptr;
use esp32_sys::*;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[no_mangle]
pub fn app_main() {
}
```
   The idea is that your code goes in `app_main`, which is called
   from the ESP-IDF main() (over which you have no control). We also
   don't use the std crate (hence `no_std`) and want to use the
   `esp32_sys` crate. Fianlly we define a panic handler to give us
   information if bad things happen.
   
10. Build it! You need something like this:

```
make -j6 app && \
rustup run xtensa xargo build --release --verbose && \
$IDF_PATH/components/esptool_py/esptool/esptool.py \
        --chip esp32 \
        elf2image \
        --flash_mode "dio" \
        --flash_freq "40m" \
        --flash_size "2MB" \
        -o $TARGET_DIR/rusty_treadmill.bin \
        $TARGET_DIR/rusty_treadmill
```
   
## References ##

* https://github.com/mattcaron/esp32-hello

