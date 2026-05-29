---
title: Gas Tracker
app_type: gas-tracker
wallet: 0x0e9088ffb66Dd16C6eDf1f3e3500d243778A6E42
---

A small page that watches the ethereum gas price in real time. The current base fee, the priority fee suggestions for slow medium and fast inclusion, a sparkline of the last hour of base fees, and a small calculator that takes a gas limit and a priority fee and shows the total cost in eth and in usd. The page reads from a public json rpc endpoint, no api key, and falls back gracefully when the rpc is rate limited.

## surface

A terminal black background. Soft off white for body type. Three accents only, eth blue for the current base fee number and the chart line, accent green for the slow inclusion row when the base fee is below the visitor's threshold, alert orange for the fast row when the network is congested. Geist Mono everywhere, weight 400 for body and 600 for the big numbers, no other typeface on the page. The layout is one column at most 720px wide, centered, 24px gutter on phones.

## now

The top of the page holds the current state in three big numbers stacked.

The first is the current base fee in gwei in 56px Geist Mono 600 eth blue, with the unit gwei in 14px Geist Mono off white muted to the right. The number updates on every new block, refreshed every 4 seconds. A small green or orange dot to the left of the number indicates the trend, green when the next pending block is expected to drop the base fee, orange when it is expected to rise. The trend is computed from the relationship between the parent block's gas used and the gas target (15 million on mainnet), per the EIP 1559 update rule.

The second is the priority fee suggestions in Geist Mono 16px, three rows for slow medium fast, each row with the priority fee number on the left and a small label on the right. The slow row's number is the 25th percentile of priority fees in the last 20 blocks, the medium is the 50th, the fast is the 75th. The labels read slow (under 5 minutes), medium (under 1 minute), fast (next block).

The third is the latest block number in 14px Geist Mono off white, with a small green dot pulsing once per second next to it to confirm the page is connected.

## chart

Below the now block, a single inline svg sparkline showing the last 60 minutes of base fees, sampled at one minute intervals. The line is drawn in eth blue 1.5px wide, with a 2px circle at the most recent point. The y axis is auto scaled to the min and max of the sample window. The x axis is implicit, oldest sample on the left, newest on the right. A faint horizontal line at the visitor's threshold is drawn in accent green at thirty percent opacity, so the visitor can see at a glance whether the current base fee is in their target range.

A small Geist Mono 11px line below the chart shows the min, max, and average for the window, like 9.4 to 47.2 gwei, average 21.7. The threshold value is editable through a small text button reading set my threshold which opens a single number input.

## calculator

Below the chart, a small calculator block. Three inputs in a single row.

The first input is the gas limit, in Geist Mono 14px on a soft black card with a 1px off white hairline at fifteen percent opacity. The placeholder reads gas limit, like 21000 (the cost of a simple eth transfer). The visitor can type any positive integer.

The second input is the priority fee in gwei, with a small dropdown above it offering the slow medium fast presets to fill it from the now block.

The third is implicit, the current base fee, shown as a read only 14px Geist Mono off white muted with a small refresh chevron next to it.

Below the inputs, the result block. Two lines.

The first line is the total cost in eth, in 22px Geist Mono 600 eth blue, computed as the gas limit times (base fee plus priority fee) in wei, then converted to eth through a single BigInt division. The number is shown to six decimal places.

The second line is the total cost in usd, in 18px Geist Mono off white. The eth to usd rate is fetched from a public price endpoint at boot and refreshed every 60 seconds. If the rate fetch fails, the line reads usd unavailable in italic 14px off white muted, the eth line still works.

```
const wei = gasLimit * (baseFee + priorityFee);  // BigInt math
const eth = Number(wei) / 1e18;
const usd = eth * ethToUsdRate;
```

## the rpc

The page reads from a public json rpc endpoint. The default url is one of the well known free public rpcs (cloudflare's eth rpc, llamanodes, ankr's free tier), the visitor can override the url through a small settings input that persists in localStorage. The page never asks for a private key, it never builds a transaction, it never sends anything to any wallet, it only reads the chain.

The page polls the rpc with three calls every 4 seconds.

```
{ method: 'eth_blockNumber',         params: [] }
{ method: 'eth_getBlockByNumber',    params: [<latest>, false] }
{ method: 'eth_feeHistory',          params: ['0x14', 'latest', [25, 50, 75]] }
```

The first returns the latest block number. The second returns the block header, which carries the base fee per gas in the baseFeePerGas field. The third returns the fee history for the last 20 blocks at the 25, 50, and 75 percentiles, which the page uses to build the priority fee suggestions.

A small Geist Mono 11px line at the bottom of the page shows the rpc endpoint url and the response time of the latest poll. Pressing the refresh chevron next to the base fee value forces an immediate poll.

## the chain

The page defaults to ethereum mainnet (chain id 1). A small dropdown in the top right of the page offers a small list of common chains: mainnet, base, optimism, arbitrum, polygon. Switching the chain swaps the rpc url and clears the history buffer. Each chain holds its own threshold value in localStorage so the visitor's targets are remembered per chain.

The price endpoint switches with the chain (price feeds for matic on polygon, eth on the eth chains).

## save

The visitor's chain choice, threshold value, custom rpc url override, and history buffer for the current chain are saved to localStorage under the key gas.tracker.v1. The history buffer is bounded at 60 samples (one hour at one minute resolution), older samples drop off when a new one comes in. The threshold and rpc url persist across sessions, the buffer is rehydrated on boot so the chart paints immediately even before the first network response.

If localStorage is unavailable, the page holds in memory and a small italic 11px line at the bottom reads this history is not being kept on this device.

## edges

A rate limited rpc returns a 429 response, the page catches it, switches to a 30 second poll interval, and shows a small italic 11px line at the bottom reading the rpc is rate limited, slowing the polls. The next successful poll restores the 4 second interval.

A network outage that prevents any rpc response shows a small line in italic 11px alert orange at the top of the page reading the rpc is unreachable, last refresh was 18 seconds ago, with the elapsed time updating every second. The page never blanks, the most recent values stay visible.

A chain that does not support EIP 1559 (legacy chains, some sidechains) shows the priority fee section as a single legacy gas price line instead, the calculator falls back to the legacy formula gas limit times gas price.

The eth to usd rate failing to fetch leaves the eth line working, the usd line shows usd unavailable.

## build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding terminal black, off white, off white muted, eth blue, accent green, alert orange. Vite as the build tool. State is plain useState and one useReducer for the polling state, the history buffer, and the calculator inputs. No router, no global store, no context provider, no ethers, no viem, no web3, no chain library, no chart library, no icon pack. fetch, BigInt, Intl.NumberFormat are used directly.

Files. index.html with the Geist Mono link in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the chain, the now state, the history buffer, and the calculator inputs. src/components/Now.tsx for the three big readouts. src/components/Chart.tsx for the inline svg sparkline. src/components/Calculator.tsx for the three input row and the two result lines. src/components/Settings.tsx for the chain dropdown, threshold input, and rpc override. src/lib/rpc.ts for the json rpc fetch wrapper and the polling loop. src/lib/fee.ts for the base fee trend rule and the percentile math. src/lib/price.ts for the eth to usd fetch. src/lib/storage.ts for the localStorage. src/lib/chains.ts for the small list of chains and their default rpc urls. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop. The terminal black surface paints, three big eth blue numbers settle into place at the top showing 14.2 gwei, the slow medium fast rows fill in just below, the latest block number ticks up. The sparkline chart paints with the last 60 minutes of base fee history. They type 21000 into the gas limit, click the medium preset to fill the priority fee, the calculator block shows the total cost in eth and in usd. They switch the chain dropdown to base, the rpc swaps, the readouts show base mainnet's much lower base fee. They reload the page, the same chain and threshold come back.
