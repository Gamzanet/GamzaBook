# Vulnerable Hook Detection

## Vulnerable Hook Detection

### StopLoss

***

The StopLoss contract leverages Uniswap V4’s custom user hook functionality to implement a stop loss feature. This hook automatically sells tokens when the price in a specified pool meets the user-defined stop-loss conditions, thereby preventing further losses. The functionality of this hook is summarized as follows:

1. Users create a stop-loss order by executing the `placeStopLoss()` function.
2. After each swap, the `afterSwap` function compares the previous and current ticks, executing the order if the conditions are met.

```json
{
    "currency0": "0x0197481B0F5237eF312a78528e79667D8b33Dcff",
    "currency1": "0xA56569Bd93dc4b9afCc871e251017dB0543920d4",
    "fee": 3000,
    "hooks": "0xA3da6f46f93C7B3090A48D7bFeC0345156ED5040",
    "tickSpacing": 60,
    "deployer":"0x4e59b44847b379578588920cA78FbF26c0B4956C"
}
```

[Code Details](https://unichain-sepolia.blockscout.com/address/0xA3da6f46f93C7B3090A48D7bFeC0345156ED5040?tab=contract)

Let's take a look at the `afterSwap` function in this hook.

```javascript

    function afterSwap(
        address,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata params,
        BalanceDelta,
        bytes calldata
    ) external override returns (bytes4, int128) {
        int24 prevTick = tickLowerLasts[key.toId()];
        (, int24 tick,,) = poolManager.getSlot0(key.toId());
        int24 currentTick = getTickLower(tick, key.tickSpacing);
        tick = prevTick;

        int256 swapAmounts;

        // fill stop losses in the opposite direction of the swap
        // avoids abuse/attack vectors
        bool stopLossZeroForOne = !params.zeroForOne;

        // TODO: test for off by one because of inequality
        if (prevTick < currentTick) {
            for (; tick < currentTick;) {
                swapAmounts = stopLossPositions[key.toId()][tick][stopLossZeroForOne];
                if (swapAmounts > 0) {
                    fillStopLoss(key, tick, stopLossZeroForOne, swapAmounts);
                }
                unchecked {
                    tick += key.tickSpacing;
                }
            }
        } else {
            for (; currentTick < tick;) {
                swapAmounts = stopLossPositions[key.toId()][tick][stopLossZeroForOne];
                if (swapAmounts > 0) {
                    fillStopLoss(key, tick, stopLossZeroForOne, swapAmounts);
                }
                unchecked {
                    tick -= key.tickSpacing;
                }
            }
        }
        return (StopLoss.afterSwap.selector, 0);
    }

```

The function lacks a modifier and any additional ACL mechanism. Through this function, the `fillStopLoss()` function can be called separately, allowing storage values to be altered. This may lead to unexpected behavior in the hook, and appropriate measures are necessary. **Herbicide** can detect these risks and alert users accordingly.

### ArrakisHook

***

This `ArrakisHook` stores information about the pool key in `beforeInitialize` right before initialization, enabling it to manage liquidity through ERC1155.

```json
{
    "currency0": "0x0197481B0F5237eF312a78528e79667D8b33Dcff",
    "currency1": "0xA56569Bd93dc4b9afCc871e251017dB0543920d4",
    "fee": 3000,
    "hooks": "0x7Ba42C294124A8037707399823D56b8a589b6080",
    "tickSpacing": 60,
    "deployer": "0x4e59b44847b379578588920cA78FbF26c0B4956C"
}
```

[Code Detail](https://unichain-sepolia.blockscout.com/address/0x7Ba42C294124A8037707399823D56b8a589b6080?tab=contract)

Let's take a look at the `beforeInitialize()`in this hook.

```javascript
    function beforeInitialize(
        address,
        PoolKey calldata poolKey_,
        uint160 sqrtPriceX96_,
        bytes calldata
    ) external override returns (bytes4) {
        poolKey = poolKey_;
        lastSqrtPriceX96 = sqrtPriceX96_;
        lastBlockNumber = block.number;
        return this.beforeInitialize.selector;
    }
```

This contract is a hook implemented to initialize information about the pool key in `beforeInitialize`. However, this hook does not implement any access control for `beforeInitialize`, nor does it take any measures regarding the number of initializations. Since the pool key is `initialized` in `beforeInitialize`, if a new initialization is performed on this hook, all existing hook assets will be locked.

Herbicide detects the storage values of such cases to ensure that users can use the hook safely.
