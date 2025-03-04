# elm test [![Travis build Status](http://travis-ci.org/elm-explorations/test.svg?branch=master)](https://travis-ci.org/elm-explorations/test)

Write unit and fuzz tests for Elm code.

When people say “elm-test” they usually refer to either:

- This Elm package for writing tests.
- [rtfeldman/node-test-runner](https://github.com/rtfeldman/node-test-runner) – a CLI tool (called `elm-test`) for running tests defined using this package in the terminal.

## Quick Start

Here are three example tests:

```elm
suite : Test
suite =
    describe "The String module"
        [ describe "String.reverse" -- Nest as many descriptions as you like.
            [ test "has no effect on a palindrome" <|
                \_ ->
                    let
                        palindrome =
                            "hannah"
                    in
                        Expect.equal palindrome (String.reverse palindrome)

            -- Expect.equal is designed to be used in pipeline style, like this.
            , test "reverses a known string" <|
                \_ ->
                    "ABCDEFG"
                        |> String.reverse
                        |> Expect.equal "GFEDCBA"

            -- fuzz runs the test 100 times with randomly-generated inputs!
            , fuzz string "restores the original string if you run it again" <|
                \randomlyGeneratedString ->
                    randomlyGeneratedString
                        |> String.reverse
                        |> String.reverse
                        |> Expect.equal randomlyGeneratedString
            ]
        ]
```

This code uses a few common functions:

* [`describe`](https://package.elm-lang.org/packages/elm-explorations/test/latest/Test#describe) to add a description string to a list of tests
* [`test`](https://package.elm-lang.org/packages/elm-explorations/test/latest/Test#test) to write a unit test
* [`Expect`](https://package.elm-lang.org/packages/elm-explorations/test/latest/Expect) to determine if a test should pass or fail
* [`fuzz`](https://package.elm-lang.org/packages/elm-explorations/test/latest/Test#fuzz) to run a function that produces a test several times with randomly-generated inputs

Check out [a large real-world test suite](https://github.com/rtfeldman/elm-css/tree/master/tests) for more.

Tip! Run [elm-test init](https://github.com/rtfeldman/node-test-runner#init) to generate some example tests to get you started.

### Running tests

This package lets you define tests ([Test](https://package.elm-lang.org/packages/elm-explorations/test/latest/Test#Test) values).

To execute your tests and see if they pass you need a program that can consume your tests, run them and report the results.

Two most popular test runners are:
- [rtfeldman/node-test-runner](https://github.com/rtfeldman/node-test-runner)
- [mpizenberg/elm-test-rs](https://github.com/mpizenberg/elm-test-rs)

Both are CLI tools that let you run tests in the terminal. Head over to those projects for instructions on how to get started!

### Not running tests

During development, you'll often want to focus on specific tests, silence failing tests, or jot down many ideas for tests that you can't implement all at once. We've got you covered with `skip`, `only`, and `todo`:

```elm
wipSuite : Test
wipSuite =
    describe "skip, only, and todo"
        [ only <|
            describe "Marking this test as `only` means no other tests will be run!"
                [ test "This test will be run" <|
                    \_ -> 1 + 1 |> Expect.equal 2
                , skip <| test "This test will be skipped, even though it's in an `only`!" <|
                    \_ -> 2 + 3 |> Expect.equal 4
                ]
        , test "This test will be skipped because it has no `only`" <|
            \_ -> "left" |> Expect.equal "right"
        , todo "Make sure all splines are reticulated"
        ]
```

If you run this example, or any suite that uses one of these three functions, it will result in an _incomplete_ test run. No tests failed, but you also didn't run your entire suite, so we can't call it a success either. Incomplete test runs are reported to CI systems as indistinguishable from failed test runs, to safeguard against accidentally committing a gutted test suite!

## Strategies for effective testing

1. [Make impossible states unrepresentable](https://www.youtube.com/watch?v=IcgmSRJHu_8) so that you don't have to test that they can't occur.
1. When doing TDD, treat compiler errors as a red test. So feel free to write the test you wish you had even if it means calling functions that don't exist yet!
1. If your API is difficult for you to test, it will be difficult for someone else to use. You are your API's first client.
1. Prefer fuzz tests to unit tests, where possible. If you have a union type with a small number of values, list them all and map over the list with a unit test for each. Unit tests are also great for when you know the edge cases, and for regression tests.
1. If you're writing a library that wraps an existing standard or protocol, use examples from the specification or docs as unit tests.
1. For simple functions, it's okay to copy the implementation to the test; this is a useful regression check. But if the implementation isn't obviously right, try to write tests that don't duplicate the suspect logic. The great thing about fuzz tests is that you don't have to arrive at the exact same value as the code under test, just state something that will be true of that value.
1. Tests should be small and focused: call the code under test and set an expectation about the result. Setup code should be moved into reusable functions, or custom fuzzers. For example, a test to remove an element from a data structure should be given a nonempty data structure; it should not have to create one itself.
1. If you find yourself inspecting the fuzzed input and making different expectations based on it, split each code path into its own test with a fuzzer that makes only the right kind of values.
1. Consider using [elm-verify-examples](https://github.com/stoeffel/elm-verify-examples) to extract examples in your docs into unit tests.
1. Not even your test modules can import unexposed functions, so test them only as the exposed interface uses them. Don't expose a function just to test it. Every exposed function should have tests. (If you practice TDD, this happens automatically!)
1. How do you know when to stop testing? This is an engineering tradeoff without a perfect answer. If you don't feel confident in the correctness of your code, write more tests. If you feel you are wasting time testing better spent writing your application, stop writing tests for now.

### Application-specific techniques
There are a few extra ideas that apply to testing webapps and reusable view packages:

1. Avoid importing your `Main` module. Most of your code belongs in other modules, so import those instead.
1. Test your views using `Test.Html.Query`, `Test.Html.Selector`, and `Test.Html.Event`.
1. There is currently no Elm solution for integration or end-to-end testing. Use tools from outside the Elm ecosystem such as [Capybara](https://www.rubydoc.info/github/teamcapybara/capybara) or [Cypress](https://www.cypress.io).
