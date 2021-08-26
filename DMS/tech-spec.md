#  Document Manager Stats [Enhancement]

## Overview
In this enhancement we are going to make DMS Stats more configurable by allow the app owner set his desired stats, and make the plugin support offline mode with the last 5 entries cached at any time. (make it easy to change from 5 to X in the future)


## Configure DMS Stats

- Add new tab called **Content** that will contain sortable list of stats.

- Allow app owner to add predefined stats: 
  
   * Total number of submissions.
   * Total score
   * Average score (All time)
   * Average score (over the last 30 days)
   * Current streak
   * Longest streak

-  Add custom status: this will allow app owner to create stats for specific questions with ability to select proper `calculation (Aggregation)`,
 `formatting` and `target value`:

   * Specify the question: `FTQ` should expose Question ID for app owner, so he can go to copy and paste it in custom stats form 
   * Calculations (Aggregations): Min, Max, Average, Sum
   * Formatting: Number, Duration (HH:MM:SS)
   * Value: Score, Answer's Numeric Value

- Custom status will not work when "Allow Override Questionnaire" setting is turned on
   - custom option will be disabled
   - greys out and says "Does not work with validation"


### Calculations (Aggregations)

Aggregation operations process data records and return computed results. Aggregation operations group values from multiple documents together, and can perform a variety of operations on the grouped data to return a single result.

`The operations are  Min, Max, Average and Sum`
 
In general **Aggregation** done on Database side which provides ways to perform it.

In our case we want to perform the Aggregation for `User's FTQ Submissions` for specific question, and as we know we save FTQ Submissions in `AppData` storage service and now our storage services don't support  `Aggregation operations`

So to perform the Aggregation we have multiple suggested solutions:

**Solution 1 (Recommended)**

 - Make `AppData` storage service support Aggregation by implement this feature on the core system
 
 - Pros:
   * Easy to perform the Aggregation for huge data on plugin, without need to load all submissions and perform it on app side. 
 
 - Cons: 
   * Might have performance issue on DB side by consider the prod data the we have and indexing. 
   * Might this feature take time to be implemented  on core system with consider the scope, consistency and the way that we need to implement
   
 - I talked with Ayman about that and he told me we can implement this feature but it's not easy and I can work with it within  my working time or my 
 plugin time. he told me to ask Danny about that`.


**Solution 2**
   
  - Implement the aggregation on App Side (Client), each time the plugin open or app owner add new stats will load all submissions in background and 
   calculate app owner's stats, maybe we can use the cache for enhancing the performance, for example load all submission at first time then cache the 
   stats result for them with point for last submission. 

  - Pros
    * No additional works need to be implemented on core system, the solution will be within plugin scope

  - Cons
    * Might have performance issue with huge submissions
    * Each time app owner add new stats need to load all data to do the calculation for it

  - Might have limitation for stats calculation if we need to enhance  the performance.


### Formatting

  - Always save the stats result as number then handle the formatting  based on user option (Number, Time, Currency) .


### Data Classes

```
  class Stats {
     id: string;
     title: string;
     type: Custom | Predefined;
     aggregation: Min | Max | Average | Sum;
     questionId: string;
     aggregationValue: Score | Answer's Numeric Value;
     outputFormat: Number | Duration (HH:MM:SS);
     isActive: boolean;
  } 

```

 - Will use `Datastore` to save the array of stats, we have two options:

   * Save configurable stats to existing Setting Document (Datastore) as Array 
     
      Current 
      ```
        class Setting {

          showScores: boolean;
          showStats: boolean;
          show1stAnswerInTitle: boolean;
          ftq: FTQ;
          allowMultipleTakes: boolean;
          isOverridesActive: boolean;
          customFtqs: FTQ[];
        }
      ```
      Updated
      ```
         class Setting {
           ....
           ....
           stats: Stats[];
         }
      ```

   * Save configurable stats to new Document called `ContentConfig`.    
      ```
         class ContentConfig{
            stats: Stats[];
          }
      ```



## Support Offline

- Using file system for caching.

- Cache last 5 entries  at any time. (make it easy to change from 5 to X in the future).

- Cache DMS settings (Default FTQ, Ovrride FTQs, Stats, etc) to allow DMS work with Offline FTQ.

- Show dialog for the user to ask him if he need to save submission that made in offline mode.



#  Document Manager Stats V2.0 [Geolocation]

## Overview 

We will fork Document Manager Stats to create a custom plugin: Document Manager Stats [Geolocation] that will support showing google maps and make some enhancement on app Layouts.


## Approch

- Support showing Map on the top of the screen.

- Show on Map all markers of all location entries made on the FTQ.

- Map must zoom to show all locations in the initial viewport
 
- change showing submission details to be in a  new page instead of the expand way.

- DMS will display the total duration in HHH:MM:SS format.

- Config "Total Dive Time" stats that is a sum of score.

- Config "Total Dives" stats that is "Total Submissions" by default.


## Support Offline

- Map must be graceful during offline mode

- Expand the cache limit for last 200 location entries display on the map instantly with our pagination.



