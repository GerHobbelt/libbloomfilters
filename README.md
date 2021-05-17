**libbf** is a C++11 library which implements a basic bloom filter, including:

This repository is a modified version of libbf, which originally includes various filters, but did not allowed for a serialization of the filters.

This repository only contains a basic bloom filter, but expose a new function, 'basic_bloom_filter::save(std::string filename)', which allows you to save the filter. A new constructor, 'basic_bloom_filter::basic_bloom_filter(hasher h, std::string filename)', allows you to load the filter from your disk.


[blog-post]: http://matthias.vallentin.net/blog/2011/06/a-garden-variety-of-bloom-filters/

Synopsis
========

    #include <iostream>
    #include <bf.h>

    int main()
    {
      bf::basic_bloom_filter b(0.8, 100);

      // Add two elements.
      b.add("foo");
      b.add(42);

      // Test set membership
      std::cout << b.lookup("foo") << std::endl;  // 1
      std::cout << b.lookup("bar") << std::endl;  // 0
      std::cout << b.lookup(42) << std::endl;     // 1

      // Remove all elements.
      b.clear();
      std::cout << b.lookup("foo") << std::endl;  // 0
      std::cout << b.lookup(42) << std::endl;     // 0

      return 0;
    }

Requirements
============

- A C++11 compiler (GCC >= 4.7 or Clang >= 3.2)
- CMake (>= 2.8.12)

Installation
============

The build process uses CMake, wrapped in autotools-like scripts. The configure
script honors the `CXX` environment variable to select a specific C++compiler.
For example, the following steps compile libbf with Clang and install it under
`PREFIX`:

    ./build.sh

Usage
=====

After having installed libbf, you can use it in your application by including
the header file `bf.h` and linking against the library. All data structures
reside in the namespace `bf` and the following examples assume:

    using namespace bf;

Each Bloom filter inherits from the abstract base class `bloom_filter`, which
provides addition and lookup via the virtual functions `add` and `lookup`.
These functions take an *object* as argument, which serves a light-weight view
over sequential data for hashing.

For example, if you can create a basic Bloom filter with a desired
false-positive probability and capacity as follows:

    // Construction.
    bloom_filter* bf = new basic_bloom_filter(0.8, 100);

    // Addition.
    bf->add("foo");
    bf->add(42);

    // Lookup.
    assert(bf->lookup("foo") == 1);
    assert(bf->lookup(42) == 1);

    // Remove all elements from the Bloom filter.
    bf->clear();

In this case, libbf computes the optimal number of hash functions needed to
achieve the desired false-positive rate which holds until the capacity has been
reached (80% and 100 distinct elements, in the above example). Alternatively,
you can construct a basic Bloom filter by specifying the number of hash
functions and the number of cells in the underlying bit vector:

    bloom_filter* bf = new basic_bloom_filter(make_hasher(3), 1024);

Since not all Bloom filter implementations come with closed-form solutions
based on false-positive probabilities, most constructors use this latter form
of explicit resource provisioning.

In the above example, the free function `make_hasher` constructs a *hasher*-an
abstraction for hashing objects *k* times. There exist currently two different
hasher, a `default_hasher` and a
[`double_hasher`](http://www.eecs.harvard.edu/~kirsch/pubs/bbbf/rsa.pdf). The
latter uses a linear combination of two pairwise-independent, universal hash
functions to produce the *k* digests, whereas the former merely hashes the
object *k* times.

Evaluation
----------

libbf also ships with a small Bloom filter tool `bf` in the test directory.
This program supports evaluation the accuracy of the different Bloom filter
flavors with respect to their false-positive and false-negative rates. Have a
look at the console help (`-h` or `--help`) for detailed usage instructions.

The tool operates in two phases:

1. Read input from a file and insert it into a Bloom filter
2. Query the Bloom filter and compare the result to the ground truth

For example, consider the following input file:

    foo
    bar
    baz
    baz
    foo

From this input file, you can generate the real ground truth file as follows:

    sort input.txt | uniq -c | tee query.txt
       1 bar
       2 baz
       2 foo

The tool `bf` will compute false-positive and false-negative counts for each
element, based on the ground truth given. In the case of a simple counting
Bloom filter, an invocation may look like this:

    bf -t counting -m 2 -k 3 -i input.txt -q query.txt | column -t

Yielding the following output:

    TN  TP  FP  FN  G  C  E
    0   1   0   0   1  1  bar
    0   1   0   1   2  1  baz
    0   1   0   2   2  1  foo

The column headings denote true negatives (`TN`), true positives (`TP`), false
positives (`FP`), false negatives (`FN`), ground truth count (`G`), actual
count (`C`), and the queried element. The counts are cumulative to support
incremental evaluation.

Versioning
==========
We follow [Semantic Versioning](http://semver.org/spec/v1.0.0.html). The version X.Y.Z indicates:

* X is the major version (backward-incompatible),
* Y is the minor version (backward-compatible), and
* Z is the patch version (backward-compatible bug fix).

License
========

libbf comes with a BSD-style license (see [COPYING](COPYING) for details).

