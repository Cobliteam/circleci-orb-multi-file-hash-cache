# circleci-orb-multi-file-hash-cache
[CircleCI Orb](https://circleci.com/docs/2.0/orb-intro/) code to handle the
cache system considering multiples files to generate the hash part of keys

Useful when you need to chase multiple package managers

Note that we assume a cache name in the format:

  *$cache-prefix-${checksum out-file}-branch*


Also we will restore it in the following sequence:
  - *$cache-prefix-${checksum out-file}-branch*
  - *$cache-prefix-${checksum out-file}*
  - *$cache-prefix*


## Example
```yaml
version: 2.1
variables:
  multi_hash_args: &multi_hash_args
    in-files: "file1.txt file2.yml"
    out-file: /tmp/infiles_checksum
    cache-prefix: dep-v1
  cache_name: &cache_name dep-v1-{{ checksum "/tmp/infiles_checksum" }}-{{ .Branch }}
orbs:
  multi_file_hash_cache: cobli/multi-file-hash-cache@x.x.x
workflows:
  tests:
    jobs:
      - test1
      - test2
jobs:
  test1:
    docker:
      - image: circleci/openjdk:8u222-jdk-stretch
    steps:
      - multi_file_hash_cache/restore:
          <<: *multi_hash_args
      - run:
          name: My first test
          command: echo "Testing"
      - save_cache:
          paths:
            - ~/.ivy2/cache
            - ~/sbt/boot/
            - ~/.cousier/cache
          key: *cache_name
  test2:
    docker:
      - image: circleci/openjdk:8u222-jdk-stretch
    steps:
      - multi_file_hash_cache/restore:
          <<: *multi_hash_args
      - run:
          name: My second test
          command: echo "Testing again..."
```

## Needs:
- [sha256sum](https://linux.die.net/man/1/sha256sum) => This orb makes use of
  the command line sha256sum so the CircleCI machine or base container needs to
  have it.
