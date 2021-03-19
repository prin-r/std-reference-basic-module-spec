# Keepers

- The stdreference module provides three different exported keeper interfaces which can be passed to other modules which need to read price data. The price data should only be updated by the relayer.

## Types

### Ref

A raw data from BandChain, which has a requestID inside it. The requestID will allow anyone to check that the price data actually exists in the BandChain.

```go
type Ref struct {
    Rate        sdk.Int
    ResolveTime time.Time
    RequestID   uint64
}
```

### ReferenceData

An output that derived from the raw data or Ref. The Rate of ReferenceData will calculated by dividing of 2 Rates from the 2 Refs (base and quote) and then keep the timestamp of them but cut out the reqeustID.

```go
type ReferenceData struct {
    Rate            sdk.Int
    LastUpdateBase  time.Time
    LastUpdateQuote time.Time
}
```

For example

```go
// We use normal int in this example just for simplicity.
// Normally we would use sdk.Int for Rate and time.Time for any time argument.
base := Ref {Rate: 2, ResolveTime: 4, RequestID: 6}
quote := Ref {Rate: 1, ResolveTime: 3, RequestID: 5}

referenceData := ReferenceData {Rate: base.Rate * int(1e9) / quote.Rate, LastUpdateBase: base.ResolveTime, LastUpdateQuote: quote.ResolveTime}

fmt.Println(base, quote, referenceData)

// An outout should be "{2 4 6} {1 3 5} {2 4 3}"
```

## Keeper

```go
// Keeper defines a module interface that facilitates the transfer of coins
// between accounts.
type Keeper interface {
	storeKey         sdk.StoreKey
	cdc              *codec.Codec
	paramSpace       params.Subspace

    // public
    GetRef(ctx sdk.Context, symbol string) types.Ref
    // internal, only be called by Relay function
	SetRef(ctx sdk.Context, symbol string, refData types.Ref)

    //public
    GetReferenceData(ctx sdk.Context, base string, quote string) types.ReferenceData
    GetReferenceDataBulk(ctx sdk.Context, bases []string, quotes []string) []types.ReferenceData

    // public, only relayer
    Relay(ctx sdk.Context, symbols []string, refDataSlice []types.Ref)

    // public
    GetParams(ctx sdk.Context) types.Params
    // can only be set by a proposal
	SetParams(ctx sdk.Context, params types.Params)

    // public, check that the relayer has been authorized
    IsRelayer(ctx sdk.Context, relayer sdk.AccAddress) bool
}
```

### Example Implementation

#### GetRef

```go
func (k Keeper) GetRef(ctx sdk.Context, symbol string) types.Ref {
	var ref Ref
	bz := ctx.KVStore(k.storeKey).Get(types.GetRefStoreKey(symbol))
	k.cdc.MustUnmarshalBinaryLengthPrefixed(bz, &ref)
	return ref
}
```

#### SetRef

```go
func (k Keeper) SetRef(ctx sdk.Context, symbol string, refData types.Ref) {
	bz := k.cdc.MustMarshalBinaryLengthPrefixed(refData)
	ctx.KVStore(k.storeKey).Set(types.GetRefStoreKey(symbol), bz)
}
```

#### GetReferenceData

```go
func (k Keeper) GetReferenceData(ctx sdk.Context, base string, quote string) types.ReferenceData {
	baseRef := k.GetRef(ctx,base)
    quoteRef := k.GetRef(ctx,quote)
	return ReferenceData {
        Rate: baseRef.Rate.Mul(sdk.Int.New(1000000000)).Quo(quoteRef.Rate)
        LastUpdateBase: baseRef.ResolveTime,
        LastUpdateQuote: quote.ResolveTime,
    }
}
```

#### GetReferenceDataBulk

```go
func (k Keeper) GetReferenceDataBulk(ctx sdk.Context, bases []string, quotes []string) []types.ReferenceData {
    referenceDataSlice := []ReferenceData{}
    if len(bases) == len(quotes) {
        for i := 0; i < len(bases); i++ {
            referenceDataSlice = append(referenceDataSlice, k.GetReferenceData(ctx,bases[i],quotes[i]))
        }
    }
    return referenceDataSlice
}
```

#### Relay

```go
func (k Keeper) Relay(ctx sdk.Context, symbols []string, refDataSlice []types.Ref) {
    for i := 0; i < len(symbols); i++ {
        k.SetRef(ctx, symbols[i], refDataSlice[i])
    }
}
```

#### GetParams

```go
func (k Keeper) GetParams(ctx sdk.Context) (params types.Params) {
	k.paramSpace.GetParamSet(ctx, &params)
	return params
}
```

#### SetParams

```go
func (k Keeper) SetParams(ctx sdk.Context, params types.Params) {
	k.paramstore.SetParamSet(ctx, &params)
}
```

#### IsRelayer

```go
func (k Keeper) IsRelayer(ctx sdk.Context, relayer sdk.AccAddress) bool
	params := k.GetParams(ctx)
    for _, existedRelayer := range params.Relayers {
        if relayer == existedRelayer {
            return true
        }
    }
    return false
}
```
