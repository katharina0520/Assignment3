# Assignment 3 Report

## Team Members

Anastasia Eric, Katharina Wette

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

4. Examine the execution history. Explain your observations regarding the planning of jobs, stages, and tasks. (0.4 pt)
The WordCount job ran as one Spark job with two stages.
Stage 0 included flatMap and map, which were local tasks on each partition (no data movement).
Stage 1 started with reduceByKey, which required shuffling data so that all words with the same key were sent to the same worker for counting.
Each stage had several parallel tasks, one per partition.
Stage 0 processed the input, and Stage 1 combined the results and sent them back to the driver.

### Task 2

1. For each of the above computation, analyze the execution history and describe the key stages and tasks that were involved. In particular, identify where data shuffling occurred and explain why. (0.5pt)
As we can see in the Spark History server the columns Shuffle Read and Shuffle Write list all stages where shuffling occured. We noticed the largest Bytes occured in stages:
Data shuffling occurred in stages where Spark had to redistribute data across executors due to wide dependencies.
The first major shuffle appeared during the groupBy("month", "hour") and orderBy("month", "hour") in stages (stage id): 8, 10, 13, 17, 18, 20, 23, 43, 45 with around 70-76 MiB (at Show), where Spark grouped sensor readings by month and hour to compute average CO₂ levels.
A second shuffle occurred in Stage ID 67, 77, 85, 95, 97, 100, 104 with around 70-76 MiB triggered by groupBy("month") and orderBy("month") in Step C, which required reorganizing data by month to calculate maximum and minimum hourly CO₂ changes.
The largest and most relevant shuffles occurred in Stage ID 0, 2, 7 bei pivot with around 70-76 MiB

3. You had to manually partition the data. Why was this essential? Which feature of the dataset did you use to partition and why?(0.5pt)
This was essential because we used window functions, which need to know where one group ends and the next begins.
If we didn’t partition by month, it would compare CO2 values between months, which makes no sense for finding hourly changes within the same month. This avoids extra data shuffling, which is slow and expensive. Hence keeping related data on the same node, which makes the job faster.

4. Optional: Notice that in the already provided pre-processing (in the class DatasetHelper), the long form of timeseries data, i.e., with a column _field that contained values like temperature etc., has been converted to wide form, i.e. individual column for each measurement kind through and operation called pivoting. Analyze the execution log and describe why this happens to be an expensive transformation.

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
