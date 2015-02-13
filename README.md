# D binding to Duktape library #

### Notes ###

This has been used/tested with Duktape 1.1.x on:

* Ubuntu 14.04 x64
* Windows 7 x64

To run uppercase example:

```
dub run duktape:uppercase --build=release --arch=x86_64 -- "this is a test"
```

For more information about Duktape check out its official website: [http://duktape.org/](http://duktape.org/)

## Examples
```D
import std.conv;
import std.stdio;
import std.string;

import duktape;

extern (C) duk_ret_t native_prime_check(duk_context *ctx) {
    int val = duk_require_int(ctx, 0);
    int lim = duk_require_int(ctx, 1);
    int i;

    for (i = 2; i <= lim; i++) {
        if (val % i == 0) {
            duk_push_false(ctx);
            return 1;
        }
    }

    duk_push_true(ctx);
    return 1;
}

int main() {
    duk_context *ctx = duk_create_heap_default();
    if (!ctx) {
        writeln("Failed to create a Duktape heap.");
        return 1;
    }

    scope(exit) duk_destroy_heap(ctx);

    duk_push_global_object(ctx);
    duk_push_c_function(ctx, &native_prime_check, 2 /*nargs*/);
    duk_put_prop_string(ctx, -2, "primeCheckNative");

    if (duk_peval_file(ctx, "prime.js") != 0) {
        writef("Error: %s\n", duk_safe_to_string(ctx, -1).to!string);
        return 0;
    }
    duk_pop(ctx);  /* ignore result */

    duk_get_prop_string(ctx, -1, "primeTest");
    if (duk_pcall(ctx, 0) != 0) {
        writef("Error: %s\n", duk_safe_to_string(ctx, -1).to!string);
    }
    duk_pop(ctx);  /* ignore result */

    return 0;
}
```