# Producer–Consumer with Bounded Blocking Queue
- [Overview](#overview)
- [1) Data Model: Item](#1-data-model-item)
- [2) SharedQueue](#2-core-component-sharedqueue)
  - [2.1 Constructor & Internal State](#21-constructor-and-internal-state)
  - [2.2 stop() and is_stopped()](#22-stop-and-is_stopped)
  - [2.3 enqueue()](#23-enqueue)
  - [2.4 dequeue()](#24-dequeue)
- [3) Producer Thread](#3-producer-thread)
- [4) Consumer Thread](#4-consumer-thread)
- [5) DataTransferManager](#5-orchestration-datatransfermanager)
- [6) Demo: main_demo()](#6-demo-main_demo)
- [Improvements](#improvements)
- [Tests](#tests)
- [Scaling Considerations](#scaling)
- [Design Assumptions](#design-assumptions)

# Sales Analytics
- [Why @dataclass](#why-dataclass)
- [SalesDataLoader](#salesdataloader)
  - [load_from_csv](#load_from_csv)
  - [_parse_row](#_parse_row)
- [SalesAnalyzer](#salesanalyzer)
  - [getSalesByDateRange](#getsalesbydaterange)
  - [getTotalSalesByRegion](#gettotalsalesbyregion)
  - [getAverageSaleByCategory](#getaveragesalebycategory)
  - [getTopSalespersons](#gettopsalespersons)
  - [getMonthlySalesTrend](#getmonthlysalestrend)
  - [generateSummaryReport](#generatesummaryreport)
- [Sales Improvements](#sales-improvements)
- [Scalability Enhancements](#scalability-enhancements)
- [Testing Coverage](#testing-coverage)
- [Assumptions](#assumptions)


“This code implements a producer-consumer pipeline using a bounded blocking queue of capacity 10. 
Producer pushes Items into the queue, consumer pulls them out, and we use Lock + Condition(wait/notify) for correct blocking and graceful shutdown.”

---

## 1) Data model: Item

**Where:** `@dataclass(frozen=True) class Item`

**What to say:**

“Item is the unit of transfer. It has id, data, and timestamp.”

“I used frozen=True to make it immutable, so once created no thread can modify it. That avoids shared-mutation bugs when multiple threads read the same object.”

**Key point:**

Immutable objects reduce concurrency risk.

---

## 2) Core component: SharedQueue

### 2.1 Constructor and internal state (1 min)

**Where:** `class SharedQueue.__init__`

**What to say:**

“This queue is bounded by capacity (default 10).”

“The storage is a deque, because append and popleft are O(1).”

“All shared state is protected by one lock: _lock.”

“Two Condition variables use the same lock:”

- `_not_empty`: consumer waits when queue is empty  
- `_not_full`: producer waits when queue is full  

“_stopped is a shutdown flag to break out of waits.”

**Why two conditions:**

“Producer and consumer wait for different conditions, so splitting them makes signaling precise.”

---

### 2.2 stop() and is_stopped()

**Where:** `stop()` and `is_stopped()`

**What to say:**

“stop() is the shutdown trigger: set _stopped=True under lock, and notify_all() on both conditions.”

“The notify_all is important because threads might be blocked inside wait(); this wakes them so they can check _stopped and exit.”

“is_stopped() is a thread-safe getter for _stopped under the same lock.”

---

### 2.3 enqueue()

**Where:** `enqueue(self, item, timeout=None)`

**Walkthrough:**

“Producer calls enqueue to add an item.”

“We enter with self._not_full: which acquires the underlying lock.”

“If the queue is full, we wait in a while loop.”

“I use while, not if, because condition waits can wake up spuriously or another thread may fill the queue before I reacquire the lock.”

“Before waiting, we check _stopped and exit early with TransferStoppedError.”

**Timeout:**

“If timeout is given, we compute remaining using time.monotonic() so clock changes don’t break the math.”

“If remaining <= 0, raise TimeoutError.”

**When space exists:**

“Append to deque.”

“notify() on _not_empty so a waiting consumer wakes up.”

**Important correctness:**

Lock protects buffer size check + append as one atomic section.

---

### 2.4 dequeue()

**Where:** `dequeue(self, timeout=None)`

**Walkthrough:**

“Consumer calls dequeue to remove an item.”

“We enter with self._not_empty: to hold the lock.”

“If queue is empty, wait in a while loop.”

“If _stopped is true while empty, raise TransferStoppedError to exit cleanly.”

Timeout logic mirrors enqueue.

**When item exists:**

“Pop left.”

“Notify _not_full so a blocked producer can enqueue again.”

“Return the item.”

---

## 3) Producer thread

**Where:** `class Producer(threading.Thread)`

**What to say:**

“Producer is a thread that iterates through sourceContainer and enqueues each item.”

“It checks stop_event before working and also inside retry loop.”

“If queue is full and enqueue times out, it retries. This prevents permanent blocking and keeps the thread responsive.”

“If any unexpected exception happens, it stores the error and sets stop_event to trigger a coordinated shutdown.”

**Mention:**

`produced_count` is for observability/testing.

---

## 4) Consumer thread

**Where:** `class Consumer(threading.Thread)`

**What to say:**

“Consumer loops continuously, dequeues items, and appends them into destinationContainer.”

“If queue is empty, dequeue may timeout, and consumer simply retries.”

“If TransferStoppedError happens, it exits — meaning no more items will come and queue is shutting down.”

“It tracks consumed_count and captures errors similarly.”

**Key detail:**

“Consumer exit is controlled by queue stop signal and exception path.”

---

## 5) Orchestration: DataTransferManager

### 5.1 Constructor (20 sec)

“Creates the shared queue, stop event, and instantiates producer + consumer.”

---

### 5.2 startTransfer()

“Starts threads once. _started prevents double-start.”

---

### 5.3 stopTransfer()

“Sets stop_event and calls sharedQueue.stop().”

“stop_event stops producer loop, and queue stop wakes any waits.”

---

### getQueueStatus()

Returns current queue state. Useful for monitoring or debugging.

---

### 5.4 waitForCompletion()

**What to say clearly:**

“Join producer first.”

“If the producer is still running after the timeout: stop everything and raise timeout error.”

“If the producer finished but had an error: stop everything and raise original error.”

“Once producer finishes, call sharedQueue.stop() to tell consumer no more items will come.”

“Then join consumer, with remaining timeout.”

“If the consumer is still alive after that: stop and raise.”

“If consumer error exists → raise.”

This method guarantees no hanging threads.

---

## 6) Demo: main_demo()

**What to say:**

“Creates 25 items as the source.”

“Starts manager with capacity 10.”

Creates an empty list called dest, which will store the items processed by the Consumer.

Calls startTransfer(), which starts both Producer and Consumer threads concurrently.

“Prints queue status while producer is alive — so you can observe backpressure.”

Calls waitForCompletion() with a timeout to ensure both threads finish cleanly and don’t hang.

Prints statistics like how many items were produced and consumed.

“Asserts all items were transferred in order.”

---

## Improvements

### 🥇 Simplify Shutdown Coordination

shutdown is where most concurrency bugs hide because threads may be:
- blocked in wait()
- mid enqueue/dequeue
- checking different flags at different times

Currently 3 signals:

- stop_event  
- _stopped  
- TransferStoppedError  

Risk:

- Threads may observe inconsistent shutdown states.

Improvement:

Use single authoritative shutdown mechanism:

Either sentinel (poison pill)  
Producer enqueues sentinel item at the end.  
Consumer exits only when it consumes sentinel.

---

### 3️⃣ Encapsulation

- Right now, the Producer is handling retry logic when the queue is full. That means it knows too much about how the queue works.
- A cleaner design would move that retry logic into the queue itself. The Producer should just produce items and call enqueue(), and the queue should handle waiting and retry internally.
- This improves encapsulation because each class handles its own responsibility. The Producer focuses on producing data, and the queue handles synchronization and blocking policy. It also makes the code cleaner and easier to maintain if the retry behavior changes later.”

---

### 4️⃣ Remove daemon=True

Daemon threads:
- Automatically terminate when the main thread exits.
- Prevent the program from hanging if something goes wrong
but
- Terminate automatically when main thread exits  
- Can terminate abruptly  
- Bypass cleanup  

Improvement:

Remove daemon flag.  
Rely entirely on join().  
Improves lifecycle safety.

---

### 🏅 Add Stress Testing

Test scenarios: Currently I test correctness, blocking behavior, graceful shutdown, and timeout handling

- capacity=1  
- fast producer / slow consumer  
- slow producer / fast consumer  
- random sleeps  

Concurrency bugs often appear only under stress.

# Tests

This test suite verifies:

- Correct FIFO data transfer
- Blocking behavior (full and empty queue)
- Timeout handling
- Graceful shutdown
- Early termination
- No thread leaks
- No data loss or duplication
- Proper error propagation
---

## Test Suite: `TestSharedQueue`

### 1️⃣ test_enqueue_dequeue_basic

Validates basic functionality of the queue.

- Enqueue one item.
- Dequeue the same item.
- Verify equality.

✅ Ensures basic FIFO behavior works correctly.

---

### 2️⃣ test_dequeue_timeout_when_empty

- Create empty queue.
- Attempt to dequeue with timeout.

Expected result:

- Raises `TimeoutError`.

✅ Ensures consumer blocks correctly and timeout handling works.

---

### 3️⃣ test_enqueue_timeout_when_full

- Create queue with capacity = 1.
- Enqueue one item.
- Attempt to enqueue second item with timeout.

Expected result:

- Raises `TimeoutError`.

✅ Ensures producer blocks correctly when queue is full.

---

### 4️⃣ test_stop_unblocks_waiters

- Fill queue.
- Call `stop()`.
- Attempt enqueue → should raise `TransferStoppedError`.
- Dequeue existing item.
- Attempt dequeue again → should raise `TransferStoppedError`.

✅ Ensures:
- Shutdown flag works.
- Waiting threads are unblocked.
- No deadlock occurs during shutdown.

---

## Test Suite: `TestDataTransferManager`

### 5️⃣ test_complete_transfer_all_items

- Create 50 items.
- Start transfer.
- Wait for completion.

Assertions:

- All items transferred.
- Produced count == source size.
- Consumed count == source size.
- FIFO ordering preserved.

✅ Validates full end-to-end correctness.

---

### 6️⃣ test_stop_transfer_early

- Start transfer with large dataset.
- Stop early.
- Join both threads.

Assertions:

- Destination size ≤ source size.
- Producer not alive.
- Consumer not alive.

✅ Ensures:
- Early shutdown works.
- No hanging threads.
- System terminates safely mid-transfer.

---

### 7️⃣ test_wait_for_completion_timeout

- Start transfer with very large dataset.
- Call `waitForCompletion()` with very small timeout.

Expected:

- Raises `TimeoutError`.

Then:

- Stop transfer.
- Ensure threads terminate.

✅ Validates:
- Timeout handling.
- Manager-level safety control.
- No indefinite blocking.

---

# Scaling

## 1) Replace custom queue with Python’s built-in `queue.Queue` for reliability

You built your own `SharedQueue` using:

- Lock  
- Condition  
- wait/notify  

It works, but it’s easy to make mistakes in custom concurrency code.

### Why `queue.Queue` is better

This queue is:

- Already thread-safe  
- Already handles blocking on full/empty  
- Already handles timeouts correctly  
- Extremely well-tested  

---

## 2) If workload becomes CPU-bound, switch to multiprocessing to bypass the GIL

### What is CPU-bound?

CPU-bound means the consumer spends time doing heavy computation, like:

- parsing huge files  
- encryption/compression  
- ML inference  

### Why threads won’t scale in Python (GIL)

Python has a GIL (Global Interpreter Lock) which means:

- only one thread runs Python code at a time  
- even if you create 10 threads, they don’t truly run in parallel on multiple CPU cores (for CPU-heavy work)  

So adding more threads won’t speed up CPU-heavy consumer work much.

### What multiprocessing does

Multiprocessing creates separate processes, not threads.

Each process has:

- its own Python interpreter  
- its own GIL  

So multiple processes can run truly in parallel on different cores.

Example idea:

- 1 producer process  
- 4 consumer processes  

That actually scales on a 4-core machine.

---

## 3) If IO-bound, move to an async model

### What is IO-bound?

IO-bound means tasks spend most of their time waiting for:

- network calls (API requests)  
- database queries  
- reading/writing files  
- calling cloud services  

Here, CPU is mostly idle because the program is waiting.

### Why async is better for IO

With threads:

- each waiting task uses a thread  
- many threads = memory cost + context switching overhead  

With async:

- one thread can manage thousands of waiting tasks  
- while one task waits for IO, the event loop runs another task  

So for IO-heavy work, async gives:

- higher concurrency  
- less overhead  
- better scalability on one machine  

Example:

If the consumer calls an API, an async consumer can handle 1,000 requests concurrently without 1,000 threads.

---

## 4) For horizontal scaling across machines, replace in-memory queue with Kafka/Redis

### Why in-memory queue doesn’t scale across machines

Your current queue exists inside one Python process.

If you want 5 machines running consumers:

- they cannot share your in-memory deque  
- if the machine dies, you lose queued data  

### What Kafka / Redis / RabbitMQ / SQS gives you

A message broker is a shared system that stores messages centrally.

A message queue decouples producers and consumers.

Now you can have:

- many producers (on different machines)  
- many consumers (on different machines)  

And the broker handles:

- durability (messages don’t vanish if one consumer dies)  
- distribution (load spreads across consumers)  
- retry (failed processing can be re-delivered)  
- ordering guarantees (Kafka partitions)  
- scaling (add more consumers to increase throughput)  

So this is the true “scale-out” step.

# Design Assumptions

## 1️⃣ Single Producer and Single Consumer

The assignment didn’t explicitly say how many producers/consumers.

I assumed:

- One producer  
- One consumer  

If multiple producers/consumers were required, we would need:

- Stronger contention testing  
- Fairness guarantees  
- Possibly different signaling strategy  

---

## 2️⃣ FIFO Ordering Is Required

They didn’t explicitly say ordering must be preserved.

I assumed:

- Items must be consumed in the same order they were produced.  

That’s why I used `deque` and FIFO semantics.

If ordering was not required, we could use:

- Priority queue  
- LIFO stack  
- Work stealing model  

---

## 3️⃣ Cooperative Shutdown Model

The assignment says “terminate gracefully.”

I assumed:

- Threads should not be force-killed.  
- Shutdown should be cooperative via flags and condition signaling.  

Python does not support safe forced thread termination, so cooperative shutdown is the correct assumption.

---

## 4️⃣ All Items Fit in Memory

Since the queue is in-memory, I assumed:

- Items are small  
- Memory capacity is not a constraint  

If items were large (e.g., files, images), we would need:

- Streaming model  
- Disk-backed queue  
- External message broker  

---

## 5️⃣ Consumer Work Is Not Heavy CPU-Bound

Because we use threads, I assumed:

- Work is IO-bound or lightweight.  

If the consumer performs heavy CPU computation, Python’s GIL becomes a bottleneck and multiprocessing would be required.

# Sales Analytics System

---

## Why @dataclass

“I used dataclass to reduce boilerplate and clearly represent structured sales data.”

---

## SalesDataLoader

### load_from_csv

Takes a file path as input and returns a list of SaleRecord.  
Uses try block to safely handle file-related errors.

Opens the file using with so it auto-closes properly.  
newline="" → prevents newline parsing issues in CSV  
encoding="utf-8" → handles special characters safely  

Uses csv.DictReader so each row is a dictionary (safer than index-based access).  

“I used DictReader instead of index-based parsing to avoid column-order dependency and improve readability.”

Checks if the CSV has a header row.

Validates that all required columns are present.

Creates an empty list to store parsed records.

Loops through each row using enumerate.

Uses start=2 because line 1 is the header.

Calls _parse_row() to handle parsing and validation.

Appends each valid SaleRecord to the list.

Returns the final list of records.

Catches FileNotFoundError and raises a clear error message.

---

### _parse_row(row: Dict[str, str], line_no: int)

This method takes one CSV row and converts it into a SaleRecord.

It validates all required fields.

It converts string values into proper types.

It applies basic business rules.

If anything is wrong, it raises a meaningful error with the exact line number.

---

### Why I Created _parse_row Separately

I didn’t want parsing logic mixed inside the file-reading loop.

This keeps the code clean and modular.

It makes testing easier — I can test row parsing independently.

---

### The req() Helper Function

I created a small helper function inside _parse_row.

Its job is to safely fetch required fields.

If a field is missing or empty, it throws a clear error.

---

### Extracting Required Fields

I call req() for all required columns.

This guarantees no required field is missing.

It also strips whitespace to avoid subtle data issues.

---

### Type Conversions

I convert quantity to int.

I convert unitPrice to float.

I parse date using strict ISO format.

If conversion fails, it automatically gets caught in the exception block.

This ensures the object has correct data types.

---

### Handling totalAmount

If totalAmount exists in the CSV, I use it.

Otherwise, I calculate it as quantity * unitPrice.

This makes the system flexible.

---

### Business Rule Validation

I check that quantity, unit price, and total amount are not negative.

If any are negative, I raise a validation error.

This protects data integrity.

---

### Creating the Final Object

After validation and conversion, I create and return a SaleRecord.

Since it’s a frozen dataclass, it becomes immutable.

That prevents accidental modification later.

---

### Exception Handling Strategy

If it’s already a MalformedCSVError, I re-raise it.

If it’s any other error (like type conversion), I wrap it inside MalformedCSVError.

I include the line number in every error.

This makes debugging very easy.

---

## SalesAnalyzer

### getSalesByDateRange()

What This Method Does  
It filters sales between a start date and end date.

It can optionally filter by region.

It can optionally filter by product category.

It returns a list of matching sale records.

First Step — Validate Date Range  
It checks if start_date is greater than end_date.

If yes, it raises a ValueError.

This prevents logical mistakes early.

Second Step — Filter by Date  
It uses filter() with a lambda function.

Keeps only records where the date falls within the range.

At this point, it creates a filtered iterator, not a list yet.

Third Step — Optional Region Filter  
If region is provided, it filters further.

Only keeps records that match the region.

If region is None, it skips this step.

Fourth Step — Optional Category Filter  
Same logic as region.

If category is provided, it filters further.

If not provided, it skips it.

Final Step — Convert to List  
Since filter() returns an iterator, it converts it to a list.

Returns the final filtered records.

---

### getTotalSalesByRegion()

Calculates total sales amount per region.

Returns a dictionary like:

{
 "West": 12000.0,
 "East": 8000.0
}

Creates an empty dictionary called totals.

Loops through every sale record.

For each record:

Gets the region.

Adds totalAmount to that region’s running total.

Uses:

totals.get(r.region, 0.0)

This means:

If region exists → get current total.

If not → start from 0.0.

Returns the final dictionary.

---

### getAverageSaleByCategory()

Calculates average sale amount per product category.

Returns something like:

{
 "Electronics": 250.0,
 "Clothing": 100.0
}

Step 1: Track sum and count  
Creates a dictionary:

{ category: (sum, count) }

For each record:

Gets existing sum and count.

Adds sale amount to sum.

Increments count.

Step 2: Compute average  
Uses dictionary comprehension:

s / c

Protects against division by zero:

if c else 0.0

This is:

Group by category  
Calculate sum  
Divide by count

---

### getTopSalespersons(n)

Returns top n salespersons based on total sales.

Example:

[("Alice", 12000), ("Bob", 10000)]

Step 1: Handle invalid n  
If n <= 0, return empty list.

Step 2: Group by salesperson  
Build dictionary:

{ salesperson: total_sales }

Step 3: Sort  
Uses:

sorted(..., key=lambda kv: kv[1], reverse=True)

Sorts by total sales descending.

Step 4: Slice  
Takes first n elements:

[:n]

Group → Sort → Take Top N

---

### getMonthlySalesTrend()

Groups sales by month (YYYY-MM format).

Returns:

{
 "2025-01": 5000,
 "2025-02": 7000
}

Step 1: Create month key  
Formats date like:

2025-01

Uses zero padding for consistency.

Step 2: Aggregate  
Adds totalAmount to that month.

Step 3: Sort  
Uses:

sorted(trend.items())

Ensures chronological order.

Group by month → Sum → Sort

---

### generateSummaryReport()

Creates a full analytics summary.

Combines all other methods into one structured report.

Step 1: Compute Grand Total  
Uses reduce():

total_sales = reduce(lambda acc, r: acc + r.totalAmount, ...)

Accumulates total sales across all records.

Step 2: Build Report Dictionary  
Includes:

Grand total  
Sales by region  
Average by category  
Top performers  
Monthly trend  
Record count

---

## Sales Improvements

1️⃣ Validate totalAmount Consistency  
Right now:

total_amount = float(row.get("totalAmount", "")) if row.get("totalAmount") else quantity * unit_price

If CSV gives wrong total, you accept it.

Inside _parse_row()

Right after computing total_amount.

Add:

expected_total = quantity * unit_price  
if abs(total_amount - expected_total) > 1e-6:  
   raise MalformedCSVError(  
       f"Line {line_no}: totalAmount mismatch (expected {expected_total})"  
   )

---

2️⃣ Enforce Unique transactionId  
Duplicates currently allowed.

Inside load_from_csv()

Before appending record to list.

Add a set at top of method:

seen_ids = set()

Then inside loop:

if record.transactionId in seen_ids:  
   raise MalformedCSVError(f"Duplicate transactionId at line {i}")  
seen_ids.add(record.transactionId)

---

3️⃣ Case-Insensitive Filtering

Filtering currently strict match.

Inside getSalesByDateRange():

Change:

r.region == region

To:

r.region.lower() == region.lower()

Same for category.

---

4️⃣ Optimize Top-N With Heap

Currently:

sorted(...)[ : n ]  
O(n log n)

Inside getTopSalespersons()

Replace sorting line with:

import heapq  
return heapq.nlargest(n, totals.items(), key=lambda kv: kv[1])

O(n log k)

---

5️⃣ Support Streaming for Large Files

Currently

CSV → load entire list → store in memory → analyze

Modify load_from_csv():

Instead of building list:

records.append(SalesDataLoader._parse_row(row, line_no=i))

Use:

yield SalesDataLoader._parse_row(row, line_no=i)

Then analyzer would accept iterable instead of list.

Update:

def __init__(self, records: Iterable[SaleRecord]):

If loader returns a generator:

Generator gets exhausted after first use.

Subsequent methods break.

A fully streaming solution would require restructuring the analyzer to compute aggregations in a single pass or re-read the file per method, which increases architectural complexity.

---

## Scalability Enhancements

Current scale limitation?

Right now, I load the entire CSV into memory.

All records are stored in a list.

That works fine for small files.

But if the file has millions of rows, memory usage will become a problem.

How would I improve it?

1️⃣ Use streaming instead of loading everything  
Instead of returning a full list, I would use a generator.

Process one row at a time.

That way, I don’t keep the whole dataset in memory.

2️⃣ Aggregate while reading  
Instead of loading data first and then calculating totals,

I would calculate totals during file reading.

That reduces memory and improves efficiency.

3️⃣ Avoid unnecessary list conversions  
Right now, some methods convert filters into lists.

For large datasets, I would return iterators or generators.

Only convert to list if absolutely needed.

4️⃣ Use a database for very large data  
If the dataset becomes very large,

I would load it into a database.

Use SQL GROUP BY for aggregation.

Databases are optimized for this.

5️⃣ Use optimized libraries  
For analytics-heavy workloads,

I could use Pandas.

It is faster and memory-efficient because it uses optimized C code internally.

6️⃣ Optimize top-N calculation  
Instead of sorting the entire dataset,

I could use a heap-based approach.

That reduces sorting cost from O(n log n) to O(n log k).

---

## Testing Coverage

1️⃣ What You Implemented AND Tested (Strong Areas)

CSV loads correctly  
Malformed date raises error  
Total sales calculation  
Average calculation  
Top N logic  
Monthly grouping  
Date + region + category filtering  
Summary report structure  

2️⃣ Implemented BUT Did NOT Test (Missing Tests)

Missing column in CSV header  
Negative quantity  
Negative unitPrice  
Negative totalAmount  
start_date > end_date  
n <= 0 in getTopSalespersons  
Empty CSV (header only, no rows)  

3️⃣ Missing in Implementation (Not Handled at All)

Duplicate transactionId not checked  
totalAmount mismatch not validated  
Case-insensitive filtering  
Very large file memory handling  

---

## Assumptions

1️⃣ The Dataset Fits in Memory  
The CSV file is small to medium sized and can be fully loaded into memory.

2️⃣ transactionId Is Unique  
Each transaction ID is unique.

3️⃣ Dates Follow ISO Format (YYYY-MM-DD)  
Dates are in ISO format.

4️⃣ Numeric Fields Are Clean and Well-Formatted  
quantity and unitPrice are valid numeric values without commas, currency symbols, or localization formatting.

5️⃣ Data Volume Is Moderate Enough for Full Sorting  
Sorting entire datasets for top-N is acceptable.

Assumption: One Transaction Per Row  

Each row in the CSV represents exactly one complete transaction.  

All fields in that row belong to a single sale.  

There are no multi-line transactions.  

There are no parent-child relationships across rows.
