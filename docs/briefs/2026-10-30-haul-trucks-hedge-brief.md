---
type: brief
title: "Hedging the EUR 18M Haul-Truck Receivable"
scenario: haul-trucks
engagement: fx-hedging
capability: fx-hedging
date: 2026-10-30
status: committed
hypothesis: "I expect the forward and money-market hedge to lock in about $20.39M each, and the put to beat them only if EURUSD ends above about 1.158, because once the placeholder forward is replaced, covered interest parity ties the two locks together and the put's premium sets its hurdle."
llm_used: "Claude Opus 5.5 (Anthropic)"
---

# Hedging the EUR 18M Haul-Truck Receivable

**Created by:** Landon Momohara  
**Updated by:** Landon Momohara  
**Date Created:** 2026-10-30  
**Date Updated:** 2026-10-30  
**Version:** 1.0
**LLM Used:** Claude Opus 5.5 (Anthropic)

---

## Executive Summary

We will receive EUR 18,000,000 from our Portuguese copper-mining customer in 365 days. At today's rate that is worth about $20.1 million, but a 10% fall in the euro would cost us roughly $2.0 million. Three hedge families can protect the payment: a forward, a money-market hedge, and a put option. This brief frames the risk and sets out the four-stage analysis that will end in a recommendation.

---

## Background & Objectives

The invoice is fixed in euros, so our dollar proceeds move with EURUSD. At the October 29 close of about 1.1176, the receivable is worth $20.1 million (Equiti, 2026). The euro has already slid from near 1.20 to about 1.11 this year, and the Fed (3.75–4.00%) and the ECB (2.50%) are both weighing further hikes (Board of Governors, 2026; Equiti, 2026). Our objective is to protect the dollar value of this payment at an acceptable cost.

---

## Methods

**Forward contract.** We agree today to sell EUR 18M in one year at a fixed rate. Pro: complete budget certainty. Con: we give up any gain if the euro strengthens, and it uses a bank credit line. The course's 1.1807 quote is a placeholder; at current rates, interest-rate parity implies about 1.133, or about $20.39 million.

**Money-market hedge.** We borrow euros today against the receivable, convert them at spot, and invest the dollars; the customer's payment repays the loan. Pro: the same certainty as a forward, and useful if forward pricing is poor. Con: it uses borrowing capacity and has more moving parts.

**Put option.** We buy the right to sell euros at today's spot for USD 0.0245 per euro, or $441,000 up front. It works like fire insurance. Pro: proceeds cannot fall below about $19.66 million, and we keep the upside if the euro rallies. Con: the premium is spent whether or not we use it. The quoted euro call protects a payable, not a receivable, so it does not apply here.

### What I am assuming

The customer pays in full and on time. Spot and rates are the October 29 close (USD 3.90%, EUR 2.50%), with fees and bid-ask spreads ignored. With more time, I would test customer credit risk and whether a partial hedge fits our policy.

### Hypothesis

I expect the forward and money-market hedge to each lock in about $20.39 million. I expect the put to beat them only if the euro finishes above about 1.158, a 3.6% rally. This is because interest-rate parity ties the two locks together, and the premium sets the put's hurdle.

### How I would know I was wrong

The hypothesis fails if a live dealer forward in Stage 4 differs from the parity rate by more than about 0.5%, or if the money-market hedge's real borrowing and lending rates break the tie.

---

## Limitations & Next Steps

All market inputs are provisional until Stage 4. The steps that follow are mine to complete:

1. **Stage 2, Model specification:** design the workbook's inputs, named ranges, and calculation flow on paper.
2. **Stage 3, AI-assisted build:** generate the workbook from the spec and audit its output.
3. **Stage 4, Market data:** replace placeholders with sourced, timestamped live prices.
4. **Stage 5, Validation and decision memo:** check against an independent model run and deliver the recommendation.

---

## References

Board of Governors of the Federal Reserve System. (2026, October 28). *Implementation note issued October 28, 2026*. https://www.federalreserve.gov/monetarypolicy/files/monetary20261028a1.pdf

Equiti. (2026, October 29). *ECB holds rates steady, signals further hikes if inflation persists; euro gains*. https://www.equiti.com/uae-en/news/trade-reviews/ecb-holds-rates-steady-signals-further-hikes-if-inflation-persists-euro-gains/
