# State Growth Economics Calculator - Specification

## Overview

A single-page web application that compares the economics of state storage between Ethereum (with EIP-8037 pricing) and Monad. Users can edit parameters and see real-time calculations of fee revenue vs infrastructure costs.

## Architecture

- **Single HTML file** with embedded CSS and JavaScript
- **Vanilla JavaScript** - no frameworks or dependencies
- **Local file hosting** - must work when opened directly in browser
- **Reactive updates** - calculations update immediately when any input changes

## Visual Design

### Color Palette

| Purpose | Color |
|---------|-------|
| Background | `#0E091C` |
| Foreground text | `#DDD7FE` |
| Muted text (labels) | `#9990B8` |
| Card background | `#1A1329` |
| Highlight 1 (Ethereum accent) | `#85E6FF` (cyan) |
| Highlight 2 (Monad accent) | `#FF8EE4` (pink) |
| Highlight 3 (Shared accent) | `#FFAE45` (orange) |

### Design Style: Glowing Cards

- Output boxes: dark cards (`#1A1329`) with subtle glow/box-shadow
- Each chain has its own accent color for glow effects:
  - Ethereum: `#85E6FF` (cyan glow)
  - Monad: `#FF8EE4` (pink glow)
- Output values: 56px bold font, colored per chain accent
- Inputs: filled dark background (`#1A1329`), rounded corners
- Shared parameters section: bordered box with `#FFAE45` (orange) accent

## Layout Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    State Growth Economics                           │
├────────────────────────────┬────────────────────────────────────────┤
│        ETHEREUM            │              MONAD                     │
├────────────────────────────┼────────────────────────────────────────┤
│ ┌────────────────────────┐ │ ┌──────────────────────────────────┐  │
│ │ FEE REVENUE/YEAR       │ │ │ FEE REVENUE/YEAR                 │  │
│ │ $XXX,XXX,XXX           │ │ │ $XXX,XXX,XXX                     │  │
│ ├────────────────────────┤ │ ├──────────────────────────────────┤  │
│ │ LIFETIME INFRA COST    │ │ │ LIFETIME INFRA COST              │  │
│ │ $XXX,XXX               │ │ │ $XXX,XXX                         │  │
│ └────────────────────────┘ │ └──────────────────────────────────┘  │
├────────────────────────────┼────────────────────────────────────────┤
│ Token Price:    [$____]    │ Token Price:    [$____]                │
│ Base Fee:       [____]gwei │ Base Fee:       [____] gwei            │
│ Storage Gas:    [____]     │ Storage Gas:    [____]                 │
│ Block Size:     [____]M    │ Block Size:     [____]M                │
│ Block Time:     [____]s    │ Block Time:     [____]ms               │
│ State Write %:  [____]%    │ State Write %:  [____]%                │
│ Nodes:          [____]     │ Nodes:          [____]                 │
├─────────────────────────────────────────────────────────────────────┤
│  SHARED PARAMETERS                                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Drive Cost: [____]/TB   Lifetime Multiplier: [____]x        │   │
│  │ Disk Overhead: [____] bytes/slot                            │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

- Outputs are **prominent** at the top of each chain column
- Chain parameters are in the middle, side by side
- Shared parameters are at the bottom
- Labels on left, input fields on right

## Default Values

### Shared Parameters

| Parameter | Default | Unit |
|-----------|---------|------|
| Drive Cost | 120 | $/TB |
| Lifetime Multiplier | 5 | x |
| Disk Overhead | 128 | bytes/slot |

### Ethereum Parameters

| Parameter | Default | Unit |
|-----------|---------|------|
| Token Price | 3200 | USD |
| Base Fee | 0.5 | gwei |
| Storage Gas | 23520 | gas/slot |
| Block Size | 60 | M gas |
| Block Time | 12 | seconds |
| State Write % | 50 | % |
| Nodes | 10000 | nodes |

### Monad Parameters

| Parameter | Default | Unit |
|-----------|---------|------|
| Token Price | 0.03 | USD |
| Base Fee | 100 | gwei |
| Storage Gas | 120000 | gas/slot |
| Block Size | 200 | M gas |
| Block Time | 0.4 | seconds (400ms) |
| State Write % | 50 | % |
| Nodes | 1000 | nodes |

## Calculations

All calculations should update in real-time when any input changes.

### Derived Constants

```
SECONDS_PER_YEAR = 365 * 24 * 60 * 60 = 31,536,000
```

### Per-Chain Calculations

#### Step 1: Blocks Per Year

```
blocks_per_year = SECONDS_PER_YEAR / block_time_seconds
```

#### Step 2: Total Gas Capacity Per Year

```
gas_per_year = block_size_gas * blocks_per_year
```

#### Step 3: Storage Gas Per Year

```
storage_gas_per_year = gas_per_year * (state_write_percent / 100)
```

#### Step 4: New Storage Slots Per Year

```
slots_per_year = storage_gas_per_year / storage_gas_per_slot
```

#### Step 5: Fee Revenue Per Year

Convert gas to native token, then to USD:

```
# Gas to native token (ETH or MON)
native_token_cost = storage_gas_per_year * base_fee_gwei * 1e-9

# Native token to USD
fee_revenue_per_year = native_token_cost * token_price_usd
```

#### Step 6: State Growth in Bytes Per Year

```
state_growth_bytes = slots_per_year * disk_overhead_bytes_per_slot
```

#### Step 7: State Growth in TB Per Year

```
state_growth_tb = state_growth_bytes / 1e12
```

#### Step 8: Infrastructure Cost Per Year (Single Node)

```
infra_cost_per_node = state_growth_tb * drive_cost_per_tb
```

#### Step 9: Lifetime Infrastructure Cost (All Nodes)

```
lifetime_infra_cost = infra_cost_per_node * num_nodes * lifetime_multiplier
```

### Summary of Output Formulas

**Fee Revenue Per Year:**
```
fee_revenue = (block_size * blocks_per_year * state_write_pct * base_fee_gwei * 1e-9) * token_price
```

**Lifetime Infrastructure Cost:**
```
lifetime_cost = ((slots_per_year * disk_overhead) / 1e12) * drive_cost * nodes * lifetime_multiplier
```

Where:
```
slots_per_year = (block_size * (SECONDS_PER_YEAR / block_time) * state_write_pct) / storage_gas
```

## Input Validation

- All numeric inputs should accept decimal values
- Prevent negative values
- Block time for Monad should handle sub-second values (400ms = 0.4s)
- Large numbers should be formatted with commas in output

## Output Formatting

- Dollar amounts: `$XXX,XXX,XXX.XX` (2 decimal places, comma separators)
- Large values over $1B should show as `$X.XX B`
- Large values over $1M should show as `$X.XX M`
- Values under $1M show full number with commas

## Implementation Notes

1. Use `input` event listeners for real-time updates (not `change`)
2. Store all parameters in a JavaScript object for easy access
3. Create a single `recalculate()` function that updates all outputs
4. Use CSS Grid or Flexbox for the two-column layout
5. Add `box-shadow` with chain accent colors for the glow effect
6. Inputs should have `:focus` states using accent colors

## File Structure

Single file: `index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>State Growth Economics</title>
    <style>
        /* All CSS here */
    </style>
</head>
<body>
    <!-- All HTML here -->
    <script>
        /* All JavaScript here */
    </script>
</body>
</html>
```
