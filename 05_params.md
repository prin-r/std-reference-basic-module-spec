# Parameters

The stdreference module contains the following parameters:

- Paramsspace: `stdreference`

| Key      | Type             | Example                                           |
| -------- | ---------------- | ------------------------------------------------- |
| Relayers | []sdk.AccAddress | [band163wg6pa78s3cpew3xh0a70x5d4gnu7aft9gj6e,...] |

## Type

```go
// Params is the data structure that keeps the parameters of the oracle module.
type Params struct {
	Relayers []sdk.AccAddress
}
```

## Relayers

The relayers parameter is an array of account who has been authorized to set the price data for stdreference module.

```go
var (
	// KeyRelayers is store's key for Relayers Params
	KeyRelayers = []byte("Relayers")
)

// NewParams creates a new parameter configuration for the stdreference module
func NewParams(relayers []sdk.AccAddress) Params {
	return Params{
		Relayers: relayers,
	}
}

// ParamSetPairs implements the params.ParamSet interface for Params.
func (p *Params) ParamSetPairs() params.ParamSetPairs {
	return params.ParamSetPairs{
		params.NewParamSetPair(KeyRelayers, &p.Relayers, nil,
	}
}
```
