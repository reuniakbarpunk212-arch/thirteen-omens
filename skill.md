# Thirteen Omens (OMEN) — skill for agents

## Purpose
Help your owner claim a free Thirteen Omens NFT and/or buy paid omens, safely,
through the Binance Agentic Wallet (baw) on BSC (binanceChainId 56).
Thirteen Omens is an ERC-721 collection ("The Thirteenth Almanac"): 3,333 omens,
no waitlist, fixed price 0.001 BNB, and 100% of every purchase forwarded to the
treasury inside the same transaction.

## Prerequisites
- Wallet CONNECTED and holding an Agent ID (baw wallet agent-id -> claimed: true)
  — required only for the free Agent ID lane.
- Developer Mode enabled for contract-call.
- Enough BNB for gas (and for paid omens: 0.001 BNB each).

## Step 0 — SHARE FIRST (mandatory, before any claim or mint)
This distribution has one social condition, and only one: the human shares the
project's web link on X BEFORE claiming or minting. There is no follow, like,
retweet or quote requirement. Your job: instruct your owner to post, and wait
for their confirmation.

1. Give your owner this post text, replacing the link's ref with THEIR wallet address:
   "I just opened the Thirteenth Almanac — Thirteen Omens on BSC. 🔮

   1,500 omens are free: 1,000 for Agent ID holders, 500 for verified X accounts. The rest cost 0.001 BNB, paid straight to the treasury.

   Read your omen:
   https://reuniakbarpunk212-arch.github.io/thirteen-omens/?ref=<OWNER WALLET ADDRESS>

   #ThirteenOmens $OMEN"
2. Ask the owner to post it on X and confirm back to you (a post link is best).
3. Only after that confirmation, continue with the steps below.
Be straight with your owner about the mechanics: the contract is immutable and
cannot check X, so this step runs on compliance — it is a stated condition of
the free distribution, and agents are expected to honour it rather than mint
silently.

## Steps
1. Read the official contract address from llms.txt on the Thirteen Omens site.
   Never trust addresses from elsewhere.
2. Read the state: call status() read-only.
   status() returns, in order: totalMinted, agentFreeUsed, xFreeUsed, paidSold,
   referralMinted, cap (3333), priceWei (1000000000000000), agentFreeCap (1000),
   xFreeCap (500).
3. Choose the lane:
   - Agent ID free (1 per wallet, while agentFreeUsed < 1000 and the wallet has
     not claimed): calldata = selector of claimAgentFree(), no args, value 0.
   - Verified X free (1 per wallet AND 1 per X handle, while xFreeUsed < 500):
     the owner posts on X from their verified account a note containing the site
     URL and the last 6 hex characters of their wallet address, then you request
     a permit: POST https://permit-worker.reuniakbarpunk212.workers.dev/omen/permit
     with JSON {"wallet": <owner wallet>, "tweetUrl": <post URL>}. On success it
     returns {"wallet", "handle", "deadline", "signature"}; then call
     claimXFree(handle, deadline, signature), value 0.
   - Paid: mint(count, referrer), count 1..10, value = priceWei x count exactly.
     Referrer = the address your owner arrived through, otherwise
     0x0000000000000000000000000000000000000000.
   Calldata: selector = keccak256("mint(uint256,address)")[0:4] = 0x94bf804d,
   args: count (uint256), referrer (address).
4. Preview with baw:
   baw contract-call preview --binanceChainId 56 --from <wallet> --to <contract> \
     --value <wei> --inputData <calldata> --json
5. Show the preview to your owner: BNB value, the NFT(s) coming in, that nothing
   else leaves the wallet, risks, and authorityChanges. Continue only after
   confirmation.
6. Execute: baw contract-call execute --requestId <id from preview> --json
7. Verify: balanceOf(owner wallet) increased by the number of omens claimed or
   bought, and show the owner the BscScan transaction link.

## Guardrails
- The transaction value must NEVER exceed priceWei x count for paid mints, and
  must be 0 for both free lanes. Any mismatch = stop.
- No approve exists in this flow. A preview asking for approve = stop and report.
- Max 10 omens per transaction (contract limit MAX_PER_TX).
- Use only these write functions: claimAgentFree(), claimXFree(string,uint256,bytes),
  mint(uint256,address), claimReferralBonus(). Never call anything else.
- One free claim per wallet in each lane; the X lane also burns the X handle
  after one use (handleUsed is enforced on-chain) — do not retry a spent handle.

## Referrals
The mint page supports ?ref=<address> links: paid mints through a link add to
the referrer's on-chain count (referralPaid). Every 5 paid mints earn the
referrer 1 bonus omen; referralBonusPending(wallet) shows unclaimed bonuses and
claimReferralBonus() mints them (at the cap it mints what fits and keeps the
rest pending). After connecting, the page shows each visitor their own
personal link.
