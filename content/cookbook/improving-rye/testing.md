---
title: "Testing*"
date: 2024-10-24T19:30:08+10:00
draft: false
group: true
weight: 250
summary: ""
mygroup: true
---

*work in progress*

# Testing

You can find current testing tool and test in `tests/` folder. To run then use:

```
cd tests
rye .                  # displays more information
rye . test             # runs all the tests
rye . test basics      # runs specific test group
rye . doc              # generates reference docs
```

&nbsp;

Example of use:

![usage and running a test](../testing_passed.png)

## But ...

We are currently improving and unifying how builtin functions are documented which includes the test so the
system is changing. Look at the [One source page](../one-source/) for more information about it.

Tests will first have to be moved to their respective function definitions, additional tests will be written
and the structure with tests and all other information will then get generated from that.

We are in the process of transition, so right now the best way is to write tests at builtin definitions, but
there is not yet a good way to execute them yet. But it should change in a week or two max.


