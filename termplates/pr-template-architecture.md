# Explain it Like You’ll Be Asked About It in 6 Months
## Because Git Blame Doesn’t Show Intent
### If It’s Not Documented, It Didn’t Happen
#### Write the PR You Wish You Were Reviewing
##### PRs Are Where Engineering Happens
##### and so on ...

---

## 🧭 Why write pull requests like this?

Writing well-structured and thoughtful pull request descriptions may take a few more minutes —  
but those minutes **save hours of confusion, miscommunication, and bug hunting** later.

A good PR description:

✅ Builds **trust** — reviewers can focus on substance instead of guessing intent  
✅ Improves **review quality** — clear scope, context, and risks lead to better feedback  
✅ Documents **technical decisions** — months later, you’ll know why you did it  
✅ Enables **faster onboarding** — new team members learn from decisions, not just code  
✅ Reduces **back-and-forth** — when tests, impact, and edge cases are clear, discussions are shorter  
✅ Demonstrates **ownership** — you don’t just write code; you shape the system consciously

> “The best engineers don’t just write code.  
> They write *clarity*.”

By writing PRs like this, you're **thinking like a system owner**, not a ticket processor.


# 🛠 Pull Request Template

## 🚀 Title

> Short and purposeful summary of the change.  
> Example: Expose topic creation errors in Kafka Streams heartbeat status

---

## 🧠 Summary

What does this change introduce and who benefits from it?  
Focus on the **outcome and system/user value**, not just what was done.

---

## 🧨 Problem / Motivation

What was the real limitation, bug, maintenance issue, or technical debt?  
Why is this change necessary? What were the symptoms or blockers?

---

## 🛠 Solution

How was the problem solved?  
Describe the **patterns, components, architectural decisions** used.  
Highlight any **trade-offs, fallbacks, and backward compatibility** considerations.

---

## 🎯 Acceptance Criteria

What conditions must be met for this functionality to be considered complete and valid?  
✅ Always format as a checklist to clarify expectations.

- [ ] ...
- [ ] ...
- [ ] ...

---

## 🧪 Testing

What tests were added or modified?  
What scenarios are covered (unit, integration, e2e)?  
Were edge cases handled? Any mock data or config dependencies?

---

## 📉 Impact / Risk

Could this change:
- Affect performance?
- Introduce regression?
- Alter the behavior of existing components?

Also specify:
- Whether public APIs were changed or remain untouched
- Whether the change is backward compatible
- Whether it can be safely rolled back

---

## 🧹 Code / Module Changes

List of affected files or modules, e.g.:
- `AutoTopicCreationManager.scala` (+70 LOC)
- `KafkaApis.scala` (+15 LOC)
- `BrokerServer.scala` (+2 LOC)
- `AutoTopicCreationManagerTest.scala` (+170 LOC)

---
# EXAMPLE

# 🚀 Title

> Surface Kafka Streams internal topic creation failures in group status response

---

## 🧠 Summary

This PR implements error caching for internal topic creation failures in Kafka Streams.  
It allows detailed error reasons to be surfaced via the Streams group heartbeat status instead of only appearing in broker logs.

---

## 🧨 Problem / Motivation

When Kafka Streams attempts to create internal topics (e.g. repartition topics), failures are only visible in broker logs.  
During group heartbeat, the controller does not await the response from topic creation — failures are silently swallowed.

As a result, users see `MISSING_INTERNAL_TOPICS` with no explanation, making debugging extremely difficult.

---

## 🛠 Solution

- Introduced `CachedTopicCreationError` with timestamp
- Cached only real failures (error code ≠ NONE)
- TTL-based expiration (~90 seconds), size limit (1000 entries)
- Lazy cleanup (no background thread)
- Hooked into `ControllerRequestCompletionHandler.onComplete`
- Surfaced errors in `KafkaApis.scala` when group status = `MISSING_INTERNAL_TOPICS`
- Cleanup via `close()` in `BrokerServer.shutdown()`

---

## 🎯 Acceptance Criteria

- [x] If topic creation fails, the error is cached and exposed in group status
- [x] Cached errors expire after ~90 seconds (3× `request.timeout.ms`)
- [x] No error is cached for successful topic creations
- [x] Cache does not exceed 1000 entries; oldest are evicted
- [x] Errors are only queried when `MISSING_INTERNAL_TOPICS` is already reported
- [x] Cleanup is triggered on shutdown
- [x] Integration test verifies full flow

---

## 🧪 Testing

-  Unit tests:
    - TTL-based expiration
    - Size-based eviction
    - Filtering of only failed responses
-  Integration test:
    - End-to-end from topic creation failure to heartbeat error exposure
- All existing tests pass unmodified
- TTL values in tests adjusted to reflect real conditions

---

## 📉 Impact / Risk

- No changes to public API
- Fully backward compatible (inactive unless error occurs)
- Bounded memory usage (TTL + entry limit)
- Component is thread-safe (`ConcurrentHashMap`)
- Safe cleanup at shutdown

---

## 🧹 Code / Module Changes

- `AutoTopicCreationManager.scala` → +70 LOC
- `KafkaApis.scala` → +15 LOC
- `BrokerServer.scala` → +2 LOC
- `AutoTopicCreationManagerTest.scala` → +170 LOC

---