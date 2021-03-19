# Messages

## MsgRelay

### struct

```go
type MsgRelay struct {
    Relayer      sdk.AccAddress
	Symbols      []string
	Rates        []sdk.Int
    ResolveTimes []time.Time
    RequestIDs   []uint64
}
```

### handleMsgRelay

```go
func handleMsgRelay(ctx sdk.Context, k Keeper, m MsgRelay) (*sdk.Result, error) {
	if !k.IsRelayer(ctx, m.Relayer) {
		return nil, types.ErrRelayerNotAuthorized
	}

    // check that the sizes of symbols,rates,resolveTimes,requestIDs are equal
    symbolsCount := len(m.Symbols)
    if len(m.Rates) !== symbolsCount {
        return nil, types.ErrBadRatesCount
    }
    if len(m.ResolveTimes) !== symbolsCount || {
        return nil, types.ErrBadResolveTimesCount
    }
    if len(m.RequestIDs) !== symbolsCount {
        return nil, types.ErrBadRequestIDsCount
    }

    // create slice of ref objects
    refs := make([]types.Ref, symbolsCount)
    for i := 0; i < symbolsCount; i++ {
        refs[i] = types.Ref.NewRef(m.Rates[i], m.ResolveTimes[i], m.RequestIDs[i])
    }

    // save all ref to the state
	err := k.Relay(ctx, m.Symbols, refs)
	if err != nil {
		return nil, err
	}

	ctx.EventManager().EmitEvent(sdk.NewEvent(
		types.EventTypeRelay,
        sdk.NewAttribute(types.AttributeKeyRelayer, fmt.Sprintf("%s", m.Relayer)),
		sdk.NewAttribute(types.AttributeKeySymbols, fmt.Sprintf("%v", m.Symbols)),
        sdk.NewAttribute(types.AttributeKeyRates, fmt.Sprintf("%v", m.Rates)),
        sdk.NewAttribute(types.AttributeKeyResolveTimes, fmt.Sprintf("%v", m.ResolveTimes)),
        sdk.NewAttribute(types.AttributeKeyRequestIDs, fmt.Sprintf("%v", m.RequestIDs)),
	))
	return &sdk.Result{Events: ctx.EventManager().Events()}, nil
}
```
