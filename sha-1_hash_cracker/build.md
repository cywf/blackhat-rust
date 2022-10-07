# Building a SHA-1 Hash Cracker

```sh
$ cargo new sha1_cracker
```

Will create a new project in the folder sha1_cracker

Note that by default, will create a binary (application) project
You can create a library project with the flag: `cargo new my_lib --lib`

[SHA-1](https://en.wikipedia.org/wiki/SHA-1) is a [hash function](https://en.wikipedia.org/wiki/Hash_function) used by a lot of old websites to store the passwords of the users. In theory, a hashed password can’t be recovered from its hash. Thus by storing the hash in their database, a website can assert that a given user has the knowledge of its password without storing the password in cleartext, by comparing the hashes. So if the website’s database is breached, there is no way to recover the passwords and access the users’ data.

Reality is quite different. Let’s imagine a scenario where we just breached such a website, and we now want to recover the credentials of the users in order to gain access to their accounts. This is where a “hash cracker” is useful. A hash cracker is a program that will try many different hashes in order to find the original password.

This is why when creating a website, you should use a hash function specifically designed for this use case, such as [argon2id](https://en.wikipedia.org/wiki/Argon2) which require way more resource to bruteforce than SHA-1, for example.

## What we will learn from this

- How to use [Command Line Interface (CLI)](https://en.wikipedia.org/wiki/Command-line_interface) arguments
- How to read files
- Ho to use an external library
- Basic error handling
- Resources management 

The entrypoint of a Rust program is its main function (main.rs)

```rs
fn main() {
    // ...
}
```

Reading command line arguments

```rs
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
}
```

Where `std::env` imports the module `env` from the standard library and `env::args()` calls the `args` function from this module and returns an [iterator](https://doc.rust-lang.org/book/ch13-02-iterators.html) which can be “collected” into a `Vec<String>` , a `Vector` of `String` objects. A `Vector` is an array type that can be resized.

It is then easy to check for the number of arguments and display an error message if it does not match what is expected

```rs
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() != 3 {
        println!("Usage:");
        println!("sha1_cracker: <wordlist.txt> <sha1_hash>");
        return;
    }
}
```

Note: `println!` is not a classic function, but a macro. [Ref](https://doc.rust-lang.org/book/ch19-06-macros.html)
`println!` is a macro and not a function because Rust doesn't support [variadic generics](https://github.com/rust-lang/rust/issues/17190#issuecomment-71330815). It has the advantage of being compile-time evaluated and checked and thus prevent vulnerabilities such as [format string vulnerabilities](https://owasp.org/www-community/attacks/Format_string_attack)

---

## Error Handling

For our simple program, we will `Box` `errors`: we will allow our program to return any type that implements the `std::error::Error` trait. What is a trait? More on that later

```rs
use std::{
    env,
    error::Error,
};

const SHA1_HEX_STRING_LENGTH: usize = 40;

fn main() -> Result<(), Box<dyn Error>> {
    let args: Vec<String> = env::args().collect();

    if args.len() != 3 {
        println!("Usage:");
        println!("sha1_cracker: <wordlist.txt> <sha1_hash>");
        return Ok(());
    }

    let has_to_crack = args[2].trim();
    if hash_to_crack.len() != SHA1_HEX_STRING_LENGTH {
        return Err("sha1 hash is not valid".into());
    }

    Ok(())

}
```

---

## Reading Files

As it takes too much time to test all possible combinations of letters, numbers, and special characters, we need to reduce the number of SHA-1 hashes generated. For that, we use a special kind of dictionary, known as a wordlist, which contains the most common passwords found in breached websites.

Reading a file in Rust can be achieved with the standard library like that:

```rs
use std::{
    env,
    error::Error,
    fs::File,
    io::{BufRead, BufReader},
};

const SHA1_HEX_STRING_LENGTH: usize = 40;

fn main() -> Result<(), Box<dyn Error>> {
    let args: Vec<String> = env::args().collect();

    if args.len() != 3 {
        println!("Usage:");
        println!("sha1_cracker: <wordlist.txt> <sha1_hash>");
        return Ok(());
    }

    let hash_to_crack = args[2].trim();
    if hash_to_crack.len() != SHA1_HEX_STRING_LENGTH {
        return Err("sha1 hash is not valid".into());
    }

    let wordlist_file = File::open(&args[1])?;
    let reader = BufReader::new(&wordlist_file);

    for line in reader.lines() {
        let line = line?.trim().to_string();
        println!("{}", line);
    }

    Ok(())

}
```

---

## Creates

Now that the basic structure of our program is in place, we need to actually compute the SHA- 1 hashes. Fortunately for us, some talented developers have already developed this complex piece of code and shared it online, ready to use in the form of an external library. In Rust, we call those libraries, or packages, crates. They can be browsed online at [crates.io](https://crates.io)

They are managed with [cargo](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html) : Rust’s package manager. Before using a crate in our
program, we need to declare its version in Cargo’s manifest file: `Cargo.toml`

```toml
[package]
name = "sha1_cracker"
version = "0.1.0"
authors = ["Kyle Prisher"]
edition = "2022"

# See more keys and their definitions at
# https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
sha-1 = "0.9"
hex = "0.4"
```

We can then import it in our SHA-1 cracker:

```rs
use sha1::Digest;
use std::{
    env,
    error::Error,
    fs::File,
    io::{BufRead, BufReader},
};

const SHA1_HEX_STRING_LENGTH: usize = 40;

fn main() -> Result<(), Box<dyn Error>> {
    let args: Vec<String> = env::args().collect();

    if args.len() != 3 {
        println!("Usage:");
        println!("sha1_cracker: <wordlist.txt> <sha1_hash>");
        return Ok(());
    }

    let hash_to_crack = args[2].trim();
    if hash_to_crack.len() != SHA1_HEX_STRING_LENGTH {
        return Err("sha1 hash is not valid".into());
    }

    let wordlist_file = File::open(&args[1])?;
    let reader = BufReader::new(&wordlist_file);

    for line in reader.lines() {
        let line = line?;
        let common_password = line.trim();
        if hash_to_crack ==
            &hex::encode(sha1::Sha1::digest(common_password.as_bytes()) {
            println!("Password found: {}", &common_password);
            return Ok(());
        }
    }

    println!("password not found in wordlist: {}");

    Ok(())
}
```

Our first program is now complete. We can test it by running:

```sh
$ cargo run -- wordlist.txt 7c6a61c68ef8b9b6b061b28c348bc1ed7921cb53
```

Please note that in a real-world scenario, we may want to use optimized hash crackers such as [hashcat](https://hashcat.net) or [John the Ripper](https://www.openwall.com/john/), which, among other things, may use the GPU to significantly speed up the cracking.

Another point would be to first load the wordlist in memory before performing the computa- tions
