- [x] #q one of methods does support windowing and watermarking? ✅ 2025-08-14

- [ ] #q is options for custom aggregations?

- [ ] what is role of reduce functions? where in the general big picture they fit? compare theire functions in different scenarios, batch and structured streaming, in related to working with dataframe and dataset

- [ ]  **declarative aggregations** (like in SQL), while `KeyValueGroupedDataset` is a type-safe handle for applying **custom, programmatic logic** to groups of whole objects. i see some conflicts with this and existance of both custom aggregators for both of them

- [ ] when we do groupByKey does shuffle happen? In this paragraph: This is the object you get when you call `.groupByKey()` on a typed `Dataset<T>`. It represents a collection of groups where each group contains all the original objects (`T`) that share a common key (`K`).
- [ ]  what is concept of group? is it partition? or it is values distributed between multiple partitions maybe in different machines that share same key?
	The design prioritizes flexibility. Spark must collect all objects for a given key on a single machine before your custom function can run. This often prevents the pre-aggregation optimizations seen with `RelationalGroupedDataset`, potentially leading to lower performance and higher memory usage if the groups are large.
	this has contractions with method in custom aggregation. doesn't it?
- [ ] next question about agg methods on KeyValueGropedDataSet

- [ ]  on `KeyValueGroupedDataset`, You don't use `.agg()`. instead you use mapGroups, flatMapGroups reduceGroups and mapGroupsWithState, flatMapGroupsWithState

- [ ]  this list is excellent. ask to repeate first question for this liest of 5 methods in `KeyValueGroupedDataset`

- [ ] about `DataSet.reduce()` and `KeyValueGroupedDataSet.reduceGroups()`
- [ ] what is difference between agg options in KeyValueGroupedDataset
- [ ] input of agg is TypedCoulumn this leads me to the concept that argument is not only custom aggregator, what are other options?




