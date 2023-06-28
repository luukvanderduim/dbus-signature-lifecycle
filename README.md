# DBus signature lifecycle

Simple crate to quickly demonstrate just how how lenient DBus is with respect to  
    constructing `Message`s
    and deserializing their bodies.

This produces:

```bash
   Compiling dbus_signature_lifecycle v0.1.0 (/home/luuk/code/dbus_signature_lifecycle)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/dbus_signature_lifecycle`
The wrapped type: `Number3` has the unmarshalled signature: (((ii)))
Marshalled Signature of body: ((((ii))))

This deserializes to a Number just fine
has the body type: Number { num1: 1, num2: 2 }

This deserializes to a tuple with two i's just fine
```

with:

```rust

use serde::{Deserialize, Serialize};
use zbus::MessageBuilder;
use zvariant::Type;

fn main() {
    #[derive(Debug, Clone, Serialize, Type, Deserialize, PartialEq)]
    struct Number {
        num1: i32,
        num2: i32,
    }

    #[derive(Debug, Clone, Serialize, Type, Deserialize, PartialEq)]
    struct Number2 {
        num: Number,
    }

    #[derive(Debug, Clone, Serialize, Type, Deserialize, PartialEq)]
    struct Number3 {
        num: Number2,
    }

    #[derive(Debug, Clone, Serialize, Type, Deserialize, PartialEq)]
    struct Number4 {
        num: Number3,
    }

    #[derive(Debug, Clone, Serialize, Type, Deserialize, PartialEq)]
    struct Number5 {
        num: Number4,
    }

    let t = Number5 {
        num: Number4 {
            num: Number3 {
                num: Number2 {
                    num: Number { num1: 1, num2: 2 },
                },
            },
        },
    };

    println!(
        "The wrapped type: `Number3` has the unmarshalled signature: {}",
        Number3::signature()
    );

    let mb = MessageBuilder::signal("/", "org.zbus.MyInterface", "MySignal")
        .unwrap()
        .build(&t)
        .unwrap();

    println!(
        "Marshalled Signature of body: {}",
        mb.body_signature().unwrap()
    );

    print!("\nThis deserializes to a Number just fine\n");
    let new_number = mb.body::<Number>().unwrap();
    println!("has the body type: {:?}", new_number);

    print!("\nThis deserializes to a tuple with two i's just fine\n");
    let new_number = mb.body::<(i32, i32)>().unwrap();
    println!("has the body type: {:?}", new_number);
}
```
