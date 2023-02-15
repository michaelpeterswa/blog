---
title: "Embedded Rust Programming on WSL (RISC-V)"
date: 2023-02-14T20:31:04-08:00
tags: ["wsl", "rust", "risc-v"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---

![SiFive HiFive1 RevB (discontinued)](https://images.prismic.io/sifive/3ea0c69619fcf3f16db9f8c85514a89a4f2d9d00_boards_hifive1.jpg?auto=compress,format#center)
Image Courtesy of [SiFive, Inc.](https://www.sifive.com/)

I recently picked up the SiFive HiFive 1 Rev B RISC-V microcontroller and decided that I wanted to start with Rust as the first language I developed in for this platform. Here is a thorough write-up of the process on Windows 11 via Windows Subsystem for Linux today (February 2023).

## Process
This post assumes that you already have a properly configured Ubuntu 20.04 WSL install. Installation of Ubuntu on WSL is out of the scope of this post, although more information can be found [here](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview)
### Prepare WSL for USB Devices
Following the provided [Microsoft Documentation](https://learn.microsoft.com/en-us/windows/wsl/connect-usb), the first step to prepare WSL for USB is to run the following commands:

```bash
sudo apt install linux-tools-5.4.0-77-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/5.4.0-77-generic/usbip 20
```

Once complete, it's important to ensure you have USB devices available on the host Windows 11 machine to passthrough. That can be accomplished with the following command (run in Powershell as Administrator):

```powershell
usbipd wsl list
```

If an error is encountered stating that the `usbipd` cmdlet cannot be found (or anything similar), my first suggestion would be to update WSL (older versions of WSL didn't have USB support). That can be done with two commands (again in an Administrator Powershell).

```powershell
wsl --update
wsl --shutdown
```

Once you are able to verify that you have devices available, we can continue to attaching the USB device to a WSL instance.

It should look something like this:

```
PS C:\WINDOWS\system32> usbipd wsl list
BUSID  VID:PID    DEVICE                    STATE
1-10   0b05:185c  Realtek Bluetooth Adapter attached
1-12   1366:1061  JLink CDC UART Port (COM9), JLink CDC UART Port (COM8), B... Not attached
```

### Attaching a USB Device to WSL

Our SiFive HiFive 1 Rev B uses a SEGGER JLink integration to communicate with the computer. We can see above that our board has a `BUSID` of `1-12`. We will need to note that for a future step.

Next, we want to identify the active distributions for WSL that we have on our computer. There is a set of WSL flags that can help us with this. (again in an Administrator Powershell)

```powershell
wsl --list --verbose
```

For this post, we will be using the name `Ubuntu-20.04`.

To attach the USB device to our WSL instance, issue the following command (matching the two flags to the values we found above):

```powershell
usbipd wsl attach --busid 1-12 --distribution Ubuntu-20.04
```

### Installing the JLink software on our WSL VM

Some trickery is needed to download the JLink package from their website programatically. To accept the license agreement and download and install the `.deb` package (explicitly for `x86_64` architectures), run the following:

```bash
curl 'https://www.segger.com/downloads/jlink/JLink_Linux_V784f_x86_64.deb' --data-raw 'accept_license_agreement=accepted&submit=Download+software' --output JLink_Linux_x86_64.deb
sudo dpkg -i JLink_Linux_x86_64.deb
sudo apt-get -f install
```

### Installing Rust via Rustup

`rustup` is the installer for the Rust Programming Language. For WSL, installation is managed with the following command:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will need to select the default install for the purposes of this post. I ended up with this version: `rustc 1.67.1`

### Installing cargo-generate

To install `cargo-generate`, run the next command:

```bash
cargo install cargo-generate
```

### Installing RISC-V Compilation Target

To use our board, we will need a special compilation target. To install it:

```bash
rustup target add riscv32imac-unknown-none-elf
```

### Installing SiFive GNU Toolchain for RISC-V

To install the toolchain, we will need to download and unpack a file from the SiFive website. It's no longer accessible (via their site) to my knowledge, but a direct link to their downloads server still works as of February 2023.

The last `chmod` may or may not be optional... I didn't bother to check.
```bash
curl https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.12/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14.tar.gz --output /home/{your username}/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14.tar.gz
tar -xvf /home/{your username}/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14.tar.gz
chmod +x /home/{your username}/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14/bin/riscv64-unknown-elf-gdb
```

Then, add the following to your `$PATH` in `.bashrc`:

```bash
export PATH="$PATH:/home/{your username}/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14/bin"
```

### Starting the JLink GDB Server

Starting the GDB server at this point is a fairly straightforward process. Just one command:

```bash
JLinkGDBServer -device FE310 -if JTAG -speed 4000 -port 3333 -nogui
```

### Building and Deploying the Example Rust Code

First, we need to instantiate the new application:

```bash
cargo generate --git https://github.com/riscv-rust/riscv-rust-quickstart
```

After naming the app, enter the new app's directory:

```bash
cd {app name}
```

In the `Cargo.toml` file, I needed to make one adjustment. I bumped the dependency `riscv-rt` to `0.11.0` from `0.10.0`. My final dependencies looked like this:

```toml
[dependencies]
embedded-hal = "0.2.7"
hifive1 = { version = "0.10.0", features = ["board-hifive1-revb"] }
panic-halt = "0.2.0"
riscv = "0.10.0"
riscv-rt = "0.11.0"
```

To run the LED Blink example, two final commands will be needed:

```bash
cargo build --example leds_blink
cargo run --example leds_blink
```

Additionally, to view the serial console, you may try the following command:

```bash
sudo screen /dev/ttyACM0 115200
```

### Final Thoughts

I'm excited to try writing some of my own Rust code for the SiFive HiFive 1 Rev B here in the coming weeks. I was admittedly late to the party (this board was released in April 2019), but I still see value in getting into the RISC-V architecture, especially in such an approachable form factor. If you'd like to pick one up for yourself, check out the [Crowd Supply](https://www.crowdsupply.com/sifive/hifive1-rev-b) page!

### Links for Future Research
* [University of Kansas EECS 388 Lab #1](https://www.ittc.ku.edu/~heechul/courses/eecs388/lab1.pdf)
  * [UK EECS 388 Schedule](http://www.ittc.ku.edu/~heechul/courses/eecs388/schedule.html)
* [The Embedded Rust Book](https://docs.rust-embedded.org/book/)
