# Seam Catalog — worked examples

Read this when the blocker doesn't obviously match a seam shape in SKILL.md, or you want the before/after for a specific move. Examples are TypeScript; the moves are language-agnostic (notes where they aren't).

## 1. Parameter seam

**Blocker:** behavior depends on ambient state — clock, env, random, global config.

```ts
// BEFORE — untestable: "expired" depends on the real clock
function expireSessions(sessions: Session[]) {
  const cutoff = Date.now() - TTL_MS;
  return sessions.filter(s => s.lastSeen > cutoff);
}

// AFTER — default preserves every existing caller
function expireSessions(sessions: Session[], now: number = Date.now()) {
  const cutoff = now - TTL_MS;
  return sessions.filter(s => s.lastSeen > cutoff);
}
```

Test injects `now`; production calls unchanged. Prefer passing the *value* (`now`) over the *capability* (`clock`) — values can't surprise you.

Languages without default args (Go, Java pre-overload): add an overload/variant `expireSessionsAt(sessions, now)` and have the old name delegate to it.

## 2. Extract-the-logic seam

**Blocker:** the decision you need to test is mid-function, surrounded by I/O.

```ts
// BEFORE — testing the discount rule requires a DB and a mail server
async function checkout(orderId: string) {
  const order = await db.orders.find(orderId);
  let discount = 0;
  if (order.total > 100 && order.customer.tier === 'gold' && !order.flags.includes('clearance')) {
    discount = order.total * 0.1;
  }
  await mailer.sendReceipt(order, discount);
}

// AFTER — the rule is a pure function; checkout keeps its shape
export function goldDiscount(order: Order): number {
  return order.total > 100 && order.customer.tier === 'gold' && !order.flags.includes('clearance')
    ? order.total * 0.1 : 0;
}
async function checkout(orderId: string) {
  const order = await db.orders.find(orderId);
  const discount = goldDiscount(order);
  await mailer.sendReceipt(order, discount);
}
```

Test `goldDiscount` exhaustively with plain objects. The extraction is mechanical: select expression → extract function (let the IDE do it).

## 3. Constructor seam

**Blocker:** the class builds its own dependencies.

```ts
// BEFORE
class ReportService {
  private db = new PgClient(process.env.DATABASE_URL!);  // connects on construction
  ...
}

// AFTER — optional injection, default preserves behavior
class ReportService {
  constructor(private db: PgClient = new PgClient(process.env.DATABASE_URL!)) {}
  ...
}
```

Tests pass an in-memory fake implementing the same surface. If `PgClient`'s surface is huge, pair with a wrap seam (#6): inject a narrow `ReportStore` interface instead.

## 4. Extract-and-override seam

**Blocker:** dependency access is woven through a class; constructor injection would cascade through dozens of call sites.

```ts
// BEFORE — fetch is called in three methods
class PriceFeed {
  async latest(sym: string) { const r = await fetch(`${BASE}/q/${sym}`); ... }
}

// AFTER — access funneled through one protected method
class PriceFeed {
  protected async fetchQuote(sym: string): Promise<Response> {
    return fetch(`${BASE}/q/${sym}`);
  }
  async latest(sym: string) { const r = await this.fetchQuote(sym); ... }
}

// TEST
class FakePriceFeed extends PriceFeed {
  protected async fetchQuote(sym: string) { return jsonResponse({ price: 42 }); }
}
```

Deliberately temporary: it gets the class pinned so a real injection refactor can happen later WITH a net. Log the debt (worklog line). Not available in `final` classes / languages without override — fall back to #3 or #6.

## 5. Sprout seam

**Blocker:** the host function is enormous, untested, and hostile; you must ADD behavior, not change it.

```ts
// The 400-line monster keeps its shape; your feature is one call:
function processUpload(req) {
  // ...390 untouched lines...
  quarantineIfSuspicious(file, scanResult);   // <- sprout: new, fully tested elsewhere
  // ...
}
```

`quarantineIfSuspicious` is born test-first in its own module — 100% coverage, zero legacy entanglement. The monster gains one line (review it carefully; that line placement is the only untested risk). Characterize the monster's path through that line if its behavior could change (does the function throw? return early?) — keep the sprout's contract non-throwing/void if you can't.

## 6. Wrap seam (adapter)

**Blocker:** a global/third-party API (`Stripe`, `fs`, `localStorage`) is called directly at the site you're changing.

```ts
// You own this interface; it's as narrow as YOUR usage
interface ChargeGateway {
  charge(cents: number, token: string): Promise<ChargeResult>;
}
class StripeGateway implements ChargeGateway {
  charge(cents, token) { return stripe.charges.create({ amount: cents, source: token }); }
}
```

Introduce it ONLY at the call site in scope. The other 30 direct `stripe.*` calls are someone's future task — retrofitting them now is scope creep (the `verifying-done` diff check will flag it).

## Choosing under pressure — quick table

| Blocker shape | Seam |
|---|---|
| clock / random / env / config value | 1 parameter |
| pure decision buried in I/O | 2 extract-the-logic |
| ctor does real work | 3 constructor |
| access scattered through one class | 4 extract-and-override |
| adding behavior to a hostile function | 5 sprout |
| third-party/global API at the change site | 6 wrap |

## Safe-edit reminders (apply to every move)

- One mechanical transformation per commit; compile/run between micro-steps.
- Defaults/overloads preserve every existing call site.
- No behavior edits in seam commits — not even log lines or typo fixes.
- If the IDE offers the refactor (extract function/parameter), prefer it over hand-editing.
