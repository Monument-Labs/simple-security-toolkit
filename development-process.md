# Development Process
One of the most crucial factors in having a secure codebase is a solid development process: "an ounce of prevention is worth a pound of cure." This document gives an *example* development process that we, at [Nascent](https://www.nascent.xyz), have found works well. In addition to getting better overall code quality, this process serves as a companion to our [audit readiness checklist](audit-readiness-checklist.md). If you follow the below process, you should naturally check off most of the items on the checklist.

```
Feature Request
 |
 └> Specification
     |
     └> Evaluation (time + complexity + risks)
        |
        └> Implementation
           |
           └> Testing
              |
              └> Deployment
                 |
                 └> Monitoring
```
## Specification
Make note of the kinds of variables that affect the feature:
1. User input?
2. Time?
3. Other protocols?
4. Existing state?

## Evaluation
Make dedicated time to evaluate:
1. How long will this realistically take? Without specification most estimates will be wrong.
2. Is the specification overly complex? Complexity leads to bugs and worse overall code
3. What are the risks associated with this feature? Spend ample time evaluating this. Consider every module the feature may touch. Then go back through excluded modules and *ensure* they cannot be affected.

You may have millions of dollars at risk already, or will after launch. As such, you **have** to consider how each feature may put *all* of it at risk. The cost of a misstep will likely be in the millions of dollars.
## Implementation
1. Draft a PR
  - Include the feature request + specification
2. Write an initial implementation
  - Get an initial implementation in place
  - Document all functions' intended behavior ([NatSpec](https://docs.soliditylang.org/en/develop/natspec-format.html) for public/external functions) and add inline documentation/comments
  - Any use of `unchecked` should include `safety` documentation like:
    ```solidity
    // Safety:
    //  1. a + b: a and b are uint64s, and a is casted up to uint128, so a uint128(uint64) + uint64
    //     cannot overflow
    //  2. c * 2: c is casted up to uint256, so a uint256(uint128) * 2 cannot overflow
    uint256 d;
    unchecked {
      uint128 c = uint128(a) + b;
      d = uint256(c) * 2;
    }
    ```
  - **Every** line of assembly is documented/commented on

## Testing

3. Write initial concrete [tests](https://book.getfoundry.sh/forge/tests.html)
  - Write an initial test for the implementation. Each write to storage should be checked, each revert should be checked. Line-by-line of the implementation, look for storage writes
  - For non-state changing functions, good practice is have a test contract that inherits the contract that does the math
  - Use coverage tools (like Foundry's [coverage tool](https://github.com/foundry-rs/foundry/pull/1576)) to see how well your tests cover your code
4. Cleanup the implementation
5. Improve tests, then move back to step 4 based on any found bugs
6. Write [fuzz tests](https://book.getfoundry.sh/forge/fuzz-testing.html)
  - Now that you are pretty confident in your implementation, throw a monkey at it. A good fuzz test should consider all valid inputs, and include as many state transition assertions as possible (think: is this function monotonically in/decreasing, should it be always less than something else,  etc.)
7. Move back to step 4 if any bugs are found
8. Write integration tests
  - Your feature now likely does exactly what you think it does. In a complex system, that is not enough. Ideally you have tested how it affects the entire system as well. Invariant tests are coming soon to [Foundry](https://github.com/foundry-rs/foundry), which should help integration style tests, but make do with what you can with fuzz tests on a broader basis (not just for a single function)
9. Move back to step 4 if any bugs are found
10. Run [slither](https://github.com/crytic/slither)
  - Analyze the output and return to step 4 if needed
11. Cleanup documentation
12. PR review
  - The implementer is just the first line of defense. If you are a reviewer, confirm that the implementer followed the above principals (test-per-state-transition, test-per-revert, fuzz test, and integration test)
  - Review the documentation and ensure the implementation matches the documented behavior. If it does not, touch base with the implementer and confirm which needs to be updated.

## Deployment
13. Write a deployment script
  - Write the deployment script
  - Most likely Foundry has scripting ready when you are reading this. Check out this [PR](https://github.com/foundry-rs/foundry/pull/1208)
14. Write a deployment test
  - Ensure deployment goes exactly as planned by writing a test testing *every state transition* and make sure no changes unexpectedly happen. One way to accomplish this is the `record` cheatcode. If performing an upgrade to an existing protocol, create a list of your entire protocol's addresses, call record, perform the upgrade. Then, call `accesses` for each address of your protocol. Ensure there are no slots/addresses that unexpectedly changed.
15. Have an audit performed
  - Consider if this feature/contract needs an audit. Always lean towards being safe rather than sorry.
  - Have the complexity & code size inform if and who should audit your contracts
  - At Nascent, we have an internal tier list of quality of auditors. You should check with other developers about who are good auditors and who aren't. Some auditors are just there to check a box for a protocol, others actually care about finding vulnerabilities.
16. Implement audit fixes
17. Setup monitoring service
  - Have an internal tool that monitors important aspects of your system.
  - Use tools like [Check the Chain](https://github.com/fei-protocol/checkthechain) + [Grafana](https://grafana.com/), or use an off-the-shelf monitoring tool like [Tenderly](https://tenderly.co/alerting) or OpenZeppelin's [Defender Sentinels](https://www.openzeppelin.com/defender).
18. Prepare/update your [Incident Response Plan](incident-response-plan-template.md)
19. Deploy the contract(s)
  - Congrats, you probably just crushed 99% of Solidity devs in terms of a secure development + deployment

## Monitoring
20. Monitor next couple hours
  - Use the monitoring service you set up to watch carefully for unexpected behaviors and be ready to take action
21. Relax, have a beer, you earned it.
