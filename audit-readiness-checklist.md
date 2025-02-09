# Audit Readiness Checklist

### Bare minimum quality checklist

- [x]  Use the [latest](https://docs.soliditylang.org/en/latest/) major version of Solidity.
- [x]  Use known/established libraries where possible. [OpenZeppelin contracts](https://github.com/OpenZeppelin/openzeppelin-contracts/) are preferred because they prioritize security above all else, and most auditors are already familiar with them. [Solmate contracts](https://github.com/Rari-Capital/solmate) can be a good alternative for functions where gas optimization is paramount.
- [x]  Contracts compile without any errors or warnings from the compiler.
- [x]  Document all functions. Use [NatSpec documentation](https://docs.soliditylang.org/en/develop/natspec-format.html) for `public`/`external` functions. Consider this part of the public interface of the contract.
- [x]  Any `public` function that can be made `external` should be made `external`. This is not just a gas consideration, but it also reduces the cognitive overhead for auditors because it reduces the number of possible contexts in which the function can be called.
- [x]  Have tests for all "happy path" user stories. All tests should be passing.
- [x]  Use the [Checks-Effects-Interactions pattern](https://docs.soliditylang.org/en/v0.8.13/security-considerations.html#use-the-checks-effects-interactions-pattern) everywhere possible. Otherwise use reentrancy guards. Treat all token and ETH transfers as 'interactions'.
- [x]  Avoid using assembly as much as possible. Use of assembly increases audit times because it throws away Solidity's guardrails and must be checked much more carefully.
- [x]  Document use of `unchecked`. Concretely describe *why* it is safe to not perform arithmetic checks on each code block. Preferably for each operation.
- [x]  Run the code through a spellchecker.
- [x]  Run a static analysis tool ([Slither](https://github.com/crytic/slither) preferred, [MythX](https://mythx.io/) is an alternative) on your code and consider what it tells you. They very often raise flags for non-issues, but they can sometimes catch low-hanging fruit, so it's worth doing.
- [x]  Have at least one trusted Solidity dev or security person outside your organization sanity check your contracts. If your code is a tire fire and in need of major changes, you want to hear about that early and from a trusted friend (for free) rather than after an expensive audit.

### Nice to haves

- [ ]  If you are using [Foundry](https://github.com/foundry-rs/foundry), **strongly** recommend using fuzz tests. There is low additional overhead and highest likelihood of finding bugs using fuzzing. If you are using hardhat, consider adding Foundry for this functionality.
- [x]  Write negative tests. E.g., if users should NOT be able to withdraw within 100 blocks of depositing, then write a test where a users tries to withdraw early and make sure the user's attempt fails.
- [ ]  You can also try to use formal verification tools to verify the invariants, but be aware that -- in practice -- current formal verification tools are rarely useful for anything non-trivial.
- [x]  Write down your security assumptions. This doesn't have to be super formal. E.g., "We assume that the `owner` is not malicious, that the Chainlink oracles won't lie about the token price, that the Chainlink oracles will always report the price at least once every 24 hours, that all tokens that the `owner` approves are ERC20-compliant tokens with no transfer hooks, and that there will never be a chain reorg of more than 30 blocks". This helps you understand how things could possibly go wrong *even if your contracts are bug-free*. Auditors may also be able to tell you whether or not your assumptions are realistic. They may also be able point out assumptions you're making that you didn't realize you were making.
- [x]  If you're unsure about something in your own code, or there are areas where you'd like auditors to spend more time, make a list of these to share with the auditors.
