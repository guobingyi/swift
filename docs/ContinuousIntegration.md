
# Continuous Integration for Swift

**Table of Contents**

- [Introduction](#introduction)
- [Pull Request Testing](#pull-request-testing)
    - [@swift-ci](#swift-ci)
    - [Smoke Testing](#smoke-testing)
    - [Validation Testing](#validation-testing)
    - [Lint Testing](#lint-testing)
- [Cross Repository Testing](#cross-repository-testing)
- [ci.swift.org bots](#ciswiftorg-bots)

## Introduction

FIXME: FILL ME IN!

## Pull Request Testing

In order for the Swift project to be able to advance quickly, it is important that we maintain a green build [[1]](#footnote-1). In order to help maintain this green build, the Swift project heavily uses pull request (PR) testing. Specifically, an important general rule is that **all** non-trivial checkins to any Swift Project repository should should at least perform a [smoke test](#smoke-testing) if simulators will not be impacted *or* a full [validation test](#validation-testing) if simulators may be impacted. If in addition one is attempting to make a source breaking change across multiple repositories one should follow the cross repo source breaking changes workflow. We now continue by describing the Swift system for Pull Request testing, @swift-ci:

### @swift-ci

swift-ci pull request testing is triggered by writing a comment on this PR addressed to the GitHub user @swift-ci. Different tests will run depending on the specific comment that you use. The current test types are:

1. Smoke Testing
2. Validation Testing
3. Benchmarking.
4. Lint Testing

We describe each in detail below:

### Smoke Testing

        Platform     | Comment | Check Status
        ------------ | ------- | ------------
        All supported platforms     | @swift-ci Please smoke test       		 | Swift Test Linux Platform (smoke test) <br> Swift Test OS X Platform (smoke test)
        All supported platforms     | @swift-ci Please smoke test and merge		 | Swift Test Linux Platform (smoke test) <br> Swift Test OS X Platform (smoke test)
        OS X platform               | @swift-ci Please smoke test OS X platform  | Swift Test OS X Platform (smoke test)
        Linux platform              | @swift-ci Please smoke test Linux platform | Swift Test Linux Platform (smoke test)

A smoke test on macOS does the following:

1. Builds LLVM/Clang incrementally.
2. Builds Swift clean.
3. Builds the standard library clean only for macOS. Simulator standard libraries and
   device standard libraries are not built.
4. lldb is not built.
5. The test and validation-test targets are run only for macOS. The optimized
   version of these tests are not run.

A smoke test on Linux does the following:

1. Builds LLVM/Clang incrementally.
2. Builds Swift clean.
3. Builds the standard library clean.
4. lldb is built incrementally.
5. Foundation, SwiftPM, LLBuild, XCTest are built.
6. The swift test and validation-test targets are run. The optimized version of these
   tests are not run.
7. lldb is tested.
8. Foundation, SwiftPM, LLBuild, XCTest are tested.

### Validation Testing

        Platform     | Comment | Check Status
        ------------ | ------- | ------------
        All supported platforms     | @swift-ci Please test							| Swift Test Linux Platform (smoke test)<br>Swift Test OS X Platform (smoke test)<br>Swift Test Linux Platform<br>Swift Test OS X Platform<br>  
        All supported platforms     | @swift-ci Please clean test					| Swift Test Linux Platform (smoke test)<br>Swift Test OS X Platform (smoke test)<br>Swift Test Linux Platform<br>Swift Test OS X Platform<br>  
        All supported platforms     | @swift-ci Please test and merge				| Swift Test Linux Platform (smoke test) <br> Swift Test OS X Platform (smoke test)<br> Swift Test Linux Platform <br>Swift Test OS X Platform
        OS X platform               | @swift-ci Please test OS X platform 			| Swift Test OS X Platform (smoke test)<br>Swift Test OS X Platform
        OS X platform               | @swift-ci Please clean test OS X platform     | Swift Test OS X Platform (smoke test)<br>Swift Test OS X Platform
        OS X platform               | @swift-ci Please benchmark                    | Swift Benchmark on OS X Platform
        Linux platform              | @swift-ci Please test Linux platform          | Swift Test Linux Platform (smoke test) <br> Swift Test Linux Platform
        Linux platform              | @swift-ci Please clean test Linux platform    | Swift Test Linux Platform (smoke test) <br> Swift Test Linux Platform

The core principles of validation testing is that:

1. A validation test should build and run tests for /all/ platforms and all
   architectures supported by the CI.
2. A validation test should not be incremental. We want there to be a
   definitiveness to a validation test. If one uses a validation test, one
   should be sure that there is no nook or cranny in the code base that has not
   been tested.

With that being said, a validation test on macOS does the following:

1. Removes the workspace.
2. Builds the compiler.
3. Builds the standard library for macOS and the simulators for all platforms.
4. lldb is /not/ build/tested [[2]](#footnote-2)
5. The tests, validation-tests are run for all simulators and macOS both with
   and without optimizations enabled.

A validation test on Linux does the following:

1. Removes the workspace.
2. Builds the compiler.
3. Builds the standard library.
4. lldb is built.
5. Builds Foundation, SwiftPM, LLBuild, XCTest
6. Run the swift test and validation-test targets with and without optimization.
7. lldb is tested.
8. Foundation, SwiftPM, LLBuild, XCTest are tested.

### Benchmarking

        Platform     | Comment | Check Status
        ------------ | ------- | ------------
        OS X platform  | @swift-ci Please benchmark | Swift Benchmark on OS X Platform

### Lint Testing

        Language     | Comment | Check Status
        ------------ | ------- | ------------
        Python       | @swift-ci Please Python lint | Python lint

## Cross Repository Testing

Simply provide the URL from corresponding pull requests in the same comment as "@swift-ci Please test" phrase. List all of the pull requests and then provide the specific test phrase you would like to trigger. Currently, it will only merge the main pull request you requested testing from as opposed to all of the PR's. 

For example:   

```
Please test with following pull request:
https://github.com/apple/swift/pull/4574

@swift-ci Please test Linux platform
```

```
Please test with following PR:
https://github.com/apple/swift-lldb/pull/48
https://github.com/apple/swift-package-manager/pull/632

@swift-ci Please test macOS platform
```

1. Create a separate PR for each repository that needs to be changed. Each should reference the main Swift PR and create a reference to all of the others from the main PR.

2. Gate all commits on @swift-ci smoke test and merge. As stated above, it is important that *all* checkins perform PR testing since if breakage enters the tree PR testing becomes less effective. If you have done local testing (using build-toolchain) and have made appropriate changes to the other repositories then perform a smoke test and merge should be sufficient for correctness. This is not meant to check for correctness in your commits, but rather to be sure that no one landed changes in other repositories or in swift that cause your PR to no longer be correct. If you were unable to make workarounds to th eother repositories, this smoke test will break *after* Swift has built. Check the log to make sure that it is the expected failure for that platform/repository that coincides with the failure your PR is supposed to fix.

3. Merge all of the pull requests simultaneously.

4. Watch the public incremental build on ci.swift.org to make sure that you did not make any mistakes. It should complete within 30-40 minutes depending on what else was being committed in the mean time.


## ci.swift.org bots

FIXME: FILL ME IN!

<a name="footnote-1">[1]</a> Even though it should be without saying, the reason why having a green build is important is that:

1. A full build break can prevent other developers from testing their work.
2. A test break can make it difficult for developers to know whether or not their specific commit has broken a test, requiring them to perform an initial clean build, wasting time.
3. @swift-ci pull request testing becomes less effective since one can not perform a test and merge and one must reason about the source of a given failure.

<a name="footnote-2">[2]</a> This is due to unrelated issues relating to running lldb tests on macOS.
