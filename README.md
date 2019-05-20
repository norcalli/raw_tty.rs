# `raw_tty`

This crate can be used for generally interacting with a tty's mode safely, but was
created originally to solve the problem of using raw mode with /dev/tty while reading
stdin for data.

# Usage


## Raw Mode

Description from the `termion` crate:
>Managing raw mode.

>Raw mode is a particular state a TTY can have. It signifies that:

>1. No line buffering (the input is given byte-by-byte).
>2. The input is not written out, instead it has to be done manually by the programmer.
>3. The output is not canonicalized (for example, `\n` means "go one line down", not "line
>   break").

>It is essential to design terminal programs.

### Example

```no_run
use raw_tty::IntoRawMode;
use std::io::{Write, stdin, stdout};
                                                                                           
fn main() {
    let stdin = stdin().into_raw_mode().unwrap();
    let mut stdout = stdout();
                                                                                           
    write!(stdout, "Hey there.").unwrap();
}
```

### Example with /dev/tty

```
use raw_tty::IntoRawMode;
use std::io::{self, Read, Write, stdin, stdout};
use std::fs;
                                                                                           
fn main() -> io::Result<()> {
    let mut tty = fs::OpenOptions::new().read(true).write(true).open("/dev/tty")?;
    // Can use the tty_input for keys while also reading stdin for data.
    let mut tty_input = tty.try_clone()?.into_raw_mode();
    let mut buffer = String::new();
    stdin().read_to_string(&mut buffer)?;
                                                                                           
    write!(tty, "Hey there.")
}
```

## General example

```no_run
use raw_tty::GuardMode;
use std::io::{self, Write, stdin, stdout};
                                                                                           
fn test_into_raw_mode() -> io::Result<()> {
    let mut stdin = stdin().guard_mode()?;
    stdin.set_raw_mode()?;
    let mut out = stdout();
                                                                                           
    out.write_all(b"this is a test, muahhahahah\r\n")?;
                                                                                           
    drop(out);
    Ok(())
}
                                                                                           
fn main() -> io::Result<()> {
    let mut stdout = stdout().guard_mode()?;
    stdout.modify_mode(|ios| /* do stuff with termios here */ ios)?;
                                                                                           
    // Have to use &* since TtyModeGuard only implements
    // deref, unlike RawReader which implements read specifically.
    // Otherwise, it wouldn't be recognized as `Write`able.
    write!(&mut *stdout, "Hey there.")
}
                                                                                           
```
