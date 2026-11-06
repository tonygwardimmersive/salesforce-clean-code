---
marp: true
title: Advanced Code
author: Tony Ward
paginate: true
---

# Advanced Code

Tony Ward

---

# What is Good Code?

<!-- Open it up to the room. Collect answers before moving on. -->

---

# Good code is…

1. **Easy to test**
2. **Easy to read**
3. **Easy to change**

*The rest of this talk is how to get there.*

---

# The journey

| Good code is… | So we cover… |
|---|---|
| **Easy to test** | Good tests |
| **Easy to read** | Clean code — names, comments, functions |
| **Easy to change** | Primitives, encapsulation, Demeter, connascence, design |

---

# Easy to test · Good tests

- **Test the actual outcome** — check what the code *did*, not just that it ran.
- **Tests are specifications** — each acceptance criterion on the user story becomes one test. The test names read back the story.
- **Fast or slow?**
  - **Fast** — quick feedback. Run them every time the code changes (watch mode).
  - **Slow** — regression. Run the full suite on every pull request and before a deploy.

---

# Easy to test · Remember: GUTS > NUTS > BUTS

- **GUTS** — Good Unit Tests. They fail when the code is wrong.
- **NUTS** — No Unit Tests. Every change is a guess.
- **BUTS** — Bad Unit Tests. Green, full coverage… and they check nothing.

> A bad test is **worse** than no test — because it lies to you.

---

# Easy to read · Why clean code matters

*Clean Code* — Robert C. Martin, 2008

- Code is **read far more** than it is written.
- Messy code slows a team down, a little more every sprint.
- **Leave it cleaner than you found it.**

**Even with AI:**
- AI reads your code too — clean code gives AI better context
- AI copies the patterns it sees. Clean in, clean out.
- AI writes the code. You own it.

Human in the loop.

---

# Easy to read · Meaningful names

> Names are the cheapest documentation you'll ever write.

| Instead of… | Write… |
|---|---|
| `lstAccs` | `accounts` |
| `bActive` | `isActive` |
| `get` / `fetch` / `load` for the same thing | one word, used everywhere |
| `test1()` | `closedOrdersCannotBeRefunded()` |

*If something is hard to name, the design is probably unclear.*

---

# Easy to read · Aside: Hungarian notation

`strName` · `lstAccounts` · `mapIdToAccount` · `bIsActive` · `m_description`

- Rife in Salesforce code — but it's a **1980s pattern**, from before editors could show you a variable's type.
- Microsoft, where it started, now says **don't use it**.
- The compiler already knows the type — e.g. `accounts` is a `List<Account>`.
- The prefix is **noise**.
- The prefix can **lie** — `sAccountNumber` is a number, not a String!

> Name what it **means**, not what it **is**: `accounts`, `accountsById`, `isActive`.

---

# Easy to read · Comments

- We used to judge code by how well it was commented.
- Not any more — **good code explains itself**.
- Comments go stale: the code changes, the comment doesn't. A wrong comment is worse than none.
- Code says **how**. Comments say **why** — and only when the code can't.

---

# Easy to read · Comments 1: saying what the code says

```apex
// increment the counter
counter++;
```

The code already says it. **Delete.**

---

# Easy to read · Comments 2: version history

```apex
/*
 * Modified 2019-03-12 by JB - added Old_Field__c
 * Modified 2020-07-01 by SK - removed Old_Field__c
 */
// List<Account> accs = [SELECT Id, Old_Field__c FROM Account];
```

We have **Git** now. It remembers who changed what, when, and why. **Delete.**

---

# Easy to read · Comments 3: rename instead

```apex
// Check if the account is an active enterprise customer eligible for renewal
if (acc.Status__c == 'Active' && acc.Tier__c == 'Enterprise' && acc.Contract_End__c != null)
```

```apex
if (isRenewableEnterpriseAccount(account))
```

A good name **is** the comment.

---

# Easy to read · Comments 4: comments lie

```apex
// Process in chunks of 200 to stay under the DML governor limit
Integer chunkSize = 150;
```

Someone changed the code. Nobody changed the comment.

**Which one is right?** The code always runs — the comment never does.

---

# Easy to read · Formatting

- Reads top to bottom, like a newspaper — headline first, details below.
- Keep related code close together.
- One team style — let a tool enforce it.

---

# Easy to read · Functions: do one thing well

> The first rule of functions: they should be small.
> The second rule: they should be smaller than that.

1. **Small, and do one thing**
2. **No hidden side effects**
3. **Few arguments** - **No true/false switches**

Easy to read
- 20 lines is easier than 200 (for AI and you)

Easy to test
- one thing to test

Easy to change
- it's in one place with less to break
- bonus: easier to reuse

---

# Easy to read · Functions 1: small, and one thing

```apex
public void processRenewals(List<Opportunity> opps) {
    List<Opportunity> toRenew = new List<Opportunity>();
    for (Opportunity opp : opps) {
        if (opp.StageName == 'Closed Won' && opp.CloseDate < Date.today().addDays(-330)) {
            toRenew.add(opp);
        }
    }
    List<Opportunity> renewals = new List<Opportunity>();
    for (Opportunity opp : toRenew) {
        Opportunity renewal = opp.clone(false, true, false, false);
        renewal.CloseDate = opp.CloseDate.addYears(1);
        renewal.StageName = 'Prospecting';
        renewals.add(renewal);
    }
    insert renewals;
}
```

---

# Easy to read · Functions 1: small, and one thing

```apex
public void processRenewals(List<Opportunity> opportunities) {
    List<Opportunity> eligible = filterRenewable(opportunities);
    List<Opportunity> renewals = buildRenewals(eligible);
    insert renewals;
}
```

- Reads like a story — **what**, not **how**.
- The detail lives one level down, in well-named functions.
- Aim for ~20 lines. Over 30 is a warning sign.
- *If you can pull out a piece and give it a good name — it was doing more than one thing.*

---

# Easy to read · Functions: nested ifs

```apex
if (opp.StageName == 'Closed Won') {
    if (opp.Account.IsActive__c) {
        if (opp.Amount > 0) {
            if (opp.Contact__r.Email != null) {
                sendRenewalNotice(opp);      // the one line that matters
            }
        }
    }
}
```

- **Arrow-shaped code** — you hold four conditions in your head to reach one line.
- **Hard to test** — every `if` adds another path to cover.
- **Hard to change** — where does the next rule go? Which `}` closes which `if`?

**Fix:** exit early, and give the conditions a name.

```apex
if (!isRenewable(opp)) { return; }
if (!hasEmail(opp))    { return; }
sendRenewalNotice(opp);
```

---

# Easy to read · Functions 2: few arguments

```apex
createTask(whatId, ownerId, 'Call back', dueDate, 'High', true);
```

What's `true`? What if someone swaps two of them?

```apex
TaskRequest request = new TaskRequest();
request.subject  = 'Call back';
request.priority = 'High';
...
createTask(request);
```

**0–2 arguments is ideal.** More than 3 → group them into an object.

---

# Easy to read · Functions 3: no true/false switches

```apex
saveAccounts(accounts, true);   // true what?
```

The caller has to know — and pass in — something the record already knows.

```apex
saveAccounts(accounts);

public void saveAccounts(List<Account> accounts) {
    upsert accounts;   // new or existing — Salesforce works it out
}
```

**One method, no switch.** New or existing, it just works.

---

# Easy to read · Functions 4: no hidden side effects

```apex
public Decimal getDiscountedTotal() {
    this.total = this.total * 0.9;   // surprise!
    return this.total;
}
```

Call it twice — you've given two discounts.

```apex
public Decimal calculateDiscountedTotal() {
    return this.total * 0.9;
}
```

A function either **does** something or **answers** something — never both.

---

# Easy to change · Code smells

A **code smell** is a hint that something's wrong. The code works — but it will be hard to change.
*— Kent Beck & Martin Fowler, Refactoring, 1999*

**Primitive obsession** — plain Strings standing in for one real thing. *(Next.)*

**God class** — one class that does everything:

```apex
public class ShippingManager {
    public Decimal calculateCost(...)  { ... }
    public void printLabel(...)        { ... }
    public void emailCustomer(...)     { ... }
    public void trackParcel(...)       { ... }
    // …and 80 more
}
```

Every change touches it. Everyone edits it. Nobody can test it on its own.

---

# Easy to change · What's wrong with this?

```apex
String addressLine1;
String addressLine2;
String postcode;
```

<!-- Pause. Let the room answer. -->

---

# Easy to change · Loose Strings, one address

- **They always travel together** — every method needs all of them:
  `createShipment(line1, line2, postcode)`
- **Easy to mix up** — swap `addressLine2` and `postcode` and it still compiles.
- **Add a County?** Change every method that passes an address around.
- **The rules get copied** — check the postcode, format it for a label… in every class.

This is **primitive obsession**: plain Strings and numbers standing in for one real thing.

---

# Easy to change · Make it one thing

```apex
public class Address {
    private String addressLine1;
    private String addressLine2;

    public Address(String addressLine1, String addressLine2) {
        this.addressLine1 = addressLine1;
        this.addressLine2 = addressLine2;
    }
}
```

```apex
Address shipTo = new Address('1 Market Street', 'London');
createShipment(shipTo);
```

- One parameter, can't be mixed up.
- Salesforce does this too: **Billing Address** is one field on screen, `acc.BillingAddress` in Apex.

---

# Easy to change · Train wrecks crash

```apex
String country = order.getCustomer().getAddress().getCountry();
```

The customer has no address yet…

```
System.NullPointerException: Attempt to de-reference a null object
```

- Every `.` is another thing that can be **null**.
- Guard every step — in every caller?

```apex
if (order.isUkBased()) { ... }
```

**`Order` handles the missing pieces — once.**

---

# Easy to change · Law of Demeter

> Don't talk to strangers.

- That train wreck broke this rule.
- Talk to your neighbours, not their neighbours.
- **Tell, don't ask** — don't pull data out of an object to make its decisions. Ask it to act.

---

# Easy to change · Encapsulate

- An **object** keeps its data private and exposes **behaviour**.
- A **data structure** exposes data and has no behaviour — records, DTOs.
- Both are fine. A mix of the two is the worst of both.

> A getter and setter for every field isn't encapsulation.
> Keep the data **and the rules about it** together.

---

# Easy to change · Encapsulate: public data

What if `Address` made its fields public?

```apex
public class Address {
    public String addressLine1 { get; set; }
    public String addressLine2 { get; set; }
}
```

…and in 20 places across the codebase:

```apex
String label = shipTo.addressLine1 + '\n' + shipTo.addressLine2;
```

- Rename `addressLine1` to `street`… **20 places break.**
- Add a County… **20 places to update** — and one gets missed.
- And anyone can change it: `shipTo.addressLine1 = '';`

**If callers must read it** — public getter, **private setter**:

```apex
public String addressLine1 { get; private set; }
```

---

# Easy to change · Encapsulate: hide the data

```apex
public class Address {
    public String addressLine1 { get; private set; }
    public String addressLine2 { get; private set; }

    public Address(String addressLine1, String addressLine2) { ... }

    public String formatForLabel() {
        return addressLine1 + '\n' + addressLine2;
    }
}
```

```apex
String label = shipTo.formatForLabel();
```

Rename a field, add a County — **one class changes.** The rule lives in one place.

---

# Easy to change · Encapsulate: tell, don't ask

```apex
public class Address {
    public Date endDate { get; private set; }

    public void endOn(Date moveDate) {
        endDate = moveDate;
    }

    public Boolean isCurrent() {
        return endDate == null || endDate > Date.today();
    }
}
```

```apex
if (shipTo.isCurrent()) { createShipment(shipTo); }

shipTo.endOn(moveDate);
```

- **Primitive obsession** → an `Address` class, not `addressLine1` Strings.
- **Encapsulation** → the address controls how it changes: `endOn()`
- **Tell, don't ask** → the address makes its own decisions: `isCurrent()`

---

# Easy to change · Tell, don't ask: in Salesforce

It's not just code.

| | Ask | Tell |
|---|---|---|
| **Flow** | Three flows each: Get Case → Decision *"Is it overdue?"* → Update Case | Each flow calls one subflow: **Escalate If Overdue** |
| **API integration** | The other system reads the Order, checks the status and stock, then sets `Status = Shipped` | The other system says **"ship this order"** — Salesforce decides if it can |

- **Ask** — the rule is copied into every flow, and into every system that calls you.
- **Tell** — the rule lives in **one place**. Change it once.

---

# Easy to change · Connascence

A **measure of how easy code is to change**.
Two pieces of code are **connascent** if changing one forces you to change the other.

**Static** — you can see it in the code

| Type | Must agree on… | Example |
|---|---|---|
| 1. Name | a name | `shipTo.formatForLabel()` — rename it, every caller changes |
| 2. Type | a type | `String label = shipTo.formatForLabel();` |
| 3. Meaning | what a value means | `if (status == 2)` — what is 2? |
| 4. Position | the order of things | `createShipment(line1, line2, postcode)` |
| 5. Algorithm | the same calculation | `price * 1.2` for VAT, in two classes |

**Dynamic** — only shows up at run time

| Type | Must agree on… | Example |
|---|---|---|
| 6. Execution order | the order **you** wrote it | `service.init();` must come before `service.process();` |
| 7. Timing | the order **the system** runs things | two triggers on Account — Salesforce doesn't promise which runs first |
| 8. Value | values that change together | `opp.Amount` must equal `Quantity * UnitPrice` |
| 9. Identity | being the *same* object | changing a re-queried copy, not `Trigger.new` |

**Weakest → strongest.** Refactor toward the top of the list.

*Fix it by moving a line? Execution order. Only fails under load, async or bad luck? Timing.*

---

# Easy to change · Inheritance

A child class gets everything its parent has: `ShippingRate extends Rate`

**Widely used** — it's the first thing taught about objects. Free reuse!

**…but terrible for change:**
- **Welded to the parent** — change the parent, and every child can break.
- **You get everything** — wanted or not.
- **Combinations explode** — `Standard`, `StandardInsured`, `Express`, `ExpressInsured`, `ExpressInsuredWeekend`…


> *Favour composition over inheritance* — Gang of Four, 1994

---

# Easy to change · Interfaces

An interface is a **promise**: "anything that implements me can do this."
No code, just the contract.

You already use them — Salesforce is built on them:

```apex
public class NightlyCleanup implements Schedulable {
    public void execute(SchedulableContext context) { ... }
}
```

`Schedulable` · `Queueable` · `Database.Batchable` · `HttpCalloutMock`

- Salesforce doesn't care **what** your class is — only that it keeps the promise.
- A class can keep **many** promises, but only have one parent.
- Depend on the promise, and you can swap in a fake for testing.

---

# Easy to change · Composition

Inheritance says a class **is a** something. Composition says it **has a** something.

Every part keeps the same promise:

```apex
public interface ShippingCharge {
    Decimal cost();
}
```

The shipment just adds them up — it doesn't know or care what each part is:

```apex
public class Shipment {
    private List<ShippingCharge> charges = new List<ShippingCharge>();

    public void add(ShippingCharge charge) {
        charges.add(charge);
    }

    public Decimal totalCost() {
        Decimal total = 0;
        for (ShippingCharge charge : charges) {
            total += charge.cost();
        }
        return total;
    }
}
```

*Aside: Lightning Web Components work the same way — you build a component **from** other components, not by inheriting from one.*

---

# Easy to change · SOLID

Five principles for using classes and interfaces well — Robert C. Martin

| | Principle | Description | Example |
|---|---|---|---|
| **S** | Single responsibility | One reason to change | Break up `ShippingManager`: `Shipment` adds up the cost. Printing labels and emailing the customer belong somewhere else. |
| **O** | Open / closed | Add new code, don't edit old code | Next-day delivery? A new `NextDayDelivery` class — `Shipment` doesn't change. |
| **L** | Liskov substitution | Anything that keeps the promise must work the same way | Swap `NextDayDelivery` for `NextWorkingDayDelivery` — both return a cost, so `Shipment` still works. |
| **I** | Interface segregation | Keep promises small | `ShippingCharge` only asks for `cost()`. Add `printLabel()` and `Insurance` has to pretend it can print. |
| **D** | Dependency inversion | Depend on the promise, not the class | `Shipment` knows `ShippingCharge`, not `ExpressDelivery` — so a test can hand it a fake charge. |

---

# Good code is…

1. **Easy to test** — tests that check the outcome, read like the spec, and run fast.
2. **Easy to read** — good names, few comments, small functions that do one thing.
3. **Easy to change** — encapsulate the data, keep connascence weak, build from components.

---

# Bonus: teaching AI good code

Everything in this talk is packaged as **15 skills** for Claude Code.

- Claude applies them as it writes your Apex and LWC.
- A review skill checks existing code against all of it.
- People can use them too — they're plain, readable checklists.

> Give AI your standards, and it writes code your team wants to keep.

---

# Get started

```bash
npx skills add tonygward/salesforce-skills
```

`github.com/tonygward/salesforce-skills`

**Questions?**
