# State

The `x/stdreference` module keeps state of one object, the Ref. Ref is a raw data from BandChain, which has a requestID inside it. The requestID will allow anyone to check that the price data actually exists in the BandChain. The state is basically a mapping from string(symbol) to a struct Ref.

```go
type Ref struct {
    Rate        sdk.Int
    ResolveTime time.Time
    RequestID   uint64
}
```

- Ref: `0x0 | byte(string length) | []byte(string) -> ProtocolBuffer(type Ref struct {\n Rate sdk.Int \n ResolveTime sdk.Int \n RequestID sdk.Int \n })`
