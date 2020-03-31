## Upgrading from 3.x to 4.0

### BER String types verification

Some BER String types (`IA5String`, `NumericString`, `PrintableString` and `UTF8String`) are now
verified, and will now only parse if the characters are valid.

Their types have change from slice to `str` in the `BerObjectContent` enum.

### BerClass

The `class` field of `BerObject` struct now uses the newtype `BerClass`. Use the provided constants
(for ex `BerClass:Universal`). To access the value, just use `class.0`.

### Oid

This is probably the most impacting change.

OID objects have been refactored, and are now zero-copy. This has several consequences:

- `Oid` struct now has a lifetime, which must be propagated to objects using them
  - This makes having globally static structs difficult. Obtaining a `'static` object is possible
    using the `oid` macro. For ex:

```rust
const SOME_STATIC_OID: Oid<'static> = oid!(1.2.456);
```

- Due to limitations of procedural macros  ([rust
  issue](https://github.com/rust-lang/rust/issues/54727)) and constants used in patterns ([rust issue](https://github.com/rust-lang/rust/issues/31434)), the `oid` macro can not directly be used in patterns, also not through constants.
You can do this, though:

```rust
# use der_parser::{oid, oid::Oid};
# let some_oid: Oid<'static> = oid!(1.2.456);
const SOME_OID: Oid<'static> = oid!(1.2.456);
if some_oid == SOME_OID || some_oid == oid!(1.2.456) {
    println!("match");
}

// Alternatively, compare the DER encoded form directly:
const SOME_OID_RAW: &[u8] = &oid!(raw 1.2.456);
match some_oid.bytes() {
    SOME_OID_RAW => println!("match"),
    _ => panic!("no match"),
}
```
*Attention*, be aware that the latter version might not handle the case of a relative oid correctly. An
extra check might be necessary.

- To build an `Oid`, the `from`, `new` or `new_relative` methods can be used.
- The `from` method now returns a `Result` (failure can happen if the first components are too
  large, for ex)
- An `oid` macro has also been added in the `der-oid-macro` crate to easily build an `Oid` (see
  above).