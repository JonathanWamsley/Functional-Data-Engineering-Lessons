# Functional Data Engineering — a modern paradigm for batch data processing

By: Maxime Beauchemin

Source: [medium link](https://medium.com/@maximebeauchemin/functional-data-engineering-a-modern-paradigm-for-batch-data-processing-2327ec32c42a)

### Batch Data Processing

Historically known as ETL is:  
- time-consuming
- brittle
- unrewarding
- hard to operate
- constantly evolves
- lots of troubleshooting

In this post, he applies the **functional programming paradigm** to data engineering which can bring clairty to the process.  

What is functional programming?  

> In computer science, functional programming is a programming paradigm — a style of building the structure and elements of computer programs — that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data. It is a declarative programming paradigm, which means programming is done with expressions[1] or declarations[2] instead of statements. In functional code, the output value of a function depends only on the arguments that are passed to the function, so calling a function f twice with the same value for an argument x produces the same result f(x)each time; this is in contrast to procedures depending on a local or global state, which may produce different results at different times when called with the same arguments but a different program state. Eliminating side effects, i.e., changes in state that do not depend on the function inputs, can make it much easier to understand and predict the behavior of a program, which is one of the key motivations for the development of functional programming.

Functions are 'pure' meaning they do not have:
- side-effects
- can be written and tested
- reasoned-about
- debugged in isolation
- without the need to understand external context or history of events surrounding execution

**As ETL pipelines grow in complexity, and as a data team grows in numbers, using methodologies that provide clairty is not a luxary, but a necessity**

### Reproducibility

Reproducibility and replicability are the foundation to the scientific method. The greater the claim made using analytics, the greater the scrutiny on the process should be. In order to be able to trust the data, the process by which results were obtained should be transparent and reproducible.

Immutable data long with versioned logic are key to reproducibility.

It allows one to create a story as raw data is inputed into a function that will always work the same way. This means The data can also be traced through a backfilled recreation process.

### Pure tasks

To match batch processing functionally, the first thing is to avoid any form of side-effects outside the scope of the task. A pure task should be **deterministic** and **idempotent**, meaning that it will produce the same result every time it runs or re-runs. This requires forcing an overwrite approach, meaning re-executing a purse task with the same input parameters should overwrite any previous output that could have been left out from a previous run of the same task.  

Having idempotent tasks is vital for the **operability** of pipelines. When tasks fail, or when the compute logic needs to be altered for whatever reason, we need the certainty that re-running a task is safe and won't lead to a double-counting or any other form of bad state. Without this assumption, making decisions about what needs to be re-executed requires the full context and understanding of all potential side-effects of previous executions.  


> comment - I face this problem a lot when using pandas on jupyter. I change a dataframe, and then changing it again will produce an error or give an unintended side-effect  

Note that contrarily to a pure-function, the pure-task is typically not 'returning' an object in the programming sense of the term. In the context of a SQL ELT approach, which has become common now-a-days, it is likely to be simply overwriting a portion of a table (partition). While this may look like a side-effect, you can think of this output as akin to the immutable object that a typical pure-function would return.  

Also, for the sake of simplicity as a best practice, individual tasks should target a single output, the same way that a pure-function should not return a set of heterogenous objects.  

In some cases it may appear difficult to avoid side-effects. One potential solution is to re-think the size of the unit of work. Can the task become pure if it encompasses other related tasks? Can it be purified by breaking down into a set of smaller tasks?  

It's also important that all transistional states within the pure-task are insulated much like locally scoped variables in pure-functions. Multiple instances of the pure task should be fully independent in their exectuion. If temporary tables or dataframes are used, they should be implemented in a way that task instances cannot interfere with one another so they can be parallelized.

### Table partitions as immutable objects

One of the ways to enforce the functional programming paradigm is to use a immutable objects. While purely functional languages will prevent mutations by enforcing the use of immutable objects, it's possible to use functional practices even in a context where the exposed objects are in fact mutable.  

![img](https://miro.medium.com/max/2919/1*259i5-w_VMgk_ktazB9-NQ.png)  

While your database of choice may allow you to update, insert, and delete at will, it does not mean you should. DML operations like UPDATE, APPEND, and DELETE are performing mutations that ripple in side-effects. Thinking of paritions as immutable blocks of data and systematically overwritting paritions is the way to make your tasks functional. A pure task should always fully overwrite a partition as its output.  

Your data store may not have support for physical partitions, but that does not prevent you from taking a functional approach. To work around this limitation, one option is to implement your own paritioning scheme by using different physical tables as the output of your pure tasks that can be UNIONed ALL into views that act as logical tables. Results may vary depending on how smarr your database optimizer is. Alternatively, it's also possible to logically partition a table and to systematically DELETE prior to INSERTing using partitioning key that reflect the parameters used to instantiate the task. Note that where a TRUNCATE PARTITION is typically a 'free' metadata operation, a DELETE operation may be expensive and that should be taken into consideration. If you decide to go that route, note that on many database engines it may be more efficient to check whether the DELETE operation is needed first to avoid unecessary locking.  

Take-away: treat database tables like an immutable object. Only use INSERT not UPDATE, APPEND and DELETE. Even if that means creating a function that overwriting all elements with a new transformed version. You should be able to apply that function repeatably and get the same output. There are cases where DELETE can be used if it is done systematically.  

### A persistent and immutable staging area

The staging area is the conceptual loading dock of your data warehouse, and while in the case a real physical retail-type warehouse you'd want to use a transient staging area and keep it unobstructed, in most modern data warehouses you'll want to accumulate and persist all of your source data there, and keep it unchanged forever.  

Given a persistent immutable staging area and pure tasks, in theory it's possble to recompute the state of the entire warehouse from scratch (not that you should), and get to the exact same state. Knowing this, the retention policy on derived tables can be shorter, knowing that it's possible to backfill historical data at will  

### changing logical over time  

Business logic and related computations tend to change over time, sometimes in retroactive fashion, sometimes not. When retroactive, you can think of the change as making existing downstream paritions 'dirty' and it call from them to recomputed. When non-retroactive, the ETL logica should capture the change and apply the proper logic to the corresponding time range. Logic that changes over time should always be captured inside the task logic and be applied conditionally when instantiated. This means that ideally the logic in source control describes how to build the full state of the data warehouse throughout all time periods.  

example - if a change is introduced as a new rule around how to calculate taxes for 2018 that should not be applied prior to 2018, it would be dangerous to simply update the task to reflect his new moving forward. If someone else was to introduce an unrelated change that required backfilling 2017, they would apply the 2018 rulle to 2017 data without knowing. The solution here is to apply conditional logic within the task with a certain effective date, and depending on the slice of data getting computed, the extra tax log would apply where needed.  

Take-away: create a function that accepts a condition parameter and active date
def func taxes(data, year, tax_operation) => tax report  

Also note that in many business rules changes over time are best expressed with data as opposed to code. In those cases it's desirable to store this information in 'parameter tables' using effective dates, and have logic that joins and apply the right parameters for the facts being processed. An oversimplfied example of this would be a 'taxation rate' table that would detail the rates and effect periods, as opposed to hard-coded rates wrapped in conditional blocks of code.  

Take-away: have the data use data columns to extract from. Don't hard code operations subject to change.  

### But what about dimensions?

By nature, dimensions and other referential data is slowly changing, and it's tempting to model this with UPSERTs (UPdating what has changed and inSERTing new dimension members) or to apply other slowly-changing-dimension methodology to reflect changes while keeping track of history.  

But how do we model this in a functional data warehouse without mutating data?  

Simple. With demension snapshots where a new partition is appended at each ETL schedule. The dimension table becomes a collection of dimension snapshots where each partition contains the full dimension as-of a point in time. 'But only a small percentage of the data changes every dayy, that's a lot of data duplication!'. That's right, though typically dimension tables are negligible in size in proportion to facts. It's also an elegant way t osolve SCD-type problematic by its simplicity and reproducibility. Now that storage and compute are dirt cheap compared to engineering time, snapeshotting dimensions make sense in most cases.  

While the traditional type-2 slowly changing dimension approach is conceptially sound and may be more computationallly effecient overall, it's cumbersome to manage. The processes around this approach, like managing surrogate keys on dimensions and performing surrogate key lookup when loading facts, are error-prone, full of mutations and hardly reproducible.  

In case of very large dimensions, mixing the snapshot approach along with SCD-type methodology may be reasonable or necessary. Though in general it's sufficient to have the current attribute only in the current snapshot, and denormalize dimension attributes into fact tables where the attribute-at-the-time-of-event is important. FOr the rare cases where attribute-at-the-time-of-the-event was not forseen and denormalized into fact upfront, you can always run a more expensive query that joins the facts to their-relative dimension snapshots as opposed to the latest snapshot.  

You could almost think about the 'dimension snapshotting' approach as a type of further denormalization that has similar tradeoffs as other types of denormalization.

Another modern approach to historical values in dimensions is to use complex or nested data types. For example if we wanted to keep track of the customer's state over time for tax purposes, we could introduce a 'State_history' colum as a map where keys are effective dates and values are the state. This allows for histroy tracking without altering the grain of the table like a typpe-t SCD requires, and is more dynamic than a type 3 SCD since it does not require creating a new column for every piece of information.

### Unit of work

Clairity around the unit of work is also important, and the unit of work should be directly aligned to output to a single parition. This makes it tribial to map each logic table to a task, and each partition to a task instance. By being strict in this area it makes it easy to directly map each partitions in your data store to its corresponding compute logic.  

![img](https://miro.medium.com/max/1182/1*zxAqj8glCHIyqle8HMepkw.png)  

For example, in this Airflow 'tree view' where squares represent task instances of a DAG of tasks over time, it's comforting to know that each row represents a task that corresponds to a table, and that each cell is a task instance that corresponds to a partition. it's easy to track down the log file that correspond to any given partition. Given that tasks are idempotent, rerunning any of these cells is a safe operation. This set of rules makes everything easier to maintain since a clear scope on the units of work make everything clear and maintainable.  

As a side note, Airflow was designed with functional data engineering in mind, and provides a solid framework to orchrestrate idempotent, pure-tasks at scale. The author of this post also happens to be the creator of Airflow, so no wonder the tooling works well with the methodology prescribed here.  

### A DAG of partitions

# Outside Notes  

Source: [stack exchange](https://softwareengineering.stackexchange.com/questions/293851/what-is-it-about-functional-programming-that-makes-it-inherently-adapted-to-para)

Questions: What is it about functional programming that makes it inherently adapted to parallel execution?  


The main reason is that referential transparency (and even more so laziness) abstracts over the execution order. This makes it trivial to parallelize evaluation.

For example, if both a, b, and || are referentially transparent, then it doesn't matter if in

a || b
a gets evaluated first, b gets evaluated first, or b doesn't get evaluated at all (because a was evaluated to true).

In

a || a

it doesn't matter if a gets evaluated once or twice (or, heck, even 5 times … which wouldn't make sense, but doesn't matter nonetheless).

So, if it doesn't matter in which order they are evaluated and it doesn't matter whether they are evaluated needlessly, then you can simply evaluate every sub-expression in parallel. So, we could evaluate a and b in parallel, and then, || could wait for either one of the two threads to finish, look what it returned, and if it returned true, it could even cancel the other one and immediately return true.

Every sub-expression can be evaluated in parallel. Trivially.

Note, however, that this is not a magic bullet. Some experimental early versions of GHC did this, and it was a disaster: there was just too much potential parallelism. Even a simple program could spawn hundreds, thousands, millions of threads and for the overwhelming majority of sub-expressions spawning the thread takes much longer than just evaluating the expression in the first place. With so many threads, context switching time completely dominates any useful computation.

You could say that functional programming turns the problem on its head: typically, the problem is how to break apart a serial program into just the right size of parallel "chunks", whereas with functional programming, the problem is how to group together parallel sub-programs into serial "chunks".

The way GHC does it today is that you can manually annotate two subexpressions to be evaluated in parallel. This is actually similar to how you would do it in an imperative language as well, by putting the two expressions into separate threads. But there is an important difference: adding this annotation can never change the result of the program! It can make it faster, it can make it slower, it can make it use more memory, but it cannot change its result. This makes it way easier to experiment with parallelism to find just the right amount of parallelism and the right size of chunks.
