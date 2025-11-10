# Assignment 3 Report

## Team Members

Please list the members here

## Responses to questions posed in the assignment

_Note:_ Include the Spark execution history for each task. Name the zip file as `assignment-3-task-<x>-history.zip`.

### Task 1: Word counting

1. If you were given an additional requirement of excluding certain words (for example, conjunctions), at which step you would do this and why? (0.1 pt)
If we were required to exclude specific words, we would do it immediately after the flatMap step (Step A), so before mapping to pairs. At that point, we already split the text into words, so it’s easy to filter them out.


2. In Lecture 1, the potential of optimizing the mapping step through combined mapping and reduce was discussed. How would you use this in this task? (in your answer you can either provide a description or a pseudo code). Optional: Implement this optimization and observe the effect on performance (i.e., time taken for completion). (0.1 pt)
Instead of sending (word, 1) for every single word,
each mapper first counts how many times each word appears in its own part of the data.
Then it only sends one pair per word, like (word, total count). Hence less data needs to move across the network
and the reduce step becomes faster because it already gets partial sums.
Our Pseudocode:
map(document):
    localCounts = {}
    for each word w in document:
        localCounts[w] = localCounts.get(w, 0) + 1
    emit all (w, localCounts[w])

reduce(word, counts):
    emit(word, sum(counts))


3. In local execution mode (i.e. standalone mode), change the number of cores that is allocated by the master (.setMaster("local[<n>]") and measure the time it takes for the applicationto complete in each case. For each value of core allocation, run the experiment 5 times (to rule out large variances). Plot a graph showing the time taken for completion (with standard deviation) vs the number of cores allocated. Interpret and explain the results briefly in few sentences. (0.4 pt)
Find our plot png (Cores vs Times (ms)) inside the Assignment 3 folder.
Interpretation:
From 1 to 3 cores, the program gets faster.
At 4 cores, the time suddenly decreases surprisingly, probably due to system overhead.
With 6 and 8 cores, the time is almost the same and among the fastest.
This means Spark scales well until a point, then extra cores don’t help much.



5. Examine the execution history. Explain your observations regarding the planning of jobs, stages, and tasks. (0.4 pt)


### Task 2

1. For each of the above computation, analyze the execution history and describe the key stages and tasks that were involved. In particular, identify where data shuffling occurred and explain why. (0.5pt)


2. You had to manually partition the data. Why was this essential? Which feature of the dataset did you use to partition and why?(0.5pt)
This was essential because we used window functions, which need to know where one group ends and the next begins.
If we didn’t partition by month, it would compare CO2 values between months, which makes no sense for finding hourly changes within the same month. This avoids extra data shuffling, which is slow and expensive. Hence keeping related data on the same node, which makes the job faster.

3. Optional: Notice that in the already provided pre-processing (in the class DatasetHelper), the long form of timeseries data, i.e., with a column _field that contained values like temperature etc., has been converted to wide form, i.e. individual column for each measurement kind through and operation called pivoting. Analyze the execution log and describe why this happens to be an expensive transformation.

### Task 3

1. Explain how the K-Means program you have implemented, specifically the centroid estimation and recalculation, is parallelized by Spark (0.5pt)
Spark runs the K-Means steps on many nodes at the same time. In each iteration, the current centroids are broadcast to all worker nodes. Then every worker does the same work in parallel, basically it finds the nearest centroid for the data points it stores (the E-step).
Next, each worker groups its assigned points and computes partial averages for new centroids.
Spark then reduces these partial results from all workers to form the final new centroids (the M-step).
This means that both the assignment and centroid recalculation are distributed,
each partition works independently and Spark combines the results.   


## Declarations (if any)
Used ChatGPT for the following:
- Understanding setup instructions
- Assistence with understanding how spark and docker works
- Understanding the raw code skeleton
