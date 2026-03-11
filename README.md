# CL-FLASH-LOANS

A standalone, dependency-free flash loan protocol implementation in pure Common Lisp.

## Overview

CL-FLASH-LOANS provides atomic flash loan execution with callback handling for DeFi applications. Flash loans enable borrowing assets without collateral, provided the borrowed amount plus fees is returned within the same transaction.

## Features

- **Uncollateralized flash loans** with atomic execution
- **Callback interface** for custom operation execution
- **Liquidity pool management** with multi-asset support
- **Fee calculation and distribution** with volume-based tiers
- **Reentrancy protection** and security controls
- **Borrowing limits** and rate limiting
- **Pure Common Lisp** - zero external dependencies

## Use Cases

- Arbitrage across DEXes and lending protocols
- Collateral swaps without unwinding positions
- Self-liquidation to avoid penalties
- Leveraged positions without upfront capital

## Installation

```lisp
;; Load with ASDF
(asdf:load-system :cl-flash-loans)
```

## Quick Start

```lisp
(use-package :cl-flash-loans)

;; Create a liquidity pool
(let ((pool (create-flash-pool :name "My Pool" :fee-rate 9)))

  ;; Add liquidity
  (add-pool-asset pool "0xTOKEN" :symbol "TKN" :liquidity 1000000000000000000)

  ;; Execute a flash loan
  (execute-flash-loan-simple
   '("0xTOKEN")                      ; Assets to borrow
   '(100000000000000000)             ; Amounts (0.1 token)
   :initiator "0xBORROWER"
   :pool pool
   :callback-fn (lambda (assets amounts premiums params)
                  ;; Your arbitrage/swap logic here
                  ;; Must return T for success
                  t)))
```

## API Reference

### Pool Management

```lisp
;; Create a pool
(create-flash-pool &key id name protocol fee-rate config)

;; Get a pool
(get-flash-pool pool-id)

;; Add asset to pool
(add-pool-asset pool address &key symbol decimals liquidity fee-rate borrow-cap)

;; Manage liquidity
(add-liquidity pool asset-address amount)
(remove-liquidity pool asset-address amount)

;; Pool state
(pause-flash-pool pool)
(unpause-flash-pool pool)
```

### Flash Loan Execution

```lisp
;; Simple execution
(execute-flash-loan-simple assets amounts &key initiator callback-fn pool)

;; Full execution with request object
(execute-flash-loan request &key pool callback-fn)

;; Simulation (no actual execution)
(simulate-flash-loan request &key pool callback-fn)

;; Create a request
(make-flash-loan-request &key initiator receiver assets amounts modes params)
```

### Callback Interface

```lisp
;; Register a callback
(register-callback address &key name abi-hash trusted verified)

;; Execute callback
(execute-callback callback callback-fn)

;; The callback function signature:
;; (lambda (assets amounts premiums initiator params) -> return-value)
```

### Fee Calculation

```lisp
;; Calculate premium for an amount
(calculate-flash-loan-premium amount &key fee-rate)

;; Calculate total premium for multiple amounts
(calculate-total-premium amounts &key fee-rate)

;; Get user's fee tier based on volume
(get-fee-tier total-volume)
```

## Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `+default-flash-fee+` | 9 | Default fee: 0.09% (9 basis points) |
| `+min-flash-fee+` | 1 | Minimum fee: 0.01% |
| `+max-flash-fee+` | 100 | Maximum fee: 1.00% |
| `+max-flash-loan-assets+` | 10 | Max assets per loan |
| `+max-reentrancy-depth+` | 3 | Max reentrancy depth |
| `+flash-loan-timeout+` | 300 | Timeout in seconds |

## Flash Loan Modes

| Mode | Constant | Description |
|------|----------|-------------|
| Standard | `+flash-mode-standard+` | Must repay within transaction |
| Stable Debt | `+flash-mode-stable-debt+` | Convert to stable rate debt |
| Variable Debt | `+flash-mode-variable-debt+` | Convert to variable rate debt |
| Collateral Swap | `+flash-mode-collateral-swap+` | Position restructuring |
| Arbitrage | `+flash-mode-arbitrage+` | Optimized for arbitrage |
| Liquidation | `+flash-mode-liquidation+` | Flash liquidation |

## Testing

```lisp
;; Run the test suite
(asdf:test-system :cl-flash-loans)

;; Or directly
(cl-flash-loans.test:run-tests)
```

## Architecture

```
cl-flash-loans/
├── cl-flash-loans.asd   ; System definition
├── package.lisp         ; Package exports
├── src/
│   ├── util.lisp        ; Constants, math utilities
│   ├── loan.lisp        ; Request/execution types and logic
│   ├── callback.lisp    ; Callback protocol
│   └── pool.lisp        ; Liquidity pool management
└── test/
    └── test-flash.lisp  ; Test suite
```

## Error Handling

The library uses the Common Lisp condition system. All errors inherit from `flash-loan-error`:

- `invalid-request-error` - Invalid request parameters
- `invalid-amount-error` - Amount out of bounds
- `deadline-exceeded-error` - Transaction deadline passed
- `callback-failed-error` - Callback returned failure
- `insufficient-liquidity-error` - Pool lacks liquidity
- `reentrancy-detected-error` - Reentrancy attack blocked
- `limit-exceeded-error` - Borrowing limit exceeded

## License

MIT License

## Attribution

Extracted from CLPIC (Common Lisp P2P Intellectual Property Chain).
