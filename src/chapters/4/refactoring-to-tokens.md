# Refactoring to tokens

We could continue tracking point balances in our own custom leaderboard, but let's instead refactor towards an Ethereum native design pattern for our problem: tracking points using a custom token.

We'll need to take a quick detour through Ethereum token standards, but along the way we'll see that the smart contracts powering tokens on Ethereum are not much more complicated than our existing `totalPoints` mapping!
