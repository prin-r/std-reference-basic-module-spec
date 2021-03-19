# `stdreference`

## Abstract

This specification specifies the stdreference module, which will behave like an oracle that other modules can ask for price data.

The module enables Cosmos-SDK based blockchain to support the Band oracle user. In this module, there is a group of accounts who have the right to save and update price data within the module. These groups of relayer accounts are determined by the gov parameters of the respective chain.

So there is two importance things

- The relayer accounts will be stored in paramSpace, so it can only be change via the proposal.
- The price data will be stored in the state of stdreference module in a mapping like scheme ( symbol (string) -> price data (Ref) ). And it can only be set by an account who is the member of relayers.

## Contents

1. **[State](01_state.md)**
2. **[Keeper](02_keeper.md)**
   - [Types](02_keeper.md#Types)
   - [Keeper](02_keeper.md#Keeper)
3. **[Messages](03_messages.md)**
4. **[Events](04_events.md)**
5. **[Params](05_params.md)**
