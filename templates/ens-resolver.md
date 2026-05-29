---
title: ENS Resolver
app_type: ens-resolver
wallet: 0x636e39c31aD86E29F3201EC015D4C32CC93c85F7
---

Type a name like vitalik.eth or an address like 0xd8da6bf26964af9d7eed9e03e53415d37aa96045 into one input. The page resolves the name to an address, the address to a name, fetches the avatar text record, decodes the contenthash if one is set, and prints every other text record it finds. The whole resolver runs in the page through plain json rpc calls to a public ethereum node, no key, no library, no api beyond eth_call. The page does not write anything to chain.

### surface

A midnight blue background, the kind of dark that lets a small accent stay loud. Soft white for body type. Two accents only, ens blue for the resolved name and address, accent magenta for any error line and for the small chevron that opens the records detail. Geist at 18px 400 for body and at 32px 600 for the central name display. Geist Mono at 14px for hex addresses and the small rpc readout. Two typefaces, two roles, no overlap. The page is one column at most 720px wide, centered, 32px gutter on phones.

### lookup

A single text input at the top of the page, 16px Geist soft white on a slightly lighter midnight band with a 1px soft white hairline at fifteen percent opacity, padded 14px. The placeholder reads vitalik.eth or 0xd8da... The input accepts both forms, the page detects which through a small heuristic (a 0x prefix and 42 hex characters means address, otherwise name).

To the right of the input, a single text button in 14px Geist 500 ens blue reading look up. Tapping it (or pressing enter inside the input) starts the lookup. While the lookup is in flight, a small spinner glyph animates next to the button at 1200ms cycle. The input is dimmed during the lookup but the visitor can still type a new query (the page cancels the in flight rpc through an AbortController).

### result

Below the input, the result region. Three blocks stacked vertically.

The first block shows the central name in 32px Geist 600 ens blue, with a small text button next to it reading copy in 12px Geist soft white muted. The address (or the resolved address for a name lookup) sits below the name in 14px Geist Mono soft white, with another small copy button. If the visitor entered an address that has no ENS name set, the first block shows the address as the central display and a small italic 14px line below it reads no ens name set for this address.

The second block shows the avatar. The text record for the avatar key is read from the public resolver, the value is parsed (it can be a regular https url or an eip 155 url like eip155:1/erc721:0x.../1234 for an nft), and the page renders the resolved image in a small 96 by 96 round mask in the top right of the result region. If the avatar text record is not set, this block is omitted.

The third block shows the rest of the text records. The page reads a fixed list of well known keys (name, description, url, email, com.twitter, com.github, org.telegram, com.discord, location), and renders any that are set as a small two column list in 14px Geist soft white. Each row has the key in soft white muted on the left and the value in soft white on the right, with the value rendered as a clickable link when it is a url. Below the well known list, a small text button reads show every record found, which expands the block to include any additional text records the resolver returned.

### contenthash

If the contenthash record is set, a small block below the records shows the decoded value. The encoding is one of ipfs, ipns, swarm, or onion, the page detects the prefix bytes and decodes accordingly. The decoded line is rendered in 13px Geist Mono soft white with a small text button reading open this site that opens the decoded url in a new tab through the visitor's chosen ipfs gateway (a small dropdown lets the visitor pick from cloudflare ipfs, dweb link, or a custom gateway).

### namehash

The namehash of an ens name is the keccak256 hash of a recursive structure that reduces the name to a single 32 byte value. The page computes it in source through a small keccak256 implementation (around 180 lines of TypeScript in src/lib/keccak.ts), since the Web Crypto api does not expose keccak. The namehash function below.

```
function namehash (name: string): Uint8Array {
  let node = new Uint8Array(32);  // 32 zero bytes for the root
  if (name) {
    const labels = name.split('.').reverse();
    for (const label of labels) {
      const labelHash = keccak256(textEncoder.encode(label));
      node = keccak256(concat(node, labelHash));
    }
  }
  return node;
}
```

The result is a 32 byte node that the page passes to the resolver's addr, name, text, and contenthash functions through eth_call.

### the rpc

The page reads from a public json rpc endpoint, defaulting to one of the well known free public rpcs. The visitor can override the url through a small settings input. The page polls the chain on demand only, no continuous polling.

The four eth_call invocations the page makes are the standard ENS reads. The selector and the calldata are constructed in source from the abi function signatures.

```
{
  jsonrpc: '2.0',
  id:      1,
  method:  'eth_call',
  params:  [
    {
      to:   '0x00000000000C2E074eC69A0dFb2997BA6C7d2e1e',  // ENS Registry
      data: '0x0178b8bf' + nodeHex                            // resolver(node)
    },
    'latest'
  ]
}
```

Then a second eth_call against the resolved resolver address with the addr selector (0x3b3b57de plus the node), a third with the text selector (0x59d1d43c plus the node plus the encoded key string), and a fourth with the contenthash selector (0xbc1c58d1 plus the node).

The reverse lookup (address to name) calls the reverse registrar's namehash for `<address>.addr.reverse` and reads the name text record from the resolver, then checks the forward resolution matches the input address (the standard ENS reverse safety check).

### history

The last 20 lookups are remembered in localStorage under the key ens.resolver.history.v1. A small panel below the result region shows the history as a vertical list, each row showing the input the visitor typed, the resolved name or address, and a small remove glyph. Tapping a row re runs the lookup. The list rotates oldest out as new lookups come in. A small text button below the list reads forget every lookup, with a confirm.

### the chain

The page reads from ethereum mainnet by default. A small dropdown in the top right of the page offers mainnet (chain id 1) and sepolia (chain id 11155111). The ENS Registry address differs across chains, the page holds the small mapping in src/lib/registry.ts. Switching chains clears the result region and updates the rpc url.

### edges

A name that does not exist in ENS shows a small italic 14px line in accent magenta reading no ens record found for this name. The history still records the attempt.

A name with a valid forward resolution but no reverse record (the address has not set their primary name) shows the resolved address in the central block but the small italic line reads the address has not set a primary name yet.

A reverse record that fails the forward resolution safety check (the name resolves back to a different address) shows a small italic accent magenta line reading the reverse record does not match, the address may have spoofed it. The page does not show the unverified name as the central display.

A rate limited rpc shows a small italic 11px line at the bottom reading the rpc is rate limited, slowing the next lookup. The next successful call clears the line.

### build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding midnight, soft white, soft white muted, ens blue, accent magenta. Vite as the build tool. State is plain useState and one useReducer for the lookup state machine and the history. No router, no global store, no context provider, no ethers, no viem, no ens library, no icon pack. fetch with AbortController and the small in source keccak256 are the only platform features used directly.

Files. index.html with the Geist and Geist Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the input value, the result, the history, and the chain. src/components/Lookup.tsx for the input row. src/components/Result.tsx for the central block. src/components/Avatar.tsx for the round mask. src/components/Records.tsx for the text records list. src/components/Contenthash.tsx for the decoded url block. src/components/History.tsx for the side list. src/components/Settings.tsx for the rpc and chain dropdown. src/lib/keccak.ts for the in source keccak256. src/lib/namehash.ts for the recursive namehash. src/lib/abi.ts for the resolver function selectors and the calldata builders. src/lib/rpc.ts for the json rpc fetch wrapper. src/lib/contenthash.ts for the multicodec decode (ipfs, ipns, swarm, onion). src/lib/registry.ts for the ENS Registry addresses per chain. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop. The midnight surface paints, the input sits empty at the top, the placeholder reads vitalik.eth or 0xd8da... They type vitalik.eth, press enter, the spinner runs for a moment, the central block fills with vitalik.eth in ens blue and the matching address in mono below. The avatar mask shows the resolved image in the corner. The records list shows the description, url, com.twitter, and com.github. The contenthash block decodes an ipfs url. They paste an address, the page returns vitalik.eth as the reverse name. They reload the page, the history of lookups waits for them.
