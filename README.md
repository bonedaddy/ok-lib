# ok-lib
[![Build Status](https://travis-ci.org/brackeen/ok-lib.svg?branch=master)](https://travis-ci.org/brackeen/ok-lib)
[![Build Status](https://ci.appveyor.com/api/projects/status/8hq8b21kb4xts5b3/branch/master?svg=true)](https://ci.appveyor.com/project/brackeen/ok-lib/branch/master)

Generic vector, hash map, and concurrent queue for C.

## Goals
* Easy-to-use API.
* Collections that work with any type, including user-defined structs.
* C11 support where available (using `_Generic`).
* No function generation via macros.
* Single-file header - just include [`ok_lib.h`](ok_lib.h).

## Non-Goals
* Support C versions older than C99.
* Thread safety for all collection types.
* Create the best possible implementation. It is not a goal to create the fastest or most memory efficient collection, or provide every possible option a developer may want. However, these implementations do compare favorably to others. In short, these collections are *ok*.

## Thread Safety
* The `ok_queue` is a thread-safe concurrent queue. Specifically, it is a lock-free multi-producer multi-consumer concurrent queue, and it is wait-free when there is only one producer thread and only one consumer thread.
* The `ok_vec` and `ok_map` are *not* thread-safe.

## Vector Example
```C
typedef struct ok_vec_of(const char *) str_vec_t;

str_vec_t vec;
ok_vec_init(&vec);

ok_vec_push(&vec, "dave");
ok_vec_push(&vec, "mary");
ok_vec_push(&vec, "steve");

const char *value = ok_vec_get(&vec, 2);

printf("A person: %s\n", value);

ok_vec_foreach(&vec, const char *value) {
    printf("Name: %s\n", value);
}

ok_vec_deinit(&vec);

```
## Map Example
```C
typedef struct ok_map_of(const char *, char *) str_map_t;

str_map_t map;
ok_map_init(&map);

ok_map_put(&map, "dave", "bananas");
ok_map_put(&map, "mary", "grapes");
ok_map_put(&map, "steve", "pineapples");

char *value = ok_map_get(&map, "dave");

printf("dave likes %s.\n", value);

ok_map_foreach(&map, const char *key, char *value) {
    printf("> %s likes %s.\n", key, value);
}

ok_map_deinit(&map);
```

## Map Example With a Custom Key Type
```C
typedef struct {
    float x;
    float y;
} point_t;

static ok_hash_t point_hash(point_t p) {
    ok_hash_t hash_x = ok_float_hash(p.x);
    ok_hash_t hash_y = ok_float_hash(p.y);
    return ok_hash_combine(hash_x, hash_y);
}

static bool point_equals(const void *v1, const void *v2) {
    const point_t *p1 = (const point_t *)v1;
    const point_t *p2 = (const point_t *)v2;
    return (p1->x == p2->x && p1->y == p2->y);
}

...

typedef struct ok_map_of(point_t, char *) point_map_t;

point_map_t map;
ok_map_init_custom(&map, point_hash, point_equals);

point_t key = { 100.0, 200.0 };
ok_map_put(&map, key, "Buried treasure");

printf("Value at (%f, %f): %s\n", key.x, key.y, ok_map_get(&map, key));

ok_map_deinit(&map);

```

## Queue Example
```C
typedef struct ok_queue_of(int) int_queue_t;

int_queue_t queue;
ok_queue_init(&queue);

for (int i = 0; i < 3; i++) {
    ok_queue_push(&queue, i);
}

...

int value;
while (ok_queue_pop(&queue, &value)) {
    printf("Got %i\n", value);
}

ok_queue_deinit(&queue);

```

## Implementation Overview

The `ok_vec` is implemented as a structure containing an array that is reallocated as needed.

The `ok_map` is implemented using [open addressing](https://en.wikipedia.org/wiki/Open_addressing), with linear probing and cleanup on deletion (no lazy deletion).

The `ok_queue` is implemented as a two-lock concurrent queue, with blocks of elements instead of nodes. It uses `<stdatomic.h>` if available, otherwise it uses the Windows Interlocked API or GCC's atomic builtins (which also works on Clang).

## Tests
* The [Tests](extras/test) are run on [Travis CI](https://travis-ci.org/brackeen/ok-lib) (Linux and macOS) and [Appveyor](https://ci.appveyor.com/project/brackeen/ok-lib/branch/master) (Windows).
* Valgrind is used on Linux if available. Tests fail if there is a memory leak.
* The queue has a multithreaded test with eight producer threads and eight consumer threads.

## Extras
* [More examples](extras/example)
* [C++ wrapper](extras/wrapper)
* [Tests](extras/test)
