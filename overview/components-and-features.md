# Components & Features

## **Before Use**

This page provides a brief overview of the components and tests within Herbicide. Herbicide requires Uniswap V4’s PoolKey as an input to operate. It is recommended that the Hook Contract be deployed for analysis on the Unichain Sepolia Network (Chain ID: 1301) and initialized with the Uniswap V4 Pool Manager Contract on Unichain Sepolia.

For comprehensive static analysis, the source code must be verified and uploaded to the [UniChain Blockscout](https://unichain-sepolia.blockscout.com/) Contract. Static analysis is powered by [Semgrep](https://semgrep.dev/) and [Slither](https://github.com/crytic/slither), while dynamic analysis operates on a [Foundry](https://book.getfoundry.sh/)-based framework. Threats that may arise within Uniswap V4 Hooks are detailed in the Uniswap V4 Hook Security documentation.

## Features

### Hook Token Delta Simulation

<figure><img src="../.gitbook/assets/master_page_m -O - 6 – 1.png" alt="" width="563"><figcaption></figcaption></figure>

When using specific Hooks, the logic of `afterSwap`, `beforeSwap`, `beforeAddLiquidity`, `afterAddLiquidity`, `beforeRemoveLiquidity`, and `afterRemoveLiquidity` can impact token movements by influencing Delta Amount. Through Dynamic Analysis, we simulate the token amounts users are expected to pay or receive during swap/modifyLiquidity actions, tracking fund movements across each execution entity (Hook Contract, PoolManager, Router).

By comparing the simulation results for amountIn and amountOut, users can verify the Actual Price when using a particular Pool. Additionally, this feature enhances user convenience by displaying real-time market prices, sourced through Pyth Oracle, for reference alongside simulation results.

$$actualPrice = amountOut / amountIn$$

$$oraclePrice = (token1/usdc) / (token0/usdc)$$

$$expectedPrice = expectedAmountOut / expectedAmountIn$$

### Hook Contract Scanner

Provides simple threat detection and information extraction functions for Hook Contract code written in Solidity.

The Hook Contract Scanner helps developers easily understand Libraries, Modifiers, require/assert/revert conditions, and Access Control Logic in the Hook Contract.

<figure><img src="../.gitbook/assets/master_page_m – PF - 6.png" alt="" width="563"><figcaption></figcaption></figure>











