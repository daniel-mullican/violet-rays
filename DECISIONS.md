# Violet-Rays: decisions and trade-offs

Each entry is a decision I made, the alternative I rejected, what it cost, and what would make me change my mind.

---

## 1. Enforce weighted-average cost accounting at the database layer

**Chose:** database triggers recompute weighted-average material cost when a purchase is received, and a stock-adjustment ledger records every movement.

**Rejected:** computing cost in application code when inventory changes.

**Why:** cost accounting has to be right in every path, including the admin form written six months later by someone in a hurry. Application-level enforcement means every new write path has to remember, and the failure is silent: margin drifts, nothing errors, and the number is wrong until year end. A trigger cannot be forgotten by a code path that did not know it existed.

**Cost:** business logic now lives in the database, where it is less visible, harder to test with the application's test tooling, and easier to overlook when reading the code.

**Would revisit if:** the costing rules got complex enough to need real branching. At that point the logic wants to be in one auditable service with every write forced through it, which is the same principle applied differently.

---

## 2. Tier-aware recipes with base-recipe fallback

**Chose:** a recipe can be defined per tier, or left unset to inherit the product's base recipe.

**Rejected:** requiring a complete recipe for every tier of every product.

**Why:** most tiers differ from the base in one or two materials. Requiring a full recipe each time means adding a tier is a backfill across the whole catalog, and duplicated recipes drift apart the first time a material is substituted.

**Cost:** the effective recipe is now resolved rather than read, so anyone reading the schema has to know the fallback exists.

---

## 3. Replace quick add-to-cart with a mandatory configurator

**Chose:** every purchase goes through tier, upgrade, add-ons, note, and quantity.

**Rejected:** a standard add-to-cart button with options as an afterthought.

**Why:** the configuration determines the bill of materials and the shipping class. An order without one is an order the owner cannot fulfill and cannot cost. Making configuration mandatory pushes the ambiguity to the only place it can be resolved, which is before payment.

**Cost:** more friction between arriving and buying. On a boutique product with a considered purchase, that friction is mostly reassurance. On a commodity product it would be a conversion problem.

---

## 4. Change the product dimension instead of raising prices

**Chose:** trim one dimension so the parcel falls under the carrier's volumetric threshold.

**Rejected:** absorbing the dimensional-weight fee, or raising prices to cover it.

**Why:** the fee was triggered by sitting just over a threshold, not by the parcel being genuinely large. Raising prices would have made the customer pay for a packaging choice. Trimming the dimension cut projected standard shipping cost by roughly half with no change to what arrives.

**Cost:** a constraint on packaging design that has to be respected by every future product variant, and a threshold that the carrier can move. When the carrier changed its volumetric divisor, the model had already quantified the effect.

---

## 5. Live carrier rates with a static zone-banded fallback

**Chose:** call the carrier API for rates, and fall back to a static table banded by postal prefix on failure.

**Rejected:** live rates only, or static rates only.

**Why:** a shipping estimator that hard-fails when a third party is down turns a vendor outage into lost checkouts. Static-only is always available and always slightly wrong. The fallback is approximate, and approximate beats absent at the moment someone is deciding to buy.

**Cost:** two rate paths to maintain, and a fallback table that goes stale unless someone refreshes it.

---

## 6. Single-owner write model rather than per-user policies

**Chose:** row-level security scoped to authenticated owner writes, with public read limited to active listings.

**Rejected:** a per-user policy model.

**Why:** this is a single-tenant store with one operator. Per-user policies would add real complexity to protect against a tenant that does not exist. I accepted the tooling warnings that flag the broader policy, because the alternative is machinery justified by a hypothetical.

**Cost:** adding a second staff account later means revisiting the policy model rather than adding a row. That is a deliberate deferral, not an oversight, and it is written down here so it stays one.

---

## 7. Ship a passcode gate instead of a staging environment

**Chose:** middleware-based passcode gate on the live site, with the payment webhook route exempted.

**Rejected:** a full staging environment mirroring production.

**Why:** the owner needed to validate the real store with real payment configuration and real carrier rates. A staging environment with test credentials validates the staging environment. Gating production tested the actual thing, and the exemption kept real webhooks flowing.

**Cost:** production was briefly serving a gate to everyone, and a misconfigured exemption would have broken order provisioning silently. That route needed verifying first, not last.
