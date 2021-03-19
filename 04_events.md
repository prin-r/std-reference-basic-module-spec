# Events

The stdreference module emits the following events:

## Types

```go
const (
    // event types
    EventTypeRelay = "relay"

    // attribute types
    AttributeKeyRelayer = "relayer"
    AttributeKeySymbols = "symbols"
    AttributeKeyRates = "rates"
    AttributeKeyResolveTimes = "resolve_times"
    AttributeKeyRequestIDs = "request_ids"
)
```

## Service Messages

### Msg/Relay

| Attribute Key            | Attribute Value                     |
| ------------------------ | ----------------------------------- |
| AttributeKeyRelayer      | Address of the relayer              |
| AttributeKeySymbols      | slice of symbols ([]string)         |
| AttributeKeyRates        | slice of rates ([]sdk.Int)          |
| AttributeKeyResolveTimes | slice of resolveTimes ([]time.Time) |
| AttributeKeyRequestIDs   | slice of requestIDs ([]uint64)      |
